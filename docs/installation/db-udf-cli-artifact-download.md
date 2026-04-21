# DB UDF CLI Download

이 문서는 외부 운영자가 현재 공개 경로에서 DADP CLI를 내려받아 DB UDF 설치를 수행하는 기준 절차를 설명한다.

핵심은 두 가지다.

1. 현재 외부에서 실제로 사용할 수 있는 공개 다운로드 경로를 기준으로 한다.
2. 아직 공개 artifact에서 검증되지 않은 명령이나 옵션은 운영 기준으로 가정하지 않는다.

## Current External Baseline

현재 외부 운영자 기준의 공개 다운로드 경로는 다음과 같다.

```text
https://dadp-artifacts.s3.ap-northeast-2.amazonaws.com/cli/v1.3.0/dadp-linux-amd64
```

이 문서는 위 경로에서 내려받은 `dadp` CLI를 기준으로 작성한다.

운영 문서 기준에서 중요한 점은 다음과 같다.

- 현재 외부 운영자 경로는 `db-udf` 전용 tarball 기준이 아니다.
- 현재 외부 운영자 경로는 공개적으로 접근 가능한 단일 CLI 바이너리 기준이다.
- 따라서 외부 운영자 문서는 실제 공개 경로가 검증된 명령 집합만 다뤄야 한다.

## Download

```bash
curl -fLo dadp \
  https://dadp-artifacts.s3.ap-northeast-2.amazonaws.com/cli/v1.3.0/dadp-linux-amd64
chmod +x dadp
```

또는:

```bash
wget -O dadp \
  https://dadp-artifacts.s3.ap-northeast-2.amazonaws.com/cli/v1.3.0/dadp-linux-amd64
chmod +x dadp
```

## Validate the Downloaded Binary

실제 설치 전에 먼저 다운로드한 바이너리가 현재 공개 명령 집합을 정상적으로 제공하는지 확인한다.

```bash
./dadp --help
./dadp udf --help
./dadp udf install --help
./dadp udf generate --help
```

외부 운영자 기준에서는 help 출력이 현재 공개 CLI의 실제 지원 범위를 확인하는 기준선이다.

## Current Public CLI Limits

현재 공개 경로에서 확인된 CLI 기준으로는 다음을 전제하지 않는다.

- `dadp udf update`
- `--mysql-deploy-plugin`
- `--mysql-ssh-user`
- `--mysql-ssh-password`

즉, 외부 운영자 문서에서는 아직 공개 바이너리에서 재검증되지 않은 `update` 흐름이나 MySQL 원격 플러그인 자동 배포 옵션을 설치 기본선으로 쓰지 않는다.

## Typical Use

### Generate Installation Assets

운영팀 또는 DBA가 SQL 산출물과 보조 파일을 먼저 검토해야 하는 경우:

```bash
./dadp udf generate \
  --db-type oracle \
  --db-user DADPUSER \
  --engine-url http://10.0.1.50:9003 \
  --output-dir ./dadp-udf-scripts
```

이 방식은 변경관리와 DB 검토 절차가 엄격한 환경에 적합하다.

### Direct Install

공개 CLI가 직접 설치를 지원하는 DB 유형에서는 다음과 같이 사용한다.

```bash
./dadp udf install \
  --db-type postgres \
  --db-host 10.0.1.30 \
  --db-port 5432 \
  --db-service appdb \
  --db-user dadpuser \
  --db-password mypassword \
  --engine-url http://10.0.1.50:9003
```

## Current MySQL Baseline

현재 외부 운영자 기준에서 MySQL은 one-step 플러그인 배포로 설명하지 않는다. 검증된 기준 절차는 다음 순서다.

1. 공개 CLI 바이너리 다운로드
2. `dadp udf generate`
3. 생성된 소스에서 `dadp_crypto.so` 빌드
4. `plugin_dir`에 `dadp_crypto.so` 반입
5. `dadp udf install`

### 1. Generate MySQL Assets

```bash
./dadp udf generate \
  --db-type mysql \
  --db-user soe \
  --engine-url http://10.0.1.50:9003 \
  --output-dir ./dadp-udf-mysql
```

생성 결과에는 일반적으로 다음 자산이 포함된다.

- `mysql_crypto.c`
- `build.sh`
- install SQL
- verify SQL
- uninstall SQL

### 2. Build `dadp_crypto.so`

```bash
cd ./dadp-udf-mysql
chmod +x build.sh
./build.sh
```

### 3. Copy to `plugin_dir`

대상 MySQL 서버의 `plugin_dir`는 먼저 확인한다.

```sql
SHOW VARIABLES LIKE 'plugin_dir';
```

그 다음 운영 표준 방식으로 `dadp_crypto.so`를 해당 디렉터리에 배치한다.

이 문서는 SSH 자동 복사나 원격 플러그인 자동 배포를 전제하지 않는다. 현재 공개 CLI 기준에서는 그 옵션이 외부 운영자 기본선으로 검증되지 않았기 때문이다.

### 4. Install DB Objects

```bash
./dadp udf install \
  --db-type mysql \
  --db-host 10.0.1.210 \
  --db-port 3306 \
  --db-service appdb \
  --db-user soe \
  --db-password mypassword \
  --engine-url http://10.0.1.50:9003
```

## Validation

다운로드 후 최소 점검 항목은 다음과 같다.

- `./dadp --help`가 정상 종료하는가
- `./dadp udf --help`가 정상 종료하는가
- 설치 대상 벤더에 필요한 명령이 실제로 help에 존재하는가

MySQL 설치 후에는 다음 검증을 함께 수행한다.

- `dadp_health_check()` 호출
- encrypt/decrypt round trip
- 대상 스키마의 함수 또는 프로시저 존재 여부

## What to Avoid

- 외부 운영자에게 아직 공개 검증되지 않은 `db-udf` 전용 S3 경로를 현재 기준이라고 설명하지 않는다.
- 현재 공개 CLI 기준으로 `dadp udf update`를 기본 설치 절차에 포함하지 않는다.
- `--mysql-deploy-plugin`, `--mysql-ssh-user`, `--mysql-ssh-password`를 외부 운영자 기본 옵션처럼 문서화하지 않는다.
- MySQL 설치를 공유 객체 빌드와 `plugin_dir` 반입 없이 one-step 명령으로 설명하지 않는다.

## Related Documents

- [DB UDF Installation](db-udf-installation.md)
- [DB UDF Integration](../integrations/db-udf-integration.md)
