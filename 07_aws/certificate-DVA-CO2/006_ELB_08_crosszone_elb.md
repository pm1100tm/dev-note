# 🚀 Cross-Zone Load Balancing(교차 존 로드 밸런싱) 이란?

• 정의: 로드 밸런서가 들어온 요청을 모든 가용 영역(AZ)의 인스턴스들에 균등하게 분산하는 기능

## 🔹 동작 방식

- [1.] 활성화된 경우 (With Cross-Zone LB)
  - 각 로드 밸런서 노드가 모든 AZ의 인스턴스에 트래픽을 고르게 분배
  - 예: AZ A에 로드 밸런서 노드 1개, AZ B에 노드 1개가 있어도 → 두 노드 모두 AZ A와 B의 모든
    인스턴스로 요청을 고르게 나눔
- [2.] 비활성화된 경우 (Without Cross-Zone LB)
  - 각 로드 밸런서 노드는 자기 AZ에 있는 인스턴스에게만 트래픽을 전달
  - AZ마다 인스턴스 수가 다르면 부하 불균형 발생

## 🔹 로드 밸런서별 기본 설정 및 요금

- Application Load Balancer (ALB)

  - 기본값: 활성화(Enabled by default)
  - Target Group 단위에서 비활성화 가능
  - 교차 AZ 데이터 전송 비용 없음

- Network Load Balancer (NLB) & Gateway Load Balancer (GWLB)

  - 기본값: 비활성화(Disabled by default)
  - 활성화하면 교차 AZ 데이터 전송 비용 발생 ($)

- Classic Load Balancer (CLB)
  - 기본값: 비활성화(Disabled by default)
  - 활성화해도 교차 AZ 데이터 전송 비용 없음

---

✅ 쉽게 요약

- Cross-Zone LB 켜짐 → 모든 AZ 인스턴스에 균등 분배 (더 안정적)
- 꺼짐 → 각 AZ 안에서만 분배 (불균형 위험)
- 비용:
  - ALB, CLB → 교차 AZ 데이터 전송 무료
  - NLB, GWLB → 유료
