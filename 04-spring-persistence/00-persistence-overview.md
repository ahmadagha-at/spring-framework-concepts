# Spring Persistence

Persistence is the part of an application responsible for storing and retrieving data beyond the lifetime of a single process. In a Spring application, several technologies often participate in this responsibility, but they operate at different abstraction levels.

## Scope of this series

This series explains:

- relational persistence and ORM
- Jakarta Persistence (JPA) and Hibernate
- entities and the persistence context
- entity state transitions and dirty checking
- relationships, fetching, and the N+1 problem
- Spring Data repositories
- derived queries, JPQL, and native SQL
- Spring Boot persistence auto-configuration

## The layers

```text
Application service
→ Spring Data JPA repository
→ Jakarta Persistence API
→ Hibernate
→ JDBC driver
→ relational database
```

Not every application needs every layer. JDBC can be used directly, and JPA can be used without Spring Data. Spring Boot does not replace any of these technologies; it configures common infrastructure when the required dependencies and properties are present.

## Recommended order

Read the articles in order. The distinction between specification, implementation, and Spring abstraction is essential before repository interfaces appear.

## References

- [Spring Framework: ORM Data Access](https://docs.spring.io/spring-framework/reference/data-access/orm.html)
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/)
- [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/)
- [Hibernate ORM User Guide](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)
