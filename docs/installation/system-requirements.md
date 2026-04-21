# System Requirements

DADP 설치는 단순 패키지 복사 작업이 아니라 제어면, 데이터면, 연동 경로가 어느 환경에서 어떤 경계로 배치될지 먼저 결정하는 작업이다. 이 문서는 공개 모듈 기준의 설치 전 요구사항을 정리한다.

## 설치 범위 결정

먼저 어떤 조합을 설치할지 확정해야 한다.

| 구성 | 필수 여부 | 설명 |
|------|-----------|------|
| Hub | 필수 | 제어면 |
| Engine | 필수 | 런타임 실행 |
| Wrapper | 선택 | JDBC 경로 연동 |
| DB UDF | 선택 | DB SQL 경로 연동 |
| Aggregator | 선택 | 통계와 SQL Insight |

## 공통 요구사항

| 항목 | 설명 |
|------|------|
| 실행 환경 | 서버 또는 컨테이너 런타임 |
| 컨테이너 도구 | Docker Engine과 Docker Compose plugin 사용 가능 여부 |
| 네트워크 | 모듈 간 통신 경로와 방화벽 규칙 |
| 메타데이터 저장소 | Hub용 Meta DB와 Redis |
| 비밀정보 관리 | DB 계정, 런타임 자격정보, 신뢰 자산 |
| 운영 URL | Hub, Engine, 선택적 Aggregator 접근 경로 |

## Container Runtime Readiness

컨테이너 설치를 선택한 경우에는 이미지 선택보다 먼저 런타임 준비 상태를 확인한다.

```bash
docker --version
docker compose version
```

특히 Amazon Linux 2023 계열에서는 `docker` 패키지 설치 후에도 `docker compose`가 바로 제공되지 않을 수 있다. 이 경우에는 Compose plugin을 별도로 준비한 뒤 설치를 진행한다.

## 기본 포트

| 컴포넌트 | 기본 포트 | 비고 |
|-----------|-----------|------|
| Engine | `9003` | 런타임 실행 |
| Hub | `9004` | 제어면, 기본 context-path `/hub` |
| Aggregator | `9010` | 선택 구성 |

## 모듈별 준비사항

### Hub

- Meta DB 연결 정보
- Redis 연결 정보
- `DB_*` 계열 환경변수와 `SPRING_DATASOURCE_*` 계열 환경변수 정합성
- 운영자 진입용 도메인 또는 프록시 경로
- 연결형 또는 에어갭 배포 모드 결정

### Engine

- Hub 연결 정보
- 암복호화 provider 설정
- 캐시 또는 영속 저장 경로
- 초기 기동 시 Hub 준비 완료 여부를 판정할 운영 절차

### Wrapper

- Hub URL
- 대상 JDBC URL과 드라이버 전환 계획
- 애플리케이션 배포 방식

### DB UDF

- 대상 DB 접속 정보
- Engine URL
- DBA 권한 또는 ACL 반영 권한

## 설치 전 확인 질문

1. 어떤 모듈을 이번 배포 범위에 포함하는가
2. Hub와 Engine 사이의 통신 경로가 준비되었는가
3. 운영자 진입 경로와 내부 런타임 경로를 분리했는가
4. 연결형인지 에어갭인지 운영 모드를 확정했는가
5. 설치 후 어떤 round trip과 어떤 로그 기준으로 성공을 판정할지 정했는가
