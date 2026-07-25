# Part 5: Configuration, Annotations, and the Environment

Spring needs configuration metadata to know which beans exist, how they should be constructed, and under which conditions they should be available.

Modern applications commonly combine Java configuration, component scanning, annotations, and external properties.

## Java configuration

`@Configuration` marks a class that declares bean definitions:

```java
@Configuration
public class ClientConfiguration {

    @Bean
    public Clock clock() {
        return Clock.systemUTC();
    }

    @Bean
    public AuditService auditService(Clock clock) {
        return new AuditService(clock);
    }
}
```

Spring resolves the `Clock` dependency when creating `AuditService`.

Configuration classes are useful when object creation should be explicit or when registering classes that cannot be annotated.

## Component scanning

`@ComponentScan` tells Spring where to search for managed components:

```java
@Configuration
@ComponentScan("com.example.application")
public class ApplicationConfiguration {
}
```

In Spring Boot, `@SpringBootApplication` includes component scanning. By default, scanning begins in the package containing the application class and continues through its subpackages.

Placing the main class near the root package helps ensure that controllers, services, repositories, and configuration classes are discovered.

## Common Spring Core annotations

| Annotation | Purpose |
|---|---|
| `@Component` | Registers a general managed component |
| `@Service` | Identifies an application service |
| `@Repository` | Identifies a persistence component |
| `@Configuration` | Declares Java-based configuration |
| `@Bean` | Registers the returned object as a bean |
| `@Autowired` | Marks an injection point |
| `@Qualifier` | Selects a specific bean candidate |
| `@Primary` | Marks the preferred candidate |
| `@Scope` | Selects a bean scope |
| `@Lazy` | Delays bean creation |
| `@Value` | Injects a configuration value |
| `@Profile` | Activates configuration for selected environments |

Annotations such as `@RestController` and `@GetMapping` belong to Spring MVC rather than Spring Core.

## The Environment abstraction

Spring's `Environment` represents two important concepts:

- properties
- active profiles

It can be injected when programmatic access is needed:

```java
@Component
public class StorageSettings {

    private final Environment environment;

    public StorageSettings(Environment environment) {
        this.environment = environment;
    }

    public String bucketName() {
        return environment.getProperty("storage.bucket");
    }
}
```

For structured application configuration, Spring Boot's `@ConfigurationProperties` is usually clearer than repeated property lookups.

## Injecting values with `@Value`

```java
@Value("${token.expiration-ms}")
private long expirationMs;
```

Default values can be supplied:

```java
@Value("${token.expiration-ms:900000}")
private long expirationMs;
```

`@Value` is convenient for individual properties. Large related groups are better represented as validated configuration objects.

## Profiles

Profiles allow selected beans or configuration classes to exist only in particular environments:

```java
@Configuration
@Profile("development")
public class DevelopmentConfiguration {
}
```

```java
@Configuration
@Profile("production")
public class ProductionConfiguration {
}
```

Profiles should describe meaningful environment differences. They should not become a replacement for ordinary properties or create many incompatible versions of the application.

## Conditional configuration

Spring and Spring Boot support conditional bean registration. Spring Boot auto-configuration uses conditions extensively, for example:

- a class exists on the classpath
- a property has a particular value
- a bean is missing
- the application is a web application

This explains why defining your own bean can cause an auto-configured default to “back off.”

## Configuration should remain outside source code

Environment-specific secrets and infrastructure values should not be hard-coded:

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

The source code defines which configuration is required. The runtime environment supplies the actual values.

## Common mistakes

### Assuming every annotation belongs to Spring Boot

Many annotations used in Boot applications come from Spring Framework modules. Spring Boot configures their surrounding infrastructure but does not redefine their purpose.

### Component scanning too broadly

Scanning unrelated packages can register unintended components and slow startup.

### Hiding required configuration with defaults

Defaults are helpful for harmless development settings, but security credentials should normally be required explicitly.

### Overusing profiles

If only a URL changes, use a property. Use profiles when bean definitions or application behavior genuinely differ.

## Key takeaway

Configuration metadata tells the container what to manage. Annotations provide a concise way to declare that metadata, while the `Environment` separates application code from runtime-specific properties and profiles.

## References

- [Spring Framework: Annotation-based Container Configuration](https://docs.spring.io/spring-framework/reference/core/beans/annotation-config.html)
- [Spring Framework: Java-based Container Configuration](https://docs.spring.io/spring-framework/reference/core/beans/java.html)
- [Spring Framework: Environment Abstraction](https://docs.spring.io/spring-framework/reference/core/beans/environment.html)
- [Spring Boot: Externalized Configuration](https://docs.spring.io/spring-boot/reference/features/external-config.html)
- [GeeksforGeeks: Spring Tutorial](https://www.geeksforgeeks.org/advance-java/spring/)
