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
6. Hub 동기화 가능 여부 확인

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

### 4. 캐시 동기화 상태 확인

```bash
curl http://localhost:9003/engine/api/v1/cache/sync-check
```

## 설치 완료 기준

- `/api/health` 응답이 정상이다.
- 단건 암복호화 round trip이 성공한다.
- `/engine/api/v1/cache/sync-check`로 런타임 동기화 상태를 판정할 수 있다.
- Hub가 Engine을 식별하고 동기화할 수 있다.

## 운영상 주의점

- Engine 생존 여부와 캐시 정합성은 같은 신호가 아니다.
- 제어면 URL과 실행면 URL을 같은 값으로 취급하지 않는다.
- Direct API, Wrapper, DB UDF가 모두 같은 Engine을 향한다면 한 번의 Engine 장애가 세 경로 모두에 영향을 줄 수 있다.
