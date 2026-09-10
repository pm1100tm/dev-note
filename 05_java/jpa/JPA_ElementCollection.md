# 🚀 ElementCollection

예제로 보는 ElementCollection 어노테이션 사용

```java
@ElementCollection(fetch = FetchType.EAGER)
@CollectionTable(
    name = "user_terms",
    joinColumns = @JoinColumn(name = "user_info_id")
)
private List<UserTerms> userTerms;
```

이 코드는 JPA에서 UserInfo 엔티티가 user_terms 테이블에 매핑된 복합 값 타입(컬렉션) 을 포함하고 있음을 의미한다.


## ✅ 1. @ElementCollection

- 역할: 이 필드가 엔티티가 아닌 값 타입의 컬렉션임을 JPA에 알린다.
- `UserTerms`는 별도 엔티티가 아니라 `@Embeddable`로 정의한 값 객체여야 한다.
- DB에는 별도 테이블로 저장되지만, JPA에서는 부모 엔티티(`UserInfo`)에 소속된 데이터로 취급한다.

```java
@Embeddable
public class UserTerms {
    private String termsType;
    private boolean agreed;
    ...
}
```

## ✅ 2. @CollectionTable

- 역할: `@ElementCollection` 필드가 저장될 테이블을 지정한다.
- `name = "user_terms"`: 값 객체를 저장할 테이블 이름이다.
- `joinColumns = @JoinColumn(name = "user_info_id")`: 부모 엔티티와의 연결을 지정한다.
- `user_terms` 테이블은 `user_info_id`를 외래키로 가지며 `UserInfo.userId`와 연결된다.

## ✅ 3. fetch = FetchType.EAGER

- 역할: `userTerms` 필드를 즉시 로딩한다는 뜻이다.
- `UserInfo` 엔티티를 조회하면 `user_terms` 테이블의 값도 함께 조회한다.

```sql
SELECT * FROM user_info;
SELECT * FROM user_terms WHERE user_info_id = ?;
```

- `EAGER`는 데이터 양이 많으면 성능을 떨어뜨릴 수 있으므로, 필요에 따라 `LAZY`를 사용한다.

## ✅ 4. 전체 동작 흐름 요약

1.	UserInfo 엔티티를 저장하거나 조회하면,
2.	JPA는 user_terms 필드를 user_terms 테이블에 매핑하고,
3.	user_info_id를 외래키로 연결하여 insert/select/delete 작업을 처리합니다.
4.	UserTerms 클래스는 엔티티가 아닌 값 타입이므로 @Entity가 아닌 @Embeddable로 선언됩니다.


---
## ✳️ 추가 팁

- `@ElementCollection`은 주로 값 객체의 `List`, `Set`, `Map`에 사용한다.
- 부모 엔티티 생명 주기에 따라 삭제·삽입되므로, 값 객체는 식별자를 가질 수 없다.
