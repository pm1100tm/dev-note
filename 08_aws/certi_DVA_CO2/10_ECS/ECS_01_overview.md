# ✍️ ECS Overview

- 컨테이너란?
- ECS란?
- ECS 전체 흐름
- ECS 주요 구성요소

## 컨테이너란?

애플리케이션 실행에 필요한 코드, 런타임, 라이브러리 설정을 하나로 묶은 실행 단위입니다.
컨테이너의 장점은 환경 차이를 줄이는 것입니다.

- local, dev, prd 환경에서 동일하게 실행 가능

## ECS란?

ECS(Elastic Container Service)는 AWS 컨테이너 오케스트레이션 서비스입니다.
공식 설명 기준으로 ECS는 컨테이너를 실행, 중지, 관리하는 확장 가능한 컨테이너 관리 서비스입니다.

ECS는 다음과 같은 일을 합니다.

```text
컨테이너 실행
컨테이너 개수 유지
컨테이너 재시작
배포 관리
로드밸런서 연결
로그 수집 연동
IAM 권한 연결
Auto Scaling 연동
```

## ECS 전체 흐름

ECS의 큰 흐름은 이렇게 보면 됩니다.

```text
Dockerfile 작성
→ Docker Image build
→ ECR에 Image push
→ ECS Task Definition 작성
→ ECS Service 생성
→ ECS가 Task 실행
→ ALB, CloudWatch, IAM 등과 연동
```

실제 AWS 구성으로 보면 다음과 같습니다.

```text
Developer
  ↓
Docker Image
  ↓
Amazon ECR
  ↓
ECS Task Definition
  ↓
ECS Service
  ↓
ECS Task
  ↓
Fargate or EC2
```

## ECS 주요 구성요소

| 구성요소             | 의미                                                  |
| -------------------- | ----------------------------------------------------- |
| Cluster              | ECS 리소스를 묶는 논리적 공간                         |
| Task Definition      | 컨테이너 실행 설계도                                  |
| Task                 | Task Definition으로 실행된 실제 컨테이너 묶음         |
| Service              | Task 개수를 유지하고 배포를 관리하는 단위             |
| Container Definition | Task 안에서 실행할 개별 컨테이너 설정                 |
| Launch Type          | 컨테이너를 Fargate에서 실행할지 EC2에서 실행할지 결정 |

### ECR과 ECS 관계

ECR은 Docker Image 저장소입니다.

ECS는 ECR에 저장된 이미지를 가져와서 컨테이너로 실행합니다.

```text
ECR = 이미지 저장소
ECS = 이미지를 실행하는 서비스
```

```text
1. Spring Boot 앱 Docker Image 생성
2. ECR에 push
3. ECS Task Definition에서 image URI 지정
4. ECS가 해당 이미지를 pull
5. Task 실행
```
