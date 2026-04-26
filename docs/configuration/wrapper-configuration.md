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

## 운영 권장 사항

### 기본은 자동 동기화다

Wrapper는 기본적으로 Hub에서 정책과 endpoint 정보를 받아 로컬 storage에 반영한다. 일반적인 연결형 환경에서는 이 자동 동기화 경로를 기본으로 사용한다.

### 더 안전하게 운영하려면 사전 동기화를 사용할 수 있다

재기동 직후 첫 요청 타이밍을 보수적으로 관리해야 하는 환경에서는 다음 두 단계를 분리해서 보는 편이 안전하다.

1. Wrapper가 기동 가능한 상태가 되는 것
2. Wrapper가 실제 보호 계획을 받은 상태가 되는 것

첫 번째만 만족한 상태에서 애플리케이션 요청이 너무 빨리 들어오면, cold start 직후 첫 요청이 정책 동기화보다 먼저 도착해 평문 처리나 `Missing parameter encryption plan` 같은 오류가 나타날 수 있다.

### `alias`만 사용하는 환경에서도 사전 동기화는 유효하다

alias 기반 등록이 정상 동작하더라도, 재기동 직후 첫 요청 타이밍이 민감한 환경이라면 Schema Collector와 `export-config` 기반 동기화를 먼저 수행해 로컬 storage를 채워 둔 뒤 애플리케이션 요청을 여는 방법을 사용할 수 있다.

## Manual Apply Path

Wrapper가 Hub와 직접 동기화할 수 없는 환경에서는 export-config를 로컬 파일로 반영하는 수동 절차를 사용할 수 있다. 이 경우 핵심은 Hub에서 파일을 내려받는 것보다, Wrapper 런타임이 실제로 읽는 storage directory와 파일명 규칙에 맞게 배치하는 것이다.

세부 절차는 [Wrapper Export and Offline Apply](../operational-tools/wrapper-export-and-offline-apply.md)를 따른다.

## 권장 초기화 경로

자동 동기화만으로 충분한 환경도 있다. 다만 첫 요청 타이밍을 더 안전하게 관리하려면 다음 초기화 경로를 사용할 수 있다.

1. Schema Collector로 스키마를 사전 수집한다.
2. Hub에서 정책과 datasource 구성을 완료한다.
3. Hub CLI 또는 API로 Wrapper export를 수행한다.
4. export 결과를 Wrapper storage에 반영한다.
5. Wrapper를 기동하거나 재기동한다.
6. 동기화 완료 후에만 애플리케이션 요청을 허용한다.
