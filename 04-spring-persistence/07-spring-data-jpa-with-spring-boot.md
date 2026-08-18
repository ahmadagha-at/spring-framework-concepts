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


## Auto-configuration conditions

Boot auto-configuration is conditional. Conceptually, it asks questions such as:

```text
Is JDBC available?
Is a DataSource available?
Is JPA available?
Is an EntityManagerFactory missing?
Is a transaction manager missing?
Where are the entities and repositories?
```

When the conditions match, configuration classes register the infrastructure beans. Defining an application-specific bean can make a default back off.

This is why debugging Boot configuration should include the condition evaluation report rather than assuming that a starter always creates the same objects.

## Repository and entity scanning

`@SpringBootApplication` establishes a default package boundary. Entities and repositories under that root are commonly discovered automatically.

A misplaced application class can cause repositories or entities to disappear from scanning. Explicit annotations such as `@EntityScan` and `@EnableJpaRepositories` are available, but reorganizing packages is often clearer for a single persistence unit.

## Configuration properties

Common settings include connection details, connection-pool behavior, schema validation, SQL logging, and provider properties.

Provider-specific settings belong under the appropriate Hibernate property namespace. Configuration should document why a non-default value exists; copying tuning properties without measuring the application can reduce reliability.

## Database migrations

A production schema evolves independently of one application startup. Migration tools provide ordered, reviewable changes:

```text
V1__create_users.sql
V2__add_user_status.sql
V3__create_order_indexes.sql
```

A safe startup can validate that entity mappings match the migrated schema without allowing Hibernate to rewrite production tables automatically.

## Multiple data sources

Applications with more than one database may need separate:

- data sources
- entity manager factories
- transaction managers
- entity packages
- repository packages

At that point, Boot's single-data-source defaults are insufficient and bean qualification becomes important.

## Startup failures to understand

Common causes include:

- missing JDBC driver
- invalid connection URL
- unreachable database
- wrong credentials
- entity mapping conflicts
- migration failure
- schema validation mismatch
- multiple transaction managers without qualification

The exception chain usually contains a lower-level database or mapping cause. Reading only the top-level bean-creation error hides the useful diagnosis.

## Key takeaway

Spring Boot assembles and configures JPA infrastructure. Jakarta Persistence defines the contract, Hibernate performs ORM work, Spring Data generates repository implementations, and the database remains responsible for relational integrity and query execution.

## References

- [Spring Boot: SQL Databases](https://docs.spring.io/spring-boot/reference/data/sql.html)
- [Spring Framework: JPA](https://docs.spring.io/spring-framework/reference/data-access/orm/jpa.html)
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/)
