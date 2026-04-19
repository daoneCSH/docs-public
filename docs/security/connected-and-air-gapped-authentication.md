# Connected and Air-Gapped Authentication

DADP의 인증과 신뢰 해석은 배포 위치보다 운영 모드에 더 크게 영향을 받는다. 그래서 공개 문서에서는 `Connected`와 `Air-Gapped`를 인프라 라벨이 아니라 운영 모드로 사용한다.

## Connected

Connected 모드에서는 사용자 인증, 신뢰 자산 검증, 복구 절차가 연결형 운영 경로를 전제로 해석된다.

주요 특징은 다음과 같다.

- 연결형 인증 경로 사용
- 연결형 검증 상태를 기준으로 운영 판단
- bootstrap과 steady-state 채널을 분리해서 해석

## Air-Gapped

Air-Gapped 모드에서는 Hub가 배포 경계 내부에서 인증과 운영 상태를 해석한다.

주요 특징은 다음과 같다.

- 로컬 인증 경계 사용
- 배포 내부 자산을 기준으로 운영 판단
- 복구 절차도 배포 내부 규칙을 따름

## 문서 해석 규칙

- `Connected`와 `Air-Gapped`는 인증과 신뢰 동작을 설명하는 용어다.
- `On-Premise`, `Cloud` 같은 배포 위치 용어와 혼용하지 않는다.
- 두 모드는 같은 화면과 같은 API 경로를 제공하더라도 운영 의미가 다를 수 있다.
