# Module Overview

공개 DADP 문서는 고객이 실제로 이해하고 운영해야 하는 모듈만 중심에 둔다. 이 기준에서 DADP의 공개 모듈 집합은 Hub, Engine, Wrapper, DB UDF 네 축이다.

## 공개 모듈 집합

| 모듈 | 역할 | 운영 관점의 질문 |
|------|------|------------------|
| Hub | 제어면 | 정책과 운영 상태의 원본이 정상인가 |
| Engine | 런타임 실행 | 암복호화가 정상 실행되는가 |
| Wrapper | JDBC 연동 | 애플리케이션 SQL 경로가 정상 보호되는가 |
| DB UDF | 데이터베이스 연동 | DB 내부 실행 경로가 정상 동작하는가 |

## 모듈 간 관계

- Hub는 정책과 매핑의 원본을 가진다.
- Engine은 Hub에서 동기화된 상태로 실행한다.
- Wrapper는 Hub와 Engine을 함께 참조하지만, 실행 요청은 Engine으로 보낸다.
- DB UDF는 데이터베이스 내부에서 Engine을 호출한다.

## 읽는 순서

1. [Hub](hub.md)로 제어면을 이해한다.
2. [Engine](engine.md)으로 실행면을 이해한다.
3. [Wrapper](wrapper.md)와 [DB UDF](db-udf.md)를 비교해 연동 경계 차이를 이해한다.
