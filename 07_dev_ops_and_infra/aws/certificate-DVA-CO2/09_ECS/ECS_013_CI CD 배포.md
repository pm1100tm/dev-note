# CI/CD 배포

[전체 목차](<./ECS_001_컨테이너와 ECS 기본 개념.md>)

## 학습 목표

- CodePipeline, CodeBuild, ECR, ECS의 배포 흐름을 이해한다.
- Rolling과 Blue/Green 파이프라인의 차이를 안다.
- 배포 Role과 애플리케이션 Role을 구분한다.

## 기본 파이프라인

```text
Source
  -> CodePipeline
  -> CodeBuild: test, docker build
  -> ECR: image push
  -> ECS Deploy: Task Definition / Service update
  -> health verification
```

CodePipeline은 단계를 연결하고, CodeBuild는 명령을 실행해 테스트와 이미지 빌드를 수행한다. ECR은 결과 이미지를 저장하고 ECS는 새 이미지를 실행한다.

## Build 단계

대표 작업은 다음과 같다.

1. 단위 테스트와 정적 분석
2. ECR 로그인
3. Docker image build
4. commit SHA와 release version 태그 지정
5. ECR push
6. 이미지 스캔 결과 또는 보안 Gate 확인
7. 배포 입력 artifact 생성

CodeBuild가 Docker image를 빌드하려면 빌드 환경 설정과 권한이 필요하다. ECR push 권한은 CodeBuild Service Role에 부여하며 ECS Task Role과 혼동하지 않는다.

## Rolling 배포 파이프라인

새 이미지 URI를 반영한 Task Definition revision을 등록하고 ECS Service를 업데이트한다. ECS Scheduler가 배포 설정에 따라 Task를 교체한다.

장점은 구조가 단순하고 추가 환경 비용이 비교적 적다는 것이다. 단점은 새 버전과 기존 버전이 잠시 동시에 트래픽을 처리하고 즉시 트래픽 전환/복귀가 어렵다는 점이다.

## Blue/Green 배포 파이프라인

DVA-C02에서 자주 다루는 CodeDeploy 기반 ECS Blue/Green 구성 요소는 다음과 같다.

- 기존 Blue와 새 Green Task set
- 두 개의 Target Group
- Production Listener와 선택적 Test Listener
- CodeDeploy Application과 Deployment Group
- AppSpec에서 Task Definition 및 Load Balancer container 정보 지정
- CodeDeploy가 ECS와 Load Balancer를 조작할 Service Role

Green을 테스트한 뒤 트래픽을 전환하며 실패 시 Blue로 빠르게 돌릴 수 있다. 대신 두 Task set을 실행할 용량과 더 많은 구성이 필요하다.

## IAM 분리

| 주체 | 필요한 권한 예 |
| --- | --- |
| CodeBuild Role | ECR push, build log 작성, artifact 접근 |
| Pipeline Role | 단계별 서비스 호출과 artifact 전달 |
| CodeDeploy Role | ECS Service/Task set과 Load Balancer 조작 |
| Task Execution Role | 배포된 Task의 ECR pull과 로그 전송 |
| Task Role | 실행 중 애플리케이션의 AWS API 호출 |

## 안전한 실무 배포

- 태그가 아닌 commit SHA/digest로 결과를 추적한다.
- 같은 artifact를 환경 간 승격하고 다시 build하지 않는다.
- health check뿐 아니라 오류율과 핵심 기능 검증을 둔다.
- CloudWatch Alarm과 자동 rollback을 구성한다.
- DB schema는 이전/새 앱 버전과 동시 호환되게 단계적으로 변경한다.
- 배포 실패 시 이전 Task Definition과 이미지가 남아 있어야 한다.

## 시험 포인트

- 빌드/테스트/이미지 push: CodeBuild
- 전체 단계 오케스트레이션: CodePipeline
- ECS Blue/Green 트래픽 전환: CodeDeploy
- Blue/Green의 두 대상: 두 Target Group
- build가 ECR push 거부: CodeBuild Service Role 확인
- Task가 ECR pull 거부: Task Execution Role 확인

## 연습문제

**문제.** 새 ECS 버전을 운영 트래픽 없이 검증하고, 오류 시 기존 버전으로 빠르게 전환해야 한다. 가장 적절한 배포 방식은?

**정답:** ECS Blue/Green 배포를 사용한다. DVA-C02 문맥에서는 CodeDeploy, 두 Target Group, production/test listener 구성을 함께 판단한다.

## 확인 문제

- [ ] 각 CI/CD 서비스의 역할을 설명할 수 있다.
- [ ] Build Role과 Task Execution Role을 구분할 수 있다.
- [ ] Blue/Green에 두 Target Group이 필요한 이유를 안다.

[이전](<./ECS_012_보안 구성.md>) | [다음: 장애 대응 시나리오](<./ECS_014_장애 대응 시나리오.md>)
