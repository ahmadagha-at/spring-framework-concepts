# Part 1: Transactional Proxies, Propagation, and Isolation

`@Transactional` is commonly implemented with Spring AOP proxies. Understanding this mechanism explains several surprising behaviors.

## Proxy-based interception

```text
caller → Spring proxy → transaction interceptor → target method
```

The proxy begins the transaction before invoking the target and completes it afterward.

## Self-invocation

A call from one method to another method on the same object does not normally pass through the proxy:

```java
public void process() {
    saveInTransaction();
}

@Transactional
public void saveInTransaction() {
}
```

Because `process()` calls `this.saveInTransaction()`, the interceptor may not run. Put the transactional operation behind another injected bean, redesign the boundary, or use an appropriate alternative intentionally.

## Propagation

Propagation describes how a method relates to an existing transaction.

| Propagation | Meaning |
|---|---|
| `REQUIRED` | Join an existing transaction or create one |
| `REQUIRES_NEW` | Suspend the current transaction and create another |
| `SUPPORTS` | Join if one exists; otherwise run without one |
| `MANDATORY` | Require an existing transaction |
| `NOT_SUPPORTED` | Run without a transaction |
| `NEVER` | Fail when a transaction exists |
| `NESTED` | Use a nested transaction when supported |

`REQUIRED` is the default and suits most service-layer use cases. `REQUIRES_NEW` changes atomicity and should not be used merely to hide rollback problems.

## Isolation

Isolation controls which concurrent transaction effects can be observed. Common levels include read committed, repeatable read, and serializable. Stronger isolation can prevent anomalies but may reduce concurrency.

The database and driver determine the effective behavior. Application code should not select an isolation level without understanding the workload and database guarantees.

## Read-only transactions

`readOnly = true` expresses intent and may enable optimizations. It is not a universal security mechanism that makes writes impossible.

## Key takeaway

`@Transactional` works through interception. Propagation controls transaction participation, while isolation controls concurrent visibility. Both affect correctness and must follow the use case rather than habit.

## References

- [Spring Framework: Declarative Transaction Implementation](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-decl-explained.html)
- [Spring Framework: Transaction Propagation](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html)
