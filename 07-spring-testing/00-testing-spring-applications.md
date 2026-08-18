# Testing Spring Applications

Spring applications can be tested at different levels. A larger test context is not automatically a better test.

## The testing pyramid

```text
many focused unit tests
→ selected slice tests
→ fewer integration tests
→ a small number of end-to-end tests
```

Each level provides different confidence and cost.

## Unit tests

A unit test creates the class directly without starting Spring:

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    PaymentClient paymentClient;

    @Test
    void rejectsPaymentFailure() {
        OrderService service =
                new OrderService(paymentClient);

        when(paymentClient.charge(any()))
                .thenThrow(new PaymentException());

        assertThrows(
                PaymentException.class,
                () -> service.placeOrder(request)
        );
    }
}
```

Constructor injection makes this possible. Unit tests are fast and explain business behavior clearly.

## Spring integration tests

Spring's TestContext Framework loads an `ApplicationContext` and manages test integration such as dependency injection, transactions, and cached contexts.

`@SpringBootTest` starts a context through Spring Boot. Use it when the interaction of the complete application configuration matters, not as the default for every class.

## What to test

Tests should verify observable behavior:

- returned values and state changes
- database constraints and mappings
- HTTP status, headers, and JSON
- authentication and authorization decisions
- transaction rollback
- integration with external boundaries

Avoid asserting private implementation details.


## Test boundaries

A test boundary defines which parts are real and which parts are replaced.

A service unit test can use real domain objects and mocked infrastructure:

```text
real service
+ real domain rules
+ mocked repository
+ mocked external client
```

A repository integration test uses real mapping and database behavior:

```text
real repository proxy
+ real EntityManager
+ real Hibernate mappings
+ test database
```

A full application test can include filters, controllers, services, repositories, and infrastructure. Because failure diagnosis becomes broader, full-context tests should cover important integration paths rather than every small rule.

## Arrange, act, assert

A clear test separates preparation, execution, and verification:

```java
@Test
void rejectsDuplicateEmail() {
    // arrange
    when(users.existsByEmail("user@example.com"))
            .thenReturn(true);

    // act
    Executable operation = () ->
            registration.register(command);

    // assert
    assertThrows(
            EmailAlreadyUsedException.class,
            operation
    );
    verify(users, never()).save(any());
}
```

The test name should describe behavior, not the method implementation.

## Mocking decisions

Mock boundaries such as repositories, clocks, message publishers, and remote clients when isolating a service rule. Do not mock simple value objects or the class being tested.

Over-mocking can make a test pass even when real components cannot integrate. A mock returns what the test instructs it to return; it does not verify SQL mappings, serialization, framework configuration, or network protocols.

## Test data builders

Large constructors make tests difficult to read. Builders or fixtures can supply valid defaults while each test overrides only relevant values.

Fixtures should still make important state visible. A “magic” fixture with hidden relationships can make failures difficult to understand.

## Time and randomness

Code using `Instant.now()`, random UUIDs, or nondeterministic ordering can produce unstable tests. Inject abstractions such as `Clock` when time affects behavior:

```java
Clock fixed = Clock.fixed(
        Instant.parse("2026-01-01T10:00:00Z"),
        ZoneOffset.UTC
);
```

Deterministic inputs make failures repeatable.

## Test failures should be informative

Prefer assertions that show meaningful differences. AssertJ provides fluent collection and object assertions. Verify the complete relevant response rather than many unrelated implementation calls.

A test suite is executable documentation. A future reader should understand the business rule from the test without reading the production method first.

## Key takeaway

Choose the smallest test that can provide the required confidence. Unit tests verify isolated behavior, slices verify one application layer, and integration tests verify components working together.

## References

- [Spring Framework: Testing](https://docs.spring.io/spring-framework/reference/testing.html)
- [Spring Boot: Testing](https://docs.spring.io/spring-boot/reference/testing/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org.mockito/org/mockito/Mockito.html)
