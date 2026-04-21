# DB UDF CLI Artifact Download

이 문서는 DADP DB UDF 설치에 사용하는 공개 CLI artifact를 어디서 내려받고, 어떤 기준으로 버전을 선택하며, 다운로드 후 어떤 명령으로 설치 또는 갱신을 진행하는지 정리한다.

이 문서의 대상은 DB UDF 배포를 수행하는 운영자와 DBA 협업 경로다. 핵심은 "어떤 파일을 내려받아 어떤 명령으로 사용할지"를 재현 가능하게 만드는 것이다.

## Public Artifact Path

공개 DB UDF CLI artifact의 기준 경로는 다음과 같다.

- Public base URL: `https://dadp-artifacts.s3.ap-northeast-2.amazonaws.com/`
- Metadata URL: `https://dadp-artifacts.s3.ap-northeast-2.amazonaws.com/metadata.json`
- Metadata key: `artifacts.dbUdf`
- Versioned artifact path:
  `https://dadp-artifacts.s3.ap-northeast-2.amazonaws.com/db-udf/v<version>/dadp-db-udf-cli-<version>-linux-amd64.tar.gz`

현재 공개 artifact는 Linux amd64 기준으로 제공한다.

## Choosing a Version

운영 기준의 원칙은 다음과 같다.

1. 디렉터리 추측이 아니라 `metadata.json`으로 배포 버전을 확인한다.
2. 운영 환경에는 검증된 고정 버전을 사용한다.
3. 기존 DB UDF runtime을 갱신할 때는 `install`이 아니라 `update` 흐름을 우선 검토한다.

### Check `latestVersion`

```bash
curl -fsSL https://dadp-artifacts.s3.ap-northeast-2.amazonaws.com/metadata.json
```

확인해야 하는 값은 다음 구조다.

```json
{
  "artifacts": {
    "dbUdf": {
      "latestVersion": "2.5.1"
    }
  }
}
```

`latestVersion`은 최신 공개 버전을 찾는 기준이다. 다만 실제 배포 버전은 운영 검증과 변경관리 기준으로 따로 확정할 수 있다.

## Download by Exact Version

예를 들어 `2.5.1` 버전을 직접 내려받는 경우:

```bash
curl -fLo dadp-db-udf-cli-2.5.1-linux-amd64.tar.gz \
  https://dadp-artifacts.s3.ap-northeast-2.amazonaws.com/db-udf/v2.5.1/dadp-db-udf-cli-2.5.1-linux-amd64.tar.gz
```

또는:

```bash
wget -O dadp-db-udf-cli-2.5.1-linux-amd64.tar.gz \
  https://dadp-artifacts.s3.ap-northeast-2.amazonaws.com/db-udf/v2.5.1/dadp-db-udf-cli-2.5.1-linux-amd64.tar.gz
```

## Download from Metadata

`metadata.json`에서 `latestVersion`을 읽은 뒤 다운로드하는 기본 절차는 다음과 같다.

```bash
VERSION=2.5.1
curl -fLo "dadp-db-udf-cli-${VERSION}-linux-amd64.tar.gz" \
  "https://dadp-artifacts.s3.ap-northeast-2.amazonaws.com/db-udf/v${VERSION}/dadp-db-udf-cli-${VERSION}-linux-amd64.tar.gz"
```

운영 자동화 스크립트를 작성할 때도 이 패턴을 기준으로 삼는 것이 안전하다.

## Extract

다운로드 후 tarball을 해제한다.

```bash
tar -xzf dadp-db-udf-cli-2.5.1-linux-amd64.tar.gz
chmod +x dadp
```

해제 후 기본 점검:

```bash
./dadp --help
./dadp udf --help
```

이 단계가 통과해야 실제 설치나 갱신 절차로 넘어간다.

## Typical Use

### Generate Installation Scripts

운영팀 또는 DBA가 SQL 산출물을 먼저 검토해야 하는 경우:

```bash
./dadp udf generate \
  --db-type oracle \
  --db-user DADPUSER \
  --engine-url http://10.0.1.50:9003 \
  --output-dir ./dadp-udf-scripts
```

이 방식은 변경관리와 DB 검토 절차가 엄격한 환경에 적합하다.

### Direct Install

CLI가 대상 DB에 직접 연결해 필요한 UDF 오브젝트를 설치하는 경우:

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

### Update Existing Installation

이미 설치된 DB UDF runtime을 현재 설정을 유지한 채 갱신하는 경우:

```bash
./dadp udf update \
  --db-type postgres \
  --db-host 10.0.1.30 \
  --db-port 5432 \
  --db-service appdb \
  --db-user dadpuser \
  --db-password mypassword
```

운영자가 명시적으로 바꾸지 않으면 현재 설치된 runtime 설정을 유지하는 흐름을 우선 사용한다.

## Validation After Download

artifact를 내려받은 뒤에는 최소한 다음 항목을 확인한다.

- tarball 안에 `dadp` 실행 파일이 포함되어 있는가
- `./dadp --help`가 정상 종료하는가
- `./dadp udf --help`가 정상 종료하는가
- 다운로드한 버전이 `metadata.json` 기준과 일치하는가

## What to Avoid

- 디렉터리 목록만 보고 최신 버전을 추정하지 않는다.
- `metadata.json` 확인 없이 로컬에 남은 오래된 tarball을 재사용하지 않는다.
- 비문서화된 플랫폼 artifact가 공개되어 있다고 가정하지 않는다.

## Related Documents

- [DB UDF Installation](db-udf-installation.md)
- [DB UDF Integration](../integrations/db-udf-integration.md)
