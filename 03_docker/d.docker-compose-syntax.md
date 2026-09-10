# Docker Compose 문법

Docker Compose는 여러 컨테이너의 이미지, 환경 변수, 네트워크, 볼륨을 YAML로 선언한다.
Compose Specification에서는 최상위 `version` 필드가 필요 없다. 명령은 `docker compose`를
사용한다.

## 기본 예시

```yaml
services:
  app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - '127.0.0.1:8080:8080'
    environment:
      SPRING_PROFILES_ACTIVE: local
    depends_on:
      db:
        condition: service_healthy
    networks: [backend]

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: app
      POSTGRES_USER: app
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD:?set POSTGRES_PASSWORD}
    volumes:
      - postgres-data:/var/lib/postgresql/data
    healthcheck:
      test: ['CMD-SHELL', 'pg_isready -U app -d app']
      interval: 5s
      timeout: 3s
      retries: 10
    networks: [backend]

networks:
  backend:

volumes:
  postgres-data:
```

- `services`: 실행할 컨테이너를 선언한다. 서비스 이름은 같은 네트워크에서 DNS 이름이 된다.
- `build`: 이미지 빌드 설정이며 `image`를 함께 두면 생성 이미지의 이름을 지정할 수 있다.
- `ports`: `호스트:컨테이너` 포트를 게시한다. DB처럼 내부 전용 서비스에는 보통 불필요하다.
- `environment`: 설정을 주입한다. 비밀값은 `.env`를 Git에서 제외하거나 secret 관리 도구를 사용한다.
- `depends_on`: 생성·시작 순서를 제어한다. healthcheck 없이 애플리케이션 준비까지 보장하지 않는다.
- `volumes`: named volume은 Docker가 관리하며, bind mount는 호스트 경로를 직접 연결한다.

## 자주 쓰는 명령

```shell
docker compose config
docker compose up -d --build
docker compose ps
docker compose logs -f --tail 100 app
docker compose exec app /bin/sh
docker compose down
```

- `config`: 변수 치환 결과를 포함한 Compose 설정을 검증한다.
- `up -d`: 서비스를 생성하거나 갱신해 백그라운드에서 실행한다.
- `down`: 컨테이너와 기본 네트워크를 제거한다. `-v`는 named volume도 삭제하므로 주의한다.

## 운영 시 유의사항

- `container_name`은 프로젝트를 여러 개 띄울 때 이름 충돌과 확장 제약을 만들므로 보통 생략한다.
- 소스 bind mount와 개발용 명령은 개발 오버라이드 파일로 분리한다.
- 이미지는 태그 또는 digest로 고정하고, 비밀값과 데이터 디렉터리를 이미지에 포함하지 않는다.
- 서비스 간 연결에는 호스트 포트가 아니라 `db:5432`처럼 서비스 이름과 컨테이너 포트를 사용한다.
