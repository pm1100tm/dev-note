# 🚀 Spring 로그 제대로 쓰기 — SLF4J의 올바른 사용법

스프링(Spring Boot) 프로젝트를 하다 보면 아래처럼 로그를 남기는 코드를 자주 보게 됩니다 👇

```java
log.info("Response: " + response);
```

딱히 틀려 보이지 않지만, 이런 코드가 누적되면 서비스 성능을 은근히 잡아먹습니다.

이번 글에서는 Spring에서 사용하는 SLF4J 로깅의 올바른 사용법과 좋은 예 / 나쁜 예를 코드로
비교해봅니다.

## 🔍 1. SLF4J란?

SLF4J(Simple Logging Facade for Java)는 Logback, Log4j, java.util.logging 등
다양한 로깅 프레임워크를 하나의 통일된 인터페이스로 감싸주는 추상화 계층입니다.

즉,

```java
private static final Logger log = LoggerFactory.getLogger(MyService.class);
```

이 한 줄만 있으면 어떤 로깅 엔진(Logback, Log4j2, etc.)을 쓰든 log.info() log.error()
같은 공통 문법으로 로그를 남길 수 있습니다.

## ⚙️ 2. 자주 하는 잘못된 로그 작성 방식

❌ 문자열 결합 방식

```java
log.info("Response: " + response);
```

겉보기엔 문제 없어 보이지만, response가 toString() 연산을 포함한 복잡한 객체일 경우 성능 저하를
유발합니다.

왜냐면, 로그 레벨이 INFO보다 낮아서 로그가 실제로 출력되지 않더라도 "S3 list: " + response 문자열
결합은 이미 수행됩니다. 즉, 불필요한 StringBuilder 연산과 toString() 호출이 일어납니다.

## ✅ 3. 올바른 로그 작성 방식 — 지연 평가(Lazy Evaluation)

SLF4J는 지연 평가(lazy evaluation) 를 지원합니다. 아래처럼 {} 플레이스홀더 문법을 쓰면,
로그가 실제로 출력될 때만 문자열이 만들어집니다.

```java
log.info("Response: {}", response);
```

이 코드는 다음과 같은 장점이 있습니다:

| 항목      | 설명                                                          |
| --------- | ------------------------------------------------------------- |
| 🚀 성능   | 로그 레벨이 비활성화(INFO 미사용)면 문자열 연산 자체가 생략됨 |
| 🧠 가독성 | {} 위치만 보면 어떤 값이 출력될지 바로 파악 가능              |
| 🔄 일관성 | log.debug(), log.warn() 등 모든 레벨에서 동일한 문법 사용     |

## 👏 4. 비교: 좋은 예 vs 나쁜 예

| 구분       | 코드                                        | 결과/문제점                                |
| ---------- | ------------------------------------------- | ------------------------------------------ |
| ❌ 나쁜 예 | log.info("Response: " + response);          | 문자열 결합 항상 수행됨                    |
| ❌ 나쁜 예 | log.info("{}" + "Response: " + response);   | {} 쓸 이유 없음, 동일하게 문자열 결합 발생 |
| ✅ 좋은 예 | log.info("Response: {}", response);         | 지연 평가로 효율적                         |
| ✅ 좋은 예 | log.info("[INFO] {} {}", userId, userName); | 여러 변수도 깔끔하게 출력 가능             |

## ⚠️ 5. 예외(Exception) 로깅 주의점

예외는 단순히 getMessage()만 찍으면 안 됩니다.

❌ 잘못된 방식

```java
catch (Exception e) {
    log.error(e.getMessage());
}
```

→ 이렇게 하면 스택 트레이스(Stack Trace) 가 로그에 안 찍힙니다.

✅ 올바른 방식

```java
catch (Exception e) {
    log.error("S3 업로드 실패", e);
}
// 스택 트레이스(Stack Trace) 로그에 출력

// 또는
catch (Exception e) {
    log.error("S3 업로드 실패: {}", e.getMessage(), e);
}
// → 메시지 + 전체 예외 트레이스가 함께 출력됩니다.
```

## 💡 6. 로그 포맷(Pattern)에서 [INFO]는 굳이 안 써도 됨

아래처럼 직접 [INFO]를 넣는 경우도 종종 보이죠:

```java
log.info("[INFO] {} {} ", userId, userName);
```

하지만 Logback이나 Log4j 설정에서 이미 %level로 레벨을 출력하기 때문에 중복 표기될 수 있습니다.

예) logback-spring.xml

```xml
<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
```

출력 예:

```log
2025-10-31 14:35:02.123 [main] INFO  com.example.MyService - HR001 DEV01
```

---

## 8. 결론

- 항상 {} 플레이스홀더를 사용하자.
- 문자열 결합(+) 금지.
- 예외는 반드시 log.error("메시지", e)로 로그.
- [INFO] 등 레벨 표시는 로거 설정에 맡기자.
