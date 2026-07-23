# Spring Core

Spring Core is the foundation of the Spring ecosystem. It provides the container that creates application objects, connects their dependencies, manages their lifecycle, and makes application configuration available to them.

This series explains these responsibilities step by step. The goal is not only to show the annotations commonly used in Spring Boot applications, but to explain what the Spring container does behind those annotations.

## Scope of this series

The articles focus on the Spring IoC container and its main responsibilities:

- creating and managing beans
- resolving dependencies
- registering components
- selecting between multiple bean candidates
- managing lifecycle callbacks and scopes
- reading configuration from the environment
- understanding how Spring Boot starts and configures the container

`DispatcherServlet` is not a Spring Core component. It belongs to Spring MVC and is covered in the separate Spring MVC article. Similarly, Actuator and DevTools are Spring Boot features; they appear only in the final integration article so that the boundary between Spring Framework and Spring Boot remains clear.

## Recommended order

Read the articles in order. Later parts assume an understanding of IoC, Dependency Injection, bean definitions, and the container.

## References

- [Spring Framework Reference: The IoC Container](https://docs.spring.io/spring-framework/reference/core/beans.html)
- [Spring Framework Reference: Core Technologies](https://docs.spring.io/spring-framework/reference/core.html)
- [Spring Boot Reference](https://docs.spring.io/spring-boot/reference/)
- [GeeksforGeeks: Spring Tutorial](https://www.geeksforgeeks.org/advance-java/spring/)
- [GeeksforGeeks: Spring Boot Tutorial](https://www.geeksforgeeks.org/advance-java/spring-boot/)
