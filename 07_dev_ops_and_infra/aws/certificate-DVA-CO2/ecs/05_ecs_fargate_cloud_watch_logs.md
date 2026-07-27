# ECS Fargate와 CloudWatch Logs

> **Fargate에서도 CloudWatch와 연동하여 로그를 확인할 수 있다.**

ECS Fargate에서 애플리케이션 로그를 확인하는 가장 일반적인 방법이 **CloudWatch Logs**이다.

Task Definition에서 `awslogs` 로그 드라이버를 설정하면 컨테이너의 `stdout`과 `stderr`가 CloudWatch Logs로 전송된다.

---

## 전체 구조

```text
Fargate Task
    │
    │ 애플리케이션 로그
    │ stdout / stderr
    ▼
awslogs Log Driver
    ▼
CloudWatch Logs
    │
    ├── Log Group
    └── Log Stream
```

예를 들어 Spring Boot 애플리케이션에서 다음과 같이 로그를 출력한다고 가정하자.

```java
log.info("주문 요청이 들어왔습니다.");
log.error("결제 처리 중 오류가 발생했습니다.");
```

이 로그들은 컨테이너의 `stdout` 또는 `stderr`로 출력되고,

`awslogs` 드라이버가 이를 CloudWatch Logs로 자동 전송한다.

---

## Task Definition 설정 예시

```json
{
  "containerDefinitions": [
    {
      "name": "spring-app",
      "image": "123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/spring-app:latest",
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/spring-app",
          "awslogs-region": "ap-northeast-2",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

## 설정 항목

| 설정                    | 설명                                           |
| ----------------------- | ---------------------------------------------- |
| `logDriver`             | CloudWatch Logs를 사용하기 위해 `awslogs` 지정 |
| `awslogs-group`         | 로그가 저장될 Log Group                        |
| `awslogs-region`        | CloudWatch Logs가 위치한 AWS Region            |
| `awslogs-stream-prefix` | Task별 Log Stream Prefix                       |

실제 생성되는 Log Stream은 다음과 같다.

```
ecs/spring-app/Task-ID
```

---

## 필요한 IAM 권한

Fargate가 CloudWatch Logs에 로그를 기록하려면 **Task Execution Role**에 아래 권한이 필요하다.

```json
{
  "Effect": "Allow",
  "Action": ["logs:CreateLogStream", "logs:PutLogEvents"],
  "Resource": "*"
}
```

실무에서는 대부분 아래 AWS 관리형 정책을 사용한다.

```
AmazonECSTaskExecutionRolePolicy
```

---

## Task Role과 Task Execution Role 차이

많은 사람들이 헷갈리는 부분이다.

| 역할                    | 용도                                                 |
| ----------------------- | ---------------------------------------------------- |
| **Task Execution Role** | ECR에서 이미지 Pull, CloudWatch Logs 전송            |
| **Task Role**           | 실행 중인 애플리케이션이 S3, DynamoDB, SQS 등에 접근 |

즉, CloudWatch Logs는 **Task Execution Role**이 담당한다.

---

## Fargate에서 볼 수 없는 것

여기서 중요한 점은, CloudWatch Logs를 사용할 수 없는 것이 아니라,
**Host(Server) 자체를 볼 수 없다는 것**이다.

AWS가 Host를 관리하기 때문에 다음과 같은 정보는 접근할 수 없다.

- Host OS 로그
- Docker Daemon 로그
- ECS Container Agent 로그
- `/var/log`
- Kernel 로그
- Host CPU/Memory/Disk 직접 확인
- Host에 CloudWatch Agent 설치

---

### Fargate에서 가능한 것

| 항목               | 가능 여부 |
| ------------------ | --------- |
| Spring Boot 로그   | ✅        |
| stdout / stderr    | ✅        |
| CloudWatch Logs    | ✅        |
| CloudWatch Metrics | ✅        |
| Container Insights | ✅        |
| ECS Service 상태   | ✅        |
| ECS Task 상태      | ✅        |

---

### Fargate에서 불가능한 것

| 항목               | 가능 여부 |
| ------------------ | --------- |
| SSH 접속           | ❌        |
| Host OS 로그 확인  | ❌        |
| Docker Daemon 로그 | ❌        |
| Kernel 로그        | ❌        |
| Host에 Agent 설치  | ❌        |

---

### 이전 설명에서 "Agent 설치가 어렵다"의 의미

이전에는 다음과 같이 설명했다.

> 시스템 모니터링 Agent 설치가 어렵다.

이 말은

> **CloudWatch를 사용할 수 없다는 의미가 아니다.**

정확한 의미는

> **Host OS에 Agent를 설치할 수 없다는 의미이다.**

예를 들어 EC2에서는

```
EC2
├── Spring Boot
├── CloudWatch Agent
├── Datadog Agent
└── Node Exporter
```

처럼 Host에 여러 Agent를 설치할 수 있다.

반면 Fargate는 Host를 AWS가 관리하기 때문에

Host에 직접 Agent를 설치하는 것이 불가능하다.

---

## Fargate에서 사용할 수 있는 모니터링

실무에서는 다음 서비스를 함께 사용한다.

- CloudWatch Logs
- CloudWatch Metrics
- CloudWatch Container Insights
- FireLens (Fluent Bit)
- AWS X-Ray
- OpenTelemetry

---

## EC2와 Fargate 비교

| 항목               | EC2 Launch Type | Fargate Launch Type |
| ------------------ | --------------- | ------------------- |
| CloudWatch Logs    | ✅              | ✅                  |
| Application Log    | ✅              | ✅                  |
| stdout / stderr    | ✅              | ✅                  |
| Container Metrics  | ✅              | ✅                  |
| Host OS 로그       | ✅              | ❌                  |
| Docker Daemon 로그 | ✅              | ❌                  |
| SSH 접속           | ✅              | ❌                  |
| Host Agent 설치    | ✅              | ❌                  |

---

## 핵심 정리

### 기억해야 할 한 문장

> **Fargate도 CloudWatch Logs를 사용할 수 있다.**

다만

> **Application(Container) 수준까지만 관찰 가능하고, Host(Server) 수준은 AWS가 관리하므로 접근할 수 없다.**

즉,

- **Application Monitoring** → 가능
- **Container Monitoring** → 가능
- **Host Monitoring** → 불가능

이 차이를 이해하면 Fargate의 모니터링 구조를 쉽게 이해할 수 있다.
