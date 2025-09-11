# 🚀 ARN 이란 무엇인가

ARN(Amazon Resource Name)은 AWS 리소스를 전 세계에서 고유하게 식별하는 이름(식별자) 입니다.
즉, S3 버킷, EC2 인스턴스, IAM 유저, Lambda 함수 등 모든 리소스를 ARN 으로 표현할 수 있다.

## 📌 ARN 형식

```shell
arn:partition:service:region:account-id:resource
```

각 요소 설명

- arn → 항상 arn 으로 시작 (고정 접두어)
- partition → AWS 파티션 (일반적으로 aws, 중국 리전은 aws-cn, GovCloud는 aws-us-gov)
- service → 리소스가 속한 AWS 서비스 (예: s3, ec2, iam)
- region → 리소스가 있는 리전 (예: ap-northeast-2 → 서울 리전)
- account-id → 리소스를 소유한 AWS 계정 ID (12자리 숫자)
- resource → 리소스 이름, 경로, 또는 타입/ID 조합

예:

```shell
"Arn": "arn:aws:iam::9*************:user/dev-swd",
```
