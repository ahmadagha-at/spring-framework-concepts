# Part 4: Relationships, Fetching, and the N+1 Problem

Entity relationships model associations between persistent objects. They also determine how and when related data is loaded.

## Relationship types

Jakarta Persistence defines:

- `@OneToOne`
- `@OneToMany`
- `@ManyToOne`
- `@ManyToMany`

A typical many-to-one association is:

```java
@ManyToOne(fetch = FetchType.LAZY, optional = false)
@JoinColumn(name = "customer_id")
private Customer customer;
```

The side containing the foreign key is normally the owning side. In bidirectional relationships, `mappedBy` identifies the inverse side.

## Cascades are not fetch strategies

Cascades propagate entity lifecycle operations such as persist or remove. Fetch strategies control when association data is loaded. They solve different problems.

Using `CascadeType.ALL` everywhere can cause unintended writes or deletions. Cascades should match aggregate ownership.

## Lazy and eager loading

Lazy loading delays association retrieval until it is accessed. Eager loading requests immediate availability, but it does not guarantee that one efficient SQL query will be used.

Entities should not be returned directly as API responses. Serialization can trigger unexpected lazy loading, expose internal fields, and create cycles. Map entities to explicit response models.

## The N+1 problem

The N+1 problem occurs when one query loads a collection of parent entities and additional queries load related data for every parent.

```text
1 query for orders
+ N queries for each order's customer
= N + 1 queries
```

Possible solutions include:

- fetch joins
- entity graphs
- DTO projections
- batch fetching
- use-case-specific queries

Making every relationship eager is not a general solution because it can over-fetch data and produce large joins.

## Key takeaway

Relationship mappings describe association semantics. Query design determines performance. Fetching should be selected for each use case rather than hidden behind global eager loading.

## References

- [Spring Data JPA: Entity Graphs](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html)
- [Hibernate ORM User Guide: Associations](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)
- [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/)
