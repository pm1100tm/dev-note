# 🚀 AWS CLI 설정

## 1. AWS CLI 설치 (Mac)

```shell
brew install awscli

aws --version

╭─    ~/──────────────────────────────────────────────────────── ✔
╰─ aws --version
aws-cli/2.28.26 Python/3.13.7 Darwin/24.6.0 source/arm64
```

## 2. AWS CLI 설정

AWS 계정의 Access Key 와 Secret Key를 입력해야 합니다.

```shell
aws configure
```

```shell
╭─    ~/──────────────────────────────────────────────────────── ✔
╰─ aws configure
AWS Access Key ID [None]: <입력>
AWS Secret Access Key [None]: <입력>
Default region name [None]: ap-northeast-2
Default output format [None]: json
```

- AWS Access Key ID: (IAM에서 발급받은 키)
- AWS Secret Access Key: (IAM에서 발급받은 시크릿)
- Default region name: ap-northeast-2 (서울 리전이면)
- Default output format: json (또는 table, text)
