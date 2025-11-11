# 🚀 EFS - Elastic File System

![efs](./assets/efs.png)

- Managed NFS(Network File System)
- AWS 에서 완전관리형으로 제공
- 여러 개의 EC2 인스턴스에서 동시에 마운트 가능
- 멀티 AZ 지원
- EC2 인스턴스가 서로 다른 가용 영역(AZ)에 있어도 같은 EFS 에 접근 가능
- 고가용성(Highly Available) 보장

## 특징

- 자동으로 확장/축소(Scalable) -> 저장 용량을 미리 지정할 필요 없음
- 비용이 비쌈 -> 보통 EBS gp2 의 3배 수준
- 사용만 만큼 과금(Pay-per-use) -> 실제로 저장된 데이터 용량만큼 요금 발생

✅ 쉽게 설명

- EBS = “외장 SSD” (한 인스턴스 전용)
- EFS = “공유 폴더 서버” (여러 인스턴스가 동시에 접근 가능)
- 가격은 비싸지만, 여러 서버에서 같은 파일에 접근해야 할 때 최적

---

## 📌 EFS 사용 사례

- 콘텐츠 관리 시스템(CMS)
- 웹 서비스(Web Serving)
- 데이터 공유(Data Sharing)
- 워드프레스(WordPress) 같은 다중 서버 기반 애플리케이션

### ✅ 기술적 특징

- 프로토콜: NFSv4.1 사용(네트워크 파일 시스템 표준)
- 보안: Security Group 으로 EC2와 마찬가지로 접근 제어 가능
- 호환성: Linux 기반 AMI만 지원(Windows 미지원)
- 암호화: AWS KMS를 사용한 저장 데이터 암호화(Encryption at Rest)
- 파일 시스템: POSIX 준수 -> 일반 리눅스 파일 API 처럼 사용 가능(open, read, write)

### ✅ 확장성 & 비용

- 파일 시스템 용량이 자동 확장됨 → 별도의 용량 계획(capacity planning) 불필요
- Pay-per-use → 실제로 저장한 용량만큼 과금
