# Engine Operations

Engine 운영의 핵심은 런타임 실행면이 정상인지, 그리고 Hub에서 배포된 상태가 현재 캐시에 반영되어 있는지를 함께 확인하는 것이다.

## 일상 점검 항목

- 프로세스 또는 컨테이너 상태
- `9003` 포트 응답
- `/api/health` 응답
- 단건 암복호화 round trip
- `/engine/api/v1/cache/sync-check` 상태

## 점검 예시

```bash
curl http://localhost:9003/api/health
```

```bash
curl -X POST http://localhost:9003/api/encrypt \
  -H "Content-Type: application/json" \
  -d '{"data":"DADP_TEST","policyName":"sample"}'
```

```bash
curl http://localhost:9003/engine/api/v1/cache/sync-check
```

## 운영 해석 기준

- 정책이 없다고 나오면 먼저 동기화 상태를 본다.
- 복호화 실패면 본문 형식, provider, 버전 정합성을 함께 본다.
- 특정 연동만 실패하면 Engine 자체보다 연동 경계를 먼저 의심할 수 있다.

## 운영자가 주의할 점

- health와 round trip은 같은 신호가 아니다.
- sync-check는 단순 생존 확인보다 더 중요한 신호일 수 있다.
- Direct API, Wrapper, DB UDF에서 공통 증상이 보이면 Engine 공통 원인을 먼저 확인한다.
