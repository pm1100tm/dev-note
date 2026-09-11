# 🚀 보안 그룹 설정 시 TCP vs HTTP의 차이

## TCP (Transmission Control Protocol)

- 네트워크 계층 중 전송 계층 프로토콜
- “데이터를 안정적으로 주고받는 방식”을 정의
- 포트 번호만 지정하면, 그 포트에서 어떤 상위 프로토콜(HTTP, SSH, DB 등) 을 쓰는지는 신경 쓰지 않음

## HTTP (HyperText Transfer Protocol)

- TCP 위에서 동작하는 애플리케이션 계층 프로토콜
- 주로 TCP 80 포트를 사용 (HTTPS는 TCP 443 포트)
- 즉, HTTP는 TCP를 기반으로 동작하는 구체적인 서비스 프로토콜

---

👉 정리:

- TCP = 데이터 전송의 기본 파이프
- HTTP = TCP 위에서 동작하는 웹 통신 규약
- TCP vs HTTP → TCP는 전송 계층, HTTP는 애플리케이션 계층 (TCP 위에서 동작)
- 보안 그룹 규칙에서 “TCP 8080 허용”과 “HTTP 80 허용”은 결국 특정 포트 열기라는 점에서 비슷한
  개념이지만, “HTTP”는 미리 포트(80)를 가리키고 “TCP”는 내가 원하는 포트를 직접 지정할 수 있다는
  차이가 존재합니다.
