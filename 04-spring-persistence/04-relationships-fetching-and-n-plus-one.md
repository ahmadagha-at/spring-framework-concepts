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


## Owning and inverse sides

In a bidirectional relationship, both Java objects can reference each other, but only one side controls the database relationship.

```java
@Entity
class Order {

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "customer_id")
    private Customer customer;
}
```

```java
@Entity
class Customer {

    @OneToMany(mappedBy = "customer")
    private List<Order> orders = new ArrayList<>();
}
```

The `Order.customer` side owns the foreign key. `Customer.orders` is the inverse view because `mappedBy` points to the owning field.

Updating only the inverse collection may leave the database relationship unchanged. Helper methods keep both sides consistent:

```java
public void addOrder(Order order) {
    orders.add(order);
    order.assignCustomer(this);
}
```

## Cascade and orphan removal

Cascade settings propagate operations from one entity to related entities:

```java
@OneToMany(
    mappedBy = "order",
    cascade = CascadeType.ALL,
    orphanRemoval = true
)
private List<OrderItem> items = new ArrayList<>();
```

This can be appropriate when order items belong exclusively to one order. It is dangerous for shared entities such as users, roles, or products.

`orphanRemoval = true` means that removing a child from the relationship can schedule its row for deletion. This is stronger than merely clearing a foreign key.

## Why N+1 appears

Suppose one query loads 100 orders. Accessing a lazy customer association for each order can cause 100 additional selects. The Java loop looks harmless because database access is hidden behind entity navigation.

```java
for (Order order : orders) {
    log.info(order.getCustomer().getEmail());
}
```

Inspecting only the repository method misses the real query pattern. SQL logging, metrics, or profiling should confirm the number of round trips.

## Fetch joins and entity graphs

A fetch join expresses that one query should initialize an association:

```java
@Query("""
       select o
       from Order o
       join fetch o.customer
       where o.status = :status
       """)
List<Order> findWithCustomerByStatus(OrderStatus status);
```

An entity graph describes fetch requirements separately from the JPQL text:

```java
@EntityGraph(attributePaths = "customer")
List<Order> findByStatus(OrderStatus status);
```

Both approaches should be use-case-specific. Fetching several collections in one query can multiply rows and make pagination unreliable.

## DTO projections

For read-only API views, a projection can select exactly the required columns and avoid navigating an entity graph:

```java
select new com.example.OrderSummary(
    o.id,
    o.status,
    c.email
)
from Order o
join o.customer c
```

This separates read performance from write-oriented entity mappings.

## Key takeaway

Relationship mappings describe association semantics. Query design determines performance. Fetching should be selected for each use case rather than hidden behind global eager loading.

## References

- [Spring Data JPA: Entity Graphs](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html)
- [Hibernate ORM User Guide: Associations](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)
- [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/)
