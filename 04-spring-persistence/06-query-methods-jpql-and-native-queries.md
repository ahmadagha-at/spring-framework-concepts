# Part 6: Query Methods, JPQL, and Native Queries

Spring Data JPA supports several query styles. The correct choice depends on query complexity, portability, and performance requirements.

## Derived query methods

Spring Data can derive a query from a method name:

```java
List<User> findByActiveTrueAndEmailContainingIgnoreCase(
        String email
);
```

Derived queries are effective for short, readable conditions. Very long method names hide query intent and should be replaced with an explicit query or specification.

## JPQL

JPQL queries entities and their attributes rather than database tables and columns:

```java
@Query("""
       select u
       from User u
       where u.active = true
         and lower(u.email) like lower(concat('%', :term, '%'))
       """)
List<User> searchActiveUsers(@Param("term") String term);
```

JPQL is provider-translated and usually more portable than database-specific SQL.

## Native SQL

```java
@Query(
    value = "select * from users where email = :email",
    nativeQuery = true
)
Optional<User> findNative(@Param("email") String email);
```

Native SQL is useful for database-specific features or carefully optimized queries. It couples the repository more closely to the database schema and may require separate count queries for pagination.

## Projections

Queries should retrieve the shape required by the use case. DTO or interface projections can avoid loading complete entities when only selected fields are needed.

## Specifications

`JpaSpecificationExecutor` supports composable predicates for dynamic filters. Specifications are useful when optional search criteria would otherwise produce many repository methods.

## Query validation and performance

A query is not good merely because it returns correct data. Inspect generated SQL, query plans, indexes, number of database round trips, and selected columns.

## Key takeaway

Use derived queries for simple intent, JPQL for explicit entity-oriented queries, native SQL for justified database-specific needs, and projections or specifications for focused read models and dynamic filtering.

## References

- [Spring Data: Query Methods](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html)
- [Spring Data: Query Creation](https://docs.spring.io/spring-data/jpa/reference/repositories/query-methods-details.html)
- [Spring Data JPA: Specifications](https://docs.spring.io/spring-data/jpa/reference/jpa/specifications.html)
