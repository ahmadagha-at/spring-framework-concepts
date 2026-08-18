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

## Key takeaway

A test slice loads the infrastructure for one concern. It is broader than a unit test and narrower than `@SpringBootTest`.

## References

- [Spring Boot: Testing Spring Boot Applications](https://docs.spring.io/spring-boot/reference/testing/spring-boot-applications.html)
- [Spring Framework: MockMvc](https://docs.spring.io/spring-framework/reference/testing/mockmvc.html)
