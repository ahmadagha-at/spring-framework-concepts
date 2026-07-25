# Part 4: BeanFactory and ApplicationContext

`BeanFactory` and `ApplicationContext` are central interfaces of the Spring IoC container. They are related, but they represent different levels of container functionality.

## BeanFactory

`BeanFactory` is the basic interface for accessing Spring-managed beans.

Its main responsibilities include:

- storing bean definitions
- creating beans
- resolving dependencies
- returning bean instances
- managing basic lifecycle behavior

Conceptually:

```java
BeanFactory factory = ...;
PaymentService service =
        factory.getBean(PaymentService.class);
```

Application code should usually receive dependencies through injection rather than repeatedly calling `getBean()`. Direct lookups hide dependencies and resemble the Service Locator pattern.

## ApplicationContext

`ApplicationContext` extends `BeanFactory` and adds application-level capabilities.

These include:

- application event publication
- internationalization through message resources
- resource loading
- environment and profile access
- easier integration with Spring AOP
- automatic detection of many container extension components

Most modern Spring applications use an `ApplicationContext` rather than interacting with a plain `BeanFactory`.

## Common ApplicationContext implementations

Examples include:

- `AnnotationConfigApplicationContext`
- `ClassPathXmlApplicationContext`
- web-aware application context implementations

A small application can create an annotation-based context manually:

```java
try (AnnotationConfigApplicationContext context =
             new AnnotationConfigApplicationContext(
                     ApplicationConfiguration.class
             )) {

    UserService service =
            context.getBean(UserService.class);
}
```

Spring Boot creates and refreshes the appropriate application context automatically when `SpringApplication.run(...)` executes.

## Eager creation differences

An `ApplicationContext` commonly creates non-lazy singleton beans during context startup. This behavior detects missing dependencies and invalid configuration early.

Basic `BeanFactory` usage is traditionally more demand-driven. In normal application development, early validation from `ApplicationContext` is usually beneficial.

## Application events

Beans can publish events:

```java
public record UserRegisteredEvent(Long userId) {
}
```

```java
@Service
public class RegistrationService {

    private final ApplicationEventPublisher publisher;

    public RegistrationService(
            ApplicationEventPublisher publisher
    ) {
        this.publisher = publisher;
    }

    public void register(Long userId) {
        publisher.publishEvent(
                new UserRegisteredEvent(userId)
        );
    }
}
```

Listeners can react without creating a direct dependency from the publisher to every consumer:

```java
@Component
public class WelcomeEmailListener {

    @EventListener
    public void handle(UserRegisteredEvent event) {
        // send welcome email
    }
}
```

By default, event delivery is synchronous. Long-running work may require asynchronous processing or a durable messaging system.

## Resource abstraction

The `Resource` abstraction provides a consistent way to access files and classpath resources:

```java
@Value("classpath:templates/email.txt")
private Resource emailTemplate;
```

Possible resource locations include:

```text
classpath:...
file:...
https:...
```

This avoids tying application code directly to one resource-loading mechanism.

## Internationalization

`ApplicationContext` supports message lookup through `MessageSource`:

```java
String message = messageSource.getMessage(
        "user.created",
        null,
        locale
);
```

Message bundles can provide different text for different locales.

## Should application code inject ApplicationContext?

Usually not. Injecting `ApplicationContext` everywhere gives classes access to the complete container and hides their real dependencies.

Prefer:

```java
public BillingService(PaymentClient client) {
```

over:

```java
public BillingService(ApplicationContext context) {
```

Inject the narrower capability when container access is genuinely needed, such as `ApplicationEventPublisher`, `Environment`, or `ResourceLoader`.

## BeanFactory vs. ApplicationContext

| BeanFactory | ApplicationContext |
|---|---|
| Basic bean container contract | Extends `BeanFactory` |
| Creates and retrieves beans | Adds application-level services |
| Resolves dependencies | Supports events, resources, messages, and environment access |
| Useful for low-level infrastructure | Standard choice for most applications |

## Key takeaway

`BeanFactory` defines the fundamental bean-container behavior. `ApplicationContext` builds on it with capabilities needed by complete applications. Spring Boot normally creates an `ApplicationContext`, while developers work mainly through dependency injection.

## References

- [Spring Framework: Introduction to the IoC Container](https://docs.spring.io/spring-framework/reference/core/beans/introduction.html)
- [Spring Framework: Additional Capabilities of the ApplicationContext](https://docs.spring.io/spring-framework/reference/core/beans/context-introduction.html)
- [Spring Framework: The BeanFactory API](https://docs.spring.io/spring-framework/reference/core/beans/beanfactory.html)
- [GeeksforGeeks: Spring Tutorial](https://www.geeksforgeeks.org/advance-java/spring/)
