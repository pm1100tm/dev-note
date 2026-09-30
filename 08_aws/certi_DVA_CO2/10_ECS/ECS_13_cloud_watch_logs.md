# ECS Fargate와 CloudWatch Logs

> **Fargate에서도 CloudWatch Logs로 컨테이너 로그를 수집하고
> 조회할 수 있습니다.**

- ECS Fargate에서 애플리케이션 로그를 확인하는 가장 일반적인 방법은 CloudWatch Logs입니다.
- Task Definition에서 `awslogs` 로그 드라이버를 설정하면 컨테이너가 표준 출력(`stdout`)과
  표준 오류(`stderr`)로 쓴 내용을 CloudWatch Logs로 보냅니다.

이 문서는 `awslogs`를 사용하는 Fargate Task를 기준으로 설명합니다.

설정 항목과 지원 로그 드라이버는 변경될 수 있으므로,
2026-09-30에 확인한
[AWS ECS 공식 문서](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/fargate-tasks-services.html)
를 함께 참고합니다.

## 전체 구조

```text
Fargate Task
    │
    │ 애플리케이션 로그: stdout / stderr
    ▼
awslogs 로그 드라이버
    ▼
CloudWatch Logs
    │
    ├── Log Group: 애플리케이션 또는 환경 단위
    └── Log Stream: 컨테이너와 Task 단위
```

예를 들어 Spring Boot 애플리케이션이 다음 로그를 출력한다고 가정합니다.

```java
log.info("주문 요청이 들어왔습니다.");
log.error("결제 처리 중 오류가 발생했습니다.");
```

- 콘솔에 출력된 로그를 `awslogs` 드라이버가 수집하여 지정한 Log
  Group의 Log Stream에 기록합니다.
- 애플리케이션이 파일에만 로그를 기록하면 이 설정만으로는 파일
  내용을 수집할 수 없습니다.

### 애플리케이션이 파일에만 로그를 기록하낟?

Spring Boot는 기본적으로 로그를 콘솔에 출력합니다. 따라서 별도
설정이 없어도 `log.info()`와 `log.error()` 등의 로그를 `awslogs`가
수집할 수 있습니다.

로그 설정은 `application.yml`에서 로그 수준과 콘솔 출력 형식을 조정할 수 있습니다.

```yaml
logging:
  level:
    root: INFO
    com.example: DEBUG
  pattern:
    console: '%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger{36} - %msg%n'
```

- 반대로 다음처럼 파일 로그만 설정하고 콘솔 출력을 끄면, 로그는 컨테이너 내부의
  `/app/logs/application.log` 파일에만 기록됩니다.
- `awslogs`는 이 파일을 읽지 않으므로 CloudWatch Logs에는 로그가 표시되지 않습니다.

```yaml
logging:
  file:
    name: /app/logs/application.log
```

- Fargate에서 `awslogs`를 사용한다면 애플리케이션 로그를 콘솔에도 출력하도록 유지합니다.
- 파일 로그를 반드시 수집해야 한다면 파일을 읽어 전송하는 별도 수집 구성이나 FireLens 사용을 검토합니다.

## Task Definition 설정

다음은 `containerDefinitions` 일부입니다. `awslogs-group`은 미리
생성한 Log Group 이름이며, `awslogs-region`에는 그 Log Group이 있는
리전을 지정합니다.

```json
{
  "name": "spring-app",
  "image": "123456789012.dkr.ecr.ap-northeast-2.amazonaws.com/spring-app:latest",
  "logConfiguration": {
    "logDriver": "awslogs",
    "options": {
      "awslogs-group": "/ecs/spring-app/production",
      "awslogs-region": "ap-northeast-2",
      "awslogs-stream-prefix": "ecs",
      "mode": "non-blocking",
      "max-buffer-size": "25m"
    }
  }
}
```

| 설정                    | 설명                                                                               |
| ----------------------- | ---------------------------------------------------------------------------------- |
| `logDriver`             | CloudWatch Logs를 사용하려면 `awslogs`를 지정합니다.                               |
| `awslogs-group`         | Log Stream을 저장할 Log Group 이름입니다.                                          |
| `awslogs-region`        | 지정한 Log Group이 존재하는 AWS 리전입니다.                                        |
| `awslogs-stream-prefix` | 서비스와 컨테이너를 식별할 Log Stream 접두사입니다. Fargate에서는 지정해야 합니다. |
| `mode`                  | 로그 전송이 지연될 때 애플리케이션의 동작 방식을 정합니다.                         |
| `max-buffer-size`       | `non-blocking` 모드에서 로그를 임시 보관할 메모리 버퍼 크기입니다.                 |

`awslogs-stream-prefix`를 `ecs`로 지정하고 컨테이너 이름이
`spring-app`이면 Log Stream 이름은 다음 형식이 됩니다.

```text
ecs/spring-app/<ecs-task-id>
```

각 Task와 컨테이너에 별도 Stream이 생기므로, 같은 서비스에서 여러
Task가 실행되어도 로그 출처를 구분할 수 있습니다.

ECS 콘솔의 Logs 탭에서 로그를 보려면 Stream Prefix를 지정해야 합니다.

형식과 필수 여부는 [Task Definition 파라미터 문서](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_definition_parameters.html)
를 기준으로 확인합니다.

### 로그 전송 모드 선택

`blocking` 모드는 로그를 보낼 수 없을 때 애플리케이션의 로그 출력 작업이 대기할 수 있습니다.
로그 유실 가능성은 줄어들지만, CloudWatch Logs의 지연이나 권한 오류가 애플리케이션의 응답 지연으로
이어질 수 있습니다.

`non-blocking` 모드는 먼저 메모리 버퍼에 로그를 쌓고 전송합니다.
버퍼가 가득 차면 로그 일부가 유실될 수 있지만, 로그 전송 문제로 애플리케이션이 멈추는 위험을 낮춥니다.

따라서 운영 서비스에서는 예상 로그량을 기준으로 `max-buffer-size`를 정하고, 유실보다 완전한
감사 기록이 중요한 워크로드는 별도로 `blocking` 모드의 영향을 검토합니다.

## Log Group 준비와 보존 기간

Log Group은 인프라 코드 또는 CloudWatch 콘솔에서 먼저 만드는 방식을 권장합니다.
이때 환경을 이름에 포함하고, 보존 기간과 태그를 함께 설정하면 비용과 권한 범위를 관리하기 쉽습니다.

```shell
aws logs create-log-group \
  --log-group-name /ecs/spring-app/production \
  --region ap-northeast-2

aws logs put-retention-policy \
  --log-group-name /ecs/spring-app/production \
  --retention-in-days 30 \
  --region ap-northeast-2
```

- Log Group은 기본적으로 로그 이벤트를 만료하지 않습니다.
- 따라서 보존 기간을 명시하지 않으면 로그가 계속 쌓여 비용이 늘 수 있습니다.
- 보존 기간은 장애 분석과 감사 요구 사항을 고려해 정합니다.
- 기본 동작은 [CloudWatch Logs CreateLogGroup 문서](https://docs.aws.amazon.com/AmazonCloudWatchLogs/latest/APIReference/API_CreateLogGroup.html)
  에서 확인할 수 있습니다.

`awslogs-create-group: "true"` 옵션으로 Log Group을 자동 생성할 수도 있습니다.

그러나 Task Execution Role에 `logs:CreateLogGroup` 권한을 추가해야 하며, 생성 시 보존 기간과
태그를 설정하지 못합니다.

운영 환경에서는 미리 생성하는 편이 관리에 유리합니다.

## 필요한 IAM 권한

Fargate 실행 계층이 CloudWatch Logs에 Stream을 만들고 이벤트를
기록하려면 Task Execution Role에 권한이 있어야 합니다.
애플리케이션 코드가 사용하는 Task Role과는 목적이 다릅니다.

```json
{
  "Effect": "Allow",
  "Action": ["logs:CreateLogStream", "logs:PutLogEvents"],
  "Resource": "arn:aws:logs:ap-northeast-2:123456789012:log-group:/ecs/spring-app/production:log-stream:*"
}
```

- 위의 계정 ID, 리전, Log Group 이름은 실제 환경의 값으로 바꿉니다.
- 자동 Log Group 생성을 선택했다면 `logs:CreateLogGroup`도 추가해야 합니다.
- ECR 이미지 가져오기와 일반적인 로그 전송 권한은 AWS 관리형 정책
  `AmazonECSTaskExecutionRolePolicy`에 포함되어 있습니다.

| 역할                | 용도                                                                                |
| ------------------- | ----------------------------------------------------------------------------------- |
| Task Execution Role | ECS/Fargate가 ECR 이미지를 가져오고 CloudWatch Logs에 기록할 때 사용합니다.         |
| Task Role           | 컨테이너 안의 애플리케이션이 S3, DynamoDB, SQS 등의 AWS API를 호출할 때 사용합니다. |

Task Execution Role의 역할과 추가 권한 조건은 [ECS IAM 역할 문서](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs-iam-role-overview.html)에서 확인할 수 있습니다.

## 로그가 보이지 않을 때 확인할 항목

| 증상                             | 먼저 확인할 항목                             | 조치                                                                           |
| -------------------------------- | -------------------------------------------- | ------------------------------------------------------------------------------ |
| Log Group이 없습니다.            | `awslogs-group` 이름, 리전, 자동 생성 옵션   | Log Group을 미리 만들거나 `logs:CreateLogGroup` 권한을 추가합니다.             |
| Log Stream이 없습니다.           | Task 실행 실패 이벤트, Execution Role        | ECS 서비스 이벤트와 중지 사유를 확인하고 `CreateLogStream` 권한을 점검합니다.  |
| Stream은 있지만 로그가 없습니다. | 애플리케이션의 표준 출력과 오류 출력         | 로그 설정을 콘솔 출력으로 바꾸고 컨테이너 시작 명령을 확인합니다.              |
| 일부 로그가 없습니다.            | `non-blocking` 모드의 버퍼 크기와 로그 폭주  | 버퍼 크기, 로그 수준, 전송량을 검토하고 필요한 경우 로그 처리 방식을 바꿉니다. |
| Task가 바로 종료됩니다.          | ECS stopped reason, 종료 코드, 서비스 이벤트 | CloudWatch Logs와 ECS 이벤트를 시간순으로 함께 확인합니다.                     |

- 권한 오류나 이미지 가져오기 실패처럼 컨테이너 시작 전에 발생한 문제는 애플리케이션 Log Stream에
  남지 않을 수 있습니다.
- 이 경우 ECS 서비스 이벤트와 Task의 `stoppedReason`을 먼저 확인합니다.

## Fargate에서 관찰할 수 있는 범위

Fargate는 AWS가 실행 호스트를 관리하는 서버리스 컨테이너 실행 환경입니다.

따라서 컨테이너와 서비스 수준의 정보는 볼 수 있지만, 호스트 운영체제 자체에는 접근할 수 없습니다.

| 항목                           | Fargate에서 확인 | 설명                                                              |
| ------------------------------ | ---------------- | ----------------------------------------------------------------- |
| 애플리케이션 로그              | 가능             | `stdout`과 `stderr`를 CloudWatch Logs로 전송합니다.               |
| ECS 서비스와 Task 상태         | 가능             | ECS 콘솔, 서비스 이벤트, Task 중지 사유를 사용합니다.             |
| 컨테이너 지표                  | 가능             | CloudWatch 지표와 Container Insights 설정을 사용합니다.           |
| 호스트 OS와 Kernel 로그        | 불가             | Fargate 호스트는 AWS가 관리합니다.                                |
| Docker daemon과 ECS agent 로그 | 불가             | 실행 호스트의 구성 요소에 접근할 수 없습니다.                     |
| 호스트에 Agent 설치            | 불가             | CloudWatch Agent, Node Exporter 등을 호스트에 설치할 수 없습니다. |
| SSH 접속                       | 불가             | 호스트에 직접 로그인할 수 없습니다.                               |

컨테이너 내부를 조사해야 하면 SSH 대신 ECS Exec 사용 가능 여부를
검토할 수 있습니다. 다만 ECS Exec는 운영 장애 분석을 보조하는
기능이며, 로그 수집 설정을 대체하지는 않습니다.

## `awslogs`와 FireLens 선택 기준

| 상황                                                | 권장 선택                            | 이유                                                                   |
| --------------------------------------------------- | ------------------------------------ | ---------------------------------------------------------------------- |
| CloudWatch Logs에서 애플리케이션 로그만 조회합니다. | `awslogs`                            | 설정이 단순하고 ECS 콘솔에서 Stream을 바로 찾을 수 있습니다.           |
| 여러 로그 대상에 전달하거나 형식을 변환해야 합니다. | FireLens                             | Fluent Bit 기반 로그 라우터로 필터링과 다중 전송을 구성할 수 있습니다. |
| 분산 추적을 함께 분석합니다.                        | `awslogs` + OpenTelemetry 또는 X-Ray | 로그와 trace ID를 함께 남겨 요청 흐름을 연결합니다.                    |

FireLens를 사용할 때는 로그 라우터 컨테이너의 리소스와 대상 서비스에
쓰기 위한 IAM 권한도 설계해야 합니다.

Fargate에서 지원하는 로그 드라이버와 FireLens 사용 조건은
[ECS LogConfiguration API 문서](https://docs.aws.amazon.com/AmazonECS/latest/APIReference/API_LogConfiguration.html)
및 [FireLens 공식 문서](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/using_firelens.html)
를 참고합니다.

## 핵심 정리

- Fargate도 `awslogs` 로그 드라이버로 `stdout`과 `stderr`를
  CloudWatch Logs에 보낼 수 있습니다.
- Log Group을 미리 만들고 보존 기간을 설정하면 비용과 운영 기준을
  일관되게 관리할 수 있습니다.
- 로그 전송 권한은 애플리케이션의 Task Role이 아니라 Task Execution
  Role에 부여합니다.
- 로그가 없으면 CloudWatch Logs만 보지 말고 ECS 서비스 이벤트와
  Task 중지 사유를 함께 확인합니다.
- Fargate에서는 컨테이너와 서비스 수준을 관찰할 수 있으며,
  호스트 운영체제와 Docker daemon에는 접근할 수 없습니다.
