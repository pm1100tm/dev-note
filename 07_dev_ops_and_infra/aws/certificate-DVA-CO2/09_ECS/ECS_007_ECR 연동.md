# ECR 연동

[전체 목차](<./ECS_001_컨테이너와 ECS 기본 개념.md>)

## 학습 목표

- ECR 로그인, push, pull 흐름을 이해한다.
- 태그 불변성, 이미지 스캔, Lifecycle Policy의 목적을 안다.
- ECS image pull 문제를 IAM과 네트워크로 나누어 진단한다.

## ECR 구성

Amazon ECR Private Registry는 계정과 리전에 속하고 그 안에 Repository를 만든다. Repository URI의 형태는 다음과 같다.

```text
123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/orders-api
```

이미지는 태그와 digest로 식별할 수 있다.

- 태그: `1.4.0`, `prod`, `latest`처럼 사람이 읽기 쉬운 이름
- digest: 이미지 내용으로 계산한 불변 식별자

## Push 흐름

```bash
aws ecr get-login-password --region ap-northeast-2 \
  | docker login --username AWS --password-stdin \
    123456789012.dkr.ecr.ap-northeast-2.amazonaws.com

docker build -t orders-api:1.4.0 .
docker tag orders-api:1.4.0 \
  123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/orders-api:1.4.0
docker push \
  123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/orders-api:1.4.0
```

로그인 토큰은 영구 자격 증명이 아니다. CI에서는 실행 Role의 임시 자격 증명을 사용하고 Repository push 권한을 최소 범위로 부여한다.

## ECS pull 흐름

1. Task Definition의 `image`에 ECR URI를 지정한다.
2. ECS/Fargate Agent가 Task Execution Role로 ECR 인증 정보를 요청한다.
3. Registry에서 manifest와 layer를 가져온다.
4. 컨테이너를 생성해 실행한다.

따라서 pull 실패 원인은 크게 세 종류다.

- 권한: Execution Role 또는 Repository Policy 오류
- 네트워크: NAT, VPC Endpoint, DNS, Security Group 오류
- 이미지: 잘못된 URI/태그, 삭제된 manifest, 아키텍처 불일치

## 태그 불변성과 digest

Mutable 태그는 같은 태그가 다른 이미지로 덮어써질 수 있다. 운영에서는 tag immutability를 활성화하거나 digest로 배포하면 같은 식별자가 다른 바이너리를 가리키는 문제를 줄일 수 있다.

`latest`만 사용하면 어떤 코드가 배포됐는지 추적하기 어렵다. 버전 또는 commit SHA 태그를 기본으로 사용한다.

## 이미지 스캔

ECR 이미지 스캔은 컨테이너 이미지의 알려진 취약점을 찾는 데 사용한다. 스캔 결과는 패치와 배포 차단 정책의 입력이지, 런타임 보안을 모두 대신하는 기능은 아니다.

CI/CD에서는 심각도 기준을 정하고 새 이미지의 스캔 결과를 확인한 후 운영 배포를 승인하도록 구성할 수 있다.

## Lifecycle Policy

Lifecycle Policy는 태그 상태, prefix, 이미지 수 또는 기간 조건에 따라 오래된 이미지를 자동 정리한다. 규칙 우선순위와 조건을 평가한 뒤 만료 대상을 결정한다.

실무 권장 방식:

- 운영에서 참조하는 release 이미지는 충분히 보존한다.
- 오래된 untagged 이미지와 개발용 태그를 먼저 정리한다.
- Policy preview로 삭제 대상을 확인한다.
- 빠른 롤백에 필요한 revision의 이미지를 너무 일찍 지우지 않는다.

## 교차 계정과 암호화

다른 계정이 이미지를 pull하도록 하려면 Repository Policy와 호출 주체의 IAM 권한을 함께 구성한다. ECR 이미지는 저장 시 암호화되며 요구사항에 따라 AWS 관리형 키 또는 고객 관리형 KMS 키를 선택할 수 있다.

## 실무 적용

- CI Role에 대상 Repository의 push 권한만 부여한다.
- commit SHA와 release version으로 이미지를 추적한다.
- 취약점 스캔 결과를 배포 승인 조건에 연결한다.
- Lifecycle Policy preview를 확인한 뒤 정리 규칙을 적용한다.
- 운영 rollback 기간보다 이미지를 오래 보존한다.

## 시험 포인트

- Docker 인증: `aws ecr get-login-password` 결과를 `docker login`에 전달
- ECS pull 주체 권한: Task Execution Role
- 이미지 자동 정리: Lifecycle Policy
- 알려진 취약점 점검: Image scanning
- 덮어쓰기 방지: Tag immutability

## 연습문제

**문제.** 동일한 `prod` 태그를 실수로 덮어써 배포 결과를 재현하기 어렵다. 가장 직접적인 개선은?

**정답:** Repository의 tag immutability를 활성화하고 버전/commit SHA 태그 또는 digest를 사용한다.

## 확인 문제

- [ ] ECR push 명령의 흐름을 설명할 수 있다.
- [ ] pull 오류의 세 범주를 구분할 수 있다.
- [ ] Lifecycle Policy와 이미지 스캔의 목적을 구분할 수 있다.

[이전](<./ECS_006_IAM 권한.md>) | [다음: ECS Service와 배포](<./ECS_008_ECS Service와 배포.md>)
