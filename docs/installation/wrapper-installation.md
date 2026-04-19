# Wrapper Installation

Wrapper 설치의 목적은 애플리케이션 JDBC 경로에 DADP 보호 계층을 추가하는 것이다. 핵심은 JAR 배치보다 JDBC 설정 전환과 Hub 연동 검증에 있다.

## 설치 전 준비

- 애플리케이션 실행 환경
- Hub 접근 URL
- 대상 DB JDBC URL
- Wrapper JAR 배치 위치

## 설치 절차 개요

1. Wrapper 배포 파일 준비
2. 애플리케이션 classpath에 Wrapper 추가
3. JDBC URL과 드라이버를 Wrapper 기준으로 전환
4. Hub URL과 인스턴스 식별자 설정
5. 애플리케이션 재기동
6. Hub 연동과 대상 SQL 동작 검증

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
- 대상 SQL 경로에서 보호 동작이 확인된다.

