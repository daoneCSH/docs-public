# DADP Official Manual

DADP 공식 매뉴얼은 운영자와 연동 개발자가 제품을 설치, 배포, 구성, 운영, 연동, 장애 대응할 때 따라야 하는 기준 문서다.

이 매뉴얼은 현재 공개 제품 표면을 기준으로 작성한다. 내부 작업 기록, 개발 중간 산출물, 임시 검토 문서는 포함하지 않는다.

## Manual Scope

| 구분 | 문서 | 목적 |
| --- | --- | --- |
| 설치 매뉴얼 | [Installation](../installation/index.md) | 시스템 요구사항, 컨테이너 설치, 서버 설치, Engine, Wrapper, DB UDF 설치 절차 |
| 배포 매뉴얼 | [Deployment](../deployment/index.md) | 배포 경계, 네트워크, TLS, 연결형/폐쇄망 배포 기준 |
| 운영 매뉴얼 | [Operations](../operations/index.md) | Hub/Engine 운영 점검, 복구, 백업, 장애 대응 기준 |
| 설정 매뉴얼 | [Configuration](../configuration/index.md) | Hub, Engine, Wrapper 설정 책임과 운영 조건 |
| 연동 매뉴얼 | [Integrations](../integrations/index.md) | Wrapper, Direct API, DB UDF 연동 방식과 선택 기준 |
| 운영 도구 매뉴얼 | [Operational Tools](../operational-tools/index.md) | Hub CLI, Schema Collector, 설정 동기화, offline apply 절차 |
| API Reference | [API](../api/index.md) | Hub API와 Engine API의 공개 계약 경계 |
| 보안 운영 매뉴얼 | [Security](../security/index.md) | 인증, 인가, 신뢰 자산, 연결형/폐쇄망 인증 기준 |
| 장애 대응 매뉴얼 | [Troubleshooting](../troubleshooting/index.md) | 설치, 연결, 정책 동기화, license, trust 문제 진단 |

## Recommended Reading Order

1. [Platform Overview](../getting-started/platform-overview.md)
2. [System Requirements](../installation/system-requirements.md)
3. [Container Installation](../installation/container-installation.md)
4. [Deployment Overview](../deployment/deployment-overview.md)
5. [Operations Overview](../operations/operations-overview.md)
6. [Configuration Overview](../configuration/configuration-overview.md)
7. [Integration Overview](../integrations/integration-overview.md)
8. [API Overview](../api/api-overview.md)
9. [Troubleshooting Overview](../troubleshooting/troubleshooting-overview.md)

## Audience

- 운영자: 설치, 배포, 설정, 운영, 백업, 복구, 장애 대응을 수행한다.
- 연동 개발자: Wrapper, Direct API, DB UDF, Hub CLI, Schema Collector를 제품 경계에 맞게 연동한다.
- 보안 담당자: 인증, 권한, 신뢰 자산, 감사, 운영 책임 경계를 검토한다.

## Document Authority

이 섹션은 `docs.dadp.co.kr` 기준 공식 매뉴얼의 진입점이다. 각 세부 문서는 해당 범주의 정본 문서로 관리한다.
