# 🚀 Java Enum 비교에 == 연산자를 사용해야 하는 이유.md

Java에서 enum은 상수를 표현할 때 자주 쓰입니다. 예를 들어 회원 상태, 성별, 권한 등을 enum으로
표현하면 코드가 훨씬 명확해집니다.

하지만 많은 개발자들이 한 번쯤은 이렇게 씁니다 👇

```java
if (member.getStatus().equals(MemberStatus.ACTIVE)) {
    ...
}
```

겉보기엔 문제없지만, 사실 == 연산자로 비교하는 것이 더 정확하고 안전한 방법입니다.
오늘은 그 이유를 자세히 정리해보겠습니다.

---

## 🧩 1. Enum은 “싱글턴(Singleton)“이다

Java의 enum은 단순한 상수 집합이 아니라 JVM이 클래스처럼 관리하는 단일 인스턴스 객체입니다.

예를 들어 다음 코드가 있다면:

```java
public enum MemberStatus {
    ACTIVE,
    INACTIVE,
    BANNED
}
```

컴파일 시점에 JVM 내부에서는 다음과 유사한 코드가 만들어집니다.

```java
public final class MemberStatus extends Enum<MemberStatus> {
    public static final MemberStatus ACTIVE = new MemberStatus("ACTIVE", 0);
    public static final MemberStatus INACTIVE = new MemberStatus("INACTIVE", 1);
    public static final MemberStatus BANNED = new MemberStatus("BANNED", 2);
}
```

즉, MemberStatus.ACTIVE 는 프로그램 전체에서 딱 하나만 존재하는 인스턴스입니다.
따라서 동일한 enum 상수는 항상 같은 메모리 주소를 가리키게 됩니다.

## ⚖️ 2. equals() vs == 비교 방식

- `==`: 동일한 인스턴스를 비교한다. enum 상수는 단일 인스턴스이므로 정확하고 null에도 안전하다.
- `equals()`: 값 비교에는 맞지만 `null.equals(...)` 호출 시 예외가 나며 enum에서는 불필요하다.

## 🖥️ 3. 예제 코드

```java
// Member
public class Member {

    private final String name;
    private final MemberStatus status;

    public Member(String name, MemberStatus status) {
        this.name = name;
        this.status = status;
    }

    public MemberStatus getStatus() {
        return status;
    }
}

// MemberStatus
public enum MemberStatus {
    ACTIVE,
    INACTIVE,
    BANNED
}
```

### 🚫 4. 잘못된 비교 방식 (equals() 사용)

```java
Member member = new Member("Alice", MemberStatus.ACTIVE);

if (member.getStatus().equals(MemberStatus.ACTIVE)) {
    System.out.println("활성 회원입니다.");
}
```

이 코드, 얼핏 보기엔 문제 없어 보이지만, member.getStatus()가 null일 경우
NPE(NullPointerException) 가 발생합니다.

### ✅ 5. 올바른 비교 방식 (== 사용)

```java
Member member = new Member("Alice", MemberStatus.ACTIVE);

if (member.getStatus() == MemberStatus.ACTIVE) {
    System.out.println("활성 회원입니다.");
}
```

👍 이렇게 하면 좋은 이유

- Enum은 싱글턴이므로, == 비교가 완벽히 동작
- null이어도 안전하게 false로 처리
  - → member.getStatus()가 null이어도 NPE 없이 그냥 비교 결과는 false
- equals() 호출 오버헤드가 없음

### ⚡ 6. null 방어가 꼭 필요하다면

가끔 외부 데이터 등으로 인해 null 가능성이 높을 때는 이렇게 써도 됩니다 👇

```java
if (MemberStatus.ACTIVE.equals(member.getStatus())) {
    System.out.println("활성 회원입니다.");
}
```

이건 equals()를 뒤집은 형태로 왼쪽(MemberStatus.ACTIVE)이 null이 아니므로
NPE 없이 안전하게 동작합니다. 하지만 JPA 엔티티의 enum 필드처럼 nullable = false라면
굳이 이렇게 쓸 필요는 없습니다.

### 🧩 7. 성능 비교

테스트 코드를 보면 차이는 미미하지만 equals()는 메서드 호출, ==은 단순 주소 비교입니다.

```java
if (status == MemberStatus.ACTIVE) {
    // JVM: 한 번의 메모리 비교 연산
}

if (status.equals(MemberStatus.ACTIVE)) {
    // JVM: equals() 호출 → 오버헤드 약간
}
```

---

## ✨ 마무리

💬 “Java에서 enum은 ‘값 비교’가 아니라 ‘인스턴스 동일성 비교’다.”

그래서 ✅ ==를 쓰는 게 정확하고 안전합니다.

- equals()는 null 위험 + 의미 중복이므로 피하는 것이 좋습니다.

---

## 🧩 1️⃣ equals()를 반드시 사용해야 하는 경우

==는 참조(주소값) 비교, equals()는 내용(값) 비교를 합니다.

따라서 동일한 객체가 아닌데도 값만 같을 수 있는 경우, equals()를 써야 합니다.

### ✅ 예시 1: String 비교

```java
String a = new String("hello");
String b = new String("hello");

System.out.println(a == b);      // ❌ false (서로 다른 객체)
System.out.println(a.equals(b)); // ✅ true (내용이 동일)
```

→ 문자열 리터럴이 아닌 동적 문자열이나 DB, JSON, 요청 파라미터에서 넘어온 경우엔 항상 equals()로
비교해야 합니다.

### ✅ 예시 2: Integer, Long 등 Wrapper 타입

```java
Integer a = 1000;
Integer b = 1000;

System.out.println(a == b);      // ❌ false (다른 인스턴스)
System.out.println(a.equals(b)); // ✅ true
```

💡 이유:

- 자바는 -128~127 범위의 숫자만 캐싱하고, 그 외에는 새 객체를 생성합니다.
- 따라서 Wrapper 비교는 항상 equals()를 사용해야 합니다.

### ✅ 예시 3: 커스텀 VO, DTO 클래스

```java
public class Address {
    private String city;
    private String street;

    // equals/hashCode 재정의 필요
    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Address)) return false;
        Address address = (Address) o;
        return Objects.equals(city, address.city)
                && Objects.equals(street, address.street);
    }

    @Override
    public int hashCode() {
        return Objects.hash(city, street);
    }
}
```

```java
Address a = new Address("Seoul", "Gangnam");
Address b = new Address("Seoul", "Gangnam");

System.out.println(a == b);      // ❌ false
System.out.println(a.equals(b)); // ✅ true (내용 동일)
```

즉, 값 객체(Value Object), DTO, Wrapper, String 등은 equals()로 “내용이 같은지”를
비교해야 합니다.

### 🧠 2️⃣ JPA 엔티티 비교의 핵심

- JPA에서는 엔티티는 “식별자(ID)”로 동일성을 판단합니다.
- 하지만 주의할 점이 몇 가지 있습니다.

⚠️ 잘못된 비교

```java
Member m1 = memberRepository.findById("USER_1").get();
Member m2 = memberRepository.findById("USER_1").get();

System.out.println(m1 == m2);      // ❌ false (다른 영속성 컨텍스트면 다른 객체)
System.out.println(m1.equals(m2)); // ❌ 기본 Object.equals() → false
```

→ 영속성 컨텍스트가 다르면(예: 트랜잭션 다름)

- 같은 id를 가진 엔티티라도 == 비교는 실패합니다.

#### ✅ 올바른 방법: 엔티티의 equals()/hashCode() 재정의

엔티티는 보통 식별자(ID) 기준으로 equals()를 재정의합니다.

```java
@Entity
public class Member {

    @Id
    private String id;

    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Member)) return false;
        Member member = (Member) o;
        return id != null && id.equals(member.id);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}
```

이제 다음 코드에서:

```java
Member m1 = memberRepository.findById("USER_1").get();
Member m2 = memberRepository.findById("USER_1").get();

System.out.println(m1.equals(m2)); // ✅ true
```

→ ID가 같으면 동일한 엔티티로 인식됩니다.

### 🧩 3️⃣ DTO 비교는 “내용” 기준으로

DTO는 주로 요청/응답 데이터를 담는 값 객체이므로 항상 equals() 기반 비교를 해야 합니다.

```java
public class MemberDto {
    private String id;
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof MemberDto)) return false;
        MemberDto that = (MemberDto) o;
        return Objects.equals(id, that.id)
                && Objects.equals(name, that.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }
}
```
