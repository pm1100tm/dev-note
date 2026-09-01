# 🚀 Cache 제거와 무효화의 차이

## 🧩 Cache Eviction (캐시 제거)

> ✅ “캐시 서버가 자체 정책에 따라 데이터를 지워버리는 것.”

예시:

- TTL(Time-to-Live) 만료
- LRU(Least Recently Used) 정책 적용
- 메모리 부족 발생 시 자동 제거

Cache Eviction (자동 제거: TTL 기반):

```java
@Cacheable(value = "user", key = "#id")
public User getUser(String id) {
    return userRepository.findById(id);
}

// Redis 설정에서 TTL 5분 지정
spring.cache.redis.time-to-live=300000
```

---

## 🧩 2️⃣ Cache Invalidation (캐시 무효화)

> ✅ “원본 데이터가 변경되었을 때, 그 데이터를 참조하는 캐시를 직접 지워서 무효화하는 것.”

- 이건 데이터 일관성(Consistency) 을 보장하기 위한 동작입니다.
- 즉, “캐시에 있는 데이터가 이제 더 이상 최신이 아니야”라고 명시적으로 알려주는 것.

Cache Invalidation (수동 무효화):

```java
@CacheEvict(value = "user", key = "#user.id")
public void updateUser(User user) {
    userRepository.save(user); // DB 업데이트
}
```
