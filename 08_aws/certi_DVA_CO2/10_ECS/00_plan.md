# ECS 학습 목차별 계획

- [] 1

| 순서 | 목차                       | 학습 목표                                 | 핵심 키워드                                                                 |
| ---: | -------------------------- | ----------------------------------------- | --------------------------------------------------------------------------- |
|    1 | 컨테이너와 ECS 기본 개념   | ECS가 무엇을 관리하는지 이해              | Container, Image, Cluster, Task, Service, Task Definition                   |
|    2 | ECS 구성요소               | 시험에 자주 나오는 용어 정리              | Cluster, Task Definition, Task, Service, Container Definition               |
|    3 | Fargate vs EC2 Launch Type | 어떤 상황에서 Fargate/EC2를 고르는지 판단 | Serverless, EC2 capacity, Capacity Provider                                 |
|    4 | Task Definition            | ECS 문제의 중심. JSON 구성 이해           | Image, CPU, Memory, Port Mapping, Environment, Secrets, Logging             |
|    5 | ECS Networking             | 장애/배포 문제에서 자주 출제              | `awsvpc`, ENI, Subnet, Security Group, Public IP, NAT Gateway, VPC Endpoint |
|    6 | IAM 권한                   | DVA-C02에서 매우 중요                     | Task Role, Task Execution Role, ECR Pull, CloudWatch Logs, Secrets Manager  |
|    7 | ECR 연동                   | 컨테이너 이미지 저장/배포 흐름 이해       | ECR Repository, Image Tag, Lifecycle Policy, Scan                           |
|    8 | ECS Service와 배포         | 무중단 배포와 desired count 이해          | Rolling Update, Desired Count, Deployment Configuration                     |
|    9 | Load Balancer 연동         | 외부 트래픽을 ECS Task로 연결             | ALB, Target Group, Health Check, Dynamic Port Mapping                       |
|   10 | Auto Scaling               | 부하에 따라 Task 수 조정                  | Service Auto Scaling, CPU/Memory metric, Target Tracking                    |
|   11 | 로그와 모니터링            | 문제 해결 관점 학습                       | CloudWatch Logs, Container Insights, ECS Events                             |
|   12 | 보안 구성                  | 실전/시험 모두 중요                       | Secrets Manager, SSM Parameter Store, Security Group, least privilege       |
|   13 | CI/CD 배포                 | DVA-C02 배포 도메인 대응                  | CodePipeline, CodeBuild, CodeDeploy, Blue/Green Deployment                  |
|   14 | 장애 대응 시나리오         | 시험형 문제 풀이 대비                     | Task stopped, Image pull error, Health check fail, Permission denied        |
|   15 | 비용/운영 최적화           | 선택지 제거 능력 강화                     | Fargate size, Spot, Scaling policy, log retention                           |

## 추천 학습 순서

1일차: ECS 전체 구조
ECS, ECR, Fargate 관계 이해
Cluster, Task Definition, Task, Service 차이 암기
공식 문서: Amazon ECS task definitions

2일차: Task Definition 집중
컨테이너 이미지, CPU/Memory, port mapping, env, log 설정
Fargate에서 지원/비지원되는 설정 확인
공식 문서: Fargate task definition differences

3일차: 네트워크
awsvpc 모드
public subnet vs private subnet
NAT Gateway 없이 private task가 ECR 이미지를 못 가져오는 상황
Security Group과 ALB Target Group 연결

4일차: IAM
Task Execution Role: ECS가 ECR pull, CloudWatch Logs 전송할 때 사용
Task Role: 컨테이너 내부 애플리케이션이 S3, DynamoDB, SQS 등을 호출할 때 사용
이 둘을 구분 못 하면 ECS 문제를 자주 틀립니다.

5일차: 배포와 로드밸런싱
ECS Service desired count
rolling update
ALB health check 실패 시 배포가 멈추는 흐름
CodeDeploy blue/green 개념

6일차: 모니터링과 트러블슈팅
CloudWatch Logs
ECS service events
stopped task reason
image pull error, permission error, health check error 구분

7일차: 실습
ECR에 이미지 push
ECS Fargate Service 생성
ALB 연결
CloudWatch Logs 확인
환경변수/Secrets Manager 연결

## 시험에서 특히 잘 나오는 구분

| 헷갈리는 개념       | 정리                                                 |
| ------------------- | ---------------------------------------------------- |
| Task Definition     | 컨테이너 실행 설계도                                 |
| Task                | Task Definition으로 실제 실행된 단위                 |
| Service             | Task 개수를 유지하고 배포를 관리                     |
| Task Role           | 애플리케이션 코드가 AWS API 호출할 때 사용           |
| Task Execution Role | ECS Agent/Fargate가 이미지 pull, 로그 전송할 때 사용 |
| Fargate             | 서버 관리 없이 Task 실행                             |
| EC2 Launch Type     | 직접 EC2 용량과 인스턴스 관리                        |
| CloudWatch Logs     | 컨테이너 stdout/stderr 로그 수집                     |
| ALB Health Check    | Task 정상 여부 판단, 배포 성공/실패에 영향           |

## 최소 실습 목표

최소한 아래 흐름은 직접 한 번 해보는 걸 추천합니다.

```
Docker image build
→ ECR push
→ ECS Task Definition 생성
→ ECS Fargate Service 생성
→ ALB 연결
→ CloudWatch Logs 확인
→ Task Role로 S3 또는 DynamoDB 접근
→ Service Auto Scaling 설정
```
