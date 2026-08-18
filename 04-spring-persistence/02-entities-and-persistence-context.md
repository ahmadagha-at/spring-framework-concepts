# Part 2: Entities and the Persistence Context

An entity is a domain object whose identity and state can be persisted. The persistence context is the environment in which entity instances are tracked.

## A basic entity

```java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false, unique = true)
    private String email;

    protected User() {
    }

    public User(String email) {
        this.email = email;
    }
}
```

`@Entity` marks the class as persistent metadata. `@Id` identifies its primary key. Mapping annotations describe how object state corresponds to relational structures.

## Entity identity

Database identity is different from object reference identity. Two Java instances may represent the same database row if they have the same persistent identifier. This affects implementations of `equals()` and `hashCode()`, especially before generated identifiers exist.

## EntityManager

The Jakarta Persistence `EntityManager` provides operations such as:

```java
User user = entityManager.find(User.class, id);
entityManager.persist(new User("user@example.com"));
entityManager.remove(user);
```

It interacts with a persistence context rather than immediately executing SQL for every method call.

## First-level cache

Within one persistence context, loading the same entity identity normally returns the same managed instance. This identity map is sometimes called the first-level cache.

## Persistence context boundaries

In Spring applications, the persistence context is commonly associated with a transaction. Entities loaded inside it are managed. After the context closes, those objects become detached.

Keeping the persistence context open through the web layer can hide lazy-loading problems and produce uncontrolled database access. Transaction boundaries should normally follow application use cases.

## Key takeaway

An entity represents persistent identity and state. The persistence context tracks managed entity instances and coordinates their changes with the database.

## References

- [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/)
- [Hibernate ORM User Guide: Persistence Context](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)
- [Spring Framework: JPA](https://docs.spring.io/spring-framework/reference/data-access/orm/jpa.html)
