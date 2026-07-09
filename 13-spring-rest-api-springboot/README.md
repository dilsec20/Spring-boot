# 🚀 Spring REST API & Spring Boot — Complete In-Depth Guide

> **"Spring Boot makes building production-ready REST APIs effortless. This guide covers everything from basic CRUD to advanced patterns."**

---

## 📑 Table of Contents

1. [Spring Boot Fundamentals](#1-spring-boot-fundamentals)
2. [Project Setup & Structure](#2-project-setup--structure)
3. [REST Controllers](#3-rest-controllers)
4. [Request Handling](#4-request-handling)
5. [Response Handling](#5-response-handling)
6. [Validation](#6-validation)
7. [Exception Handling](#7-exception-handling)
8. [Service Layer Pattern](#8-service-layer-pattern)
9. [DTO Pattern & Mapping](#9-dto-pattern--mapping)
10. [Pagination & Sorting](#10-pagination--sorting)
11. [File Upload & Download](#11-file-upload--download)
12. [Configuration & Properties](#12-configuration--properties)
13. [Actuator & Monitoring](#13-actuator--monitoring)
14. [API Documentation (OpenAPI/Swagger)](#14-api-documentation-openapiswagger)
15. [CORS Configuration](#15-cors-configuration)
16. [Best Practices](#16-best-practices)
17. [Interview Questions & Answers (50+)](#17-interview-questions--answers-50)

---

## 1. Spring Boot Fundamentals

### What is Spring Boot?

Spring Boot is an **opinionated** framework that simplifies Spring development by providing auto-configuration, embedded servers, and production-ready features.

```
Spring Framework:          Spring Boot:
- Manual config            - Auto-configuration
- External server          - Embedded Tomcat
- XML/Java config          - application.properties
- Choose dependencies      - Starter dependencies
- Build WAR               - Build JAR (java -jar app.jar)

Spring Boot = Spring Framework + Auto-Configuration + Embedded Server + Opinionated Defaults
```

### Starters

```xml
<!-- Instead of manually adding 10+ dependencies, use ONE starter: -->

<!-- Web (Tomcat + Spring MVC + Jackson) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>

<!-- JPA (Hibernate + HikariCP + Spring Data JPA) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>

<!-- Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- Validation -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-validation</artifactId>
</dependency>

<!-- Test -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>
```

---

## 2. Project Setup & Structure

### Standard Project Structure

```
src/
├── main/
│   ├── java/com/example/myapp/
│   │   ├── MyAppApplication.java          ← Entry point
│   │   ├── controller/                    ← REST controllers
│   │   │   ├── UserController.java
│   │   │   └── OrderController.java
│   │   ├── service/                       ← Business logic
│   │   │   ├── UserService.java
│   │   │   └── impl/
│   │   │       └── UserServiceImpl.java
│   │   ├── repository/                    ← Data access
│   │   │   └── UserRepository.java
│   │   ├── model/                         ← Entity classes
│   │   │   └── User.java
│   │   ├── dto/                           ← Data Transfer Objects
│   │   │   ├── CreateUserDTO.java
│   │   │   ├── UpdateUserDTO.java
│   │   │   └── UserResponseDTO.java
│   │   ├── exception/                     ← Custom exceptions
│   │   │   ├── ResourceNotFoundException.java
│   │   │   └── GlobalExceptionHandler.java
│   │   ├── config/                        ← Configuration classes
│   │   │   ├── SecurityConfig.java
│   │   │   └── CorsConfig.java
│   │   └── mapper/                        ← Object mappers
│   │       └── UserMapper.java
│   └── resources/
│       ├── application.properties
│       ├── application-dev.properties
│       └── application-prod.properties
└── test/
    └── java/com/example/myapp/
        ├── controller/
        │   └── UserControllerTest.java
        └── service/
            └── UserServiceTest.java
```

---

## 3. REST Controllers

### Complete CRUD Controller

```java
@RestController                           // REST controller (returns JSON, not views)
@RequestMapping("/api/v1/users")          // Base URL for all endpoints
@RequiredArgsConstructor                  // Lombok: generates constructor for final fields
public class UserController {
    
    private final UserService userService;  // Injected via constructor
    
    // GET /api/v1/users — Get all users (paginated)
    @GetMapping
    public ResponseEntity<Page<UserResponseDTO>> getAllUsers(
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(defaultValue = "id") String sortBy,
            @RequestParam(defaultValue = "asc") String direction) {
        
        Sort sort = direction.equalsIgnoreCase("desc") 
            ? Sort.by(sortBy).descending() 
            : Sort.by(sortBy).ascending();
        
        Page<UserResponseDTO> users = userService.findAll(PageRequest.of(page, size, sort));
        return ResponseEntity.ok(users);
    }
    
    // GET /api/v1/users/42 — Get single user
    @GetMapping("/{id}")
    public ResponseEntity<UserResponseDTO> getUserById(@PathVariable Long id) {
        UserResponseDTO user = userService.findById(id);
        return ResponseEntity.ok(user);
    }
    
    // POST /api/v1/users — Create user
    @PostMapping
    public ResponseEntity<UserResponseDTO> createUser(
            @Valid @RequestBody CreateUserDTO dto) {
        
        UserResponseDTO created = userService.create(dto);
        
        URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(created.getId())
            .toUri();
        
        return ResponseEntity.created(location).body(created);  // 201
    }
    
    // PUT /api/v1/users/42 — Update user
    @PutMapping("/{id}")
    public ResponseEntity<UserResponseDTO> updateUser(
            @PathVariable Long id,
            @Valid @RequestBody UpdateUserDTO dto) {
        
        UserResponseDTO updated = userService.update(id, dto);
        return ResponseEntity.ok(updated);
    }
    
    // DELETE /api/v1/users/42 — Delete user
    @DeleteMapping("/{id}")
    public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
        userService.delete(id);
        return ResponseEntity.noContent().build();  // 204
    }
    
    // GET /api/v1/users/search?name=Dilip — Search
    @GetMapping("/search")
    public ResponseEntity<List<UserResponseDTO>> search(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) String email,
            @RequestParam(required = false) String role) {
        
        List<UserResponseDTO> results = userService.search(name, email, role);
        return ResponseEntity.ok(results);
    }
}
```

---

## 4. Request Handling

### All Ways to Receive Data

```java
@RestController
@RequestMapping("/api/demo")
public class DemoController {
    
    // Path Variable: /api/demo/users/42
    @GetMapping("/users/{id}")
    public String pathVariable(@PathVariable Long id) {
        return "User ID: " + id;
    }
    
    // Multiple Path Variables: /api/demo/users/42/orders/7
    @GetMapping("/users/{userId}/orders/{orderId}")
    public String multiPath(@PathVariable Long userId, @PathVariable Long orderId) {
        return "User: " + userId + ", Order: " + orderId;
    }
    
    // Query Parameters: /api/demo/search?name=Dilip&age=25
    @GetMapping("/search")
    public String queryParams(
            @RequestParam String name,
            @RequestParam(required = false, defaultValue = "0") int age) {
        return "Name: " + name + ", Age: " + age;
    }
    
    // Request Body (JSON): POST with {"name": "Dilip", "email": "d@mail.com"}
    @PostMapping("/users")
    public String requestBody(@RequestBody CreateUserDTO dto) {
        return "Created: " + dto.getName();
    }
    
    // Request Header
    @GetMapping("/headers")
    public String headers(
            @RequestHeader("Authorization") String auth,
            @RequestHeader(value = "X-Custom", required = false) String custom) {
        return "Auth: " + auth;
    }
    
    // Cookie
    @GetMapping("/cookies")
    public String cookies(@CookieValue(name = "sessionId", required = false) String sessionId) {
        return "Session: " + sessionId;
    }
    
    // Matrix Variable: /api/demo/users;role=admin;active=true
    @GetMapping("/users")
    public String matrix(
            @MatrixVariable(required = false) String role,
            @MatrixVariable(required = false) Boolean active) {
        return "Role: " + role;
    }
}
```

---

## 5. Response Handling

### ResponseEntity

```java
// ResponseEntity gives you FULL control over the HTTP response

// 200 OK with body
return ResponseEntity.ok(user);

// 200 OK with custom headers
return ResponseEntity.ok()
    .header("X-Total-Count", String.valueOf(count))
    .body(users);

// 201 Created with Location header
URI location = URI.create("/api/users/" + user.getId());
return ResponseEntity.created(location).body(user);

// 204 No Content (for DELETE)
return ResponseEntity.noContent().build();

// 400 Bad Request
return ResponseEntity.badRequest().body(errorResponse);

// 404 Not Found
return ResponseEntity.notFound().build();

// Custom status
return ResponseEntity.status(HttpStatus.CONFLICT).body(error);

// Optional pattern (elegant!)
return userService.findById(id)
    .map(ResponseEntity::ok)                        // If found → 200
    .orElse(ResponseEntity.notFound().build());     // If not → 404
```

---

## 6. Validation

### Bean Validation Annotations

```java
public class CreateUserDTO {
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be 2-100 characters")
    private String name;
    
    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    private String email;
    
    @NotNull(message = "Age is required")
    @Min(value = 18, message = "Must be at least 18")
    @Max(value = 150, message = "Invalid age")
    private Integer age;
    
    @NotBlank(message = "Password is required")
    @Size(min = 8, message = "Password must be at least 8 characters")
    @Pattern(regexp = "^(?=.*[A-Z])(?=.*[a-z])(?=.*\\d).*$",
             message = "Password must contain uppercase, lowercase, and digit")
    private String password;
    
    @NotNull
    @Past(message = "Birth date must be in the past")
    private LocalDate birthDate;
    
    // getters, setters
}

// In controller — @Valid triggers validation
@PostMapping
public ResponseEntity<UserResponseDTO> create(@Valid @RequestBody CreateUserDTO dto) {
    // If validation fails, Spring throws MethodArgumentNotValidException
    // Handle it in GlobalExceptionHandler
    return ResponseEntity.status(201).body(userService.create(dto));
}
```

### Custom Validator

```java
// Custom annotation
@Documented
@Constraint(validatedBy = UniqueEmailValidator.class)
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
public @interface UniqueEmail {
    String message() default "Email already exists";
    Class<?>[] groups() default {};
    Class<? extends Payload>[] payload() default {};
}

// Validator implementation
@Component
public class UniqueEmailValidator implements ConstraintValidator<UniqueEmail, String> {
    
    @Autowired
    private UserRepository userRepository;
    
    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        return email != null && !userRepository.existsByEmail(email);
    }
}

// Usage
public class CreateUserDTO {
    @UniqueEmail
    private String email;
}
```

---

## 7. Exception Handling

```java
// Custom exception
public class ResourceNotFoundException extends RuntimeException {
    public ResourceNotFoundException(String resource, String field, Object value) {
        super(String.format("%s not found with %s: '%s'", resource, field, value));
    }
}

// Standard error response
@Getter @Setter @AllArgsConstructor
public class ApiError {
    private int status;
    private String error;
    private String message;
    private String path;
    private LocalDateTime timestamp;
    private List<FieldError> fieldErrors;
    
    @Getter @AllArgsConstructor
    public static class FieldError {
        private String field;
        private String message;
    }
}

// Global exception handler
@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ApiError> handleNotFound(
            ResourceNotFoundException ex, HttpServletRequest request) {
        
        ApiError error = new ApiError(
            404, "Not Found", ex.getMessage(),
            request.getRequestURI(), LocalDateTime.now(), null
        );
        return ResponseEntity.status(404).body(error);
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiError> handleValidation(
            MethodArgumentNotValidException ex, HttpServletRequest request) {
        
        List<ApiError.FieldError> fieldErrors = ex.getBindingResult()
            .getFieldErrors().stream()
            .map(e -> new ApiError.FieldError(e.getField(), e.getDefaultMessage()))
            .toList();
        
        ApiError error = new ApiError(
            400, "Validation Failed", "Request body validation failed",
            request.getRequestURI(), LocalDateTime.now(), fieldErrors
        );
        return ResponseEntity.badRequest().body(error);
    }
    
    @ExceptionHandler(DataIntegrityViolationException.class)
    public ResponseEntity<ApiError> handleDuplicate(
            DataIntegrityViolationException ex, HttpServletRequest request) {
        
        ApiError error = new ApiError(
            409, "Conflict", "Data integrity violation: duplicate entry",
            request.getRequestURI(), LocalDateTime.now(), null
        );
        return ResponseEntity.status(409).body(error);
    }
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiError> handleAll(Exception ex, HttpServletRequest request) {
        log.error("Unhandled exception at {}", request.getRequestURI(), ex);
        
        ApiError error = new ApiError(
            500, "Internal Server Error", "An unexpected error occurred",
            request.getRequestURI(), LocalDateTime.now(), null
        );
        return ResponseEntity.status(500).body(error);
    }
}
```

---

## 8. Service Layer Pattern

```java
// Service interface
public interface UserService {
    Page<UserResponseDTO> findAll(Pageable pageable);
    UserResponseDTO findById(Long id);
    UserResponseDTO create(CreateUserDTO dto);
    UserResponseDTO update(Long id, UpdateUserDTO dto);
    void delete(Long id);
}

// Service implementation
@Service
@Transactional
@RequiredArgsConstructor
public class UserServiceImpl implements UserService {
    
    private final UserRepository userRepository;
    private final UserMapper userMapper;
    
    @Override
    @Transactional(readOnly = true)
    public Page<UserResponseDTO> findAll(Pageable pageable) {
        return userRepository.findAll(pageable)
            .map(userMapper::toResponseDTO);
    }
    
    @Override
    @Transactional(readOnly = true)
    public UserResponseDTO findById(Long id) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User", "id", id));
        return userMapper.toResponseDTO(user);
    }
    
    @Override
    public UserResponseDTO create(CreateUserDTO dto) {
        if (userRepository.existsByEmail(dto.getEmail())) {
            throw new DuplicateResourceException("Email already exists: " + dto.getEmail());
        }
        
        User user = userMapper.toEntity(dto);
        User saved = userRepository.save(user);
        return userMapper.toResponseDTO(saved);
    }
    
    @Override
    public UserResponseDTO update(Long id, UpdateUserDTO dto) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User", "id", id));
        
        userMapper.updateEntity(user, dto);  // Update fields
        // No save() needed — @Transactional + dirty checking handles it!
        
        return userMapper.toResponseDTO(user);
    }
    
    @Override
    public void delete(Long id) {
        if (!userRepository.existsById(id)) {
            throw new ResourceNotFoundException("User", "id", id);
        }
        userRepository.deleteById(id);
    }
}
```

---

## 9. DTO Pattern & Mapping

### Why DTOs?

```
Entity (database model):        DTO (API model):
┌─────────────────────┐        ┌────────────────────┐
│ User                │        │ UserResponseDTO    │
│ - id                │        │ - id               │
│ - name              │────→   │ - name             │
│ - email             │        │ - email            │
│ - password (SECRET!)│        │ - role             │
│ - role              │        │ - createdAt        │
│ - createdAt         │        │                    │
│ - updatedAt         │        │ (NO password!)     │
│ - deletedAt         │        └────────────────────┘
│ - internalNotes     │
└─────────────────────┘

Why? Don't expose internal fields (password!), database structure, 
or lazy-loaded collections to API consumers.
```

### Manual Mapper

```java
@Component
public class UserMapper {
    
    public UserResponseDTO toResponseDTO(User user) {
        UserResponseDTO dto = new UserResponseDTO();
        dto.setId(user.getId());
        dto.setName(user.getName());
        dto.setEmail(user.getEmail());
        dto.setRole(user.getRole().name());
        dto.setCreatedAt(user.getCreatedAt());
        return dto;
    }
    
    public User toEntity(CreateUserDTO dto) {
        User user = new User();
        user.setName(dto.getName());
        user.setEmail(dto.getEmail());
        user.setPassword(passwordEncoder.encode(dto.getPassword()));
        user.setRole(UserRole.USER);
        return user;
    }
    
    public void updateEntity(User user, UpdateUserDTO dto) {
        if (dto.getName() != null) user.setName(dto.getName());
        if (dto.getEmail() != null) user.setEmail(dto.getEmail());
    }
}
```

---

## 10. Pagination & Sorting

```java
// Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByActive(boolean active, Pageable pageable);
}

// Service
public Page<UserResponseDTO> findAll(Pageable pageable) {
    return userRepository.findAll(pageable).map(userMapper::toResponseDTO);
}

// Controller
@GetMapping
public ResponseEntity<Map<String, Object>> getAll(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(defaultValue = "id,asc") String[] sort) {
    
    Page<UserResponseDTO> result = userService.findAll(PageRequest.of(page, size));
    
    Map<String, Object> response = Map.of(
        "data", result.getContent(),
        "currentPage", result.getNumber(),
        "totalItems", result.getTotalElements(),
        "totalPages", result.getTotalPages()
    );
    
    return ResponseEntity.ok(response);
}

// Request: GET /api/v1/users?page=0&size=20&sort=name,asc
```

---

## 11. File Upload & Download

```java
@RestController
@RequestMapping("/api/v1/files")
public class FileController {
    
    @PostMapping("/upload")
    public ResponseEntity<String> upload(@RequestParam("file") MultipartFile file) {
        if (file.isEmpty()) {
            return ResponseEntity.badRequest().body("File is empty");
        }
        
        String filename = UUID.randomUUID() + "_" + file.getOriginalFilename();
        Path path = Paths.get("uploads/" + filename);
        Files.copy(file.getInputStream(), path);
        
        return ResponseEntity.ok("Uploaded: " + filename);
    }
    
    @GetMapping("/download/{filename}")
    public ResponseEntity<Resource> download(@PathVariable String filename) {
        Path path = Paths.get("uploads/" + filename);
        Resource resource = new UrlResource(path.toUri());
        
        return ResponseEntity.ok()
            .contentType(MediaType.APPLICATION_OCTET_STREAM)
            .header(HttpHeaders.CONTENT_DISPOSITION, 
                    "attachment; filename=\"" + filename + "\"")
            .body(resource);
    }
}
```

---

## 12. Configuration & Properties

```yaml
# application.yml
server:
  port: 8080
  servlet:
    context-path: /api

spring:
  application:
    name: my-app
  datasource:
    url: jdbc:mysql://localhost:3306/mydb
    username: root
    password: ${DB_PASSWORD}  # Environment variable
  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
  jackson:
    date-format: yyyy-MM-dd HH:mm:ss
    time-zone: UTC
    default-property-inclusion: non_null

app:
  jwt:
    secret: ${JWT_SECRET}
    expiration: 86400000  # 24 hours
  cors:
    allowed-origins: http://localhost:3000
  upload:
    max-size: 10MB
    path: ./uploads
```

---

## 13. Actuator & Monitoring

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

```properties
# Expose all endpoints
management.endpoints.web.exposure.include=health,info,metrics,env
management.endpoint.health.show-details=always

# Custom info
info.app.name=My Application
info.app.version=1.0.0
```

```
Available endpoints:
  /actuator/health     — Application health status
  /actuator/info       — Application info
  /actuator/metrics    — Metrics (JVM, HTTP, DB)
  /actuator/env        — Environment properties
  /actuator/loggers    — View/change log levels at runtime
  /actuator/beans      — All registered beans
```

---

## 14. API Documentation (OpenAPI/Swagger)

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

```java
@RestController
@RequestMapping("/api/v1/users")
@Tag(name = "User Management", description = "APIs for managing users")
public class UserController {
    
    @Operation(summary = "Get user by ID", description = "Returns a single user")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "User found"),
        @ApiResponse(responseCode = "404", description = "User not found")
    })
    @GetMapping("/{id}")
    public ResponseEntity<UserResponseDTO> getById(
            @Parameter(description = "User ID", example = "1") @PathVariable Long id) {
        return ResponseEntity.ok(userService.findById(id));
    }
}

// Access Swagger UI: http://localhost:8080/swagger-ui.html
// API docs JSON:     http://localhost:8080/v3/api-docs
```

---

## 15. CORS Configuration

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("http://localhost:3000", "https://myapp.com")
            .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")
            .allowedHeaders("*")
            .allowCredentials(true)
            .maxAge(3600);
    }
}
```

---

## 16. Best Practices

1. **Use DTOs** — Never expose entities in API responses
2. **Validate input** — @Valid on all @RequestBody parameters
3. **Global exception handling** — @RestControllerAdvice
4. **Pagination** — Never return all records, always paginate
5. **Proper HTTP status codes** — 201 for create, 204 for delete
6. **API versioning** — /api/v1/resources
7. **Consistent response format** — Same structure for success and errors
8. **@Transactional on services** — Not controllers
9. **@Transactional(readOnly=true)** — For read operations
10. **Use ResponseEntity** — Full control over response

---

## 17. Interview Questions & Answers (50+)

### Beginner

**Q1: What is @RestController?** `@Controller` + `@ResponseBody`. Returns JSON directly, no view resolution.

**Q2: Difference between @RequestParam and @PathVariable?** @RequestParam: query param (`?key=value`). @PathVariable: URL path (`/users/{id}`).

**Q3: What is @RequestBody?** Deserializes JSON request body to Java object using Jackson.

**Q4: What is ResponseEntity?** Represents full HTTP response with status code, headers, and body.

**Q5: How to handle validation?** Add `spring-boot-starter-validation`, use @Valid on @RequestBody, annotations like @NotBlank, @Email on fields.

**Q6: What is @Valid?** Triggers bean validation on the annotated parameter. Throws MethodArgumentNotValidException on failure.

**Q7: What is Spring Boot Actuator?** Production-ready features: health checks, metrics, environment info.

**Q8: What is application.properties?** Externalized configuration file. Configures server, database, logging, custom properties.

---

### Intermediate

**Q9: What is @RestControllerAdvice?** Global exception handler for all REST controllers. Centralizes error handling.

**Q10: What is the DTO pattern?** Data Transfer Objects decouple API layer from entity layer. Prevent exposing internal fields.

**Q11: How to implement pagination?** Use `Pageable` parameter, `PageRequest.of(page, size)`, `Page<T>` return type.

**Q12: What is @Transactional(readOnly)?** Optimization for read operations. Hibernate skips dirty checking.

**Q13: How does auto-configuration work?** Spring Boot reads `AutoConfiguration.imports`, uses @Conditional to decide what to configure based on classpath and properties.

**Q14: What is @ConfigurationProperties?** Type-safe binding of properties to Java class. Better than @Value for complex config.

**Q15: How to handle file upload?** `@RequestParam("file") MultipartFile file`. Configure `spring.servlet.multipart.max-file-size`.

---

### Advanced

**Q16: How to implement API versioning?** URL path (/api/v1), header (Accept: vnd.app.v1+json), or query param (?version=1). URL path is most common.

**Q17: What is content negotiation?** Client specifies desired response format via Accept header. Spring uses HttpMessageConverters.

**Q18: What is @ControllerAdvice vs @RestControllerAdvice?** @ControllerAdvice for MVC (views). @RestControllerAdvice for REST (JSON responses).

**Q19: How to implement rate limiting?** Use Bucket4j, Resilience4j, or Spring Cloud Gateway. Return 429 Too Many Requests.

**Q20: What is WebClient vs RestTemplate?** RestTemplate: synchronous, blocking (deprecated for new projects). WebClient: reactive, non-blocking, preferred.

---

### Rapid-Fire (Q21–Q50)

**Q21: Default embedded server?** Tomcat. Can switch to Jetty or Undertow.

**Q22: What is @SpringBootApplication?** @Configuration + @EnableAutoConfiguration + @ComponentScan.

**Q23: What is @GetMapping?** Shortcut for `@RequestMapping(method = RequestMethod.GET)`.

**Q24: What is @PostMapping?** Shortcut for `@RequestMapping(method = RequestMethod.POST)`.

**Q25: What is Jackson?** JSON serialization/deserialization library. Default in Spring Boot.

**Q26: What is @JsonIgnore?** Excludes field from JSON serialization/deserialization.

**Q27: What is @JsonProperty?** Customizes JSON field name: `@JsonProperty("user_name")`.

**Q28: What is spring.profiles.active?** Activates a specific profile (dev, prod, test).

**Q29: What is DevTools?** Auto-restart on code change, live reload, dev-time properties.

**Q30: What is CommandLineRunner?** Interface to run code at application startup.

**Q31: What is @Scheduled?** Schedules method execution with cron expressions.

**Q32: What is @Async?** Runs method asynchronously in separate thread.

**Q33: What is @Cacheable?** Caches method return value based on parameters.

**Q34: What is @ConditionalOnProperty?** Creates bean only if property matches value.

**Q35: Default port?** 8080. Change with `server.port=9090`.

**Q36: How to enable HTTPS?** Configure `server.ssl.key-store` with a keystore.

**Q37: What is Spring Boot Banner?** ASCII art printed at startup. Customize with banner.txt.

**Q38: What is @CrossOrigin?** Per-controller CORS configuration.

**Q39: What is MultipartFile?** Interface representing uploaded file.

**Q40: What is ObjectMapper?** Jackson class for JSON ↔ Java conversion.

**Q41: What is @DateTimeFormat?** Specifies date format for query parameters.

**Q42: What is logging.level.root?** Sets global log level (DEBUG, INFO, WARN, ERROR).

**Q43: What is spring.jpa.open-in-view?** Keeps Hibernate Session open during view rendering. Default true, set false.

**Q44: What is @EnableCaching?** Enables Spring's caching abstraction.

**Q45: What is ETag?** Entity tag for HTTP caching. Avoids resending unchanged responses.

**Q46: How to disable Actuator endpoints?** `management.endpoints.web.exposure.exclude=env,beans`.

**Q47: What is graceful shutdown?** `server.shutdown=graceful`. Waits for active requests to finish.

**Q48: What is @Order?** Controls bean initialization/filter execution order.

**Q49: What is @Retryable?** Automatically retries failed methods (Spring Retry).

**Q50: What is Spring Boot CLI?** Command-line tool for quickly prototyping Spring applications.

---

## 📚 References

- [Spring Boot Reference](https://docs.spring.io/spring-boot/docs/current/reference/html/)
- [Spring Boot Guides](https://spring.io/guides)
- [Baeldung Spring Boot](https://www.baeldung.com/spring-boot)
- [Spring Initializr](https://start.spring.io/)
- [Springdoc OpenAPI](https://springdoc.org/)

---

> **Previous Topic:** [← 12 - Spring Framework](../12-spring-framework/README.md)  
> **Next Topic:** [14 - Spring JDBC →](../14-spring-jdbc/README.md)
