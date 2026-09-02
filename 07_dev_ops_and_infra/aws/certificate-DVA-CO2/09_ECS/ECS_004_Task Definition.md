# Task Definition

[전체 목차](<./ECS_001_컨테이너와 ECS 기본 개념.md>)

## 학습 목표

- Task Definition의 주요 필드를 읽을 수 있다.
- Task 수준과 컨테이너 수준 설정을 구분한다.
- 환경 변수, secret, 로그, health check를 안전하게 구성한다.

## 설계도와 revision

Task Definition은 ECS가 하나 이상의 컨테이너를 실행하는 방법을 선언한 JSON 문서다. 등록된 revision은 수정하지 않고 변경할 때마다 새 revision을 만든다.

```json
{
  "family": "orders-api",
  "requiresCompatibilities": ["FARGATE"],
  "networkMode": "awsvpc",
  "cpu": "512",
  "memory": "1024",
  "executionRoleArn": "arn:aws:iam::123456789012:role/ecsTaskExecutionRole",
  "taskRoleArn": "arn:aws:iam::123456789012:role/ordersTaskRole",
  "containerDefinitions": [
    {
      "name": "app",
      "image": "123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/orders:1.4.0",
      "essential": true,
      "portMappings": [{"containerPort": 8080}],
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/orders",
          "awslogs-region": "ap-northeast-2",
          "awslogs-stream-prefix": "app"
        }
      }
    }
  ]
}
```

## 주요 필드

| 필드 | 의미 |
| --- | --- |
| `family` | revision을 묶는 이름 |
| `requiresCompatibilities` | Fargate 등 호환성 확인 |
| `networkMode` | Task의 네트워크 방식 |
| `cpu`, `memory` | Task 전체 리소스 |
| `containerDefinitions` | 컨테이너별 실행 설정 |
| `taskRoleArn` | 애플리케이션이 사용하는 AWS 권한 |
| `executionRoleArn` | ECS가 시작 준비에 사용하는 권한 |
| `volumes` | Task가 사용할 볼륨 선언 |

## CPU와 메모리

Task 수준 리소스는 Task 전체 한도 또는 예약량을 정하고, 컨테이너 수준 설정은 Task 안에서 컨테이너 간 리소스 배분에 사용한다. Fargate에서는 지원되는 Task CPU와 메모리 조합을 선택해야 한다.

메모리 부족으로 컨테이너가 종료되면 애플리케이션 튜닝과 함께 Task 크기를 검토한다. 무조건 크게 잡기보다 CloudWatch 지표로 실제 사용량을 확인한다.

## 환경 변수와 secret

- 일반 설정: `environment`
- 여러 환경 변수를 파일로 관리: `environmentFiles`
- 민감 정보 참조: `secrets`에서 Secrets Manager 또는 SSM Parameter Store 사용

Task Definition에 비밀번호 원문을 넣지 않는다. Secret 참조 값을 Task 시작 시 가져오는 권한은 일반적으로 Task Execution Role에 필요하고, 실행 중 애플리케이션이 API로 Secret을 다시 조회한다면 Task Role 권한이 필요하다.

## Port Mapping

`containerPort`는 애플리케이션이 수신하는 포트다. Fargate의 `awsvpc` 모드에서는 각 Task가 ENI를 가지며 Target Group은 일반적으로 Task IP와 컨테이너 포트로 연결된다.

## 로그와 health check

`awslogs` 드라이버를 설정하면 컨테이너의 표준 출력과 표준 오류를 CloudWatch Logs로 전송한다. 컨테이너 health check는 컨테이너 안에서 명령을 실행하며 ALB health check와 목적과 실행 위치가 다르다.

`essential` 컨테이너가 종료되면 Task 전체가 중지된다. 보조 컨테이너의 실패가 Task 종료로 이어져야 하는지 설계해야 한다.

## 실무 적용

- Infrastructure as Code로 JSON/YAML을 버전 관리한다.
- 이미지 태그뿐 아니라 digest 기반 배포로 불변성을 높일 수 있다.
- CPU, 메모리, 환경별 값은 배포 파이프라인에서 명시적으로 관리한다.
- 새 revision 등록 후 Service 배포와 안정 상태를 확인한다.
- 이전 revision을 남겨 빠르게 롤백할 수 있게 한다.

## 시험 포인트

- 설정 변경은 기존 revision 수정이 아니라 **새 revision 등록**이다.
- 이미지 pull과 `awslogs` 전송은 **Task Execution Role**이다.
- 컨테이너 코드의 S3 호출은 **Task Role**이다.
- secret 원문을 `environment`에 넣지 않고 `secrets`로 참조한다.
- Fargate는 `awsvpc`를 사용한다.

## 연습문제

**문제.** 컨테이너의 DB 암호를 코드와 Task Definition 평문에서 제거하고 배포 시 안전하게 주입하려 한다. 가장 적절한 구성은?

**정답:** Secrets Manager에 값을 저장하고 Task Definition의 `secrets`에서 참조한다. ECS가 시작 시 값을 가져올 수 있도록 Task Execution Role 권한도 확인한다.

## 확인 문제

- [ ] Task 수준과 컨테이너 수준 CPU/메모리를 구분할 수 있다.
- [ ] 두 IAM Role 필드를 설명할 수 있다.
- [ ] 컨테이너 health check와 ALB health check의 차이를 안다.

[이전](<./ECS_003_Fargate와 EC2 Launch Type.md>) | [다음: ECS Networking](<./ECS_005_ECS Networking.md>)
