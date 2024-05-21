# Dockerfile syntax

| syntax     |
| ---------- |
| FROM       |
| RUN        |
| WORKDIR    |
| COPY       |
| ADD        |
| CMD        |
| ENTRYPOINT |
| ENV        |
| ARG        |
| EXPOSE     |
| VOLUME     |
| USER       |
| LABEL      |
| SHELL      |
| STOPSIGNAL |

## ⚡️ Description

### FROM

- 베이스 이미지 지정한다.
- Docker Hub 에서 원하는 이미지를 찾을 수 있다.

### RUN

- Dockerfile에서 이미지 빌드 과정 중에 실행된다.
- 각 RUN 명령어마다 새로운 레이어가 생성되므로, 캐시를 효과적으로 사용하여 빌드 속도를 높일 수 있다.

```dockerfile
RUN apt-get update && apt-get install -y \
    git \
    curl
```

### WORKDIR

- 명령이 실행될 작업 디렉토리를 지정한다.
- 해당 디렉토리가 없다면 생성된다.

### COPY

- 호스트 머신의 파일을 컨테이너 내부로 복사한다.

### ADD

- COPY와 유사하나 추가적인 기능 제공한다.
- 로컬 파일이나 URL 복사할 수 있으며, 압축 파일 자동으로 해제하고 원격 파일 다운로드 할 수 있다.

### CMD

- 컨테이너가 시작될 때 실행하는 명령을 설정한다.
- 도커 파일에서 한 번만 사용해야 한다.
- 동시에 여러 개의 CMD 지시문을 사용할 수 있지만, 마지막 CMD만 적용된다.

```dockerfile
CMD ["executable", "param1", "param2"]
CMD ["node", "app.js"]
```

### ENTRYPOINT

- 컨테이너가 시작될 때 실행할 실행 파일 또는 스크립트를 설정한다.
- CMD와 달리 항상 실행되며, CMD와 함께 사용될 경우에는 CMD의 인자로 사용된다.

```dockerfile
ENTRYPOINT ["executable", "param1", "param2"]
```

### ENV

- 환경 변수를 설정한다.

### ARG

- 빌드 중에 전달되는 인수를 정의한다.

### EXPOSE

- 컨테이너가 노출할 포트 설정한다.

### VOLUME

- 호스트 머신과 컨테이너 간에 볼륨을 공유할 수 있도록 설정한다.

### USER

- 컨테이너가 실행될 사용자를 설정한다.

### HEALTHCHECK

- 컨테이너의 상태를 확인하는 방법을 설정한다.

### LABEL

- 이미지에 메타데이터를 추가
- LABEL version="1.0"

### SHELL

- 기본 쉘 지정
- SHELL ["/bin/bash", "-c"]

### STOPSIGNAL

- 컨테이너가 종료될 때 보내질 시그널을 지정
- STOPSIGNAL SIGTERM

## 실습

```dockerfile
FROM node:14

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

CMD ["node", "app.js"]

EXPOSE 3000
```

```dockerfile
FROM node:20.11-alpine

RUN apk update && apk add bash sudo vim

WORKDIR /app

COPY . .

RUN npm cache clean --force && rm -rf node_modules && npm install

EXPOSE 3000

CMD ["npm", "run", "start"]
```

```dockerfile
FROM python:3.11

ENV PYTHONUNBUFFERED=1

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENTRYPOINT ["uvicorn", "myproject.asgi:application", "--host", "0.0.0.0", "--port", "8000"]
```

### Multi-Stage Build

```dockerfile
FROM node:14 AS development

WORKDIR /app

COPY package*.json ./

RUN npm install

COPY . .

CMD ["npm", "run", "dev"]

FROM node:14 AS production

WORKDIR /app

COPY package*.json ./
RUN npm install --production

COPY . .

RUN npm run build

CMD ["npm", "start"]
```

Build Docker image with Multi-Stage

```shell
docker build --target development --t node:dev .
docker build --target production --t node:dev .
```

## RUN, CMD, ENTRYPOINT 차이점과 쓰임새

| syntax     | desc                                                                                              |
| ---------- | ------------------------------------------------------------------------------------------------- |
| RUN        | 이미지 빌드 중에 실행할 명령어 설정                                                               |
| CMD        | 컨테이너가 시작될 때 실행할 기본 명령 설정                                                        |
| ENTRYPOINT | 컨테이너가 시작될 때 실행할 실행 파일 또는 스크립트를 설정하며, CMD명령어의 인자로 사용될 수 있음 |

```dockerfile
FROM node:14
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

# 컨테이너가 시작될 때 실행할 실행 파일을 설정합니다.
ENTRYPOINT ["node"]

# 컨테이너가 시작될 때 실행할 명령 및 파라미터를 설정합니다.
CMD ["app.js"]
```
