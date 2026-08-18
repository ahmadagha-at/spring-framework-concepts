# Part 3: Sessions, JWT, OAuth 2.0, and Request Protection

Authentication state can be represented in different ways. Sessions and bearer tokens solve related problems but have different operational properties.

## Session-based authentication

After authentication, the server can associate the security context with an HTTP session. The client sends a session identifier, while the authoritative authentication state remains on the server.

Sessions support immediate server-side invalidation but require session storage or distribution when an application scales across instances.

## JWT bearer tokens

A JWT is a signed token format containing claims. A resource server can validate its signature, issuer, audience, and expiration without loading server-side session state for every request.

A signed JWT is not encrypted. Its payload must not contain secrets.

```text
Bearer token
→ signature and claim validation
→ Authentication
→ authorization
```

Statelessness shifts responsibilities: token expiration, revocation strategy, key rotation, refresh tokens, and secure client storage must be designed explicitly.

## OAuth 2.0 and OpenID Connect

OAuth 2.0 is an authorization framework for delegated access. OpenID Connect adds an identity layer. They are not synonyms for JWT, and OAuth tokens are not required to use the JWT format.

## CSRF

Cross-Site Request Forgery exploits browser behavior that automatically attaches credentials such as cookies. CSRF protection is especially relevant for session or cookie-based authentication.

Disabling CSRF simply because an application exposes JSON endpoints is unsafe. The credential transport mechanism determines the risk.

## CORS

CORS is a browser policy controlling whether frontend code from another origin may read responses. It is not authentication and does not protect an API from non-browser clients.

## Passwords and headers

Use adaptive one-way password hashing and allow Spring Security to provide protective headers. Security headers, TLS, validation, authorization, and safe secret handling form separate defense layers.

## Key takeaway

Choose sessions, bearer tokens, and OAuth flows based on trust boundaries and clients. Then configure CSRF, CORS, token validation, and storage consistently with that choice.

## References

- [Spring Security: OAuth 2.0 Resource Server](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/index.html)
- [Spring Security: CSRF](https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html)
- [Spring Security: CORS](https://docs.spring.io/spring-security/reference/servlet/integrations/cors.html)
- [Spring Security: Security HTTP Response Headers](https://docs.spring.io/spring-security/reference/servlet/exploits/headers.html)
