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


## Session flow

A simplified session login works as follows:

```text
credentials submitted
→ server authenticates user
→ SecurityContext stored in session
→ session identifier sent in cookie
→ browser sends cookie on later requests
→ server restores SecurityContext
```

The cookie should be protected with appropriate `HttpOnly`, `Secure`, and `SameSite` settings. Session fixation protection should issue a new session identifier after authentication.

Logout can invalidate server-side state immediately.

## JWT validation

A resource server should validate more than the signature:

- allowed algorithm
- trusted issuer
- intended audience
- expiration
- not-before time where used
- key selection
- required claims

```text
well-formed token
≠
trusted token
```

Claims should be converted to application authorities through an explicit mapping. Accepting arbitrary role claims without issuer-specific rules can grant unintended permissions.

## Access and refresh tokens

Access tokens should be short-lived and sent only to resource servers. Refresh tokens are longer-lived credentials used to obtain new access tokens and require stronger storage and rotation controls.

A custom “refresh JWT” implementation is easy to get wrong. When OAuth 2.0 or OpenID Connect is required, use a trusted authorization server and standard flows rather than inventing a protocol.

## Token revocation tradeoff

Stateless validation means a resource server can accept a token until it expires even if account state changes. Possible strategies include:

- short token lifetime
- server-side revocation state
- token introspection
- rotating signing keys in emergencies
- checking critical account state

Each strategy moves the design along a spectrum between statelessness and immediate control.

## CSRF depends on credential transport

Browsers automatically send cookies, including session cookies. An attacker can cause a victim's browser to send a state-changing request, which creates CSRF risk.

A bearer token placed only in an explicit `Authorization` header is not automatically attached by the browser in the same way. However, storing bearer credentials in cookies can reintroduce CSRF concerns.

## CORS preflight

For some cross-origin requests, the browser first sends an `OPTIONS` preflight request. CORS processing must occur early enough for the browser to learn whether the real request is permitted.

Allow only required origins, methods, and headers. Combining wildcard origins with credentials is unsafe and restricted by browsers.

## XSS and token storage

Moving a token from a cookie to browser storage changes the threat model. JavaScript-accessible storage makes token theft through XSS easier. HttpOnly cookies reduce JavaScript access but require correct CSRF protection.

There is no universally safe storage choice independent of the client architecture.

## OAuth 2.0 roles

Keep the participants distinct:

| Participant | Responsibility |
|---|---|
| Resource owner | Grants access |
| Client | Requests access |
| Authorization server | Authenticates and issues tokens |
| Resource server | Validates tokens and protects APIs |

OpenID Connect adds identity information for login scenarios. The ID token is intended for the client and should not automatically be treated as an API access token.

## Key takeaway

Choose sessions, bearer tokens, and OAuth flows based on trust boundaries and clients. Then configure CSRF, CORS, token validation, and storage consistently with that choice.

## References

- [Spring Security: OAuth 2.0 Resource Server](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/index.html)
- [Spring Security: CSRF](https://docs.spring.io/spring-security/reference/servlet/exploits/csrf.html)
- [Spring Security: CORS](https://docs.spring.io/spring-security/reference/servlet/integrations/cors.html)
- [Spring Security: Security HTTP Response Headers](https://docs.spring.io/spring-security/reference/servlet/exploits/headers.html)
