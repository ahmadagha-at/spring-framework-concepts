# Part 3: Entity Lifecycle and Dirty Checking

An entity can move through several lifecycle states. Understanding these states explains why some modifications are persisted automatically while others are ignored.

## Lifecycle states

| State | Meaning |
|---|---|
| Transient | New object not associated with a persistence context |
| Managed | Tracked by the current persistence context |
| Detached | Has persistent identity but is no longer tracked |
| Removed | Scheduled for deletion |

## Persisting a new entity

```java
User user = new User("user@example.com");
entityManager.persist(user);
```

The object begins as transient and becomes managed after `persist()`.

## Dirty checking

A persistence provider tracks managed entities. If a managed entity changes during a transaction, Hibernate can detect the difference and generate an update during flushing.

```java
@Transactional
public void changeEmail(Long id, String email) {
    User user = repository.findById(id).orElseThrow();
    user.changeEmail(email);
}
```

A second explicit `save()` is not required for this managed entity. The important conditions are the managed state and an active transaction.

## Flush is not commit

Flushing synchronizes pending changes with the database connection. Committing completes the transaction.

```text
entity change → dirty checking → flush → SQL → commit
```

A flush can occur before commit, for example before a query. SQL execution does not by itself mean that the transaction has committed successfully.

## Persist and merge

`persist()` makes a new entity managed. `merge()` copies state from a detached object into a managed instance and returns that managed instance. The argument itself does not become managed.

This distinction matters because Spring Data JPA's `save()` chooses between persistence and merge behavior depending on whether the entity is considered new.

## Key takeaway

Automatic updates are not magic. They result from a managed entity, change tracking, flushing, and transaction completion.

## References

- [Spring Data JPA: Persisting Entities](https://docs.spring.io/spring-data/jpa/reference/jpa/entity-persistence.html)
- [Hibernate ORM User Guide: Flushing](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)
- [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/)
