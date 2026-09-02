# ECS Networking

[전체 목차](<./ECS_001_컨테이너와 ECS 기본 개념.md>)

## 학습 목표

- `awsvpc`, ENI, Subnet, Security Group의 관계를 이해한다.
- Public IP, NAT Gateway, VPC Endpoint의 용도를 구분한다.
- 이미지 pull 실패를 네트워크 관점에서 진단한다.

## awsvpc와 ENI

Fargate Task는 `awsvpc` 네트워크 모드를 사용한다. 각 Task에 ENI가 연결되고 VPC 내부 IP 주소와 Security Group을 가진다. 따라서 Task는 VPC 안의 독립된 네트워크 대상처럼 동작한다.

```text
VPC
  +-- public subnet: ALB
  +-- private subnet: ECS Task ENI
                         +-- Security Group
                         +-- private IP
```

Fargate 또는 `awsvpc` Task를 Load Balancer에 연결할 때 Target Group의 대상 유형은 `ip`를 사용한다.

## Public Subnet과 Private Subnet

Subnet이 public인지 여부는 인터넷 게이트웨이로 향하는 라우트만으로 결정되는 것이 아니라 리소스가 공인 IP를 갖고 있는지도 함께 봐야 한다.

운영 웹 서비스의 일반적인 구성은 다음과 같다.

- 인터넷-facing ALB: public subnet
- ECS Task: private subnet, public IP 비활성화
- 외부로 나가는 통신: NAT Gateway 또는 필요한 VPC Endpoint
- 인바운드: Task Security Group은 ALB Security Group에서 오는 앱 포트만 허용

## 이미지 pull 경로

Private subnet의 Task가 ECR 이미지를 가져오려면 ECR API, ECR Docker registry와 이미지 layer가 저장된 S3에 접근할 네트워크 경로가 필요하다.

선택지는 다음과 같다.

1. NAT Gateway를 통한 인터넷 경로
2. ECR API 및 ECR DKR용 Interface VPC Endpoint와 S3 Gateway Endpoint

CloudWatch Logs, Secrets Manager 같은 서비스도 NAT 없이 사용하려면 각각 필요한 VPC Endpoint를 검토한다. Endpoint를 만들었다는 사실만으로 충분하지 않으며 Endpoint Security Group, Private DNS, 라우팅과 IAM 권한도 확인해야 한다.

## Security Group 설계

```text
ALB-SG
  inbound: 443 from Internet
  outbound: 8080 to TASK-SG

TASK-SG
  inbound: 8080 from ALB-SG
  outbound: 필요한 목적지로 허용
```

IP 범위 대신 Security Group을 소스로 참조하면 ALB 노드의 주소 변경을 직접 관리할 필요가 없다. Security Group은 상태 저장 방식이므로 허용된 연결의 응답 트래픽을 별도 인바운드 규칙으로 추가하지 않는다.

## EC2 방식의 네트워크 모드

- `awsvpc`: Task별 ENI와 Security Group
- `bridge`: Docker bridge와 host port mapping 사용
- `host`: 컨테이너가 호스트 네트워크를 직접 사용하며 포트 충돌에 주의

시험에서는 Fargate가 보이면 `awsvpc`, Task별 ENI, `ip` Target Group을 함께 연결해 기억한다.

## 실무 진단 순서

Task가 시작되지 않으면 다음 순서로 확인한다.

1. ECS stopped reason과 Service event 확인
2. Subnet route table과 NAT 또는 Endpoint 확인
3. Task Security Group egress와 Endpoint Security Group 확인
4. DNS 및 VPC의 DNS 설정 확인
5. Task Execution Role의 ECR 권한 확인

네트워크 오류와 IAM 오류는 증상이 비슷할 수 있다. `timeout` 계열은 경로를, `AccessDenied`는 권한을 우선 의심한다.

## 시험 포인트

- Private subnet은 기본적으로 인터넷에 직접 나갈 수 없다.
- NAT Gateway는 public subnet에 두고 Elastic IP 및 인터넷 게이트웨이 경로가 필요하다.
- ECR VPC Endpoint만 만들고 S3 경로를 빠뜨리지 않는다.
- Task 인바운드는 `0.0.0.0/0`보다 ALB Security Group 소스가 적절하다.
- `awsvpc` Task의 Target Group 대상 유형은 `ip`다.

## 연습문제

**문제.** Public IP가 없는 Fargate Task가 private subnet에서 시작되지 않으며 ECR 이미지 pull timeout이 발생한다. 인터넷 경로 없이 해결하려 한다. 무엇이 필요한가?

**정답:** ECR API와 ECR DKR Interface Endpoint, S3 Gateway Endpoint를 구성하고 관련 DNS, Security Group, IAM을 확인한다. 로그와 secret도 사용하면 해당 서비스 Endpoint가 추가로 필요할 수 있다.

## 확인 문제

- [ ] ENI가 Task에 제공하는 기능을 설명할 수 있다.
- [ ] NAT와 VPC Endpoint의 차이를 안다.
- [ ] `ip` Target Group이 필요한 이유를 설명할 수 있다.

[이전](<./ECS_004_Task Definition.md>) | [다음: IAM 권한](<./ECS_006_IAM 권한.md>)
