# Elastic BeanStark Overview

Elastic Beanstalk는 한마디로 “AWS 인프라를 직접 하나씩 조립하지 않고, 애플리케이션 코드만 올리면
실행 환경을 자동으로 만 들어주는 배포 서비스”입니다.

예를 들어 웹 서비스를 AWS에 직접 올리려면 보통 이런 것들을 직접 구성해야 합니다.

- EC2
- Auto Scaling Group
- Load Balancer
- Security Group
- RDS
- CloudWatch
- 배포 방식
- 헬스 체크
- 스케일링 정책

Elastic Beanstalk는 이 복잡한 구성을 개발자 관점에서 단순화합니다. 개발자는 애플리케이션 코드를
업로드하고, Beanstalk가 내부적으로 EC2, ASG, ELB, CloudFormation 등을 사용해서 실행 환경을
만들어 줍니다.

중요한 점은 “서버리스”가 아니라는 것입니다. Beanstalk 자체는 무료지만, 내부에서 생성되는 EC2,
Load Balancer, RDS 같은 리소스 비용은 그대로 발생합니다.

## Chapter

- 1. Elastic Beanstalk 개요
- 2. Beanstalk 구성 요소
- 3. 지원 플랫폼
- 4. Web Server Tier와 Worker Tier
- 5. 환경 구성 모드: Single Instance vs High Availability
- 6. 배포 전략
- 7. EB CLI와 배포 프로세스
- 8. Application Version Lifecycle Policy
- 9. .ebextensions와 CloudFormation 내부 동작
- 10. 환경 복제와 Load Balancer 마이그레이션
- 11. RDS 연동과 운영 환경 분리 전략

---

### 1. Elastic Beanstalk 개요

Elastic Beanstalk가 왜 필요한지 설명하는 챕터입니다.

핵심은 개발자가 인프라 관리보다 애플리케이션 실행에 집중하게 해주는 것입니다. Beanstalk는
capacity provisioning, load balancing, scaling, health monitoring,
instance configuration 등을 자동 처리합니다.

시험 포인트:

```shell
Elastic Beanstalk = Managed Service
개발자 책임 = 애플리케이션 코드
AWS/Beanstalk 책임 = 인프라 생성과 운영 자동화
비용 = Beanstalk는 무료, 하위 리소스는 과금
```

### 2. Elastic Beanstalk 구성 요소

Beanstalk의 기본 단위를 이해하는 챕터입니다.

주요 구성은 다음입니다.

- Application
- Application Version
- Environment
- Tier

> Application은 Beanstalk에서 관리하는 전체 앱 단위입니다.

> Application Version은 배포되는 코드의 특정 버전입니다. 예를 들어 v1, v2,
> 2026-09-01-release 같은 개념입니다.

> Environment는 실제 AWS 리소스가 떠 있는 실행 환경입니다. 예를 들어 dev, test, prod를
> 각각 별도 environment로 만들 수 있습니다.

### 3. 지원 플랫폼

Beanstalk가 어떤 런타임을 지원하는지 보는 챕터입니다.

슬라이드 기준으로 Go, Java SE, Tomcat, .NET, Node.js, PHP, Python, Ruby, Docker 등을
지원합니다.

DVA-C02에서는 “Elastic Beanstalk가 다양한 플랫폼을 지원하고, Docker 배포도 가능하다”
정도를 기억하면 됩니다.

### 4. Web Server Tier vs Worker Tier

Beanstalk 환경 유형을 구분하는 챕터입니다.

Web Server Tier는 HTTP 요청을 받는 일반적인 웹 애플리케이션 구조입니다.

```shell
사용자 요청
-> Load Balancer
-> EC2 Web Server
```

Worker Tier는 백그라운드 작업 처리용입니다.

```shell
SQS Queue
-> EC2 Worker
-> 비동기 작업 처리
```

예를 들어 이미지 리사이징, 이메일 발송, 주문 후처리 같은 작업은 Worker Tier에 어울립니다.

시험 포인트:

```shell
Web Tier = HTTP 요청 처리
Worker Tier = SQS 기반 백그라운드 작업 처리
```

### 5. 배포 모드: Single Instance vs High Availability

Beanstalk 환경을 개발용과 운영용으로 나누는 챕터입니다.

Single Instance는 EC2 하나로 구성됩니다. 빠르고 단순해서 개발 환경에 적합합니다.

High Availability with Load Balancer는 Load Balancer와 Auto Scaling Group을
사용합니다. 여러 AZ에 걸쳐 인스턴스를 배치할수 있어 운영 환경에 적합합니다.

시험 포인트:

```shell
Dev/Test = Single Instance 가능
Prod = Load Balancer + Auto Scaling 기반 HA 구성 권장
```

### 6. 애플리케이션 업데이트 배포 전략

Elastic Beanstalk 시험에서 가장 중요한 챕터입니다.

배포 옵션은 다음처럼 나눌 수 있습니다.

```shell
All at once
Rolling
Rolling with additional batches
Immutable
Blue/Green
Traffic Splitting
```

- All at once는 모든 인스턴스를 한 번에 교체합니다.
  가장 빠르지만 다운타임이 있습니다. 개발 환경에 적합합니다.
- Rolling은 일부 인스턴스씩 나눠서 배포합니다. 다운타임은 줄지만, 배포 중에는 전체 용량보다 낮은
  상태로 서비스됩니다.
- Rolling with additional batches는 새 인스턴스를 추가로 띄우면서 배포합니다.
  기존 용량을 유지할 수 있어 운영에 더 적합하지만 추가 비용이 있습니다.
- Immutable은 새 Auto Scaling Group에 새 버전을 배포한 뒤, 정상 확인 후 교체합니다.
  안정적이고 롤백이 쉽지만 비용이 큽니다.
- Blue/Green은 새 환경을 따로 만들고, 검증 후 URL 또는 Route 53을 전환합니다.
  다운타임을 최소화할 수 있습니다.
- Traffic Splitting은 새 버전에 일부 트래픽만 보내는 Canary 방식입니다.
  실패하면 자동 롤백이 빠릅니다.

### 7. EB CLI와 배포 프로세스

CLI로 Beanstalk를 운영하는 방법입니다.

대표 명령어:

```shell
eb create
eb status
eb health
eb events
eb logs
eb open
eb deploy
eb config
eb terminate
```

배포 흐름은 보통 다음입니다.

```shell
의존성 정의
-> 코드 zip 패키징
-> Application Version 생성
-> Environment에 배포
-> EC2에서 의존성 설치 후 애플리케이션 실행
```

Python이면 requirements.txt, Node.js면 package.json 같은 파일이 중요합니다.

### 8. Application Version Lifecycle Policy

Beanstalk의 애플리케이션 버전 관리 챕터입니다.

Elastic Beanstalk는 최대 1000개의 application version을 저장할 수 있습니다.
오래된 버전을 정리하지 않으면 새 배포가 막힐수 있습니다.

Lifecycle Policy는 오래된 버전을 시간 기준 또는 개수 기준으로 삭제합니다.

시험 포인트:

```shell
최대 application version = 1000개
현재 사용 중인 버전은 삭제되지 않음
S3 source bundle 삭제 여부 옵션 존재
```

### 9. .ebextensions와 내부 동작

Beanstalk 환경 설정을 코드로 관리하는 챕터입니다.

.ebextensions/ 디렉터리에 .config 파일을 넣으면 콘솔에서 설정하던 값을 코드로 정의할 수 있습니다.

예:

```shell
환경 변수
인스턴스 설정
로그 설정
RDS, ElastiCache, DynamoDB 같은 추가 리소스
```

내부적으로 Elastic Beanstalk는 CloudFormation을 사용합니다. 즉, Beanstalk가 뒤에서
CloudFormation 스택을 만들어 EC2, ELB, ASG 같은 리소스를 생성합니다.

주의할 점:

> .ebextensions로 만든 리소스는 환경 삭제 시 같이 삭제될 수 있음

### 10. 환경 복제와 마이그레이션

운영 중인 Beanstalk 환경을 복제하거나 구조를 바꾸는 챕터입니다.

Cloning은 기존 환경과 같은 설정의 새 환경을 만드는 기능입니다. 테스트 환경을 만들 때 유용합니다.

하지만 Load Balancer 타입은 환경 생성 후 변경할 수 없습니다. 예를 들어,
Classic Load Balancer에서 Application Load Balancer로 바꾸려면 새 환경을 만들고 배포한 뒤
CNAME swap 또는 Route 53 업데이트를 해야 합니다.

시험 포인트:

```shell
생성 후 Load Balancer 타입 변경 불가
변경하려면 새 환경 생성 후 CNAME swap 또는 Route 53 전환
```

### 11. RDS와 Elastic Beanstalk

Beanstalk에서 RDS를 같이 만들 수 있지만, 운영 환경에서는 주의해야 합니다.

개발/테스트 환경에서는 Beanstalk가 RDS까지 같이 만들어도 괜찮습니다.

하지만 운영 환경에서는 권장되지 않습니다. 이유는 RDS의 lifecycle이 Beanstalk environment
lifecycle에 묶이기 때문입니다. 환경을 삭제할 때 DB까지 영향을 받을 수 있습니다.

운영 Best Practice:

```shell
RDS는 Beanstalk 밖에서 별도로 생성
Beanstalk 애플리케이션에는 connection string만 전달
```

이미 Beanstalk에 묶인 RDS를 분리하려면:

- RDS snapshot 생성
- RDS 삭제 방지 설정
- RDS 없는 새 Beanstalk 환경 생성
- 기존 RDS connection string 연결
- CNAME swap 또는 Route 53 전환
- 기존 환경 종료
- CloudFormation stack 정리
