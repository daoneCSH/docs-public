# Deployment Overview

DADP 배포 설계의 핵심은 어떤 파일을 어느 서버에 올리느냐보다, 제어면과 데이터면과 연동 경계를 어떻게 분리할 것인가에 있다. 공개 배포 문서는 현재 구현 기준에서 고객이 실제로 운영해야 하는 경계를 기준으로 설명한다.

## 배포 도메인

| 도메인 | 주요 컴포넌트 | 책임 |
|--------|----------------|------|
| 제어면 | Hub | 정책, 키 메타데이터, 운영 상태 |
| 데이터면 | Engine, Aggregator | 암복호화 실행, 선택적 통계 |
| 연동 경계 | Wrapper, Direct API, DB UDF | 애플리케이션 또는 DB 실행 경로 진입점 |

## 대표 토폴로지

### 최소 구성

- Hub 1식
- Engine 1식

정책 관리와 실행을 빠르게 시작하는 가장 단순한 배포다.

### 분리 구성

- Hub
- Engine
- 선택적 Aggregator

제어면과 실행면을 분리해 운영 경계를 나누는 일반적인 운영 구조다.

### 연동 확장 구성

- Hub
- Engine
- Wrapper 또는 DB UDF
- 선택적 Aggregator

애플리케이션 또는 데이터베이스 경로를 실제 서비스 경로에 연결하는 구성이다.

## 배포 규칙

1. Hub는 운영자 진입 경계에 배치한다.
2. Engine은 런타임 실행 경계로 분리한다.
3. Wrapper와 DB UDF는 보호하려는 실행 경로 가까이에 둔다.
4. Aggregator는 통계와 SQL Insight가 필요한 경우에만 배치한다.

## Hub 배포 모델

현재 공개 배포 모델에서 Hub는 UI와 API를 같은 origin에서 제공한다.

- `https://hub.example.com/hub/`
- `https://hub.example.com/hub/api/...`

즉 경로는 분리되지만 운영자 진입점은 하나로 유지된다.
