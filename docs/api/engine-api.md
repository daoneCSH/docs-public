# Engine API

Engine API는 DADP 런타임 실행 경계다. Direct API 호출, Wrapper 실행, DB UDF 실행은 모두 최종적으로 Engine API를 사용한다. 동시에 Hub는 별도의 캐시 동기화 API를 통해 Engine 런타임 상태를 갱신한다.

## API 구분

| 구분 | 대표 경로 | 호출 주체 | 목적 |
|------|-----------|-----------|------|
| 실행 API | `/api/*` | Direct API, Wrapper, DB UDF | 암호화, 복호화, 헬스 점검 |
| 캐시 동기화 API | `/engine/api/v1/cache/*` | Hub | 정책, 키, 마스킹 정책, 버전 동기화 |

## 대표 실행 경로

| 경로 | 메서드 | 역할 |
|------|--------|------|
| `/api/health` | `GET` | 런타임 가용성 점검 |
| `/api/encrypt` | `POST` | 단건 암호화 |
| `/api/decrypt` | `POST` | 단건 복호화 |
| `/api/encrypt/batch` | `POST` | 배치 암호화 |
| `/api/decrypt/batch` | `POST` | 배치 복호화 |
| `/api/encrypt/stream` | `POST` | 스트림 암호화 |
| `/api/decrypt/stream` | `POST` | 스트림 복호화 |

## 대표 캐시 동기화 경로

| 경로 | 메서드 | 역할 |
|------|--------|------|
| `/engine/api/v1/cache/policies/sync` | `POST` | 정책 증분 동기화 |
| `/engine/api/v1/cache/policies/sync-all` | `POST` | 정책 전체 동기화 |
| `/engine/api/v1/cache/keys/sync` | `POST` | 키 증분 동기화 |
| `/engine/api/v1/cache/keys/sync-all` | `POST` | 키 전체 동기화 |
| `/engine/api/v1/cache/mask-policies/sync` | `POST` | 마스킹 정책 증분 동기화 |
| `/engine/api/v1/cache/mask-policies/sync-all` | `POST` | 마스킹 정책 전체 동기화 |
| `/engine/api/v1/cache/status` | `GET` | 현재 캐시 상태 조회 |
| `/engine/api/v1/cache/version` | `GET` | 캐시 버전 조회 |
| `/engine/api/v1/cache/sync-check` | `GET` | 버전 비교 기반 동기화 필요 여부 확인 |

## 단건 호출 계약

### 단건 암호화 요청 예시

```json
{
  "data": "DADP_TEST",
  "policyName": "sample"
}
```

### 단건 복호화 요청 예시

```json
{
  "data": "<encrypted-data>"
}
```

구현에 따라 복호화 요청은 `encryptedData` 필드도 수용할 수 있다. 공개 문서에서는 운영자가 가장 단순한 표준 본문부터 사용하는 것을 권장한다.

## 배치 및 전송 형식

| 모드 | Content-Type | 설명 |
|------|--------------|------|
| 단건 JSON | `application/json` | 단건 암복호화 |
| 배치 JSON | `application/json` | `items[]` 기반 배치 처리 |
| 스트림 | `application/x-ndjson` | 연속 스트림 처리 |
| 바이너리 프레임 | `application/x-dadp-binary-frame` | 고성능 배치 전송 |

## 운영 해석

- Engine은 정책 원본을 소유하지 않는다.
- Engine은 Hub에서 배포된 런타임 캐시를 기준으로 실행한다.
- `sync-check`와 캐시 버전 상태는 단순 프로세스 생존 여부보다 더 중요한 운영 신호다.
- 실행 실패와 캐시 미동기화는 구분해서 진단해야 한다.
