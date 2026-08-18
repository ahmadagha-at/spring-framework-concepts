# Spring AOP

Aspect-Oriented Programming separates cross-cutting behavior from the business methods to which it applies.

Examples include:

- transaction management
- method security
- logging
- metrics
- caching

## Core terminology

| Term | Meaning |
|---|---|
| Aspect | Module containing cross-cutting behavior |
| Advice | Code executed at a join point |
| Join point | A point during program execution |
| Pointcut | Rule selecting join points |
| Target | Object whose method is advised |
| Proxy | Wrapper that applies advice around the target |

Spring AOP supports method-execution join points on Spring-managed beans.

## Proxy flow

```text
caller
→ proxy
→ advice before method
→ target method
→ advice after method
→ caller
```

The object obtained from the application context may therefore be a proxy rather than the original class instance.

## JDK and class-based proxies

Spring can use JDK dynamic proxies for interfaces or class-based proxies. Class-based proxying has limitations, including methods that cannot be overridden.

Application design should not depend unnecessarily on one proxy mechanism.

## Aspect example

```java
@Aspect
@Component
public class TimingAspect {

    @Around("@annotation(Timed)")
    public Object measure(ProceedingJoinPoint point)
            throws Throwable {

        long start = System.nanoTime();
        try {
            return point.proceed();
        } finally {
            record(System.nanoTime() - start);
        }
    }
}
```

Aspects should remain focused. Hiding core business behavior inside advice makes control flow difficult to understand.


## Why cross-cutting concerns are difficult

Without AOP, the same infrastructure logic can appear in many services:

```java
beginTransaction();
checkPermission();
startTimer();
try {
    return executeBusinessOperation();
} finally {
    stopTimer();
    commitOrRollback();
}
```

This mixes business behavior with infrastructure behavior. AOP allows framework infrastructure to wrap eligible method calls consistently.

## Types of advice

Spring AOP supports advice at different points:

| Advice | Execution time |
|---|---|
| Before | Before the target method |
| After returning | After successful return |
| After throwing | After an exception |
| After | After completion in either case |
| Around | Controls invocation before and after `proceed()` |

Around advice is powerful because it can prevent invocation, change arguments, replace the result, or translate exceptions. It should therefore be used carefully.

## Pointcuts

A pointcut selects methods. Selection can be based on packages, types, methods, annotations, or combinations.

```java
@Around(
    "within(com.example.application..*) " +
    "&& @annotation(Timed)"
)
```

Broad pointcuts can advise unintended beans. Pointcuts should describe a stable architectural boundary rather than match accidental naming.

## Proxy creation

Bean post-processors inspect eligible beans during context creation. When advice applies, the object exposed by the context can be a proxy.

```text
bean definition
→ target instance created
→ bean post-processors inspect target
→ proxy created when eligible
→ proxy stored as exposed bean
```

Dependencies receive the exposed proxy. The proxy delegates to the target after applying interceptors.

## Interceptor chain

Several concerns can wrap the same method:

```text
method-security interceptor
→ transaction interceptor
→ metrics interceptor
→ target method
```

Ordering changes behavior. For example, whether timing includes authorization work depends on interceptor order.

## AOP and domain logic

AOP is suitable when behavior is:

- cross-cutting
- infrastructure-oriented
- consistently applicable
- understandable outside the target method

Do not hide essential domain state changes inside aspects. A reader should still understand the business use case from the service and domain code.

## Key takeaway

Spring AOP applies cross-cutting behavior through proxies around Spring-managed method calls. It is infrastructure, not a replacement for clear application design.

## References

- [Spring Framework: Aspect-Oriented Programming](https://docs.spring.io/spring-framework/reference/core/aop.html)
- [Spring Framework: AOP Proxies](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)
