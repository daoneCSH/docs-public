# Container Installation

DADP 컨테이너 설치 문서는 공개 배포 기준으로 확인된 Docker Hub 이미지를 기준으로 작성한다. 이 문서의 범위는 Docker Hub에서 이미지를 가져와 Hub와 Engine을 기동하고, 제어면과 실행면이 함께 동작하는지 확인하는 절차다.

## Official Images

현재 GitHub 워크플로우 기준으로 공개 배포 대상 이미지는 다음과 같다.

| Component | Image |
|---|---|
| Hub | `docker.io/ichito67/dadp-hub` |
| Engine | `docker.io/ichito67/dadp-engine` |
| Aggregator | `docker.io/ichito67/dadp-aggregator` |

Hub, Engine, Aggregator 워크플로우는 모두 버전 태그와 `latest` 태그를 Docker Hub로 푸시한다. 운영 환경에서는 `latest` 대신 검증된 고정 버전 태그를 사용한다.

## Scope

이 문서는 다음 범위를 다룬다.

- Hub 컨테이너 설치
- Engine 컨테이너 설치
- Hub-Engine 연결 확인
- 선택 구성으로서 Aggregator 이미지 기준

Wrapper와 DB UDF는 컨테이너 설치 문서가 아니라 각 모듈 설치 문서와 연동 문서에서 다룬다.

## Prerequisites

컨테이너 설치 전에 다음 항목을 준비한다.

- Docker Engine
- Docker Compose plugin
- Hub용 PostgreSQL
- Hub용 Redis
- Hub와 Engine이 서로 접근 가능한 네트워크
- 배포에 사용할 고정 이미지 태그

## Tag Policy

운영 기준의 태그 선택 원칙은 다음과 같다.

1. 운영 배포는 릴리즈 버전 태그를 사용한다.
2. `latest`는 평가나 임시 검증에만 사용한다.
3. Hub와 Engine은 같은 릴리즈 라인으로 맞춘다.
4. 이미지 태그만 맞추고 설정 기준이 다르면 정상 동작을 보장하지 않는다.

## Hub and Engine Baseline

가장 먼저 안정적으로 구성해야 하는 기준선은 Hub와 Engine이다. Hub는 제어면 원본을 제공하고, Engine은 실행면을 제공한다. 두 컴포넌트가 모두 기동해야만 정책 동기화와 실제 암복호화 호출을 검증할 수 있다.

### Reference Compose

다음 예시는 Docker Hub 이미지 기준의 최소 검증 예시다.

```yaml
services:
  postgres:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: dadp_hub
      POSTGRES_USER: dadp_user
      POSTGRES_PASSWORD: change-me

  redis:
    image: redis:7-alpine

  hub:
    image: docker.io/ichito67/dadp-hub:<version>
    ports:
      - "9004:9004"
    environment:
      SERVER_PORT: 9004
      SERVER_SERVLET_CONTEXT_PATH: /hub
      SPRING_DATASOURCE_URL: jdbc:postgresql://postgres:5432/dadp_hub
      SPRING_DATASOURCE_USERNAME: dadp_user
      SPRING_DATASOURCE_PASSWORD: change-me
      SPRING_REDIS_HOST: redis
      SPRING_REDIS_PORT: 6379
      SPRING_CLOUD_VAULT_ENABLED: "false"
      DADP_ENGINE_DIRECT_URL: http://engine:9003
      DADP_ENGINE_NGINX_GATEWAY_ENABLED: "false"
    depends_on:
      - postgres
      - redis

  engine:
    image: docker.io/ichito67/dadp-engine:<version>
    ports:
      - "9003:9003"
    environment:
      SERVER_PORT: 9003
      DADP_HUB_BASE_URL: http://hub:9004
      ENGINE_ID: engine-1
      SPRING_CLOUD_VAULT_ENABLED: "false"
      CACHE_DIR: /data/cache
      CACHE_CRYPTO_MODE: PLAINTEXT
      STATS_ENABLED: "false"
    depends_on:
      - hub
```

이 예시는 Docker Hub 배포 이미지를 기준으로 Hub와 Engine을 함께 올리는 최소 검증 예시다. 운영 환경에서는 기본 비밀번호, 평문 캐시 모드, 단순 노출 포트를 그대로 사용하지 않는다.

## Pull and Start

고정 태그를 정했다면 먼저 이미지를 가져온다.

```bash
docker pull docker.io/ichito67/dadp-hub:<version>
docker pull docker.io/ichito67/dadp-engine:<version>
```

Aggregator를 함께 검증해야 하는 경우에는 다음 이미지를 추가로 가져온다.

```bash
docker pull docker.io/ichito67/dadp-aggregator:<version>
```

그 다음 Compose로 기동한다.

```bash
docker compose up -d
```

## Verification

설치 완료 판단은 단순 기동 여부가 아니라 제어면, 실행면, 동기화 상태를 함께 확인하는 방식으로 한다.

### Hub Health

```bash
curl http://localhost:9004/hub/actuator/health
```

### Engine Health

```bash
curl http://localhost:9003/api/health
```

### Policy Sync Check

```bash
curl http://localhost:9003/engine/api/v1/cache/sync-check
```

## Aggregator

Aggregator는 선택 구성이다. 공개 배포 이미지는 Docker Hub에 게시되지만, 설치 기준은 통계 저장소, 수집 경로, 운영 목적에 따라 달라진다. 따라서 공개 설치 문서에서는 Aggregator를 Hub-Engine 기준선 이후에 추가하는 컴포넌트로 다룬다.

Aggregator를 배포할 때는 다음 두 가지를 먼저 확정한다.

1. 통계 저장소 연결 방식
2. Hub 또는 Engine에서 어떤 경로로 통계 이벤트를 전달할지

## Completion Criteria

다음 조건을 모두 만족하면 컨테이너 설치가 완료된 것으로 본다.

- Hub 컨테이너가 정상 기동한다.
- Engine 컨테이너가 정상 기동한다.
- Hub health 응답이 정상이다.
- Engine health 응답이 정상이다.
- `cache/sync-check`로 Hub-Engine 동기화 상태를 확인할 수 있다.

## Related Documents

- [System Requirements](system-requirements.md)
- [Server Installation](server-installation.md)
- [Engine Installation](engine-installation.md)
- [Deployment Overview](../deployment/deployment-overview.md)
