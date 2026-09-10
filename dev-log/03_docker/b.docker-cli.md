# Docker CLI

명령의 전체 옵션은 항상 `docker <command> --help`로 확인한다. 아래 명령은 자주 쓰는
최소 단위이며, 삭제 명령은 운영 환경에서 특히 신중히 사용한다.

## 이미지

```shell
docker pull nginx:1.27-alpine
docker image ls
docker build -t my-api:1.0 -f Dockerfile .
docker image inspect my-api:1.0
docker image rm my-api:1.0
```

- `pull`: 레지스트리에서 이미지를 받는다.
- `build`: Dockerfile과 빌드 컨텍스트를 사용해 이미지를 만든다.
- `image rm`: 이미지를 삭제한다. 실행 중인 컨테이너가 사용하면 삭제할 수 없다.

## 컨테이너

```shell
docker run -d --name api -p 127.0.0.1:8080:8080 my-api:1.0
docker ps
docker ps -a
docker logs --tail 100 -f api
docker exec -it api /bin/sh
docker stop api
docker start api
docker rm api
```

- `-d`: 백그라운드에서 실행한다.
- `-p [host-ip:]host-port:container-port`: 호스트 포트를 컨테이너 포트에 게시한다.
- `--rm`: 컨테이너 종료 시 자동 삭제한다. 일회성 작업에 적합하다.
- `exec`: 실행 중인 컨테이너 안에서 별도 명령을 실행한다.

`-p 8080:8080`은 모든 호스트 인터페이스에 포트를 열 수 있다. 로컬에서만 접근해야
한다면 `-p 127.0.0.1:8080:8080`처럼 바인딩 주소를 명시한다.

## 볼륨과 네트워크

```shell
docker volume create pg-data
docker volume ls
docker volume inspect pg-data
docker network create app-net
docker network connect app-net api
docker network inspect app-net
```

- named volume은 Docker가 저장 위치를 관리하며 컨테이너를 지워도 남는다.
- bind mount는 호스트 경로를 직접 연결한다. 개발 시 소스 코드 마운트에 유용하다.
- 사용자 정의 bridge 네트워크에서는 컨테이너 이름을 DNS 이름으로 사용할 수 있다.

## 정리 명령

```shell
docker container prune
docker image prune
docker system df
```

- `prune`은 사용하지 않는 리소스를 삭제하므로 실행 전 삭제 대상을 확인한다.
- `docker system prune -a --volumes`는 사용하지 않는 이미지와 볼륨까지 지울 수 있다.
- 데이터 볼륨이 포함될 수 있으므로 운영 서버에서 일괄 정리 명령을 습관적으로 실행하지 않는다.

## Compose

```shell
docker compose up -d --build
docker compose ps
docker compose logs -f --tail 100
docker compose exec app /bin/sh
docker compose down
```

Compose v2의 표준 명령은 `docker compose`다. 구형 설치에서만 `docker-compose`가
사용될 수 있다.
