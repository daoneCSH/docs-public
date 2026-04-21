# Platform Overview

DADP는 정책 제어와 런타임 실행을 분리해 운영하는 데이터 보호 플랫폼이다. Hub는 정책과 운영 상태를 관리하고, Engine은 실제 암복호화 요청을 처리한다. Wrapper, Direct API, DB UDF는 애플리케이션과 데이터베이스의 실행 경로를 런타임 계층에 연결하는 공식 연동 방식이다.

## 플랫폼 계층

| 계층 | 주요 구성 요소 | 역할 |
|------|----------------|------|
| 제어면 | Hub | 정책, 키 메타데이터, 연동 매핑, 운영 상태 관리 |
| 데이터면 | Engine, Aggregator | 암복호화 실행과 선택적 관측 데이터 처리 |
| 연동 계층 | Wrapper, Direct API, DB UDF | 애플리케이션 및 데이터베이스 경계에 맞춘 실행 경로 연결 |

## 운영 원칙

1. 정책은 제어면에서 정의하고 관리한다.
2. 암복호화 실행은 데이터면에서 처리한다.
3. 연동 방식은 진입 경계를 바꾸지만 실행 주체를 바꾸지는 않는다.

## 공식 연동 경로

| 방식 | 진입 경계 | 적합한 상황 |
|------|-----------|-------------|
| Wrapper | JDBC 경계 | 애플리케이션 측 데이터베이스 접근 경로를 최소 수정으로 보호할 때 |
| Direct API | 애플리케이션 서비스 코드 | 코드에서 암복호화 호출 시점과 범위를 명시적으로 제어할 때 |
| DB UDF | 데이터베이스 SQL 경계 | 데이터베이스 내부 SQL 실행 경로에서 직접 보호할 때 |

## 다음 문서

- [Core Concepts](core-concepts.md)
- [Platform Architecture](../architecture/platform-architecture.md)
- [Integration Overview](../integrations/integration-overview.md)
