# 🚀 Java Long.valueOf() vs Long.parseLong() 차이 완벽 정리

Java에서 문자열을 숫자로 변환할 때 다음 두 가지 메서드를 많이 사용합니다.

```java
Long.valueOf("123");
Long.parseLong("123");
```

겉보기에는 비슷하지만, 리턴 타입, 내부 동작, 성능 측면에서 중요한 차이가 존재합니다.
이번 글에서는 이 차이를 실무 기준으로 명확하게 설명드립니다.

## 📌 핵심 차이 요약

| 구분      | Long.valueOf()   | Long.parseLong() |
| --------- | ---------------- | ---------------- |
| 반환 타입 | Long (객체)      | long (primitive) |
| 객체 생성 | 있음 (또는 캐싱) | 없음             |
| 성능      | 상대적으로 느림  | 더 빠름          |
| null 처리 | ❌ NPE           | ❌ NPE           |
| 실무 사용 | 상황에 따라 선택 | 기본 선택        |

<br>

## 📌 1. Long.parseLong() 동작 방식

```java
long value = Long.parseLong("123");
```

🔍 특징

- 문자열 → primitive long으로 변환
- 불필요한 객체 생성 없음
- 가장 가볍고 빠른 방식

📌 내부 동작

```java
return parseLong(s, 10);
```

👉 순수하게 숫자 파싱만 수행합니다.

<br>

## 📌 2. Long.valueOf() 동작 방식

```java
Long value = Long.valueOf("123");
```

🔍 내부 구현 (핵심)

```java
public static Long valueOf(String s) {
    return Long.valueOf(parseLong(s));
}
```

📌 추가 특징 (중요)

```java
Long.valueOf(127) == Long.valueOf(127); // true
Long.valueOf(128) == Long.valueOf(128); // false
```

👉 -128 ~ 127 범위는 캐싱됩니다

<br>

## 📌 성능 관점 비교

👉 Long.parseLong()이 더 빠릅니다

🔍 이유

### 1. parseLong

```java
long value = Long.parseLong("123");
```

- primitive 반환
- 객체 생성 없음
- GC 부담 없음

### 2. valueOf

```java
Long value = Long.valueOf("123");
```

- 내부적으로 parseLong 호출
- 이후 Long 객체 생성 또는 캐싱 확인

👉 추가 비용 발생

### 📌 성능 요약

| 항목      | parseLong | valueOf |
| --------- | --------- | ------- |
| 객체 생성 | 없음      | 있음    |
| GC 영향   | 없음      | 있음    |
| 속도      | 빠름      | 느림    |

<br>

## 📌 실무에서 자주 사용하는 방식

### ✅ 1. 기본 원칙

👉 가능하면 parseLong()을 사용합니다

```java
long userId = Long.parseLong(request.getUserId());
```

🔍 이유

- 대부분의 경우 primitive로 충분합니다
- 불필요한 객체 생성 방지
- 성능 최적화

### ✅ 2. 객체가 필요한 경우만 valueOf()

```java
Long userId = Long.valueOf(request.getUserId());
```

🔍 사용 사례

- 컬렉션 (List, Map<Long, …>)
- JPA Entity 필드
- nullable 값 처리

#### ✅ 더 좋은 코드

```java
long userId = Long.parseLong(request.getUserId());
```

#### ✅ 객체가 필요할 때만

```java
Long userId = Long.parseLong(request.getUserId()); // auto-boxing
```

👉 JVM이 자동으로 boxing 수행

<br>

---

## 📌 예외 처리 (실무 중요)

두 메서드 모두 동일하게 예외를 발생시킵니다.

```java
Long.parseLong("abc"); // NumberFormatException
Long.valueOf("abc");   // NumberFormatException
```

### ✅ 안전한 처리

```java
public long parseLongSafe(String value) {
    try {
        return Long.parseLong(value);
    } catch (NumberFormatException e) {
        return 0L;
    }
}
```
