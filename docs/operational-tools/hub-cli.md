# Hub CLI

Hub CLI는 DADP 운영자가 Hub 제어면과 일부 실행면 관리 기능을 명령줄에서 수행할 수 있도록 제공되는 공식 운영 클라이언트다. 이 도구는 서비스 API를 대체하지 않으며, 운영 절차를 표준화하는 용도로 사용한다.

## 주요 책임

- Hub 인증과 사용자 세션 관리
- 정책, 키, 역할, 사용자, 설정 조회와 변경
- Engine 상태 점검과 캐시 동기화 제어
- Wrapper 인스턴스 설정 조회와 변경
- DB UDF 설치, 검증, 업데이트, 제거
- 라이선스, 부트스트랩, 인증서 상태 점검

## 도구 경계

Hub CLI는 다음을 수행한다.

- 공개 운영 계약에 포함된 Hub 기능 호출
- 제어면과 실행면 상태 점검
- 설치와 운영 작업의 표준 절차 제공

Hub CLI는 다음을 직접 수행하지 않는다.

- Hub나 Engine의 서비스 역할 대체
- Wrapper 런타임 자체 실행
- 데이터베이스 내부 암복호화 로직 실행

## 대표 명령 그룹

| 명령 그룹 | 목적 |
|-----------|------|
| `auth` | 로그인, 로그아웃, 현재 사용자 확인 |
| `policy`, `key`, `mask-policy`, `role`, `user` | 제어면 원본 조회와 변경 |
| `engine`, `engine-cache`, `monitor` | Engine 상태, 캐시, 모니터링 점검 |
| `proxy`, `wrapper`, `config` | Wrapper 인스턴스와 인스턴스별 설정 관리 |
| `udf` | DB UDF 설치, 검증, 갱신, 제거 |
| `license`, `bootstrap` | 라이선스, 인증서, 연결형 초기화와 복구 |
| `migration` | Hub 마이그레이션 번들 export/import/verify |

## Wrapper 운영에서의 의미

Wrapper 환경에서는 Hub CLI가 특히 중요하다. 운영자는 이 도구를 통해 다음 작업을 수행할 수 있다.

- 대상 Wrapper 인스턴스 식별
- wrapper enable 또는 disable 상태 제어
- 인스턴스별 Engine URL override 관리
- 로그 설정과 통계 수집 설정 관리
- schema reload 요청과 상태 확인
- `export-config` 기반 수동 반영 파일 생성

이 흐름은 Wrapper가 Hub를 직접 동기화하지 못하는 환경에서도 운영 대안을 제공한다.

## 운영 원칙

- Hub CLI는 운영 표준화 도구이지 서비스 경계 자체가 아니다.
- CLI 성공이 곧 서비스 상태 정상임을 의미하지는 않는다.
- CLI 출력은 현재 API 응답과 운영 상태를 요약한 것이므로, 문제를 해석할 때는 제어면과 실행면 경계를 함께 봐야 한다.
