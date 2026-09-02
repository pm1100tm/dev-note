# Load Balancer 연동

[전체 목차](<./ECS_001_컨테이너와 ECS 기본 개념.md>)

## 학습 목표

- ALB, NLB를 요구사항에 맞게 선택한다.
- Listener, Rule, Target Group, ECS Task의 흐름을 이해한다.
- Health check와 Dynamic Port Mapping을 설명한다.

## 요청 흐름

```text
Client
  -> Load Balancer Listener :443
  -> Listener Rule
  -> Target Group
  -> Healthy ECS Task
```

ECS Service는 Task가 시작되면 Target Group에 등록하고 중지할 때 등록을 해제한다. Load Balancer는 health check를 통과한 대상에만 트래픽을 전달한다.

## ALB와 NLB

| 요구사항 | 선택 |
| --- | --- |
| HTTP/HTTPS, host/path routing | ALB |
| 여러 서비스에 URL 경로별 라우팅 | ALB |
| TCP/UDP, Layer 4 처리 | NLB |
| 고정 IP 또는 소스 IP 관련 요구 | NLB 검토 |

일반 웹 API는 ALB가 기본 선택이다. 한 ALB의 규칙으로 `/orders/*`와 `/payments/*`를 서로 다른 ECS Service Target Group에 연결할 수 있다.

## Target Group 대상 유형

- Fargate 또는 `awsvpc`: `ip`
- EC2 `bridge` 모드의 Dynamic Port Mapping: 보통 `instance`

`awsvpc` Task는 EC2 인스턴스가 아니라 Task ENI의 IP가 네트워크 대상이므로 `ip` 유형을 사용한다.

## Dynamic Host Port Mapping

EC2 Launch Type의 `bridge` 모드에서 `hostPort`를 0 또는 생략하면 ECS가 동적 포트를 할당할 수 있다. ALB는 `EC2 instance ID + dynamic port` 조합으로 Task에 연결한다. 따라서 같은 인스턴스에서 같은 `containerPort`를 쓰는 Task 여러 개를 실행할 수 있다.

Fargate `awsvpc`에서는 각 Task에 별도 IP가 있으므로 동일한 컨테이너 포트를 Task마다 사용할 수 있다.

## Health Check

좋은 health endpoint는 프로세스가 떠 있다는 사실뿐 아니라 요청을 처리할 준비가 되었는지 빠르게 판단해야 한다. 다음 설정이 서로 맞아야 한다.

- Health check path와 port
- 성공 status code 범위
- interval, timeout, healthy/unhealthy threshold
- Task Security Group의 인바운드 규칙
- 애플리케이션 listen address와 container port

ALB health check 실패가 계속되면 ECS Service가 Task를 반복 교체할 수 있다. Service events와 Target Group의 reason code, 컨테이너 로그를 함께 본다.

## Deregistration Delay

배포 또는 scale-in 시 Target을 해제해도 진행 중 요청을 마칠 시간이 필요할 수 있다. Deregistration delay와 애플리케이션의 graceful shutdown 시간을 조정해 연결 중단을 줄인다.

## 실무 보안 구성

- ALB SG: 인터넷 또는 신뢰 네트워크에서 443 허용
- Task SG: ALB SG에서 오는 애플리케이션 포트만 허용
- TLS 인증서는 ACM에서 관리하고 ALB Listener에 연결
- Task를 private subnet에 두고 직접 공개하지 않음

## 시험 포인트

- 경로 기반 라우팅: ALB
- TCP/UDP: NLB
- Fargate Target Group: `ip`
- 같은 EC2에서 동일 container port Task 여러 개: Dynamic Host Port Mapping + ALB
- 배포 후 Task 반복 종료: Target Group health check와 SG 우선 확인

## 연습문제

**문제.** Fargate Service를 ALB에 연결했지만 Target 등록 시 대상 유형 오류가 발생한다. 무엇을 수정해야 하는가?

**정답:** Target Group의 target type을 `ip`로 생성한다. Fargate Task는 Task ENI의 IP로 등록된다.

## 확인 문제

- [ ] ALB와 NLB 선택 기준을 설명할 수 있다.
- [ ] Listener부터 Task까지 흐름을 그릴 수 있다.
- [ ] Dynamic Port Mapping의 장점을 설명할 수 있다.

[이전](<./ECS_008_ECS Service와 배포.md>) | [다음: Auto Scaling](<./ECS_010_Auto Scaling.md>)
