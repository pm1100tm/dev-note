# 🚀 도커 이미지 경량화 및 멀티스테이징

멀티 스테이지 Dockerfile이 무엇인가?

하나의 Dockerfile 안에

```Dockerfile
FROM ①빌드 이미지 AS builder
FROM ②런타임 이미지
```

이렇게 두 개 이상의 FROM이 들어가면, 각각은 별도의 단계(Stage) 로 동작한다.

- 첫 번째 FROM → 빌드용 스테이지 (build stage)
- 두 번째 FROM → 최종 실행용 스테이지 (runtime stage)

이런 구조가 된다.

## 📌 왜 “AS builder” 를 쓰는가?

AS builder처럼 이름을 붙이면

```Dockerfile
COPY --from=builder /app/build /app
```

처럼 이전 스테이지의 산출물을 다음 스테이지로 복사할 수 있기 때문이다.

- 1단계: 컴파일, 빌드
- 2단계: 빌드 결과물만 가져와서 가벼운 실행 이미지 구성

## 🧱 멀티 스테이지 Dockerfile의 일반적 구조

```Dockerfile
# 1단계: 빌드 용 이미지
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build   # 최종 빌드 결과가 dist/ 폴더에 생김

# 2단계: 런타임 이미지
FROM nginx:alpine AS runtime
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

동작 흐름

- 1단계 (builder): Node.js 기반 소스 복사 → npm install → 빌드
- 2단계 (runtime): nginx:alpine 빌드 결과물만 복사 → 실행

## 📌 왜 멀티 스테이지를 써야 하는가?

### 1. 이미지 사이즈가 극단적으로 줄어듦

Node, Java JDK, Python dev 패키지 등 빌드에만 필요한 것들을 최종 이미지에 포함시키지 않음.

예:

- 기존 단일 Dockerfile → 1GB
- 멀티 스테이지 → 50MB 이하

### 2. 보안·성능 향상

최종 이미지에 컴파일러, 빌드 도구, 테스트 라이브러리 등이 없으므로 공격 표면이 작아짐.

### 3. 캐싱이 훨씬 효율적

빌드 스테이지와 런타임 스테이지가 분리되므로 수정된 부분에 따라 캐시가 독립적으로 적용됨.

### 4. 완전한 CI/CD 친화적 패턴

GitHub Actions, GitLab, Jenkins 등에서 빌드 → 런타임 생성까지 하나로 처리됨.

### 🧪 Spring Boot 예시 (Java 기반)

```dockerfile
# 1. 빌드 스테이지
FROM gradle:8.3-jdk17 AS builder
WORKDIR /app
COPY . .
RUN gradle clean bootJar --no-daemon

# 2. 런타임 스테이지
FROM eclipse-temurin:17-jre AS runtime
WORKDIR /app
COPY --from=builder /app/build/libs/*.jar app.jar

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

- 1단계: JDK + gradle → jar 파일 생성
- 2단계: JRE만 포함된 초경량 이미지 → 실행만 함

### 요약

```
| 개념 | 설명 |
|------|-------|
| Multi-Stage Dockerfile | 하나의 Dockerfile에 여러 `FROM`을 사용하여 빌드와 실행을 분리 |
| AS builder | 해당 단계에 이름을 붙여 이후 단계에서 빌드 산출물을 가져올 수 있게 함 |
| 장점 | 이미지 사이즈 감소, 보안 향상, 빌드 속도 향상, 더 깔끔한 구조 |
```

---

## 🔥 파이썬은 빌드를 안 하지만 멀티 스테이지로 사이즈가 줄어드는 이유

### ✔️ 1. 빌드용 패키지(컴파일러 등)를 최종 이미지에서 제거할 수 있기 때문

파이썬 패키지 중 상당수가 C 확장 모듈을 포함한다.

예:

- numpy
- pandas
- scipy
- opencv
- uvloop
- orjson
- psycopg2
- cryptography

이런 패키지는 설치 시 반드시 아래 패키지가 필요함:

- build-essential
- gcc
- g++
- libssl-dev
- libffi-dev
- python3-dev
- musl-dev
- linux headers

- 이들을 설치하면 이미지 사이즈가 200~300MB 이상 즉시 증가한다.
- 멀티 스테이지를 쓰면 → 최종 이미지에서는 전부 제거 가능.

### ✔️ 2. pip install 결과물(wheels)만 최종 이미지에 넣고 개발 도구는 버릴 수 있음

```dockerfile
FROM python:3.11-slim AS builder
WORKDIR /app

RUN apt-get update && apt-get install -y gcc build-essential
COPY requirements.txt .
RUN pip install --prefix=/install -r requirements.txt

FROM python:3.11-slim
WORKDIR /app
COPY --from=builder /install /usr/local
COPY . .

CMD ["python", "main.py"]
```

➤ 동작 정리

- 1단계: pip install + gcc + build-essential 사용
- 2단계: 이미 빌드된 wheel 결과물만 복사
- gcc, build-essential, python-dev, header 파일들 → 최종 이미지에는 없음

👉 최종 이미지가 300500MB → 60100MB 수준으로 줄어듦

### ✔️ 3. 가장 큰 효과: apt 패키지가 최종 이미지에서 사라진다

pip 설치를 위해 임시로 설치한:

- gcc
- build-essential
- libpq-dev
- musl-dev
- libssl-dev
- libmagic-dev

- 이런 것들이 사실 이미지에서 제일 크게 차지한다.
- 멀티 스테이지로 하면 → 최종 이미지에서 전부 제거됨.

### ✔️ 4. 최종 이미지는 ultra-slim 이미지로 구성 가능

예:

- python:3.11-slim → 약 90MB
- python:3.11-alpine → 약 15MB (단, musl 빌드 문제 발생 가능)

빌드 단계에만 heavy 패키지를 넣고, 최종 단계는 슬림 + built wheels만 복사 방식으로 이미지 크기를
최대한 줄일 수 있다.

### 📦 파이썬에서 멀티 스테이지가 효과 있는 경우

| 상황                                    | 효과                      |
| --------------------------------------- | ------------------------- |
| **numpy, pandas 설치**                  | 매우 큼                   |
| **opencv 설치**                         | 엄청 큼 (200MB 이상 절감) |
| **psycopg2 등 C 확장 설치**             | 큼                        |
| **알파인 기반에서 빌드**                | 필수                      |
| **PyTorch/TF처럼 외부 wheel 가져올 때** | 효과 큼                   |

### 👎 멀티 스테이지 효과가 “적은” 경우

- pure Python 프로젝트
- pip 패키지들이 전부 pure python wheels
- apt 패키지를 전혀 설치하지 않는 경우

이런 경우에는 거의 영향 없음(20~30MB 정도).

### 정리

| 파이썬 멀티 스테이지        | 효과                  |
| --------------------------- | --------------------- |
| C 확장 패키지 설치 시       | 매우 큼               |
| APT 빌드 도구 포함 시       | 매우 큼               |
| wheels만 최종 이미지에 복사 | 이미지 무게 대폭 감소 |
| pure Python만 있을 때       | 효과 적음             |
