# 🚀 Connection Draining (연결 종료 처리)

## 🔹 이름 차이

- CLB (Classic Load Balancer) → Connection Draining
- ALB & NLB → Deregistration Delay

## 🔹 기능 설명

- 역할: EC2 인스턴스가 등록 해제 중(de-registering) 이거나 비정상(unhealthy) 상태일 때,
  새 요청은 해당 인스턴스로 보내지 않음
- 이미 처리 중이던 요청(in-flight requests)은 완료될 때까지 유지

## 🔹 설정값

- 대기 시간: 1초 ~ 3600초 (기본값: 300초 = 5분)
- 0으로 설정하면 비활성화 (즉시 연결 종료)
- 요청이 짧은 경우 → 작은 값으로 설정
- 요청이 긴 경우(파일 업로드 등) → 큰 값으로 설정

✅ 쉽게 요약

- Connection Draining = 인스턴스를 내려도, 기존 요청은 끝까지 처리하는 기능
- 기본 300초 동안 대기 → 세션이 자연스럽게 종료되도록 보장
- 클라이언트가 “갑자기 연결 끊김” 문제를 겪지 않게 해줌

---

## ☕️ Connection Draining (Deregistration Delay) 기능에서 기존 요청이 전부 끝났는지 확인

### 📌 1. AWS 관점 (로드 밸런서 레벨에서 확인)

- 로드 벨런서는 Deregistration Delay 동안 인스턴스 상태를 모니터링 합니다.
- Target Group(ALB/NLB) -> 상태 값 확인:
  - draining (등록 해제 중, in-flight 요청 처리 중)
  - unused (더 이상 연결 없음 → 요청 완료됨)
- 확인 방법:
  - AWS 콘솔 → EC2 → Target Groups → Targets 탭
  - 특정 인스턴스 상태가 draining 에서 unused 로 바뀌면 모든 요청이 끝난 것

### 📌 2. 애플리케이션 관점 (서버 레벨에서 확인)

- 서버 내부에서 현재 처리 중인 연결 수를 확인 가능
- 예시 (리눅스 기반):

```shell
netstat -anp | grep ESTABLISHED
# → 아직 처리 중인 연결이 있는지 확인
```

- 웹 서버(Apache, Nginx, Spring Boot 내장 Tomcat 등) 로그나 모니터링 툴로도 현재 활성 세션 수 확인 가능

### 📌 3. CloudWatch 지표 활용

- ELB / ALB 지표 중 ActiveConnectionCount 또는 RequestCount 등을 추적
- Deregistration Delay 시간 동안 지표가 0으로 떨어지면 모든 요청이 완료된 것

✅ 쉽게 설명

- AWS 콘솔에서 Target Group 인스턴스 상태를 보면 제일 간단함 → draining → unused
- 좀 더 정밀하게 보려면 서버 로그 / netstat / CloudWatch 지표 확인
