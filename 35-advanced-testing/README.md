# 🧪 Advanced Testing & Testcontainers — Complete In-Depth Guide

> **"A codebase without tests is legacy code the moment it is written. Advanced Spring Boot testing goes beyond basic unit tests to spin up real databases using Docker and test your entire application end-to-end."**

---

## 📑 Table of Contents

1. [The Testing Pyramid](#1-the-testing-pyramid)
2. [Spring Boot Test Slices](#2-spring-boot-test-slices)
3. [Testing the Web Layer (`@WebMvcTest`)](#3-testing-the-web-layer-webmvctest)
4. [Testing the Data Layer (`@DataJpaTest`)](#4-testing-the-data-layer-datajpatest)
5. [Integration Testing (`@SpringBootTest`)](#5-integration-testing-springboottest)
6. [The Problem with In-Memory Databases (H2)](#6-the-problem-with-in-memory-databases-h2)
7. [What are Testcontainers?](#7-what-are-testcontainers)
8. [Testcontainers Setup & Example](#8-testcontainers-setup--example)
9. [Mocking External APIs (WireMock)](#9-mocking-external-apis-wiremock)
10. [Test Best Practices](#10-test-best-practices)
11. [Interview Questions & Answers (50+)](#11-interview-questions--answers-50)

---

## 1. The Testing Pyramid

*   **Unit Tests (70%)**: Test individual methods/classes in isolation. Use `Mockito` to mock dependencies. Fast execution.
*   **Integration Tests (20%)**: Test how components work together (e.g., Service + Database).
*   **End-to-End (E2E) Tests (10%)**: Test the entire application from the outside in (HTTP Request → Controller → Service → Real DB). Slowest execution.

---

## 2. Spring Boot Test Slices

Spring Boot provides "Slices" to test specific layers of your application without loading the entire heavy application context.

| Annotation | What it Loads | What it Ignores | Use Case |
|------------|---------------|-----------------|----------|
| `@WebMvcTest` | Controllers, Filters, Security | Services, Repositories | Testing HTTP routes, JSON serialization, auth. |
| `@DataJpaTest` | Repositories, EntityManager, DB connection | Controllers, Services | Testing custom SQL queries, transactions. |
| `@JsonTest` | Jackson ObjectMapper | Everything else | Testing JSON serialization/deserialization. |
| `@SpringBootTest` | EVERYTHING (The entire app) | Nothing | True Integration/E2E testing. |

---

## 3. Testing the Web Layer (`@WebMvcTest`)

Loads *only* the web layer. You must use `@MockBean` to provide fake implementations for your Services.

```java
@WebMvcTest(UserController.class) // Only load UserController!
class UserControllerTest {

    @Autowired
    private MockMvc mockMvc; // Used to simulate HTTP requests

    @MockBean
    private UserService userService; // Provide a mock service

    @Test
    void shouldReturnUserById() throws Exception {
        // 1. Arrange (Mock the behavior)
        User mockUser = new User(1L, "Alice");
        when(userService.findById(1L)).thenReturn(mockUser);

        // 2. Act & Assert (Perform HTTP request and check response)
        mockMvc.perform(get("/users/1"))
                .andExpect(status().isOk())
                .andExpect(jsonPath("$.name").value("Alice"))
                .andExpect(jsonPath("$.id").value(1));
    }
}
```

---

## 4. Testing the Data Layer (`@DataJpaTest`)

Loads *only* JPA components and automatically configures an in-memory database (like H2) and makes every test transactional (rolls back after each test).

```java
@DataJpaTest
class UserRepositoryTest {

    @Autowired
    private UserRepository userRepository;

    @Test
    void shouldFindUserByEmail() {
        // Arrange
        User user = new User("bob@mail.com");
        userRepository.save(user);

        // Act
        Optional<User> found = userRepository.findByEmail("bob@mail.com");

        // Assert
        assertTrue(found.isPresent());
        assertEquals("bob@mail.com", found.get().getEmail());
        // Data is automatically rolled back after this method finishes!
    }
}
```

---

## 5. Integration Testing (`@SpringBootTest`)

Loads the entire application context. Useful for end-to-end tests.

```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class FullIntegrationTest {

    @Autowired
    private TestRestTemplate restTemplate; // Real HTTP client

    @Test
    void shouldCreateAndFetchUser() {
        // Real HTTP POST request
        User newUser = new User("Charlie");
        ResponseEntity<User> createResponse = restTemplate.postForEntity("/users", newUser, User.class);
        
        assertEquals(HttpStatus.CREATED, createResponse.getStatusCode());
        
        // Real HTTP GET request
        ResponseEntity<User> getResponse = restTemplate.getForEntity("/users/" + createResponse.getBody().getId(), User.class);
        
        assertEquals("Charlie", getResponse.getBody().getName());
    }
}
```

---

## 6. The Problem with In-Memory Databases (H2)

For years, developers used H2 (an in-memory Java DB) for integration testing.

**The Problem:**
1. H2 behaves differently than PostgreSQL/MySQL.
2. If you write a Postgres-specific query (e.g., `JSONB` operations, native queries), it will CRASH in your H2 tests.
3. Your tests pass locally, but fail in production because the databases are different!

**The Solution:** Test against a REAL database using **Testcontainers**.

---

## 7. What are Testcontainers?

Testcontainers is a Java library that supports JUnit tests, providing lightweight, throwaway instances of common databases, Selenium web browsers, or anything else that can run in a Docker container.

Instead of mocking a database, Testcontainers spins up a REAL PostgreSQL Docker container, runs your tests against it, and kills the container when the tests finish.

---

## 8. Testcontainers Setup & Example

**1. Dependencies**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-testcontainers</artifactId>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>postgresql</artifactId>
    <scope>test</scope>
</dependency>
```

**2. The Integration Test**
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers // Tells JUnit to manage containers
class PostgresIntegrationTest {

    // Spin up a real Postgres container!
    @Container
    @ServiceConnection // Spring Boot 3.1+ magic: auto-configures the datasource URL/username/password!
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15-alpine");

    @Autowired
    private UserRepository userRepository;

    @Test
    void testRealDatabase() {
        assertTrue(postgres.isRunning());
        
        userRepository.save(new User("Test User"));
        assertEquals(1, userRepository.count());
    }
}
```
*Note: Docker must be running on your machine for Testcontainers to work.*

---

## 9. Mocking External APIs (WireMock)

If your service calls an external API (like GitHub or Stripe), you shouldn't make real HTTP calls during tests. Instead, use **WireMock** to spin up a fake server.

```java
@SpringBootTest
@AutoConfigureWireMock(port = 8081) // Starts fake server on port 8081
class GithubClientTest {

    @Autowired
    private GithubClient githubClient;

    @Test
    void testGithubApi() {
        // Tell the fake server how to respond
        stubFor(get(urlEqualTo("/users/octocat"))
            .willReturn(aResponse()
                .withHeader("Content-Type", "application/json")
                .withBody("{\"login\": \"octocat\", \"id\": 1}")));

        // Call your client (make sure it's configured to hit localhost:8081 in test properties)
        GithubUser user = githubClient.getUser("octocat");

        assertEquals("octocat", user.getLogin());
    }
}
```

---

## 10. Test Best Practices

1. **Given-When-Then**: Structure your tests logically.
   * `Given` (Arrange): Setup data.
   * `When` (Act): Call the method.
   * `Then` (Assert): Verify results.
2. **Don't use H2 for Integration Tests**: Use Testcontainers to ensure production parity.
3. **Use AssertJ**: Replace JUnit's `assertEquals` with AssertJ's fluent API: `assertThat(user.getName()).isEqualTo("Alice")`.
4. **Don't over-mock**: Mock external systems, but test your internal business logic using real objects where possible.
5. **Test unhappy paths**: Don't just test success. Test what happens when the DB fails, or the input is null.

---

## 11. Interview Questions & Answers (50+)

### Beginner

**Q1: What is Unit Testing?** Testing the smallest piece of code (a single method or class) in complete isolation.

**Q2: What is Integration Testing?** Testing multiple components together (e.g., a Controller, Service, and Repository interacting).

**Q3: What is Mockito?** A Java mocking framework used to create fake objects (mocks) to simulate the behavior of real dependencies.

**Q4: `@Mock` vs `@InjectMocks`?** `@Mock` creates a fake object. `@InjectMocks` creates a real object and injects the `@Mock` objects into it.

**Q5: What is `MockMvc`?** A Spring testing utility that allows you to test Spring MVC controllers without starting a real HTTP server.

**Q6: What is a Testcontainer?** A library that runs throwaway Docker containers for databases/message brokers during tests.

**Q7: Why not use H2 for testing a PostgreSQL application?** H2 doesn't support Postgres-specific features (like JSONB or native functions). If you use them, tests fail locally but pass in prod, or vice versa.

---

### Intermediate

**Q8: What does `@SpringBootTest` do?** Bootstraps the entire Spring Application Context. Very heavy, slow to start, used for end-to-end tests.

**Q9: What does `@WebMvcTest` do?** Only loads the web layer (Controllers, Filters). Much faster than `@SpringBootTest`.

**Q10: What does `@DataJpaTest` do?** Only loads JPA components, configures an in-memory database, and makes tests transactional.

**Q11: Why are `@DataJpaTest` methods transactional?** So that any database changes made during the test are automatically rolled back at the end, ensuring tests don't affect each other (test isolation).

**Q12: What is `@MockBean`?** A Spring Boot annotation that adds a Mockito mock to the Spring ApplicationContext, replacing any existing bean of the same type.

**Q13: Difference between `@Mock` and `@MockBean`?** `@Mock` is pure Mockito (no Spring context). `@MockBean` is used within Spring tests (like `@WebMvcTest`) to replace a bean in the Spring Context.

**Q14: What is WireMock?** A tool for mocking HTTP-based APIs. Used to test code that relies on third-party services.

**Q15: What is `@ServiceConnection` in Spring Boot 3.1?** Auto-configures database properties (URL, username, password) directly from a Testcontainer instance, replacing the need for `DynamicPropertySource`.

---

### Rapid-Fire (Q16–Q50)

**Q16: `assertEquals(expected, actual)` or `assertEquals(actual, expected)`?** It is ALWAYS `assertEquals(expected, actual)`.

**Q17: What is AssertJ?** A fluent assertion library. `assertThat(actual).isEqualTo(expected)`. Much more readable than JUnit assertions.

**Q18: What is `when().thenReturn()`?** Mockito syntax to define how a mock should behave when a specific method is called.

**Q19: What is `verify()` in Mockito?** Checks if a specific method on a mock was called (and how many times). `verify(mock, times(1)).doSomething()`.

**Q20: What is a Spy in Mockito?** A partial mock. Real methods are called by default, but you can choose to mock specific methods.

**Q21: How do you mock a `void` method?** `doNothing().when(mock).voidMethod()`.

**Q22: How to test exceptions in JUnit 5?** `assertThrows(ExpectedException.class, () -> methodThatThrows());`.

**Q23: How do you disable a test?** `@Disabled("Reason for disabling")`.

**Q24: What is TDD?** Test-Driven Development. Write the failing test first, write code to pass the test, then refactor.

**Q25: What is Code Coverage?** A metric measuring the percentage of source code executed by tests.

**Q26: Popular Java coverage tool?** JaCoCo.

**Q27: Should you aim for 100% test coverage?** No, it leads to diminishing returns and brittle tests. Aim for 70-80% covering critical business logic.

**Q28: How to test private methods?** You shouldn't. Test the public methods that call the private methods. If the private method is too complex, extract it to a new class.

**Q29: What does `RandomPort` do in `@SpringBootTest`?** Starts a real embedded web server on a random available port to avoid port conflicts.

**Q30: What is `TestRestTemplate`?** A RestTemplate configured specifically for testing, useful for making real HTTP calls against the `RandomPort` server.

**Q31: What is `WebTestClient`?** A non-blocking, reactive client for testing WebFlux apps (can also test MVC apps).

**Q32: How do you pass active profiles in tests?** `@ActiveProfiles("test")`.

**Q33: Can you use multiple profiles?** Yes, `@ActiveProfiles({"test", "integration"})`.

**Q34: What happens to the Spring Context between tests?** By default, Spring caches the Context. It is reused across test classes to save time, unless the context is modified.

**Q35: What does `@DirtiesContext` do?** Tells Spring to destroy and recreate the ApplicationContext for the next test. Use sparingly, as it makes test suites very slow.

**Q36: How to mock static methods?** Since Mockito 3.4.0, use `mockStatic()`.

**Q37: What is Mutation Testing?** A tool (like PIT) that modifies your source code (mutants) and checks if your tests fail. If tests still pass, your tests are weak!

**Q38: Why do integration tests fail in CI/CD but pass locally?** Often due to environment differences, missing environment variables, or port conflicts.

**Q39: How does Testcontainers work in CI/CD?** As long as the CI runner has a Docker daemon (Docker-in-Docker), Testcontainers spins up perfectly.

**Q40: Do Testcontainers leave garbage containers behind?** No, Testcontainers uses a library called "Ryuk" to ensure containers are killed even if the test crashes.

**Q41: What is `@DataMongoTest`?** Slices the context for testing MongoDB repositories.

**Q42: What is `@JsonTest`?** Slices the context to test Jackson serialization (`@JsonSerialize`, `@JsonDeserialize`).

**Q43: What is the `ArgumentCaptor` in Mockito?** Captures arguments passed to a mock method so you can assert against them.

**Q44: What is BDD?** Behavior-Driven Development. Given-When-Then format. Frameworks: Cucumber.

**Q45: `BDDMockito` vs standard `Mockito`?** BDDMockito provides aliases for Mockito methods to fit Given-When-Then: `given(mock.get()).willReturn("value")`.

**Q46: Can I test Spring Security?** Yes, use `@WithMockUser(username="admin", roles={"ADMIN"})` to simulate a logged-in user.

**Q47: How to test file uploads?** Using `MockMultipartFile` with `MockMvc`.

**Q48: How to test a scheduled task (`@Scheduled`)?** Extract the logic into a separate method and test that method directly.

**Q49: What is Hamcrest?** A framework for writing matcher objects (`assertThat(x, is(y))`). Mostly superseded by AssertJ.

**Q50: Why do we write tests?** To catch bugs early, facilitate fearless refactoring, act as living documentation, and ensure code quality.

---

## 📚 References

- [Spring Boot Testing Docs](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.testing)
- [Testcontainers Java](https://www.testcontainers.org/)
- [Mockito](https://site.mockito.org/)
- [AssertJ](https://assertj.github.io/doc/)

---

> **Previous Topic:** [← 34 - Caching & Redis](../34-caching-redis/README.md)  
> **Next Topic:** [36 - GraalVM Native Image →](../36-graalvm-native/README.md)
