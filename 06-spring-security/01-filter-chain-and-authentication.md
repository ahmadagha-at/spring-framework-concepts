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

## Key takeaway

Spring Security authenticates requests through ordered filters and providers. The successful identity is stored in a security context and becomes input for later authorization decisions.

## References

- [Spring Security: Servlet Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
- [Spring Security: Authentication Architecture](https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html)
- [Spring Security: Password Storage](https://docs.spring.io/spring-security/reference/features/authentication/password-storage.html)
