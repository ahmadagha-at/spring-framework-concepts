# Spring Framework, Spring MVC, and Spring Boot: What Is the Difference?

Spring Framework, Spring MVC, and Spring Boot are closely related technologies, but they do not serve the same purpose.

The Spring Framework provides the foundation for building Java applications. Spring MVC is a module of the Spring Framework for developing web applications and REST APIs. Spring Boot builds on Spring and simplifies the configuration and startup of Spring-based applications.

Understanding this relationship helps remove much of the confusion around the Spring ecosystem.

## What Is the Spring Framework?

The Spring Framework is the foundation of the Spring ecosystem. It provides the infrastructure and programming model used to build modular Java applications.

One of its most important concepts is **Inversion of Control (IoC)**. Instead of application classes creating and managing their dependencies themselves, Spring creates and manages these objects.

Objects managed by Spring are called **beans**.

Dependencies between beans are normally supplied through **Dependency Injection (DI)**. This reduces coupling between components and makes applications easier to test, replace, and maintain.

The Spring Framework contains modules for different areas of application development, including:

- core container functionality
- web development
- data access
- transaction management
- application security
- testing
- integration with other technologies

Spring therefore provides the foundation and the tools required to structure a Java application.

## What Is Spring MVC?

Spring MVC is a web framework and module within the Spring Framework.

MVC stands for:

```text
Model
View
Controller
```

The MVC pattern separates an application into different responsibilities:

- **Model** represents application data and business state.
- **View** represents the user interface.
- **Controller** receives requests and coordinates the response.

Spring MVC uses a central component called the `DispatcherServlet`. It receives incoming HTTP requests and forwards them to the correct controller method.

A typical controller looks like this:

```java
@RestController
@RequestMapping("/users")
public class UserController {

    @GetMapping("/{id}")
    public UserResponse getUser(@PathVariable Long id) {
        return userService.getUser(id);
    }
}
```

Annotations such as the following belong to Spring MVC:

```java
@Controller
@RestController
@RequestMapping
@GetMapping
@PostMapping
@RequestBody
@PathVariable
@RequestParam
```

Spring MVC can be used to build:

- server-rendered web applications
- REST APIs
- request and response handling
- validation and exception handling
- file upload endpoints

Spring MVC provides the web layer, but it does not automatically configure the entire application.

## What Is Spring Boot?

Spring Boot is built on top of the Spring Framework. It does not replace Spring Framework or Spring MVC.

Its purpose is to make Spring applications easier to configure, start, and run.

Without Spring Boot, developers may need to configure dependencies, web infrastructure, component scanning, server deployment, and other application details manually.

Spring Boot reduces this setup by providing:

- auto-configuration
- starter dependencies
- embedded web servers
- externalized configuration
- sensible defaults
- production-oriented features
  

In simple terms:

> Spring Framework provides the foundation, Spring MVC provides the web layer, and Spring Boot simplifies the setup and execution of the application.

## What Does Spring Boot Take Care Of?

### 1. Auto-Configuration

Spring Boot examines the dependencies on the classpath, existing beans, and application properties.

Based on this information, it automatically configures suitable parts of the application.

For example, when the required web dependencies are present, Spring Boot can configure the infrastructure needed for a Spring MVC application.

Auto-configuration is conditional. It is applied only when the relevant conditions are satisfied and can be customized or replaced.

### 2. Starter Dependencies

Spring Boot provides starter dependencies for common application types.

For example:

```gradle
implementation 'org.springframework.boot:spring-boot-starter-web'
```

This starter provides the dependencies commonly required for a web application, including Spring MVC and JSON processing.

Other starters include:

```gradle
spring-boot-starter-security
spring-boot-starter-data-jpa
spring-boot-starter-validation
spring-boot-starter-test
```

Starters reduce the need to select and manage every related dependency individually.

### 3. Embedded Web Server

A Spring Boot web application can run with an embedded server such as Tomcat.

The application can be started directly:

```bash
./gradlew bootRun
```

or packaged and executed:

```bash
java -jar application.jar
```

A separate external application server is normally not required.

### 4. Externalized Configuration

Spring Boot allows configuration to be supplied through:

- `application.properties`
- `application.yaml`
- environment variables
- command-line arguments
- profile-specific configuration files

For example:

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

This makes it possible to use different configurations for development, testing, and production without changing the source code.

### 5. Production-Oriented Features

Spring Boot Actuator can expose information about an application's:

- health
- metrics
- configuration
- environment
- runtime behavior

These features help monitor and manage an application after deployment.

### 6. Sensible Defaults

Spring Boot follows a convention-over-configuration approach.

It provides reasonable default behavior for common situations, while still allowing developers to override the configuration when necessary.

## How Do They Work Together?

A typical Spring Boot REST API uses all three technologies:

```text
Spring Framework
→ creates and manages beans
→ provides Dependency Injection

Spring MVC
→ receives HTTP requests
→ maps requests to controller methods
→ creates HTTP responses

Spring Boot
→ configures the Spring application
→ provides starter dependencies
→ starts the embedded server
```

For example:

```java
@SpringBootApplication
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

`@SpringBootApplication` starts the Spring Boot application and enables important functionality such as component scanning and auto-configuration.

A REST controller inside the same application uses Spring MVC:

```java
@RestController
@RequestMapping("/api")
public class ExampleController {

    @GetMapping("/hello")
    public String hello() {
        return "Hello";
    }
}
```

The controller itself is also a Spring-managed bean.

The application therefore does not choose between Spring Framework, Spring MVC, and Spring Boot. They usually work together.

## Comparison

| Technology | Main responsibility |
|---|---|
| Spring Framework | Provides the core programming model, IoC container, beans, and Dependency Injection |
| Spring MVC | Provides the web layer for handling HTTP requests and responses |
| Spring Boot | Simplifies configuration, dependency management, startup, and deployment |
| `DispatcherServlet` | Routes incoming web requests to the correct Spring MVC controller |
| Embedded server | Runs the Spring Boot web application without a separate server installation |

## Spring MVC vs. Spring Boot

Spring MVC and Spring Boot are not direct alternatives.

Spring MVC answers:

> How should HTTP requests be handled inside the application?

Spring Boot answers:

> How can the Spring application be configured, started, and packaged with less manual setup?

A Spring Boot application can use Spring MVC, but Spring Boot can also be used for applications that do not provide a web interface, such as:

- scheduled applications
- command-line applications
- message consumers
- batch-processing applications

Spring MVC is specifically concerned with web development.

## Does Spring Boot Replace Spring MVC?

No.

When a Spring Boot project includes a web starter, Spring Boot commonly configures Spring MVC automatically.

The controller annotations, request mappings, validation, `DispatcherServlet`, and response handling still come from Spring MVC.

Spring Boot makes the setup easier, but Spring MVC continues to provide the web functionality.

## Does Spring Boot Remove the Need to Understand Spring?

No.

Spring Boot reduces configuration, but the application still depends on Spring concepts such as:

- IoC container
- beans
- Dependency Injection
- component scanning
- bean lifecycle
- application configuration
- Spring MVC request handling

Without understanding these concepts, Spring Boot can appear to work like magic.

Knowing the underlying Spring Framework makes it easier to:

- debug configuration problems
- understand startup errors
- replace auto-configuration
- write testable components
- make better architectural decisions

## Conclusion

The three technologies represent different layers of the Spring ecosystem:

- **Spring Framework** provides the foundation and programming model.
- **Spring MVC** provides the web framework for controllers, requests, and responses.
- **Spring Boot** simplifies the configuration, startup, and deployment of Spring-based applications.

The most important distinction is:

> Spring Framework manages the application components, Spring MVC handles the web layer, and Spring Boot makes the complete application easier to configure and run.

Spring Boot is therefore not a replacement for Spring Framework or Spring MVC. It is a convenient and opinionated way to build applications that use them.

## Reference

- [GeeksforGeeks: Spring Boot Tutorial](https://www.geeksforgeeks.org/advance-java/spring-boot/)
