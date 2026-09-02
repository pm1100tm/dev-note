# ECS 구성요소

[전체 목차](<./ECS_001_컨테이너와 ECS 기본 개념.md>)

## 학습 목표

- Cluster, Task Definition, Task, Service를 구분한다.
- Container Definition과 Capacity Provider의 위치를 이해한다.
- 일회성 Task와 장기 실행 Service를 선택할 수 있다.

## 전체 구조

```text
ECS Cluster
  +-- Service: desired count 유지 및 배포
  |     +-- Task
  |     |     +-- application container
  |     |     +-- sidecar container
  |     +-- Task
  +-- RunTask: 일회성 Task
```

## Cluster

Cluster는 Task와 Service를 논리적으로 묶는 관리 경계이다. Cluster 자체가 서버는 아니다.
Fargate Cluster는 사용자가 EC2 인스턴스를 등록하지 않아도 되며, EC2 방식에서는 컨테이너 인스턴스의 용량이 필요하다.

실무에서는 환경이나 워크로드의 운영 경계에 따라 Cluster를 나눈다. 지나치게 세분화하면 관리 비용이 증가하고, 모든 시스템을 한 Cluster에 넣으면 권한과 장애 범위를 분리하기 어렵다.

## Task Definition과 revision

Task Definition은 컨테이너 실행 방법을 선언한 설계도다. 변경할 수 없는 revision으로 등록되며 설정을 바꾸면 새 revision을 만든다.

주요 내용은 다음과 같다.

- 컨테이너 이미지 URI
- Task 및 컨테이너 CPU와 메모리
- 포트 매핑
- 환경 변수와 secret 참조
- Task Role과 Task Execution Role
- 로그 드라이버
- 볼륨과 네트워크 모드

`family:revision`의 예는 `orders-api:12`이다. Service가 새 revision을 사용하도록 업데이트하면 배포가 시작된다.

## Container Definition

Task Definition 안에서 개별 컨테이너를 정의한다. 한 Task에 애플리케이션 컨테이너와 로그 라우터 같은 sidecar를 함께 둘 수 있다.

- `essential: true`인 핵심 컨테이너가 종료되면 Task도 중지된다.
- 같은 Task의 컨테이너는 생명주기와 리소스를 함께 고려해야 한다.
- 서로 독립적으로 확장해야 하는 프로세스는 별도 Service가 더 적합하다.

## Task

Task는 Task Definition을 실제로 실행한 인스턴스다. Task 상태는 `PROVISIONING`, `PENDING`, `RUNNING`, `DEPROVISIONING`, `STOPPED` 등으로 바뀐다.

배치 변환처럼 한 번 실행하고 종료하는 작업은 `RunTask`가 적합하다. EventBridge Scheduler가 정해진 시간에 Task를 실행하도록 만들 수도 있다.

## Service

Service는 지정한 `desiredCount`만큼 Task를 지속적으로 유지한다. Task가 실패하면 Scheduler가 대체 Task를 시작한다. 웹 API, 상시 실행 worker처럼 계속 살아 있어야 하는 워크로드에 사용한다.

```text
desiredCount = 3
Task 하나 중지
  -> Scheduler가 새 Task 시작
  -> RUNNING Task를 다시 3개로 유지
```

Service는 배포, 로드 밸런서 등록, Service Auto Scaling도 관리한다. 반면 Task 자체는 원하는 개수를 유지하지 않는다.

## Capacity Provider

Capacity Provider는 Task에 컴퓨팅 용량을 공급하는 방법을 표현한다.

- `FARGATE`: 온디맨드 Fargate 용량
- `FARGATE_SPOT`: 중단 가능한 저비용 Fargate 용량
- EC2 Auto Scaling Group 기반 Capacity Provider

Capacity Provider Strategy의 `base`는 특정 Provider에서 먼저 실행할 최소 Task 수이고, `weight`는 나머지 Task의 상대적 분배 비율이다.

## 실무 적용

- 상시 API: Service + ALB + desired count 2 이상
- 큐 소비자: Service + SQS 기반 Auto Scaling
- 야간 정산: EventBridge Scheduler + `RunTask`
- 로그 라우팅: 앱 컨테이너 + FireLens sidecar

Service와 독립 Task를 구분하면 불필요한 재시작을 피할 수 있다. 종료가 정상인 배치 작업을 Service로 만들면 ECS가 계속 다시 실행한다.

## 시험 포인트

| 요구사항                    | 선택                        |
| --------------------------- | --------------------------- |
| 컨테이너 실행 설정 변경     | 새 Task Definition revision |
| 항상 N개 실행               | Service의 desired count     |
| 한 번만 실행                | RunTask                     |
| 실패한 장기 실행 Task 교체  | Service Scheduler           |
| Fargate와 Fargate Spot 혼합 | Capacity Provider Strategy  |

## 연습문제

**문제.** 매일 새벽 데이터 변환 컨테이너를 한 번 실행하고 정상 종료해야 한다. 가장 적절한 구성은?

**정답:** EventBridge Scheduler가 ECS `RunTask`를 호출하도록 구성한다. Service는 종료된 Task를 다시 시작하므로 일회성 작업과 맞지 않는다.

## 확인 문제

- [ ] Task Definition과 Task의 차이를 설명할 수 있다.
- [ ] Service가 하는 일을 세 가지 이상 말할 수 있다.
- [ ] `base`와 `weight`의 의미를 구분할 수 있다.

[이전](<./ECS_001_컨테이너와 ECS 기본 개념.md>) | [다음: Fargate와 EC2 Launch Type](<./ECS_003_Fargate와 EC2 Launch Type.md>)
