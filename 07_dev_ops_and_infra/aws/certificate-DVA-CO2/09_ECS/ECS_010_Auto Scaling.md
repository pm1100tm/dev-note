# Auto Scaling

[전체 목차](<./ECS_001_컨테이너와 ECS 기본 개념.md>)

## 학습 목표

- Service Auto Scaling과 Cluster 용량 확장을 구분한다.
- Target Tracking, Step Scaling, Scheduled Scaling을 선택한다.
- 적절한 지표와 cooldown을 설계한다.

## 두 가지 확장 계층

```text
Application Load
  -> ECS Service Auto Scaling
     -> desired Task count 증가
        -> EC2 Launch Type라면 Cluster capacity도 필요
```

Fargate에서는 Task에 필요한 기반 용량을 AWS가 관리한다. EC2 방식에서는 Service가 Task 수를 늘려도 인스턴스의 CPU/메모리가 부족하면 Task가 `PENDING`이므로 Auto Scaling Group 또는 Capacity Provider의 managed scaling도 필요하다.

## Service Auto Scaling

Application Auto Scaling이 ECS Service의 desired count를 최소값과 최대값 사이에서 조정한다.

### Target Tracking

지표를 목표값 근처로 유지하도록 Task 수를 자동 조정한다. 가장 단순하고 일반적인 선택이다.

- `ECSServiceAverageCPUUtilization`
- `ECSServiceAverageMemoryUtilization`
- ALB의 Task당 요청 수
- 요구조건을 만족하는 사용자 지정 CloudWatch 지표

### Step Scaling

CloudWatch Alarm의 초과 정도에 따라 정해진 단계만큼 Task 수를 변경한다. 부하 수준별로 서로 다른 증감 폭이 필요할 때 사용한다.

### Scheduled Scaling

예측 가능한 시간대에 최소/최대/desired capacity를 미리 바꾼다. 매일 오전 9시 트래픽처럼 일정한 패턴에 적합하다.

## 좋은 확장 지표

지표는 수요에 비례하고 Task 수가 늘면 Task당 값이 낮아지는 특성이 있어야 한다. CPU가 실제 병목이면 CPU가 적절하지만, SQS worker는 queue backlog가 더 직접적일 수 있다.

예를 들어 다음 사용자 지정 지표를 사용할 수 있다.

```text
Backlog per task = ApproximateNumberOfMessagesVisible / RunningTaskCount
```

메시지 처리 시간이 길다면 단순 queue length보다 Task당 backlog 목표가 확장 의도를 잘 반영한다.

## Cooldown과 안정성

- Scale-out cooldown: 새 Task 효과가 지표에 반영될 시간을 고려
- Scale-in cooldown: 너무 빠른 축소로 진동하거나 요청을 끊지 않도록 보수적으로 설정
- 최소 Task 수: 가용성과 cold start 요구를 반영
- 최대 Task 수: downstream과 비용 한도를 보호

일반적으로 장애 대응을 위해 scale-out은 빠르게, scale-in은 더 신중하게 설정하지만 애플리케이션 특성으로 검증해야 한다.

## 배포 중 Auto Scaling

배포와 확장이 동시에 일어나면 Task 수와 지표 해석이 복잡해질 수 있다. 배포 구성의 `maximumPercent`, Cluster 여유 용량, Scaling 최대값을 함께 점검한다. Scale-in이 배포 안정성을 해치지 않도록 배포 동작과 정책을 테스트한다.

## 실무 적용

1. 병목과 사용자 영향에 가까운 지표를 정한다.
2. 부하 테스트로 Task 1개당 처리량을 측정한다.
3. min/max와 target을 설정한다.
4. Task 시작 시간에 맞춰 cooldown을 조정한다.
5. Alarm, Service events, 비용을 함께 관찰한다.

## 시험 포인트

- 평균 CPU를 목표값으로 유지: Target Tracking
- 임계치 초과 정도별 증감: Step Scaling
- 정해진 영업 시간 전에 확장: Scheduled Scaling
- EC2에서 Task가 `PENDING`: Cluster capacity 확인
- 요청량과 상관없는 지표는 Scaling 지표로 부적절할 수 있다.

## 연습문제

**문제.** SQS 메시지 처리 Service의 CPU 사용률은 낮지만 큐 대기가 계속 증가한다. 가장 적절한 확장 기준은?

**정답:** 메시지 backlog, 가능하면 running Task당 backlog를 CloudWatch 지표로 만들어 Service Auto Scaling에 사용한다. CPU는 실제 수요를 나타내지 못하고 있다.

## 확인 문제

- [ ] Task 확장과 EC2 용량 확장을 구분할 수 있다.
- [ ] 세 Scaling 정책의 차이를 설명할 수 있다.
- [ ] 좋은 Scaling 지표의 특성을 말할 수 있다.

[이전](<./ECS_009_Load Balancer 연동.md>) | [다음: 로그와 모니터링](<./ECS_011_로그와 모니터링.md>)
