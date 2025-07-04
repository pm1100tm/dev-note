# 🚀 JPA 엔티티의 Detached 상태가 되는 경우

## 📌 1. 트랜잭션이 종료되면서 Persistence Context가 닫힌 경우

✔️ findByUserId()로 조회한 ictInfo는 처음에는 Managed 상태

✔️ 하지만, 메서드 실행이 끝나면서 영속성 컨텍스트가 닫힘 → Detached 상태가 됨

✔️ 이후 변경을 해도 DB에 반영되지 않음!

```java
@Service
public class ICTInfoService {

    private final ICTInfoRepository ictInfoRepository;

    public void process(String userId, String newLocalIp) {
        // 🔹 1. 데이터베이스에서 조회
        ICTInfo ictInfo = ictInfoRepository.findByUserId(userId)
                .orElseThrow(() -> new IllegalArgumentException("ICTInfo not found: " + userId));

        // 🔹 2. 트랜잭션이 끝나면서 ictInfo는 Detached 상태가 됨
        // 이후 수정해도 DB에 반영되지 않음
        ictInfo.setLocalIp(newLocalIp);

        // ❌ save()를 하지 않으면 DB에 반영되지 않음
    }
}
```

## 📌 2. detach() 또는 clear()를 사용하여 강제로 Detached 상태로 변경

- ✔️ entityManager.detach(entity)를 호출하면 해당 엔티티가 Persistence Context에서 분리됨(Detached)

- ✔️ 이후 setLocalIp(newLocalIp)을 호출해도 DB에 반영되지 않음

```java
@Service
@RequiredArgsConstructor
public class ICTInfoService {

    private final ICTInfoRepository ictInfoRepository;
    private final EntityManager entityManager; // ✅ EntityManager 주입

    public void process(String userId, String newLocalIp) {
        // 🔹 1. 데이터 조회 (영속 상태)
        ICTInfo ictInfo = ictInfoRepository.findByUserId(userId)
                .orElseThrow(() -> new IllegalArgumentException("ICTInfo not found: " + userId));

        // 🔹 2. 강제로 Detached 상태로 변경
        entityManager.detach(ictInfo);

        // ❌ Detached 상태에서는 set()만으로 업데이트되지 않음!
        ictInfo.setLocalIp(newLocalIp);

        // save() 또는 merge() 필요
    }
}
```

## 📌 3. findById()로 가져온 후 다른 트랜잭션에서 수정하려는 경우

✔️ 첫 번째 findByUserId()로 조회한 ictInfo는 Detached 상태

✔️ updateIpInNewTransaction()에서 새로운 트랜잭션을 열어 같은 데이터를 수정

✔️ 그런데, 이전에 가져온 Detached 상태의 엔티티를 다시 save()하면 StaleObjectStateException 발생

```java
@Service
@RequiredArgsConstructor
public class ICTInfoService {

    private final ICTInfoRepository ictInfoRepository;

    public void process(String userId, String newLocalIp) {
        // 🔹 1. findById()를 호출한 후 메서드가 종료되면 트랜잭션이 종료됨 (Detached)
        ICTInfo ictInfo = ictInfoRepository.findByUserId(userId)
                .orElseThrow(() -> new IllegalArgumentException("ICTInfo not found: " + userId));

        // 다른 트랜잭션에서 변경 시도
        updateIpInNewTransaction(userId, newLocalIp);

        // ❌ 이 상태에서 다시 save() 하면 StaleObjectStateException 발생 가능
        ictInfoRepository.save(ictInfo);
    }

    @Transactional
    public void updateIpInNewTransaction(String userId, String newLocalIp) {
        ICTInfo ictInfo = ictInfoRepository.findByUserId(userId)
                .orElseThrow(() -> new IllegalArgumentException("ICTInfo not found: " + userId));

        ictInfo.setLocalIp(newLocalIp);
    }
}
```

### ✅ Detached 상태 해결 방법

- ✔️ 해결 방법 1: @Transactional 사용 (가장 추천!)
```java
@Transactional
public void process(String userId, String newLocalIp) {
    ICTInfo ictInfo = ictInfoRepository.findByUserId(userId)
            .orElseThrow(() -> new IllegalArgumentException("ICTInfo not found: " + userId));

    ictInfo.setLocalIp(newLocalIp); // ✅ Dirty Checking으로 자동 업데이트됨
}
```
- ✔️ 해결 방법 2: merge()를 사용하여 영속 상태로 변경
```java
public void process(String userId, String newLocalIp) {
    ICTInfo ictInfo = ictInfoRepository.findByUserId(userId)
            .orElseThrow(() -> new IllegalArgumentException("ICTInfo not found: " + userId));

    ICTInfo managedEntity = entityManager.merge(ictInfo); // ✅ merge()로 다시 Managed 상태로 변경
    managedEntity.setLocalIp(newLocalIp);
}
```
- ✔️ 해결 방법 3: save() 대신 saveAndFlush() 사용
```java
public void process(String userId, String newLocalIp) {
    ICTInfo ictInfo = ictInfoRepository.findByUserId(userId)
            .orElseThrow(() -> new IllegalArgumentException("ICTInfo not found: " + userId));

    ictInfo.setLocalIp(newLocalIp);
    ictInfoRepository.saveAndFlush(ictInfo); // ✅ 즉시 DB 반영
}
```

## 참고: JPA에서 엔티티는 4가지 상태를 가집니다.

|상태|설명|
|---|---|
|Transient (비영속, New)|DB에 저장되지 않은 새로운 객체|
|Managed (영속, Persistent)|JPA가 관리하는 상태|
|Detached (준영속, Detached)|트랜잭션 종료 후 관리되지 않는 상태|
|Removed (삭제, Removed)|삭제된 상태|
