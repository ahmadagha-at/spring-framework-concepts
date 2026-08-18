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

## References

- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
- [Spring Security: Servlet Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
