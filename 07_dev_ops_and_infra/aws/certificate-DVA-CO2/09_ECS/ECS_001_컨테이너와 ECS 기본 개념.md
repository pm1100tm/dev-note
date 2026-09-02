# 컨테이너와 ECS 기본 개념

## 전체 목차

1. [컨테이너와 ECS 기본 개념](<./ECS_001_컨테이너와 ECS 기본 개념.md>)
2. [ECS 구성요소](<./ECS_002_ECS 구성요소.md>)
3. [Fargate와 EC2 Launch Type](<./ECS_003_Fargate와 EC2 Launch Type.md>)
4. [Task Definition](<./ECS_004_Task Definition.md>)
5. [ECS Networking](<./ECS_005_ECS Networking.md>)
6. [IAM 권한](<./ECS_006_IAM 권한.md>)
7. [ECR 연동](<./ECS_007_ECR 연동.md>)
8. [ECS Service와 배포](<./ECS_008_ECS Service와 배포.md>)
9. [Load Balancer 연동](<./ECS_009_Load Balancer 연동.md>)
10. [Auto Scaling](<./ECS_010_Auto Scaling.md>)
11. [로그와 모니터링](<./ECS_011_로그와 모니터링.md>)
12. [보안 구성](<./ECS_012_보안 구성.md>)
13. [CI CD 배포](<./ECS_013_CI CD 배포.md>)
14. [장애 대응 시나리오](<./ECS_014_장애 대응 시나리오.md>)
15. [비용과 운영 최적화](<./ECS_015_비용과 운영 최적화.md>)

## 학습 목표

- 컨테이너, 이미지, 레지스트리의 차이를 설명한다.
- Amazon ECS와 Amazon ECR의 역할을 구분한다.
- 이미지 빌드부터 ECS Task 실행까지의 흐름을 이해한다.

> 이 시리즈의 연습문제는 유출 문항을 복원한 것이 아니라, DVA-C02에서 평가하는 AWS 공식 개념과 대표 시나리오를 바탕으로 구성했다. 서비스 사양은 변경될 수 있으므로 시험 직전에는 AWS 공식 문서와 시험 안내서를 함께 확인한다.

## 1. 컨테이너란

컨테이너는 애플리케이션 코드와 실행에 필요한 런타임, 라이브러리, 설정을 함께 묶은 실행 단위이다. 같은 이미지를 사용하면 개발 PC와 운영 환경의 차이를 줄일 수 있다.

| 개념 | 쉬운 설명 |
| --- | --- |
| Dockerfile | 이미지를 만드는 절차서 |
| Image | 실행 파일과 의존성을 담은 변경 불가능한 패키지 |
| Container | 이미지를 실제로 실행한 프로세스 |
| Registry | 이미지를 저장하고 배포하는 저장소 |

이미지는 설계도이고 컨테이너는 그 설계도로 만든 실행 인스턴스라고 생각하면 쉽다. 하나의 이미지로 여러 컨테이너를 실행할 수 있다.

## 2. 컨테이너와 가상 머신

가상 머신은 각각 게스트 운영체제를 포함한다. 컨테이너는 호스트 운영체제의 커널을 공유하므로 일반적으로 시작이 빠르고 더 작은 단위로 배포할 수 있다. 다만 컨테이너가 곧 완전한 보안 경계를 의미하는 것은 아니다.

## 3. ECS와 ECR

- **Amazon ECS**: 컨테이너를 배치하고 실행 상태와 개수를 관리하는 오케스트레이션 서비스
- **Amazon ECR**: 컨테이너 이미지를 저장하는 AWS 관리형 레지스트리
- **AWS Fargate**: ECS Task를 서버 관리 없이 실행할 수 있는 컴퓨팅 방식

```text
Dockerfile
  -> docker build
Container Image
  -> docker push
Amazon ECR
  -> Task Definition의 image URI
Amazon ECS
  -> Fargate 또는 EC2에서 실행
Running Task
```

ECR은 이미지를 실행하지 않고, ECS는 일반적으로 이미지를 장기 보관하지 않는다. `저장=ECR`, `실행 및 관리=ECS`로 구분한다.

## 4. ECS가 관리하는 것

ECS Service를 사용하면 다음 작업을 자동화할 수 있다.

- 원하는 Task 개수 유지
- 실패한 Task 교체
- 새 Task Definition revision 배포
- 로드 밸런서 대상 등록과 해제
- Service Auto Scaling 연동
- CloudWatch 및 EventBridge와 상태 연동

## 실무 적용

웹 API를 컨테이너로 배포한다고 가정한다.

1. CI에서 애플리케이션을 테스트하고 이미지를 빌드한다.
2. 버전 태그와 이미지 digest를 가진 이미지를 ECR에 push한다.
3. 새 이미지 URI로 Task Definition revision을 등록한다.
4. ECS Service를 업데이트한다.
5. 새 Task의 health check와 로그를 확인한다.

운영에서는 `latest` 태그만 쓰기보다 커밋 SHA나 릴리스 버전처럼 추적 가능한 태그를 사용한다. 재현성과 롤백 판단이 쉬워진다.

## 시험 포인트

- 여러 컨테이너를 원하는 개수로 계속 실행: **ECS Service**
- 컨테이너 이미지 저장: **ECR**
- EC2 인스턴스 관리 없이 컨테이너 실행: **Fargate**
- 실행할 이미지, CPU, 메모리, 포트 정의: **Task Definition**
- Kubernetes가 명시되면 ECS가 아니라 **Amazon EKS**를 검토한다.

## 연습문제

**문제 1.** 개발팀은 Docker 이미지를 AWS의 비공개 저장소에 보관하고 ECS에서 가져와 실행하려 한다. 가장 적절한 서비스는?

A. Amazon S3  
B. Amazon ECR  
C. Amazon EFS  
D. AWS CodeArtifact

**정답: B**

ECR은 OCI/Docker 컨테이너 이미지를 위한 관리형 레지스트리다. S3와 EFS는 일반 파일 저장소이며 CodeArtifact는 패키지 저장소다.

**문제 2.** 운영팀이 EC2 인스턴스의 패치와 클러스터 용량 관리를 피하면서 ECS Task를 실행하려 한다. 적절한 선택은?

**정답: AWS Fargate**

Fargate에서는 Task에 필요한 CPU와 메모리를 지정하며 기반 서버를 직접 관리하지 않는다.

## 확인 문제

- [ ] 이미지와 컨테이너를 구분할 수 있다.
- [ ] ECS, ECR, Fargate의 역할을 한 문장씩 설명할 수 있다.
- [ ] 이미지 빌드부터 Task 실행까지의 순서를 설명할 수 있다.

[다음: ECS 구성요소](<./ECS_002_ECS 구성요소.md>)
