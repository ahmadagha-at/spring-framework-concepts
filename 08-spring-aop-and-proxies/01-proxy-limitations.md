# Part 1: Proxy Boundaries and Common Limitations

Proxy-based features only run when invocation passes through the proxy.

## Self-invocation

```java
@Service
public class ReportService {

    public void generate() {
        auditedOperation();
    }

    @Audited
    public void auditedOperation() {
    }
}
```

The internal call uses `this` and bypasses the proxy. The same issue can affect `@Transactional`, `@Cacheable`, `@Async`, and method security.

A common solution is to move the intercepted operation into another bean with a meaningful responsibility.

## Non-managed objects

Objects created with `new` are not automatically processed by Spring:

```java
ReportService service = new ReportService();
```

Spring cannot apply bean post-processors or create framework proxies around an object it does not manage.

## Method visibility and final methods

Proxy capabilities depend on the proxy strategy. Class-based proxies cannot override final methods. Avoid relying on interception of methods that the selected proxy cannot advise.

## Objects exposed outside the context

Keeping and using the original target reference can bypass the proxy. Application components should normally obtain collaborators through dependency injection.

## Debugging proxies

When an annotation appears to be ignored, ask:

1. Is the object a Spring bean?
2. Is the call entering through the proxy?
3. Does the pointcut match the method?
4. Can the selected proxy advise the method?
5. Is the required infrastructure enabled?
6. Is ordering between multiple aspects relevant?

## AOP is not every form of instrumentation

Spring AOP is proxy-based and focused on Spring beans. AspectJ can apply broader bytecode weaving, but it introduces a different model and additional complexity.


## Interface and class proxies

A JDK dynamic proxy implements one or more interfaces. Code interacting with it should use the proxied interface type.

A class-based proxy subclasses the target class. Methods that cannot be overridden cannot be intercepted through subclassing.

The exact defaults can depend on configuration and Spring Boot behavior. Application code should focus on valid bean boundaries rather than detecting proxy classes manually.

## Proxy identity

Because the exposed object can differ from the target instance, assumptions about runtime class equality can fail:

```java
bean.getClass() == ReportService.class
```

This may be false for class-based proxies. Use behavior and declared types rather than exact runtime-class checks.

## Final, private, and static methods

Private methods cannot be overridden and are not external bean entry points. Static methods belong to the class rather than a bean instance. Final methods cannot be overridden by a class proxy.

Placing an infrastructure annotation on such a method does not guarantee interception.

## Constructor execution

A proxy cannot advise work that happens before the proxied instance exists. Heavy logic in constructors is therefore outside normal method interception and also makes object creation fragile.

Constructors should establish valid state and receive dependencies. Use explicit lifecycle mechanisms for initialization requiring the container.

## Self-injection is not the default solution

Injecting a bean into itself or retrieving itself from `ApplicationContext` can route a call through the proxy, but it hides architecture and creates container coupling.

Prefer extracting the intercepted operation into a collaborator:

```text
OrderService
→ PaymentTransactionService
```

The new bean boundary should represent a real responsibility, not exist only to satisfy the framework.

## Programmatic alternatives

When dynamic transaction boundaries are genuinely required, `TransactionTemplate` can express them directly. When asynchronous execution is required, an explicit task executor can be used.

Programmatic APIs can be clearer than forcing proxy annotations onto an unsuitable call structure.

## AspectJ weaving

AspectJ can advise calls beyond Spring-managed proxy boundaries through compile-time or load-time weaving. It can address self-invocation, but it changes build, runtime, and debugging complexity.

Use it only when the broader join-point model is a real requirement, not as a shortcut around poor bean boundaries.

## Diagnostic example

If `@Transactional` appears ineffective:

1. confirm the class is a Spring bean
2. confirm the method is invoked from another bean
3. inspect whether the bean is proxied
4. check method visibility and proxy type
5. confirm a transaction manager exists
6. enable transaction logging
7. verify the exception is not swallowed
8. confirm rollback rules

This systematic path is more reliable than moving the annotation between methods randomly.

## Key takeaway

Annotations such as `@Transactional` describe behavior; a proxy must intercept a matching call to apply it. Understanding the invocation path is essential when debugging Spring infrastructure.

## References

- [Spring Framework: Understanding AOP Proxies](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)
- [Spring Framework: Declaring Advice](https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/advice.html)
