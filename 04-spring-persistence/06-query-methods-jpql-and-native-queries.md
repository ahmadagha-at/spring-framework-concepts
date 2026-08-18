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


## Query derivation rules

Spring Data parses a method name into a subject and predicate:

```text
findTop10ByStatusAndCreatedAtBeforeOrderByCreatedAtDesc
│         │                              │
subject   predicate                      ordering
```

Property expressions are checked against the domain model. This provides useful startup validation, but names become difficult to read when many conditions are combined.

Reserved repository methods such as `findById` have predefined semantics. Similar-looking domain properties can therefore create confusing names, so method intent should remain explicit.

## Parameter binding

Named parameters make explicit queries easier to maintain:

```java
@Query("""
       select o
       from Order o
       where o.customer.id = :customerId
         and o.status = :status
       """)
List<Order> findCustomerOrders(
        @Param("customerId") Long customerId,
        @Param("status") OrderStatus status
);
```

String concatenation must never be used to insert untrusted values into native SQL. Parameter binding protects query structure and allows the driver to handle values correctly.

## Fetch queries and pagination

A collection fetch join can duplicate root rows because one parent appears once for every joined child. Combining that query with pagination may produce inefficient in-memory behavior or incorrect expectations.

A common approach is:

1. page only the parent identifiers
2. fetch the required graph for those identifiers
3. preserve the requested ordering

There is no universal query annotation that solves every pagination and association combination.

## Bulk updates

JPQL update and delete statements operate directly on database rows:

```java
@Modifying(clearAutomatically = true)
@Query("""
       update User u
       set u.active = false
       where u.lastLoginAt < :cutoff
       """)
int deactivateInactiveUsers(Instant cutoff);
```

Bulk operations bypass normal entity dirty checking and can leave managed entities stale. Clearing or synchronizing the persistence context must be considered explicitly.

## Specifications and dynamic queries

Specifications are useful when filters are optional:

```java
Specification<Order> specification =
        hasStatus(filter.status())
        .and(createdAfter(filter.createdAfter()));
```

They improve composition, but deeply nested criteria code can still become difficult to understand. For complex reporting, explicit query objects or SQL may be clearer.

## Validate with SQL

For important queries, verify:

- generated SQL
- selected columns
- joins
- number of statements
- query plan
- index use
- row counts
- pagination behavior

The repository method name describes intent; the database execution plan determines performance.

## Key takeaway

Use derived queries for simple intent, JPQL for explicit entity-oriented queries, native SQL for justified database-specific needs, and projections or specifications for focused read models and dynamic filtering.

## References

- [Spring Data: Query Methods](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html)
- [Spring Data: Query Creation](https://docs.spring.io/spring-data/jpa/reference/repositories/query-methods-details.html)
- [Spring Data JPA: Specifications](https://docs.spring.io/spring-data/jpa/reference/jpa/specifications.html)
