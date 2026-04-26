# Configuration and Sync Workflows

DADP는 정상적인 연결형 동기화 경로를 우선으로 사용하지만, 운영 환경에 따라 보조 동기화 절차가 필요할 수 있다. 공개 문서에서는 이런 절차를 운영 도구 계층의 일부로 다룬다.

## 보조 동기화가 필요한 상황

- Wrapper가 Hub와 직접 동기화할 수 없는 환경
- 에어갭 또는 제한된 네트워크 환경
- 운영 승인 절차 때문에 설정 반영을 수동 단계로 나눠야 하는 환경
- 스키마 수집과 런타임 반영을 분리해야 하는 환경

## 대표 절차

### `export-config`

Hub CLI의 `wrapper export-config`는 Wrapper가 사용할 설정 스냅샷을 외부 파일로 내보내는 절차다. 직접 동기화가 어려운 환경에서 로컬 반영 파일을 생성할 때 사용한다.

실제 운영 절차는 단순 export에서 끝나지 않는다. 대상 `hubId` 확인, export 수행, Wrapper storage directory 확인, 파일 배치, Wrapper 재기동, 적용 검증까지 하나의 흐름으로 다뤄야 한다. 세부 절차는 [Wrapper Export and Offline Apply](wrapper-export-and-offline-apply.md)를 따른다.

### CLI 기반 설정 반영

운영자는 Hub CLI를 사용해 인스턴스별 Engine URL, 로그, 통계 수집, schema reload 같은 설정을 표준 절차로 제어할 수 있다. 이 방식은 UI 대신 명령형 운영 경로를 제공한다.

### Collector 기반 사전 등록

Schema Collector는 스키마 수집과 정책 매핑 준비를 실제 Wrapper 기동보다 앞단으로 분리한다. 이 절차는 초기 배포나 변경 승인 절차가 긴 환경에서 특히 유용하다.

!!! info "기본은 자동 동기화이고, 더 안전한 운영 옵션으로 사전 sync를 사용할 수 있다"
    Wrapper는 기본적으로 기동 후 Hub와 동기화한다. 다만 재기동 직후 첫 요청이 너무 빨리 들어오는 환경에서는, Schema Collector로 스키마를 먼저 등록하고 `wrapper export-config` 또는 동등한 절차로 storage를 선반영한 뒤 Wrapper를 기동하는 방법을 사용할 수 있다.

## 운영 해석

- 보조 동기화 절차는 정상 경로의 우회가 아니라 운영 제약을 수용하기 위한 표준 절차다.
- 수동 반영 환경에서는 원본 변경 시점과 런타임 반영 시점을 반드시 구분해 기록해야 한다.
- 설정 파일 export 성공, CLI 호출 성공, 실제 Wrapper 반영 성공은 서로 다른 상태다.

## 점검 순서

1. Hub 원본이 실제로 변경되었는지 확인한다.
2. 보조 도구가 최신 설정을 수집하거나 export했는지 확인한다.
3. 대상 Wrapper 또는 연동 컴포넌트가 그 결과를 실제로 반영했는지 확인한다.
4. 최종적으로 Engine 실행 경로에서 기대한 정책이 적용되는지 확인한다.

## Wrapper cold start 완화 절차

자동 동기화만으로 충분하지 않거나, 첫 요청 타이밍을 보수적으로 관리해야 한다면 다음 순서를 사용할 수 있다.

1. Schema Collector로 대상 스키마를 수집하고 Hub에 등록한다.
2. Hub에서 datasource와 정책 매핑을 완료한다.
3. Hub CLI `wrapper export-config` 또는 동등한 절차로 Wrapper storage를 채운다.
4. Wrapper 또는 애플리케이션을 기동한다.
5. storage 파일과 bootstrap 로그를 확인한다.
6. 보호 동작을 검증한 뒤 사용자 요청을 허용한다.
