# ECS Service와 배포

[전체 목차](<./ECS_001_컨테이너와 ECS 기본 개념.md>)

## 학습 목표

- desired count와 Scheduler의 역할을 이해한다.
- Rolling Update의 최소/최대 실행 Task 수를 계산한다.
- Circuit Breaker와 Blue/Green의 사용 목적을 안다.

## Service의 역할

ECS Service는 지정한 수의 Task를 유지하고 새 Task Definition으로 배포한다. Task가 비정상 종료되거나 Load Balancer health check를 통과하지 못하면 대체 Task를 시작한다.

주요 설정:

- `desiredCount`: 정상적으로 유지할 Task 수
- 배포 전략 또는 Controller
- `minimumHealthyPercent`, `maximumPercent`
- health check grace period
- Load Balancer와 Target Group
- Capacity Provider Strategy

## Rolling Update

Rolling Update는 새 Task를 시작하고 정상 상태가 되면 기존 Task를 단계적으로 중지한다.

desired count가 4이고 다음과 같이 설정했다고 가정한다.

```text
minimumHealthyPercent = 50
maximumPercent = 200
```

- 배포 중 정상 상태 Task의 하한: `ceil(4 x 0.5) = 2`
- 배포 중 전체 Task의 상한: `floor(4 x 2.0) = 8`

최소 비율이 낮으면 용량 여유가 적어도 배포할 수 있지만 서비스 처리 능력이 줄 수 있다. 최대 비율이 높으면 빠르게 새 Task를 띄울 수 있지만 일시적으로 더 많은 컴퓨팅 용량이 필요하다.

## Health Check Grace Period

시작이 느린 애플리케이션은 준비되기 전에 ALB health check 실패로 교체 루프에 빠질 수 있다. Grace period 동안 초기 Load Balancer health 실패를 Scheduler 판단에서 유예할 수 있다. 단순히 시간을 늘리기 전에 애플리케이션 시작 시간과 health endpoint도 점검한다.

## Deployment Circuit Breaker

새 배포가 안정 상태에 도달하지 못하면 Circuit Breaker가 배포 실패를 감지한다. rollback 옵션을 켜면 마지막으로 완료된 배포로 되돌릴 수 있다. CloudWatch Alarm 기반 실패 감지도 함께 사용할 수 있다.

## Blue/Green

Blue/Green은 기존 Task set과 새 Task set을 동시에 준비하고 검증한 후 트래픽을 전환한다.

```text
Blue: current version <- production traffic
Green: new version    <- test traffic
                 검증 후 traffic shift
```

빠른 롤백과 사전 검증에 유리하지만 두 환경을 동시에 실행할 용량, Target Group 및 트래픽 전환 구성이 필요해 비용과 복잡성이 증가한다. DVA-C02 문맥에서는 CodeDeploy 기반 ECS Blue/Green, 두 Target Group, production/test listener가 자주 함께 등장한다.

## 실무 배포 절차

1. 고유 버전 태그로 이미지를 ECR에 push한다.
2. 새 Task Definition revision을 등록한다.
3. Service를 새 revision으로 업데이트한다.
4. 배포 상태, Service events, ALB health, 오류율을 관찰한다.
5. 안정 상태를 확인하고 이전 이미지를 보존한다.

배포 전에 Cluster 또는 Fargate quota가 `maximumPercent`가 요구하는 추가 Task를 수용할 수 있는지 확인한다.

## 시험 포인트

- desired count는 Service가 유지할 정상 Task 수다.
- `minimumHealthyPercent`는 배포 중 유지할 정상 Task 하한이다.
- `maximumPercent`는 배포 중 허용할 전체 Task 상한이다.
- 자동 실패 감지와 rollback: Deployment Circuit Breaker
- 새 환경을 미리 검증하고 트래픽 전환: Blue/Green

## 연습문제

**문제.** desired count 10, minimum 50%, maximum 100%인 Rolling Update에서 배포 중 최소 정상 Task 수와 최대 전체 Task 수는?

**정답:** 최소 5개, 최대 10개다. 추가 Task를 10개 초과 실행할 수 없으므로 기존 Task 일부를 먼저 중지해야 한다.

## 확인 문제

- [ ] 백분율로 Task 하한과 상한을 계산할 수 있다.
- [ ] Grace period와 Circuit Breaker의 차이를 설명할 수 있다.
- [ ] Blue/Green의 장점과 비용을 설명할 수 있다.

[이전](<./ECS_007_ECR 연동.md>) | [다음: Load Balancer 연동](<./ECS_009_Load Balancer 연동.md>)
