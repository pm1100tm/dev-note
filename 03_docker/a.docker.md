# Docker

## Docker란

Docker는 애플리케이션과 실행 환경을 이미지로 패키징하고, 그 이미지를 격리된
컨테이너 프로세스로 실행하는 플랫폼이다. 컨테이너는 가상 머신처럼 애플리케이션을
격리하지만, 일반적으로 호스트 커널을 공유한다.

## 핵심 구성 요소

- 이미지(Image): 컨테이너 실행에 필요한 파일 시스템과 메타데이터를 가진 읽기 전용 템플릿이다.
- 컨테이너(Container): 이미지에 쓰기 가능한 계층을 더해 실행한 프로세스 단위다.
- 레지스트리(Registry): 이미지를 저장하고 배포하는 저장소다. Docker Hub가 대표적이다.
- 볼륨(Volume): 컨테이너 생명 주기와 분리해 데이터를 유지하는 Docker 관리 저장소다.
- 네트워크(Network): 컨테이너 간 통신과 외부 노출을 제어하는 가상 네트워크다.

## VM과 컨테이너

| 구분      | 가상 머신(VM)      | 컨테이너                     |
| --------- | ------------------ | ---------------------------- |
| 격리 단위 | 게스트 OS          | 프로세스와 네임스페이스      |
| 커널      | VM마다 자체 커널   | 호스트 커널 공유             |
| 시작 속도 | 상대적으로 느림    | 일반적으로 빠름              |
| 사용 사례 | 다른 OS, 강한 격리 | 애플리케이션 배포, 개발 환경 |

컨테이너는 호스트 커널을 공유하므로 호스트와 다른 커널의 OS를 그대로 실행하는
기술은 아니다. 예를 들어 Linux 컨테이너는 Linux 커널이 필요하며, macOS와 Windows의
Docker Desktop은 보통 내부 Linux VM 위에서 이를 제공한다.

## 이미지에서 컨테이너까지

```text
Dockerfile ──docker build──> Image ──docker run──> Container(process)
                                      │
                                      └── Registry에 push/pull
```

```shell
docker build -t hello-docker:1.0 .
docker run --rm --name hello hello-docker:1.0
```

## 컨테이너를 사용할 때의 원칙

- 컨테이너는 가능한 한 하나의 주 프로세스를 실행하고, 상태는 볼륨이나 외부 저장소에 둔다.
- 이미지는 불변으로 취급하고, 설정은 환경 변수나 설정 파일 마운트로 주입한다.
- `latest` 대신 재현 가능한 버전 또는 digest를 사용한다.
- 컨테이너 내부에서 수동 수정하지 말고 Dockerfile과 배포 설정을 수정해 다시 빌드한다.

## 참고

- [Docker 개요](https://docs.docker.com/get-started/docker-overview/)
