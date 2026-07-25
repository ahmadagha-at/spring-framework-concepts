# Part 3: Bean Lifecycle and Scopes

The Spring container does more than construct objects. It also controls when beans are created, how they are initialized, how long they live, and when destruction callbacks are invoked.

## Simplified bean lifecycle

A typical singleton bean moves through these stages:

```text
Bean definition is registered
→ Bean is instantiated
→ Dependencies are injected
→ Aware callbacks run when applicable
→ BeanPostProcessor before-initialization hooks run
→ Initialization callbacks run
→ BeanPostProcessor after-initialization hooks run
→ Bean is ready for use
→ Destruction callbacks run when the context closes
```

The exact lifecycle can include additional framework extension points and proxy creation.

## Initialization and destruction callbacks

Modern Spring applications commonly use `@PostConstruct` and `@PreDestroy`:

```java
@Component
public class CacheManager {

    @PostConstruct
    void initialize() {
        // load initial cache data
    }

    @PreDestroy
    void shutdown() {
        // release resources
    }
}
```

Use lifecycle callbacks for work that depends on injected dependencies or for releasing resources owned by the bean.

Constructors should establish basic object validity. Expensive startup work should be used carefully because it can slow or fail application startup.

## Lifecycle callbacks with `@Bean`

Initialization and destruction methods can be declared explicitly:

```java
@Bean(
        initMethod = "connect",
        destroyMethod = "disconnect"
)
public ExternalClient externalClient() {
    return new ExternalClient();
}
```

This is useful for third-party classes that cannot be modified with annotations.

## Spring-specific lifecycle interfaces

Spring also provides interfaces such as:

```java
InitializingBean
DisposableBean
```

They work, but they couple application classes directly to Spring APIs. Annotation-based callbacks or `@Bean` method metadata are usually preferred for ordinary application code.

## BeanPostProcessor

`BeanPostProcessor` allows infrastructure code to inspect or modify bean instances before and after initialization:

```java
public interface BeanPostProcessor {

    Object postProcessBeforeInitialization(
            Object bean,
            String beanName
    );

    Object postProcessAfterInitialization(
            Object bean,
            String beanName
    );
}
```

Spring uses bean post-processors for important functionality, including annotation processing and proxy creation. Application developers rarely need to create one, but understanding it explains why the object returned by the container may be a proxy rather than the original instance.

## What is a bean scope?

A scope defines how many instances Spring creates and how long each instance lives.

## Singleton scope

Singleton is the default scope:

```java
@Service
public class PricingService {
}
```

Spring creates one instance per bean definition per IoC container.

This is not the same as the classic JVM-wide Singleton pattern. Two separate application contexts can each contain their own instance.

Singleton beans should normally be stateless or thread-safe because many requests and threads can use the same instance.

Avoid request-specific mutable fields such as:

```java
private String currentUsername;
```

inside singleton services.

## Prototype scope

Prototype scope creates a new instance each time the bean is requested from the container:

```java
@Component
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
public class ReportBuilder {
}
```

Spring creates and configures prototype instances, but it does not manage their complete destruction lifecycle. Code using a prototype bean is responsible for cleaning up resources when necessary.

## Prototype inside a singleton

Injecting a prototype bean directly into a singleton does not automatically create a new prototype on every method call. The prototype is resolved when the singleton is constructed.

Use `ObjectProvider` when a fresh instance is needed:

```java
@Service
public class ReportService {

    private final ObjectProvider<ReportBuilder> builders;

    public ReportService(
            ObjectProvider<ReportBuilder> builders
    ) {
        this.builders = builders;
    }

    public void createReport() {
        ReportBuilder builder = builders.getObject();
    }
}
```

## Web-aware scopes

Spring provides additional scopes in a web-aware application context:

| Scope | Lifetime |
|---|---|
| `request` | One HTTP request |
| `session` | One HTTP session |
| `application` | One `ServletContext` |
| `websocket` | One WebSocket lifecycle |

Example:

```java
@Component
@RequestScope
public class RequestMetadata {
}
```

When a shorter-lived bean is injected into a singleton, Spring commonly uses a scoped proxy so the singleton can resolve the correct current instance.

## Custom scopes

Spring allows custom `Scope` implementations to be registered with the container. A custom scope defines how instances are stored, retrieved, and destroyed.

Custom scopes are useful for specialized lifecycles, but they introduce framework-level complexity. Existing scopes should be preferred unless the domain has a clear lifecycle that cannot be represented otherwise.

## Lazy initialization

By default, singleton beans are commonly created when the application context starts. `@Lazy` delays creation until the bean is first requested:

```java
@Lazy
@Component
public class ExpensiveClient {
}
```

Lazy initialization can reduce startup work but can also move configuration errors from startup to runtime.

## Key takeaway

Lifecycle describes what happens to a managed object from creation to destruction. Scope describes how many instances exist and how long they live. Choosing the correct scope is important for thread safety, resource management, and predictable application behavior.

## References

- [Spring Framework: Bean Scopes](https://docs.spring.io/spring-framework/reference/core/beans/factory-scopes.html)
- [Spring Framework: Customizing the Nature of a Bean](https://docs.spring.io/spring-framework/reference/core/beans/factory-nature.html)
- [GeeksforGeeks: Spring Tutorial](https://www.geeksforgeeks.org/advance-java/spring/)
