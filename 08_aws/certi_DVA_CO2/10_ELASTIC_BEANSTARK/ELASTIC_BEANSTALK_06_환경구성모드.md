# 🚀 Elastic Beanstalk 환경 구성 모드

Elastic Beanstalk 환경 구성 모드는 애플리케이션을 어떤 인프라 형태로 실행할지 선택하는 방식입니다.

여기서는 두 가지 구성을 비교합니다.

```shell
Single Instance
High Availability with Load Balancer
```

초보자 관점에서는 다음처럼 이해하면 됩니다.

```shell
Single Instance = EC2 1대로 단순하게 실행하는 개발용 구성
High Availability with Load Balancer = 여러 EC2와 Load Balancer를 사용하는 운영용 구성
```

<br>

## 1. 환경 구성 모드가 필요한 이유

- 모든 애플리케이션이 처음부터 운영 수준의 고가용성 구성을 필요로 하지는 않습니다.
- 개발자가 기능을 빠르게 테스트하는 환경에서는 EC2 1대만 있어도 충분할 수 있습니다.
- 하지만 실제 사용자가 접속하는 운영 환경에서는 한 대의 EC2에 장애가 발생해도 서비스가 계속 동작해야 합니다.
- 그래서 Elastic Beanstalk는 목적에 따라 다른 환경 구성을 제공합니다.

| 목적                        | 적합한 구성                          |
| --------------------------- | ------------------------------------ |
| 개발, 테스트, 빠른 실험     | Single Instance                      |
| 운영, 고가용성, 트래픽 분산 | High Availability with Load Balancer |

<br>

## 2. Single Instance

Single Instance는 하나의 EC2 인스턴스에서 애플리케이션을 실행하는 구성입니다.

슬라이드에서는 Single Instance를 개발 환경에 적합한 구성으로 설명합니다.

```shell
Single Instance

사용자
  ↓
Elastic IP
  ↓
EC2 Instance
  ↓
Application
```

구조가 단순하기 때문에 빠르게 만들 수 있고 비용도 상대적으로 낮습니다.

<br>

## 3. Single Instance의 특징

Single Instance의 핵심 특징은 단순함입니다.

- EC2 인스턴스 1대에서 애플리케이션을 실행합니다.
- Load Balancer가 필수로 필요하지 않습니다.
- Auto Scaling을 통한 다중 인스턴스 확장 구성이 아닙니다.
- 개발 환경이나 테스트 환경에 적합합니다.
- 비용이 상대적으로 낮습니다.
- 구조가 단순해서 빠르게 생성하고 삭제할 수 있습니다.

예를 들어 개발자가 Spring Boot API를 빠르게 올려서 동작 여부만 확인하려는 경우
Single Instance 구성이 적합합니다.

```shell
개발자 코드 배포
  ↓
EC2 1대에서 애플리케이션 실행
  ↓
기능 테스트
```

<br>

## 4. Single Instance의 한계

Single Instance는 단순하지만 운영 환경에는 위험할 수 있습니다.

가장 큰 문제는 단일 장애 지점입니다.

```shell
EC2 1대 장애
  ↓
애플리케이션 중단
```

주의할 점은 다음과 같습니다.

- EC2 인스턴스 한 대에 장애가 발생하면 서비스가 중단될 수 있습니다.
- 트래픽이 늘어나도 자동으로 여러 인스턴스에 분산하는 구조가 아닙니다.
- Availability Zone 장애에 취약합니다.
- 운영 환경에서 요구되는 고가용성 기준을 만족하기 어렵습니다.

따라서 Single Instance는 개발이나 테스트 목적에 적합하고, 운영 환경에서는 신중하게 사용해야 합니다.

<br>

## 5. High Availability with Load Balancer

High Availability with Load Balancer는 운영 환경에 적합한 구성입니다.

슬라이드에서는 이 구성을 production에 적합한 방식으로 설명합니다.

기본 구조는 다음과 같습니다.

```shell
사용자
  ↓
Application Load Balancer
  ↓
Auto Scaling Group
  ↓
EC2 Instance 여러 대
```

여러 EC2 인스턴스를 사용하고, Load Balancer가 요청을 분산합니다.

Auto Scaling Group을 통해 인스턴스 수를 조절할 수 있으며, 여러 Availability Zone에
인스턴스를 배치할 수 있습니다.

<br>

## 6. High Availability 구성의 특징

High Availability with Load Balancer의 핵심은 가용성과 확장성입니다.

- Load Balancer가 요청을 여러 EC2 인스턴스로 분산합니다.
- Auto Scaling Group이 EC2 인스턴스 수를 관리합니다.
- 여러 Availability Zone에 인스턴스를 배치할 수 있습니다.
- 특정 EC2 인스턴스에 장애가 발생해도 다른 인스턴스가 요청을 처리할 수 있습니다.
- 운영 환경에 적합합니다.
- 트래픽 증가에 대응하기 쉽습니다.

구조 예시는 다음과 같습니다.

```shell
Application Load Balancer
  ↓
Auto Scaling Group
  ├── AZ 1: EC2 Instance
  └── AZ 2: EC2 Instance
```

이 구성에서는 EC2 한 대에 문제가 생겨도 Load Balancer가 정상 인스턴스로 트래픽을 보낼 수 있습니다.

<br>

## 7. RDS와 함께 보는 구성 차이

슬라이드에서는 Single Instance와 High Availability 구성을 RDS와 함께 보여줍니다.

Single Instance 쪽은 EC2 한 대와 RDS Master가 연결된 단순 구조로 볼 수 있습니다.

```shell
EC2 Instance
  ↓
RDS Master
```

High Availability 구성 쪽은 여러 Availability Zone에 걸친 구성을 보여줍니다.

```shell
ALB
  ↓
Auto Scaling Group
  ├── AZ 1: EC2 Instance
  └── AZ 2: EC2 Instance

RDS Master
RDS Standby
```

운영 환경에서는 애플리케이션 계층뿐 아니라 데이터베이스 계층도 고가용성을 고려해야 합니다.

다만 Elastic Beanstalk에서 RDS를 함께 생성하면 RDS의 생명주기가 Beanstalk Environment와
묶일 수 있으므로 운영 환경에서는 별도 RDS 구성이 더 적합한 경우가 많습니다.

<br>

## 8. Single Instance vs High Availability 비교

| 구분               | Single Instance              | High Availability with Load Balancer |
| ------------------ | ---------------------------- | ------------------------------------ |
| 주요 목적          | 개발, 테스트                 | 운영                                 |
| EC2 수             | 보통 1대                     | 여러 대 가능                         |
| Load Balancer      | 일반적으로 사용하지 않음     | 사용                                 |
| Auto Scaling Group | 제한적 구성                  | 핵심 구성 요소                       |
| Availability Zone  | 보통 단일 AZ                 | 여러 AZ 구성 가능                    |
| 장애 대응          | EC2 장애 시 서비스 중단 위험 | 정상 인스턴스로 트래픽 분산 가능     |
| 비용               | 낮음                         | 상대적으로 높음                      |
| 적합한 상황        | 빠른 실험, 개발 검증         | 실제 사용자 서비스                   |

<br>

## 9. 어떤 구성을 선택해야 합니까?

개발 환경에서는 Single Instance가 적합합니다.

이유는 다음과 같습니다.

- 빠르게 만들 수 있습니다.
- 비용이 낮습니다.
- 구조가 단순합니다.
- 복잡한 고가용성 구성이 필요하지 않습니다.

운영 환경에서는 High Availability with Load Balancer가 적합합니다.

이유는 다음과 같습니다.

- EC2 장애에 대응할 수 있습니다.
- 트래픽을 여러 인스턴스로 분산할 수 있습니다.
- Auto Scaling으로 확장성을 확보할 수 있습니다.
- 여러 Availability Zone을 사용해 가용성을 높일 수 있습니다.

<br>

## 10. 초보자가 헷갈리기 쉬운 부분

### 10.1 Single Instance도 실제 EC2를 사용합니다

Elastic Beanstalk가 관리형 서비스라서 EC2가 없는 것처럼 보일 수 있지만, Single Instance
구성도 내부적으로 EC2 인스턴스를 사용합니다.

<br>

### 10.2 High Availability 구성은 비용이 더 높을 수 있습니다

Load Balancer, 여러 EC2 인스턴스, RDS Multi-AZ 같은 구성을 사용하면 비용이 증가합니다.

따라서 개발 환경까지 무조건 운영 수준으로 만들 필요는 없습니다.

<br>

### 10.3 운영 환경에서 Single Instance는 위험할 수 있습니다

Single Instance는 EC2 1대에 의존하므로 장애에 취약합니다.

실제 사용자 트래픽을 처리하는 운영 환경에서는 Load Balancer와 Auto Scaling Group을 사용하는
구성이 더 적합합니다.

<br>

## 11. DVA-C02 시험 포인트

- Elastic Beanstalk에는 Single Instance 구성과 High Availability with Load Balancer
  구성이 있습니다.
- Single Instance는 개발 환경에 적합합니다.
- Single Instance는 EC2 한 대 중심의 단순 구성입니다.
- High Availability with Load Balancer는 운영 환경에 적합합니다.
- High Availability 구성은 Load Balancer와 Auto Scaling Group을 사용합니다.
- High Availability 구성은 여러 EC2 인스턴스와 여러 Availability Zone을 활용할 수 있습니다.
- 운영 환경에서는 단일 장애 지점을 줄이는 구성이 중요합니다.
- Beanstalk 자체는 무료지만 EC2, Load Balancer, RDS 같은 하위 리소스 비용은 발생합니다.

<br>

## 12. 시험 예상 문제

### 문제 1

개발자가 Elastic Beanstalk에서 빠르게 개발용 애플리케이션을 테스트하려고 합니다.
비용을 낮게 유지하고 단순한 구성을 원합니다. 가장 적절한 환경 구성 모드는 무엇입니까?

- A. Single Instance
- B. High Availability with Load Balancer
- C. Multi-AZ RDS Only
- D. VPC Peering

<br>

### 정답

A. Single Instance

<br>

### 해설

Single Instance는 EC2 한 대 중심의 단순한 Elastic Beanstalk 구성입니다.
슬라이드에서도 Single Instance는 개발 환경에 적합한 구성으로 설명됩니다.
운영 수준의 고가용성이 필요하지 않고 빠른 테스트가 목적이라면 Single Instance가 적합합니다.

<br>

### 문제 2

운영 환경에서 Elastic Beanstalk 애플리케이션을 배포하려고 합니다. EC2 인스턴스 장애에도 서비스가
계속 동작해야 하며, 트래픽을 여러 인스턴스로 분산해야 합니다. 가장 적절한 구성은 무엇입니까?

- A. Single Instance
- B. High Availability with Load Balancer
- C. S3 Static Website Hosting
- D. IAM Role

<br>

### 정답

B. High Availability with Load Balancer

<br>

### 해설

High Availability with Load Balancer 구성은 Load Balancer와 Auto Scaling Group을 사용합니다.
여러 EC2 인스턴스에 트래픽을 분산할 수 있고, 특정 인스턴스에 장애가 발생해도 정상 인스턴스가 요청을 처리할 수 있습니다.
슬라이드에서도 이 구성은 운영 환경에 적합한 방식으로 설명됩니다.

<br>

### 문제 3

Elastic Beanstalk의 Single Instance 구성에 대한 설명으로 가장 올바른 것은 무엇입니까?

- A. 운영 환경의 고가용성을 위해 항상 여러 EC2 인스턴스와 Load Balancer를 사용합니다.
- B. 개발 환경에 적합하며, 보통 EC2 한 대 중심으로 단순하게 구성됩니다.
- C. SQS Queue 메시지 수를 기준으로만 스케일링됩니다.
- D. RDS Standby 인스턴스만 생성하는 데이터베이스 전용 구성입니다.

<br>

### 정답

B. 개발 환경에 적합하며, 보통 EC2 한 대 중심으로 단순하게 구성됩니다.

<br>

### 해설

Single Instance는 이름 그대로 단일 EC2 인스턴스 중심의 단순 구성입니다.
비용이 낮고 빠르게 만들 수 있어 개발이나 테스트 환경에 적합합니다.
다만 EC2 한 대에 의존하므로 운영 환경에서는 장애 대응 측면의 한계가 있습니다.

<br>

### 문제 4

Elastic Beanstalk의 High Availability with Load Balancer 구성에 대한 설명으로 가장 올바른 것은 무엇입니까?

- A. Load Balancer 없이 EC2 한 대만 사용합니다.
- B. 개발 환경에서만 사용할 수 있습니다.
- C. Load Balancer와 Auto Scaling Group을 사용해 운영 환경의 가용성과 확장성을 높입니다.
- D. Elastic Beanstalk에서는 Auto Scaling Group을 사용할 수 없습니다.

<br>

### 정답

C. Load Balancer와 Auto Scaling Group을 사용해 운영 환경의 가용성과 확장성을 높입니다.

<br>

### 해설

High Availability with Load Balancer 구성은 운영 환경에 적합한 Beanstalk 구성입니다.
Load Balancer가 여러 EC2 인스턴스로 요청을 분산하고, Auto Scaling Group이 인스턴스 수를 관리합니다.
여러 Availability Zone을 활용하면 단일 장애 지점을 줄이고 서비스 가용성을 높일 수 있습니다.

<br>

## 요약

- Elastic Beanstalk 환경 구성 모드는 애플리케이션을 어떤 인프라 형태로 실행할지 결정합니다.
- Single Instance는 EC2 한 대 중심의 단순한 개발용 구성입니다.
- High Availability with Load Balancer는 Load Balancer와 Auto Scaling Group을
  사용하는 운영용 구성입니다.
- DVA-C02에서는 `Single Instance = dev`, `High Availability with Load Balancer = prod`로
  먼저 기억하면 됩니다.
