# Wrapper Configuration

Wrapper 설정의 목적은 애플리케이션 JDBC 경로와 DADP 제어면/데이터면을 정확히 연결하는 것이다.

## 주요 설정 범주

- JDBC URL과 드라이버 전환
- Hub URL
- 인스턴스 식별자
- 로컬 스냅샷과 export-config 기반 설정
- 선택적 Engine URL override

## 운영 해석

- Wrapper 설정은 정책 원본을 직접 보관하는 것이 아니라 실행에 필요한 사본을 유지하는 작업이다.
- `hubId`와 `alias`는 인스턴스 축, `datasourceId`는 스키마 축이라는 점을 운영자가 구분해야 한다.
- 내부 전송 튜닝보다 Hub 기준 설정 정합성을 우선 본다.

## Manual Apply Path

Wrapper가 Hub와 직접 동기화할 수 없는 환경에서는 export-config를 로컬 파일로 반영하는 수동 절차를 사용할 수 있다. 이 경우 핵심은 Hub에서 파일을 내려받는 것보다, Wrapper 런타임이 실제로 읽는 storage directory와 파일명 규칙에 맞게 배치하는 것이다.

세부 절차는 [Wrapper Export and Offline Apply](../operational-tools/wrapper-export-and-offline-apply.md)를 따른다.
