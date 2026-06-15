# DADP Documentation

DADP 공식 문서 사이트는 고객사 운영자와 연동 개발자가 현재 구현된 제품 표면을 기준으로 DADP를 이해하고 배포하고 운영할 수 있도록 정리한 공개 문서 집합이다.

이 사이트의 [DADP Official Manual](manuals/index.md)은 설치, 배포, 구성, 운영, 연동, API, 보안 운영, 장애 대응을 위한 공식 매뉴얼 진입점이다. 내부 작업 기록, 임시 조사 메모, 개발 중간 산출물은 포함하지 않는다.

## DADP란 무엇인가

DADP는 애플리케이션과 데이터베이스 실행 경로에 암복호화 정책을 적용하는 데이터 보호 플랫폼이다. 공개 문서 기준에서 DADP는 다음 세 계층으로 이해하면 된다.

- 제어면: Hub가 정책, 키 메타데이터, 연동 매핑, 운영 상태를 관리한다.
- 데이터면: Engine이 실제 암복호화 요청을 처리한다.
- 연동 계층: Wrapper, Direct API, DB UDF가 애플리케이션 또는 데이터베이스 경로를 DADP 런타임에 연결한다.

## 이 사이트의 문서 범위

- [Getting Started](getting-started/index.md): 제품 개요와 핵심 개념
- [Manuals](manuals/index.md): 설치·배포·운영·연동·API 기준을 묶은 공식 매뉴얼
- [Installation](installation/index.md): 모듈별 설치와 초기 검증
- [Deployment](deployment/index.md): 배포 경계, 네트워크, 운영 모드
- [Operations](operations/index.md): 운영 점검과 복구 기준
- [Configuration](configuration/index.md): 모듈별 설정 책임
- [Architecture](architecture/index.md): 현재 구현 기준 아키텍처
- [Integrations](integrations/index.md): 연동 방식별 특성
- [Modules](modules/index.md): 공개 모듈별 책임
- [API](api/index.md): 공개 API 표면과 계약 경계
- [Security](security/index.md): 인증, 인가, 신뢰 자산, 복구 경계
- [Troubleshooting](troubleshooting/index.md): 장애 범주별 진단
- [Release Notes](release-notes/index.md): 공개 릴리즈 노트 기준
- [Glossary](glossary/index.md): 공통 용어

## 먼저 읽을 문서

1. [Platform Overview](getting-started/platform-overview.md)
2. [DADP Official Manual](manuals/index.md)
3. [Core Concepts](getting-started/core-concepts.md)
4. [Platform Architecture](architecture/platform-architecture.md)
5. [Module Overview](modules/module-overview.md)
6. [Integration Overview](integrations/integration-overview.md)
7. [API Overview](api/api-overview.md)
