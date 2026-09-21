# 🚀 Java String.toString() vs String.valueOf() 차이 완벽 정리

Java 개발을 하다 보면 객체를 문자열로 변환해야 하는 상황이 매우 자주 발생합니다.
이때 흔히 사용하는 두 가지 방법이 있습니다.

```java
obj.toString();
String.valueOf(obj);
```

겉보기에는 동일해 보이지만, 실제로는 안정성과 실무 활용 측면에서 매우 중요한 차이가 있습니다.
이번 글에서는 그 차이를 정확하게 설명드립니다.

## 📌 핵심 차이 요약

| 구분        | toString()      | String.valueOf() |
| ----------- | --------------- | ---------------- |
| 호출 방식   | 인스턴스 메서드 | static 메서드    |
| null 처리   | ❌ NPE 발생     | ✅ "null" 반환   |
| 안정성      | 낮음            | 높음             |
| 실무 사용성 | 제한적          | 매우 높음        |

### 📌 1. toString() 동작 방식

```java
Object obj = null;
String result = obj.toString(); // ❌ NullPointerException 발생
```

### 📌 2. String.valueOf() 동작 방식

```java
Object obj = null;
String result = String.valueOf(obj); // "null"
```

#### 🔍 내부 구현

```java
public static String valueOf(Object obj) {
    return (obj == null) ? "null" : obj.toString();
}
```

📌 특징 정리

- 내부적으로 null 체크를 수행합니다
- null이면 "null" 문자열을 반환합니다
- 절대 NullPointerException이 발생하지 않습니다

### 📌 실무에서 자주 사용하는 방식

✅ 결론: String.valueOf()를 사용합니다

```java
String userId = String.valueOf(user.getId());
```

#### 🔍 왜 이렇게 사용할까요? ->

1. null-safe (가장 중요합니다)

- DB 값이 null일 가능성
- 외부 API 응답이 null일 가능성
- 리팩토링 중 null이 들어올 가능성

2. 로그 안정성 확보

```java
log.info("userId = {}", String.valueOf(user.getId()));
```

- 로그는 절대 실패하면 안 됩니다
- valueOf는 null이어도 안전하게 출력됩니다

3. 유지보수성

- 코드 작성 시점에는 null이 아니어도
- 이후 코드 변경으로 null이 들어올 수 있습니다

👉 valueOf는 이런 변화에도 안전합니다

### 📌 성능 관점에서의 차이

👉 성능 차이는 거의 없습니다

String.valueOf()는 내부적으로 다음과 같이 동작합니다:

```java
(obj == null) ? "null" : obj.toString();
```

- null 체크 1회
- 조건 분기 1회

👉 CPU 비용은 매우 미미합니다

#### 📌 핵심 포인트

- 성능보다 안정성이 훨씬 중요합니다

---

## ✅ 상황별 더 좋은 코드

1. null을 다른 값으로 치환해야 할 때

```java
String id = Optional.ofNullable(user.getId())
    .map(String::valueOf)
    .orElse("UNKNOWN");
```

2. 문자열 결합

```java
String result = "ID: " + user.getId();
```

👉 내부적으로 String.valueOf()가 호출됩니다

3. primitive 처리 차이

```java
String.valueOf(123); // OK
```

👉 primitive도 바로 처리 가능합니다
