# Docker 네트워크

Docker 네트워크는 컨테이너 간 통신과 외부 노출을 분리한다. 같은 사용자 정의 bridge
네트워크에 있는 컨테이너는 서비스 또는 컨테이너 이름을 DNS 이름으로 사용해 통신한다.

## 주요 드라이버

| 드라이버  | 용도                                                   |
| --------- | ------------------------------------------------------ |
| `bridge`  | 단일 Docker 호스트의 일반적인 컨테이너 통신이다.       |
| `host`    | 호스트 네트워크를 직접 사용한다. 포트 격리가 사라진다. |
| `none`    | 루프백 외 네트워크를 제공하지 않는다.                  |
| `overlay` | Swarm 등 다중 호스트 환경의 통신에 사용한다.           |
| `macvlan` | 컨테이너에 LAN상의 별도 MAC 주소를 부여한다.           |

## 기본 명령

```shell
docker network ls
docker network create --driver bridge app-net
docker network inspect app-net
docker network connect app-net api
docker network disconnect app-net api
docker network rm app-net
```

## 통신과 포트 게시

```shell
docker network create app-net
docker run -d --name db --network app-net postgres:16-alpine
docker run --rm --network app-net alpine getent hosts db
```

- 같은 네트워크에서는 `db:5432`처럼 컨테이너 포트로 접근한다.
- `-p 5432:5432`는 호스트 또는 외부 클라이언트를 위한 게시이며 컨테이너 간 통신에는 불필요하다.
- 사용자 정의 bridge 네트워크를 사용하면 이름 기반 서비스 발견을 안정적으로 사용할 수 있다.
- `host` 모드는 격리를 줄이고 포트 충돌 위험이 있어 특별한 성능·네트워크 요구가 있을 때만 쓴다.

네트워크 연결은 통신 경로를 열 뿐 인증과 인가를 대체하지 않는다. DB·캐시의 계정과 TLS 정책을
별도로 적용한다.
