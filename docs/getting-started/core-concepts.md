# Core Concepts

이 문서는 DADP 공개 문서 전반에서 반복해서 사용하는 핵심 운영 개념을 설명한다.

## 제어면

제어면은 정책, 키 메타데이터, 연동 매핑, 운영 상태를 관리하는 경계다. 공개 문서 기준에서 Hub가 제어면의 중심 구성 요소다.

## 데이터면

데이터면은 암복호화와 마스킹 요청을 실제로 처리하는 경계다. 공개 문서 기준에서 Engine이 데이터면의 핵심 구성 요소다.

## 런타임 캐시

Engine은 정책 원본을 직접 작성하거나 소유하지 않는다. Hub에서 동기화된 실행 준비 상태를 런타임 캐시로 유지하고, 그 상태를 바탕으로 요청을 처리한다.

## 연동 계층

연동 계층은 애플리케이션이나 데이터베이스의 실행 경로를 Engine에 연결한다. DADP의 공개 기준 주요 연동 방식은 다음 세 가지다.

- Wrapper
- Direct API
- DB UDF

## 원본과 실행 소비 관계

| 정보 | 원본 | 실행 시 소비 주체 |
|------|------|-------------------|
| 정책과 키 메타데이터 | Hub | Engine 런타임 캐시 |
| 연동 매핑 | Hub | Wrapper 또는 연동 보조 상태 |
| 운영 모드 | 배포 구성 | Hub와 연동 컴포넌트 동작 |

## 운영 규칙

- 제어면 문제와 데이터면 문제를 같은 장애 범주로 묶지 않는다.
- 런타임 실패와 정책 전파 지연을 혼동하지 않는다.
- 연동 경계 차이와 Engine 실행 동작을 분리해서 해석한다.

## 다음 문서

- [Platform Overview](platform-overview.md)
- [Platform Architecture](../architecture/platform-architecture.md)
- [Module Overview](../modules/module-overview.md)
