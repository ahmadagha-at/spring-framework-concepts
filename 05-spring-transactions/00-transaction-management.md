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


## What happens around a transactional method

A typical intercepted call follows this sequence:

```text
caller invokes proxy
→ transaction metadata is read
→ transaction manager joins or creates a transaction
→ database resources are associated with the execution
→ target method runs
→ transaction commits or rolls back
→ resources are released
```

The target service does not manually pass a connection to every repository. Spring coordinates resource access around the method boundary.

With JPA, the transaction manager coordinates an `EntityManager`. Repository operations inside the same transaction participate in the same persistence context.

## Atomicity does not replace validation

A transaction prevents partial database changes, but it does not decide whether a business operation is valid.

```java
@Transactional
public void transfer(AccountId sourceId,
                     AccountId targetId,
                     Money amount) {

    Account source = accounts.getRequired(sourceId);
    Account target = accounts.getRequired(targetId);

    source.withdraw(amount);
    target.deposit(amount);
}
```

The domain objects still enforce rules such as sufficient balance and positive amounts. The transaction only ensures that the resulting persistence operations succeed or fail together.

## Runtime and checked exceptions

Default rollback rules matter:

```java
@Transactional
public void importFile() throws IOException {
}
```

A checked exception does not necessarily trigger rollback by default. If an exception represents a failed unit of work, configure the rule explicitly or translate it into an application exception with the intended semantics.

```java
@Transactional(rollbackFor = IOException.class)
```

Rollback rules should be deliberate rather than added after inconsistent behavior appears.

## Database constraints and rollback

Java validation can provide early, friendly errors, while database constraints remain the final protection against invalid persisted state.

A unique constraint can still fail during flush. When this happens, the transaction is normally no longer usable and should be allowed to roll back. Catching the exception and continuing with more writes in the same transaction is unsafe.

## Transactions and external systems

A database transaction does not automatically include email, HTTP calls, or message brokers.

```text
database commit succeeds
email sending fails
```

This cannot be solved merely by adding `@Transactional`. Reliable cross-system workflows may require an outbox pattern, idempotency, retries, or distributed coordination.

## Testing transaction behavior

Useful integration tests verify:

- all writes roll back after a failure
- database constraints are enforced
- transaction propagation matches the use case
- events or messages are published at the intended phase
- concurrent updates are detected

## Key takeaway

A transaction is a business consistency boundary. Spring supplies a technology-independent abstraction and applies transaction behavior around selected application methods.

## References

- [Spring Framework: Transaction Management](https://docs.spring.io/spring-framework/reference/data-access/transaction.html)
- [Spring Framework: Declarative Transaction Management](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative.html)
