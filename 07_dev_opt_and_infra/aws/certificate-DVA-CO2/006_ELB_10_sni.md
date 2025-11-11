# 🚀 SSL – Server Name Indication (SNI)

## 🔹 배경

- 예전에는 한 서버(또는 로드 벨런서)에서 여러 개의 도메인을 운영하려면, SSL 인증서를 하나 밖에 적용
  할 수 없어서 곤란
- 즉, 여러 도메인을 HTTPS로 서비스할 때 인증서 충돌 문제가 발생

## 🔹 SNI란?

SNI(Server Name Indication) = 하나의 서버(혹은 로드 벨런서)에 여러 개의 SSL 인증서를 올려서,
접속하려는 도메인에 맞는 인증서를 선택해서 제공하는 기능입니다.

⸻

## 🔹 동작 방식

- [1.] 클라이언트(브라우저)가 SSL Handshake를 시작할 때, 접속하려는 호스트네임
  (예: www.mycorp.com) 을 함께 보냄
- [2.] 서버(ALB/NLB/CloudFront)가 해당 호스트네임에 맞는 인증서를 찾아서 응답
- [3.] 만약 해당 인증서가 없으면 → 기본(Default) 인증서를 반환.

## 🔹 지원 여부

- ✅ 지원: ALB, NLB, CloudFront (신세대 서비스)
- ❌ 미지원: CLB (구세대 Classic Load Balancer)

⸻

✅ 쉽게 설명

- SNI = 한 서버에서 여러 HTTPS 도메인을 지원하는 기술
- 예: www.mycorp.com → mycorp 인증서 / shop.mycorp.com → shop 인증서
- 덕분에 하나의 ALB에서 여러 도메인 HTTPS를 손쉽게 처리 가능

---

## 📚 Elastic Load Balancers – SSL Certificates

- 🔹 Classic Load Balancer (CLB, v1)

  - SSL 인증서 1개만 지원
  - 여러 도메인(호스트네임)에서 각기 다른 SSL 인증서를 사용하려면, CLB를 여러 개 생성해야 함

- 🔹 Application Load Balancer (ALB, v2)

  - 여러 리스너(Listener)를 지원
  - 따라서 다수의 SSL 인증서 적용 가능
  - SNI(Server Name Indication) 활용하여, 클라이언트 요청 도메인에 맞는 인증서를 선택

- 🔹 Network Load Balancer (NLB, v2)
  - ALB와 동일하게 여러 리스너 + 여러 SSL 인증서 지원
  - 역시 SNI를 사용해 요청한 호스트네임에 맞는 인증서를 제공

⸻

✅ 쉽게 요약

- CLB: 인증서 1개 제한 → 여러 도메인이면 CLB 여러 개 필요
- ALB/NLB: 인증서 여러 개 가능 → SNI로 도메인별 인증서 구분
