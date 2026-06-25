# DB UDF Tutorial

이 문서는 DB 내부 SQL 경로에서 DADP DB UDF를 호출하는 공식 튜토리얼이다.

Wrapper는 애플리케이션 JDBC 경로를 담당한다. 저장 프로시저, 함수, 트리거, 배치 SQL처럼 DB 서버 내부에서 실행되는 로직은 Wrapper가 대신 처리하지 않는다. 이 경로에서 암복호화가 필요하면 DB UDF를 사용한다.

## 1. Core Contract

DB UDF의 현재 제품 코어는 두 가지다.

| 구분 | 의미 |
| --- | --- |
| Single primitive | 값 1건을 Engine에 보내고 결과 1건을 받는다. |
| Batch primitive | `items` 배열을 가진 request JSON을 Engine batch endpoint로 보내고 결과 JSON을 받는다. |

batch는 테이블명, source column, target column, `where` 조건을 받아 테이블을 직접 스캔하거나 업데이트하는 시나리오가 아니다.

프로시저가 대상 데이터를 읽고 request JSON을 만든다. DB UDF는 primitive batch 처리만 수행한다.

## 2. Common Function Surface

벤더별 SQL 문법은 다를 수 있지만 공개 문서에서는 다음 기능명을 기준으로 설명한다.

| 기능 | 설명 |
| --- | --- |
| `encrypt` | 단건 암호화 |
| `decrypt` | 단건 복호화 |
| `batch_encrypt` | request JSON 기반 다수건 primitive 암호화 |
| `batch_decrypt` | request JSON 기반 다수건 primitive 복호화 |

PostgreSQL 기준 함수명은 다음과 같다.

| 기능 | PostgreSQL function |
| --- | --- |
| 단건 암호화 | `dadp_encrypt(data, policy)` |
| 단건 복호화 | `dadp_decrypt(data)` |
| 배치 암호화 | `dadp_batch_encrypt(request_json[, batch_size])` |
| 배치 복호화 | `dadp_batch_decrypt(request_json[, batch_size])` |

## 3. Primitive Batch Request

암호화 request JSON:

```json
{
  "items": [
    { "data": "alpha", "policyName": "default-policy" },
    { "data": "beta", "policyName": "default-policy" }
  ]
}
```

복호화 request JSON:

```json
{
  "items": [
    { "data": "encrypted-a" },
    { "data": "encrypted-b" }
  ]
}
```

응답은 벤더별 구현과 내부 transport에 관계없이 결과 JSON으로 반환된다. 호출자는 결과 JSON을 해석해 필요한 테이블 업데이트나 후속 처리를 수행한다.

## 4. PostgreSQL Single Procedure

PostgreSQL DB UDF는 SQL function으로 설치된다. 따라서 PL/pgSQL procedure 내부에서 일반 SQL 함수처럼 호출할 수 있다.

아래 예시는 고객 카드 값을 읽고 단건 암호화 결과를 별도 컬럼에 저장한다.

```sql
CREATE OR REPLACE PROCEDURE encrypt_customer_card(
  IN p_customer_id BIGINT,
  IN p_policy TEXT
)
LANGUAGE plpgsql
AS $$
DECLARE
  v_plain TEXT;
  v_encrypted TEXT;
BEGIN
  SELECT card_no
  INTO v_plain
  FROM customer_card
  WHERE customer_id = p_customer_id;

  v_encrypted := dadp_encrypt(v_plain, p_policy);

  UPDATE customer_card
  SET card_no_enc = v_encrypted
  WHERE customer_id = p_customer_id;
END;
$$;
```

호출 예시:

```sql
CALL encrypt_customer_card(1001, 'default-policy');
```

## 5. PostgreSQL Primitive Batch Procedure

아래 예시는 프로시저가 입력 배열을 받아 primitive batch request JSON을 만든 뒤 `dadp_batch_encrypt`를 호출한다.

```sql
CREATE OR REPLACE PROCEDURE encrypt_values_batch(
  IN p_values TEXT[],
  IN p_policy TEXT,
  IN p_batch_size INTEGER,
  INOUT p_result JSONB
)
LANGUAGE plpgsql
AS $$
DECLARE
  v_request TEXT;
BEGIN
  SELECT jsonb_build_object(
           'items',
           jsonb_agg(
             jsonb_build_object(
               'data', v,
               'policyName', p_policy
             )
           )
         )::TEXT
  INTO v_request
  FROM unnest(p_values) AS v;

  p_result := dadp_batch_encrypt(v_request, COALESCE(p_batch_size, 1000))::JSONB;
END;
$$;
```

호출 예시:

```sql
CALL encrypt_values_batch(
  ARRAY['alpha', 'beta', 'gamma'],
  'default-policy',
  1000,
  NULL
);
```

## 6. PostgreSQL Primitive Batch Decrypt Procedure

복호화도 동일하다. 프로시저가 암호문 배열을 받아 request JSON을 만들고 `dadp_batch_decrypt`를 호출한다.

```sql
CREATE OR REPLACE PROCEDURE decrypt_values_batch(
  IN p_values TEXT[],
  IN p_batch_size INTEGER,
  INOUT p_result JSONB
)
LANGUAGE plpgsql
AS $$
DECLARE
  v_request TEXT;
BEGIN
  SELECT jsonb_build_object(
           'items',
           jsonb_agg(jsonb_build_object('data', v))
         )::TEXT
  INTO v_request
  FROM unnest(p_values) AS v;

  p_result := dadp_batch_decrypt(v_request, COALESCE(p_batch_size, 1000))::JSONB;
END;
$$;
```

호출 예시:

```sql
CALL decrypt_values_batch(
  ARRAY['encrypted-a', 'encrypted-b'],
  1000,
  NULL
);
```

## 7. Responsibility Boundary

| 항목 | 담당 |
| --- | --- |
| 대상 행 조회 | 고객 DB 프로시저 또는 SQL |
| request JSON 생성 | 고객 DB 프로시저 또는 SQL |
| Engine batch 호출 | DB UDF |
| 결과 JSON 반환 | DB UDF |
| 결과를 테이블에 반영 | 고객 DB 프로시저 또는 SQL |

DB UDF 자체가 테이블을 스캔하거나 컬럼을 업데이트하는 구조는 현재 제품 코어 범위가 아니다.

## 8. When To Use Wrapper Or DB UDF

| 경로 | 사용 대상 |
| --- | --- |
| Wrapper | 애플리케이션 JDBC 경로에서 SQL 입출력을 보호해야 할 때 |
| DB UDF | 프로시저, 함수, 트리거, 배치 SQL처럼 DB 내부 로직에서 암복호화를 호출해야 할 때 |

두 방식은 같은 Engine 기반 암복호화를 사용하지만 개입 지점이 다르다.
