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


## Threat modeling

Security configuration should begin with the assets and trust boundaries of the application.

For a backend API, relevant questions include:

- Which data is sensitive?
- Which clients are trusted?
- Where do credentials enter the system?
- Which operations change valuable state?
- Can one tenant access another tenant's data?
- Which external systems can influence authorization?
- What happens when a token, password, or signing key is compromised?

A threat model does not need to predict every attack. Its purpose is to connect controls to concrete risks.

```text
asset
→ possible attacker
→ attack path
→ preventive control
→ detection and recovery
```

For example, protecting an order endpoint with authentication addresses anonymous access, but it does not address cross-tenant object access. That risk requires ownership or tenant authorization.

## Defense in depth

No single security mechanism is sufficient. A secure write operation can combine:

1. TLS protects data in transit.
2. Authentication establishes the caller identity.
3. Request authorization checks the required capability.
4. Domain authorization checks resource ownership.
5. Input validation rejects malformed data.
6. The domain model enforces business invariants.
7. Database constraints protect persisted integrity.
8. Audit logging records the sensitive decision.
9. Monitoring detects unusual failure patterns.

If one layer is bypassed or misconfigured, another layer can still limit the impact.

## Security errors should not leak details

Authentication failures should not reveal whether a username exists, which signature check failed, or which internal provider rejected a request. Server logs can contain diagnostic context, while client responses remain stable and limited.

Sensitive values such as passwords, complete tokens, session identifiers, and secrets must never be written to logs.

## References

- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
- [Spring Security: Servlet Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
