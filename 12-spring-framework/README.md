# 🌱 Spring Framework — Complete In-Depth Guide

> **"Spring Framework is the backbone of modern Java enterprise development. It provides IoC, DI, AOP, and hundreds of integrations that make building production-grade applications effortless."**

---

## 📑 Table of Contents

1. [Introduction & History](#1-introduction--history)
2. [IoC Container & Dependency Injection](#2-ioc-container--dependency-injection)
3. [Bean Lifecycle](#3-bean-lifecycle)
4. [Bean Scopes](#4-bean-scopes)
5. [Annotations Deep Dive](#5-annotations-deep-dive)
6. [Configuration Approaches](#6-configuration-approaches)
7. [Aspect-Oriented Programming (AOP)](#7-aspect-oriented-programming-aop)
8. [Spring MVC Architecture](#8-spring-mvc-architecture)
9. [Spring Boot Auto-Configuration](#9-spring-boot-auto-configuration)
10. [Profiles & Environment](#10-profiles--environment)
11. [Event System](#11-event-system)
12. [Spring Expression Language (SpEL)](#12-spring-expression-language-spel)
13. [Exception Handling](#13-exception-handling)
14. [Testing in Spring](#14-testing-in-spring)
15. [Best Practices](#15-best-practices)
16. [Interview Questions & Answers (50+)](#16-interview-questions--answers-50)

---

## 1. Introduction & History

### What is Spring Framework?

Spring is a comprehensive Java framework that provides infrastructure for developing enterprise applications. It simplifies Java development by handling plumbing (connections, transactions, security) so you focus on business logic.

### Spring Ecosystem

```
┌─────────────────────────────────────────────────────────────┐
│                     Spring Ecosystem                         │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ Spring Boot  │  │ Spring Cloud │  │ Spring Security  │  │
│  │ (Rapid Dev)  │  │ (Microservices)│ │ (Auth & AuthZ)   │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ Spring Data  │  │ Spring Batch │  │ Spring WebFlux   │  │
│  │ (DB Access)  │  │ (Batch Jobs) │  │ (Reactive)       │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ Spring AI    │  │ Spring Kafka │  │ Spring Session   │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │            Spring Framework (Core)                    │   │
│  │   IoC Container, DI, AOP, MVC, JDBC, Transactions   │   │
│  └──────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

---

## 2. IoC Container & Dependency Injection

### What is Inversion of Control (IoC)?

```
WITHOUT IoC (Traditional — YOU create dependencies):

class UserService {
    // YOU create the dependency manually
    private UserRepository repo = new UserRepository();
    // Problem: tightly coupled! Can't swap, can't test!
}

WITH IoC (Spring creates and injects dependencies):

class UserService {
    private final UserRepository repo;  // Just declare what you need
    
    // Spring INJECTS the dependency for you!
    public UserService(UserRepository repo) {
        this.repo = repo;
    }
}

IoC = "Don't call us, we'll call you"
YOU don't create objects. SPRING creates and manages them.
```

### What is Dependency Injection (DI)?

DI is the mechanism by which IoC is achieved. Three ways to inject:

```java
// ═══════════════════════════════════════════
// Method 1: CONSTRUCTOR INJECTION (Recommended ✅)
// ═══════════════════════════════════════════
@Service
public class UserService {
    
    private final UserRepository userRepository;
    private final EmailService emailService;
    
    // Spring injects both dependencies through the constructor
    // @Autowired is optional when there's only ONE constructor
    public UserService(UserRepository userRepository, EmailService emailService) {
        this.userRepository = userRepository;
        this.emailService = emailService;
    }
    
    // Why best? 
    // ✅ Fields can be FINAL (immutable)
    // ✅ All dependencies visible in one place
    // ✅ Easy to test (just pass mocks)
    // ✅ Fails fast if dependency is missing
}

// ═══════════════════════════════════════════
// Method 2: SETTER INJECTION (Rarely used)
// ═══════════════════════════════════════════
@Service
public class UserService {
    
    private UserRepository userRepository;
    
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    // When? Optional dependencies that have a default
}

// ═══════════════════════════════════════════
// Method 3: FIELD INJECTION (Avoid ❌)
// ═══════════════════════════════════════════
@Service
public class UserService {
    
    @Autowired  // Spring injects directly into the field
    private UserRepository userRepository;
    
    // Why avoid?
    // ❌ Can't make field FINAL
    // ❌ Hard to test (need reflection)
    // ❌ Hides dependencies
    // ❌ Encourages too many dependencies
}
```

### How IoC Container Works

```
1. Application starts
2. Spring scans for @Component, @Service, @Repository, @Controller
3. Spring creates instances (beans) and stores them in the IoC container
4. When a bean needs a dependency, Spring finds and injects it
5. Beans are managed throughout the application lifecycle

┌──────────────────────────────────────────┐
│          Spring IoC Container             │
│                                           │
│  ┌─────────────────┐                    │
│  │ UserController   │──depends on──┐     │
│  └─────────────────┘               │     │
│                                    ▼     │
│  ┌─────────────────┐  ┌─────────────┐  │
│  │ EmailService    │  │ UserService  │  │
│  └─────────────────┘  └──────┬──────┘  │
│           ▲                   │         │
│           │                   ▼         │
│           │           ┌──────────────┐  │
│           └───────────│UserRepository│  │
│                       └──────────────┘  │
│                                           │
│  Spring manages creation order:           │
│  UserRepository → UserService (+ Email)   │
│  → UserController                         │
└──────────────────────────────────────────┘
```

---

## 3. Bean Lifecycle

```
┌──────────────────────────────────────────────────────────────────────┐
│                        Bean Lifecycle                                 │
│                                                                      │
│  1. Instantiation          → Spring creates the bean (constructor)   │
│  2. Populate Properties    → Dependencies are injected               │
│  3. BeanNameAware          → setBeanName() called                    │
│  4. BeanFactoryAware       → setBeanFactory() called                 │
│  5. ApplicationContextAware→ setApplicationContext() called           │
│  6. BeanPostProcessor      → postProcessBeforeInitialization()       │
│  7. @PostConstruct         → Your initialization method              │
│  8. InitializingBean       → afterPropertiesSet()                    │
│  9. Custom init-method     → Your custom init                        │
│  10. BeanPostProcessor     → postProcessAfterInitialization()        │
│                                                                      │
│  ═══════════ BEAN IS READY FOR USE ═══════════                      │
│                                                                      │
│  11. @PreDestroy           → Your cleanup method                     │
│  12. DisposableBean        → destroy()                               │
│  13. Custom destroy-method → Your custom cleanup                     │
└──────────────────────────────────────────────────────────────────────┘
```

```java
@Component
public class MyBean {
    
    private final SomeDependency dep;
    
    // Step 1: Constructor (instantiation + DI)
    public MyBean(SomeDependency dep) {
        this.dep = dep;
        System.out.println("1. Constructor called");
    }
    
    // Step 7: Initialization (after all dependencies injected)
    @PostConstruct
    public void init() {
        System.out.println("2. @PostConstruct — Bean is fully initialized!");
        // Load cache, open connections, validate config, etc.
    }
    
    // Step 11: Cleanup (before bean is destroyed)
    @PreDestroy
    public void cleanup() {
        System.out.println("3. @PreDestroy — Bean is being destroyed!");
        // Close connections, flush cache, release resources
    }
}
```

---

## 4. Bean Scopes

```java
@Component
@Scope("singleton")   // DEFAULT — One instance for entire application
public class SingletonBean { }

@Component
@Scope("prototype")   // New instance EVERY TIME it's requested
public class PrototypeBean { }

// Web scopes (Spring MVC only):
@Scope("request")     // New instance per HTTP request
@Scope("session")     // New instance per HTTP session
@Scope("application") // One per ServletContext (similar to singleton)
@Scope("websocket")   // One per WebSocket session
```

```
Singleton (default):
  Request 1 → Bean@123
  Request 2 → Bean@123   ← SAME instance!
  Request 3 → Bean@123

Prototype:
  Request 1 → Bean@123
  Request 2 → Bean@456   ← NEW instance!
  Request 3 → Bean@789

Request scope:
  HTTP Request 1 → Bean@123
  HTTP Request 2 → Bean@456  ← New per request
  Same request, different injection → Bean@123  ← Same within request
```

---

## 5. Annotations Deep Dive

### Stereotype Annotations

```java
@Component       // Generic Spring-managed component
@Service         // Business logic layer (semantic alias for @Component)
@Repository      // Data access layer (adds exception translation)
@Controller      // Web controller (returns views)
@RestController  // REST controller = @Controller + @ResponseBody
@Configuration   // Java-based configuration class
```

```
┌──────────────────────────────────────────┐
│           @Component (base)              │
│  ┌──────────┐ ┌──────────┐ ┌─────────┐ │
│  │@Service  │ │@Repository│ │@Controller│ │
│  │(Business)│ │ (Data)   │ │ (Web)    │ │
│  └──────────┘ └──────────┘ └─────────┘ │
└──────────────────────────────────────────┘

They're all @Component! The difference is:
- @Service:    Convention for business logic (no extra behavior)
- @Repository: Exception translation (DB exceptions → Spring exceptions)
- @Controller: Web request handling + view resolution
```

### Key Annotations

```java
// ═══ Dependency Injection ═══
@Autowired          // Inject dependency (use on constructor preferably)
@Qualifier("name")  // When multiple beans of same type, specify which one
@Primary            // Mark as default bean when multiple candidates exist
@Value("${prop}")   // Inject property value from config file
@Lazy               // Don't create until first access

// ═══ Configuration ═══
@Configuration      // Declares a config class with @Bean methods
@Bean               // Method return value is registered as a bean
@ComponentScan      // Scan packages for @Component classes
@PropertySource     // Load external properties file
@Import             // Import another configuration class
@Conditional        // Register bean only if condition is met

// ═══ Lifecycle ═══
@PostConstruct      // Called after bean construction + DI
@PreDestroy         // Called before bean destruction
@Scope("prototype") // Bean scope (singleton, prototype, request, etc.)

// ═══ Spring Boot ═══
@SpringBootApplication  // = @Configuration + @EnableAutoConfiguration + @ComponentScan
@EnableAutoConfiguration // Enable Spring Boot auto-configuration
@ConditionalOnClass     // Register bean if class is on classpath
@ConditionalOnProperty  // Register bean if property is set
```

### @Qualifier and @Primary Example

```java
// Two implementations of the same interface
public interface NotificationService {
    void send(String message);
}

@Service("emailNotification")
public class EmailNotificationService implements NotificationService {
    public void send(String message) { /* send email */ }
}

@Service("smsNotification")
@Primary  // This is the DEFAULT when no @Qualifier is specified
public class SmsNotificationService implements NotificationService {
    public void send(String message) { /* send SMS */ }
}

// Using @Qualifier to pick specific implementation
@Service
public class OrderService {
    
    private final NotificationService emailService;
    private final NotificationService defaultService;
    
    public OrderService(
        @Qualifier("emailNotification") NotificationService emailService,
        NotificationService defaultService  // Gets @Primary (SMS)
    ) {
        this.emailService = emailService;
        this.defaultService = defaultService;
    }
}
```

---

## 6. Configuration Approaches

### Java-Based Configuration (Modern ✅)

```java
@Configuration
public class AppConfig {
    
    @Bean
    public DataSource dataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        ds.setUsername("root");
        ds.setPassword("password");
        return ds;
    }
    
    @Bean
    public ObjectMapper objectMapper() {
        ObjectMapper mapper = new ObjectMapper();
        mapper.registerModule(new JavaTimeModule());
        mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
        return mapper;
    }
    
    @Bean
    @Profile("dev")  // Only created when profile is "dev"
    public DataSource devDataSource() {
        // H2 in-memory database for development
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.H2)
            .build();
    }
}
```

### Properties Configuration

```properties
# application.properties
app.name=My Application
app.max-retries=3
app.api.base-url=https://api.example.com
app.features.email-enabled=true

# Using in code:
@Value("${app.name}")
private String appName;

@Value("${app.max-retries:5}")  // Default value = 5
private int maxRetries;

@Value("${app.features.email-enabled}")
private boolean emailEnabled;
```

### @ConfigurationProperties (Type-safe config)

```java
@Configuration
@ConfigurationProperties(prefix = "app")
public class AppProperties {
    private String name;
    private int maxRetries;
    private Api api = new Api();
    private Features features = new Features();
    
    // Nested class for app.api.*
    public static class Api {
        private String baseUrl;
        // getters, setters
    }
    
    // Nested class for app.features.*
    public static class Features {
        private boolean emailEnabled;
        // getters, setters
    }
    
    // getters, setters
}

// Usage:
@Service
public class MyService {
    private final AppProperties config;
    
    public MyService(AppProperties config) {
        this.config = config;
        String url = config.getApi().getBaseUrl();
    }
}
```

---

## 7. Aspect-Oriented Programming (AOP)

### What is AOP?

AOP handles **cross-cutting concerns** — logic that applies across multiple classes (logging, security, transactions).

```
WITHOUT AOP (cross-cutting concerns scattered everywhere):

class UserService {
    void createUser() {
        LOG.info("Creating user...");           // Logging
        checkPermission();                      // Security
        startTransaction();                     // Transaction
        // === Business Logic ===
        stopTransaction();                      // Transaction
        LOG.info("User created");               // Logging
    }
}

class OrderService {
    void createOrder() {
        LOG.info("Creating order...");          // Same logging!
        checkPermission();                      // Same security!
        startTransaction();                     // Same transaction!
        // === Business Logic ===
        stopTransaction();
        LOG.info("Order created");
    }
}

WITH AOP (cross-cutting concerns in ONE place):

@Aspect
class LoggingAspect {
    @Around("execution(* com.example.service.*.*(..))")
    public Object log(ProceedingJoinPoint pjp) {
        LOG.info("Calling " + pjp.getSignature());
        Object result = pjp.proceed();  // Execute the actual method
        LOG.info("Completed " + pjp.getSignature());
        return result;
    }
}
// Now UserService and OrderService ONLY have business logic! Clean!
```

### AOP Terminology

| Term | Meaning | Example |
|------|---------|---------|
| **Aspect** | Module for cross-cutting concern | LoggingAspect, SecurityAspect |
| **Join Point** | A point in execution | Method call, exception thrown |
| **Advice** | Code to execute at a join point | @Before, @After, @Around |
| **Pointcut** | Expression matching join points | `execution(* com.example.service.*.*(..))` |
| **Target** | Object being advised | UserService instance |
| **Proxy** | Wrapper created by Spring | AOP proxy around UserService |

### AOP Advice Types

```java
@Aspect
@Component
public class LoggingAspect {
    
    private static final Logger log = LoggerFactory.getLogger(LoggingAspect.class);
    
    // BEFORE: Runs BEFORE the method
    @Before("execution(* com.example.service.*.*(..))")
    public void logBefore(JoinPoint jp) {
        log.info("→ Calling: {}", jp.getSignature().getName());
    }
    
    // AFTER RETURNING: Runs after method returns SUCCESSFULLY
    @AfterReturning(pointcut = "execution(* com.example.service.*.*(..))", 
                    returning = "result")
    public void logAfterReturning(JoinPoint jp, Object result) {
        log.info("← {} returned: {}", jp.getSignature().getName(), result);
    }
    
    // AFTER THROWING: Runs when method throws an exception
    @AfterThrowing(pointcut = "execution(* com.example.service.*.*(..))", 
                   throwing = "ex")
    public void logAfterThrowing(JoinPoint jp, Exception ex) {
        log.error("✗ {} threw: {}", jp.getSignature().getName(), ex.getMessage());
    }
    
    // AFTER: Runs ALWAYS (like finally)
    @After("execution(* com.example.service.*.*(..))")
    public void logAfter(JoinPoint jp) {
        log.info("◆ {} completed (success or failure)", jp.getSignature().getName());
    }
    
    // AROUND: Wraps the method (most powerful)
    @Around("execution(* com.example.service.*.*(..))")
    public Object logAround(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.currentTimeMillis();
        
        try {
            Object result = pjp.proceed();  // Execute the actual method
            long time = System.currentTimeMillis() - start;
            log.info("⏱ {} took {}ms", pjp.getSignature().getName(), time);
            return result;
        } catch (Exception e) {
            log.error("✗ {} failed after {}ms", 
                      pjp.getSignature().getName(), 
                      System.currentTimeMillis() - start);
            throw e;
        }
    }
}
```

### Common Pointcut Expressions

```java
// Match all methods in service package
@Before("execution(* com.example.service.*.*(..))")

// Match specific method
@Before("execution(* com.example.service.UserService.createUser(..))")

// Match methods with specific return type
@Before("execution(User com.example.service.*.*(..))")

// Match all public methods
@Before("execution(public * *(..))")

// Match by annotation
@Before("@annotation(com.example.annotation.Loggable)")

// Match all methods in classes with @Service
@Before("within(@org.springframework.stereotype.Service *)")

// Combine with && || !
@Before("execution(* com.example.service.*.*(..)) && !execution(* *.get*(..))")
```

### Custom Annotation + AOP (Real-World Example)

```java
// Step 1: Define custom annotation
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface ExecutionTime { }

// Step 2: Create aspect that handles this annotation
@Aspect
@Component
public class ExecutionTimeAspect {
    
    @Around("@annotation(com.example.annotation.ExecutionTime)")
    public Object measureTime(ProceedingJoinPoint pjp) throws Throwable {
        long start = System.nanoTime();
        Object result = pjp.proceed();
        long elapsed = (System.nanoTime() - start) / 1_000_000;
        
        log.info("⏱ {}.{}() took {} ms",
            pjp.getTarget().getClass().getSimpleName(),
            pjp.getSignature().getName(),
            elapsed);
        
        return result;
    }
}

// Step 3: Use it!
@Service
public class UserService {
    
    @ExecutionTime  // Just add this annotation — logging happens automatically!
    public List<User> findAll() {
        return userRepository.findAll();
    }
}
// Output: ⏱ UserService.findAll() took 42 ms
```

---

## 8. Spring MVC Architecture

```
┌──────────┐  HTTP Request  ┌──────────────────┐
│ Browser  │───────────────►│DispatcherServlet │ (Front Controller)
└──────────┘                └────────┬─────────┘
                                     │
                            ┌────────▼─────────┐
                            │ HandlerMapping   │ Find which controller
                            └────────┬─────────┘ handles this URL
                                     │
                            ┌────────▼─────────┐
                            │ Controller       │ Process request
                            │ @GetMapping      │ Call service layer
                            └────────┬─────────┘
                                     │
                            ┌────────▼─────────┐
                            │ ViewResolver     │ Find the view template
                            └────────┬─────────┘ (Thymeleaf, JSP)
                                     │
┌──────────┐  HTTP Response ┌────────▼─────────┐
│ Browser  │◄───────────────│ View (HTML)      │
└──────────┘                └──────────────────┘

For REST APIs (@RestController):
  Skip ViewResolver → Return JSON directly!
```

```java
@RestController
@RequestMapping("/api/v1/users")
public class UserController {
    
    private final UserService userService;
    
    public UserController(UserService userService) {
        this.userService = userService;
    }
    
    @GetMapping
    public ResponseEntity<List<UserDTO>> getAll(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size) {
        Page<UserDTO> users = userService.findAll(PageRequest.of(page, size));
        return ResponseEntity.ok(users.getContent());
    }
    
    @GetMapping("/{id}")
    public ResponseEntity<UserDTO> getById(@PathVariable Long id) {
        return userService.findById(id)
            .map(ResponseEntity::ok)
            .orElse(ResponseEntity.notFound().build());
    }
    
    @PostMapping
    public ResponseEntity<UserDTO> create(@Valid @RequestBody CreateUserDTO dto) {
        UserDTO user = userService.create(dto);
        URI location = URI.create("/api/v1/users/" + user.getId());
        return ResponseEntity.created(location).body(user);
    }
    
    @PutMapping("/{id}")
    public ResponseEntity<UserDTO> update(@PathVariable Long id, 
                                           @Valid @RequestBody UpdateUserDTO dto) {
        return ResponseEntity.ok(userService.update(id, dto));
    }
    
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> delete(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();
    }
}
```

---

## 9. Spring Boot Auto-Configuration

### How It Works

```
Spring Boot auto-configuration is MAGIC that "just works":

1. You add spring-boot-starter-data-jpa to pom.xml
2. Spring Boot DETECTS Hibernate on classpath
3. Auto-configures: DataSource, EntityManagerFactory, TransactionManager
4. You just write @Entity and @Repository — everything works!

How?
  @SpringBootApplication
  └── @EnableAutoConfiguration
      └── Reads: META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
          └── Lists 100+ auto-configuration classes
              └── Each has @Conditional annotations
                  └── Only activates when conditions are met

Example:
  DataSourceAutoConfiguration:
    @ConditionalOnClass(DataSource.class)     ← Is DataSource on classpath?
    @ConditionalOnProperty("spring.datasource.url")  ← Is URL configured?
    → If both true: Create DataSource bean automatically!
```

### @SpringBootApplication Breakdown

```java
@SpringBootApplication
// Is equivalent to:
@Configuration           // This class contains bean definitions
@EnableAutoConfiguration // Auto-configure based on classpath
@ComponentScan          // Scan this package and subpackages for beans
public class MyApplication {
    public static void main(String[] args) {
        SpringApplication.run(MyApplication.class, args);
    }
}
```

---

## 10. Profiles & Environment

```properties
# application.properties (common config)
app.name=My Application

# application-dev.properties (dev-specific)
spring.datasource.url=jdbc:h2:mem:testdb
spring.jpa.show-sql=true
logging.level.root=DEBUG

# application-prod.properties (production)
spring.datasource.url=jdbc:mysql://prod-server:3306/mydb
spring.jpa.show-sql=false
logging.level.root=WARN
```

```java
// Activate profile:
// java -jar app.jar --spring.profiles.active=dev
// or: SPRING_PROFILES_ACTIVE=prod

// Profile-specific beans
@Configuration
public class DataSourceConfig {
    
    @Bean
    @Profile("dev")
    public DataSource devDataSource() {
        return new EmbeddedDatabaseBuilder().setType(EmbeddedDatabaseType.H2).build();
    }
    
    @Bean
    @Profile("prod")
    public DataSource prodDataSource() {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl("jdbc:mysql://prod:3306/mydb");
        return ds;
    }
}
```

---

## 11. Event System

```java
// Spring's event system for decoupled communication between components

// Step 1: Define event
public class UserRegisteredEvent {
    private final User user;
    
    public UserRegisteredEvent(User user) {
        this.user = user;
    }
    
    public User getUser() { return user; }
}

// Step 2: Publish event
@Service
public class UserService {
    
    private final ApplicationEventPublisher eventPublisher;
    
    public User register(CreateUserDTO dto) {
        User user = userRepository.save(mapToEntity(dto));
        
        // Publish event — decoupled from email, notification logic!
        eventPublisher.publishEvent(new UserRegisteredEvent(user));
        
        return user;
    }
}

// Step 3: Listen for events
@Component
public class WelcomeEmailListener {
    
    @EventListener
    public void onUserRegistered(UserRegisteredEvent event) {
        emailService.sendWelcomeEmail(event.getUser().getEmail());
    }
}

@Component
public class AuditLogListener {
    
    @EventListener
    @Async  // Handle asynchronously (non-blocking)
    public void onUserRegistered(UserRegisteredEvent event) {
        auditLogService.log("USER_REGISTERED", event.getUser().getId());
    }
}

// UserService doesn't know about email or audit log!
// Add more listeners without modifying UserService = Open/Closed Principle ✅
```

---

## 12. Spring Expression Language (SpEL)

```java
// SpEL evaluates expressions at runtime

@Value("#{systemProperties['user.timezone']}")  // System property
private String timezone;

@Value("#{T(java.lang.Math).random() * 100}")  // Call static method
private double randomValue;

@Value("#{userService.getDefaultRole()}")  // Call bean method
private String defaultRole;

@Value("#{${app.max-retries} > 3 ? 'HIGH' : 'LOW'}")  // Conditional
private String retryLevel;

// In @Query annotations
@Query("SELECT u FROM User u WHERE u.role = ?#{principal.role}")
List<User> findByCurrentUserRole();
```

---

## 13. Exception Handling

```java
// Global exception handler for ALL controllers

@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(ResourceNotFoundException ex) {
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(),
            ex.getMessage(),
            LocalDateTime.now()
        );
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ErrorResponse> handleValidation(MethodArgumentNotValidException ex) {
        List<String> errors = ex.getBindingResult().getFieldErrors()
            .stream()
            .map(e -> e.getField() + ": " + e.getDefaultMessage())
            .toList();
        
        ErrorResponse error = new ErrorResponse(400, "Validation failed", errors);
        return ResponseEntity.badRequest().body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
        log.error("Unexpected error", ex);
        ErrorResponse error = new ErrorResponse(500, "Internal server error");
        return ResponseEntity.status(500).body(error);
    }
}
```

---

## 14. Testing in Spring

```java
// Unit test (no Spring context)
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    
    @Mock
    private UserRepository userRepository;
    
    @InjectMocks
    private UserService userService;
    
    @Test
    void shouldCreateUser() {
        CreateUserDTO dto = new CreateUserDTO("Dilip", "dilip@mail.com");
        when(userRepository.save(any())).thenReturn(new User(1L, "Dilip"));
        
        User result = userService.create(dto);
        
        assertThat(result.getName()).isEqualTo("Dilip");
        verify(userRepository).save(any());
    }
}

// Integration test (full Spring context)
@SpringBootTest
@AutoConfigureMockMvc
class UserControllerIntegrationTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @Test
    void shouldGetUser() throws Exception {
        mockMvc.perform(get("/api/v1/users/1"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("Dilip"));
    }
}
```

---

## 15. Best Practices

1. **Use constructor injection** — Never field injection
2. **Program to interfaces** — Swap implementations easily
3. **Keep controllers thin** — Business logic in @Service
4. **Use profiles** — Separate config for dev, test, prod
5. **@Transactional on service layer** — Not on controllers/repositories
6. **Return DTOs** — Never return entities from controllers
7. **Use @ConfigurationProperties** — Type-safe config over @Value
8. **Handle exceptions globally** — @RestControllerAdvice
9. **Write tests** — Unit + integration tests
10. **Don't use @Autowired on fields** — Use constructor injection

---

## 16. Interview Questions & Answers (50+)

### Beginner Level

**Q1: What is Spring Framework?**
**A:** Comprehensive Java framework providing IoC, DI, AOP, MVC, transactions, and more for building enterprise applications.

**Q2: What is IoC (Inversion of Control)?**
**A:** Design principle where the framework controls object creation and lifecycle, not the developer. Spring IoC container creates and manages beans.

**Q3: What is Dependency Injection?**
**A:** Mechanism to inject dependencies into objects. Three types: constructor (recommended), setter, field. Spring resolves and injects dependencies automatically.

**Q4: What is a Spring Bean?**
**A:** An object managed by the Spring IoC container. Created via @Component/@Service/@Repository or @Bean methods. Singleton by default.

**Q5: What is @Autowired?**
**A:** Annotation to inject dependencies. On constructor (optional with single constructor), setter, or field. Spring finds matching bean and injects it.

**Q6: What is @Component vs @Service vs @Repository?**
**A:** All are stereotypes (detected by component scan). @Component is generic. @Service is for business logic. @Repository adds database exception translation.

**Q7: What is @SpringBootApplication?**
**A:** Meta-annotation combining @Configuration, @EnableAutoConfiguration, and @ComponentScan. Entry point for Spring Boot apps.

**Q8: What are bean scopes?**
**A:** singleton (one instance, default), prototype (new each time), request, session, application, websocket.

---

### Intermediate Level

**Q9: What is AOP?**
**A:** Aspect-Oriented Programming — separates cross-cutting concerns (logging, security, transactions) from business logic using aspects, pointcuts, and advice.

**Q10: Constructor vs Field injection?**
**A:** Constructor: fields can be final, easy testing, fails fast, all dependencies visible. Field: hidden dependencies, can't be final, hard to test. Always use constructor.

**Q11: What is @Transactional?**
**A:** Declarative transaction management. Spring creates proxy, begins transaction before method, commits on success, rolls back on RuntimeException.

**Q12: What is @Qualifier?**
**A:** Resolves ambiguity when multiple beans of same type exist. Specifies which bean to inject by name.

**Q13: What is @Primary?**
**A:** Marks a bean as the default choice when multiple candidates exist. Used with @Qualifier for explicit selection.

**Q14: What is Spring Boot auto-configuration?**
**A:** Automatically configures beans based on classpath, properties, and existing beans. Uses @Conditional annotations to decide what to configure.

**Q15: What is a Spring Profile?**
**A:** Named logical group of bean definitions and properties. Activated with `spring.profiles.active`. Used for env-specific config (dev, prod).

---

### Advanced Level

**Q16: What is BeanPostProcessor?**
**A:** Interface to modify beans before/after initialization. Used by Spring internally for @Autowired, AOP proxies, etc.

**Q17: How does @Transactional work internally?**
**A:** Spring creates a proxy (JDK dynamic proxy or CGLIB). Proxy intercepts method call, begins transaction, calls actual method, commits or rolls back.

**Q18: What is the proxy pattern in Spring?**
**A:** Spring creates proxy objects for AOP, @Transactional, @Async. JDK Dynamic Proxy for interfaces, CGLIB for concrete classes.

**Q19: What is circular dependency and how to resolve?**
**A:** A depends on B, B depends on A. Solutions: redesign (best), @Lazy on one dependency, setter injection, @PostConstruct initialization.

**Q20: What is ApplicationContext vs BeanFactory?**
**A:** BeanFactory: basic IoC container (lazy loading). ApplicationContext: extends BeanFactory with i18n, events, AOP, eager loading. Always use ApplicationContext.

---

### Rapid-Fire (Q21–Q50)

**Q21: @Bean vs @Component?** @Bean: on methods in @Configuration. @Component: on classes for component scanning.

**Q22: What is @Lazy?** Delays bean creation until first access (not at startup).

**Q23: What is @PostConstruct?** Called after bean construction and dependency injection. Used for initialization.

**Q24: What is @PreDestroy?** Called before bean is removed from container. Used for cleanup.

**Q25: What is @Value?** Injects property values: `@Value("${server.port}")`.

**Q26: What is @ConfigurationProperties?** Type-safe binding of properties to a POJO class.

**Q27: What is @Conditional?** Register bean only if custom condition is met.

**Q28: What is @ConditionalOnProperty?** Register bean only if specific property exists/matches.

**Q29: What is DispatcherServlet?** Front controller in Spring MVC that routes all requests to appropriate handlers.

**Q30: What is HandlerMapping?** Maps URLs to controller methods. Uses @RequestMapping annotations.

**Q31: What is ViewResolver?** Resolves view names to actual view implementations (Thymeleaf, JSP).

**Q32: What is @ExceptionHandler?** Handles specific exceptions in controllers or globally with @ControllerAdvice.

**Q33: What is @RestControllerAdvice?** Global exception handler + response body for REST controllers.

**Q34: What is @RequestBody?** Deserializes HTTP request body (JSON) to Java object.

**Q35: What is @ResponseBody?** Serializes Java object to HTTP response body (JSON).

**Q36: What is ResponseEntity?** Represents full HTTP response: status code, headers, body.

**Q37: What is @PathVariable?** Extracts value from URL path: `/users/{id}`.

**Q38: What is @RequestParam?** Extracts query parameter: `?name=value`.

**Q39: What is @ModelAttribute?** Binds form data to object. Used in MVC controllers.

**Q40: What is @SessionAttributes?** Stores model attributes in HTTP session.

**Q41: What is ApplicationEventPublisher?** Publishes events for decoupled communication between beans.

**Q42: What is @EventListener?** Listens for application events. Methods annotated with this are called when matching event is published.

**Q43: What is @Async?** Executes method asynchronously in a separate thread. Requires @EnableAsync.

**Q44: What is @Scheduled?** Schedules method execution: fixed rate, fixed delay, or cron expression. Requires @EnableScheduling.

**Q45: What is @Cacheable?** Caches method return value. Next call with same params returns cached result.

**Q46: What is CGLIB proxy?** Class-based proxy that subclasses the target class. Used when target doesn't implement an interface.

**Q47: What is JDK Dynamic Proxy?** Interface-based proxy. Used when target implements an interface.

**Q48: What is Spring Boot Actuator?** Production-ready features: health checks, metrics, monitoring endpoints.

**Q49: What is embedded server?** Tomcat/Jetty/Undertow bundled inside the app JAR. No external server needed.

**Q50: What is Spring Boot DevTools?** Auto-restart on code changes, live reload, relaxed security for dev.

---

## 📚 References

- [Spring Framework Documentation](https://docs.spring.io/spring-framework/reference/)
- [Spring Boot Documentation](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Baeldung Spring Tutorials](https://www.baeldung.com/spring-tutorial)
- [Spring Guides](https://spring.io/guides)
- [Spring Boot Starters](https://docs.spring.io/spring-boot/docs/current/reference/html/using.html#using.build-systems.starters)

---

> **Previous Topic:** [← 11 - Hibernate](../11-hibernate/README.md)  
> **Next Topic:** [13 - Spring REST API & Spring Boot →](../13-spring-rest-api-springboot/README.md)
