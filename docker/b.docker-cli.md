# Docker CLI(Command line interface)

| command                | description                                         |
| ---------------------- | --------------------------------------------------- |
| docker pull            | 도커 레지스트리에서 이미지 받아옴                   |
| docker images          | 도커 이미지 리스트 확인                             |
| docker run             | 도커 이미지를 생성하거나, 컨테이너를 실행           |
| docker build           | 도커 이미지를 생성                                  |
| docker rmi             | 도커 이미지 삭제                                    |
| docker rm              | 도커 컨테이너 삭제                                  |
| docker ps              | 도커 컨테이너 리스트 확인                           |
| docker start           | 도커 컨테이너 시작                                  |
| docker stop            | 도커 컨테이너 종료                                  |
| docker volume          | 도커 볼륨 관련 명령어                               |
| docker network         | 도커 네트워크 관련 명령어                           |
| docker image prune     | 사용하지 않는 이미지 삭제                           |
| docker container prune | 사용하지 않는 컨테이너 삭제                         |
| docker system prune    | 사용하지 않는 이미지, 볼륨, 네트워크, 컨테이너 삭제 |

위의 명령어는 각각의 옵션이 있으며, 보통 옵션을 조합하여 사용한다.

> 📚 명령어에 --help 를 붙이면 각 명령어의 옵션들을 볼 수 있다.
>
> - docker ps --help

## docker pull

- docker pull [OPTIONS] NAME[:TAG|@DIGEST]
  - -a, --all-tags: 이미지의 모든 태그 다운로드
  - --disable-content-trust: 이미지의 content trust 비활성화
  - --platform: 특정 플랫폼의 이미지 다운로드

```shell
docker pull ubuntu
docker pull ubuntu:20.04
docker pull --platform linux/arm64 ubuntu
```

## docker build

- docker build [OPTIONS] PATH | URL | -
  - -f, --file: 사용할 Dockerfile의 경로 지정. 기본적으로 현재 디렉토리의 Dockerfile 사용
  - --build-arg: 빌드 시에 사용할 빌드 인수 설정
  - -t, --tag: 생성할 이미지에 태그 지정(<이미지\_이름>:<태그> 형식으로 지정)
  - --target: 멀티스테이지 빌드에서 목표 빌드 스테이지 지정
  - --network: 빌드 컨텍스트를 가져오는 데 사용되는 네트워크 지정
  - --compress: 빌드 컨텍스트를 전송할 때 압축 사용
  - --progress: 빌드 진행 상황을 표시하는 방법 지정
  - --no-cache: 캐시를 사용하지 않고 빌드를 진행
  - --pull: 빌드 전에 베이스 이미지를 항상 업데이트
  -

```shell
docker build -t node-local:0.1 -f ./docker/Dockerfile.local .
```

## docker run

- docker run [OPTIONS] IMAGE [COMMAND] [ARG...]
  - -d, --detach: 컨테이너 백그라운드 실행
  - --name: 컨테이너 이름 지정
  - -p, --publish: 호스트와 컨테이너 간 포트 매핑 설정(포트포워딩)
  - -v, --volume: 호스트와 컨테이너 간 볼륨 마운트
  - -e, --env: 환경 변수 설정
  - --network: 컨테이너 특정 네트워크에 연결
  - --rm: 컨테이너가 종료되면 자동으로 삭제
  - -it: 대화형 터미널 사용하여 컨테이너를 실행
  - --restart: 컨테이너 재시작 정책 설정
  - --privileged: 컨테이너에 모든 권한 부여

```shell
docker run -it --name backend-local -p 3000:3000 node-local:0.1
docker run -d --name api-local -p 3000:3000 node-local:0.1
```

## docker ps

- docker ps [OPTIONS]
  - -a: 종료된 컨테이너도 포함하여 모든 컨테이너를 표시
  - -f, --filter:
    - 지정된 조건에 맞는 컨테이너만 표시
    - 예를 들어, --filter "status=running"은 실행 중인 컨테이너만 표시
  - --format
    - 출력 형식 지정
    - 특정 필드를 사용하여 사용자 정의 형식을 지정
  - -n, --last n: 최근에 생성된 N개의 컨테이너만 표시
  - --no-trunc: 컨테이너 ID를 자르지 않고 전체 길이로 표시
  - -q, --quiet: 컨테이너 ID만 표시
  - -s, --size: 컨테이너의 사용 중인 디스크 공간 크기를 표시
  - --help: 도움말 표시

```shell
docker ps # Show just running containers
docker ps -a # Show all containers
docker ps -q # Only display container IDs
docker ps -a -q
```

## docker start, stop

- docker start | stop [container id | container name]

```shell
docker start 4e5015ee5552
docker stop 4e5015ee5552
```

## docker rmi

- docker rmi [image id | image name]

```shell
docker rmi 4e5015ee5552
docker rmi node-local:0.1
```

## docker rm

- docker rm [container id | container name]

```shell
docker rm 4e5015ee5552
```

## docker exec

- docker exec [OPTIONS] CONTAINER COMMAND [ARG...]
  - -d, --detach: 명령을 백그라운드에서 실행
  - -i, --interactive: 명령의 표준 입력 연결
  - -t, --tty: 의사 TTY를 할당하고, 명령을 상호작용적으로 실행할 수 있도록
  - --user: 명령을 실행할 사용자 지정
  - --privileged: 컨테이너를 특권 모드로 실행
  - --workdir: 명령을 실행할 작업 디렉토리 지정

```shell
docker exec -it my-container bash
docker exec -d my-container ls -l
docker exec --user myuser my-container whoami
docker exec --workdir /app my-container ls -l
docker exec -it node-local bash
docker exec -it node-local /bin/sh
```

## docker network

```shell
docker network ls
docker network create my-network
docker network inspect my-network
docker network rm my-network
```

## prune, system prune

- 사용하지 않는 이미지, 컨테이너, 볼륨 등을 자동으로 삭제한다.
- rmi, rm 등의 명령어는 명시적으로 삭제할 때 사용된다.
- docker image prune 은 기본적으로 dangling 이미지를 삭제하며 --all 옵션을 사용하면 모든
  사용되지 않는 것들을 삭제한다.

> 📚 dangling 이미지란?
>
> 어떤 태그도 지정되지 않은 이미지로, 보통 더 이상 사용되지 않는 중간 생성 이미지나 이전 버전의
> 이미지이다. 도커 이미지의 계층 구조에서, 빌드 과정 중에 생성된 여러 이미지 레이어가 있는데, 이 중간
> 레이어들이 새로운 이미지 빌드 과정에서 필요 없어지면 dangling 이미지로 분류된다.
>
> docker images 명령어를 사용하여 dangling 이미지를 확인할 수 있다.
>
> docker images -f "dangling=true"

```shell
docker image prune
docker container prune
docker volume prune
docker network prune
```

- docker volume prune: 사용하지 않는 볼륨 삭제
- docker network prune: 사용하지 않는 네트워크 삭제
- docker system prune: 사용하지 않는 이미지, 볼륨, 네트워크 및 컨테이너 삭제
- docker system prune -a: 모든 이미지와 중단된 컨테이너까지 모두 삭제
- docker system prune -f: 강제로 모두 삭제
- docker system prune --volume: 사용하지 않는 볼륨 삭제

### docker system prune

사용하지 않는 이미지, 볼륨, 네트워크 및 컨테이너를 삭제한다.

```shell
docker system prune # 사용하지 않는 이미지, 볼륨, 네트워크 및 컨테이너 삭제
docker system prune -a # 모든 이미지와 중단된 컨테이너까지 모두 삭제
docker system prune -f # 강제로 모두 삭제
docker system prune --volume # 사용하지 않는 볼륨 삭제
```
