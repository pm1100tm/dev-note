# docker compose syntax

- 다수의 도커 컨테이너를 일괄적으로 정의하고 제어하는 도구이다.
- 확장자는 .yml 또는 .yaml 을 사용한다.
- ex\) docker-compose.yml

## ⚡️ docker commpose file syntax 구성

- version: 도커 컴포즈 버전 명시
- services: 실행하려는 컨테이너들을 정의하는 역할
  - image: 이미지명 지정
  - environment: 환경 변수 설정
  - build: 빌드할 이미지에 대한 설정
    - context: 도커 컨텍스트 경로 지정
    - dockerfile: 도커 파일 경로 지정
  - command: 컨테이너가 실행될 때 수행할 명령어
  - ports: 개방할 포트 지정, docker run 명령어의 -p와 동일
  - depends_on: 컨테이너 간 의존성 주입. 명시된 컨테이너가 먼저 생성되고 실행
  - expose: 링크로 연계된 컨테이너에게만 공개할 포트 설정
  - volumes: 컨테이너에 볼룸을 마운트
  - restart: 컨테이너가 종료될 때 재시작 정책
    - no: 재시작하지 않음
    - always: 외부에 의해 종료되었을 때 항상 재시작
    - on-failure: 오류가 있을 시 재시작
- network
- volume
- config
- secret

### docker-compose up [options] [SERVICE...]

컴포즈 파일을 기반으로 컨테이너를 빌드하고 실행한다.

- -f, --file: 파일 경로 지정
  - docker compose -f /path/to/docker-compose.yml up
- -d, --detach: 컨테이너를 백그라운드에서 실행
- --build: 항상 컨테이너를 빌드. 이미지가 없을 경우에도 빌드
- --no-build: 이미지를 빌드하지 않고 기존에 있는 이미지를 사용
- --force-recreate: 변경된 설정을 무시하고 컨테이너를 재생성
- --abort-on-container-exit: 한 컨테이너가 종료되면 모든 컨테이너를 중지

```shell
docker-compose up --build -d
```

### docker-compose down [options] [SERVICE...]

컨테이너들을 중지하고 연결된 네트워크, 볼륨등을 제거하는데 사용된다.

- -v, --volumes: 컨테이너에 연결된 볼륨도 함께 삭제
- --remove-orphans: 파일에서 정의되지 않은 서비스의 컨테이너를 함께 제거
- --timeout TIMEOUT: 컨테이너를 중지할 때 대기하는 시간을 지정

```shell
docker-compose down -v # 모든 컨테이너와 네트워크를 종료 + 볼륨 삭제
docker-compose down # 컨테이너만 중지하고 볼륨과 네트워크 유지
```

### docker-compose start, restart, stop

컴포즈 파일에 정의된 서비스를 시작/재시작/정지

> 📚 down, stop 차이점
>
> - down 은 컨테이너를 정지 + 컨테이와 관련된 모든 리소스 제거
> - stop 은 컨테이너 정지

### docker-compose run

- 새로운 컨테이너를 생성하고 명령 실행
- 특정 서비스를 실행하는 것이 목적
- 컴포즈 파일에 정의된 서비스 중 하나만 실행하며, 즉석에서 컨테이너를 시작

> 📚 up 과의 차이점
>
> - up 은 컴포즈 파일에 정의된 모든 서비스를 시작하며, 컨테이너를 빌드하고 시작한다.
> - 서비스 간의 의존성을 고려하여 모든 서비스를 함께 시작한다.

### docker-compose logs

- docker-compose logs [options] [service...]
  - -f, --follow: 실시간으로 로그를 출력합니다.
  - --tail="all": 최신 로그 몇 줄을 출력합니다. 숫자를 지정하여 출력할 줄 수를 조절할 수 있습니다.
  - --timestamps: 로그에 타임스탬프를 추가하여 출력합니다.
  - --no-color: 로그에 색상을 사용하지 않습니다.
  - --service-logs: 서비스 이름을 로그에 표시합니다.
  - --no-prefix: 로그에 서비스 이름을 표시하지 않습니다.

```shell
docker-compose logs -f # 서비스의 로그를 실시간으로 확인
docker-compose logs service1 # 특정 서비스의 로그만 확인
docker-compose logs --tail=100 # 최신 100줄의 로그만 확인
docker-compose logs --timestamps --service-logs # 로그에 타임스탬프와 서비스 이름을 출력
```

### ETC

- docker-compose pause, unpause
- docker-compose ps
- docker-compose images
- docker-compose config
- docker-compose version
- docker-compose help

## 실습

```docker-compose
version: '3.8'

services:
  db:
    container_name: postgres
    image: postgres:14.11-alpine
    restart: always
    ports:
      - '5433:5432'
    volumes:
      - ./docker/db_data:/var/lib/postgresql/data
    networks:
      - nestjs-backend
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=backend

    healthcheck:
      test: ['CMD', 'pg_isready', '-U', 'postgres']

  pgadmin:
    container_name: pgadmin
    image: dpage/pgadmin4
    restart: always
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - '5050:80'
    networks:
      - nestjs-backend

  app:
    image: node-local:0.1
    container_name: node-local
    build:
      context: .
      dockerfile: ./docker/Dockerfile.local
    restart: always
    ports:
      - '8000:80'
    volumes:
      - ./src:/app/src
    command: >
      sh -c "npm run mig:local:run && npm run start"
    networks:
      - nestjs-backend
    depends_on:
      db:
        condition: service_healthy

networks:
  nestjs-backend:
    driver: bridge
```

### ❓Q1

#### docker-compose의 service 외부와 내부에서 정의하는 volume 의 차이

##### 서비스 내에서 정의하는 volume

- 해당 서비스에만 적용
- 서비스의 컨테이너는 해당 볼륨을 사용하여 데이터를 공유하거나 저장
- 서비스가 실행되는 동안에만 유효하며, 서비스가 중지되면 삭제될 수 있음

##### 서비스 밖에서 정의하는 volume

- 모든 서비스에서 공유/재사용 할 수 있음
- 여러 서비스에서 동일한 데이터를 공유하거나 유지하기 위해 사요됨
- 컨테이너가 종료되어도 유지되며, 다른 서비스에서도 사용할 수 있음
