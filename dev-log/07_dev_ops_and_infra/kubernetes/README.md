# Kubernetes

Kubernetes의 핵심 구조, Pod, Service, Ingress, HPA와 로컬 학습 환경을 정리한다. Kubernetes는 컨테이너를 직접 관리하는 도구가 아니라, 선언한 원하는 상태를 유지하도록 제어하는 플랫폼이다.

## 기본 개념

* [Kubernetes 개요](001_kube_overview.md)
* [Horizontal Pod Autoscaler](002_kube_hpa.md)

## 로컬 학습

* [로컬 Kubernetes 학습 로드맵](001_kube_-_-_.md)
* [Docker Desktop Kubernetes 설정](002_kube_-_-_.md)
* [Pod 실습](003_kube_pod.md)

## 운영 원칙

* Deployment, Service, Ingress, HPA의 책임을 분리하고 매니페스트를 Git으로 관리한다.
* 리소스 요청·제한과 health check를 지정해 스케줄링과 장애 복구의 기준을 명확히 한다.
* 자동 확장은 metrics-server와 충분한 클러스터 여유 용량이 있어야 동작한다.
