# 🚀 Auto Scaling Group – Scaling Policies

## 🔹 1. Dynamic Scaling (동적 스케일링)

ASG가 CloudWatch 지표를 보면서 자동으로 크기를 조정

- Target Tracking Scaling (목표 추적 스케일링)
  - 가장 간단하게 설정 가능
  - 특정 지표를 일정 수준으로 유지
  - 예: “ASG 평균 CPU 사용률을 40% 근처로 유지”
- Simple / Step Scaling (단순 / 단계적 스케일링)
  - CloudWatch 알람이 발동하면 즉시 동작
  - 예: CPU > 70% → 인스턴스 2개 추가
  - 예: CPU < 30% → 인스턴스 1개 제거

⸻

## 🔹 2. Scheduled Scaling (예약 스케일링)

- 미리 예상되는 패턴에 따라 스케일링 예약
- 예: 금요일 오후 5시 트래픽 폭주 예상 → 최소 용량을 10으로 올려놓음

## 🔹 3. Predictive Scaling (예측 스케일링)

- 머신러닝 기반으로 부하를 예측
- 과거 트래픽 패턴을 학습 → 미래의 트래픽을 예측해서 미리 스케일링 예약
- 즉, 트래픽 증가가 오기 전에 선제적으로 인스턴스를 늘려 성능 저하를 방지

---

✅ 쉽게 요약

- Target Tracking: 목표 값 유지 (자동 조정, 가장 쉬움)
- Simple/Step: 알람 발생 시 지정된 대로 증감
- Scheduled: 트래픽 패턴을 예측해서 미리 조정
- Predictive: 머신러닝으로 미래 부하 예측 후 선제 대응
