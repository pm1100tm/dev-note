# ECS 분산 환경의 배치 작업

Spring Boot 애플리케이션을 ECS Service로 여러 Task 실행하면,
애플리케이션 안의 `@Scheduled` 작업도 각 Task에서 실행될 수 있습니다.
같은 배치가 여러 번 실행되지 않게 실행 구조를 정해야 합니다.

먼저 기억할 점은 **Fargate와 EC2에 따라 중복 방지 방법이 달라지지는 않는다**는 것입니다.

두 방식 모두 ECS Task로 애플리케이션을 실행합니다. 차이는 Task를 실행할 컴퓨팅 환경이며, 배치를 어떤
방식으로 실행할지는 공통으로 두 가지 중 선택합니다.

- EventBridge Scheduler가 배치 전용 Task를 실행합니다.
- 기존 앱의 `@Scheduled`에서 배치를 실행하고, ShedLock 같은 공유
  락으로 여러 Task 중 한 곳만 실행하게 합니다.

## 왜 중복 실행될 수 있나요?

- ECS Service는 설정한 수만큼 Task를 계속 유지합니다.
- 예를 들어 웹 Service에 Task가 세 개 있으면 각 Task에 같은 Spring Boot 앱이
  실행됩니다.
- 앱 안에 같은 `@Scheduled`가 있으면 세 Task가 모두 스케줄 시각을 확인하고 배치를 시작할 수 있습니다.

```text
ECS Service: 같은 앱의 Task 3개
  ├── Task A: @Scheduled 실행
  ├── Task B: @Scheduled 실행
  └── Task C: @Scheduled 실행

결과: 같은 배치가 세 번 실행될 수 있음
```

- `desiredCount: 1`이어도 중복 방지가 되지는 않습니다.
- 배포 중 새 Task와 기존 Task가 잠시 같이 실행될 수 있고, 장애 복구나 재시도, 수동 실행도 겹칠 수 있습니다.

## 방법 1: 배치 전용 Task를 스케줄에 맞춰 실행

정해진 시각에 실행하고 끝나는 작업이라면 이 방법을 우선 고려합니다.

웹 앱과 배치 앱을 별도 프로젝트로 만들 수 있고, 하나의 저장소나 이미지에서 웹 실행과 배치 실행을
분리할 수도 있습니다.

> 중요한 점은 웹 Service의 복제본 수와 배치 실행을 분리하는 것입니다.

EventBridge Scheduler가 정해진 시각에 ECS `RunTask`를 호출합니다.

배치 Task는 작업을 마친 뒤 정상 종료합니다.

```text
EventBridge Scheduler
  └── ECS RunTask
        └── 배치 앱 시작 → 작업 실행 → 종료
```

- 이 방식에서는 웹 Task를 늘려도 배치 실행 수가 함께 늘지 않습니다.
- 배치 앱에는 웹 서버가 필요하지 않으며, 실행할 작업을 명령이나
  Spring 프로파일로 지정할 수 있습니다.

예를 들어 하나의 이미지에서 다음처럼 실행 모드를 나눌 수 있습니다.

```text
웹 Service:   java -jar app.jar --spring.profiles.active=web
배치 Task:    java -jar app.jar --spring.profiles.active=batch
```

- 스케줄러가 Task 시작을 재시도하거나 운영자가 같은 작업을 다시 실행할 수 있으므로, 이 방식도 중복 가능성을
  완전히 없애지는 않습니다.
- 같은 작업이 다시 실행되어도 결과가 중복 반영되지 않도록 DB 고유
  제약이나 처리 상태를 이용해 멱등하게 만듭니다.

EventBridge Scheduler의 ECS 대상에는 `ecs:RunTask`와 Task Role 및
Execution Role을 전달할 `iam:PassRole` 권한이 필요합니다.

실행 설정과 역할은 [ECS 스케줄링 공식 문서](https://docs.aws.amazon.com/AmazonECS/latest/developerguide/scheduling_tasks.html)를 참고합니다.

## 방법 2: 기존 앱의 `@Scheduled`를 유지

- 웹 앱 안에 `@Scheduled`를 두면 앱이 실행 중인 각 Task가 스케줄을 확인합니다.
- 이때 ShedLock처럼 모든 Task가 함께 사용하는 저장소에 락을 기록하면, 락을 얻은 Task 하나만 해당 배치를
  실행하게 할 수 있습니다.
- 예를 들어 RDS를 이미 사용한다면 ShedLock JDBC Provider를 검토할 수 있습니다.

```text
웹 Service: Task A, B, C
  └── 각 Task가 스케줄 시각 확인
        └── 공유 락을 얻은 Task 한 곳만 배치 실행
```

ShedLock을 쓰려면 배치 메서드에 락 이름을 지정하고, 모든 Task가 접속하는 공유 저장소를 설정합니다.

```java
@Scheduled(cron = "0 0 2 * * *")
@SchedulerLock(name = "dailySettlement", lockAtMostFor = "PT50M")
public void runDailySettlement() {
    // 배치 실행
}
```

- 락을 가진 Task가 실행 중이면 다른 Task의 같은 작업은 대기하지 않고 건너뜁니다.
- `lockAtMostFor` 시간이 실제 작업보다 짧으면 락이 먼저 풀려 중복 실행될 수 있으므로, 예상 최대 실행 시간보다 넉넉하게
  설정합니다.
- 작업 자체도 DB 고유 제약이나 처리 상태를 사용해 멱등하게 만듭니다.

> ShedLock은 스케줄 작업의 동시 실행을 막는 락입니다. 실패한 배치를
> 어느 단계부터 재시작할지 기록하는 기능은 제공하지 않습니다.

청크 처리, 단계별 이력, 재시작이 필요하면 Spring Batch를 함께 사용하고,
공유 DB 기반 `JobRepository`를 구성합니다.

- ShedLock의 동작은 [공식 문서](https://github.com/lukas-krecan/ShedLock)
- Spring Batch의 실행 이력은 [JobRepository 문서](https://docs.spring.io/spring-batch/reference/job/configuring-repository.html)

를 참고합니다.

## Fargate와 EC2의 차이

배치 중복 방지 원칙은 같고, `RunTask`가 실행될 컴퓨팅 환경만 다릅니다.

| 실행 방식 | 배치 전용 Task                                     | 기존 앱의 `@Scheduled`                                   |
| --------- | -------------------------------------------------- | -------------------------------------------------------- |
| Fargate   | Scheduler가 Fargate Task를 시작합니다.             | 모든 앱 Task가 스케줄을 확인하므로 공유 락을 사용합니다. |
| EC2       | Scheduler가 EC2 용량을 사용하는 Task를 시작합니다. | 모든 앱 Task가 스케줄을 확인하므로 공유 락을 사용합니다. |

EC2 시작 유형은 클러스터에 등록된 EC2 인스턴스에 실행 여유 용량이
있어야 합니다. Fargate는 사용자가 EC2 인스턴스 용량을 직접 관리하지
않습니다. 이 차이는 Task 실행 환경에 관한 것이며, 배치 중복 방지
방식을 바꾸지는 않습니다.

EC2의 `DAEMON` Service는 클러스터의 각 적격 EC2 인스턴스마다 Task
하나를 실행합니다. 따라서 클러스터 전체에서 배치를 한 번만 실행하는
방법으로 사용하면 안 됩니다. Fargate는 `DAEMON` 스케줄링을 지원하지
않습니다.

## Spring Batch는 어떤 역할을 하나요?

Spring Batch는 배치 작업을 단계로 나누고, 대량 데이터를 처리하며,
실행 이력과 실패 후 재시작을 관리하는 라이브러리입니다. 배치를
언제 시작할지 정하는 스케줄러와는 역할이 다릅니다.

```text
EventBridge Scheduler 또는 @Scheduled
  → Spring Batch Job 시작
    → 데이터 읽기·처리·쓰기
    → 실행 결과와 상태 기록
```

간단한 작업이면 일반 Spring 코드로 충분할 수 있습니다. 다음 기능이
필요할 때 Spring Batch를 검토합니다.

- 대량 데이터를 작은 묶음(Chunk)으로 처리합니다.
- 실패한 작업을 확인하고 다시 시작합니다.
- Job과 단계별 실행 이력을 저장합니다.
- 작업을 여러 단계나 파티션으로 나눠 처리합니다.

여러 Task에서 배치를 실행할 가능성이 있거나 재시작 이력이 필요하면
JobRepository를 공유 데이터베이스에 저장합니다. 인메모리 상태만
사용하면 Task가 달라질 때 실행 정보를 공유할 수 없습니다.

## 어떤 방법을 선택할까요?

| 조건                                                              | 선택                                        |
| ----------------------------------------------------------------- | ------------------------------------------- |
| 매일 정산, 집계, 파일 생성처럼 정해진 시각에 실행하고 끝나는 작업 | EventBridge Scheduler + 배치 전용 `RunTask` |
| 웹 앱 안에 이미 있는 짧은 주기 작업을 유지해야 함                 | `@Scheduled` + ShedLock + 멱등 처리         |
| 대량 처리, 단계별 상태, 실패 후 재시작이 필요함                   | Spring Batch + 공유 JobRepository           |

정기 배치를 처음 구성한다면 웹 Service와 배치 실행을 분리하는
방식이 이해하고 운영하기 쉽습니다. 기존 앱에 스케줄 작업을 둘
이유가 있으면 모든 Task가 스케줄을 실행한다는 점을 기억하고,
공유 락과 멱등 처리를 함께 적용합니다.

## 핵심 정리

- Fargate와 EC2는 배치 중복 방지 원칙이 같습니다.
- 기본 선택지는 배치 전용 Task를 EventBridge Scheduler의
  `RunTask`로 실행하거나, 기존 앱에서 `@Scheduled`와 ShedLock을
  함께 사용하는 방식입니다.
- 어느 방식이든 재시도와 수동 실행에 대비해 배치 결과를 멱등하게
  처리합니다.
- Spring Batch는 배치의 이력과 재시작을 관리하며, 스케줄러를
  대신하지 않습니다.
- EC2 `DAEMON` Service는 인스턴스마다 하나씩 실행되므로
  클러스터 전체에서 한 번만 실행하는 배치 용도가 아닙니다.

<br>

## 질문: 하나의 앱에서 web·batch와 local·dev·prod를 나눌 수 있나요?

> 하나의 Spring Boot 앱에 웹 코드와 배치 코드를 모두 두고
> `--spring.profiles.active=web` 또는 `batch`로 실행 모드를 나누면,
> 실행하지 않는 쪽의 코드도 앱에 들어 있지 않나요? 배치 앱에서도
> API와 OpenAPI 문서를 볼 수 있나요? 기존의 `local`, `dev`, `prod`
> 환경별 설정과 `web`, `batch` 실행 모드는 함께 사용할 수 있나요?

- 프로필은 지정한 설정과 Bean을 선택하는 기준입니다.
- 프로필만 바꾼다고 코드가 자동으로 분리되거나 삭제되지는 않습니다.
- 웹과 배치 코드가 모두 같은 앱에 포함되어도 되지만, 각 모드에서 실행할 Bean을
  `@Profile` 또는 조건부 설정으로 구분해야 합니다.

예를 들어 웹 컨트롤러와 웹 서버 설정은 `web` 프로필에서만 등록하고,
배치 실행 설정은 `batch` 프로필에서만 등록할 수 있습니다.

배치 실행에서 웹 서버를 시작하지 않으면 API 요청을 받을 서버가 없으므로
그 실행의 OpenAPI 주소에도 접속할 수 없습니다. OpenAPI 문서 설정만
으로 API 서버가 시작되는 것은 아닙니다.

실행 모드와 배포 환경은 별도 기준으로 함께 지정할 수 있습니다.

| 구분      | 예시                   | 정하는 내용                            |
| --------- | ---------------------- | -------------------------------------- |
| 실행 모드 | `web`, `batch`         | 웹 서버·컨트롤러 또는 배치 실행 설정   |
| 실행 환경 | `local`, `dev`, `prod` | DB 접속 정보, 로그 수준 등 환경별 설정 |

예를 들어 다음처럼 두 프로필을 함께 활성화할 수 있습니다.

```shell
java -jar app.jar --spring.profiles.active=web,dev
java -jar app.jar --spring.profiles.active=batch,prod
```

- 공통 설정은 `application.yml`
- 모드별 설정은 `application-web.yml`과 `application-batch.yml`
- 환경별 설정은 `application-local.yml`, `application-dev.yml`, `application-prod.yml`
  처럼 나눌 수 있습니다.

여러 프로필에서 같은 항목을 설정하면 어떤 값이 적용되는지 헷갈릴 수 있으므로, 공통값과
환경별 값을 구분하고 실제 실행 설정을 확인합니다.

한 앱을 유지하는 방식은 코드와 배포물을 공유하기 좋지만, 다만 웹과 배치 기능이 커지면 모듈이나 프로젝트를
나누는 편이 각 실행에 필요한 코드와 설정을 더 쉽게 파악할 수 있습니다.
