# GCP Pub/Sub 관련 CLI

## A. Topic 목록 및 상태 확인

```shell
# 프로젝트의 모든 토픽 확인
gcloud pubsub topics list

# 특정 토픽의 상세 정보
gcloud pubsub topics describe bms-tasks
```

### 🥲 트러블슈팅

- gcloud pubsub topics list 를 했을 때 아무것도 보이지 않았음
- gcloud 는 현재 활성 프로젝트 기준으로만 리소스를 본다.
- 아래의 명령어에서 올바른 프로젝트가 설정되어있어야 함

```shell
gcloud config get-value project

# 프로젝트 설정 변경
gcloud config set project <YOUR_PROJECT_ID>

# Pub/Sub API 자체가 꺼져 있는 경우
## 🔍 API 활성화 여부 확인: 출력이 없으면 ❌ 비활성화 상태
gcloud services list --enabled | grep pubsub

## ✅ 활성화
gcloud services enable pubsub.googleapis.com
```

## B. Subscription 목록 및 상태 확인

```shell
# 모든 구독 확인
gcloud pubsub subscriptions list

# 특정 구독의 상세 정보
gcloud pubsub subscriptions describe task_font

# 구독별 미전달 메시지 수 확인
gcloud pubsub subscriptions list \
 --format="table(name,topic,ackDeadlineSeconds,messageRetentionDuration)"
```

## C. 실제 메시지 Pull 및 내용 확인

메시지 가져오기 (ACK 없이):

```shell
# task_font 구독에서 메시지 1개 가져오기 (미리보기)
gcloud pubsub subscriptions pull task_font \
 --limit=1 \
 --format=json

# 최대 10개까지 가져오기
gcloud pubsub subscriptions pull task_font \
 --limit=10 \
 --format=json
```

메시지 가져오고 즉시 ACK:

- ⚠️ 주의: --auto-ack는 메시지를 영구적으로 제거합니다!

```shell
# 메시지를 가져오고 자동으로 ACK (메시지가 큐에서 제거됨)
gcloud pubsub subscriptions pull task_font \
 --limit=5 \
 --auto-ack
```

메시지 내용 디코딩:

```shell
# 메시지의 base64 인코딩된 data 필드를 디코딩
gcloud pubsub subscriptions pull task_font \
  --limit=1 \
  --format=json | jq '.[0].message.data' | base64 -d

# 여러 메시지 한 번에 디코딩
gcloud pubsub subscriptions pull task_font \
  --limit=10 \
  --format=json | jq -r '.[].message.data' | base64 -d
```

## D. 메시지 발행 테스트

```shell
# 테스트 메시지 발행
gcloud pubsub topics publish bms-tasks \
  --message='{"task_id": "test_123", "rule_number": "1-1"}' \
  --attribute=task_type=font

# 발행 확인
echo "메시지가 발행되었습니다. 구독에서 확인하세요:"
gcloud pubsub subscriptions pull task_font --limit=1
```

---

## 📊 메트릭 및 모니터링 확인

A. GCP Console - Metrics Explorer

- GCP Console → Monitoring → Metrics Explorer

발행 메트릭:

```shell
리소스 타입: Pub/Sub Topic
메트릭:
  - pubsub.googleapis.com/topic/send_message_operation_count
    → 발행된 메시지 수
  - pubsub.googleapis.com/topic/send_request_count
    → 발행 요청 수
```

구독 메트릭 (가장 중요!):

```shell
리소스 타입: Pub/Sub Subscription
메트릭:
  - pubsub.googleapis.com/subscription/num_undelivered_messages
    → ⭐ 미전달 메시지 수 (밀린 메시지)
  - pubsub.googleapis.com/subscription/oldest_unacked_message_age
    → ⭐ 가장 오래된 미확인 메시지 나이
  - pubsub.googleapis.com/subscription/pull_request_count
    → Pull 요청 수
  - pubsub.googleapis.com/subscription/pull_ack_message_operation_count
    → ACK된 메시지 수
```

B. gcloud로 메트릭 확인

```shell
# Cloud Monitoring API를 사용한 메트릭 조회
gcloud logging read \
  "resource.type=pubsub_subscription AND resource.labels.subscription_id=task_font" \
  --limit=50 \
  --format=json
```

---

## 🐛 디버깅 - 메시지가 느린 이유 파악

A. 미전달 메시지 확인 (Backlog)

```shell
# 각 구독의 밀린 메시지 수 확인
for sub in task_font task_color task_logo task_image task_graphic result_font result_color result_logo result_image result_graphic; do
  echo "=== $sub ==="
  gcloud pubsub subscriptions describe $sub \
    --format="value(name,topic,numUndeliveredMessages)"
  echo ""
done


# 출력 메세지
=== task_font ===
projects/your-project/subscriptions/task_font
projects/your-project/topics/bms-tasks
523  ← 523개의 메시지가 밀려있음!
=== task_color ===
...
```

B. Worker가 메시지를 소비하고 있는지 확인

- kubectl로 Worker Pod 로그 확인:

```shell
# Worker Image Pod의 로그 확인
kubectl logs -l app=hyundai-bms-worker-image --tail=100

# 실시간으로 로그 follow
kubectl logs -l app=hyundai-bms-worker-image -f

# 여러 Worker 동시 확인
kubectl logs -l app=hyundai-bms-worker-font --tail=50
kubectl logs -l app=hyundai-bms-worker-color --tail=50
```

C. ACK Deadline 확인

```shell
# 구독의 ACK deadline 확인
gcloud pubsub subscriptions describe task_font \
  --format="value(ackDeadlineSeconds)"
```

ACK Deadline이 너무 짧으면:

- Worker가 메시지를 처리하기 전에 타임아웃되어 다시 큐로 돌아감
- 같은 메시지가 반복적으로 처리됨

해결:

```shell
# ACK deadline을 600초(10분)로 연장
gcloud pubsub subscriptions update task_font \
  --ack-deadline=600
```

D. Dead Letter Queue 확인

- 실패한 메시지들이 쌓이는 곳:

```shell
# Dead Letter Topic 확인
gcloud pubsub topics list | grep dead-letter

# Dead Letter Subscription 메시지 확인
gcloud pubsub subscriptions pull task_font-dead-letter --limit=10
```

---

## 🚀 5. 성능 문제 해결 방법

### 문제 1: 메시지가 밀려있음 (Backlog)

원인:

- Worker가 메시지를 처리하는 속도 < 메시지 생성 속도
- Worker Pod가 충분하지 않음

```shell
# Worker Pod 수 증가 (Kubernetes)
kubectl scale deployment hyundai-bms-worker-image --replicas=5

# 또는 deployment.yaml 수정
kubectl edit deployment hyundai-bms-worker-image
# replicas: 5로 변경
```

### 문제 2: 메시지가 전달되지 않음

확인 사항:

1. Worker가 실행 중인지 확인:

```shell
kubectl get pods -l app=hyundai-bms-worker-image
kubectl get pods -l app=hyundai-bms-worker-logo
kubectl get pods -l app=hyundai-bms-worker-color
kubectl get pods -l app=hyundai-bms-worker-font
```

2. Worker 로그에서 에러 확인:

```shell
kubectl logs -l app=hyundai-bms-worker-image --tail=3000 | grep -i error
kubectl logs -l app=hyundai-bms-worker-logo --tail=3000 | grep -i error
kubectl logs -l app=hyundai-bms-worker-color --tail=3000 | grep -i error
kubectl logs -l app=hyundai-bms-worker-font --tail=3000 | grep -i error
```

3. Pub/Sub 인증 확인:

```shell
# 환경 변수 확인
kubectl exec -it <worker-pod-name> -- env | grep GCP
-- kubectl exec -it hyundai-bms-worker-color-85ddc4b55-dztfr -- env | grep GCP
```

문제 3: 같은 메시지가 반복 처리됨

- Worker가 메시지 처리 후 ACK를 보내지 않음
- ACK deadline이 너무 짧음

해결

```shell
# 코드에서 ACK 확인:
# BMS-system/worker.py 확인
# message.ack() 호출 여부 확인

# ACK deadline 연장:
gcloud pubsub subscriptions update task_font --ack-deadline=600
```

---

## 🛠️ 실시간 모니터링 대시보드 만들기

```shell
#!/bin/bash
# pubsub-monitor.sh

echo "=== Pub/Sub 실시간 모니터링 ==="
echo ""

while true; do
  clear
  echo "🕐 $(date)"
  echo ""

  echo "📊 Topics:"
  gcloud pubsub topics list --format="table(name)"
  echo ""

  echo "📬 Subscriptions (미전달 메시지 수):"
  for sub in task_font task_color task_logo task_image task_graphic; do
    msgs=$(gcloud pubsub subscriptions describe $sub --format="value(name)" 2>/dev/null)
    if [ ! -z "$msgs" ]; then
      undelivered=$(gcloud pubsub subscriptions describe $sub --format="value(numUndeliveredMessages)" 2>/dev/null)
      echo "  $sub: $undelivered"
    fi
  done
  echo ""

  echo "🔄 Worker Pods 상태:"
  kubectl get pods -l app=hyundai-bms-worker-image -o wide 2>/dev/null || echo "  kubectl 연결 안 됨"

  sleep 5
done
```

```shell
chmod +x pubsub-monitor.sh
./pubsub-monitor.sh
```

---

## ✅ 7. 빠른 진단 체크리스트

현재 상태를 빠르게 확인하는 명령어들:

```shell
# 1. 미전달 메시지 수 확인 (밀린 메시지)
gcloud pubsub subscriptions list \
  --format="table(name,topic,numUndeliveredMessages,oldestUnackedMessageAge)"

# 2. 실제 메시지 내용 확인 (미리보기, ACK 안 함)
gcloud pubsub subscriptions pull task_font --limit=1 --format=json | jq

# 3. Worker Pod 로그 확인
kubectl logs -l app=hyundai-bms-worker-image --tail=20

# 4. 테스트 메시지 발행 및 확인
gcloud pubsub topics publish bms-tasks \
  --message='{"test": "message"}' \
  --attribute=task_type=font

sleep 2

gcloud pubsub subscriptions pull task_font --limit=1 --auto-ack
```
