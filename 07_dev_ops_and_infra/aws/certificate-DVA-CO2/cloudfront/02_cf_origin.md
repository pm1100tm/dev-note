# Amazon CloudFront Origin

CloudFront는 여러 종류의 Origin과 연결할 수 있습니다.

| Origin              | 사용 예시                                         |
| ------------------- | ------------------------------------------------- |
| S3 Bucket           | 이미지, 정적 웹사이트, 파일 다운로드              |
| S3 Website Endpoint | S3 정적 웹사이트 호스팅 엔드포인트                |
| ALB                 | Spring Boot, Django, Node.js 같은 웹 애플리케이션 |
| NLB                 | TCP 기반 애플리케이션                             |
| EC2                 | 직접 운영하는 웹 서버                             |
| API Gateway         | API 응답 캐싱 및 글로벌 배포                      |
| MediaPackage        | 영상 스트리밍                                     |
| 외부 HTTP 서버      | AWS 외부의 서버도 Origin으로 사용 가능            |

시험에서는 Origin 유형을 아래처럼 구분하면 됩니다.

| 구분 | 핵심 |
| --- | --- |
| S3 Bucket Origin | 파일 배포, Edge 캐싱, OAC로 private bucket 보호 |
| S3 Website Origin | S3 정적 웹사이트 호스팅을 먼저 활성화해야 함 |
| VPC Origin | private subnet의 ALB, NLB, EC2를 인터넷에 직접 공개하지 않고 연결 |
| Custom HTTP Origin | ALB, EC2, 외부 HTTP 서버처럼 HTTP/HTTPS로 접근하는 Origin |

---

## S3 + CloudFront 예시

정적 파일 배포에서 가장 자주 나오는 구조입니다.

```
사용자
  ↓
Route 53
  ↓
CloudFront
  ↓
S3 Bucket
```

예를 들어 React 빌드 파일을 S3에 올리고 CloudFront로 배포할 수 있습니다.

```
S3 Bucket
- index.html
- main.js
- style.css
- logo.png
```

사용자는 S3 URL이 아니라 CloudFront URL로 접근합니다.

```
https://d123456abcdef.cloudfront.net
```

또는 Route 53 Alias를 연결해서 커스텀 도메인으로 접근할 수 있습니다.

```
https://www.example.com
```

## CloudFront vs S3 직접 접근

| 구분                | S3 직접 접근             | CloudFront 사용                        |
| ------------------- | ------------------------ | -------------------------------------- |
| 속도                | S3 리전에 따라 지연 발생 | 사용자와 가까운 Edge Location에서 응답 |
| 캐싱                | 제한적                   | 강력한 CDN 캐싱                        |
| 보안                | Public Bucket 위험       | OAC로 S3 비공개 가능                   |
| DDoS 보호           | 상대적으로 약함          | AWS Shield, AWS WAF 연동 가능          |
| 커스텀 도메인 HTTPS | 설정이 비교적 복잡       | CloudFront + ACM으로 쉽게 구성         |
| 글로벌 서비스       | 부적합할 수 있음         | 매우 적합                              |

## S3 Origin 보안: OAC

S3를 Origin으로 사용할 때는 S3 버킷을 public으로 열지 않고, CloudFront의 OAC(Origin Access Control)를
통해 접근시키는 구성이 권장됩니다.

```
User
  |
  v
CloudFront
  |
  v
OAC
  |
  v
Private S3 Bucket
```

핵심은 S3 bucket policy에서 CloudFront distribution의 접근만 허용하는 것입니다.

| 상황 | 선택 |
| --- | --- |
| S3 파일을 전 세계 사용자에게 빠르게 배포 | S3 + CloudFront |
| S3 버킷을 private으로 유지 | CloudFront OAC |
| S3 Website Endpoint를 Origin으로 사용 | S3 정적 웹사이트 호스팅 활성화 필요 |

## ALB / EC2 Origin

ALB 또는 EC2를 Origin으로 사용할 수 있습니다.

Public network로 연결하는 경우 Origin은 CloudFront Edge Location에서 오는 요청을 받을 수 있어야 합니다.

```
User
  |
  v
CloudFront Edge Location
  |
  v
Public ALB
  |
  v
Private EC2
```

이 구조에서는 ALB는 public이어야 하고, EC2는 private subnet에 둘 수 있습니다.
ALB Security Group은 CloudFront에서 오는 트래픽을 허용해야 합니다.

EC2를 직접 Origin으로 두고 public network로 연결한다면 EC2 자체가 public 접근 가능해야 합니다.

## VPC Origin

VPC Origin을 사용하면 private subnet에 있는 애플리케이션을 CloudFront Origin으로 사용할 수 있습니다.

대상은 다음과 같습니다.

- Application Load Balancer
- Network Load Balancer
- EC2 Instance

```
User
  |
  v
CloudFront
  |
  v
VPC Origin
  |
  v
Private ALB / NLB / EC2
```

시험 포인트는 "Origin을 인터넷에 직접 노출하지 않고 CloudFront로 전달하고 싶다"는 요구사항입니다.
