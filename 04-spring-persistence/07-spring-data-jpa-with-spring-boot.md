# Part 7: Using Spring Data JPA with Spring Boot

Spring Boot configures common JPA infrastructure when suitable dependencies, a data source, and configuration properties are present.

## Starter dependency

```gradle
implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
runtimeOnly 'org.postgresql:postgresql'
```

The starter makes Spring Data JPA, Spring ORM, Hibernate, and supporting dependencies available. It is a dependency descriptor, not the persistence mechanism itself.

## Data source configuration

```yaml
spring:
  datasource:
    url: ${DB_URL}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}
```

Credentials should be supplied by the runtime environment rather than committed to source control.

## Auto-configured infrastructure

Depending on application conditions, Boot can configure:

- a `DataSource`
- an `EntityManagerFactory`
- a transaction manager
- entity scanning
- Spring Data repository scanning
- Hibernate properties

Defining custom infrastructure can cause the corresponding default configuration to back off.

## Schema management

Hibernate schema generation can be useful in disposable development environments, but production schemas should normally be managed through explicit migrations such as Flyway or Liquibase.

```text
Entity mappings describe runtime persistence
Database migrations describe schema history
```

These responsibilities should not be confused.

## Open EntityManager in View

Web applications may keep the persistence context open through request processing. This can make lazy loading appear convenient, but it allows database access from controllers or serializers and can hide N+1 queries. A clear service-layer transaction boundary is usually easier to reason about.

## Observability

SQL logging is useful during development but may expose parameters or sensitive data. Production diagnostics should use controlled logging, metrics, and query analysis.

## Key takeaway

Spring Boot assembles and configures JPA infrastructure. Jakarta Persistence defines the contract, Hibernate performs ORM work, Spring Data generates repository implementations, and the database remains responsible for relational integrity and query execution.

## References

- [Spring Boot: SQL Databases](https://docs.spring.io/spring-boot/reference/data/sql.html)
- [Spring Framework: JPA](https://docs.spring.io/spring-framework/reference/data-access/orm/jpa.html)
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/)
