# DB UDF Integration

DB UDF 연동은 데이터베이스 내부 SQL 경계에서 DADP를 호출하는 방식이다. 애플리케이션 계층이 아니라 DB 함수, 패키지, 프로시저, 트리거, 배치 SQL이 실행 주체가 된다.

Wrapper는 애플리케이션 JDBC 경로를 담당한다. DB 서버 내부의 프로시저, 트리거, 함수에서 암복호화를 호출해야 하는 경우에는 DB UDF를 사용한다.

## 연동 특성

- DB 함수 또는 패키지가 Engine을 직접 호출
- 설치 계약이 DB 벤더별로 달라질 수 있음
- 실행 품질이 DB ACL과 네트워크 상태에 크게 영향받음
- SQL 경로 안에서 직접 결과를 반환

## Primitive Batch Contract

DB UDF의 batch는 테이블명, source column, target column, `where` 조건을 받아 DB가 테이블을 스캔하고 업데이트하는 시나리오가 아니다.

현재 공식 batch 계약은 primitive request JSON이다.

- 호출자가 `items` 배열을 가진 request JSON을 만든다.
- DB UDF는 request JSON을 Engine batch endpoint로 전달한다.
- DB UDF는 Engine 응답을 JSON 결과로 반환한다.
- 대상 데이터 조회, `request_json` 생성, 결과 반영은 호출 프로시저나 SQL 로직의 책임이다.

암호화 request 예시는 다음과 같다.

```json
{
  "items": [
    { "data": "alpha", "policyName": "default-policy" },
    { "data": "beta", "policyName": "default-policy" }
  ]
}
```

복호화 request 예시는 다음과 같다.

```json
{
  "items": [
    { "data": "encrypted-a" },
    { "data": "encrypted-b" }
  ]
}
```

## 적합한 상황

- 보호 로직이 데이터베이스 내부 절차에 있어야 할 때
- 배치 SQL이나 트리거 경로에서 직접 암복호화가 필요할 때
- 애플리케이션 변경보다 DB 경계 통제가 더 중요한 환경

## PostgreSQL Procedure Path

PostgreSQL에서는 `dadp_encrypt`, `dadp_decrypt`, `dadp_batch_encrypt`, `dadp_batch_decrypt`가 SQL function으로 설치된다. 따라서 PL/pgSQL procedure, function, trigger function 내부에서 호출할 수 있다.

자세한 절차와 SQL 예시는 [DB UDF Tutorial](db-udf-tutorial.md)을 따른다.

## 운영상 주의점

- 설치 오브젝트와 검증 절차를 별도로 관리해야 한다.
- Engine URL 변경 시 DB 오브젝트 설정 정합성을 함께 봐야 한다.
- 네트워크와 ACL 문제가 설치 성공 이후 검증 단계에서 드러날 수 있다.
- DB UDF 자체가 테이블 스캔이나 컬럼 업데이트를 수행한다고 전제하지 않는다. 해당 로직은 고객 DB 프로시저나 SQL에서 명시적으로 작성한다.
