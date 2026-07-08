# 🧪 JUnit — Complete In-Depth Guide (Beginner to Expert)

> **"Untested code is broken code. JUnit is the foundation of Java testing that ensures your code works as expected, today and after every change."**

---

## 📑 Table of Contents

1. [Introduction & Overview](#1-introduction--overview)
2. [JUnit 5 Architecture](#2-junit-5-architecture)
3. [Writing Your First Test](#3-writing-your-first-test)
4. [Assertions](#4-assertions)
5. [Test Lifecycle & Annotations](#5-test-lifecycle--annotations)
6. [Display Names & Nested Tests](#6-display-names--nested-tests)
7. [Parameterized Tests](#7-parameterized-tests)
8. [Assumptions](#8-assumptions)
9. [Exception Testing](#9-exception-testing)
10. [Timeout Testing](#10-timeout-testing)
11. [Test Ordering & Execution](#11-test-ordering--execution)
12. [Conditional Test Execution](#12-conditional-test-execution)
13. [Mockito — Mocking Framework](#13-mockito--mocking-framework)
14. [Spring Boot Testing](#14-spring-boot-testing)
15. [Test Driven Development (TDD)](#15-test-driven-development-tdd)
16. [Code Coverage with JaCoCo](#16-code-coverage-with-jacoco)
17. [Best Practices](#17-best-practices)
18. [Common Mistakes & Pitfalls](#18-common-mistakes--pitfalls)
19. [Interview Questions & Answers (50+)](#19-interview-questions--answers-50)

---

## 1. Introduction & Overview

### What is JUnit?

JUnit is the most popular **unit testing framework for Java**. It provides annotations, assertions, and test runners to write and execute automated tests. JUnit 5 is the current version, completely rewritten from JUnit 4.

### Why Testing Matters

- **Catch bugs early** — Before they reach production
- **Enable refactoring** — Safely change code knowing tests catch regressions
- **Documentation** — Tests describe expected behavior
- **Design improvement** — Testable code is typically better-designed code
- **Confidence** — Deploy with confidence that everything works
- **Cost reduction** — Fixing bugs in production is 10x–100x more expensive

### Testing Pyramid

```
        /   \
       / E2E \          ← Fewest: Slow, brittle, expensive
      /  Tests \
     /──────────\
    / Integration\      ← Some: Test component interactions
   /    Tests     \
  /────────────────\
 /    Unit Tests    \   ← Most: Fast, isolated, cheap
/────────────────────\
```

| Test Type | Speed | Scope | Count |
|-----------|-------|-------|-------|
| Unit | Fast (ms) | Single class/method | 70-80% of tests |
| Integration | Medium (s) | Multiple components | 15-25% of tests |
| E2E | Slow (min) | Full system | 5-10% of tests |

---

## 2. JUnit 5 Architecture

### Module Architecture

```
JUnit 5 = JUnit Platform + JUnit Jupiter + JUnit Vintage

┌────────────────────────────────────────────┐
│              JUnit 5                        │
│                                             │
│  ┌─────────────────────────────────────┐   │
│  │         JUnit Platform              │   │
│  │  Foundation for launching testing   │   │
│  │  frameworks on the JVM              │   │
│  │  (TestEngine API, Launcher API)     │   │
│  └──────────┬──────────────┬───────────┘   │
│             │              │               │
│  ┌──────────▼─────┐  ┌────▼───────────┐  │
│  │ JUnit Jupiter  │  │ JUnit Vintage  │  │
│  │ (JUnit 5 API)  │  │ (JUnit 3 & 4) │  │
│  │                │  │                │  │
│  │ @Test          │  │ Backward       │  │
│  │ @BeforeEach    │  │ compatibility  │  │
│  │ Assertions     │  │                │  │
│  │ Extensions     │  │                │  │
│  └────────────────┘  └────────────────┘  │
└────────────────────────────────────────────┘
```

### Dependencies (Maven)

```xml
<!-- JUnit 5 (Jupiter) -->
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>5.10.2</version>
    <scope>test</scope>
</dependency>

<!-- This single dependency includes:
     - junit-jupiter-api (annotations, assertions)
     - junit-jupiter-engine (test engine)
     - junit-jupiter-params (parameterized tests)
-->

<!-- Spring Boot includes this automatically -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
    <!-- Includes: JUnit 5, Mockito, AssertJ, Hamcrest, JSONassert, Spring Test -->
</dependency>
```

### Dependencies (Gradle)

```groovy
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.2'
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
    
    // Or with Spring Boot (includes everything)
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform() // REQUIRED for JUnit 5!
}
```

---

## 3. Writing Your First Test

### The Class Under Test

```java
public class Calculator {
    
    public int add(int a, int b) {
        return a + b;
    }
    
    public int subtract(int a, int b) {
        return a - b;
    }
    
    public int multiply(int a, int b) {
        return a * b;
    }
    
    public double divide(int a, int b) {
        if (b == 0) {
            throw new ArithmeticException("Cannot divide by zero");
        }
        return (double) a / b;
    }
    
    public boolean isEven(int number) {
        return number % 2 == 0;
    }
}
```

### The Test Class

```java
import org.junit.jupiter.api.*;
import static org.junit.jupiter.api.Assertions.*;

class CalculatorTest {
    
    private Calculator calculator;
    
    @BeforeEach
    void setUp() {
        calculator = new Calculator();
    }
    
    @Test
    @DisplayName("Should add two positive numbers correctly")
    void testAddPositiveNumbers() {
        // Arrange
        int a = 5;
        int b = 3;
        
        // Act
        int result = calculator.add(a, b);
        
        // Assert
        assertEquals(8, result, "5 + 3 should equal 8");
    }
    
    @Test
    @DisplayName("Should subtract two numbers correctly")
    void testSubtract() {
        assertEquals(2, calculator.subtract(5, 3));
    }
    
    @Test
    @DisplayName("Should multiply two numbers correctly")
    void testMultiply() {
        assertEquals(15, calculator.multiply(5, 3));
    }
    
    @Test
    @DisplayName("Should divide two numbers correctly")
    void testDivide() {
        assertEquals(2.5, calculator.divide(5, 2), 0.001);
    }
    
    @Test
    @DisplayName("Should throw ArithmeticException when dividing by zero")
    void testDivideByZero() {
        assertThrows(ArithmeticException.class, () -> {
            calculator.divide(5, 0);
        });
    }
    
    @Test
    @DisplayName("Should correctly identify even numbers")
    void testIsEven() {
        assertTrue(calculator.isEven(4));
        assertFalse(calculator.isEven(5));
    }
}
```

### AAA Pattern (Arrange-Act-Assert)

```java
@Test
void testUserCreation() {
    // ARRANGE — Set up test data and dependencies
    String name = "Dilip";
    String email = "dilip@example.com";
    
    // ACT — Perform the action being tested
    User user = new User(name, email);
    
    // ASSERT — Verify the result
    assertEquals(name, user.getName());
    assertEquals(email, user.getEmail());
    assertNotNull(user.getId());
}
```

---

## 4. Assertions

### JUnit Jupiter Assertions

```java
import static org.junit.jupiter.api.Assertions.*;

class AssertionsDemo {
    
    @Test
    void basicAssertions() {
        // Equality
        assertEquals(4, 2 + 2);
        assertEquals("hello", "hello");
        assertEquals(3.14, Math.PI, 0.01); // Delta for floating point
        assertNotEquals(5, 2 + 2);
        
        // Boolean
        assertTrue(5 > 3);
        assertFalse(3 > 5);
        
        // Null
        assertNull(null);
        assertNotNull("not null");
        
        // Same reference
        String s1 = "hello";
        String s2 = s1;
        assertSame(s1, s2);       // Same object reference
        assertNotSame(s1, new String("hello")); // Different objects
        
        // Instance
        assertInstanceOf(String.class, "hello");
        assertInstanceOf(Number.class, 42);
    }
    
    @Test
    void arrayAssertions() {
        int[] expected = {1, 2, 3, 4, 5};
        int[] actual = {1, 2, 3, 4, 5};
        assertArrayEquals(expected, actual);
    }
    
    @Test
    void iterableAssertions() {
        List<String> expected = List.of("a", "b", "c");
        List<String> actual = List.of("a", "b", "c");
        assertIterableEquals(expected, actual);
    }
    
    @Test
    void stringAssertions() {
        // Lines comparison (ignores trailing whitespace)
        String expected = "line1\nline2\nline3";
        String actual = "line1\nline2\nline3";
        assertLinesMatch(
            expected.lines().toList(),
            actual.lines().toList()
        );
    }
    
    @Test
    void groupedAssertions() {
        User user = new User("Dilip", "dilip@example.com", 25);
        
        // ALL assertions are executed, even if some fail
        // Collects all failures and reports them together
        assertAll("User properties",
            () -> assertEquals("Dilip", user.getName()),
            () -> assertEquals("dilip@example.com", user.getEmail()),
            () -> assertEquals(25, user.getAge()),
            () -> assertNotNull(user.getId())
        );
    }
    
    @Test
    void messageSupplier() {
        // Lazy message (only constructed if assertion fails)
        assertEquals(4, 2 + 2, () -> "Expensive message: " + computeDetails());
    }
}
```

### AssertJ (Fluent Assertions — Included in Spring Boot Test)

```java
import static org.assertj.core.api.Assertions.*;

class AssertJDemo {
    
    @Test
    void stringAssertions() {
        String name = "Spring Boot";
        
        assertThat(name)
            .isNotNull()
            .isNotEmpty()
            .isNotBlank()
            .startsWith("Spring")
            .endsWith("Boot")
            .contains("ring")
            .hasSize(11)
            .doesNotContain("Hibernate")
            .matches("Spring \\w+");
    }
    
    @Test
    void numberAssertions() {
        int age = 25;
        
        assertThat(age)
            .isPositive()
            .isGreaterThan(18)
            .isLessThan(100)
            .isBetween(18, 65)
            .isEven();
    }
    
    @Test
    void collectionAssertions() {
        List<String> fruits = List.of("apple", "banana", "cherry");
        
        assertThat(fruits)
            .isNotEmpty()
            .hasSize(3)
            .contains("banana")
            .containsExactly("apple", "banana", "cherry")
            .containsAnyOf("apple", "grape")
            .doesNotContain("mango")
            .allMatch(f -> f.length() > 3)
            .noneMatch(String::isEmpty)
            .first().isEqualTo("apple");
    }
    
    @Test
    void objectAssertions() {
        User user = new User("Dilip", "dilip@example.com");
        
        assertThat(user)
            .isNotNull()
            .hasFieldOrPropertyWithValue("name", "Dilip")
            .extracting(User::getEmail)
            .isEqualTo("dilip@example.com");
    }
    
    @Test
    void exceptionAssertions() {
        assertThatThrownBy(() -> {
            throw new IllegalArgumentException("Invalid input");
        })
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessage("Invalid input")
            .hasMessageContaining("Invalid");
        
        assertThatCode(() -> {
            int result = 2 + 2;
        }).doesNotThrowAnyException();
    }
    
    @Test
    void mapAssertions() {
        Map<String, Integer> scores = Map.of("Alice", 95, "Bob", 87, "Charlie", 92);
        
        assertThat(scores)
            .hasSize(3)
            .containsKey("Alice")
            .containsValue(95)
            .containsEntry("Alice", 95)
            .doesNotContainKey("Dave");
    }
}
```

---

## 5. Test Lifecycle & Annotations

### Lifecycle Annotations

```java
class LifecycleTest {
    
    @BeforeAll
    static void beforeAll() {
        // Runs ONCE before ALL tests in this class
        // MUST be static (unless @TestInstance(PER_CLASS))
        System.out.println("Before ALL tests");
        // Use for: expensive setup (DB connection, test containers)
    }
    
    @BeforeEach
    void beforeEach() {
        // Runs BEFORE EACH test method
        System.out.println("Before each test");
        // Use for: creating fresh test objects, resetting state
    }
    
    @Test
    void test1() {
        System.out.println("Test 1");
    }
    
    @Test
    void test2() {
        System.out.println("Test 2");
    }
    
    @AfterEach
    void afterEach() {
        // Runs AFTER EACH test method
        System.out.println("After each test");
        // Use for: cleanup, closing resources
    }
    
    @AfterAll
    static void afterAll() {
        // Runs ONCE after ALL tests in this class
        // MUST be static (unless @TestInstance(PER_CLASS))
        System.out.println("After ALL tests");
        // Use for: cleanup expensive resources
    }
}

// Output:
// Before ALL tests
// Before each test → Test 1 → After each test
// Before each test → Test 2 → After each test
// After ALL tests
```

### @TestInstance

```java
// By default: PER_METHOD — new test instance for each test method
// PER_CLASS — single test instance shared across all test methods

@TestInstance(TestInstance.Lifecycle.PER_CLASS)
class SharedInstanceTest {
    
    private int counter = 0;
    
    @BeforeAll
    void beforeAll() { // No longer needs to be static!
        System.out.println("Before ALL (non-static)");
    }
    
    @Test
    void test1() {
        counter++;
        assertEquals(1, counter); // Counter persists between tests!
    }
    
    @Test
    void test2() {
        counter++;
        assertEquals(2, counter);
    }
}
```

### Key Annotations Reference

| Annotation | Purpose |
|-----------|---------|
| `@Test` | Marks a method as a test |
| `@BeforeEach` | Runs before each test |
| `@AfterEach` | Runs after each test |
| `@BeforeAll` | Runs once before all tests (static) |
| `@AfterAll` | Runs once after all tests (static) |
| `@DisplayName` | Custom display name for test |
| `@Disabled` | Disables a test |
| `@Tag` | Tags tests for filtering |
| `@Nested` | Nested test class |
| `@RepeatedTest` | Repeat test N times |
| `@ParameterizedTest` | Parameterized test |
| `@TestMethodOrder` | Control test execution order |
| `@Timeout` | Fail if test exceeds time limit |
| `@TempDir` | Inject a temporary directory |
| `@ExtendWith` | Register extensions |

---

## 6. Display Names & Nested Tests

### Display Names

```java
@DisplayName("User Service Tests")
class UserServiceTest {
    
    @Test
    @DisplayName("Should create user with valid data ✅")
    void testCreateUser() { }
    
    @Test
    @DisplayName("Should throw exception for duplicate email ❌")
    void testDuplicateEmail() { }
    
    @Test
    @DisplayName("Should return empty Optional for non-existent user 🔍")
    void testFindNonExistentUser() { }
}

// DisplayName generators
@DisplayNameGeneration(DisplayNameGenerator.ReplaceUnderscores.class)
class User_Service_Test {
    
    @Test
    void should_create_user_with_valid_data() { } // Displays: "should create user with valid data"
    
    @Test
    void should_throw_for_duplicate_email() { }
}
```

### Nested Tests

```java
@DisplayName("UserService")
class UserServiceTest {
    
    private UserService userService;
    private UserRepository userRepository;
    
    @BeforeEach
    void setUp() {
        userRepository = mock(UserRepository.class);
        userService = new UserService(userRepository);
    }
    
    @Nested
    @DisplayName("When creating a user")
    class CreateUser {
        
        @Test
        @DisplayName("should save user with valid data")
        void shouldSaveValidUser() {
            User user = new User("Dilip", "dilip@example.com");
            when(userRepository.save(any())).thenReturn(user);
            
            User result = userService.createUser(user);
            
            assertNotNull(result);
            verify(userRepository).save(user);
        }
        
        @Test
        @DisplayName("should throw for null name")
        void shouldThrowForNullName() {
            assertThrows(IllegalArgumentException.class, () -> {
                userService.createUser(new User(null, "email@test.com"));
            });
        }
        
        @Nested
        @DisplayName("With duplicate email")
        class WithDuplicateEmail {
            
            @Test
            @DisplayName("should throw DuplicateEmailException")
            void shouldThrowDuplicateEmail() {
                when(userRepository.existsByEmail("existing@test.com")).thenReturn(true);
                
                assertThrows(DuplicateEmailException.class, () -> {
                    userService.createUser(new User("Test", "existing@test.com"));
                });
            }
        }
    }
    
    @Nested
    @DisplayName("When finding a user")
    class FindUser {
        
        @Test
        @DisplayName("should return user when found")
        void shouldReturnUserWhenFound() {
            User user = new User("Dilip", "dilip@example.com");
            when(userRepository.findById(1L)).thenReturn(Optional.of(user));
            
            Optional<User> result = userService.findById(1L);
            
            assertTrue(result.isPresent());
            assertEquals("Dilip", result.get().getName());
        }
        
        @Test
        @DisplayName("should return empty when not found")
        void shouldReturnEmptyWhenNotFound() {
            when(userRepository.findById(999L)).thenReturn(Optional.empty());
            
            Optional<User> result = userService.findById(999L);
            
            assertTrue(result.isEmpty());
        }
    }
}
```

---

## 7. Parameterized Tests

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;

class ParameterizedTestsDemo {
    
    // @ValueSource — inline values
    @ParameterizedTest
    @ValueSource(ints = {2, 4, 6, 8, 10, 100})
    @DisplayName("Should correctly identify even numbers")
    void testIsEven(int number) {
        assertTrue(number % 2 == 0);
    }
    
    @ParameterizedTest
    @ValueSource(strings = {"", " ", "  ", "\t", "\n"})
    @DisplayName("Should identify blank strings")
    void testBlankStrings(String input) {
        assertTrue(input.isBlank());
    }
    
    @ParameterizedTest
    @NullAndEmptySource // Provides null and ""
    @ValueSource(strings = {" ", "\t", "\n"})
    void testNullEmptyAndBlank(String input) {
        assertTrue(input == null || input.isBlank());
    }
    
    // @EnumSource — enum values
    @ParameterizedTest
    @EnumSource(Month.class)
    void testAllMonths(Month month) {
        assertNotNull(month);
    }
    
    @ParameterizedTest
    @EnumSource(value = Month.class, names = {"JANUARY", "FEBRUARY", "MARCH"})
    void testFirstQuarter(Month month) {
        int monthNumber = month.getValue();
        assertTrue(monthNumber >= 1 && monthNumber <= 3);
    }
    
    // @CsvSource — CSV data inline
    @ParameterizedTest
    @CsvSource({
        "1, 1, 2",
        "2, 3, 5",
        "10, 20, 30",
        "-1, 1, 0",
        "0, 0, 0"
    })
    @DisplayName("Should add two numbers correctly")
    void testAdd(int a, int b, int expected) {
        Calculator calculator = new Calculator();
        assertEquals(expected, calculator.add(a, b));
    }
    
    @ParameterizedTest
    @CsvSource({
        "Dilip, DILIP",
        "hello, HELLO",
        "Spring Boot, SPRING BOOT"
    })
    void testToUpperCase(String input, String expected) {
        assertEquals(expected, input.toUpperCase());
    }
    
    // @CsvFileSource — CSV data from file
    @ParameterizedTest
    @CsvFileSource(resources = "/test-data.csv", numLinesToSkip = 1)
    void testFromFile(String name, int age, String email) {
        assertNotNull(name);
        assertTrue(age > 0);
        assertTrue(email.contains("@"));
    }
    
    // @MethodSource — values from a static method
    @ParameterizedTest
    @MethodSource("provideStringsForPalindrome")
    void testIsPalindrome(String input, boolean expected) {
        assertEquals(expected, isPalindrome(input));
    }
    
    static Stream<Arguments> provideStringsForPalindrome() {
        return Stream.of(
            Arguments.of("racecar", true),
            Arguments.of("radar", true),
            Arguments.of("hello", false),
            Arguments.of("level", true),
            Arguments.of("java", false),
            Arguments.of("", true),
            Arguments.of("a", true)
        );
    }
    
    // @ArgumentsSource — custom arguments provider
    @ParameterizedTest
    @ArgumentsSource(CustomArgumentsProvider.class)
    void testWithCustomProvider(String name, int age) {
        assertNotNull(name);
        assertTrue(age > 0);
    }
    
    static class CustomArgumentsProvider implements ArgumentsProvider {
        @Override
        public Stream<? extends Arguments> provideArguments(ExtensionContext context) {
            return Stream.of(
                Arguments.of("Alice", 25),
                Arguments.of("Bob", 30),
                Arguments.of("Charlie", 35)
            );
        }
    }
    
    private boolean isPalindrome(String str) {
        return str.equals(new StringBuilder(str).reverse().toString());
    }
}
```

---

## 8. Assumptions

```java
import static org.junit.jupiter.api.Assumptions.*;

class AssumptionsDemo {
    
    @Test
    void testOnlyOnDev() {
        // If assumption fails, test is SKIPPED (not failed)
        assumeTrue("dev".equals(System.getenv("ENV")),
            "Test only runs in dev environment");
        
        // This code only runs if assumption passes
        assertEquals(2, 1 + 1);
    }
    
    @Test
    void testOnlyOnWindows() {
        assumeTrue(System.getProperty("os.name").startsWith("Windows"));
        // Windows-specific test code
    }
    
    @Test
    void testWithAssumingThat() {
        String env = System.getenv("ENV");
        
        // Only run this assertion block if condition is true
        // Test itself is NOT skipped if condition is false
        assumingThat("prod".equals(env), () -> {
            assertEquals(42, computeProdValue());
        });
        
        // This always runs
        assertEquals(2, 1 + 1);
    }
}
```

---

## 9. Exception Testing

```java
class ExceptionTestingDemo {
    
    @Test
    @DisplayName("Should throw exception for invalid input")
    void testException() {
        // Basic exception test
        assertThrows(IllegalArgumentException.class, () -> {
            new User(null, "email@test.com");
        });
    }
    
    @Test
    @DisplayName("Should throw exception with specific message")
    void testExceptionMessage() {
        Exception exception = assertThrows(IllegalArgumentException.class, () -> {
            new User("", "email@test.com");
        });
        
        assertEquals("Name cannot be empty", exception.getMessage());
        assertTrue(exception.getMessage().contains("empty"));
    }
    
    @Test
    @DisplayName("Should throw specific subclass exception")
    void testExceptionType() {
        // assertThrows checks exact type or subtype
        assertThrows(RuntimeException.class, () -> {
            throw new IllegalArgumentException("test");
        }); // Passes — IllegalArgumentException IS a RuntimeException
    }
    
    @Test
    @DisplayName("Should NOT throw any exception")
    void testNoException() {
        assertDoesNotThrow(() -> {
            new User("Valid Name", "valid@email.com");
        });
    }
    
    // AssertJ exception testing (more fluent)
    @Test
    void testExceptionWithAssertJ() {
        assertThatThrownBy(() -> service.findById(-1L))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessage("ID must be positive")
            .hasMessageContaining("positive")
            .hasNoCause();
        
        assertThatExceptionOfType(UserNotFoundException.class)
            .isThrownBy(() -> service.findById(999L))
            .withMessage("User not found with id: 999")
            .withNoCause();
    }
}
```

---

## 10. Timeout Testing

```java
class TimeoutTestingDemo {
    
    @Test
    @Timeout(5) // Fails if test takes more than 5 seconds
    void testTimeout() {
        // long running operation
    }
    
    @Test
    @Timeout(value = 500, unit = TimeUnit.MILLISECONDS)
    void testTimeoutMillis() {
        // Must complete within 500ms
    }
    
    @Test
    void testAssertTimeout() {
        // Waits for execution to complete, then checks time
        assertTimeout(Duration.ofSeconds(2), () -> {
            Thread.sleep(1000); // Takes 1 second — passes
            return "result";
        });
    }
    
    @Test
    void testAssertTimeoutPreemptively() {
        // IMMEDIATELY aborts if timeout exceeded (runs in different thread)
        assertTimeoutPreemptively(Duration.ofSeconds(2), () -> {
            Thread.sleep(1000);
            return "result";
        });
    }
}
```

---

## 11. Test Ordering & Execution

```java
// Order by method name
@TestMethodOrder(MethodOrderer.MethodName.class)
class OrderedByNameTest {
    @Test void aTest() { }
    @Test void bTest() { }
    @Test void cTest() { }
}

// Order by @Order annotation
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
class OrderedByAnnotationTest {
    @Test @Order(3) void thirdTest() { }
    @Test @Order(1) void firstTest() { }
    @Test @Order(2) void secondTest() { }
}

// Random order (detect order-dependent tests)
@TestMethodOrder(MethodOrderer.Random.class)
class RandomOrderTest {
    @Test void test1() { }
    @Test void test2() { }
    @Test void test3() { }
}
```

### Repeated Tests

```java
@RepeatedTest(5) // Run 5 times
void repeatedTest() {
    assertTrue(Math.random() < 1.0);
}

@RepeatedTest(value = 5, name = "{displayName} — Repetition {currentRepetition}/{totalRepetitions}")
@DisplayName("Stress Test")
void stressTest(RepetitionInfo info) {
    System.out.println("Repetition " + info.getCurrentRepetition() + 
                       " of " + info.getTotalRepetitions());
}
```

---

## 12. Conditional Test Execution

```java
class ConditionalTests {
    
    // OS conditions
    @Test
    @EnabledOnOs(OS.WINDOWS)
    void onlyOnWindows() { }
    
    @Test
    @EnabledOnOs({OS.LINUX, OS.MAC})
    void onlyOnLinuxOrMac() { }
    
    @Test
    @DisabledOnOs(OS.WINDOWS)
    void notOnWindows() { }
    
    // JRE conditions
    @Test
    @EnabledOnJre(JRE.JAVA_21)
    void onlyOnJava21() { }
    
    @Test
    @EnabledForJreRange(min = JRE.JAVA_17, max = JRE.JAVA_21)
    void onJava17To21() { }
    
    // System property conditions
    @Test
    @EnabledIfSystemProperty(named = "os.arch", matches = "amd64")
    void onlyOnAmd64() { }
    
    // Environment variable conditions
    @Test
    @EnabledIfEnvironmentVariable(named = "ENV", matches = "ci")
    void onlyOnCI() { }
    
    // Custom conditions
    @Test
    @EnabledIf("customCondition")
    void conditionalTest() { }
    
    boolean customCondition() {
        return LocalDate.now().getDayOfWeek() != DayOfWeek.SUNDAY;
    }
    
    // Disabled with reason
    @Test
    @Disabled("Bug #1234 — will fix in next sprint")
    void disabledTest() { }
}
```

### Tags (Test Filtering)

```java
@Tag("fast")
@Tag("unit")
class FastUnitTest {
    @Test void quickTest() { }
}

@Tag("slow")
@Tag("integration")
class SlowIntegrationTest {
    @Test void dbTest() { }
}
```

```xml
<!-- Maven: Run only specific tags -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-surefire-plugin</artifactId>
    <configuration>
        <groups>fast</groups>
        <excludedGroups>slow</excludedGroups>
    </configuration>
</plugin>
```

---

## 13. Mockito — Mocking Framework

### Why Mocking?

In unit tests, you want to test a class **in isolation**. If your class depends on other classes (database, web services, etc.), you **mock** those dependencies.

### Core Mockito Concepts

```java
import org.mockito.Mock;
import org.mockito.InjectMocks;
import org.mockito.junit.jupiter.MockitoExtension;
import static org.mockito.Mockito.*;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @Mock
    private EmailService emailService;
    
    @InjectMocks // Creates UserService and injects mocks
    private UserService userService;
    
    // --- Stubbing (defining mock behavior) ---
    
    @Test
    void testFindById() {
        // ARRANGE: Define what the mock should return
        User mockUser = new User(1L, "Dilip", "dilip@example.com");
        when(userRepository.findById(1L)).thenReturn(Optional.of(mockUser));
        
        // ACT
        Optional<User> result = userService.findById(1L);
        
        // ASSERT
        assertTrue(result.isPresent());
        assertEquals("Dilip", result.get().getName());
    }
    
    @Test
    void testFindByIdNotFound() {
        when(userRepository.findById(999L)).thenReturn(Optional.empty());
        
        Optional<User> result = userService.findById(999L);
        
        assertTrue(result.isEmpty());
    }
    
    // --- Stubbing with different returns ---
    
    @Test
    void testMultipleReturns() {
        when(userRepository.count())
            .thenReturn(0L)    // First call
            .thenReturn(1L)    // Second call
            .thenReturn(5L);   // Third and subsequent calls
        
        assertEquals(0L, userRepository.count()); // First call
        assertEquals(1L, userRepository.count()); // Second call
        assertEquals(5L, userRepository.count()); // Third call
    }
    
    @Test
    void testThrowException() {
        when(userRepository.findById(-1L))
            .thenThrow(new IllegalArgumentException("Invalid ID"));
        
        assertThrows(IllegalArgumentException.class, () -> {
            userService.findById(-1L);
        });
    }
    
    // --- Argument Matchers ---
    
    @Test
    void testWithArgumentMatchers() {
        when(userRepository.findById(anyLong())).thenReturn(Optional.of(new User()));
        when(userRepository.findByEmail(anyString())).thenReturn(Optional.empty());
        when(userRepository.findByNameAndAge(eq("Dilip"), anyInt())).thenReturn(List.of());
        
        // Other matchers: any(), anyString(), anyInt(), anyList(), isNull(), notNull()
        // eq() — exact value match (needed when mixing matchers and raw values)
    }
    
    // --- Verification (did the mock get called?) ---
    
    @Test
    void testVerification() {
        User user = new User("Dilip", "dilip@example.com");
        when(userRepository.save(any())).thenReturn(user);
        
        userService.createUser(user);
        
        // Verify method was called
        verify(userRepository).save(user);
        verify(emailService).sendWelcomeEmail("dilip@example.com");
        
        // Verify call count
        verify(userRepository, times(1)).save(any());
        verify(userRepository, never()).deleteById(anyLong());
        verify(emailService, atLeastOnce()).sendWelcomeEmail(anyString());
        verify(userRepository, atMost(3)).findById(anyLong());
        
        // Verify no more interactions
        verifyNoMoreInteractions(userRepository);
        verifyNoInteractions(emailService); // No interactions at all
    }
    
    // --- Argument Captor ---
    
    @Test
    void testArgumentCaptor() {
        User user = new User("Dilip", "dilip@example.com");
        when(userRepository.save(any())).thenReturn(user);
        
        userService.createUser(user);
        
        // Capture the argument passed to save()
        ArgumentCaptor<User> captor = ArgumentCaptor.forClass(User.class);
        verify(userRepository).save(captor.capture());
        
        User savedUser = captor.getValue();
        assertEquals("Dilip", savedUser.getName());
        assertNotNull(savedUser.getCreatedAt()); // Verify service set the timestamp
    }
    
    // --- Spy (partial mock) ---
    
    @Test
    void testSpy() {
        List<String> realList = new ArrayList<>();
        List<String> spyList = spy(realList);
        
        // Real methods are called by default
        spyList.add("one");
        spyList.add("two");
        assertEquals(2, spyList.size()); // Real size
        
        // Override specific behavior
        doReturn(100).when(spyList).size();
        assertEquals(100, spyList.size()); // Mocked
        
        // Verify
        verify(spyList).add("one");
    }
    
    // --- BDD (Behavior Driven Development) style ---
    
    @Test
    void testBDDStyle() {
        // Given
        given(userRepository.findById(1L))
            .willReturn(Optional.of(new User(1L, "Dilip", "dilip@example.com")));
        
        // When
        Optional<User> result = userService.findById(1L);
        
        // Then
        then(userRepository).should().findById(1L);
        assertThat(result).isPresent();
    }
}
```

---

## 14. Spring Boot Testing

### @SpringBootTest

```java
@SpringBootTest // Loads full application context
class ApplicationIntegrationTest {
    
    @Autowired
    private UserService userService;
    
    @Test
    void contextLoads() {
        // Verifies application context loads successfully
        assertNotNull(userService);
    }
}

// With specific web environment
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
class WebIntegrationTest {
    
    @LocalServerPort
    private int port;
    
    @Autowired
    private TestRestTemplate restTemplate;
    
    @Test
    void testGetUsers() {
        ResponseEntity<String> response = restTemplate
            .getForEntity("http://localhost:" + port + "/api/users", String.class);
        
        assertEquals(HttpStatus.OK, response.getStatusCode());
    }
}
```

### @WebMvcTest (Controller Layer Only)

```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean // Spring-managed mock
    private UserService userService;
    
    @Test
    void shouldReturnAllUsers() throws Exception {
        List<UserDto> users = List.of(
            new UserDto(1L, "Dilip", "dilip@example.com"),
            new UserDto(2L, "John", "john@example.com")
        );
        when(userService.findAll()).thenReturn(users);
        
        mockMvc.perform(get("/api/users")
                .contentType(MediaType.APPLICATION_JSON))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$", hasSize(2)))
            .andExpect(jsonPath("$[0].name").value("Dilip"))
            .andExpect(jsonPath("$[1].name").value("John"));
    }
    
    @Test
    void shouldCreateUser() throws Exception {
        UserDto newUser = new UserDto(null, "Dilip", "dilip@example.com");
        UserDto savedUser = new UserDto(1L, "Dilip", "dilip@example.com");
        when(userService.create(any())).thenReturn(savedUser);
        
        mockMvc.perform(post("/api/users")
                .contentType(MediaType.APPLICATION_JSON)
                .content("""
                    {
                        "name": "Dilip",
                        "email": "dilip@example.com"
                    }
                """))
            .andExpect(status().isCreated())
            .andExpect(jsonPath("$.id").value(1))
            .andExpect(jsonPath("$.name").value("Dilip"));
    }
    
    @Test
    void shouldReturn404ForNonExistentUser() throws Exception {
        when(userService.findById(999L)).thenThrow(new UserNotFoundException(999L));
        
        mockMvc.perform(get("/api/users/999"))
            .andExpect(status().isNotFound());
    }
}
```

### @DataJpaTest (Repository Layer Only)

```java
@DataJpaTest // Configures H2, JPA, repositories only
class UserRepositoryTest {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Test
    void shouldFindByEmail() {
        // Arrange
        User user = new User("Dilip", "dilip@example.com");
        entityManager.persistAndFlush(user);
        
        // Act
        Optional<User> found = userRepository.findByEmail("dilip@example.com");
        
        // Assert
        assertTrue(found.isPresent());
        assertEquals("Dilip", found.get().getName());
    }
    
    @Test
    void shouldReturnEmptyForNonExistentEmail() {
        Optional<User> found = userRepository.findByEmail("nobody@example.com");
        assertTrue(found.isEmpty());
    }
}
```

### @MockBean vs @Mock

| Feature | `@Mock` (Mockito) | `@MockBean` (Spring) |
|---------|-------------------|---------------------|
| Context | Plain JUnit test | Spring ApplicationContext |
| Speed | Fast (no Spring) | Slower (needs Spring) |
| Replaces | Nothing | Existing bean in context |
| Use with | `@ExtendWith(MockitoExtension.class)` | `@SpringBootTest`, `@WebMvcTest` |
| Preference | Unit tests | Integration tests |

---

## 15. Test Driven Development (TDD)

### TDD Cycle: Red → Green → Refactor

```
1. RED:     Write a failing test first
2. GREEN:   Write minimum code to make it pass
3. REFACTOR: Improve the code without changing behavior
4. REPEAT
```

### TDD Example

```java
// Step 1: RED — Write a failing test
@Test
void shouldCalculateShippingCost() {
    ShippingCalculator calculator = new ShippingCalculator();
    
    // Free shipping for orders over $100
    assertEquals(0.0, calculator.calculateShipping(150.0));
    
    // $5.99 flat rate for orders $50-$100
    assertEquals(5.99, calculator.calculateShipping(75.0));
    
    // $9.99 for orders under $50
    assertEquals(9.99, calculator.calculateShipping(25.0));
}

// Step 2: GREEN — Write minimum code to pass
public class ShippingCalculator {
    public double calculateShipping(double orderTotal) {
        if (orderTotal >= 100) return 0.0;
        if (orderTotal >= 50) return 5.99;
        return 9.99;
    }
}

// Step 3: REFACTOR — Improve code
public class ShippingCalculator {
    private static final double FREE_SHIPPING_THRESHOLD = 100.0;
    private static final double STANDARD_SHIPPING_THRESHOLD = 50.0;
    private static final double STANDARD_RATE = 5.99;
    private static final double EXPRESS_RATE = 9.99;
    
    public double calculateShipping(double orderTotal) {
        if (orderTotal < 0) throw new IllegalArgumentException("Order total cannot be negative");
        if (orderTotal >= FREE_SHIPPING_THRESHOLD) return 0.0;
        if (orderTotal >= STANDARD_SHIPPING_THRESHOLD) return STANDARD_RATE;
        return EXPRESS_RATE;
    }
}
```

---

## 16. Code Coverage with JaCoCo

### Maven Configuration

```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.12</version>
    <executions>
        <execution>
            <goals><goal>prepare-agent</goal></goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals><goal>report</goal></goals>
        </execution>
    </executions>
</plugin>
```

### Coverage Metrics

| Metric | Description |
|--------|-------------|
| **Line Coverage** | % of code lines executed |
| **Branch Coverage** | % of if/else branches executed |
| **Method Coverage** | % of methods called |
| **Class Coverage** | % of classes tested |
| **Instruction Coverage** | % of bytecode instructions executed |
| **Complexity Coverage** | % of cyclomatic complexity paths covered |

**Target:** Aim for **80%+ line coverage** and **70%+ branch coverage**.

---

## 17. Best Practices

1. **One assertion per test** (or one logical assertion group) — makes failures clear
2. **Test one thing at a time** — focused, single-purpose tests
3. **Use descriptive test names** — `shouldReturnEmptyListWhenNoUsersExist()`
4. **Follow AAA pattern** — Arrange, Act, Assert
5. **Keep tests independent** — no shared mutable state, no order dependency
6. **Use `@BeforeEach` for setup, not constructors**
7. **Prefer `@Mock` over `@MockBean`** — faster tests
8. **Don't test private methods directly** — test through public API
9. **Use parameterized tests** for testing multiple inputs
10. **Use `assertAll` for grouped assertions** — see all failures at once
11. **Don't mock value objects** — mock services and repositories
12. **Test edge cases** — null, empty, boundary values, exceptions
13. **Keep tests fast** — milliseconds, not seconds
14. **Use AssertJ** — more readable than JUnit assertions
15. **Don't test framework code** — test YOUR code, not Spring/Hibernate

---

## 18. Common Mistakes & Pitfalls

```java
// ❌ MISTAKE 1: Not using @ExtendWith or @SpringBootTest
class MyTest {
    @Mock UserRepository repo; // This won't be initialized!
}
// ✅ FIX: Add @ExtendWith(MockitoExtension.class)

// ❌ MISTAKE 2: Testing implementation details
@Test
void testInternals() {
    UserService service = new UserService(repo);
    // Testing private method via reflection — FRAGILE!
}
// ✅ FIX: Test behavior through public methods

// ❌ MISTAKE 3: Assertions without messages
assertEquals(5, result); // If fails: "Expected 5, got 3" — but WHAT is 'result'?
// ✅ FIX: Add descriptive messages
assertEquals(5, result, "Active user count should be 5");

// ❌ MISTAKE 4: Forgetting useJUnitPlatform() in Gradle
// Tests silently don't run!
// ✅ FIX: tasks.named('test') { useJUnitPlatform() }

// ❌ MISTAKE 5: Tests depending on execution order
// ✅ FIX: Each test should be completely independent

// ❌ MISTAKE 6: Not cleaning up resources
// ✅ FIX: Use @AfterEach or @TempDir for temporary files

// ❌ MISTAKE 7: Excessive mocking (mocking everything)
// If you mock the class under test, you're not testing anything!
// ✅ FIX: Only mock external dependencies
```

---

## 19. Interview Questions & Answers (50+)

### Beginner Level

**Q1: What is JUnit?**
**A:** JUnit is the most widely used unit testing framework for Java. It provides annotations (@Test, @BeforeEach), assertions (assertEquals, assertTrue), and test runners for writing and executing automated tests.

**Q2: What is the difference between JUnit 4 and JUnit 5?**
**A:**
| Feature | JUnit 4 | JUnit 5 |
|---------|---------|---------|
| Annotations | `@Before`, `@After` | `@BeforeEach`, `@AfterEach` |
| Assert class | `Assert.assertEquals()` | `Assertions.assertEquals()` |
| Architecture | Monolithic | Modular (Platform + Jupiter + Vintage) |
| Extensions | `@RunWith` | `@ExtendWith` |
| Parameterized | Limited | Rich (`@ParameterizedTest`) |
| Nested tests | Not supported | `@Nested` |
| Lambda support | No | Yes |

**Q3: What is the @Test annotation?**
**A:** Marks a method as a test method. JUnit discovers and executes methods annotated with @Test. The method must be non-private and non-static. It should not return a value.

**Q4: What is the AAA pattern?**
**A:** Arrange-Act-Assert: a test structure pattern where you set up test data (Arrange), perform the action (Act), and verify the result (Assert).

**Q5: What is the difference between `assertEquals` and `assertSame`?**
**A:** `assertEquals` compares values using `.equals()`. `assertSame` compares object references using `==`. Two different objects with the same content: assertEquals passes, assertSame fails.

**Q6: What is `@BeforeEach` and `@AfterEach`?**
**A:** `@BeforeEach` runs before EVERY test method (setup). `@AfterEach` runs after EVERY test method (cleanup). Used for creating fresh test objects and cleaning up resources.

**Q7: What is `@BeforeAll` and `@AfterAll`?**
**A:** Run once before/after ALL tests in the class. Must be static (unless using `@TestInstance(PER_CLASS)`). Used for expensive setup like database connections.

**Q8: How do you skip a test?**
**A:** Use `@Disabled("reason")` annotation. The test will be skipped and reported as such.

---

### Intermediate Level

**Q9: What is Mockito and why is it used?**
**A:** Mockito is a mocking framework that creates mock objects of dependencies, allowing you to test a class in isolation. You define mock behavior with `when().thenReturn()` and verify interactions with `verify()`.

**Q10: What is the difference between `@Mock` and `@Spy`?**
**A:** `@Mock` creates a complete mock — all methods return defaults (null, 0, false). `@Spy` wraps a real object — real methods are called by default, but you can override specific methods.

**Q11: What is `@InjectMocks`?**
**A:** Creates an instance of the class under test and automatically injects `@Mock` and `@Spy` fields into it (via constructor, setter, or field injection).

**Q12: What are parameterized tests?**
**A:** Tests that run multiple times with different inputs. Sources: `@ValueSource`, `@CsvSource`, `@MethodSource`, `@EnumSource`, `@CsvFileSource`.

**Q13: What is the difference between `@MockBean` and `@Mock`?**
**A:** `@Mock` (Mockito) creates a mock outside Spring context. `@MockBean` (Spring) creates a mock and adds/replaces it in the Spring ApplicationContext. Use `@Mock` for unit tests, `@MockBean` for integration tests.

**Q14: What is `assertAll`?**
**A:** Groups multiple assertions. ALL assertions execute even if some fail. Reports all failures together instead of stopping at the first failure.

**Q15: What is `@WebMvcTest`?**
**A:** Spring Boot test slice that loads only the web layer (controllers, filters, etc.) without starting a full server. Uses MockMvc for HTTP request simulation. Dependencies must be mocked with @MockBean.

**Q16: What is `@DataJpaTest`?**
**A:** Spring Boot test slice for JPA repositories. Configures an in-memory database, scans for @Entity classes, and configures Spring Data JPA repositories. Full application context is NOT loaded.

---

### Advanced Level

**Q17: How do you test asynchronous code?**
**A:** Use `CompletableFuture.get()` with a timeout, `Awaitility` library for polling, or `assertTimeoutPreemptively()` for timeout-based assertions.

**Q18: What is the difference between `assertTimeout` and `assertTimeoutPreemptively`?**
**A:** `assertTimeout` waits for the executable to complete, then checks elapsed time. `assertTimeoutPreemptively` aborts execution immediately when timeout is exceeded (runs in a separate thread).

**Q19: How do you test Spring Security?**
**A:** Use `@WithMockUser`, `@WithAnonymousUser`, or `SecurityMockMvcRequestPostProcessors` for MockMvc. For JWT testing, create test tokens and include them in request headers.

**Q20: What are JUnit 5 Extensions?**
**A:** Extensions replace JUnit 4's `@RunWith`. They intercept the test lifecycle at various points. Examples: `MockitoExtension`, `SpringExtension`. Registered with `@ExtendWith`.

---

### Rapid-Fire (Q21–Q50)

**Q21: What is `@TempDir`?**
Injects a temporary directory that's automatically cleaned up after the test.

**Q22: How do you run tests from command line?**
Maven: `mvn test`. Gradle: `gradle test`. Specific test: `mvn test -Dtest=MyTest`.

**Q23: What is `@Tag` used for?**
Categorizes tests for selective execution. Example: `@Tag("slow")`, `@Tag("integration")`.

**Q24: What is the difference between unit test and integration test?**
Unit test: tests a single class in isolation with mocked dependencies. Integration test: tests multiple components working together.

**Q25: What does `verify()` do in Mockito?**
Checks that a method on a mock was called with specific arguments and specific number of times.

**Q26: What is `ArgumentCaptor`?**
Captures the actual arguments passed to a mock method for later assertions.

**Q27: What is `@RepeatedTest`?**
Runs the same test multiple times. Useful for testing randomized logic or detecting flaky tests.

**Q28: How do you test void methods with Mockito?**
Use `doNothing()`, `doThrow()`, or `doAnswer()`: `doThrow(Exception.class).when(mock).voidMethod()`.

**Q29: What is `@Nested`?**
Groups related tests in inner classes with shared setup. Creates a hierarchical test structure for better organization.

**Q30: What is code coverage?**
Measures the percentage of code executed during tests. JaCoCo is the standard tool. Aim for 80%+ line coverage.

**Q31: What is `MockMvc`?**
Spring test utility for testing MVC controllers without starting an HTTP server. Simulates HTTP requests and asserts responses.

**Q32: What is `TestRestTemplate`?**
Spring Boot test utility for making actual HTTP requests in `@SpringBootTest` with `RANDOM_PORT`.

**Q33: What is `@DirtiesContext`?**
Tells Spring to recreate the ApplicationContext after the test. Needed when a test modifies the context (e.g., changes bean state).

**Q34: What is `Assumptions.assumeTrue()`?**
If the assumption fails, the test is SKIPPED (not failed). Used for environment-dependent tests.

**Q35: What is BDD-style Mockito?**
Uses `given().willReturn()` and `then().should()` instead of `when().thenReturn()` and `verify()`. More readable.

**Q36: What is `@TestInstance(PER_CLASS)`?**
Shares a single test instance across all test methods. Allows non-static `@BeforeAll`/@AfterAll`.

**Q37: How do you test exceptions in JUnit 5?**
`assertThrows(ExceptionType.class, () -> { code })`. Returns the exception for further assertions.

**Q38: What is `@TestPropertySource`?**
Overrides Spring properties for a test class. Example: `@TestPropertySource(properties = "server.port=0")`.

**Q39: What is the `@Sql` annotation?**
Runs SQL scripts before/after a test. Useful for database test setup: `@Sql("/test-data.sql")`.

**Q40: What is Testcontainers?**
A Java library that runs Docker containers during tests. Use real databases, message brokers, etc. instead of mocks.

**Q41: How do you test private methods?**
You don't! Test them indirectly through public methods. If you feel the need, the class may need refactoring.

**Q42: What is `@AutoConfigureMockMvc`?**
Auto-configures MockMvc in `@SpringBootTest`. Alternative to `@WebMvcTest` when you need the full context.

**Q43: What is `lenient()` in Mockito?**
Suppresses UnnecessaryStubbingException for unused stubs. Use sparingly — unnecessary stubs often indicate test issues.

**Q44: What is `@ExtendWith(SpringExtension.class)`?**
Integrates JUnit 5 with Spring TestContext. `@SpringBootTest` already includes this.

**Q45: What is test isolation?**
Each test should be independent — not affected by other tests. Achieved by fresh setup in @BeforeEach, avoiding shared mutable state.

**Q46: What is `@Captor`?**
Annotation form of `ArgumentCaptor.forClass()`. Shorthand for declaring captors as fields.

**Q47: What is `doReturn` vs `when().thenReturn()`?**
`doReturn` is used with spies to avoid calling the real method. `when().thenReturn()` calls the real method during stubbing (problematic for spies).

**Q48: What is `@SpringBootTest(classes = ...)`?**
Specifies which configuration classes to load. Useful for loading only a subset of the application context.

**Q49: What is mutation testing?**
Testing that modifies (mutates) code and checks if tests detect the change. If tests still pass after mutation, they're weak. PIT is a popular Java mutation testing tool.

**Q50: What is `@Rollback`?**
In Spring test, `@Rollback(true)` (default for `@DataJpaTest`) rolls back database changes after each test, keeping the database clean.

---

## 📚 References & Further Reading

- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://site.mockito.org/)
- [AssertJ Documentation](https://assertj.github.io/doc/)
- [Spring Boot Testing Guide](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#features.testing)
- [Baeldung JUnit 5 Tutorials](https://www.baeldung.com/junit-5)
- [Testcontainers](https://testcontainers.com/)

---

> **Previous Topic:** [← 03 - Gradle](../03-gradle/README.md)  
> **Next Topic:** [05 - Git →](../05-git/README.md)
