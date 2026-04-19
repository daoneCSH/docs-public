# Service Communication

DADP는 운영자 접근 트래픽, 제어면 동기화 트래픽, 런타임 실행 트래픽, 관측 트래픽을 분리해서 다룬다. 공개 문서에서 서비스 간 통신을 따로 설명하는 이유는 URL이 아니라 통신 목적이 다르기 때문이다.

## 통신 매트릭스

| 송신자 | 수신자 | 채널 | 목적 |
|--------|--------|------|------|
| Browser | Hub | HTTP(S) | 운영 UI와 제어면 API |
| Hub | Engine | REST | 캐시 동기화와 상태 확인 |
| Wrapper | Hub | REST | bootstrap, 등록, 설정 동기화 |
| Wrapper | Engine | REST | 런타임 암복호화 요청 |
| Direct API | Engine | REST | 애플리케이션 명시적 실행 요청 |
| DB UDF | Engine | HTTP 또는 DB 브리지 | 데이터베이스 내부 실행 |
| Engine | Aggregator | REST | 실행 통계 전달 |
| Hub UI | Aggregator | REST | 통계와 SQL Insight 조회 |

## 대표 통신 흐름

### Wrapper

1. Wrapper가 Hub에서 필요한 설정과 매핑 정보를 가져온다.
2. Wrapper가 로컬 스냅샷 또는 export-config 기반 설정을 유지한다.
3. 실제 암복호화는 Engine으로 요청한다.

### Direct API

1. 애플리케이션이 helper 또는 공통 라이브러리를 사용해 실행 시점을 결정한다.
2. 런타임 요청을 Engine으로 보낸다.
3. 필요할 경우 제어면 상태는 별도 경로로 조회한다.

### DB UDF

1. 데이터베이스 함수나 패키지가 Engine URL을 참조한다.
2. SQL 실행 중 Engine 호출이 일어난다.
3. 결과를 다시 SQL 경로로 반환한다.

## 경계 규칙

- 브라우저는 Engine을 직접 운영자 진입점으로 사용하지 않는다.
- 모든 실행 요청이 Hub를 매번 경유하는 것은 아니다.
- Aggregator는 실행 제어 경계가 아니라 관측 경계다.

## 신뢰 및 복구 채널

Connected 환경에서는 bootstrap과 steady-state를 분리해서 해석해야 한다.

- bootstrap 채널은 초기화와 복구에 사용된다.
- steady-state 채널은 정상 운영 통신에 사용된다.

복구 채널이 살아 있다는 사실만으로 정상 운영 채널까지 복구됐다고 판단하면 안 된다.
