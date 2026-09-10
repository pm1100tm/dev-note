# Docker 컨테이너 로그 로테이션

컨테이너 로그는 기본적으로 stdout과 stderr에서 수집된다. 애플리케이션 로그를 파일로
직접 남기는 것보다 표준 출력으로 내보내고, Docker 또는 중앙 로그 시스템에서 수집하는
방식이 컨테이너 운영에 적합하다.

## Compose 서비스별 설정

```yaml
services:
  api:
    image: example/api:1.0
    logging:
      driver: json-file
      options:
        max-size: '20m'
        max-file: '5'
```

- `max-size`: 로그 파일 하나의 최대 크기다.
- `max-file`: 유지할 회전 로그 파일 수다.
- 위 설정은 서비스별 최대 약 100MB의 JSON 로그를 보관한다.
- 로그 정책 변경은 기존 컨테이너가 아니라 새로 생성한 컨테이너에 반영된다.

## 데몬 기본 설정

```json
{
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "20m",
    "max-file": "5"
  }
}
```

Linux에서 이 파일은 보통 `/etc/docker/daemon.json`이다. JSON 문법을 검증하고, Docker
데몬 재시작이 실행 중 컨테이너에 미치는 영향을 확인한 뒤 적용한다.

## 운영 원칙

- 감사·장애 분석에 필요한 로그 보존 기간은 로테이션 크기와 별도로 중앙 로그 저장소에서 관리한다.
- 로그 폭증의 원인은 디버그 레벨, 반복 예외, 요청 본문 출력인 경우가 많으므로 애플리케이션도 점검한다.
- 파일 시스템 사용률을 모니터링하고, 임계치 도달 전에 알림을 받도록 설정한다.
