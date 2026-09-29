# ECS Fargate VS EC2

많은 사람들이 “Fargate = EC2보다 좋은 것” 이라고 생각하는데 절대 아닙니다.
둘은 장단점이 명확하며, 실제 회사에서는 오히려 EC2 타입을 사용하는 곳도 매우 많습니다.

## EC2 Launch Type

```
AWS
│
├── EC2
│      ├── Docker
│      ├── Docker
│      └── Docker
```

EC2 서버를 내가 직접 관리합니다.

- EC2 생성
- CPU 선택
- Memory 선택
- Disk 선택
- Docker 설치
- Patch
- Scaling

모두 내가 합니다.

## Fargate Launch Type

```
AWS
│
├── Fargate
│      ├── Container
│      ├── Container
│      └── Container
```

EC2가 없습니다. 정확히 말하면 EC2는 존재하지만 AWS가 관리하고 우리는 볼 수도 없습니다.
우리는 “컨테이너 하나 실행해주세요.” 라고만 요청하면 됩니다.

---

## Fargate의 장점

AWS가 대신 해주는 것이 엄청 많습니다.

서버 관리 없음

```
# 기존
EC2
↓
OS 업데이트
↓
Docker 설치
↓
보안 패치
↓
용량 관리

# Fargate
Container 실행
끝
```

### Auto Scaling

- Container 수만 늘리면 됩니다.

### 장애 처리

- 서버 장애도 AWS가 처리합니다.

### 보안

- EC2에 SSH 접속할 필요가 없습니다.
- 관리할 서버 자체가 없습니다.

---

## 그런데 왜 다들 EC2를 사용할까?

바로 한계 때문입니다.

### 1. 비용이 비싸다 (가장 큰 이유)

가장 큰 단점입니다.

예를 들어

```
CPU 1개
RAM 2GB
```

Container 하나 실행한다고 합시다.

Fargate는

- 실행한 만큼
- 초 단위 과금

반면 EC2는

- m7i.large 한 대
- 그 안에, Container 20개 실행

가능합니다.

즉, Container가 많아질수록 EC2가 훨씬 저렴합니다.

- 실무에서는 Container 5개라면 -> Fargate 가 유리합니다.
- 30개라면 -> EC2

### 2. GPU 사용이 어렵다

AI 서버를 만든다고 해봅시다.

```
PyTorch
CUDA
TensorFlow
```

- GPU가 필요합니다.
- Fargate는 GPU 지원이 매우 제한적이어서 일반적인 GPU 워크로드에는 적합하지 않습니다.
- 그래서, AI 서비스는 대부분 EC2 g5 p5 g6 같은 GPU 인스턴스를 사용합니다.

### 3. 특수한 네트워크 구성이 어렵다

예를 들어,

```
Host Network
Bridge
Custom Network
iptables
```

처럼, Docker의 네트워크를 마음대로 건드리고 싶습니다.

Fargate는, AWS가 관리하는 환경이라 제약이 많습니다.

### 4. Docker Daemon 접근 불가

EC2에서는

```
docker ps
docker logs
docker exec
```

등을 마음대로 할 수 있습니다.

Fargate에서는 불가능합니다.
왜냐하면, Docker 서버 자체가 없기 때문입니다.

### 5. Host Volume 사용이 제한적

EC2에서는

```
/data
/log
/cache
```

같은 Host Disk를 Container에 연결할 수 있습니다.

Fargate는 Host 자체가 없으므로 이런 구성이 불가능합니다. 대신 EFS 같은 네트워크 스토리지를 사용해야 합니다.

### 6. Daemon 형태 프로그램 실행이 어렵다

예를 들어, EC2마다

```
CloudWatch Agent
Fluent Bit
Node Exporter
Datadog Agent
```

를 항상 실행하고 싶습니다. EC2에서는 쉽습니다.

하지만, Fargate는 Host가 없으므로 이런 구성이 어렵습니다.

### 7. 세밀한 성능 튜닝 불가

EC2에서는

```
Huge Pages
NUMA
Kernel
CPU Pinning
```

등을 조정할 수 있습니다. Fargate에서는 거의 불가능합니다.

### 8. 컨테이너 실행 옵션 제한

Docker에는

```
--privileged
--device
--pid=host
```

같은 옵션들이 있습니다. Fargate에서는 보안상 이유로 많은 옵션이 제한됩니다.

---

## 그래서 언제 Fargate를 쓰면 안 될까?

### 다음과 같은 경우에는 EC2가 더 적합합니다.

| 상황                               | 이유                                                          |
| ---------------------------------- | ------------------------------------------------------------- |
| 컨테이너가 수백~수천 개            | EC2가 비용 효율적                                             |
| GPU가 필요한 AI 서비스             | GPU 지원 및 성능 활용이 가능                                  |
| Docker를 자유롭게 제어해야 함      | Docker Daemon에 직접 접근할 수 없음                           |
| Host Disk를 사용해야 함            | 로컬 디스크(Host Volume) 사용이 제한됨                        |
| 특수 네트워크 구성이 필요          | Docker 네트워크 및 Host Network 구성에 제약이 있음            |
| 시스템 모니터링 Agent 설치         | Host 수준의 Agent(CloudWatch Agent, Datadog 등) 운영이 어려움 |
| 커널 튜닝이 필요                   | OS 및 Kernel을 직접 제어할 수 없음                            |
| 장시간 항상 실행되는 서비스가 많음 | 장기적으로 EC2가 비용 측면에서 더 유리                        |

### 반대로 Fargate를 쓰면 좋은 경우

| 상황                       | 이유                                                     |
| -------------------------- | -------------------------------------------------------- |
| 스타트업 초기 서비스       | 서버 관리 부담이 적고 빠르게 서비스를 시작할 수 있음     |
| 개발/테스트 환경           | 컨테이너를 빠르게 생성하고 삭제할 수 있음                |
| 트래픽 변동이 큰 서비스    | 필요에 따라 Task를 자동으로 확장(Auto Scaling)할 수 있음 |
| 마이크로서비스(MSA)        | 서비스별로 독립적으로 배포 및 운영하기 쉬움              |
| 운영 인력이 적은 팀        | 서버 관리 없이 애플리케이션 개발에 집중할 수 있음        |
| 배치 작업·이벤트 기반 작업 | 실행한 시간만큼만 과금되어 비용 효율적                   |
