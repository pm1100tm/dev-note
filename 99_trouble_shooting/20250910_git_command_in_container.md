# 🧨 Docker container 안에서 git commnad 가 작동하지 않았다

Java Springboot 프로젝트를, AWS EC2 에 Docker 컨테이너로 띄웠다.

그 후, 컨테이너 안에 들아가서 git 명령어를 실행하면 아래와 같은 메세지가 표시되었다.

```shell
docker exec -it myblog-backend-dev-container bash

af6ad9177f08:/app# git status
fatal: detected dubious ownership in repository at '/app'
To add an exception for this directory, call:

	git config --global --add safe.directory /app
af6ad9177f08:/app#
```

## 📌 오류 원인

해당 오류 메시지는 Git 2.35 이후 버전에서 추가된 보안 기능 때문입니다.

- 컨테이너 안에서 /app 디렉토리는 호스트(EC2)의 ~/myblog-backend 가 volume mount 된 것입니다.
- Git은 “repo 디렉토리의 소유자(uid)“와 “현재 실행하는 사용자의 uid”가 다르면
  dubious ownership(의심스러운 소유권) 에러를 냅니다.
- 즉, 호스트(EC2)에서 만든 repo → 소유자는 ubuntu (uid=1000)
- 컨테이너 안에서 실행하는 프로세스 → 보통 root (uid=0) → 소유권 mismatch 발생 →
  Git이 안전하지 않다고 판단 → git status 거부.

## 📌 해결 방법

컨테이너 안에서 Git에 /app을 safe directory로 추가하면 됩니다:

```shell
git config --global --add safe.directory /app
```

추가 후 확인

```shell
git status
git log
```

📌 영구 적용 방법

매번 컨테이너 들어가서 실행하는 게 귀찮다면, Dockerfile-app에 아래를 추가합니다.

```shell
RUN git config --global --add safe.directory /app
```

이렇게 하면 컨테이너가 빌드될 때 자동으로 /app을 safe directory로 등록합니다.
