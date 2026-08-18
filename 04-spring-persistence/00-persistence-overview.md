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


## Why persistence abstractions exist

A relational database stores rows, columns, constraints, and relationships. Java code works with objects, references, methods, and inheritance. Persistence technology must translate between these two models without hiding the fact that they are different.

A useful way to reason about the stack is to ask which responsibility belongs to which layer:

| Layer | Responsibility |
|---|---|
| Database | Stores data and enforces relational constraints |
| JDBC driver | Translates Java database calls into the database protocol |
| JDBC | Provides the standard low-level Java API |
| Hibernate | Performs object-relational mapping and change tracking |
| Jakarta Persistence | Defines the standard persistence contracts |
| Spring ORM | Integrates persistence providers with Spring |
| Spring Data JPA | Generates repository implementations |
| Spring Boot | Configures common infrastructure conditionally |

An error in one layer cannot always be solved in another. A missing database index is not fixed by a repository abstraction. A wrong entity relationship is not fixed by adding a transaction. A transaction boundary cannot compensate for an invalid domain rule.

## Write flow

A common write request crosses several boundaries:

```text
controller receives request
→ application service validates the use case
→ transaction begins
→ repository loads managed entities
→ domain state changes
→ Hibernate detects changes
→ SQL is executed during flush
→ database constraints are checked
→ transaction commits
```

The repository call is only one step. A correct write also depends on transaction scope, entity state, mappings, generated SQL, and database guarantees.

## Read flow

Reads should be designed around the data needed by the use case:

```text
request
→ query requirement
→ repository or query service
→ JPQL, Criteria, or SQL
→ selected rows and columns
→ entity or projection
→ response DTO
```

Loading a complete entity graph for every read is usually unnecessary. Persistence models support writes and identity, while read models can be shaped specifically for API responses.

## Questions to ask while debugging

When persistence behavior is surprising, check:

1. Is the code running inside a transaction?
2. Is the entity managed, detached, or transient?
3. Which SQL statements were generated?
4. Which relationships were initialized?
5. Does the database schema match the mapping?
6. Are constraints enforced in Java, in the database, or both?
7. Is the observed behavior defined by Jakarta Persistence or specific to Hibernate?

These questions locate the responsible layer before configuration is changed.

## References

- [Spring Framework: ORM Data Access](https://docs.spring.io/spring-framework/reference/data-access/orm.html)
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/)
- [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/)
- [Hibernate ORM User Guide](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)
