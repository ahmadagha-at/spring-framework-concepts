# Part 5: Spring Data JPA Repositories

Spring Data JPA creates repository implementations from interfaces. It reduces repetitive data-access code but still operates through Jakarta Persistence and a provider such as Hibernate.

## Repository interfaces

```java
public interface UserRepository
        extends JpaRepository<User, Long> {

    Optional<User> findByEmail(String email);
}
```

The interface is not manually implemented. Spring Data inspects its type information and methods, creates a proxy, and registers that proxy as a Spring bean.

## Repository hierarchy

Important abstractions include:

- `Repository` as a marker
- `CrudRepository` for CRUD operations
- `ListCrudRepository` with list-based results
- `PagingAndSortingRepository` for paging and sorting
- `JpaRepository` with JPA-specific operations

Extending the largest interface is convenient, but application repositories do not need to expose every operation automatically.

## What save means

```java
User saved = repository.save(user);
```

For a new entity, Spring Data JPA normally calls `EntityManager.persist()`. Otherwise it calls `EntityManager.merge()`. Therefore, `save()` is not a universal SQL update command.

## Repository proxies

The generated implementation delegates to JPA infrastructure and participates in Spring exception translation and transaction behavior. Custom behavior can still be implemented through repository fragments or separate query services.

## Return types

Repositories can return:

- `Optional<T>`
- collections
- `Page<T>` and `Slice<T>`
- projections
- streams where appropriate

Pagination prevents loading unbounded result sets. A `Page` normally requires an additional count query, while a `Slice` only determines whether another slice exists.

## Key takeaway

A Spring Data repository is a generated Spring proxy around a persistence abstraction. It removes boilerplate, but it does not remove the need to understand entity state, transactions, queries, and SQL performance.

## References

- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/)
- [Spring Data: Repository Definitions](https://docs.spring.io/spring-data/jpa/reference/repositories/definition.html)
- [Spring Data: Core Repository Concepts](https://docs.spring.io/spring-data/jpa/reference/repositories/core-concepts.html)
