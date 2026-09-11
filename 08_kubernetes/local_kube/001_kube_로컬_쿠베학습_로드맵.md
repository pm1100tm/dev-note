# 🚀 로컬 쿠버네티스 학습 로드맵

쿠버네티스 학습 로드맵

| 우선순위 | 환경                            | 추천 대상                        | 한 줄 평가         |
| -------- | ------------------------------- | -------------------------------- | ------------------ |
| 🥇       | **Docker Desktop Kubernetes**   | 입문 ~ 중급 학습                 | 가장 빠르고 안정적 |
| 🥈       | **kind (Kubernetes in Docker)** | 내부 구조까지 이해하고 싶은 경우 | 실무 감각 최고     |
| 🥉       | **minikube (Docker driver)**    | 전통적인 학습 루트               | 무난하지만 느림    |
| ❌       | **k3s (로컬)**                  | 임베디드 / Edge 목적             | 학습용엔 과도      |
| ❌       | **Multipass + kubeadm**         | 클러스터 구축 연습               | 학습 효율 최저     |

## 🥇 1. Docker Desktop + Kubernetes (강력 추천)

왜 가장 좋은가?

- M1 완벽 지원 (arm64)
- VM / 네트워크 신경 ❌
- 클릭 한 번으로 클러스터 생성
- 쿠버네티스 개념 학습에만 집중 가능

설치 & 활성화

```shell
# Docker Desktop 설치 후
Settings > Kubernetes > Enable Kubernetes

kubectl version --client
kubectl cluster-info
kubectl get nodes
```

장점

- ingress / service / deployment 실습 매우 편함
- Docker 이미지 빌드 → 바로 k8s에서 사용 가능
- “쿠버네티스가 뭔지” 배우기에 최적

단점

- 노드 1개 (멀티노드 실험 어려움)
- 클러스터 내부 구조 학습엔 한계

➡️ 쿠버네티스 입문자는 무조건 여기서 시작

## 🥈 2. kind (Kubernetes IN Docker) – 실무 감각 최고

언제 쓰는 게 좋은가?

- 컨트롤 플레인 / 워커 개념을 진짜로 이해하고 싶을 때
- CI 환경과 거의 동일
- 멀티 노드 실습 필수일 때

설치

```shell
brew install kind kubectl
```

멀티 노드 클러스터 생성

```shell
# kind-cluster.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
```

```shell
kind create cluster --config kind-cluster.yaml
kubectl get nodes
```

장점

- 실제 k8s 클러스터 구조와 매우 유사
- 멀티 노드, 스케줄링 실습 가능
- 가볍고 빠름

단점

- ingress, LB 설정이 약간 귀찮음
- 초보자에겐 초기 진입 장벽

➡️ Docker Desktop → kind 순서로 가는 게 베스트

---

1️⃣ Docker Desktop Kubernetes

- Pod / Deployment / Service
- ConfigMap / Secret
- Ingress / Volume

2️⃣ kind (멀티 노드)

- Scheduler 이해
- Node / Taint / Affinity
- Rolling Update / HPA

3️⃣ (선택) GKE / EKS

- 실제 클라우드 운영 감각

```shell
Docker Desktop Kubernetes  →  kind  →  EKS/GKE
```
