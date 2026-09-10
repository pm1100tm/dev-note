# Dockerfile 문법

Dockerfile은 이미지를 만드는 선언형 스크립트다. 빌드 컨텍스트에 포함된 파일만 `COPY`와
`ADD`로 가져올 수 있으므로, `.dockerignore`로 불필요한 파일과 비밀 파일을 제외한다.

## 주요 지시어

| 지시어        | 용도                                                     |
| ------------- | -------------------------------------------------------- |
| `FROM`        | 베이스 이미지를 지정한다.                                |
| `WORKDIR`     | 이후 명령의 작업 디렉터리를 지정한다. 없으면 만든다.     |
| `COPY`        | 빌드 컨텍스트의 파일을 이미지에 복사한다.                |
| `RUN`         | 이미지 빌드 중 명령을 실행한다.                          |
| `ENV`         | 런타임에도 남는 환경 변수를 설정한다.                    |
| `ARG`         | 빌드 시에만 전달되는 인수를 선언한다.                    |
| `EXPOSE`      | 사용할 컨테이너 포트를 문서화한다. 포트를 열지는 않는다. |
| `CMD`         | 기본 실행 명령 또는 인수를 지정한다.                     |
| `ENTRYPOINT`  | 컨테이너의 고정 실행 파일을 지정한다.                    |
| `USER`        | 이후 명령과 런타임의 사용자를 변경한다.                  |
| `HEALTHCHECK` | 컨테이너 상태 확인 명령을 정의한다.                      |

## Node.js 예시

```dockerfile
FROM node:22-alpine

WORKDIR /app
COPY package*.json ./
RUN npm ci --omit=dev
COPY . ./

USER node
EXPOSE 3000
CMD ["node", "app.js"]
```

`COPY package*.json`을 소스 코드보다 먼저 둬야 의존성이 바뀌지 않은 빌드에서 해당
레이어의 캐시를 재사용할 수 있다. 운영 이미지는 lock 파일을 사용하는 `npm ci`가
재현성이 높다.

## RUN, CMD, ENTRYPOINT

```dockerfile
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
CMD ["--spring.profiles.active=prod"]
```

- `RUN`은 이미지를 빌드할 때만 실행되며 결과가 이미지 레이어에 남는다.
- `CMD`는 컨테이너 시작 시의 기본 명령 또는 기본 인수이며 `docker run` 인수로 바꿀 수 있다.
- `ENTRYPOINT`는 고정 실행 명령이다. 위 예시에서 `CMD`는 `java -jar`의 기본 인수가 된다.
- 셸 형식보다 JSON 배열 형태(exec form)를 사용하면 셸을 거치지 않아 시그널 전달이 명확하다.

## 보안과 크기 관리

- `latest` 대신 고정 태그 또는 digest를 사용한다.
- 비밀값을 `ARG`, `ENV`, `COPY`로 이미지에 넣지 않는다. BuildKit secret이나 런타임 주입을 사용한다.
- 패키지 설치 후 패키지 목록 캐시를 같은 `RUN`에서 제거한다.
- 루트가 아닌 사용자로 실행하고, 빌드 도구는 멀티 스테이지 빌드에서 제외한다.

```text
# .dockerignore 예시
node_modules
.git
.env
dist
```

## 빌드와 실행

```shell
docker build -t my-app:1.0 .
docker run --rm -p 3000:3000 my-app:1.0
```

멀티 스테이지 빌드는 [이미지 경량화와 멀티 스테이지]
(001*도커이미지*경량화\_멀티스테이징.md)를 참고한다.
