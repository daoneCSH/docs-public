# API Overview

DADP 공개 API는 제어면 API와 런타임 실행 API로 분리된다. 이 구분은 단순한 URL 체계 차이가 아니라, 데이터 소유권과 운영 책임이 다른 두 경계를 구분하기 위한 계약이다.

## 공개 API 표면

| 서비스 | 역할 | 대표 경로 |
|--------|------|-----------|
| Hub | 제어면 API | `/api/v1`, `/api/v2` |
| Engine | 런타임 실행 API | `/api`, `/engine/api/v1/cache` |

## 경계 규칙

1. Hub API는 정책, 키 메타데이터, 연동 등록, 운영 상태를 다룬다.
2. Engine API는 암복호화 실행과 런타임 캐시 동기화를 다룬다.
3. 제어면에서 쓰는 식별자와 런타임 요청 본문은 같은 계약이 아니다.
4. Direct API, Wrapper, DB UDF는 모두 최종적으로 Engine 실행 API를 사용한다.

## 문서 읽는 순서

1. [Hub API](hub-api.md)에서 제어면 경계를 먼저 이해한다.
2. [Engine API](engine-api.md)에서 런타임 호출 계약을 확인한다.
3. [Wrapper Integration](../integrations/wrapper-integration.md)과 [DB UDF Integration](../integrations/db-udf-integration.md)으로 실제 연동 경로 차이를 본다.
