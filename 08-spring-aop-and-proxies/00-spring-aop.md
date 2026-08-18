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

## Key takeaway

Spring AOP applies cross-cutting behavior through proxies around Spring-managed method calls. It is infrastructure, not a replacement for clear application design.

## References

- [Spring Framework: Aspect-Oriented Programming](https://docs.spring.io/spring-framework/reference/core/aop.html)
- [Spring Framework: AOP Proxies](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)
