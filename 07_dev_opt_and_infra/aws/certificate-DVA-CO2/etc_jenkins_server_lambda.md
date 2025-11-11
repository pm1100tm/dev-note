# 🚀 Jenkins Server 가 필요할 때만 켤 수 있도록 자동화 할 수 있을까

📌 결론부터

Jenkins 자체를 Lambda로 직접 옮기는 건 사실상 불가능합니다.

- Jenkins는 상시 실행되는 서버형 애플리케이션이라서, Lambda처럼
  “짧게 실행되고 종료되는 함수 실행 환경”과 맞지 않습니다.
- 플러그인, 웹 UI, job 관리 등은 Lambda 모델에 맞지 않음.

📌 대안 (비용 절감 + 필요할 때만 Jenkins 사용)

[1.] EC2 스케줄링 (On/Off)

- Jenkins를 계속 EC2에 두되, 평소에는 꺼두고 필요할 때만 켜기
- 방법:
  - AWS EventBridge + Lambda 조합 → 정해진 시간에 EC2 Start/Stop
  - 또는 AWS CLI로 수동 시작/중지:
- 하루 1~2번 CI/CD만 필요하다면, Jenkins를 켤 때만 과금되므로 비용을 크게 줄일 수 있음

```shell
aws ec2 start-instances --instance-ids i-xxxx
aws ec2 stop-instances --instance-ids i-xxxx
```

[2.] 컨테이너 기반으로 Jenkins 구동

- ECS Fargate 나 EKS (쿠버네티스) 에 Jenkins 컨테이너를 배포
- 필요할 때만 태스크/파드를 실행
- 실행 시간만큼만 비용 지불

[3.] 서버리스 CI/CD 서비스 사용 (Jenkins 대체)

- Jenkins를 꼭 고집하지 않는다면 → AWS CodePipeline + CodeBuild
- 특징:
  - 완전 관리형 (서버 관리 불필요)
  - 실행 시간만큼만 비용 발생 (빌드 단위 과금)
  - GitHub, CodeCommit 같은 리포지토리와 바로 연동

📌 정리

- Jenkins 자체를 Lambda로 옮기는 건 불가 (아키텍처가 맞지 않음)
- 대신
  - EC2 Jenkins → 필요할 때만 Start/Stop (가장 간단, 현재 구조 유지)
  - Jenkins 컨테이너 → Fargate/EKS 로 실행 (컨테이너 기반 전환)
  - Jenkins → CodePipeline/CodeBuild 로 완전 대체 (서버리스 CI/CD)

👉 지금 상황에서는 “EC2를 껐다 켰다 하는 방법(자동화)” 이 가장 적합한데,
EventBridge + Lambda로 Jenkins EC2를 자동 시작/중지하는 구조는 아래와 같습니다.

구조 설명:

- Developer → EventBridge 트리거 (수동 또는 스케줄)
- EventBridge → Lambda 함수 호출
- Lambda → EC2 Jenkins 서버 Start/Stop API 실행
- Jenkins가 켜진 후, 개발자가 Jenkins UI 접속
- Jenkins → Spring Boot 앱 서버에 CI/CD 배포 진행
