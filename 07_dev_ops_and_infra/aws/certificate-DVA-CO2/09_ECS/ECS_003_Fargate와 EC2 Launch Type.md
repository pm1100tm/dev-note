# Fargate와 EC2 Launch Type

[전체 목차](<./ECS_001_컨테이너와 ECS 기본 개념.md>)

## 학습 목표

- 두 실행 방식의 책임 범위를 비교한다.
- 문제의 제약 조건으로 적절한 방식을 선택한다.
- Capacity Provider와 Spot 사용 시 주의점을 이해한다.

## 핵심 비교

| 항목 | Fargate | EC2 Launch Type |
| --- | --- | --- |
| 서버 관리 | AWS가 기반 인프라 관리 | 사용자가 EC2와 AMI 관리 |
| 과금 단위 | Task에 할당한 vCPU, 메모리 등 | EC2 인스턴스 용량 |
| 용량 계획 | Task 수준 | Cluster 인스턴스 수준 |
| 네트워크 | `awsvpc` 필수 | `awsvpc`, `bridge`, `host` 등 |
| 격리 | Task별 격리 경계 | 같은 인스턴스의 Task가 커널 공유 |
| 호스트 접근 | 제한적 | 가능 |
| 적합한 경우 | 운영 부담 최소화, 가변 부하 | 특수 인스턴스, 호스트 제어, 높은 사용률 |

## Fargate

Fargate는 서버리스 컴퓨팅 옵션이다. 여기서 서버리스는 서버가 없다는 뜻이 아니라 사용자가 기반 EC2 인스턴스를 선택, 패치, 확장하지 않는다는 뜻이다.

Task Definition에서 지원되는 CPU와 메모리 조합을 선택해야 한다. 각 Task는 `awsvpc` 네트워크 모드를 사용하고 ENI를 통해 VPC 네트워크에 연결된다.

적합한 상황:

- 빠르게 운영을 시작해야 한다.
- 워크로드 사용량이 변동한다.
- 인스턴스 패치와 ECS Agent 관리를 피하고 싶다.
- Task 단위 격리가 중요하다.

## EC2 Launch Type

ECS 최적화 AMI 등을 사용한 EC2 인스턴스를 Cluster에 등록한다. ECS Scheduler는 가용 CPU, 메모리, 포트, 배치 제약을 보고 Task를 배치한다.

적합한 상황:

- 특정 EC2 인스턴스 유형이나 호스트 기능이 필요하다.
- 지속적인 고사용률에서 인스턴스 단위 비용 최적화가 중요하다.
- 사용자 지정 AMI, 커널 설정 또는 호스트 수준 Agent가 필요하다.
- EC2 Spot과 다양한 구매 옵션을 세밀하게 운영한다.

EC2 방식에서는 두 종류의 확장을 구분해야 한다.

1. **Service Auto Scaling**: Task 수를 변경한다.
2. **Cluster 용량 확장**: Task를 배치할 EC2 인스턴스 수를 변경한다.

Task만 늘리고 인스턴스 용량이 부족하면 Task는 `PENDING`에 머무를 수 있다. Auto Scaling Group Capacity Provider의 managed scaling으로 이 문제를 줄일 수 있다.

## Fargate Spot

Fargate Spot은 여유 용량을 할인된 가격으로 사용하지만 용량 회수로 Task가 중단될 수 있다. 재시도 가능하고 상태를 외부 저장소에 보관하는 워크로드에 적합하다.

중단에 민감한 API의 최소 Task는 `FARGATE`에 두고, 추가 용량을 `FARGATE_SPOT`에 분산하는 전략을 사용할 수 있다.

## 실무 선택 절차

1. 호스트 접근, 특수 하드웨어, 특정 인스턴스 요구가 있는지 확인한다.
2. 없다면 운영 단순성이 높은 Fargate를 우선 검토한다.
3. 워크로드의 지속 시간과 사용률로 비용을 비교한다.
4. Spot 중단을 견딜 수 있는지 확인한다.
5. 가용 영역 분산과 최소 안정 용량을 설계한다.

## 시험 포인트

- `서버 프로비저닝과 패치 없음`은 Fargate 신호다.
- `특정 EC2 유형`, `호스트 접근`, `사용자 AMI`는 EC2 신호다.
- Fargate도 Task CPU와 메모리, 네트워크, IAM은 사용자가 설정한다.
- EC2 Launch Type에서 Task 확장과 인스턴스 확장은 별개다.
- Spot은 중단 가능성을 애플리케이션이 처리해야 한다.

## 연습문제

**문제 1.** 개발팀은 트래픽이 불규칙한 API를 운영하며 인스턴스 패치에 시간을 쓰고 싶지 않다. 가장 운영 부담이 적은 선택은?

**정답: Fargate.** Task 수준의 리소스를 지정하고 기반 인스턴스는 AWS가 관리한다.

**문제 2.** EC2 방식 Service의 desired count를 20으로 늘렸지만 새 Task가 `PENDING`이다. CPU와 메모리를 수용할 컨테이너 인스턴스가 없다. 무엇을 확장해야 하는가?

**정답:** Cluster의 EC2 용량이다. Auto Scaling Group Capacity Provider의 managed scaling을 검토한다.

## 확인 문제

- [ ] Fargate에서도 직접 설정해야 하는 항목을 말할 수 있다.
- [ ] EC2 방식의 두 확장 계층을 구분할 수 있다.
- [ ] Spot에 적합하지 않은 워크로드를 설명할 수 있다.

[이전](<./ECS_002_ECS 구성요소.md>) | [다음: Task Definition](<./ECS_004_Task Definition.md>)
