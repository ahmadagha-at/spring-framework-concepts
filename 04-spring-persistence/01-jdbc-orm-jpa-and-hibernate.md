# Part 1: JDBC, ORM, JPA, and Hibernate

JDBC, ORM, JPA, and Hibernate are related, but they are not interchangeable.

## JDBC

JDBC is the standard Java API for communicating with relational databases. Application code can open connections, prepare SQL statements, bind parameters, execute queries, and read result sets.

JDBC gives direct control over SQL, but developers must manually map rows to objects and manage repetitive resource-handling code.

## Object-Relational Mapping

Object-Relational Mapping, or ORM, maps object-oriented domain models to relational tables.

```text
Java class   ↔ table
object       ↔ row
field        ↔ column
reference    ↔ foreign key
```

ORM does not remove the relational model. Developers still need to understand SQL, indexes, constraints, joins, and transactions.

## Jakarta Persistence

Jakarta Persistence, commonly called JPA from its former name, is a specification. It defines APIs and rules for concepts such as:

- entities
- persistence contexts
- entity managers
- relationships
- lifecycle operations
- JPQL
- transactions

A specification defines contracts; it does not provide the complete runtime implementation.

## Hibernate

Hibernate ORM is a persistence provider that implements Jakarta Persistence. It also provides capabilities beyond the specification.

```text
Jakarta Persistence → contract
Hibernate           → implementation
```

Applications should distinguish portable Jakarta Persistence behavior from provider-specific Hibernate features.

## Spring ORM and Spring Data JPA

Spring ORM integrates JPA providers with Spring transaction management, exception translation, and dependency injection.

Spring Data JPA adds the repository abstraction. It can create repository implementations from interfaces and derive queries from method names.


## Why the distinction matters in practice

Consider a repository method that appears to “save an object.” Several technologies participate:

```java
userRepository.save(user);
```

Spring Data JPA supplies the repository proxy. The proxy delegates to an `EntityManager`, whose API is defined by Jakarta Persistence. Hibernate commonly implements that API. Hibernate produces SQL and executes it through JDBC. The database finally validates and stores the data.

If the generated SQL is inefficient, the problem may be the mapping or query. If the application cannot start because no `EntityManagerFactory` exists, the problem is infrastructure configuration. If a unique constraint fails, the database is enforcing a relational rule.

## JDBC without ORM

Direct JDBC can be appropriate when:

- SQL control is the primary requirement
- the domain model is simple
- bulk operations are common
- database-specific features are important
- object graphs would add unnecessary complexity

Spring's `JdbcTemplate` reduces connection and exception-handling boilerplate while keeping SQL explicit:

```java
public Optional<UserRow> findById(long id) {
    return jdbcClient.sql("""
            select id, email
            from users
            where id = :id
            """)
        .param("id", id)
        .query(UserRow.class)
        .optional();
}
```

Using JDBC directly is not a lower-quality architecture. It is a different tradeoff.

## What ORM provides

ORM adds more than column mapping. A persistence provider can provide:

- identity management inside a persistence context
- association mapping
- dirty checking
- cascading lifecycle operations
- optimistic locking
- a query language over entities
- first-level and optional second-level caching

These capabilities reduce repetitive mapping code but introduce rules about entity state, fetching, and transaction boundaries.

## Portable and provider-specific behavior

Code using only Jakarta Persistence APIs is more portable between providers. Hibernate-specific annotations and query features can still be valuable, but the dependency should be intentional.

A useful documentation habit is to state whether a behavior comes from:

```text
Jakarta Persistence specification
or
Hibernate implementation
or
Spring integration
```

This avoids attributing every persistence feature to Spring Data JPA.

## Choosing the level

Use the highest abstraction that remains predictable for the use case. Standard CRUD around a domain model often fits JPA. Reporting, analytics, and bulk updates may benefit from explicit SQL. A single application can use both when transaction and ownership boundaries remain clear.

## Key takeaway

JDBC provides low-level database access. ORM maps objects to relational data. Jakarta Persistence defines a standard ORM contract. Hibernate implements that contract. Spring ORM integrates it with Spring, and Spring Data JPA reduces repository boilerplate.

## References

- [Spring Framework: ORM Data Access](https://docs.spring.io/spring-framework/reference/data-access/orm.html)
- [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/)
- [Hibernate ORM User Guide](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)
