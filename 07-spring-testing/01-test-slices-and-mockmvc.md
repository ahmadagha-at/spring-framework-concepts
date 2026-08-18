# Part 1: Test Slices and MockMvc

Test slices load focused parts of a Spring Boot application. They provide framework integration without starting every application component.

## Web MVC tests

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

    @Autowired
    MockMvc mockMvc;

    @MockBean
    UserService userService;

    @Test
    void returnsUser() throws Exception {
        when(userService.getUser(1L))
                .thenReturn(new UserResponse(1L, "user@example.com"));

        mockMvc.perform(get("/users/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.email")
                        .value("user@example.com"));
    }
}
```

A web slice focuses on controllers, request mapping, validation, serialization, exception handling, and MVC configuration. Collaborating application services are normally replaced.

## Data JPA tests

```java
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    UserRepository repository;

    @Test
    void findsUserByEmail() {
        repository.save(new User("user@example.com"));

        assertThat(repository.findByEmail("user@example.com"))
                .isPresent();
    }
}
```

A JPA slice focuses on entities, repositories, mappings, and persistence behavior.

An embedded database can differ from the production database in SQL syntax, constraints, and query plans. Testcontainers is preferable when database-specific behavior matters.

## MockMvc

MockMvc exercises Spring MVC request handling without requiring a real network server. It still tests routing, argument resolution, filters, validation, and response rendering.

## Context caching

Spring caches compatible test contexts. Excessive custom configuration or mock combinations can create many distinct contexts and slow the suite.


## What a slice includes

A slice is selected through auto-configuration and type filtering. `@WebMvcTest` does not merely start a smaller server; it loads MVC-related infrastructure and selected web components while excluding ordinary service and repository beans.

This is why dependencies of a controller must be supplied as test beans or mocks.

`@DataJpaTest` configures JPA-oriented infrastructure and repository scanning. It commonly wraps each test in a transaction that rolls back afterward.

## MockMvc request lifecycle

MockMvc performs request processing through Spring MVC:

```text
mock request
→ filters when configured
→ DispatcherServlet
→ handler mapping
→ controller
→ validation and conversion
→ exception resolvers
→ response serialization
```

There is no real TCP connection, but most MVC behavior is exercised.

## Validation example

```java
mockMvc.perform(post("/users")
        .contentType(MediaType.APPLICATION_JSON)
        .content("""
                 {
                   "email": "invalid",
                   "password": ""
                 }
                 """))
    .andExpect(status().isBadRequest())
    .andExpect(jsonPath("$.errors.email").exists())
    .andExpect(jsonPath("$.errors.password").exists());
```

This verifies JSON deserialization, Bean Validation, controller handling, and error-response formatting together.

## Controller advice

A web slice should include relevant `@ControllerAdvice` behavior. Tests should verify the public error contract for:

- invalid input
- missing resources
- business conflicts
- malformed JSON
- authentication and authorization failures

This prevents each controller from developing a different response format.

## Repository testing details

Repository tests should flush when database behavior must be observed immediately:

```java
repository.save(user);
entityManager.flush();
```

Without flushing, an invalid constraint may not be checked until the test transaction ends. Flushing at the assertion point makes the failure attributable to the tested operation.

Clear the persistence context when a test must prove that data is truly loaded from the database rather than returned from the first-level cache:

```java
entityManager.flush();
entityManager.clear();
```

## Slice versus full context

Choose a slice when the question is focused:

- Does this request map correctly?
- Does validation produce the right response?
- Does this repository query return the right rows?
- Does JSON have the intended shape?

Use a full context when the question crosses several configured modules.

## Key takeaway

A test slice loads the infrastructure for one concern. It is broader than a unit test and narrower than `@SpringBootTest`.

## References

- [Spring Boot: Testing Spring Boot Applications](https://docs.spring.io/spring-boot/reference/testing/spring-boot-applications.html)
- [Spring Framework: MockMvc](https://docs.spring.io/spring-framework/reference/testing/mockmvc.html)
