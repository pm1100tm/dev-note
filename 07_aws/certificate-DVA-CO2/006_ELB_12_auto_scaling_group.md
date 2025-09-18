# 🚀 Auto Scaling Group (ASG) 주요 속성

Launch Template (필수 설정)

- 새 인스턴스를 어떻게 만들지 정의

> 👉 예전 Launch Configurations 는 이제 사용 중단됨

## 포함되는 항목들

- AMI + Instance Type
  - 어떤 OS/애플리케이션 이미지(AMI)와 인스턴스 유형을 쓸지 정의
- EC2 User Data
  - 인스턴스 시작 시 실행할 초기 스크립트
  - 예: 앱 설치, 환경 변수 설정
- EBS Volumes
  - 인스턴스에 붙일 스토리지 볼륨 정의
- Security Groups
  - 네트워크 보안 규칙(방화벽 역할)
- SSH Key Pair
  - EC2 인스턴스에 SSH 접속하기 위한 키 설정
- IAM Role for EC2
  - EC2 인스턴스에서 다른 AWS 서비스에 접근할 권한
- Network + Subnet 정보
  - 인스턴스가 생성될 VPC 와 서브넷
- Load Balancer 연동
  - 인스턴스가 자동으로 ELB(Target Group)에 등록되도록 설정

### 🔹 Auto Scaling 설정 값

- Min Size: 항상 유지해야 하는 최소 인스턴스 수
- Max Size: 늘어날 수 있는 최대 인스턴스 수
- Desired Capacity (초기 용량): 시작 시 몇 개를 띄울지

### 🔹 Scaling Policies (스케일링 정책)

- 언제 EC2를 늘리고 줄일지에 대한 조건
- 예:
  - CPU 사용률 70% 초과 시 인스턴스 1개 추가
  - CPU 30% 미만 시 인스턴스 1개 제거

---

✅ 쉽게 요약

- ASG Launch Template = 인스턴스 설계도
- 어떤 인스턴스를, 어떤 조건에서, 몇 개 띄울지를 정의
- 스케일링 정책에 따라 자동으로 인스턴스 증감

---

## 📌 Auto Scaling – CloudWatch Alarms & Scaling

### 동작 개념

- ASG 는 CloudWatch 알람을 기반으로 EC2 개수를 자동으로 늘리거나 줄일 수 있음
- CloudWatch Alarm = 특정 지표를 모니터링하다가 임계치를 넘으면 알림 발생

### 모니터링 지표

- 기본 제공 지표
  - ASG 전체 인스턴스의 평균 CPU 사용률
  - 네트워크 트래픽(In/Out)
- 커스텀 지표
  - 예: 요청 처리량, 대기열 길이

### 알람 기반 동작

- Scale-Out Policy (확장 정책)
  - 조건: 평균 CPU > 70%
  - 동작: EC2 인스턴스 1개 추가
- Scale-In Policy (축소 정책)
  - 조건: 평균 CPU < 30%
  - 동작: EC2 인스턴스 1개 제거

---

✅ 쉽게 요약

- CloudWatch 알람이 임계치 도달 → ASG가 자동으로 확장/축소
- 확장은 트래픽 폭주 대응, 축소는 비용 절감 목적
