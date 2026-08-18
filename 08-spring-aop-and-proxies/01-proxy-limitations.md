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

## Key takeaway

Annotations such as `@Transactional` describe behavior; a proxy must intercept a matching call to apply it. Understanding the invocation path is essential when debugging Spring infrastructure.

## References

- [Spring Framework: Understanding AOP Proxies](https://docs.spring.io/spring-framework/reference/core/aop/proxying.html)
- [Spring Framework: Declaring Advice](https://docs.spring.io/spring-framework/reference/core/aop/ataspectj/advice.html)
