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


## How the implementation is created

At startup, Spring Data scans repository interfaces. For each suitable interface, it creates metadata describing the domain type, identifier type, query methods, and fragments. A proxy is registered as a Spring bean.

```text
repository interface
→ repository metadata
→ generated proxy
→ SimpleJpaRepository and query implementations
→ EntityManager
```

The proxy can also apply transaction and exception-translation infrastructure. There is no generated Java source file that developers edit.

## Repository responsibility

A repository represents access to an aggregate or persistent domain concept. It should not become a container for unrelated queries simply because they use the same table.

Application services coordinate use cases:

```java
@Service
public class RegistrationService {

    private final UserRepository users;
    private final PasswordEncoder passwords;

    @Transactional
    public UserId register(RegisterUser command) {
        if (users.existsByEmail(command.email())) {
            throw new EmailAlreadyUsedException();
        }

        User user = new User(
                command.email(),
                passwords.encode(command.password())
        );

        return new UserId(users.save(user).getId());
    }
}
```

The repository handles persistence. The service owns the use-case transaction and orchestration.

## Generated methods are still database operations

Convenient method names do not guarantee efficient execution. For example:

```java
Page<Order> findByCustomerId(Long customerId, Pageable pageable);
```

A `Page` normally requires a data query and a count query. Associated data can still trigger N+1 queries. Generated repository code must be reviewed with the same care as handwritten persistence code.

## Custom repository fragments

When behavior does not fit derived queries or `@Query`, a repository can be composed from a custom fragment:

```java
interface UserSearchRepository {
    List<UserSummary> search(UserFilter filter);
}
```

A custom implementation can use the `EntityManager`, Criteria API, Querydsl, or another data-access tool without forcing the main interface to contain complex query construction.

## Exception translation

Persistence providers throw technology-specific exceptions. Spring can translate supported exceptions into the `DataAccessException` hierarchy. This gives application infrastructure a consistent exception model, but application code should still translate technical failures into meaningful use-case results where appropriate.

## Repository tests

Repository tests should verify mappings and real query behavior, not Spring Data itself. Useful tests include:

- derived method semantics
- custom JPQL
- constraints
- ordering and pagination
- projections
- locking behavior
- database-specific queries

## Key takeaway

A Spring Data repository is a generated Spring proxy around a persistence abstraction. It removes boilerplate, but it does not remove the need to understand entity state, transactions, queries, and SQL performance.

## References

- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/)
- [Spring Data: Repository Definitions](https://docs.spring.io/spring-data/jpa/reference/repositories/definition.html)
- [Spring Data: Core Repository Concepts](https://docs.spring.io/spring-data/jpa/reference/repositories/core-concepts.html)
