# DADP Hub CLI

DADP Hub 6.0 CLI는 Hub 설치 패키지에 포함되는 운영 명령줄 도구다.
운영자는 이 도구로 Hub 로그인, 라이선스 확인, Wrapper JAR 다운로드, Wrapper 등록, Wrapper refresh를 수행한다.

## 기본 형식

```bash
dadp [--hub-url http://127.0.0.1:9004] <command>
```

최초 로그인 시 Hub URL을 검증하고 저장한다.

```bash
dadp --hub-url http://<hub-host>:9004 login
```

저장된 Hub URL을 사용하면 이후 명령에서는 `--hub-url`을 반복 입력하지 않아도 된다.

## 도움말

CLI는 명령별 설명과 예시를 자체 도움말로 제공한다.

```bash
dadp help
dadp login --help
dadp wrapper --help
dadp wrapper schema register --help
dadp wrapper refresh --help
```

`help`, `--help`, `-h`는 동일한 도움말 요청으로 처리한다.
도움말 출력은 Hub API를 호출하지 않는다.

## 명령 목록

| 명령 | 설명 |
| --- | --- |
| `login [username] [password]` | Hub URL을 검증하고 로그인 세션을 저장한다. |
| `logout` | 저장된 로그인 세션을 제거한다. |
| `license status` | Hub 라이선스 상태를 조회한다. |
| `license register <license-key>` | 라이선스 키를 등록한다. |
| `system status` | Hub, DB, Manager, 라이선스 상태를 조회한다. |
| `wrapper download [--lib-dir <app-lib-dir>]` | 현재 Wrapper JAR를 애플리케이션 lib 디렉터리에 다운로드한다. |
| `wrapper enroll [--alias <alias>] [--lib-dir <app-lib-dir>]` | 이미 스키마가 등록된 alias에 대해 새 Wrapper tenant만 발급한다. |
| `wrapper schema register` | 대화형으로 DB 스키마를 수집하고, alias와 함께 Hub에 등록하고, wrapper tenant runtime을 저장한다. |
| `wrapper schema collect --lib-dir <app-lib-dir> --jdbc-url <jdbc:dadp:...>` | DB 스키마 JSON만 생성한다. Hub에 접속하지 않고 tenantId를 발급하지 않는다. |
| `wrapper schema register --file <schema-register.json\|-> --alias <alias>` | 기존 스키마 JSON을 alias와 함께 Hub에 등록한다. |
| `wrapper refresh [<app-lib-dir>\|--lib-dir <app-lib-dir>]` | 기존 `proxy-config.json`의 tenantId/runtimeVersion으로 wrapper runtime 파일을 갱신한다. |
| `version` | CLI 버전을 출력한다. |

## Wrapper 표준 설치 흐름

고객사 애플리케이션의 lib 디렉터리에서 실행한다.

```bash
cd /opt/customer-app/lib

dadp --hub-url http://<hub-host>:9004 login
dadp wrapper download
dadp wrapper schema register
dadp wrapper refresh
```

`wrapper schema register` 대화형 입력:

```text
Alias:
JDBC URL:
DB user:
DB password:
Wrapper lib dir [<current directory>]:
```

JDBC URL에는 DB 접속 정보만 넣는다.
`hubUrl`, `alias`, `tenantId`, `cryptoMode`, `enabled`, `failOpen`, `debugEnabled`, `debugLevel`, `policySyncAutoEnabled`, `runtime.engineUrl`은 JDBC URL에 넣지 않는다.

예:

```text
jdbc:dadp:postgresql://192.168.0.2:5434/test_app_postgres_db?ssl=false
```

## Wrapper runtime 파일

CLI는 Wrapper JAR가 있는 lib 디렉터리 기준으로 runtime 파일을 생성한다.

```text
<app-lib-dir>/dadp/wrapper/<alias>/proxy-config.json
<app-lib-dir>/dadp/wrapper/<alias>/policy-mappings.json
```

`wrapper refresh`는 alias나 tenantId를 인자로 받지 않는다.
CLI는 `proxy-config.json`에서 tenantId와 runtime version을 읽어 Hub에 요청한다.
변경이 없으면 Hub는 `304 Not Modified`를 반환하고, 변경이 있으면 `200 OK`와 함께 최신 runtime 설정과 정책 매핑을 반환한다.
이 명령은 DB 스키마를 수집하지 않고, alias를 등록하지 않고, 새 tenantId를 발급하지 않는다.

## Wrapper enrollment output

표준 설치에서는 `--output-enrollment`를 사용하지 않는다.

기본 저장 위치:

```text
<app-lib-dir>/dadp/wrapper/<alias>/proxy-config.json
```

예외적으로 직접 지정해야 한다면 파일명까지 포함한 경로를 입력한다.

```bash
dadp wrapper schema register \
  --file schema-register.json \
  --alias customer-db \
  --lib-dir /opt/customer-app/lib \
  --output-enrollment /opt/customer-app/lib/dadp/wrapper/customer-db/proxy-config.json
```

alias 디렉터리 경로가 들어오면 CLI는 `<alias>/proxy-config.json`으로 정규화한다.
해당 위치가 파일이면 Hub 등록 요청 전에 실패한다.
