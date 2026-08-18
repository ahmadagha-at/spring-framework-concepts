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

## Key takeaway

Choose the smallest test that can provide the required confidence. Unit tests verify isolated behavior, slices verify one application layer, and integration tests verify components working together.

## References

- [Spring Framework: Testing](https://docs.spring.io/spring-framework/reference/testing.html)
- [Spring Boot: Testing](https://docs.spring.io/spring-boot/reference/testing/)
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org.mockito/org/mockito/Mockito.html)
