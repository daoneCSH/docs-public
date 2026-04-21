# Container Installation

DADP 공개 설치 문서는 JAR 배포뿐 아니라 컨테이너 기반 설치 경로도 지원한다. 현재 구현 기준에서 공개 문서가 참조하는 컨테이너 설치 경로는 `docker-single` 아래의 활성 compose 기준선을 바탕으로 한다.

## 지원 범위

현재 공개 설치 문서에서 다루는 컨테이너 설치 대상은 다음과 같다.

- Hub
- Engine
- Aggregator, 필요한 경우

Wrapper와 DB UDF는 별도 연동 또는 설치 문서에서 다룬다.

## 활성 컨테이너 기준선

현재 repo 기준의 활성 compose 자산은 다음과 같다.

| 위치 | 파일 | 목적 |
|------|------|------|
| `docker-single/hub-engine-single` | `docker-compose.hub-engine.local.yml` | Hub와 Engine의 표준 로컬 컨테이너 기준선 |
| `docker-single/aggregator-single` | `docker-compose.aggregator.local.yml` | Aggregator의 표준 로컬 컨테이너 기준선 |

이 경로는 공개 문서에서 현재 구현과 맞는 컨테이너 운영 기준으로 본다. archive 아래의 compose 파일은 현재 기준선으로 취급하지 않는다.

## 설치 전 준비

컨테이너 기반 설치 전에 다음 항목을 준비한다.

- Docker Engine
- Docker Compose plugin
- 사용할 이미지 또는 빌드 가능한 배포 산출물
- Hub용 Meta DB와 Redis 연결 정보
- Aggregator를 쓰는 경우 통계 저장소 연결 정보
- 컨테이너 간 통신에 사용할 네트워크와 포트 계획

## Hub와 Engine 컨테이너 설치

Hub와 Engine은 같은 compose 기준선에서 함께 운영할 수 있다. 공개 문서 기준의 핵심은 한 파일 안에 모든 개발용 편의를 설명하는 것이 아니라, 운영자가 어떤 기준선으로 설치를 시작해야 하는지 명확히 아는 것이다.

### 기준선

- compose 파일: `docker-single/hub-engine-single/docker-compose.hub-engine.local.yml`
- 대표 컨테이너: `hub`, `engine`
- 대표 포트:
  - Hub: `9004`
  - Engine: `9003`

### 운영 해석

- Hub 컨테이너는 제어면 원본을 제공한다.
- Engine 컨테이너는 실행면을 제공한다.
- 두 컨테이너는 같은 compose 안에 있어도 같은 책임을 가지지 않는다.
- 컨테이너가 기동했다고 해서 제어면과 실행면이 모두 정상이라는 뜻은 아니다.

### 설치 후 기본 확인

```bash
curl http://localhost:9004/hub/actuator/health
curl http://localhost:9003/api/health
curl http://localhost:9003/engine/api/v1/cache/sync-check
```

Hub와 Engine은 헬스 체크만으로 설치 완료로 판단하지 않는다. Hub 접근 가능, Engine 실행 가능, Hub-Engine 동기화 가능 여부를 함께 확인해야 한다.

## Aggregator 컨테이너 설치

Aggregator는 선택 구성이다. SQL Insight나 통계 수집이 필요한 경우에만 별도 compose 기준선으로 배치한다.

### 기준선

- compose 파일: `docker-single/aggregator-single/docker-compose.aggregator.local.yml`
- 대표 컨테이너: `aggregator`
- 대표 포트:
  - Aggregator: `9010`

### 설치 후 기본 확인

```bash
curl http://localhost:9010/aggregator/api/v1/health
```

Aggregator를 포함하는 경우에는 프로세스 기동 여부뿐 아니라 통계 저장소 연결과 수집 경로도 함께 검증해야 한다.

## 이미지와 산출물 기준

공개 문서에서는 컨테이너 설치를 다음 두 방식으로 이해하면 된다.

1. 준비된 이미지로 배포
2. 현재 코드와 산출물로 로컬 이미지를 빌드해 배포

현재 repo의 로컬 deploy 스크립트는 두 번째 방식을 기준으로 유지되고 있다. 운영 환경에서는 조직의 이미지 배포 정책에 따라 사전 빌드 이미지를 사용할 수 있다.

## 설치 완료 기준

- Hub 컨테이너가 정상 기동한다.
- Engine 컨테이너가 정상 기동한다.
- Hub와 Engine의 기본 헬스와 동기화 점검이 모두 통과한다.
- 선택 구성인 Aggregator를 포함한 경우 Aggregator 헬스와 통계 경로가 함께 확인된다.

## 관련 문서

- [Server Installation](server-installation.md)
- [Engine Installation](engine-installation.md)
- [Deployment Overview](../deployment/deployment-overview.md)
