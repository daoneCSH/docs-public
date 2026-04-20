# API Overview

DADP 공개 API는 제어면 API와 실행면 API를 분리해 문서화한다. 이 구분은 URL 체계의 차이가 아니라 원본 데이터의 소유권, 호출 주체, 장애 해석 방식이 다르기 때문이다.

## 공개 API 표면

| 서비스 | API 경계 | 대표 경로 | 주요 호출 주체 |
|--------|----------|-----------|----------------|
| Hub | 제어면 API | `/api/v1`, `/api/v2` | 운영자 UI, 제어면 연동 클라이언트 |
| Engine | 실행면 API | `/api`, `/engine/api/v1/cache` | Direct API, Wrapper, DB UDF, Hub 동기화 작업 |

## API 경계 규칙

1. Hub API는 정책, 키 메타데이터, 연동 등록, 운영 상태 같은 제어면 원본을 다룬다.
2. Engine API는 암복호화 실행과 런타임 캐시 동기화를 다룬다.
3. Hub에서 쓰는 식별자 계약과 Engine 실행 요청 본문은 동일한 모델이 아니다.
4. Direct API, Wrapper, DB UDF는 모두 최종적으로 Engine 실행 API를 사용한다.

## 운영자가 구분해야 할 것

### Hub API

- 원본 상태를 생성, 변경, 조회한다.
- 운영자 인증과 권한 모델의 영향을 직접 받는다.
- 정책 변경 미반영, 인스턴스 등록 오류, 운영 설정 문제를 진단할 때 먼저 본다.

### Engine API

- 실제 암복호화 요청을 처리한다.
- 실행 지연, 실패, 캐시 불일치, 실행 오류를 진단할 때 먼저 본다.
- 프로세스 생존 여부만이 아니라 캐시 상태와 실행 계약을 함께 봐야 한다.

## 계약 해석 원칙

- 제어면 성공이 곧 실행면 성공을 의미하지는 않는다.
- 실행면 정상도 제어면 원본이 최신 상태라는 뜻은 아니다.
- 같은 HTTP 오류라도 어느 경계에서 발생했는지에 따라 운영 해석이 달라진다.
- 연동 경로 문서는 API 문서를 대체하지 않는다. 연동 문서는 호출 위치를 설명하고, API 문서는 계약 자체를 설명한다.

## 읽는 순서

1. [Hub API](hub-api.md)에서 제어면 경계와 원본 데이터를 먼저 이해한다.
2. [Engine API](engine-api.md)에서 실행 요청과 캐시 동기화 계약을 확인한다.
3. [Wrapper Integration](../integrations/wrapper-integration.md)과 [DB UDF Integration](../integrations/db-udf-integration.md)으로 실제 호출 경계 차이를 본다.
