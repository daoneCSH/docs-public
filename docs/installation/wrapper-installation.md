# Wrapper Installation

Wrapper 설치의 목적은 애플리케이션 JDBC 경로에 DADP 보호 계층을 추가하는 것이다. 핵심은 JAR 배치보다 JDBC 설정 전환과 Hub 연동 검증에 있다.

!!! info "기본은 자동 동기화이며, 필요 시 사전 동기화를 사용할 수 있다"
    Wrapper는 기본적으로 Hub에서 필요한 정책과 endpoint 정보를 자동으로 받아온다. 다만 재기동 직후 첫 사용자 요청이 정책 동기화보다 먼저 도착할 수 있는 환경에서는, 더 안전한 운영 방법으로 Schema Collector와 `export-config` 기반 선반영 절차를 사용할 수 있다.

## 설치 전 준비

- 애플리케이션 실행 환경
- Hub 접근 URL
- 대상 DB JDBC URL
- Wrapper JAR 배치 위치
- 대상 `instanceId` 또는 alias
- 사전 수집에 사용할 Schema Collector 또는 Hub CLI 접근 수단

## 설치 절차 개요

1. Wrapper 배포 파일 준비
2. 애플리케이션 classpath에 Wrapper 추가
3. JDBC URL과 드라이버를 Wrapper 기준으로 전환
4. Hub URL과 인스턴스 식별자 설정
5. 필요 시 Schema Collector 또는 `export-config` 기반 사전 동기화 수행
6. 애플리케이션 재기동
7. Hub 연동과 대상 SQL 동작 검증

## 설정 전환 예시

기존 설정:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

Wrapper 적용 후:

```properties
spring.datasource.url=jdbc:dadp:mysql://localhost:3306/mydb
spring.datasource.driver-class-name=com.dadp.jdbc.DadpJdbcDriver
dadp.proxy.hub-url=http://localhost:9004
dadp.proxy.instance-id=proxy-1
```

## 설치 완료 기준

- Wrapper JAR이 정상 로드된다.
- 애플리케이션 DB 연결이 정상이다.
- Hub 연동이 성립한다.
- `schemas.json`, `policy-mappings.json`, `crypto-endpoints.json`, `proxy-config.json`, `datasources.json` 등 초기 storage가 준비된다.
- 대상 SQL 경로에서 보호 동작이 확인된다.

## 안전한 구동 순서

Wrapper는 자동 동기화를 기본으로 사용한다. 다만 재기동 직후 첫 요청 타이밍을 더 보수적으로 관리해야 하는 환경에서는 다음 순서를 권장한다.

1. Schema Collector로 스키마를 사전 수집하고 Hub에 등록한다.
2. Hub에서 대상 Wrapper 인스턴스의 정책과 endpoint 구성을 완료한다.
3. Hub CLI `wrapper export-config` 또는 동등한 동기화 절차로 Wrapper storage를 채운다.
4. Wrapper 또는 애플리케이션을 기동한다.
5. 동기화 완료 로그와 보호 동작을 확인한 뒤 사용자 트래픽을 연다.

이 순서를 사용하면 cold start 직후 첫 요청이 정책 동기화보다 먼저 도착해 평문 처리나 보호 계획 누락이 발생할 가능성을 줄일 수 있다.

## 관련 문서

- [Schema Collector](../operational-tools/schema-collector.md)
- [Configuration and Sync Workflows](../operational-tools/configuration-and-sync.md)
- [Wrapper Export and Offline Apply](../operational-tools/wrapper-export-and-offline-apply.md)
