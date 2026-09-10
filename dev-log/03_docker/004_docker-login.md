# Docker 로그인

`docker login`은 Docker CLI가 레지스트리에 이미지를 pull 또는 push할 때 사용할 인증 정보를
등록한다. Docker Hub뿐 아니라 사설 레지스트리에도 사용할 수 있다.

```shell
# Docker Hub 로그인
docker login --username <username>

# 사설 레지스트리 로그인
docker login registry.example.com --username <username>

# 로그인 상태 확인과 로그아웃
docker info
docker logout registry.example.com
```

- 비밀번호 대신 최소 권한의 개인 액세스 토큰(PAT)을 사용한다.
- 토큰은 명령행 인수에 직접 넣지 않는다. 셸 히스토리와 프로세스 목록에 남을 수 있다.
- CI에서는 CI 플랫폼의 secret 저장소에 토큰을 등록하고, 짧은 수명의 자격 증명을 우선 사용한다.

기본 설정은 `~/.docker/config.json`에 기록될 수 있다. 운영체제의 credential helper가 설정되어
있으면 실제 자격 증명은 키체인 등 별도 저장소에 보관된다. 설정 파일과 CI 로그를 Git에 추가하지
않는다.

## 참고

- [Docker 로그인 문서](https://docs.docker.com/reference/cli/docker/login/)
