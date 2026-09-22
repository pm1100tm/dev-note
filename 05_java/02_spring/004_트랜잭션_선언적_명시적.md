# 트랜잭션 - 선언적 트랜잭션, 명시적 트랜잭션

## 선언적 트랜잭션

@Transactional 어노테이션을 활용한다.

### 1. public 이외의 접근 제어자는 해당 어노테이션이 동작하지 않음

```java
public class PracTransaction {
 @Transactional
 public void doSomething() {
    // ... works
  }

  @Transactional
  protected void doSomething2() {
    // ... not works
  }

  @Transactional
  private void doSomething3() {
    // ... not works
  }
}
```

### 2. 동일한 클래스 내 @Transactional 이 선언되지 않은 메소드애서, 선언된 메소드를 호출하면 작동하지 않음

Spring 에서 Transactional 어노테이션은 AOP를 통해서 동작하기 때문이고, 클래스 A 라는 것이
Bean 으로 등록이 되는데, 그럴 때는 doSomething 라는 것이 Proxy 메소드가 되어서 나타나지 않는다.

```java
public class A {

 public void doSomething() {
    // not works
    logic1();
    logic2();
  }

  @Transactional
  protected void logic1() {
    // ...
  }

  @Transactional
  private void logic2() {
    // ...
  }
}
```

### 3. Unchecked Exception(Runtime Exception)만 관리함

checked exception(IOException)은 관리하지 않는다. 따라서 Unchecked Exception은 롤백
가능하지만, checked exception은 롤백이 불가능하다.

> Transactional(rollback = Exception.class) 등으로 롤백하게 할 수도 있다.

### 4. Transactional 을 선언한 메소드가 부모 메소드가 됨

부모 메소드에 @Transactional 을 선언하면, 전체 트랜잭션이 부모 메소드에 대한 트랜잭션으로 묶인다.

```java
public class A {

 @Transactional
 public void doSomething() {
    // works
    logic1();
  }

  protected void logic1() {
    // ...
  }
}
```

### 4. 트랜잭션 전파

트랜잭션A 내에서 트랜잭션B가 호출되는 것을 의미한다. 이 때 어떻게 전파가 될 것인가를 정하는 옵션이
다수 존재한다.

- PROPAGATION_REQUIRED

  - 기본 값이며, 부모 트랜잭션이 존재하면 해당 트랜잭션에 합류한다. 없다면 새로운 트랜잭션을 시작
  - 자식 트랜잭션이 성공해도 부모 트랜잭션이 실패하면 모두 실패

- PROPAGATION_REQUIRES_NEW

  - 부모 트랜잭션과 별개의 트랜잭션을 생성한다.
  - 서로 독립적으로 동작한다.

- PROPAGATION_NEVER
  - 부모 트랜잭션이 존재하면 예외를 던진다.

그 외

- PROPAGATION_SUPPORTS
- PROPAGATION_MANDATORY
- PROPAGATION_NOT_SUPPORTED

등이 있다.

### 5. 트랜잭션 어노테이션은 프록시 방식으로 동작하기 떄문에, 클래스 외부에서 호출하는 경우에만 동작

어노테이션 방식은 한계가 존재하는 상황이 있다. 따라서, 메소드 내 특정 구간에만 트랜잭션을 명시적으로
선언할 때에는, 프로그래밍적인 방법이 좋다. 또한, 개발자가 트랜잭션의 시작과 종료를 명시적으로 결정할
수 있다.

## 명시적 트랜잭션

TransactionTemplate 를 사용하고, PlatformTransactionManager 구현체를 직접 사용한다.

TODO!
