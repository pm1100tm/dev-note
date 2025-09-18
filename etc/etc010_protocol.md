# 🚀 프로토콜 - Protocal

## 📌 주요 프로토콜 정리

- **TCP (Transmission Control Protocol)**

  - 연결 지향(Connection-oriented), 신뢰성 보장
  - 데이터를 순서대로, 빠짐없이 전달
  - 예: 웹(HTTP/HTTPS), 이메일(SMTP), 파일 전송(FTP)
  - 장점: 안정적 (데이터 손실, 순서 보장)
  - 단점: 느릴 수 있음 (확인 응답, 재전송 필요)

- **UDP (User Datagram Protocol)**

  - 비연결형(Connectionless), 신뢰성 보장 X
  - 빠르게 데이터 전송, 순서나 손실은 신경 안 씀
  - 예: 영상 스트리밍, 온라인 게임, VoIP(화상 통화)
  - 장점: 빠름, 지연 최소화
  - 단점: 데이터 손실 가능

- **SSL (Secure Sockets Layer)**

  - TCP 위에서 동작하는 보안 계층
  - 데이터를 암호화해서 안전하게 전달
  - 지금은 TLS가 SSL을 대체했지만, 여전히 SSL이라는 용어가 널리 사용됨

- **TLS (Transport Layer Security)**

  - SSL의 최신 버전, 더 강력한 보안 제공
  - HTTPS = HTTP + TLS
  - 예: 인터넷 뱅킹, 로그인, 전자상거래 같은 민감 데이터 전송

- **HTTP (HyperText Transfer Protocol)**

  - 웹 브라우저 ↔ 웹 서버 통신에 사용되는 표준 프로토콜
  - 평문(암호화 X) 전송 → 보안에 취약

- **HTTPS (HyperText Transfer Protocol Secure)**

  - HTTP + TLS
  - 웹 트래픽을 암호화해서 안전하게 전송
  - 모든 현대 웹사이트에서 기본적으로 사용

- **WebSocket**

  - 클라이언트 ↔ 서버 간 양방향 통신 가능
  - 예: 채팅, 실시간 알림, 온라인 협업 도구

- **IP (Internet Protocol)**

  - 네트워크 계층(L3)에서 동작
  - 데이터를 목적지 IP 주소로 전달
  - TCP/UDP는 IP 위에서 동작

- **GENEVE (Generic Network Virtualization Encapsulation)**
  - 네트워크 가상화 터널링 프로토콜 (AWS GWLB에서 사용)
  - IP 패킷을 캡슐화해서 보안 장비 같은 특수 장치로 전달
