# Server Installation

서버 측 공개 설치 문서는 Hub와 선택적 Aggregator를 대상으로 한다. 이 단계의 목적은 제어면과 관측면을 먼저 안정화한 뒤, 그 위에 Engine 및 연동 모듈 검증을 이어갈 수 있는 상태를 만드는 것이다.

## 설치 대상

- Hub
- Aggregator, 필요한 경우

## 권장 순서

1. Hub 설치
2. Hub 기본 기동 및 운영 진입 확인
3. Aggregator 설치
4. Aggregator 수집 및 조회 경로 확인

## Hub 설치 체크리스트

### 1. 배포 산출물 준비

- JAR 또는 컨테이너 이미지 준비
- Hub 설정 파일 또는 환경변수 준비
- Meta DB와 Redis 접근 정보 준비

### 2. 기본 기동 확인

Hub는 기본적으로 `9004` 포트와 `/hub` context-path를 사용한다.

```bash
curl http://localhost:9004/hub/actuator/health
```

### 3. 운영 진입 확인

- Hub UI 진입이 가능한지 확인
- 제어면 API가 응답하는지 확인
- 정책/키/연동 조회 화면이 정상인지 확인

### 4. 후속 모듈 연동 준비

- Engine이 Hub를 참조할 수 있는 URL 정합성 확인
- 운영자 진입 도메인과 내부 연동 URL을 혼동하지 않도록 정리

## Aggregator 설치 체크리스트

- 통계 저장소 연결 정보 준비
- 수집 엔드포인트와 조회 엔드포인트 분리 확인
- 필요한 경우 Hub 또는 Engine에서 통계 전송 URL 구성

## 설치 완료 기준

- Hub 프로세스 또는 컨테이너가 정상 기동한다.
- `GET /hub/actuator/health`가 정상 응답한다.
- 운영자 진입 경로가 열린다.
- 선택 구성인 Aggregator를 포함한 경우 수집과 조회 경로가 모두 성립한다.
