# IAM 권한

[전체 목차](<./ECS_001_컨테이너와 ECS 기본 개념.md>)

## 학습 목표

- Task Role과 Task Execution Role을 정확히 구분한다.
- EC2 Container Instance Role과 Service-linked Role의 용도를 안다.
- 최소 권한과 교차 계정 접근을 설계할 수 있다.

## 가장 중요한 두 Role

| Role | 누가 사용하나 | 대표 권한 |
| --- | --- | --- |
| Task Execution Role | ECS/Fargate Agent | ECR image pull, `awslogs` 전송, 시작 시 secret 조회 |
| Task Role | 컨테이너 내부 애플리케이션 | S3, DynamoDB, SQS 등 업무 API 호출 |

```text
Task 시작 준비
ECS Agent -> Execution Role -> ECR / CloudWatch Logs / Secrets

애플리케이션 실행 중
App Container -> Task Role -> S3 / DynamoDB / SQS
```

두 Role 모두 Task Definition에 ARN을 지정하지만 목적이 다르다. 시험 문제에서는 행동의 주체를 먼저 찾는다.

## Task Execution Role

Fargate Task가 private ECR 이미지를 가져오거나 `awslogs` 드라이버로 로그를 전송하려면 Execution Role이 필요하다. Task Definition의 `secrets`로 Secrets Manager 또는 Parameter Store 값을 시작 시 주입할 때도 관련 권한을 이 Role에 추가한다.

AWS 관리형 `AmazonECSTaskExecutionRolePolicy`는 대표적인 ECR pull과 로그 권한을 제공하지만, secret이나 사용자 지정 KMS 키 등은 필요한 권한을 별도로 추가해야 한다.

## Task Role

AWS SDK는 컨테이너 자격 증명 공급자를 통해 Task Role의 임시 자격 증명을 얻을 수 있다. 코드나 이미지에 장기 Access Key를 넣지 않는다.

예를 들어 주문 API가 특정 S3 bucket의 영수증만 읽는다면 해당 prefix의 `s3:GetObject`만 허용한다. 여러 서비스가 하나의 광범위한 Task Role을 공유하지 않도록 업무 단위로 분리한다.

## EC2 Container Instance Role

EC2 Launch Type에서는 EC2 인스턴스가 Cluster에 등록되고 ECS API와 통신할 권한이 필요하다. 이것이 Container Instance Role이다. 컨테이너 애플리케이션의 S3 접근을 이 Role에 부여하는 것은 권한 분리 측면에서 잘못된 선택이다.

## PassRole과 신뢰 정책

Task Definition 또는 Service를 등록하는 배포 주체는 지정한 Role을 ECS Task에 전달할 수 있도록 `iam:PassRole` 권한이 필요할 수 있다. Role의 신뢰 정책은 ECS Task가 맡을 수 있도록 `ecs-tasks.amazonaws.com`을 신뢰해야 한다.

`iam:PassRole`이 없으면 Role 자체에 필요한 API 권한이 모두 있어도 배포 단계에서 실패할 수 있다.

## 교차 계정 접근

Task가 다른 계정 리소스에 접근할 때는 다음을 함께 고려한다.

- 호출 계정의 Task Role에 필요한 권한
- 대상 리소스의 resource-based policy
- 필요하면 대상 계정 Role을 `sts:AssumeRole`할 권한과 신뢰 정책
- KMS로 암호화된 데이터라면 KMS Key policy

## 실무 적용

1. 애플리케이션의 AWS API 호출 목록을 식별한다.
2. Task Role 정책을 리소스 ARN과 작업 단위로 제한한다.
3. 실행 준비 권한은 Execution Role에만 둔다.
4. CloudTrail과 IAM Access Analyzer로 실제 사용과 외부 접근을 검토한다.
5. 역할을 환경별 또는 서비스별로 분리한다.

## 시험 포인트

- 앱 코드가 DynamoDB 호출: **Task Role**
- ECS가 ECR image pull: **Task Execution Role**
- `awslogs` 로그 전송: **Task Execution Role**
- EC2 인스턴스의 Cluster 등록: **Container Instance Role**
- 배포 사용자가 Role을 Task에 지정하지 못함: **`iam:PassRole` 확인**

## 연습문제

**문제.** Fargate Task는 정상 시작하지만 애플리케이션이 S3를 호출할 때 `AccessDenied`가 발생한다. Execution Role에는 S3 권한이 있다. 가장 적절한 해결책은?

**정답:** S3 최소 권한을 Task Role에 부여하고 Task Definition의 `taskRoleArn`에 지정한다. 실행 중 애플리케이션 호출은 Execution Role을 사용하지 않는다.

## 확인 문제

- [ ] 행동 주체를 보고 두 Role을 선택할 수 있다.
- [ ] `iam:PassRole`이 필요한 시점을 설명할 수 있다.
- [ ] Container Instance Role에 앱 권한을 주면 안 되는 이유를 안다.

[이전](<./ECS_005_ECS Networking.md>) | [다음: ECR 연동](<./ECS_007_ECR 연동.md>)
