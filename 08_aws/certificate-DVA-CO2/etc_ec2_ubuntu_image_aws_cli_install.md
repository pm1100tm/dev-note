# 🚀 EC2 Ubuntu 이미지에서 AWS CLI 설치하기

AWS가 직접 제공하는 Amazon Linux 2 / Amazon Linux 2023 AMI에는 AWS CLI v2 가 기본
탑재돼 있습니다.

하지만 Ubuntu, Debian, CentOS 같은 Community AMI 는 AWS CLI가 기본 포함되지 않습니다.
즉, Ubuntu 24.04 LTS 같은 경우 → 사용자가 직접 설치해야 합니다.

## Ubuntu 에서 AWS CLI 설치하기

```shell
sudo apt update && sudo apt upgrade -y # 패키지 업데이트
sudo apt install unzip -y # 압축 해제용 유틸
```

AWS CLI 다운로드 / 설치

```shell
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

설치 확인

```shell
aws --version

ubuntu@ip-172-31-41-240:~/download$ aws --version
aws-cli/2.30.0 Python/3.13.7 Linux/6.14.0-1012-aws exe/x86_64.ubuntu.24
```
