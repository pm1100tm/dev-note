# ECS IAM Roles

> 확인일: 2026-09-29

ECS에서는 같은 컨테이너를 실행하더라도 누가 AWS API를 호출하는지에 따라 다른 IAM Role을 사용합니다.
이 역할을 섞어 주면 권한이 과도해져 보안 문제가 생기거나, 이미지 가져오기와 애플리케이션 권한 문제를
잘못 진단하게 됩니다.

## 역할을 먼저 구분합니다

| Role                 | 사용하는 주체                     | 대표적인 책임                               |
| -------------------- | --------------------------------- | ------------------------------------------- |
| EC2 Instance Profile | EC2 Launch Type의 ECS Agent와 EC2 | Cluster 등록, ECS API 호출                  |
| Task Execution Role  | ECS/Fargate 실행 계층             | ECR 이미지 가져오기, 로그 전송, 시크릿 주입 |
| Task Role            | 컨테이너 안의 애플리케이션 코드   | S3, DynamoDB, SQS 등 업무 API 호출          |

```shell
EC2 Launch Type
  EC2 Instance Profile → ECS Agent가 Cluster에 등록하고 ECS와 통신

태스크 시작 전
  Task Execution Role → 이미지 가져오기 · 로그 설정 · 시크릿 주입

태스크 실행 중
  Task Role → 애플리케이션이 S3 · DynamoDB · SQS 호출
```

Fargate에는 사용자가 관리하는 EC2가 없으므로 EC2 Instance Profile도
없습니다. 하지만 Fargate에서도 Task Execution Role과 Task Role은
필요에 따라 사용합니다.

## EC2 Instance Profile

EC2 Instance Profile은

- EC2 Launch Type에서 ECS Container Instance에 연결하는 IAM Role입니다.
- ECS Agent는 이 권한으로 EC2 인스턴스를 ECS Cluster에 등록하고 ECS 서비스 API와 통신합니다.

이 역할은 **EC2 인스턴스와 ECS Agent의 역할**입니다.

컨테이너의 업무 권한을 여기에 넣으면, 같은 EC2에서 실행되는 다른 태스크까지 불필요하게 넓은 권한의 영향을
받을 수 있습니다.

## Task Execution Role

- Task Execution Role은 ECS Agent 또는 Fargate 실행 계층이 태스크를 시작할 때 AWS API를
  호출하도록 허용하는 Role입니다.
- Task Definition의 `executionRoleArn`에 지정합니다.

대표적인 사용 사례는 다음과 같습니다.

- 프라이빗 Amazon ECR에서 컨테이너 이미지를 가져옵니다.
- `awslogs` 로그 드라이버로 CloudWatch Logs에 컨테이너 로그를 전송합니다.
- Task Definition의 `secrets`에 지정한 Secrets Manager 또는 SSM
  Parameter Store 값을 컨테이너 환경 변수로 주입합니다.

> AWS 관리형 정책 `AmazonECSTaskExecutionRolePolicy`에는 ECR 이미지
> 가져오기와 `awslogs` 전송에 필요한 기본 권한이 포함됩니다. Secrets
> Manager, SSM Parameter Store, KMS 키를 사용하는 경우에는 대상
> 리소스에 대한 최소 권한을 추가해야 합니다.

EC2 Launch Type에서 `awslogs`와 시크릿 주입에 Execution Role을
사용하려면 ECS Agent 설정이 필요할 수 있습니다.

Fargate와 똑같이 동작한다고 가정하지 말고, 사용 중인 Agent 설정과 공식 문서를 확인해야 합니다.

## Task Role

- Task Role은 실행 중인 애플리케이션 컨테이너의 AWS 권한입니다.
- Task Definition의 `taskRoleArn`에 지정합니다.
- 컨테이너의 AWS SDK와 CLI는 이 Role의 임시 자격 증명을 사용합니다.

예를 들어 주문 처리 서비스는 주문 테이블에만 `PutItem` 권한을, 파일 처리 서비스는 특정 S3 버킷에만
`GetObject` 권한을 갖도록 서로 다른 Task Role을 사용합니다.

```shell
주문 API 컨테이너가 DynamoDB에 주문 상태 기록
  → Task Role에 대상 테이블의 DynamoDB 권한 부여

ECS가 ECR에서 컨테이너 이미지 가져오기
  → Task Execution Role에 ECR 권한 부여
```

> Task Role에 ECR 이미지 pull이나 로그 전송 권한을 주어도 태스크 시작 단계의 문제는 해결되지 않습니다.
> 반대로 Task Execution Role에 DynamoDB·S3 업무 권한을 주어도 애플리케이션이 그 권한을 사용하지 않습니다.

## 서비스별로 Task Role을 분리하는 이유

여러 ECS Service가 같은 EC2에서 실행되더라도, 서비스마다 다른
Task Role을 연결할 수 있습니다. 이를 통해 각 서비스에 필요한 권한만 줄 수 있습니다.

| 서비스           | 필요한 권한                   | 권장 Task Role           |
| ---------------- | ----------------------------- | ------------------------ |
| 주문 API         | 주문 테이블 읽기·쓰기         | `order-api-task-role`    |
| 이미지 변환 워커 | 특정 S3 버킷 읽기·쓰기        | `image-worker-task-role` |
| 이메일 발송 워커 | 특정 SQS Queue 읽기, SES 발송 | `mail-worker-task-role`  |

이 방식은 최소 권한 원칙을 지키고, CloudTrail에서 어떤 태스크가 AWS API를 호출했는지 추적하는 데도
도움이 됩니다.

EC2 Instance Profile을 공유 업무 권한 저장소처럼 사용하지 않는 것이 중요합니다.

## 문제를 만났을 때 확인할 Role

| 증상                                                           | 먼저 확인할 Role     |
| -------------------------------------------------------------- | -------------------- |
| `CannotPullContainerError`로 ECR 이미지를 가져오지 못합니다.   | Task Execution Role  |
| CloudWatch Logs에 로그가 기록되지 않습니다.                    | Task Execution Role  |
| 시크릿을 환경 변수로 주입하지 못합니다.                        | Task Execution Role  |
| 애플리케이션의 S3·DynamoDB 호출이 `AccessDenied`로 실패합니다. | Task Role            |
| EC2 인스턴스가 ECS Cluster에 등록되지 않습니다.                | EC2 Instance Profile |

## DVA-C02 핵심 정리

- ECR 이미지 가져오기와 CloudWatch Logs 전송은 Task Execution Role을 먼저 확인합니다.
- 애플리케이션 코드가 S3, DynamoDB, SQS를 호출한다면 Task Role에 최소 권한을 부여합니다.
- EC2 Launch Type의 Instance Profile은 ECS Agent와 EC2용이며, Fargate에는 사용하지 않습니다.
- 서로 다른 ECS Service에는 필요한 권한이 다른 만큼 서로 다른 Task Role을 사용합니다.
