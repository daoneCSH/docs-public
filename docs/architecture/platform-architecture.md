# Platform Architecture

DADP는 정책 원본을 관리하는 제어면, 암복호화 요청을 처리하는 데이터면, 그리고 애플리케이션 또는 데이터베이스 실행 경로에 개입하는 연동 경계를 분리해서 설계한다. 이 분리는 기능 분업을 위한 것이기도 하지만, 장애 해석과 운영 책임을 분명히 하기 위한 구조이기도 하다.

## 아키텍처 개요

```text
Control Plane
  Hub
    -> policy, key metadata, mapping, operations

Data Plane
  Engine
    -> encryption, decryption, masking execution
  Aggregator (optional)
    -> analytics and SQL insight

Integration Paths
  Wrapper
  Direct API
  DB UDF
```

## 핵심 구성요소

| 구성요소 | 역할 |
|----------|------|
| Hub | 정책, 키 메타데이터, 연동 매핑, 운영 상태의 원본 |
| Engine | 런타임 암복호화 실행 |
| Wrapper | JDBC 경계에서의 애플리케이션 연동 |
| Direct API | 애플리케이션 코드에서의 명시적 런타임 호출 |
| DB UDF | 데이터베이스 SQL 경계에서의 연동 |
| Aggregator | 선택적 통계와 SQL Insight 경로 |

## 동작 원칙

### 1. 정책 원본과 실행 사본은 다르다

정책과 키 메타데이터의 원본은 Hub에 있다. Engine은 이를 런타임 캐시 형태로 받아 실행에 사용한다. 따라서 Hub와 Engine은 같은 정보를 다루더라도 같은 책임을 갖지 않는다.

### 2. 실행 요청은 데이터면에서 끝난다

Direct API, Wrapper, DB UDF는 진입 위치만 다를 뿐 최종 실행 주체는 Engine이다. 공개 문서에서 세 연동 방식을 분리하는 이유도 진입 경계가 다르기 때문이다.

### 3. 제어면 장애와 실행면 장애는 분리해서 본다

Hub 문제가 곧바로 모든 암복호화 실패를 의미하는 것은 아니다. 반대로 Engine 문제가 생기면 세 연동 경로가 동시에 영향을 받을 수 있다.

## 현재 구현 기준 운영 모드

### Connected

Connected 환경에서는 Hub가 연결형 검증과 신뢰 배포 경로를 사용한다. 공개 문서는 내부 서비스 이름 대신, 연결형 운영이 필요한 제어면 기능과 복구 의미에 집중해서 설명한다.

### Air-Gapped

Air-Gapped 환경에서는 Hub가 배포 경계 내부에서 운영 상태를 해석한다. 이때 신뢰 자산과 검증 절차도 내부 배포 기준으로 운영된다.

## 운영자가 봐야 할 구조적 포인트

- 운영자 진입점은 Hub에 있다.
- 런타임 실행 상태는 Engine에서 판정한다.
- Wrapper와 DB UDF는 각각 애플리케이션 경계와 데이터베이스 경계에 종속된다.
- Aggregator는 핵심 실행 경계가 아니라 관측 경계다.

## 관련 문서

- [Control Plane and Data Plane](control-plane-and-data-plane.md)
- [Service Communication](service-communication.md)
- [Module Overview](../modules/module-overview.md)
