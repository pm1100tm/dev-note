# Dockerfile 레이어와 빌드 캐시

Docker 이미지는 여러 읽기 전용 레이어로 구성된다. 일반적으로 파일 시스템을 변경하는
`RUN`, `COPY`, `ADD`는 새 레이어를 만들며, 이미지들은 같은 레이어를 공유해 저장 공간과
전송량을 줄인다.

```shell
docker image inspect <image-name>
docker history <image-name>
```

`docker history`는 레이어의 명령과 크기를 확인하는 데 유용하다. 다만 이미지 히스토리에
비밀값이 남을 수 있으므로 `ARG`나 `ENV`에 비밀값을 넣지 않는다.

## 캐시를 살리는 순서

```dockerfile
FROM node:22-alpine
WORKDIR /app

# 의존성 파일이 바뀔 때만 이 레이어와 다음 레이어가 다시 빌드된다.
COPY package.json package-lock.json ./
RUN npm ci

# 일반 소스 변경은 의존성 설치 캐시를 무효화하지 않는다.
COPY . ./
RUN npm run build
```

- 한 단계의 입력 또는 명령이 바뀌면 그 단계와 뒤의 레이어 캐시가 무효화된다.
- 자주 바뀌지 않는 의존성 정의 파일을 먼저 복사하고, 자주 바뀌는 소스는 나중에 복사한다.
- `.dockerignore`로 빌드와 무관한 파일을 빌드 컨텍스트에서 제외한다.
- 패키지 설치·캐시 삭제처럼 하나의 원자적 작업은 같은 `RUN`에 둬 불필요한 레이어를 막는다.

## 주의할 점

- 레이어 수를 억지로 줄이는 것보다 캐시 효율, 보안, 읽기 쉬운 Dockerfile을 우선한다.
- 삭제 명령을 다음 `RUN`에 두면 이전 레이어에 파일이 남아 이미지 크기가 줄지 않는다.
- 멀티 스테이지 빌드는 빌드 레이어를 최종 이미지에서 제외하는 가장 안전한 경량화 방법이다.
