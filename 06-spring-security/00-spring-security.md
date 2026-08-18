# Spring Security

Spring Security provides authentication, authorization, and protection against common attacks. Its Servlet support is built primarily around filters.

## Scope of this series

The articles explain:

- the security filter chain
- authentication and the security context
- authorization at request and method level
- sessions, JWT, and OAuth 2.0
- CSRF, CORS, passwords, and security headers
- testing secured applications

## Security is cross-cutting

Security applies before a controller executes and can also protect service methods.

```text
HTTP request
→ servlet container
→ Spring Security filters
→ DispatcherServlet
→ controller
→ service
```

Spring Boot can configure defaults, but Spring Security remains the framework that performs the security work.

## Authentication and authorization

Authentication answers:

> Who is making the request?

Authorization answers:

> Is that identity allowed to perform this action?

These phases are related but distinct. A user may be successfully authenticated and still lack permission for an operation.


## Security responsibilities

Spring Security provides infrastructure, but secure behavior also depends on application design.

| Concern | Example |
|---|---|
| Authentication | Validate a session or bearer token |
| Request authorization | Protect `/admin/**` |
| Method authorization | Protect a service operation |
| Domain authorization | Confirm the current user owns an order |
| Exploit protection | CSRF and security headers |
| Credential handling | Password hashing and token validation |
| Audit | Record security-relevant decisions |

A controller matcher alone cannot express every domain rule. Conversely, domain checks should not reimplement token parsing or password verification.

## A request through the security system

```text
request enters servlet container
→ matching SecurityFilterChain is selected
→ security context is loaded
→ credentials or token are processed
→ Authentication is established
→ request authorization runs
→ DispatcherServlet handles allowed request
→ method security may run
→ security context is saved or cleared
```

Not every filter authenticates. Some manage context, authorization, logout, exceptions, headers, or exploit protection.

## Default-deny design

Security rules are easier to review when public access is explicit and the fallback requires authentication:

```java
authorize.requestMatchers("/health").permitAll()
         .requestMatchers("/public/**").permitAll()
         .anyRequest().authenticated();
```

For sensitive operations, authentication alone is insufficient. Require narrowly defined authorities and ownership rules.

## Security boundaries

Treat every input as untrusted until validated:

- HTTP parameters and bodies
- headers
- tokens
- uploaded files
- redirect URLs
- data received from other services

Authentication proves a property about the caller. It does not make all caller-supplied data safe.

## Configuration is part of the security model

A secure implementation can become insecure through deployment configuration. Important operational concerns include TLS, trusted proxies, cookie flags, allowed origins, secret storage, token issuers, key rotation, and exposed Actuator endpoints.

The application should fail clearly when required security configuration is missing.

## References

- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
- [Spring Security: Servlet Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
