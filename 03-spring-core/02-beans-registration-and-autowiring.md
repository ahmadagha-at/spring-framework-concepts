# Part 2: Beans, Registration, and Autowiring

A bean is an object whose creation and lifecycle are managed by the Spring IoC container. Not every Java object is automatically a bean: Spring must receive a bean definition describing how that object should be created.

## What is a bean definition?

Internally, Spring represents configuration metadata as `BeanDefinition` objects. A bean definition can contain information such as:

- the bean class
- constructor arguments and dependencies
- scope
- initialization and destruction callbacks
- whether the bean should be created lazily
- qualifiers used during dependency resolution

The application normally supplies this metadata through component scanning or Java configuration.

## Registration with component scanning

Spring can scan packages for stereotype annotations:

```java
@Component
public class PasswordGenerator {
}
```

Specialized stereotypes communicate the role of a component:

```java
@Service
public class UserService {
}
```

```java
@Repository
public class JdbcUserRepository {
}
```

```java
@Controller
public class PageController {
}
```

```java
@RestController
public class UserController {
}
```

All of these classes can become Spring beans when their packages are included in component scanning.

`@Repository`, `@Service`, and `@Controller` are specialized forms of `@Component`. Besides documentation value, some specialized annotations can participate in additional framework behavior.

## Registration with `@Bean`

Classes can also be registered explicitly:

```java
@Configuration
public class ApplicationConfiguration {

    @Bean
    public Clock applicationClock() {
        return Clock.systemUTC();
    }
}
```

The method name becomes the default bean name, and the returned object becomes the managed bean.

`@Bean` is particularly useful when:

- the class comes from an external library
- construction requires custom logic
- configuration values must be passed to the constructor
- the application should explicitly choose an implementation

## Component scanning vs. `@Bean`

```text
Component scanning
→ Spring discovers an annotated application class

@Bean
→ configuration code explicitly creates and registers an object
```

Neither approach is universally better. Application services often use component scanning, while infrastructure objects and third-party classes are commonly registered with `@Bean`.

## How autowiring works

When Spring creates a bean, it resolves the dependencies declared by its constructor or injection points.

For a constructor parameter such as:

```java
public UserService(UserRepository repository) {
```

Spring searches for a bean assignable to `UserRepository`.

### Exactly one candidate

If exactly one matching bean exists, Spring injects it.

### No candidate

If no matching bean exists, application startup normally fails with an unsatisfied dependency error.

### Multiple candidates

If multiple matching beans exist, Spring needs more information:

```java
@Repository
public class JpaUserRepository implements UserRepository {
}
```

```java
@Repository
public class InMemoryUserRepository implements UserRepository {
}
```

Injecting only `UserRepository` is now ambiguous.

## Selecting a candidate with `@Primary`

One implementation can be the default:

```java
@Primary
@Repository
public class JpaUserRepository implements UserRepository {
}
```

Spring chooses the primary candidate unless a more specific qualifier is used.

## Selecting a candidate with `@Qualifier`

Use a qualifier when the injection point requires a specific implementation:

```java
public UserService(
        @Qualifier("inMemoryUserRepository")
        UserRepository repository
) {
    this.repository = repository;
}
```

Custom qualifier annotations can make large applications safer than relying on string values.

## Injecting collections

Spring can inject every bean of a particular type:

```java
@Service
public class NotificationService {

    private final List<NotificationSender> senders;

    public NotificationService(
            List<NotificationSender> senders
    ) {
        this.senders = senders;
    }
}
```

This is useful for strategy pipelines, validators, handlers, and plugin-like designs.

## Optional and lazy dependencies

Optional dependencies can be expressed with `Optional<T>` or `ObjectProvider<T>`:

```java
public ReportService(
        ObjectProvider<ReportExporter> exporterProvider
) {
    this.exporterProvider = exporterProvider;
}
```

`ObjectProvider` allows lazy retrieval and can safely handle missing or multiple candidates. It should be used intentionally rather than hiding an unclear design.

## Common mistakes

### Creating a managed dependency manually

```java
UserRepository repository = new JpaUserRepository();
```

The new object is outside the container and does not receive Spring-managed dependencies or proxies.

### Scanning the wrong package

Spring Boot normally scans from the package containing the main application class downward. Components outside that hierarchy may not be discovered.

### Registering the same responsibility twice

Combining component scanning and an additional `@Bean` method for the same implementation can produce ambiguous candidates.

## Key takeaway

A Spring bean is not merely an object with an annotation. It is an object described by a bean definition and managed by the container. Registration tells Spring what it can create; autowiring tells Spring how those managed objects depend on one another.

## References

- [Spring Framework: Bean Overview](https://docs.spring.io/spring-framework/reference/core/beans/definition.html)
- [Spring Framework: Classpath Scanning and Managed Components](https://docs.spring.io/spring-framework/reference/core/beans/classpath-scanning.html)
- [Spring Framework: Using `@Autowired`](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config/autowired.html)
- [GeeksforGeeks: Spring Tutorial](https://www.geeksforgeeks.org/advance-java/spring/)
