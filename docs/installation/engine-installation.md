# Engine Installation

Engine 설치 검증의 핵심은 단순 기동 여부가 아니라, 런타임 실행과 Hub 동기화가 함께 성립하는지 확인하는 것이다. 설치가 끝났다는 판단은 round trip 성공과 캐시 동기화 가능성까지 포함해야 한다.

## 설치 전 준비

- Java 또는 컨테이너 실행 환경
- `9003` 포트 사용 가능 여부
- Hub 연결 정보
- provider 관련 설정
- 캐시 또는 영속 저장 경로

## 기본 설치 순서

1. 실행 환경 준비
2. Engine 설정 반영
3. Engine 기동
4. 헬스 응답 확인
5. 단건 암복호화 round trip 확인
6. Hub 연결 성공 여부 확인

## 설치 후 검증

### 1. 헬스 확인

```bash
curl http://localhost:9003/api/health
```

### 2. 단건 암호화 확인

```bash
curl -X POST http://localhost:9003/api/encrypt \
  -H "Content-Type: application/json" \
  -d '{"data":"DADP_TEST","policyName":"sample"}'
```

### 3. 단건 복호화 확인

```bash
curl -X POST http://localhost:9003/api/decrypt \
  -H "Content-Type: application/json" \
  -d '{"data":"<encrypted-data>"}'
```

### 4. Hub 연결 상태 확인

Engine은 프로세스가 살아 있어도 Hub 연결이 아직 성립하지 않았을 수 있다. 설치 직후에는 다음 두 조건을 함께 본다.

- Engine 로그에 Hub 헬스체크 성공 또는 Hub 연결 정상 메시지가 남는지 확인
- Hub 준비 완료 이후 Engine이 오프라인 모드로 남아 있지 않은지 확인

Compose로 함께 기동하는 경우에는 시작 순서만 보장되고 Hub 애플리케이션 준비 완료는 보장되지 않는다. Engine이 먼저 올라와 오프라인 모드로 시작한 경우에는 Hub health가 정상으로 바뀐 뒤 Engine을 다시 시작한다.

```bash
docker compose restart engine
```

## 설치 완료 기준

- `/api/health` 응답이 정상이다.
- 단건 암복호화 round trip이 성공한다.
- Engine 로그에서 Hub 연결 성공을 확인할 수 있다.
- Hub가 Engine을 식별하고 정책을 동기화할 수 있다.

## 운영상 주의점

- Engine 생존 여부와 캐시 정합성은 같은 신호가 아니다.
- `/engine/api/v1/cache/sync-check`는 외부 점검용 단일 기준으로 쓰지 않는다. 내부 호출 전제를 가질 수 있어 직접 호출 시 인증 오류로 오해할 수 있다.
- 제어면 URL과 실행면 URL을 같은 값으로 취급하지 않는다.
- Direct API, Wrapper, DB UDF가 모두 같은 Engine을 향한다면 한 번의 Engine 장애가 세 경로 모두에 영향을 줄 수 있다.
