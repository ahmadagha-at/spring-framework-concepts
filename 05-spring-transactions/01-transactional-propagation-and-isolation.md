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


## REQUIRED and rollback-only behavior

With `REQUIRED`, inner methods normally participate in the outer physical transaction.

```java
@Transactional
public void placeOrder() {
    inventory.reserve();
    payment.record();
}
```

If an inner participant marks the transaction rollback-only, catching its exception does not necessarily make the transaction committable. The outer method may finish normally but receive an unexpected rollback result during completion.

The correct response is usually to let the failure abort the use case or redesign independent work into a genuinely separate transaction.

## REQUIRES_NEW

`REQUIRES_NEW` suspends the current transaction and starts an independent one.

An audit record written in a new transaction can commit even when the main transaction rolls back. That may be intentional, but it also means the operations are no longer atomic.

It requires another database connection while the outer transaction still holds its resources. Excessive use can exhaust a small connection pool.

## Isolation anomalies

Isolation levels address concurrent behavior:

| Anomaly | Description |
|---|---|
| Dirty read | Reading data another transaction has not committed |
| Non-repeatable read | Reading the same row twice and observing a committed change |
| Phantom read | Repeating a range query and observing new matching rows |
| Lost update | One write overwrites another without detecting the conflict |

Actual guarantees depend on the database implementation, not only the Java annotation.

Optimistic locking with `@Version` is often a clearer solution for lost updates than globally increasing isolation.

## Transaction timeout

A timeout limits how long transactional work may run. It does not guarantee that every external call is interrupted immediately; database and driver behavior also matter.

Long transactions keep connections and locks for longer, increase contention, and retain more managed entities. Avoid remote network calls or user interaction inside database transactions when possible.

## Proxy placement

Transactional annotations are most predictable on public service methods invoked from another bean.

Annotating private helper methods or calling an annotated method through `this` does not create a new interception boundary. The question is not merely where the annotation appears, but how the method is invoked.

## Events after commit

Some side effects should occur only after successful commit. Transaction-bound event listeners can react during a selected transaction phase.

This is useful for in-process behavior but does not make external delivery durable. If a process crashes after commit but before publishing externally, an outbox remains the stronger reliability pattern.

## Choosing settings

Start with the default `REQUIRED` propagation and database default isolation. Change them only after expressing the consistency requirement:

- Which operations must be atomic?
- Which operation may commit independently?
- Which concurrent changes must be detected?
- Can the use case retry safely?

## Key takeaway

`@Transactional` works through interception. Propagation controls transaction participation, while isolation controls concurrent visibility. Both affect correctness and must follow the use case rather than habit.

## References

- [Spring Framework: Declarative Transaction Implementation](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-decl-explained.html)
- [Spring Framework: Transaction Propagation](https://docs.spring.io/spring-framework/reference/data-access/transaction/declarative/tx-propagation.html)
