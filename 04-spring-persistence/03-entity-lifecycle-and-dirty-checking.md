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


## State transitions

A common lifecycle looks like this:

```text
new User()
→ transient
→ persist()
→ managed
→ transaction ends
→ detached
```

Deletion follows another transition:

```text
managed
→ remove()
→ removed
→ flush and commit
→ row deleted
```

A detached entity still has identity and data, but the current persistence context no longer tracks its changes.

## How dirty checking works conceptually

When an entity becomes managed, Hibernate can retain information about its loaded state. Before flushing, it compares the current state with the tracked state or uses enhanced change tracking. If persistent attributes changed, it schedules an update.

```text
loaded state:  email = old@example.com
current state: email = new@example.com
→ attribute changed
→ UPDATE scheduled
```

Dirty checking only applies to managed entities. Changing a detached object does not automatically update the database.

## Why save is often misunderstood

Inside a transaction, this is enough:

```java
User user = repository.findById(id).orElseThrow();
user.activate();
```

The loaded entity is managed, so dirty checking can persist the change.

Calling `save(user)` again is often redundant. It may not break the code, but it hides the more important rule: the persistence context is tracking the entity.

For a detached object, `save()` may lead to `merge()`. The returned managed instance should be used:

```java
User managed = entityManager.merge(detached);
```

The `detached` argument remains detached.

## Flush timing

A provider can flush:

- when a transaction commits
- before executing a query that must observe pending changes
- after an explicit `flush()`
- according to the configured flush mode

This means a constraint violation may appear before the final commit. Explicit flushing is useful when the application must detect a database failure at a particular point, but frequent flushing reduces batching opportunities.

## Optimistic locking

Concurrent updates can overwrite one another. A version attribute enables optimistic locking:

```java
@Version
private long version;
```

The generated update includes the expected version. If another transaction changed the row first, the update affects no row and the provider reports a conflict.

Optimistic locking does not prevent concurrent work. It detects conflicting writes so the application can reject or retry them intentionally.

## Common mistakes

- modifying a detached entity and expecting automatic persistence
- treating `merge()` as reattaching the same Java object
- assuming flush and commit are identical
- relying only on Java validation instead of database constraints
- omitting versioning when lost updates matter

## Key takeaway

Automatic updates are not magic. They result from a managed entity, change tracking, flushing, and transaction completion.

## References

- [Spring Data JPA: Persisting Entities](https://docs.spring.io/spring-data/jpa/reference/jpa/entity-persistence.html)
- [Hibernate ORM User Guide: Flushing](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)
- [Jakarta Persistence Specification](https://jakarta.ee/specifications/persistence/)
