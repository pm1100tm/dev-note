# 🚀 도커 컨테이너 로그가 쌓여서 발생하는 문제(Docker 로그 폭주 방지 가이드)

가끔 개발 서버의 도커 안에 로그가 쌓여서 이게 용량을 다 먹어서 서버가 멈추는 현상이 있었다.
이걸 어떻게 해결하는지 알아보자.

## ⚙️ 1️⃣ 왜 이런 일이 자주 발생하는가?

Docker는 기본적으로 컨테이너 로그를 JSON 파일 형태로 아래의 경로에 저장합니다.

```shell
/var/lib/docker/containers/<container_id>/<container_id>-json.log
```

```shell
docker logs <container_name>
```

명령으로 보는 출력이 실제로는 저 파일에 계속 append 되는 겁니다.

❗ 그런데 문제는 —
• 로그 회전(log rotation) 설정이 기본적으로 없다.
• 앱 로그뿐만 아니라 stdout / stderr 출력까지 전부 저장된다.
• 장기 실행 서비스(특히 Spring Boot, FastAPI, Nginx, Gunicorn, Celery 등)는 매일 수백 MB~수 GB씩 쌓인다.

결과적으로 /var/lib/docker 경로 아래가 꽉 차서:
• 컨테이너 재시작 불가
• 새 이미지 pull 불가
• 심하면 OS 수준에서 No space left on device 로 서버가 멈춤

## 🧭 2️⃣ 현재 로그 상태 확인 방법

서버에 접속해서 아래 명령으로 확인할 수 있습니다 👇

```shell
# 전체 디스크 사용량 확인
df -h

# docker 디렉토리별 용량 확인
sudo du -h --max-depth=1 /var/lib/docker | sort -h

# 특정 컨테이너 로그 파일 크기 확인
sudo du -h /var/lib/docker/containers/*/*.log | sort -h
```

## 🧰 3️⃣ 근본적인 해결책 — 단계별 대응

### ✅ Step 1: log rotation (로그 자동 회전) 설정

Docker 데몬 설정 파일 /etc/docker/daemon.json 에 아래 내용을 추가하세요:

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "50m",
    "max-file": "3"
  }
}
```

🔹 의미

- 한 컨테이너 로그 파일이 50MB를 넘으면 새 파일로 회전
- 최대 3개까지 보관 → 약 150MB 이후 자동 삭제

적용 후 Docker 재시작:

```shell
sudo systemctl restart docker
```

### ✅ Step 2: 기존 로그 파일 정리 (이미 꽉 찬 경우)

실행 중 컨테이너 로그 비우기

```shell
sudo truncate -s 0 /var/lib/docker/containers/_/_.log
```

⚠️ 이 명령은 로그를 완전히 비웁니다.
운영 환경에서는 회전 정책 적용 후 이 명령을 수동으로 실행하는 식으로 관리하세요.

### ✅ Step 3: 로그 외부화 (log driver 변경)

Docker Compose 예시

```docker-compose
services:
  app:
    image: myapp:latest
    logging:
      driver: "json-file"
      options:
      max-size: "10m"
      max-file: "5"
```

혹은 더 나아가:

- driver: "syslog" → 시스템 syslog로 보냄
- driver: "fluentd" → 중앙 로깅 서버로 전달
- driver: "awslogs" → AWS CloudWatch로 전송

👉 이렇게 하면 /var/lib/docker 에 로그가 거의 쌓이지 않습니다.

### ✅ Step 4: 주기적인 청소 작업(cron or systemd timer)

```shell
# 불필요한 이미지/컨테이너/볼륨 정리
docker system prune -af --volumes
```

주 1회 cronjob 예시:

```shell
0 3 * * 0 /usr/bin/docker system prune -af --volumes >> /var/log/docker_prune.log 2>&1
```

✅ Step 5: 모니터링 및 알림 설정

서버의 / 또는 /var/lib/docker 용량이 80%를 넘을 때 알림이 오도록 설정하세요.

- 간단히 df -h + awk 기반 bash 스크립트로 Slack/Webhook 알림
- 혹은 Grafana + Prometheus / CloudWatch / Datadog 으로 자동 모니터링

---

한 줄 요약하면 👇

“Docker 로그는 자동 삭제되지 않는다. log-opts 설정으로 예방하지 않으면 서버 디스크를 반드시 먹는다.”
