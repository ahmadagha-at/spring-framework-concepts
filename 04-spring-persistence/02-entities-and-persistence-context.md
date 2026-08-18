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


## What happens when an entity is loaded

When `find()` loads an entity, the provider first checks whether that identity is already present in the persistence context. If it is, the managed instance can be returned. Otherwise, SQL may be executed, a Java object is created, its fields are populated, and the instance is registered as managed.

```java
User first = entityManager.find(User.class, 1L);
User second = entityManager.find(User.class, 1L);

assert first == second;
```

Within the same persistence context, both variables normally refer to the same managed instance. This prevents two competing in-memory representations of the same row inside one unit of work.

## Persistence context operations

Important `EntityManager` operations include:

| Operation | Effect |
|---|---|
| `persist(entity)` | Makes a new entity managed |
| `find(type, id)` | Returns a managed entity when found |
| `getReference(type, id)` | May return a lazy reference |
| `detach(entity)` | Stops tracking one entity |
| `clear()` | Detaches all managed entities |
| `remove(entity)` | Marks a managed entity for deletion |
| `merge(entity)` | Copies state into a managed instance |
| `flush()` | Synchronizes pending changes with the database |

Calling `clear()` does not undo SQL that has already been executed. Rollback is a transaction operation, while clearing is a persistence-context operation.

## Transaction-scoped context

In a typical Spring service:

```java
@Transactional
public UserResponse rename(Long id, String name) {
    User user = repository.findById(id).orElseThrow();
    user.rename(name);
    return mapper.toResponse(user);
}
```

The transaction interceptor opens or joins a transaction. The JPA infrastructure supplies an `EntityManager` associated with the current transaction. The loaded `User` remains managed until that context ends.

After the method returns and the transaction completes, the entity is normally detached. Accessing an uninitialized lazy association afterward can fail because no active persistence context is available to load it.

## Entities and DTOs

An entity represents persistent identity and participates in change tracking. A DTO represents data crossing an application boundary.

Returning entities directly from controllers creates several risks:

- lazy associations may execute queries during serialization
- bidirectional relationships may recurse
- internal columns can become part of the API
- client input can overwrite fields that should not be writable
- API changes become coupled to database mappings

Map entities to request and response models inside a controlled transaction boundary.

## Persistence context is not a global cache

The first-level cache belongs to one persistence context. It is not shared across all requests and is not a general replacement for application caching. A new transaction may use a new context and load the row again.

Second-level caching is a separate optional provider feature with different invalidation and consistency concerns.

## Key takeaway

An entity represents persistent identity and state. The persistence context tracks managed entity instances and coordinates their changes with the database.

## References

- [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/)
- [Hibernate ORM User Guide: Persistence Context](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)
- [Spring Framework: JPA](https://docs.spring.io/spring-framework/reference/data-access/orm/jpa.html)
