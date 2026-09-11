# 🚀 .ebextensions와 CloudFormation 내부 동작

Elastic Beanstalk는 콘솔에서 설정한 값을 코드로 관리할 수 있는 방법을 제공합니다.

그 대표 기능이 `.ebextensions`입니다.

초보자 관점에서는 다음처럼 이해하면 됩니다.

```shell
.ebextensions
= Elastic Beanstalk 환경 설정을 코드 파일로 관리하는 디렉터리
```

Elastic Beanstalk는 내부적으로 CloudFormation을 사용해 AWS 리소스를 생성하고 관리합니다.

따라서 `.ebextensions`를 사용하면 Elastic Beanstalk 환경 설정뿐 아니라, 추가 AWS 리소스까지
함께 정의할 수 있습니다.

<br>

## 1. .ebextensions가 필요한 이유

Elastic Beanstalk 환경은 콘솔에서 설정할 수 있습니다.

예를 들어 다음 설정을 콘솔에서 변경할 수 있습니다.

- 환경 변수
- 인스턴스 타입
- Auto Scaling 설정
- Load Balancer 설정
- 로그 설정
- 헬스 체크 설정

하지만 콘솔에서 수동으로 설정하면 문제가 생길 수 있습니다.

- 누가 어떤 값을 바꿨는지 추적하기 어렵습니다.
- dev, test, prod 환경 설정을 일관되게 맞추기 어렵습니다.
- 새 환경을 만들 때 같은 설정을 반복 입력해야 합니다.
- CI/CD 파이프라인에서 자동화하기 어렵습니다.

`.ebextensions`를 사용하면 이런 설정을 코드로 관리할 수 있습니다.

```shell
콘솔 수동 설정
  ↓
.ebextensions 파일로 설정 관리
  ↓
배포 시 Elastic Beanstalk가 설정 적용
```

<br>

## 2. .ebextensions 기본 구조

`.ebextensions` 디렉터리는 애플리케이션 소스 코드의 루트에 위치해야 합니다.

```shell
my-app/
├── app.py
├── requirements.txt
└── .ebextensions/
    └── logging.config
```

Node.js 애플리케이션이라면 다음처럼 구성할 수 있습니다.

```shell
my-node-app/
├── app.js
├── package.json
└── .ebextensions/
    └── options.config
```

중요한 규칙은 다음과 같습니다.

- `.ebextensions` 디렉터리는 소스 코드 루트에 있어야 합니다.
- 설정 파일은 `.config` 확장자를 사용해야 합니다.
- 설정 파일은 YAML 또는 JSON 형식을 사용할 수 있습니다.

<br>

## 3. .config 파일

`.ebextensions` 안에는 `.config` 확장자를 가진 파일을 둡니다.

예시는 다음과 같습니다.

```shell
.ebextensions/
├── logging.config
├── options.config
└── resources.config
```

이 파일들은 Elastic Beanstalk 환경을 구성할 때 읽힙니다.

파일 안에는 환경 옵션, 리소스 정의, 패키지 설치, 명령 실행 같은 설정을 넣을 수 있습니다.

<br>

## 4. option_settings

`option_settings`는 Elastic Beanstalk의 기본 설정 값을 코드로 지정할 때 사용합니다.

예를 들어 콘솔에서 설정할 수 있는 여러 옵션을 `.config` 파일에 정의할 수 있습니다.

개념 예시는 다음과 같습니다.

```yaml
option_settings:
  aws:elasticbeanstalk:application:environment:
    ENV: prod
    LOG_LEVEL: info
```

위 설정은 애플리케이션 환경 변수처럼 사용할 수 있습니다.

다른 예시로 인스턴스 타입 같은 설정도 코드로 관리할 수 있습니다.

```yaml
option_settings:
  aws:autoscaling:launchconfiguration:
    InstanceType: t3.micro
```

정확한 namespace와 옵션 이름은 설정하려는 Elastic Beanstalk 기능에 따라 달라집니다.

시험에서는 세부 namespace를 모두 외우기보다, `option_settings`로 Elastic Beanstalk
기본 설정을 코드화할 수 있음을 기억하는 편이 중요합니다.

<br>

## 5. 추가 AWS 리소스 정의

`.ebextensions`는 단순 환경 설정뿐 아니라 AWS 리소스 정의에도 사용할 수 있습니다.

예를 들어 다음 리소스를 함께 정의할 수 있습니다.

- RDS
- ElastiCache
- DynamoDB
- S3 Bucket
- Security Group
- IAM Role

개념적으로는 다음과 같습니다.

```yaml
Resources:
  MyBucket:
    Type: AWS::S3::Bucket
```

이 구조는 CloudFormation 리소스 정의와 유사합니다.

Elastic Beanstalk가 내부적으로 CloudFormation을 사용하기 때문에, `.ebextensions`를 통해
추가 리소스를 생성할 수 있습니다.

<br>

## 6. Elastic Beanstalk 내부 동작

Elastic Beanstalk는 겉으로 보기에는 단순한 배포 서비스처럼 보입니다.

하지만 내부에서는 CloudFormation을 사용해 여러 AWS 리소스를 생성합니다.

```shell
Elastic Beanstalk
  ↓
CloudFormation Stack
  ↓
EC2 / Auto Scaling Group / Load Balancer / Security Group / 기타 리소스
```

즉, 사용자가 Elastic Beanstalk Environment를 만들면 내부적으로 CloudFormation Stack이
생성됩니다.

이 Stack이 EC2, Auto Scaling Group, Load Balancer 같은 리소스를 프로비저닝합니다.

<br>

## 7. CloudFormation과의 관계

CloudFormation은 AWS 리소스를 코드로 정의하고 생성하는 IaC 서비스입니다.

Elastic Beanstalk는 이 CloudFormation을 내부 엔진처럼 사용합니다.

```shell
사용자: Elastic Beanstalk Environment 생성
  ↓
Elastic Beanstalk: CloudFormation Stack 생성
  ↓
CloudFormation: AWS 리소스 프로비저닝
```

그래서 `.ebextensions`에 리소스를 정의하면 Elastic Beanstalk가 CloudFormation Stack에
해당 리소스를 포함해 생성할 수 있습니다.

이 점은 운영에서 매우 중요합니다.

Elastic Beanstalk 콘솔에서는 단순하게 보이지만, 실제 리소스 생명주기는 CloudFormation Stack과
연결될 수 있기 때문입니다.

<br>

## 8. 환경 삭제 시 주의할 점

`.ebextensions`로 생성한 리소스는 Elastic Beanstalk Environment와 함께 관리될 수 있습니다.

따라서 Environment를 삭제하면 `.ebextensions`로 생성한 리소스도 함께 삭제될 수 있습니다.

```shell
.ebextensions로 RDS 생성
  ↓
Elastic Beanstalk Environment 삭제
  ↓
RDS도 삭제될 위험
```

이 부분은 운영 환경에서 매우 중요합니다.

특히 데이터베이스, 캐시, 스토리지처럼 데이터가 남아야 하는 리소스를 `.ebextensions`로 생성할 때는
신중해야 합니다.

운영 환경에서는 보통 다음 접근이 더 안전합니다.

```shell
RDS / ElastiCache / S3 같은 중요한 리소스는 별도로 생성
  ↓
Elastic Beanstalk에는 연결 정보만 환경 변수로 전달
```

<br>

## 9. 운영 환경에서의 권장 방향

`.ebextensions`는 편리하지만 모든 리소스를 여기에 넣는 방식이 항상 좋은 선택은 아닙니다.

운영 환경에서는 리소스의 생명주기를 분리하는 것이 중요합니다.

| 리소스 유형                                  | 권장 방향                                  |
| -------------------------------------------- | ------------------------------------------ |
| 환경 변수, 로그 설정, 인스턴스 옵션          | `.ebextensions` 사용 가능                  |
| 애플리케이션 실행에 필요한 간단한 설정       | `.ebextensions` 사용 가능                  |
| RDS, ElastiCache, S3 같은 중요 데이터 리소스 | 별도 생성 후 연결 정보 전달 권장           |
| 장기 보존이 필요한 리소스                    | Beanstalk Environment와 생명주기 분리 권장 |

핵심은 Environment를 삭제해도 보존되어야 하는 리소스인지 먼저 판단하는 것입니다.

<br>

## 10. .ebextensions 사용 예시 흐름

`.ebextensions`를 포함한 배포 흐름은 다음과 같습니다.

```shell
애플리케이션 코드 작성
  ↓
.ebextensions 디렉터리 생성
  ↓
.config 파일 작성
  ↓
코드를 zip으로 패키징
  ↓
Elastic Beanstalk에 배포
  ↓
Elastic Beanstalk가 .config 파일을 읽고 설정 적용
  ↓
CloudFormation을 통해 필요한 리소스 생성 또는 설정 변경
```

예를 들어 환경 변수를 코드로 관리하면 새 Environment를 만들 때도 같은 설정을 반복 적용할 수 있습니다.

```yaml
option_settings:
  aws:elasticbeanstalk:application:environment:
    APP_ENV: production
    LOG_LEVEL: info
```

<br>

## 11. 초보자가 헷갈리기 쉬운 부분

### 11.1 .ebextensions는 소스 코드 루트에 있어야 합니다

`.ebextensions` 디렉터리가 잘못된 위치에 있으면 Elastic Beanstalk가 설정 파일을 읽지 못할 수 있습니다.

```shell
올바른 위치:
my-app/.ebextensions/options.config

잘못된 위치:
my-app/src/.ebextensions/options.config
```

<br>

### 11.2 파일 확장자는 .config여야 합니다

`.ebextensions` 안의 설정 파일은 `.config` 확장자를 사용해야 합니다.

예를 들어 `logging.config`는 올바른 예시입니다.

```shell
.ebextensions/logging.config
```

<br>

### 11.3 YAML 들여쓰기가 중요합니다

YAML 형식은 들여쓰기에 민감합니다.

들여쓰기가 잘못되면 배포 중 설정 적용에 실패할 수 있습니다.

운영 환경에서는 배포 전에 설정 파일 문법을 반드시 확인해야 합니다.

<br>

### 11.4 중요한 리소스를 Environment 생명주기에 묶지 않도록 주의해야 합니다

`.ebextensions`로 만든 리소스는 Environment 삭제 시 함께 삭제될 수 있습니다.

RDS 같은 데이터 리소스는 운영 환경에서 별도로 관리하는 편이 안전합니다.

<br>

## 12. DVA-C02 시험 포인트

- `.ebextensions`는 Elastic Beanstalk 환경 설정을 코드로 관리하는 기능입니다.
- `.ebextensions` 디렉터리는 소스 코드 루트에 있어야 합니다.
- 설정 파일은 `.config` 확장자를 사용합니다.
- `.config` 파일은 YAML 또는 JSON 형식을 사용할 수 있습니다.
- `option_settings`를 사용해 Elastic Beanstalk 기본 설정을 변경할 수 있습니다.
- `.ebextensions`로 RDS, ElastiCache, DynamoDB, S3 같은 AWS 리소스를 정의할 수 있습니다.
- Elastic Beanstalk는 내부적으로 CloudFormation을 사용합니다.
- `.ebextensions`에 정의한 리소스는 CloudFormation Stack을 통해 생성될 수 있습니다.
- `.ebextensions`로 관리되는 리소스는 Environment 삭제 시 함께 삭제될 수 있습니다.
- 운영 환경에서는 RDS 같은 중요 리소스를 Beanstalk Environment와 분리하는 편이 안전합니다.

<br>

## 13. 시험 예상 문제

### 문제 1

Elastic Beanstalk 환경에서 콘솔 설정을 코드로 관리하려고 합니다. 소스 코드 루트에 어떤 디렉터리를 두어야 합니까?

- A. `.ebextensions`
- B. `.elasticcache`
- C. `.cloudwatch`
- D. `.route53`

<br>

### 정답

A. `.ebextensions`

<br>

### 해설

`.ebextensions`는 Elastic Beanstalk 환경 설정을 코드 파일로 관리하기 위한 디렉터리입니다.
이 디렉터리는 애플리케이션 소스 코드 루트에 위치해야 하며, 내부에 `.config` 파일을 둘 수 있습니다.

<br>

### 문제 2

Elastic Beanstalk의 `.ebextensions` 설정 파일에 대한 설명으로 가장 올바른 것은 무엇입니까?

- A. `.config` 확장자를 사용하며 YAML 또는 JSON 형식을 사용할 수 있습니다.
- B. 반드시 `.txt` 확장자를 사용해야 합니다.
- C. 반드시 애플리케이션 실행 후 EC2에 직접 생성해야 합니다.
- D. Elastic Beanstalk에서는 환경 설정을 코드로 관리할 수 없습니다.

<br>

### 정답

A. `.config` 확장자를 사용하며 YAML 또는 JSON 형식을 사용할 수 있습니다.

<br>

### 해설

`.ebextensions` 안의 설정 파일은 `.config` 확장자를 사용합니다.
형식은 YAML 또는 JSON을 사용할 수 있습니다.
이 파일을 통해 Elastic Beanstalk 환경 설정을 코드로 관리할 수 있습니다.

<br>

### 문제 3

Elastic Beanstalk가 내부적으로 AWS 리소스를 프로비저닝할 때 주로 의존하는 서비스는 무엇입니까?

- A. AWS CloudFormation
- B. Amazon Route 53
- C. AWS IAM Identity Center
- D. Amazon CloudFront

<br>

### 정답

A. AWS CloudFormation

<br>

### 해설

Elastic Beanstalk는 내부적으로 CloudFormation을 사용해 EC2, Auto Scaling Group,
Load Balancer 같은 AWS 리소스를 생성하고 관리합니다.
`.ebextensions`에 추가 AWS 리소스를 정의하면 CloudFormation Stack을 통해 함께 생성될 수 있습니다.

<br>

### 문제 4

운영 환경에서 `.ebextensions`로 RDS를 생성했습니다. 이후 Elastic Beanstalk Environment를
삭제하려고 합니다. 가장 주의해야 할 점은 무엇입니까?

A. `.ebextensions`로 생성한 리소스가 Environment 삭제와 함께 삭제될 수 있습니다.
B. RDS는 항상 자동으로 별도 계정으로 이전됩니다.
C. CloudFormation은 Elastic Beanstalk와 아무 관련이 없습니다.
D. `.ebextensions`는 로그 설정에만 사용할 수 있습니다.

<br>

### 정답

A. `.ebextensions`로 생성한 리소스가 Environment 삭제와 함께 삭제될 수 있습니다.

<br>

### 해설

`.ebextensions`로 생성한 리소스는 Elastic Beanstalk Environment 생명주기와 연결될 수 있습니다.
Environment를 삭제하면 해당 리소스도 함께 삭제될 수 있으므로, RDS 같은 중요 데이터 리소스는
운영 환경에서 별도로 생성하고 연결 정보만 전달하는 방식이 더 안전합니다.

<br>

## 요약

- `.ebextensions`는 Elastic Beanstalk 환경 설정을 코드로 관리하기 위한 기능입니다.
- 소스 코드 루트에 `.ebextensions` 디렉터리를 만들고, `.config` 파일에 YAML 또는 JSON 형식으로 설정을 작성합니다.
- Elastic Beanstalk는 내부적으로 CloudFormation을 사용해 AWS 리소스를 생성하므로,
  `.ebextensions`를 통해 추가 리소스도 정의할 수 있습니다.
- 다만 `.ebextensions`로 생성한 리소스는 Environment 삭제 시 함께 삭제될 수 있으므로,
  운영 환경에서는 RDS 같은 중요 리소스의 생명주기 분리를 반드시 고려해야 합니다.
