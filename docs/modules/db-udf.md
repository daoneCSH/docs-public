# DB UDF

DB UDF는 DADP의 공식 데이터베이스 연동 경로다. 프로시저, 패키지, 함수, 트리거, 배치 SQL처럼 데이터베이스 내부 실행 흐름에서 Engine을 호출해야 할 때 사용한다.

Wrapper는 JDBC 경로용 연동 수단이고, DB UDF는 데이터베이스 내부 SQL 경로용 연동 수단이다. DB 내부 프로시저, 트리거, 함수에서 암복호화를 호출해야 하는 경우에는 DB UDF를 사용한다.

## 주요 책임

- 데이터베이스 내부에서 Engine 호출 경로 제공
- 데이터베이스 벤더별 설치 계약 정의
- 설치, 검증, 제거 스크립트 또는 CLI 흐름 제공
- 단건 primitive 암복호화 함수 제공
- request JSON 기반 primitive batch 암복호화 함수 또는 프로시저 제공

## 동작 경계

DB UDF는 애플리케이션 JDBC 경계가 아니라 데이터베이스 SQL 경계에 개입한다. 따라서 문제의 원인도 애플리케이션 설정보다 다음에 더 자주 묶인다.

- DB 오브젝트 설치 상태
- ACL 또는 외부 네트워크 허용 상태
- Engine URL 정합성
- DB 벤더별 실행 제약

## Batch Boundary

DB UDF batch는 primitive request JSON 계약이다.

DB UDF가 직접 테이블을 스캔하거나 target column을 업데이트하지 않는다. 호출 프로시저나 SQL 로직이 대상 데이터를 읽고 request JSON을 만든 뒤 DB UDF를 호출한다. DB UDF는 해당 JSON을 Engine batch endpoint로 전달하고 결과 JSON을 반환한다.

이 경계는 benchmark와 운영 문서에서도 동일하게 해석한다. 테이블 조회, 업데이트, 커밋, 정리 시간은 DB UDF primitive 처리 시간이 아니라 주변 DB 작업 비용이다.

## 적합한 사용 상황

- 보호 로직이 데이터베이스 내부 절차에 있어야 할 때
- SQL 배치나 트리거에서 직접 암복호화가 필요할 때
- 애플리케이션이 아닌 DB 계층에서 보호를 강제하고 싶을 때

## PostgreSQL Procedure Support

PostgreSQL DB UDF는 SQL function으로 설치된다.

- `dadp_encrypt(data, policy)`
- `dadp_decrypt(data)`
- `dadp_batch_encrypt(request_json[, batch_size])`
- `dadp_batch_decrypt(request_json[, batch_size])`

따라서 PL/pgSQL procedure, function, trigger function에서 그대로 호출할 수 있다. 프로시저 내부 사용 예시는 [DB UDF Tutorial](../integrations/db-udf-tutorial.md)을 따른다.

## 운영 해석

DB UDF 장애는 다음 중 하나로 귀결되는 경우가 많다.

- 설치 오브젝트 누락
- 네트워크 또는 ACL 차단
- Engine 호출 실패
- 벤더별 함수/패키지 계약 불일치

따라서 DB UDF 문제는 DB 팀과 애플리케이션 팀 사이에서 책임이 흐려지기 쉽다. 공개 문서에서는 이를 데이터베이스 실행 경계 문제로 명확히 다룬다.
