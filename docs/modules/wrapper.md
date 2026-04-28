# Wrapper

Wrapper는 DADP의 공식 JDBC 연동 경로다. 애플리케이션이 기존 JDBC 경로를 유지하면서도 SQL 실행 경계에서 보호를 적용할 수 있도록 설계되었다.

## 주요 책임

- JDBC 실행 경로 개입
- Hub 기반 인스턴스 등록과 설정 동기화
- Engine 호출 또는 선택적 local crypto를 통한 런타임 실행
- 애플리케이션 코드 변경 최소화

## 식별과 매핑

Wrapper 운영에서 가장 중요한 개념은 인스턴스 식별과 스키마 식별을 분리해서 보는 것이다.

- `hubId`와 `alias`는 Wrapper 인스턴스 축이다.
- `datasourceId`는 스키마와 정책 매핑 축이다.

이 두 축을 혼동하면, 인스턴스는 살아 있는데 특정 스키마만 반영이 안 되는 문제를 정확히 해석하기 어렵다.

## 동기화 모델

Wrapper는 제어면 원본을 직접 소유하지 않는다. 실행에 필요한 메타데이터 사본을 로컬에 유지하면서 Engine을 호출한다.

직접 동기화가 어려운 환경에서는 다음 보조 경로가 운영 대안이 될 수 있다.

- collector 기반 스키마 수집 후 Hub 전송
- CLI 기반 정책 또는 설정 동기화
- export-config 기반 로컬 반영

## 실행 모드

Wrapper는 하나의 고정 실행 방식만 제공하지 않는다. 현재 구현 기준의 실행 모드는 다음과 같다.

### Remote mode

기본 모드다. Wrapper는 정책과 메타데이터를 동기화한 뒤, 실제 암복호화 실행은 Engine에 요청한다. 이 경로는 가장 넓은 기능 범위를 제공하며, Wrapper는 주로 JDBC 경계와 메타데이터 유지 역할을 맡는다.

### Local crypto mode

`cryptoMode=local` opt-in이 활성화되면, Wrapper는 지원된 경로에 한해 애플리케이션 호스트에서 직접 암복호화를 수행할 수 있다. 이 모드는 Engine과 별개의 독자 구현이 아니라 shared crypto core를 기반으로 한다.

현재 공개 기준에서 local crypto는 다음 특성을 가진다.

- 지원 provider는 제한적이다.
- 지원 algorithm은 제한적이다.
- 지원되지 않는 조합은 Engine 원격 실행으로 fallback된다.
- 일부 기능 경로는 local mode에서도 여전히 Engine 실행을 유지한다.

따라서 local crypto는 "Wrapper가 언제나 Engine을 대체한다"는 의미가 아니라, transport 비용과 실행 위치를 조정하기 위한 선택적 실행 경로다.

## 아키텍처 해석

Wrapper local crypto가 활성화되어도 제어면 원본은 여전히 Hub에 있다. Wrapper는 정책 원본을 소유하지 않고, 실행에 필요한 상태를 동기화해 사용한다. 이 구조는 중앙 제어를 유지한 채 일부 실행을 edge로 이동시키는 형태에 가깝다.

운영자는 다음 세 상태를 구분해야 한다.

- Wrapper가 정상 등록된 상태
- Wrapper가 최신 정책과 endpoint 정보를 가진 상태
- Wrapper가 실제 local crypto 또는 Engine remote path로 보호 동작을 수행하는 상태

이 세 상태를 구분하지 않으면 등록 성공과 보호 동작 성공을 같은 것으로 오해하기 쉽다.

## 적합한 사용 상황

- 애플리케이션 팀이 JDBC 설정을 제어할 수 있을 때
- SQL 경로 보호를 애플리케이션 코드 변경 없이 적용하고 싶을 때
- 중앙 정책 관리를 유지하면서 애플리케이션 수정량을 줄이고 싶을 때

## 운영 해석

Wrapper 문제는 보통 다음 범주로 나뉜다.

- 인스턴스 등록 문제
- 스키마 수집 또는 매핑 문제
- 로컬 스냅샷 갱신 문제
- Engine 실행 호출 문제
- local crypto 경로 선택 또는 fallback 문제

같은 Wrapper 오류처럼 보여도, 제어면 문제인지 로컬 메타데이터 문제인지 실행면 문제인지 분리해서 봐야 한다.
