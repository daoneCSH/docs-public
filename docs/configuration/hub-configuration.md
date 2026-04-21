# Hub Configuration

Hub 설정의 목적은 DADP 제어면을 안정적이고 예측 가능한 상태로 유지하는 데 있다.

## 주요 설정 영역

- 서버 포트와 context path
- Meta DB 연결
- Redis 연결
- 연결형 모드 상위 엔드포인트
- 로그 수준
- 연동 인스턴스 관리 설정
- 분석 및 통계 수집 설정

## Runtime Configuration Baseline

컨테이너 배포 기준에서 Hub 설정은 단순히 `SPRING_DATASOURCE_*` 계열만 채우는 방식으로 끝나지 않을 수 있다. 실제 이미지 동작 기준에서는 다음 두 계열을 함께 정합하게 유지하는 것이 안전하다.

- `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USERNAME`, `DB_PASSWORD`
- `SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`

공개 문서 기준의 운영 원칙은 "데이터소스 표현을 하나만 믿지 말고, 이미지가 참조하는 기본 키 체계를 함께 맞춘다"이다.

## 운영상 해석

- Hub는 정책과 운영 상태의 원본이므로 메타데이터 저장소와 의존성 연결이 안정적으로 유지되어야 한다.
- PostgreSQL 인증 오류가 발생하면 볼륨 상태만 보지 말고 `DB_*`와 `SPRING_DATASOURCE_*` 바인딩이 모두 기대값과 일치하는지 함께 확인해야 한다.
- 연동 인스턴스는 임시 런타임 변경보다 표준 운영 경로를 통해 관리해야 한다.
- CORS, TLS, same-origin 배포 규칙은 운영자 진입 경계에서 함께 검토해야 한다.
