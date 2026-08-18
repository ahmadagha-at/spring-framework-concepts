# Part 1: Filter Chain and Authentication

Spring Security's Servlet architecture processes requests through a chain of filters.

## DelegatingFilterProxy

The servlet container knows about servlet filters, while Spring manages beans. `DelegatingFilterProxy` connects these environments by delegating filter work to a Spring bean.

## FilterChainProxy

Spring Security uses `FilterChainProxy`, commonly reached through the bean named `springSecurityFilterChain`. It selects a matching `SecurityFilterChain` for the request.

```java
@Bean
SecurityFilterChain securityFilterChain(HttpSecurity http)
        throws Exception {

    return http
            .authorizeHttpRequests(authorize -> authorize
                    .requestMatchers("/public/**").permitAll()
                    .anyRequest().authenticated()
            )
            .build();
}
```

Multiple chains can exist for different request matchers. The first matching chain is used, so ordering matters.

## Authentication

An authentication filter extracts credentials or a token and creates an authentication request. An `AuthenticationManager` delegates to one or more `AuthenticationProvider` instances.

```text
credentials
→ Authentication filter
→ AuthenticationManager
→ AuthenticationProvider
→ authenticated Authentication
```

Providers implement specific mechanisms, such as username/password authentication.

## SecurityContextHolder

After successful authentication, the resulting `Authentication` is stored in a `SecurityContext`, which is accessed through `SecurityContextHolder`.

The context normally applies to the current execution. Passing work to another thread requires explicit consideration because security context does not automatically follow every asynchronous boundary.

## Password storage

Passwords must not be stored in plaintext or encrypted for later recovery. A `PasswordEncoder` applies a one-way adaptive password hash. The encoded result includes information needed to verify future attempts.


## Multiple security filter chains

Different parts of an application can use different authentication models:

```java
@Bean
@Order(1)
SecurityFilterChain api(HttpSecurity http) {
    return http
        .securityMatcher("/api/**")
        .oauth2ResourceServer(oauth -> oauth.jwt())
        .build();
}
```

```java
@Bean
SecurityFilterChain web(HttpSecurity http) {
    return http
        .formLogin(Customizer.withDefaults())
        .build();
}
```

The first matching chain is selected. A broad matcher with higher priority can prevent another chain from ever being used.

## Authentication object lifecycle

Before authentication, an `Authentication` object can contain submitted credentials and report `isAuthenticated() == false`. After a provider validates the request, a new authenticated object commonly contains:

- the principal
- granted authorities
- authentication details
- no reusable plaintext password

Credentials should be erased when they are no longer needed.

## AuthenticationManager and providers

A `ProviderManager` asks compatible providers in order. A provider can:

- authenticate the request
- decline because it does not support that authentication type
- reject invalid credentials

For username/password authentication, a provider may load a user through `UserDetailsService` and compare the submitted password through `PasswordEncoder`.

For JWT resource-server authentication, a provider validates the token and converts claims into authorities.

## Security context persistence

Session-based applications can save the security context between requests. Stateless bearer-token applications reconstruct authentication from the token on each request.

At the end of request processing, thread-local state must be cleared. Spring Security's filters manage this lifecycle so authentication does not leak into later work handled by the same server thread.

## Authentication failures

Failed authentication and denied authorization take different paths:

```text
no valid identity
→ AuthenticationEntryPoint

valid identity, insufficient permission
→ AccessDeniedHandler
```

For REST APIs, these handlers should return consistent status codes and response bodies rather than redirecting to an HTML login page.

## Password verification

Adaptive password encoders deliberately consume CPU and memory. Parameters should be tuned so verification is expensive for attackers but acceptable for the application.

A delegating encoder can store an algorithm identifier with the hash, allowing future upgrades without treating hashes as reversible encryption.

## Custom filters

Custom authentication filters should be added only when built-in mechanisms do not fit. Filter position matters because context loading, exception handling, and authorization occur at defined points.

A filter that parses JWT manually but bypasses standard validation, error handling, or authority conversion creates unnecessary security risk.

## Key takeaway

Spring Security authenticates requests through ordered filters and providers. The successful identity is stored in a security context and becomes input for later authorization decisions.

## References

- [Spring Security: Servlet Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
- [Spring Security: Authentication Architecture](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html)
- [Spring Security: Password Storage](https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html)
