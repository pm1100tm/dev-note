# 🚀 SNAPSHOT.jar 와 SNAPSHOT-plain.jar 의 차이점

Spring Boot 프로젝트를 Gradle 로 빌드하면 두 개의 JAR 파일이 생성될 수 있습니다.

## 1. \<app-name-version>-SNAPSHOT.jar (Executable JAR)

- Spring Boot의 실행 가능한 JAR 파일
- 내부에 **의존 라이브러리와 실행 진입점(Main-Class)**이 포함됨
- java -jar prac-jpa-0.0.1-SNAPSHOT.jar 명령으로 바로 실행 가능

## 2. \<app-name-version>-SNAPSHOT-plain.jar (Plain JAR)

- Spring Boot의 실행 가능한 JAR가 아닌, 단순 JAR 파일
- 애플리케이션의 코드만 포함, 의존 라이브러리는 포함되지 않음
- 실행 진입점(Main-Class)이 포함되지 않으므로 java -jar로 실행 불가능
- Spring Boot와 관계없는 일반적인 JAR 배포용으로 사용될 수 있음

## 📌 왜 -plain.jar 이 생성될까?

Gradle 에서 bootJar 와 jar 두 개의 작업을 실행하기 때문입니다.

- bootJar: Spring Boot 실행 가능 JAR 생성 (<...>-SNAPSHOT.jar)
- jar: 일반적인 JAR 생성 (<...>-SNAPSHOT-plain.jar)

## 📌 -plain.jar 을 제거하는 방법

만약 실행 가능한 JAR 만 필요하고 -plain.jar 이 필요 없다면, jar 작업을 비활성화할 수 있습니다.

```groovy
tasks.named("jar") {
    enabled = false // 일반 JAR(-plain.jar) 비활성화
}
```

### 변경 후 빌드 실행

```bash
./gradlew bootJar
```

### 🚀 결론

| 파일명                   | 설명                                    | 실행 가능 여부        |
| ------------------------ | --------------------------------------- | --------------------- |
| <...>-SNAPSHOT.jar       | Spring Boot 실행 가능 JAR (의존성 포함) | ✅ (java -jar 가능)   |
| <...>-SNAPSHOT-plain.jar | 일반 JAR (의존성 없음, 실행 불가)       | ❌ (java -jar 불가능) |
