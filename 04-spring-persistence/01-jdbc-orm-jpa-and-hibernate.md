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

## Key takeaway

JDBC provides low-level database access. ORM maps objects to relational data. Jakarta Persistence defines a standard ORM contract. Hibernate implements that contract. Spring ORM integrates it with Spring, and Spring Data JPA reduces repository boilerplate.

## References

- [Spring Framework: ORM Data Access](https://docs.spring.io/spring-framework/reference/data-access/orm.html)
- [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/)
- [Hibernate ORM User Guide](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)
