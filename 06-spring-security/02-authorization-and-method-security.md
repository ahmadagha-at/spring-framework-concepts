# Part 2: Authorization and Method Security

Authorization decides whether an authenticated or anonymous principal may access a resource.

## Authorities and roles

An authenticated `Authentication` contains `GrantedAuthority` values. These values represent permissions available to the principal.

A role is a naming convention built on authorities. For example, checks using `hasRole("ADMIN")` commonly expect an authority named `ROLE_ADMIN`.

## Request authorization

```java
http.authorizeHttpRequests(authorize -> authorize
        .requestMatchers(HttpMethod.GET, "/products/**")
            .permitAll()
        .requestMatchers("/admin/**")
            .hasRole("ADMIN")
        .anyRequest()
            .authenticated()
);
```

Rules should move from specific matchers to general matchers. A broad rule placed too early can make later rules unreachable.

## Method security

Request rules protect HTTP endpoints. Method security protects application operations regardless of how they are invoked.

```java
@PreAuthorize("hasAuthority('order:refund')")
public void refund(Long orderId) {
}
```

Method security is useful for protecting service-layer capabilities. It should not replace ordinary domain validation, such as confirming that a resource belongs to the current user.

## Ownership checks

Roles alone are often insufficient. Two users may share the same role but must only access their own resources. Authorization may therefore require both an authority and a domain relationship.

## Denied access

Unauthenticated access and insufficient authority are different failures. Authentication entry points handle cases where authentication is required. Access-denied handling applies when an authenticated principal lacks permission.


## AuthorizationManager

Modern Spring Security represents authorization decisions through `AuthorizationManager`. The manager receives the current authentication and information about the protected object, such as an HTTP request or method invocation.

This separates:

```text
authentication mechanism
from
authorization decision
```

A session login and a JWT can produce equivalent authorities, allowing the same authorization rules to protect application operations.

## Request matcher order

Rules are evaluated according to configuration order:

```java
authorize
    .requestMatchers("/admin/reports/public").permitAll()
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .anyRequest().authenticated();
```

Placing `/admin/**` first would also match the public report path. Security configuration should be tested as ordered behavior, not read as an unordered list.

## Method-security interception

With method security enabled, an eligible bean is proxied. Before the target method executes, an authorization interceptor evaluates the expression.

```java
@PreAuthorize("hasAuthority('order:refund')")
public RefundResult refund(OrderId id) {
}
```

Post-authorization can inspect returned data, but it runs after the method has executed. It must not be used when the method could already perform a forbidden side effect.

## Domain ownership

An authority grants a general capability. Ownership checks constrain the resource instance:

```java
@PreAuthorize(
    "hasAuthority('order:read') " +
    "and @orderAuthorization.canRead(#orderId, authentication)"
)
public OrderResponse getOrder(Long orderId) {
}
```

Authorization services should make policies explicit and testable. Hiding database access inside complex expression strings makes policies harder to review.

## Avoid insecure direct object references

Protecting an endpoint with `authenticated()` does not stop one user from requesting another user's identifier.

Every operation that accepts an object ID should answer:

- Can this principal perform this action?
- Can this principal perform it on this specific object?
- Is tenant or organization membership required?
- Should administrators bypass ownership?

## Roles versus fine-grained authorities

Broad roles are convenient for grouping, but application decisions often become clearer with authorities such as:

```text
order:read
order:create
order:refund
user:manage
```

Roles can still map to collections of authorities. This keeps service policies focused on capabilities rather than organizational labels.

## Testing the negative paths

For every protected operation, test at least:

- anonymous principal
- authenticated principal without authority
- correct authority but wrong resource owner
- valid owner and authority
- administrative override where supported

## Key takeaway

Authorization consumes the authenticated identity and applies request-level, method-level, and domain-level rules. Correct authorization is more precise than assigning broad roles.

## References

- [Spring Security: Authorization Architecture](https://docs.spring.io/spring-security/reference/servlet/authorization/architecture.html)
- [Spring Security: Authorize HTTP Requests](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html)
- [Spring Security: Method Security](https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html)
