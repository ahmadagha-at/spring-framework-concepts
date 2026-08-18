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

## Key takeaway

Authorization consumes the authenticated identity and applies request-level, method-level, and domain-level rules. Correct authorization is more precise than assigning broad roles.

## References

- [Spring Security: Authorization Architecture](https://docs.spring.io/spring-security/reference/servlet/authorization/architecture.html)
- [Spring Security: Authorize HTTP Requests](https://docs.spring.io/spring-security/reference/servlet/authorization/authorize-http-requests.html)
- [Spring Security: Method Security](https://docs.spring.io/spring-security/reference/servlet/authorization/method-security.html)
