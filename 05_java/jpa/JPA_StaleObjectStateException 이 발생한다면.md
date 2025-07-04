# 🚀 StaleObjectStateException 이 발생한다면

StaleObjectStateException 은 Hibernate 에서 동시에 같은 엔티티를 수정하려고 할 때 발생하는
동시성 충돌 문제입니다.

즉, A 트랜잭션에서 데이터를 수정하는 동안, B 트랜잭션이 같은 데이터를 수정하려고 하면 충돌이 발생할 수
있습니다.

## 주요 발생 상황

### ✅ 1. Optimistic Locking

- @Version 필드를 가진 엔티티를 다른 트랜잭션에서 동시에 수정하려고 할 때 발생.

✔️ 예: 사용자가 같은 데이터를 동시에 수정하는 경우.

### ✅ 2. Detached Entity 문제

- findById() 또는 findByUserId() 등으로 조회한 엔티티가 Detached 상태에서 업데이트될 경우 발생.
- JPA가 관리하는 Persistence Context 밖에서 수정된 엔티티를 save() 호출 시 merge() 과정에서 충돌 발생.

### ✅ 3. Long-running Transactions (긴 트랜잭션)

같은 데이터를 오랫동안 수정하는 트랜잭션이 있을 때, 다른 트랜잭션에서 변경이 발생하면 예외 발생.

### ✅ 4. 다중 요청 처리 (Concurrency Issues)

두 개 이상의 트랜잭션이 거의 동시에 같은 데이터를 수정하면 StaleObjectStateException이 발생 가능.


---

## 🥲 위의 예시 중에 내가 겪었던 것은 [2. Detached Entity] 문제..

ING
