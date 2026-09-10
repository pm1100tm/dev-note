# 🚀 낙관락(Optimistic Lock) Best Practice

❗ 기존 코드의 문제점

```java
@Transactional
public void decreaseStock(Long productId, int quantity) {
    for (int i = 0; i < 3; i++) {
        try {
            Product product = repository.findById(productId)
                .orElseThrow();

            product.decrease(quantity);
            return;
        } catch (OptimisticLockException e) {
            // retry
        }
    }
}
```

👉 문제:

- 같은 트랜잭션 안에서 반복됨
- 이미 실패한 영속성 컨텍스트 재사용
- retry가 제대로 동작하지 않음

<br>

## 🚀 해결 전략 (실무 핵심)

👉 retry마다 새로운 트랜잭션을 사용해야 합니다

### ✅ 실무용 구조 (Best Practice)

1️⃣ Service 분리 (핵심)

```java
@Service
@RequiredArgsConstructor
public class ProductService {

    private final ProductInternalService internalService;

    public void decreaseStock(Long productId, int quantity) {

        int maxRetry = 3;

        for (int i = 0; i < maxRetry; i++) {
            try {
                internalService.decreaseStock(productId, quantity);
                return;
            } catch (OptimisticLockException e) {

                // 👉 로그 남기기 (실무 중요)
                log.warn("OptimisticLockException 발생 → retry: {}", i + 1);

                // 👉 짧은 대기 (충돌 완화)
                sleep(50);
            }
        }

        throw new RuntimeException("재시도 실패");
    }

    private void sleep(long millis) {
        try {
            Thread.sleep(millis);
        } catch (InterruptedException ignored) {
        }
    }
}
```

2️⃣ 실제 트랜잭션 처리 (분리된 Bean)

```java
@Service
@RequiredArgsConstructor
public class ProductInternalService {

    private final ProductRepository repository;

    @Transactional
    public void decreaseStock(Long productId, int quantity) {

        Product product = repository.findById(productId)
            .orElseThrow();

        product.decrease(quantity);
    }
}
```

#### 🔍 retry가 제대로 동작하는 이유

```java
internalService.decreaseStock(...)
```

👉 호출마다:

- 새로운 트랜잭션 생성
- 새로운 영속성 컨텍스트
- 최신 데이터 다시 조회

<br>

## 📌 더 좋은 코드 (실무 개선 버전)

```java
public void decreaseStock(Long productId, int quantity) {

    int maxRetry = 3;

    for (int i = 0; i < maxRetry; i++) {
        try {
            internalService.decreaseStock(productId, quantity);
            return;
        } catch (OptimisticLockException e) {

            long backoff = (long) Math.pow(2, i) * 50;

            log.warn("retry={}, backoff={}ms", i + 1, backoff);

            sleep(backoff);
        }
    }

    throw new RuntimeException("재시도 실패");
}
```

🔍 왜 좋은가?

- 충돌 많은 상황에서 효과적
- DB 부하 감소
- retry 성공률 증가

<br>

## 📌 실무에서 더 많이 쓰는 방식 (참고)

👉 직접 retry 구현 대신 라이브러리 사용

예: Spring Retry

```java
@Entity
@Getter
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class Product {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private int stock;

    /**
     * 👉 낙관락 핵심 필드
     */
    @Version
    private Long version;

    public Product(int stock) {
        this.stock = stock;
    }

    /**
     * 재고 감소 로직
     */
    public void decrease(int quantity) {
        if (this.stock < quantity) {
            throw new IllegalArgumentException("재고 부족");
        }
        this.stock -= quantity;
    }
}
```

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class ProductService {

    private final ProductInternalService internalService;

    /**
     * 👉 Retry 담당 (트랜잭션 없음)
     */
    @Retryable(
        value = OptimisticLockException.class,
        maxAttempts = 3,
        backoff = @Backoff(delay = 50, multiplier = 2)
    )
    public void decreaseStock(Long productId, int quantity) {
        internalService.decreaseStock(productId, quantity);
    }

    /**
     * 👉 Retry 실패 시 fallback
     * Spring Retry가 내부적으로 자동 호출
     * 1차 시도 → 실패
     * 2차 시도 → 실패
     * 3차 시도 → 실패
     * 👉 이제 retry 끝
     * → @Recover 호출
     */
    @Recover
    public void recover(OptimisticLockException e, Long productId, int quantity) {
        log.error("재시도 실패: productId={}", productId);
        throw new RuntimeException("재시도 실패", e);
    }
}
```

```java
@Service
@RequiredArgsConstructor
public class ProductInternalService {

    private final ProductRepository repository;

    /**
     * 👉 실제 DB 작업
     */
    @Transactional
    public void decreaseStock(Long productId, int quantity) {

        Product product = repository.findById(productId)
            .orElseThrow();

        product.decrease(quantity); // 👈 핵심. Dirty Checking
    }
}
```

📌 핵심 요약

```markdown
- @Recover는 직접 호출하지 않는다
- retry 실패 시 Spring이 자동 호출한다
- fallback 처리용 메서드이다
- 프록시 기반 AOP로 동작한다
```
