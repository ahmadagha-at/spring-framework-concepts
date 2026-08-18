# Spring Transaction Management

A transaction groups data-access operations into one logical unit of work. Either the complete unit succeeds, or its changes are rolled back.

## Why transactions matter

Consider transferring money:

```text
debit source account
credit target account
record transfer
```

If only part of this sequence succeeds, the database becomes inconsistent. A transaction defines the boundary within which these operations must be coordinated.

## Spring's transaction abstraction

Spring provides a common transaction model through `PlatformTransactionManager`. Implementations coordinate a specific resource technology, such as JDBC or JPA.

Application code commonly uses declarative transactions:

```java
@Service
public class TransferService {

    @Transactional
    public void transfer(
            Long sourceId,
            Long targetId,
            BigDecimal amount
    ) {
        debit(sourceId, amount);
        credit(targetId, amount);
    }
}
```

The annotation describes transactional metadata. Infrastructure around the method begins, commits, or rolls back the transaction.

## Declarative and programmatic management

Declarative transactions keep transaction policy near application use cases. Programmatic management through `TransactionTemplate` is useful when code requires explicit control over boundaries or return values.

## Rollback behavior

By default, Spring's declarative transaction support normally rolls back for unchecked exceptions and errors. Checked exception behavior must be configured when rollback is required.

Catching an exception inside the transactional method can prevent the transaction interceptor from seeing it. Do not swallow failures that should cause rollback.

## Transaction boundaries

Transactions usually belong in the application service layer, where a complete use case is coordinated. Controllers should handle HTTP concerns, and repositories should focus on persistence operations.

## Key takeaway

A transaction is a business consistency boundary. Spring supplies a technology-independent abstraction and applies transaction behavior around selected application methods.

## References

- [Spring Framework: Transaction Management](https://docs.spring.io/spring-framework/reference/data-access/transaction.html)
- [Spring Framework: Declarative Transaction Management](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative.html)
