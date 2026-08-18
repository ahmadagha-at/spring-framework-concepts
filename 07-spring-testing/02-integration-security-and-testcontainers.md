# Part 2: Integration Tests, Security Tests, and Testcontainers

Integration tests verify that real application components cooperate correctly.

## Full application tests

```java
@SpringBootTest
@AutoConfigureMockMvc
class OrderApplicationTest {

    @Autowired
    MockMvc mockMvc;
}
```

A full context test is useful for checking application configuration, bean wiring, filters, transactions, and cross-layer behavior.

With a random web environment, tests can start a real embedded server and use an HTTP client. This is appropriate when network-level behavior must be verified.

## Security tests

Spring Security Test provides request processors and annotations for authenticated scenarios:

```java
@Test
@WithMockUser(authorities = "order:read")
void allowsAuthorizedUser() throws Exception {
    mockMvc.perform(get("/orders/1"))
            .andExpect(status().isOk());
}
```

Tests should cover:

- anonymous access
- authenticated access
- missing authorities
- resource ownership
- CSRF behavior
- invalid or expired tokens
- method security

## Testcontainers

Testcontainers starts disposable infrastructure in Docker:

```java
@Testcontainers
@SpringBootTest
class PostgreSqlIntegrationTest {

    @Container
    static PostgreSQLContainer<?> postgres =
            new PostgreSQLContainer<>("postgres:latest");
}
```

Spring Boot service connections can connect supported containers to application configuration.

Using the same database family as production catches differences that an in-memory database may hide. Container startup is slower, so these tests should focus on integration behavior rather than every business branch.

## Reliable tests

Tests should be isolated, deterministic, and independent of execution order. Control time, generated identifiers, external services, and asynchronous waiting explicitly.


## Integration-test application flow

A valuable integration test follows a complete use case:

```text
authenticated request
→ security filters
→ controller validation
→ transactional service
→ repository
→ PostgreSQL container
→ committed response
```

This can reveal missing beans, wrong filter configuration, invalid mappings, serialization errors, and transaction problems that isolated tests cannot detect.

## Dynamic container configuration

Containers use dynamic ports and credentials. Spring Boot service connections or dynamic property registration can supply these values without hard-coding them.

```java
@DynamicPropertySource
static void databaseProperties(
        DynamicPropertyRegistry registry
) {
    registry.add(
            "spring.datasource.url",
            postgres::getJdbcUrl
    );
}
```

The application under test should connect only after the container is ready.

## Reusing migrations

Integration tests should apply the same Flyway or Liquibase migrations used in production. Recreating a simplified test schema manually can hide migration mistakes and missing indexes.

A useful startup test proves:

```text
empty database
→ all migrations apply
→ application context starts
→ mappings validate
```

## Transactional tests can hide behavior

Annotating a full test with `@Transactional` rolls back convenient test data, but it can also hide when production code commits, flushes, publishes events, or accesses lazy relationships after a transaction.

Use transactional tests intentionally. For transaction-bound events or real HTTP server tests, allow the application to control its own transaction boundaries and clean data explicitly.

## Security test realism

`@WithMockUser` creates an authenticated principal without running the real login or token validation flow. It is excellent for authorization tests but cannot prove that JWT decoding, issuer validation, or password authentication is configured correctly.

Add selected tests with real signed test tokens or the configured authentication endpoint when those mechanisms are part of the required behavior.

## External services

For HTTP integrations, use a controllable stub server rather than real third-party production endpoints. Test:

- expected request method and body
- timeouts
- error responses
- retry behavior
- idempotency
- invalid payloads

End-to-end tests against shared external environments are slower and less deterministic.

## Parallel execution

Containers and shared database state can conflict when tests run in parallel. Use isolated schemas, unique data, or separate containers when parallelism matters. Never rely on test ordering.

## Key takeaway

Integration tests provide confidence across real boundaries. Security support creates precise identities, while Testcontainers supplies realistic disposable infrastructure.

## References

- [Spring Security: Testing](https://docs.spring.io/spring-security/reference/servlet/test/)
- [Spring Boot: Testcontainers](https://docs.spring.io/spring-boot/reference/testing/testcontainers.html)
- [Testcontainers for Java](https://java.testcontainers.org/)
