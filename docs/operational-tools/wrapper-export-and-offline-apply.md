# Wrapper Export and Offline Apply

이 문서는 Hub에서 Wrapper 설정을 export한 뒤, Wrapper 런타임이 그 파일을 로컬에서 읽어 초기 부트스트랩 또는 수동 반영에 사용할 수 있도록 처리하는 절차를 정리한다.

이 문서의 목적은 특정 인프라를 가정하는 것이 아니라 다음 두 단계를 명확히 하는 데 있다.

1. Hub에서 export-config를 내려받는 방법
2. Wrapper가 그 파일을 읽도록 배치하는 방법

## Scope

이 절차는 다음 상황에서 사용한다.

- Wrapper가 Hub와 즉시 연결될 수 없는 환경
- 제한망 또는 승인 절차 때문에 설정 반영을 파일 기반으로 수행해야 하는 환경
- 장애 복구 또는 초기 반입 시 온라인 동기화 이전에 로컬 부트스트랩이 필요한 환경

이 문서는 정책의 원본을 바꾸는 절차가 아니라, Hub 원본을 Wrapper 로컬 스냅샷으로 반영하는 절차를 다룬다.

## Why This Procedure Matters

이 절차는 자동 동기화를 대체하는 기본 경로가 아니라, 더 보수적인 운영을 원할 때 사용할 수 있는 안전 장치다. Wrapper는 기본적으로 Hub와 자동 동기화되지만, 기동 직후 첫 요청 타이밍이 민감한 환경에서는 이 절차를 통해 정책과 endpoint 정보를 먼저 반영할 수 있다.

운영 기준에서는 다음을 명확히 구분한다.

- 애플리케이션 프로세스가 떠 있는 상태
- Wrapper storage가 준비된 상태
- 보호 정책이 실제로 적용 가능한 상태

세 상태는 서로 같지 않다.

## What the Export Contains

Hub의 `export-config`는 Wrapper가 오프라인 또는 제한 연결 상태에서 동작하는 데 필요한 핵심 설정을 담는다.

- `hubId`
- `instanceId`
- `datasourceId`
- `policyVersion`
- `cryptoUrl`
- `hubUrl`
- 정책 매핑
- 정책 속성
- 통계 설정
- Wrapper 설정과 로그 설정

운영 관점에서 이 파일은 "Hub 원본의 다운로드본"이다. 원본 변경은 여전히 Hub에서 수행해야 하며, export 파일은 반영 수단으로만 취급한다.

## Step 1. Identify the Target `hubId`

먼저 대상 Wrapper 인스턴스의 `hubId`를 확인한다. 이 값이 export 대상과 로컬 Wrapper 식별 축을 연결한다.

CLI 예시:

```bash
dadp proxy list
```

이 단계에서 함께 확인할 항목은 다음과 같다.

- 대상 `hubId`
- 대상 `instanceId` 또는 alias
- 반영하려는 인스턴스가 실제로 그 `hubId`에 대응하는지

`hubId`와 `instanceId`를 혼동하면 다른 Wrapper 인스턴스용 설정 파일을 잘못 반영할 수 있다.

## Step 2. Export from Hub

### Option A. Hub CLI

Hub CLI 기준 export 절차는 다음과 같다.

```bash
dadp wrapper export-config --hub-id <hubId> --output-file wrapper-config.json
```

이 명령은 Hub에서 export-config를 받아 지정한 파일에 저장한다.

### Option B. Hub API

API를 직접 사용할 경우 다음 경로를 사용한다.

```bash
curl -L "http://<hub-host>:9004/hub/api/v1/proxy/export-config?hubId=<hubId>" \
  -H "Accept: application/json" \
  -o wrapper-config.json
```

운영 기준에서는 export 후 다음 값을 먼저 확인한다.

- `hubId`
- `instanceId`
- `datasourceId`
- `policyVersion`
- `cryptoUrl`

이 값이 기대한 인스턴스와 맞지 않으면 파일을 반영하지 않는다.

## Step 3. Determine the Wrapper Storage Directory

Wrapper는 임의 경로를 스캔하지 않는다. 코드 기준 저장 경로 우선순위는 다음과 같다.

1. system property `dadp.storage.dir`
2. environment variable `DADP_STORAGE_DIR`
3. `{user.dir}/dadp/wrapper/{instanceId}`
4. `{user.dir}/dadp/wrapper/shared`

따라서 반영 전에 먼저 Wrapper 런타임이 실제로 어느 경로를 storage root로 사용하는지 확인해야 한다.

점검 순서는 다음과 같다.

1. Wrapper 시작 옵션에 `-Ddadp.storage.dir=...` 가 있는지 확인
2. 프로세스 환경 변수에 `DADP_STORAGE_DIR` 가 있는지 확인
3. 둘 다 없으면 Wrapper 런타임의 `user.dir` 값을 확인
4. 최종적으로 `{user.dir}/dadp/wrapper/{instanceId}` 경로를 계산

런타임 경로를 아직 확인하지 못했다면 export 파일을 배치하기 전에 이 경로부터 확정해야 한다.

## Step 4. Place the File for Wrapper

Wrapper는 다음 파일명 규칙을 읽는다.

1. `exported-config.json`
2. `wrapper-config*.json`

운영 기준으로는 명시성과 재현성을 위해 **최종 반영 파일명을 `exported-config.json`으로 정리하는 방식**을 권장한다.

예를 들어 Wrapper storage directory가 `/path/to/storage/<instanceId>` 라면 다음과 같이 배치한다.

```bash
cp wrapper-config.json /path/to/storage/<instanceId>/exported-config.json
```

이미 Hub UI나 CLI에서 받은 파일명이 `wrapper-config-<instanceId>.json` 형태라면 그대로 두어도 로더가 인식할 수 있다. 다만 운영 표준 문서와 점검 절차는 `exported-config.json`을 기준으로 맞추는 편이 더 단순하다.

## Step 5. Restart or Re-bootstrap the Wrapper

파일을 배치했다고 해서 즉시 반영이 끝난 것은 아니다. Wrapper가 그 파일을 읽는 시점은 부트스트랩 흐름이기 때문에, 일반적으로는 Wrapper 런타임을 다시 시작해 로더가 파일을 읽게 해야 한다.

운영 기준의 순서는 다음과 같다.

1. export 파일 배치
2. Wrapper 재기동 또는 애플리케이션 재기동
3. 부트스트랩 로그에서 export 파일 로드 여부 확인

## Step 6. Verify the Apply Result

적용 성공 여부는 파일 존재만으로 판단하지 않는다. 다음 신호를 함께 확인한다.

- Wrapper가 export 파일을 발견했다는 로그
- `hubId` 적용 로그
- `policyVersion` 적용 로그
- `cryptoUrl` 또는 endpoint 정보 적용 로그
- 애플리케이션 JDBC 경로가 기대한 정책으로 동작하는지

운영상 가장 중요한 구분은 다음과 같다.

- export 성공
- 파일 배치 성공
- Wrapper 로드 성공
- 실제 정책 적용 성공

이 네 단계는 서로 다른 상태다.

## Recommended Operating Pattern

파일 기반 반영 절차는 다음 순서로 관리하는 것을 권장한다.

1. Hub에서 정책 또는 Wrapper 설정 변경
2. 대상 `hubId` 기준 export 수행
3. export 파일의 핵심 필드 검증
4. Wrapper storage directory에 파일 반영
5. Wrapper 재기동
6. 정책 적용 결과 검증

이 절차를 사용하면 "Hub를 건드렸는가"와 "Wrapper가 실제로 반영했는가"를 분리해 추적할 수 있다.

특히 Wrapper 재기동 직후 사용자 요청이 빠르게 들어오는 환경에서는, 이 절차를 통해 storage를 먼저 채운 뒤 트래픽을 여는 방식을 고려할 수 있다.

## Operational Notes

- 수동 반영 환경에서는 Hub 원본과 Wrapper 로컬 파일의 시점을 함께 기록한다.
- `hubId`가 다른 파일을 잘못 넣으면 Wrapper가 설정을 거부하거나 잘못된 인스턴스 상태를 복원할 수 있다.
- `instanceId`가 현재 런타임과 다르면 export 파일이 적용되지 않을 수 있다.
- export 파일은 장기적인 정책 원본이 아니라 운영 보조 수단이다.

## Related Documents

- [Wrapper Configuration](../configuration/wrapper-configuration.md)
- [Configuration and Sync Workflows](configuration-and-sync.md)
- [Hub CLI](hub-cli.md)
