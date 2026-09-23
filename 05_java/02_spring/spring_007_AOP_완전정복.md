# Spring AOP 완전 정복

Spring AOP는 **여러 클래스에 흩어진 공통 관심사를 한곳에 모아 메서드 실행 전후에 적용**하는 방법입니다.

호출 시간 측정·로깅·트랜잭션에 쓰이며, 여기서는 Spring Boot의 프록시 기반 AOP를 설명합니다.

> 예제의 패키지와 도메인 이름은 설명용이며 Spring Boot 4.0.1 기준입니다.

## AOP가 필요한 이유

서비스마다 실행 시간을 기록하려고 같은 `try/finally` 코드를 넣으면 **핵심 업무 코드가 흐려지고** 수정할
곳도 늘어납니다. AOP는 적용 대상과 부가 기능을 따로 정의합니다.

```text
Controller → [프록시: 시간 측정] → Service → Repository
```

프록시는 호출을 가로채 부가 기능을 수행한 뒤 실제 빈의 메서드를
호출합니다. 프록시를 거치지 않는 호출에는 해당 부가 기능이 적용되지
않습니다.

## 핵심 용어

| 용어        | 의미                                        | 예                       |
| ----------- | ------------------------------------------- | ------------------------ |
| 횡단 관심사 | 여러 기능에 공통으로 필요한 처리            | 실행 시간 기록           |
| Aspect      | 적용 대상과 처리 코드를 묶은 클래스         | `ServiceTimingAspect`    |
| Join point  | 부가 기능을 적용할 수 있는 지점             | Spring AOP의 메서드 실행 |
| Pointcut    | 실제 적용 대상을 고르는 조건                | 서비스 메서드 선택       |
| Advice      | 선택한 지점에서 실행할 코드                 | 호출 전후 시간 측정      |
| Target      | 부가 기능을 적용받는 실제 빈                | `OrderService`           |
| Proxy       | 호출을 받아 Advice와 Target을 연결하는 객체 | 서비스 프록시            |

Spring AOP의 Join point는 Spring 빈의 메서드 실행으로 제한됩니다.
생성자 호출, 필드 읽기, 일반 객체의 메서드 호출까지 가로채는 방식은
아닙니다. [Spring AOP 개념 문서](https://docs.spring.io/spring-framework/reference/core/aop/introduction-defn.html)를 참고하세요.

## Advice와 Pointcut

| Advice            | 실행 시점         | 대표 용도            |
| ----------------- | ----------------- | -------------------- |
| `@Before`         | 메서드 실행 전    | 간단한 사전 검사     |
| `@AfterReturning` | 정상 반환 후      | 성공 결과 기록       |
| `@AfterThrowing`  | 예외 발생 후      | 실패 기록            |
| `@After`          | 정상·예외 종료 후 | 종료 처리            |
| `@Around`         | 실행 전후 전체    | 시간 측정, 결과 제어 |

`@Around`는 `ProceedingJoinPoint.proceed()`를 호출해야 대상 메서드가
실행됩니다. 반환값과 예외도 호출자에게 전달해야 계약이 유지됩니다.
필요한 기능을 만족하는 가장 단순한 Advice를 선택하는 편이 좋습니다.
[Spring Advice 문서](https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/advice.html)에서 각 시점과 반환 규칙을 확인할 수 있습니다.

자주 쓰는 Pointcut 표현식은 다음과 같습니다.

| 표현식                                                         | 선택 대상                            |
| -------------------------------------------------------------- | ------------------------------------ |
| `execution(* com.myblog.backend.application.service..*.*(..))` | 서비스 패키지와 하위 패키지의 메서드 |
| `within(com.myblog.backend.adapter.in.web..*)`                 | 웹 어댑터 패키지의 타입              |
| `@annotation(com.myblog.backend.config.observability.Timed)`   | `@Timed`가 붙은 메서드               |

`execution`의 `*`는 반환 타입, `..`는 패키지 경로나 인자 목록에서
여러 항목을 허용한다는 뜻입니다. 패키지 구조가 바뀌면 넓은
Pointcut은 의도하지 않은 메서드까지 잡을 수 있으므로 범위를
검토해야 합니다. [Pointcut 공식 문서](https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/pointcuts.html)를 참고하세요.

## 실행 시간 측정 예제

전제 조건은 Spring Boot 프로젝트에 Spring AOP 관련 의존성이 있고, 아래 클래스가 컴포넌트 스캔
범위에 포함되는 것입니다. 의존성 구성과 자동 설정 방식은 사용 중인 Boot 버전에 맞춰 확인합니다.

다음 파일을 `config/observability/ServiceTimingAspect.java`에 만듭니다.
예제는 서비스 호출의 소요 시간을 기록하고 반환값과 예외를 그대로 전달합니다.

```java
package com.myblog.backend.config.observability;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Component;

@Aspect
@Component
public class ServiceTimingAspect {

    private static final Logger log =
            LoggerFactory.getLogger(ServiceTimingAspect.class);

    @Around("execution(* com.myblog.backend.application.service..*.*(..))")
    public Object measure(ProceedingJoinPoint joinPoint) throws Throwable {
        long started = System.nanoTime();
        try {
            return joinPoint.proceed();
        } finally {
            long elapsed = System.nanoTime() - started;
            log.debug("{} took {} ns", joinPoint.getSignature(), elapsed);
        }
    }
}
```

`@Aspect`만 붙여서는 스프링 빈이 되지 않습니다. `@Component` 또는
`@Bean` 등록이 필요합니다. Spring Boot는 AspectJ 관련 의존성이
클래스패스에 있으면 Aspect 프록시 자동 설정을 제공합니다. 보통
별도의 `@EnableAspectJAutoProxy` 선언은 필요하지 않습니다.
[Spring Boot AOP 문서](https://docs.spring.io/spring-boot/reference/features/aop.html)를 확인하세요.

동작 확인 시에는 다른 빈에서 서비스의 공개 메서드를 호출하고 로그를 살펴봅니다. 이 예제는 실제 프로젝트에
추가하거나 실행하지 않았으므로 로그 출력 결과는 확인된 사실로 취급하지 않습니다.

## 프록시 방식과 놓치기 쉬운 호출

Spring AOP는 JDK 동적 프록시 또는 클래스 기반 프록시를 사용합니다.
Spring Boot의 기본 AOP 자동 설정은 클래스 기반 프록시를 사용하며,
`spring.aop.proxy-target-class=false`로 JDK 프록시를 선택할 수
있습니다. JDK 프록시는 구현한 인터페이스를 통해 호출합니다.
클래스 기반 프록시는 상속을 사용하므로 `final` 클래스·메서드와
`private` 메서드에는 Advice를 적용할 수 없습니다.
[프록시 공식 문서](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)를 참고하세요.

```java
public void placeOrder() {
    validateOrder(); // 같은 객체 내부 호출: 프록시를 거치지 않음
}

public void validateOrder() {
    // 이 메서드에만 붙인 Advice는 위 호출에 적용되지 않음
}
```

같은 객체의 메서드를 `this`로 호출하면 프록시를 거치지 않습니다.
`@Transactional`도 같은 제약을 받습니다. 필요하다면 별도 빈으로
책임을 분리해 외부에서 호출합니다. 여러 Aspect가 한 메서드에
적용되면 `@Order`로 우선순위를 지정합니다. 숫자가 낮은 Advice가
먼저 진입하고 나갈 때는 나중에 종료합니다.
[Advice 순서 문서](https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/advice.html)를 참고하세요.

## DDD와 Hexagonal 아키텍처에서 선언할 위치

질문의 구조에서는 Aspect가 사용하는 기술과 적용할 경계에 따라 위치를 정합니다. `domain/`은 순수
Java로 유지하므로 Spring AOP 어노테이션과 프록시에 의존하지 않습니다. 비즈니스 불변식은 Aspect가
아니라 도메인 모델이나 도메인 서비스에 명시적으로 구현합니다.

| 관심사                            | Aspect 위치                                          | 적용 경계               |
| --------------------------------- | ---------------------------------------------------- | ----------------------- |
| 웹 요청·세션·웹 권한 처리         | `adapter/in/web/aspect/` 또는 `adapter/in/web/auth/` | Controller 등 웹 어댑터 |
| 영속성 오류 기록·어댑터 전용 캐시 | `adapter/out/persistence/` 또는 `adapter/out/cache/` | 해당 아웃바운드 어댑터  |
| 유스케이스 실행 추적·공통 로깅    | `config/observability/`, `config/logging/`           | `application/service/`  |
| 유스케이스 트랜잭션               | `application/service/`의 `@Transactional` 등         | 공개 유스케이스 메서드  |
| 비즈니스 규칙·불변식              | `domain/`의 일반 Java 코드                           | 도메인 모델·서비스      |

`config/`에 Aspect 클래스를 둔다고 자동으로 모든 계층에 적용되는 것은 아닙니다. 실제 적용 범위는
Pointcut이 결정합니다. 반대로 웹 전용 Aspect를 `config/`에 두어도 동작할 수 있지만, 웹
기술과 수명 주기에 묶인 처리라면 웹 어댑터 안에 두는 편이 책임을 드러냅니다.

제공된 참고 프로젝트 소스에서는 웹 세션·관리자 권한 Aspect가 인바운드 웹 어댑터에, 요청 범위 캐시
Aspect가 아웃바운드 캐시 어댑터에 있습니다. 서비스 추적과 영속성 오류 로깅 Aspect는 공통 설정 아래에
있습니다. 이는 관찰된 배치 사례이지 모든 프로젝트에 적용해야 할 고정 규칙은 아닙니다.

트랜잭션은 유스케이스 경계를 표현할 때 주로 `application/service/`에 선언합니다. 직접
Aspect를 작성하기보다 Spring의 선언적 트랜잭션을 먼저 검토합니다. 웹 인증·인가에는 필터나 Spring
Security가 더 적절한 경우가 많습니다. Aspect를 택하면 프록시 적용 범위와 요청 처리 시점을 확인해야
합니다. 기존 인증 계약이나 권한 판단 위치를 바꾸는 경우에는 호출 경로 전체를 검증해야 합니다.

## 적용 전 점검

- Aspect와 대상이 모두 Spring 빈인지 확인합니다.
- 외부 호출이 프록시를 통과하는지 확인합니다.
- Pointcut이 대상 패키지·어노테이션을 정확히 선택하는지 확인합니다.
- `@Around`가 반환값과 예외를 보존하는지 확인합니다.
- 로그에 토큰, 개인정보, 요청 본문을 남기지 않습니다.
- 트랜잭션·캐시·보안과 함께 적용될 때 순서와 실패 시 동작을
  통합 테스트로 확인합니다.

Spring AOP는 경계를 가로지르는 기술적 관심사를 모으는 도구입니다.
적용 위치는 책임을 알려 주고, Pointcut은 실제 적용 범위를 결정합니다.
기대한 동작을 얻으려면 호출이 프록시를 거치는지 확인해야 합니다.

## 참고 자료

- [요청한 AOP 참고 페이지](https://techmentor-avo.pages.dev/theory/spring-06):
  2026-09-23 현재 내용을 열람하지 못해 본문의 사실 근거로 사용하지 않았습니다.
- [Spring Framework AOP 공식 문서](https://docs.spring.io/spring-framework/reference/core/aop.html)
- [Spring Boot AOP 공식 문서](https://docs.spring.io/spring-boot/reference/features/aop.html)
