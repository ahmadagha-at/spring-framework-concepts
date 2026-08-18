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

## Key takeaway

Integration tests provide confidence across real boundaries. Security support creates precise identities, while Testcontainers supplies realistic disposable infrastructure.

## References

- [Spring Security: Testing](https://docs.spring.io/spring-security/reference/servlet/test/)
- [Spring Boot: Testcontainers](https://docs.spring.io/spring-boot/reference/testing/testcontainers.html)
- [Testcontainers for Java](https://java.testcontainers.org/)
