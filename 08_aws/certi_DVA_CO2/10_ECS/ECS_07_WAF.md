# AWS WAF와 ECS 웹 애플리케이션 보호

AWS WAF는 웹 애플리케이션으로 들어오는 HTTP(S) 요청을 검사하고 규칙에 따라 허용하거나 차단하는
웹 애플리케이션 방화벽입니다.

ECS 서비스 자체에 직접 연결하는 것이 아니라, ECS 서비스 앞단의 Application Load Balancer(ALB)
또는 CloudFront에 Web ACL을 연결해 사용하는 경우가 일반적입니다.

## AWS WAF가 검사하는 것

AWS WAF는 HTTP(S) 요청의 애플리케이션 계층 정보를 기준으로 검사합니다.

예를 들어 다음 조건을 규칙으로 만들 수 있습니다.

- 요청 IP 주소나 국가
- URI 경로, 쿼리 문자열, HTTP 헤더, 쿠키, 요청 본문
- SQL injection, XSS처럼 알려진 공격 패턴
- 특정 IP에서 짧은 시간에 발생하는 과도한 요청 수

규칙이 일치하면 요청을 `Allow`, `Block`, `Count` 처리하거나
CAPTCHA 및 브라우저 Challenge를 수행할 수 있습니다.

`Count`는 실제 차단 전에 정상 요청까지 막히지 않는지 관찰할 때 유용합니다.

## 핵심 구성 요소

Web ACL은 어떤 요청을 검사하고 어떻게 처리할지 정의하는 단위입니다.

ALB 같은 보호 대상 리소스에 Web ACL을 연결하고, 그 안에 규칙 또는
규칙 그룹을 우선순위와 함께 추가합니다.

- Web ACL: 기본 동작과 규칙 집합을 정의하고 보호 대상에 연결합니다.
- Rule: 요청 조건과 처리 동작을 정의합니다.
- Rule group: 여러 규칙을 재사용 가능한 그룹으로 묶습니다.
- AWS Managed Rules: AWS가 관리하는 알려진 공격 패턴 규칙 그룹입니다.
- Rate-based rule: 일정 시간 동안의 요청 수를 기준으로 과도한
  요청을 제한합니다.

규칙은 숫자가 작은 우선순위부터 평가합니다. `Allow` 또는 `Block`이 일치하면 뒤의 규칙은 평가하지 않으므로,
예외 허용 규칙과 차단 규칙의 순서를 신중하게 정해야 합니다.

## 보안 그룹과의 차이

AWS WAF와 보안 그룹은 서로 대체하는 기능이 아닙니다.

보호하는 계층과 판단 기준이 다르므로 함께 사용합니다.

| 구분      | AWS WAF                                     | 보안 그룹(Security Group)                   |
| --------- | ------------------------------------------- | ------------------------------------------- |
| 보호 계층 | HTTP(S) 애플리케이션 계층                   | VPC 네트워크 연결                           |
| 판단 기준 | URL, 헤더, 본문, 쿠키, 요청 빈도 등         | IP/CIDR, 보안 그룹, 프로토콜, 포트          |
| 연결 대상 | ALB, CloudFront, API Gateway 등             | ENI를 가진 EC2, ALB, ECS 태스크 등          |
| 대표 용도 | SQL injection, XSS, 봇, 과도한 웹 요청 차단 | 443 포트 공개, ALB에서 태스크 포트만 허용   |
| 상태 추적 | 웹 요청 규칙에 따라 처리                    | Stateful 방화벽으로 응답 트래픽을 자동 허용 |

보안 그룹은 `TCP 443` 포트에 연결할 수 있는지 판단하지만, 그 안의
`/login` 요청에 SQL injection 문자열이 있는지는 판단하지 못합니다.

반대로 WAF는 HTTP 요청의 내용을 검사하지만, VPC 내부에서 데이터베이스
포트가 열려 있는지 제어하지는 않습니다.

## ECS와 함께 사용하는 흐름

인터넷에 공개하는 ECS 웹 서비스는 다음처럼 역할을 나눌 수 있습니다.

```text
클라이언트
  -> AWS WAF가 연결된 ALB
  -> ALB 보안 그룹
  -> ECS 태스크 보안 그룹
  -> 컨테이너 애플리케이션
```

- ALB 보안 그룹은 인터넷에서 HTTPS(443) 접근을 허용합니다.
- 태스크 보안 그룹은 인터넷 전체가 아니라 ALB 보안 그룹에서 오는 애플리케이션 포트(예: TCP 8080)만 허용합니다.
- ALB에 연결한 WAF Web ACL은 애플리케이션에 도달하기 전에 악성 HTTP(S) 요청과 과도한 요청을 검사합니다.

이 구성에서 WAF는 웹 공격을 줄이고, 보안 그룹은 허용되지 않은 네트워크 경로가 태스크에 닿지 못하게 막습니다.
WAF가 있어도 태스크 보안 그룹을 인터넷에 열어 두면 안 됩니다.

## 적용 시 유의 사항

- Web ACL을 바로 `Block`으로 운영하기 전에 관리형 규칙을 `Count`로
  적용하고 정상 요청 차단 여부를 로그와 지표로 확인합니다.
- ALB 등 리전 리소스용 Web ACL은 보호 대상과 같은 리전에 만듭니다.
- CloudFront 배포판용 Web ACL과 관련 WAF 리소스는 `us-east-1`에 만들어야 합니다.
- WAF는 DDoS 전체를 해결하는 서비스가 아닙니다. 대규모 DDoS 보호가 필요하면 AWS Shield와의 역할도 함께 검토합니다.

## 참고 자료

- [How AWS WAF works](https://docs.aws.amazon.com/waf/latest/developerguide/how-aws-waf-works.html)
- [Resources that you can protect with AWS WAF](https://docs.aws.amazon.com/waf/latest/developerguide/how-aws-waf-works-resources.html)
- [AWS WAF rules](https://docs.aws.amazon.com/waf/latest/developerguide/waf-rules.html)
- [Control traffic using security groups](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-security-groups.html)
