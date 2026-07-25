# Part 6: Using Spring Core with Spring Boot

Spring Boot does not replace Spring Core. It starts, configures, and customizes the Spring container using conventions and conditional auto-configuration.

This final part connects the Core concepts from the previous articles to the tools commonly used in a Spring Boot project.

## `@SpringBootApplication`

A typical Boot application starts with:

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

`@SpringBootApplication` combines three important ideas:

- configuration class registration
- component scanning
- Spring Boot auto-configuration

`SpringApplication.run(...)` prepares and refreshes an `ApplicationContext`. The context then discovers bean definitions, creates singleton beans, injects dependencies, and runs lifecycle callbacks.

## Auto-configuration

Spring Boot examines the classpath, properties, application type, and existing beans. It then applies matching configuration.

For example, when web dependencies are present, Boot can configure the infrastructure for a web application. When a developer supplies a custom bean, the corresponding default may back off.

```text
Dependencies + properties + existing beans
→ conditional checks
→ matching auto-configuration
→ bean definitions
→ ApplicationContext
```

Auto-configuration does not remove control. It provides defaults that can be customized, excluded, or replaced.

## Starter dependencies

Starters group dependencies for a common use case:

```gradle
implementation 'org.springframework.boot:spring-boot-starter-web'
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
implementation 'org.springframework.boot:spring-boot-starter-security'
```

A starter is not a code generator. It is a dependency descriptor that brings together a compatible set of libraries and enables related auto-configuration conditions.

## Maven and Gradle

Maven and Gradle are build tools, not parts of Spring Core.

They are responsible for tasks such as:

- resolving dependencies
- compiling source code
- running tests
- packaging the application
- executing build plugins

Gradle example:

```gradle
plugins {
    id 'java'
    id 'org.springframework.boot' version 'VERSION'
    id 'io.spring.dependency-management' version 'VERSION'
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}
```

Maven and Gradle make the required libraries available on the classpath. Spring Boot then uses classpath information when evaluating auto-configuration.

## Dependency management

Spring Boot manages compatible dependency versions for supported libraries. Developers can usually omit versions for dependencies covered by Boot's dependency management.

This reduces version conflicts, but it does not eliminate the need to understand the dependency graph or keep the Boot version updated.

## Properties and YAML

Spring Boot supports both `.properties` and YAML configuration.

Properties example:

```properties
server.port=8081
spring.datasource.url=${DB_URL}
```

YAML example:

```yaml
server:
  port: 8081

spring:
  datasource:
    url: ${DB_URL}
```

The two formats express the same configuration model. YAML is convenient for hierarchical settings, while properties files are explicit and simple.

Configuration can also come from environment variables and command-line arguments. Spring Boot defines a property-source precedence so higher-priority sources can override lower-priority values.

## DevTools

Spring Boot DevTools improves the local development experience. Depending on the environment, it can provide automatic restarts and development-friendly defaults.

```gradle
developmentOnly 'org.springframework.boot:spring-boot-devtools'
```

DevTools should be treated as a development dependency and not as application functionality.

## Actuator

Actuator adds production-oriented management capabilities:

```gradle
implementation 'org.springframework.boot:spring-boot-starter-actuator'
```

It can expose information such as:

- application health
- metrics
- environment and configuration details
- registered beans and mappings
- runtime behavior

Actuator endpoints can reveal sensitive operational data. Exposure and authorization must be configured carefully, especially when Spring Security is present.

## Where DispatcherServlet belongs

`DispatcherServlet` belongs to Spring MVC, not Spring Core or Spring Boot.

Spring Boot can configure it automatically when the MVC web dependencies are available. Spring MVC still owns request mapping and controller dispatching.

```text
Spring Core → manages controller and service beans
Spring MVC  → routes HTTP requests through DispatcherServlet
Spring Boot → configures and starts the application
```

## Recommended practices

- place the application class in a root package
- prefer constructor injection
- keep configuration outside source code
- use `@ConfigurationProperties` for related settings
- understand which auto-configuration creates important beans
- replace defaults intentionally rather than adding duplicate beans
- expose only required Actuator endpoints
- keep DevTools out of production runtime
- use focused tests instead of always starting the complete context

## Key takeaway

Spring Core provides the IoC container and bean model. Build tools assemble the classpath. Spring Boot interprets that classpath and the application configuration to prepare an `ApplicationContext` with useful defaults.

Understanding this relationship turns Boot's apparent “magic” into predictable container behavior.

## References

- [Spring Boot: Developing with Spring Boot](https://docs.spring.io/spring-boot/reference/using/index.html)
- [Spring Boot: Auto-configuration](https://docs.spring.io/spring-boot/reference/using/auto-configuration.html)
- [Spring Boot: Externalized Configuration](https://docs.spring.io/spring-boot/reference/features/external-config.html)
- [Spring Boot: Developer Tools](https://docs.spring.io/spring-boot/reference/using/devtools.html)
- [Spring Boot: Production-ready Features](https://docs.spring.io/spring-boot/reference/actuator/)
- [GeeksforGeeks: Spring Boot Tutorial](https://www.geeksforgeeks.org/advance-java/spring-boot/)
- [GeeksforGeeks: Spring Tutorial](https://www.geeksforgeeks.org/advance-java/spring/)
