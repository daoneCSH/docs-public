# DB UDF Installation

DB UDF 설치는 애플리케이션 연동이 아니라 데이터베이스 SQL 실행 경로에 DADP를 연결하는 작업이다. 따라서 설치 검증도 SQL 오브젝트 생성 여부와 Engine 연결 여부를 함께 확인해야 한다.

## 설치 전 준비

- 공개 DB UDF CLI artifact 또는 사내 배포본
- 대상 DB 접속 정보
- Engine URL
- DBA 권한 또는 ACL 반영 권한
- UDF를 소유할 DB 계정

공개 artifact 다운로드 절차는 [DB UDF CLI Artifact Download](db-udf-cli-artifact-download.md)를 따른다.

## 설치 방식

공개 문서 기준에서 DB UDF 설치는 현재 외부 운영자 기준으로 검증된 흐름만 다룬다.

### Track A: 스크립트 생성 후 수동 반영

운영팀 또는 DBA가 검토 가능한 SQL 산출물을 만든 뒤 수동 반영하는 방식이다.

```bash
dadp udf generate \
  --db-type oracle \
  --db-user DADPUSER \
  --engine-url http://10.0.1.50:9003 \
  --output-dir ./dadp-udf-scripts
```

이 방식은 변경관리와 검토 절차가 엄격한 환경에 적합하다.

### Track B: CLI 직접 설치

CLI가 대상 DB에 직접 연결해 필요한 오브젝트를 설치하는 방식이다.

```bash
dadp udf install \
  --db-type oracle \
  --db-host 10.0.1.100 \
  --db-service XEPDB1 \
  --db-user DADPUSER \
  --db-password mypassword \
  --db-dba-user SYSTEM \
  --db-dba-password dbapassword \
  --engine-url http://10.0.1.50:9003
```

현재 외부 운영자 문서에서는 `dadp udf update`를 기본 절차로 전제하지 않는다. 기존 설치 갱신이 필요하면 먼저 현재 공개 CLI가 해당 명령을 실제로 제공하는지 재검증해야 한다.

## MySQL Installation Path

MySQL은 현재 외부 운영자 기준에서 one-step 플러그인 설치로 설명하지 않는다. 검증된 절차는 다음과 같다.

1. `dadp udf generate`
2. 생성된 소스에서 `dadp_crypto.so` 빌드
3. 대상 MySQL 서버의 `plugin_dir` 확인
4. `dadp_crypto.so`를 `plugin_dir`에 배치
5. `dadp udf install`

현재 공개 문서에서는 `--mysql-deploy-plugin`, `--mysql-ssh-user`, `--mysql-ssh-password`를 설치 기본 옵션으로 전제하지 않는다.

## 설치 후 검증

```bash
dadp udf verify \
  --db-type oracle \
  --db-host 10.0.1.100 \
  --db-service XEPDB1 \
  --db-user DADPUSER \
  --db-password mypassword
```

검증 단계에서는 일반적으로 다음을 확인한다.

- 설치된 버전 조회
- health check 함수 호출
- 단건 round trip
- 배치 경로가 있는 경우 배치 검증

## 설치 완료 기준

- UDF 관련 SQL 오브젝트가 정상 생성된다.
- DB에서 Engine으로의 호출이 성립한다.
- `verify`가 통과한다.
- 기본 암복호화 round trip이 성공한다.

## 운영상 주의점

- artifact 다운로드와 실제 설치 성공은 다른 단계다. tarball 확보 후 CLI 자체 점검을 먼저 수행한다.
- DB ACL 또는 외부 네트워크 정책이 누락되면 설치보다 검증 단계에서 더 자주 실패한다.
- DB UDF는 Engine URL 정합성에 직접 의존하므로, 런타임 URL 변경 절차를 따로 관리해야 한다.
- 생성형 설치와 직접 설치는 변경관리 책임이 다르므로 한 환경에서 기준 방식을 먼저 정해야 한다.
- 외부 운영자 문서에는 현재 공개 CLI에서 확인되지 않은 명령 집합이나 옵션을 넣지 않는다.
