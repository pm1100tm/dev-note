# 🧨 탄력적 IP 를 적용한 후에 jenkins 가 실패하였다

jenkins 에서 준 log 는 아래와 같다

```logs
+ ssh -o StrictHostKeyChecking=no ubuntu@3.34.72.118
                        cd ~/myblog-backend &&
                        git pull origin main &&
                        docker compose -f docker-dev/compose-container.yml down &&
                        docker compose -f docker-dev/compose-container.yml up --build -d

Warning: Permanently added '3.34.72.118' (ED25519) to the list of known hosts.
Host key verification failed.
fatal: Could not read from remote repository.
```

## known_hosts 불일치 문제

- Jenkins 컨테이너 내부에서 SSH 접속 시, ~/.ssh/known_hosts에 이전 EC2의 호스트키가 저장되어
  있을 수가 있다.
- 엘라스틱 IP를 붙이면서 호스트키 fingerprint가 달라짐 → 보안상 Jenkins가 접속 거부.

### 해결방법

Jenkins 컨테이너 (혹은 Jenkins 서버)에서 아래 실행:

```shell
ssh-keygen -R 3.34.72.118
```

그다음 수동으로 한번 접속해서 fingerprint 갱신:

```shell
ssh -o StrictHostKeyChecking=no ubuntu@3.34.72.118
```
