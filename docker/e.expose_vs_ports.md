# Docker Network

- 도커 컨테이너들이 서로 통신할 수 있도록 네트워크 인터페이스를 제공하는 시스템이다.
- 컨테이너 간의 통신을 허용하여 호스트 시스템 또는 외부 네트워크와의 통신을 제어한다.
- 컨테이너의 통신을 분리하고 관리함으로써 보안을 강화하고 네트워크 구성을 간소화한다.

기본적으로 docker create 또는 docker run을 사용하여 컨테이너를 만들거나 실행하면, 컨테이너의
포트가 외부에 노출되지 않는다. Docker외부의 서비스에서 포트를 사용할 수 있게 하려면 --publish 또는
-p 플래그를 사용한다. 이렇게 하면 호스트에 방화벽 규칙이 생성되어 컨테이너 포트를 외부 세계로 향하는
Docker Host의 포트애 매핑한다.

다른 컨테이너에서 컨테이너에 액세스할 수 있도록 하려는 경우 컨테이너의 포트를 게시할 필요가 없다.
컨테이너를 동일한 네트워크(일반적으로 브리지 네트워크)에 연결하여 컨테이너 간 통신을 활성화할 수 있다.

기본적으로 컨테이너는 연결되는 모든 Docker 네트워크에 대해 IP주소를 할당받는다. 컨테이너는 네트워크의
IP 서브넷에서 IP 주소를 받는다. Docker 데몬은 컨테이너에 대한 동적 서브넷 설정 및 IP 주소 할당을
수행한다. 각 네트워크에는 기본 서브넷 마스크와 게이트웨이도 있다.

## 🧐 Docker Network Driver

| Driver  | Description                                                                                |
| ------- | ------------------------------------------------------------------------------------------ |
| bridge  | 단일 호스트 내 컨테이너 간의 통신(default)                                                 |
| host    | 컨테이너가 호스트의 네트워크를 직접 사용. 즉, 호스트와 컨테이너 간의 네트워크 격리를 삭제. |
| none    | 네트워크가 없는 격리된 컨테이너                                                            |
| overlay | 다중 호스트 간의 컨테이너 통신(connect multiple Docker daemons together)                   |
| ipvlan  | IPvlan networks provide full control over both IPv4 and IPv6 addressing.                   |
| macvlan | 컨테이너에 고유한 MAC 주소와 IP 주소 할당                                                  |

## 🧵 Docker Network CLI

```shell
docker network ls
docker network create --driver bridge my-bridge-network
docker network rm my-bridge-network
docker network inspect my-bridge-network
docker network connect my-bridge-network my-container
docker network disconnect my-bridge-network my-container
```

### bridge

- 기본 네트워크
- 동일한 호스트에서 실행되는 컨테이너 간의 통신을 지원한다.
- 호스트 네트워크와 분리되어 있으며, 기본적으로 NAT(Network Address Translation)가 활성화

```shell
docker network create my-net
docker network rm my-net
docker create --name my-nginx --network my-net -p 8080:80 nginx:latest
docker network connect my-net my-nginx # 실행 중인 컨테이너에 연결해야 할 경우
docker network disconnect my-net my-nginx # 실행 중인 컨테이너에 연결 해제해야 할 경우
```

### host

- 호스트 시스템의 네트워크 인터페이스를 사용하여 컨테이너를 실행한다.
- 이 경우 컨테이너는 호스트와 동일한 네트워크에 연결되며, 호스트의 네트워크와 동일한 IP주소를 갖는다.

### overlay

- 여러 호스트 간의 컨테이너 통신을 지원하는 네트워크
- Docker Swarm과 같은 오케스트레이션 도구에서 사용된다.

```shell
docker network create -d overlay --attachable my-attachable-overlay
docker network create --help
```

### macvlan

- 컨테이너에 고유한 MAC 주소를 할당하여, 호스트 네트워크와 직접 연결되는 네트워크이다.
- 이 네트워크를 사용하면 컨테이너가 호스트와 동일한 네트워크에서 직접 통신할 수 있습니다.

### none

- 컨테이너의 네트워킹 스택을 완전히 분리하려면 컨테이너를 시작할 때 --network none 플래그를 사용
- 컨테이너 내에는 루프백 장치만 생성
- 외부에서 접근할 수 없는 서비스에 적합

```shell
docker run --rm --network none alpine:latest ip link show
```

## ☕️ Ref

- [docker network](https://docs.docker.com/network/)
