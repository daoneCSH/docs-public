# Wrapper Integration

Wrapper 연동은 JDBC 경계에 개입하는 방식이다. 비즈니스 로직을 전면 수정하지 않고도 SQL 실행 경로를 보호하고 싶을 때 가장 적합한 연동 모델이다.

## 연동 특성

- JDBC URL과 드라이버 전환 기반 진입
- Hub 기반 인스턴스 등록과 bootstrap
- Engine 원격 실행 또는 선택적 local crypto 실행
- 로컬 스냅샷 또는 export-config 기반 설정 유지

## 연동 흐름

1. Wrapper 인스턴스가 Hub에 등록된다.
2. 필요한 스키마와 정책 매핑 정보가 동기화된다.
3. 애플리케이션 SQL 실행 시 Wrapper가 보호 대상을 식별한다.
4. 기본 경로에서는 런타임 실행을 Engine으로 요청한다.
5. local crypto가 활성화된 경우에는 지원된 경로를 Wrapper가 직접 실행하고, 나머지는 Engine으로 fallback한다.

## 적합한 상황

- JDBC 설정을 제어할 수 있는 애플리케이션
- 중앙 정책 관리와 낮은 코드 수정량을 함께 원하는 환경
- SQL 경계에서 테이블 또는 컬럼 보호를 적용해야 하는 경우

## 운영상 주의점

- `hubId`와 `datasourceId`를 같은 개념으로 취급하지 않는다.
- 동기화 실패와 실행 실패를 분리해서 진단한다.
- Hub 직접 동기화가 어렵다면 collector 또는 CLI 기반 보조 흐름을 사용할 수 있다.
- local crypto가 활성화되어도 모든 경로가 로컬 실행으로 바뀌는 것은 아니다.
