# Docker

Docker는 애플리케이션과 실행에 필요한 의존성을 이미지로 만들고, 이를 컨테이너로 실행하는 플랫폼이다. 개발·테스트·운영 환경의 차이를 줄이고 배포 단위를 일관되게 관리하는 데 사용한다.

Docker Compose는 여러 컨테이너의 설정과 관계를 하나의 YAML 파일로 선언하고 함께 실행하는 도구다. 현재 Docker Desktop과 Docker Engine에서는 `docker compose`(공백 포함) 명령을 우선 사용한다.

## 학습 순서

* [Docker 개념](a.docker.md)
* [Docker CLI](b.docker-cli.md)
* [Dockerfile 문법](c.dockerfile-syntax.md)
* [Docker Compose 문법](d.docker-compose-syntax.md)
* [컨테이너 로그 로테이션](e.docker_container_log_lotation.md)
* [Docker 네트워크](f.docker-network.md)
* [EXPOSE와 ports](g.expose_vs_ports.md)
* [Dockerfile 레이어와 빌드 캐시](h.dockerfile-layer.md)
* [이미지 경량화와 멀티 스테이지](001_-_-_.md)
* [다른 Compose 프로젝트의 네트워크 연결](002_-_-_-_-_-_.md)
* [Docker 로그 정리](003_-_.md)
* [Docker 로그인](004_docker-login.md)

## 운영 전 확인할 것

* 비밀값은 이미지와 Git 저장소에 넣지 말고 환경 변수, secret 또는 외부 비밀 관리 도구로 주입한다.
* 데이터베이스처럼 상태를 갖는 서비스는 named volume 또는 외부 저장소의 백업 정책을 마련한다.
* 로그 크기 제한과 디스크 사용량 알림을 먼저 설정한다.
* `docker system prune --volumes`는 사용하지 않는 볼륨을 삭제하므로 대상 확인 없이 운영에서 실행하지 않는다.
