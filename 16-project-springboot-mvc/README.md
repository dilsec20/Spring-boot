# 🏗️ Spring Boot MVC Project — Complete In-Depth Guide

> **"Building a full MVC application ties together everything: controllers, services, repositories, views, validation, and security. This guide walks through a complete, production-ready project."**

---

## 📑 Table of Contents

1. [MVC Architecture](#1-mvc-architecture)
2. [Project Structure](#2-project-structure)
3. [Controller Layer](#3-controller-layer)
4. [Thymeleaf View Layer](#4-thymeleaf-view-layer)
5. [Form Handling & Validation](#5-form-handling--validation)
6. [Service Layer](#6-service-layer)
7. [Repository Layer](#7-repository-layer)
8. [Entity & DTO Design](#8-entity--dto-design)
9. [File Upload in MVC](#9-file-upload-in-mvc)
10. [Internationalization (i18n)](#10-internationalization-i18n)
11. [Error Pages & Exception Handling](#11-error-pages--exception-handling)
12. [Session Management](#12-session-management)
13. [REST + MVC Hybrid](#13-rest--mvc-hybrid)
14. [Testing MVC Applications](#14-testing-mvc-applications)
15. [Deployment](#15-deployment)
16. [Complete Project Example](#16-complete-project-example)
17. [Interview Questions & Answers (50+)](#17-interview-questions--answers-50)

---

## 1. MVC Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Spring MVC Flow                           │
│                                                              │
│  Browser ──→ DispatcherServlet ──→ HandlerMapping           │
│                                        │                     │
│                                   Controller                 │
│                                   (@Controller)              │
│                                    │       │                 │
│                              Service     Model               │
│                                │       (data for view)       │
│                           Repository                         │
│                                │                             │
│                            Database                          │
│                                                              │
│  Browser ←── View (HTML) ←── ViewResolver ←── Controller    │
│              (Thymeleaf)      resolves        returns        │
│                              "viewName"      "viewName"      │
└─────────────────────────────────────────────────────────────┘
```

```
@Controller (MVC — returns views):
  @GetMapping("/users")
  public String list(Model model) {
      model.addAttribute("users", userService.findAll());
      return "users/list";  ← returns VIEW NAME (Thymeleaf template)
  }

@RestController (REST — returns JSON):
  @GetMapping("/api/users")
  public List<User> list() {
      return userService.findAll();  ← returns DATA directly
  }
```

---

## 2. Project Structure

```
src/main/
├── java/com/example/app/
│   ├── Application.java
│   ├── controller/
│   │   ├── HomeController.java
│   │   ├── UserController.java
│   │   └── ProductController.java
│   ├── service/
│   │   ├── UserService.java
│   │   └── ProductService.java
│   ├── repository/
│   │   ├── UserRepository.java
│   │   └── ProductRepository.java
│   ├── model/
│   │   ├── User.java
│   │   └── Product.java
│   ├── dto/
│   │   ├── UserForm.java
│   │   └── ProductForm.java
│   ├── config/
│   │   └── WebConfig.java
│   └── exception/
│       └── GlobalExceptionHandler.java
├── resources/
│   ├── application.properties
│   ├── messages.properties           ← i18n messages
│   ├── templates/                    ← Thymeleaf templates
│   │   ├── layout/
│   │   │   └── main.html            ← Base layout
│   │   ├── fragments/
│   │   │   ├── header.html
│   │   │   └── footer.html
│   │   ├── home.html
│   │   ├── users/
│   │   │   ├── list.html
│   │   │   ├── form.html
│   │   │   └── detail.html
│   │   └── error/
│   │       ├── 404.html
│   │       └── 500.html
│   └── static/                       ← Static resources
│       ├── css/style.css
│       ├── js/app.js
│       └── images/
```

---

## 3. Controller Layer

### MVC Controller

```java
@Controller
@RequestMapping("/users")
@RequiredArgsConstructor
public class UserController {
    
    private final UserService userService;
    
    // ═══ LIST all users ═══
    @GetMapping
    public String list(Model model,
                       @RequestParam(defaultValue = "0") int page,
                       @RequestParam(defaultValue = "10") int size) {
        
        Page<User> users = userService.findAll(PageRequest.of(page, size, Sort.by("name")));
        model.addAttribute("users", users);
        return "users/list";  // → templates/users/list.html
    }
    
    // ═══ SHOW create form ═══
    @GetMapping("/new")
    public String showCreateForm(Model model) {
        model.addAttribute("user", new UserForm());
        model.addAttribute("roles", UserRole.values());
        return "users/form";  // → templates/users/form.html
    }
    
    // ═══ HANDLE create form submission ═══
    @PostMapping
    public String create(@Valid @ModelAttribute("user") UserForm form,
                         BindingResult result,
                         RedirectAttributes redirectAttributes,
                         Model model) {
        
        if (result.hasErrors()) {
            model.addAttribute("roles", UserRole.values());
            return "users/form";  // Show form again with errors
        }
        
        userService.create(form);
        redirectAttributes.addFlashAttribute("successMessage", "User created successfully!");
        return "redirect:/users";  // PRG pattern (Post-Redirect-Get)
    }
    
    // ═══ SHOW edit form ═══
    @GetMapping("/{id}/edit")
    public String showEditForm(@PathVariable Long id, Model model) {
        User user = userService.findById(id);
        model.addAttribute("user", UserForm.fromEntity(user));
        model.addAttribute("roles", UserRole.values());
        return "users/form";
    }
    
    // ═══ HANDLE edit form submission ═══
    @PostMapping("/{id}")
    public String update(@PathVariable Long id,
                         @Valid @ModelAttribute("user") UserForm form,
                         BindingResult result,
                         RedirectAttributes redirectAttributes,
                         Model model) {
        
        if (result.hasErrors()) {
            model.addAttribute("roles", UserRole.values());
            return "users/form";
        }
        
        userService.update(id, form);
        redirectAttributes.addFlashAttribute("successMessage", "User updated!");
        return "redirect:/users";
    }
    
    // ═══ DELETE ═══
    @PostMapping("/{id}/delete")
    public String delete(@PathVariable Long id, RedirectAttributes redirectAttributes) {
        userService.delete(id);
        redirectAttributes.addFlashAttribute("successMessage", "User deleted!");
        return "redirect:/users";
    }
    
    // ═══ VIEW single user ═══
    @GetMapping("/{id}")
    public String detail(@PathVariable Long id, Model model) {
        model.addAttribute("user", userService.findById(id));
        return "users/detail";
    }
}
```

---

## 4. Thymeleaf View Layer

### Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

### Base Layout (Thymeleaf Layout Dialect)

```html
<!-- templates/layout/main.html -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      xmlns:layout="http://www.ultraq.net.nz/thymeleaf/layout">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title layout:title-pattern="$CONTENT_TITLE - My App">My App</title>
    <link rel="stylesheet" th:href="@{/css/style.css}">
</head>
<body>
    <!-- Navigation -->
    <nav th:replace="~{fragments/header :: header}"></nav>
    
    <!-- Flash messages -->
    <div th:if="${successMessage}" class="alert alert-success" th:text="${successMessage}"></div>
    <div th:if="${errorMessage}" class="alert alert-danger" th:text="${errorMessage}"></div>
    
    <!-- Page content -->
    <main layout:fragment="content">
        <!-- Child pages inject content here -->
    </main>
    
    <!-- Footer -->
    <footer th:replace="~{fragments/footer :: footer}"></footer>
    
    <script th:src="@{/js/app.js}"></script>
</body>
</html>
```

### User List Page

```html
<!-- templates/users/list.html -->
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
      layout:decorate="~{layout/main}">
<head>
    <title>Users</title>
</head>
<body>
<main layout:fragment="content">
    <h1>Users</h1>
    
    <a th:href="@{/users/new}" class="btn btn-primary">Add User</a>
    
    <!-- User table -->
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>Name</th>
                <th>Email</th>
                <th>Role</th>
                <th>Actions</th>
            </tr>
        </thead>
        <tbody>
            <tr th:each="user : ${users.content}">
                <td th:text="${user.id}"></td>
                <td>
                    <a th:href="@{/users/{id}(id=${user.id})}" th:text="${user.name}"></a>
                </td>
                <td th:text="${user.email}"></td>
                <td th:text="${user.role}"></td>
                <td>
                    <a th:href="@{/users/{id}/edit(id=${user.id})}">Edit</a>
                    <form th:action="@{/users/{id}/delete(id=${user.id})}" method="post"
                          style="display:inline"
                          onsubmit="return confirm('Are you sure?')">
                        <button type="submit" class="btn-danger">Delete</button>
                    </form>
                </td>
            </tr>
            <tr th:if="${users.content.empty}">
                <td colspan="5">No users found.</td>
            </tr>
        </tbody>
    </table>
    
    <!-- Pagination -->
    <nav th:if="${users.totalPages > 1}">
        <ul class="pagination">
            <li th:classappend="${users.first} ? 'disabled'">
                <a th:href="@{/users(page=${users.number - 1})}">Previous</a>
            </li>
            <li th:each="i : ${#numbers.sequence(0, users.totalPages - 1)}"
                th:classappend="${i == users.number} ? 'active'">
                <a th:href="@{/users(page=${i})}" th:text="${i + 1}"></a>
            </li>
            <li th:classappend="${users.last} ? 'disabled'">
                <a th:href="@{/users(page=${users.number + 1})}">Next</a>
            </li>
        </ul>
    </nav>
</main>
</body>
</html>
```

### User Form (Create/Edit)

```html
<!-- templates/users/form.html -->
<main layout:fragment="content">
    <h1 th:text="${user.id != null ? 'Edit User' : 'Create User'}"></h1>
    
    <form th:action="${user.id != null ? '/users/' + user.id : '/users'}"
          th:object="${user}" method="post">
        
        <!-- Name field -->
        <div class="form-group">
            <label for="name">Name</label>
            <input type="text" id="name" th:field="*{name}"
                   th:classappend="${#fields.hasErrors('name')} ? 'is-invalid'">
            <span class="error" th:if="${#fields.hasErrors('name')}" th:errors="*{name}"></span>
        </div>
        
        <!-- Email field -->
        <div class="form-group">
            <label for="email">Email</label>
            <input type="email" id="email" th:field="*{email}"
                   th:classappend="${#fields.hasErrors('email')} ? 'is-invalid'">
            <span class="error" th:if="${#fields.hasErrors('email')}" th:errors="*{email}"></span>
        </div>
        
        <!-- Role dropdown -->
        <div class="form-group">
            <label for="role">Role</label>
            <select id="role" th:field="*{role}">
                <option value="">Select role...</option>
                <option th:each="role : ${roles}" th:value="${role}" th:text="${role}"></option>
            </select>
        </div>
        
        <button type="submit" class="btn btn-primary">
            <span th:text="${user.id != null ? 'Update' : 'Create'}"></span>
        </button>
        <a th:href="@{/users}" class="btn btn-secondary">Cancel</a>
    </form>
</main>
```

### Thymeleaf Syntax Reference

```html
<!-- Text output -->
<span th:text="${user.name}">Default Name</span>
<span th:utext="${htmlContent}">Unescaped HTML</span>

<!-- Links -->
<a th:href="@{/users}">Users</a>
<a th:href="@{/users/{id}(id=${user.id})}">User Detail</a>
<a th:href="@{/users(page=${page}, size=${size})}">With params</a>

<!-- Conditionals -->
<div th:if="${users.size() > 0}">Has users</div>
<div th:unless="${users.empty}">Not empty</div>
<span th:switch="${user.role}">
    <span th:case="'ADMIN'">Administrator</span>
    <span th:case="'USER'">Regular User</span>
    <span th:case="*">Unknown</span>
</span>

<!-- Iteration -->
<tr th:each="user, stat : ${users}">
    <td th:text="${stat.index}">Row number (0-based)</td>
    <td th:text="${stat.count}">Row count (1-based)</td>
    <td th:text="${stat.even}">Is even row?</td>
    <td th:text="${user.name}">Name</td>
</tr>

<!-- Form binding -->
<form th:object="${user}">
    <input th:field="*{name}">       <!-- Binds to user.name -->
    <input th:field="*{email}">
</form>

<!-- Fragments -->
<div th:insert="~{fragments/header :: header}"></div>
<div th:replace="~{fragments/footer :: footer}"></div>

<!-- Date formatting -->
<span th:text="${#temporals.format(user.createdAt, 'yyyy-MM-dd HH:mm')}"></span>

<!-- Number formatting -->
<span th:text="${#numbers.formatDecimal(price, 1, 2)}"></span>

<!-- String utilities -->
<span th:text="${#strings.toUpperCase(user.name)}"></span>
<span th:if="${#strings.isEmpty(user.bio)}">No bio</span>

<!-- CSS classes -->
<tr th:classappend="${stat.even} ? 'even-row' : 'odd-row'">
<div th:class="${user.active} ? 'active' : 'inactive'">
```

---

## 5. Form Handling & Validation

```java
// Form DTO with validation
public class UserForm {
    
    private Long id;  // null for create, set for edit
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100)
    private String name;
    
    @NotBlank(message = "Email is required")
    @Email(message = "Please enter a valid email")
    private String email;
    
    @NotNull(message = "Role is required")
    private UserRole role;
    
    // For create only
    @Size(min = 8, message = "Password must be at least 8 characters")
    private String password;
    
    // Convert entity to form
    public static UserForm fromEntity(User user) {
        UserForm form = new UserForm();
        form.setId(user.getId());
        form.setName(user.getName());
        form.setEmail(user.getEmail());
        form.setRole(user.getRole());
        return form;
    }
}
```

### Post-Redirect-Get (PRG) Pattern

```
WHY PRG?
Without PRG: After form submission, if user refreshes → duplicate submission!

  POST /users → Show "User created" page
  User presses F5 → POST /users AGAIN → Duplicate user! 😱

With PRG: After form submission, redirect to GET endpoint:

  POST /users → 302 Redirect to GET /users
  User presses F5 → GET /users → No duplicate! ✅

In Spring: return "redirect:/users";
```

---

## 6. Service Layer

```java
@Service
@Transactional
@RequiredArgsConstructor
public class UserService {
    
    private final UserRepository userRepository;
    private final PasswordEncoder passwordEncoder;
    
    @Transactional(readOnly = true)
    public Page<User> findAll(Pageable pageable) {
        return userRepository.findAll(pageable);
    }
    
    @Transactional(readOnly = true)
    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found: " + id));
    }
    
    public User create(UserForm form) {
        if (userRepository.existsByEmail(form.getEmail())) {
            throw new DuplicateResourceException("Email already exists");
        }
        
        User user = new User();
        user.setName(form.getName());
        user.setEmail(form.getEmail());
        user.setRole(form.getRole());
        user.setPassword(passwordEncoder.encode(form.getPassword()));
        user.setActive(true);
        
        return userRepository.save(user);
    }
    
    public User update(Long id, UserForm form) {
        User user = findById(id);
        user.setName(form.getName());
        user.setEmail(form.getEmail());
        user.setRole(form.getRole());
        return user;  // Dirty checking saves automatically
    }
    
    public void delete(Long id) {
        userRepository.deleteById(id);
    }
}
```

---

## 7. Repository Layer

```java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
    List<User> findByActiveTrue();
    Page<User> findByNameContainingIgnoreCase(String name, Pageable pageable);
    
    @Query("SELECT u FROM User u WHERE u.role = :role AND u.active = true")
    List<User> findActiveByRole(@Param("role") UserRole role);
}
```

---

## 8. Entity & DTO Design

```java
@Entity
@Table(name = "users")
public class User extends BaseEntity {
    
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String name;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @Column(nullable = false)
    private String password;
    
    @Enumerated(EnumType.STRING)
    @Column(nullable = false)
    private UserRole role = UserRole.USER;
    
    private boolean active = true;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true, 
               fetch = FetchType.LAZY)
    private List<Order> orders = new ArrayList<>();
}

public enum UserRole {
    USER, ADMIN, MODERATOR
}
```

---

## 9. File Upload in MVC

```java
@Controller
@RequestMapping("/users")
public class UserController {
    
    @PostMapping("/{id}/avatar")
    public String uploadAvatar(@PathVariable Long id,
                                @RequestParam("avatar") MultipartFile file,
                                RedirectAttributes redirectAttributes) {
        
        if (file.isEmpty()) {
            redirectAttributes.addFlashAttribute("error", "Please select a file");
            return "redirect:/users/" + id;
        }
        
        // Validate file type
        String contentType = file.getContentType();
        if (!contentType.startsWith("image/")) {
            redirectAttributes.addFlashAttribute("error", "Only images allowed");
            return "redirect:/users/" + id;
        }
        
        String filename = id + "_" + UUID.randomUUID() + 
                          "." + FilenameUtils.getExtension(file.getOriginalFilename());
        Path path = Paths.get("uploads/avatars/" + filename);
        Files.createDirectories(path.getParent());
        Files.copy(file.getInputStream(), path, StandardCopyOption.REPLACE_EXISTING);
        
        userService.updateAvatar(id, filename);
        redirectAttributes.addFlashAttribute("success", "Avatar uploaded!");
        return "redirect:/users/" + id;
    }
}
```

```html
<!-- File upload form -->
<form th:action="@{/users/{id}/avatar(id=${user.id})}" method="post"
      enctype="multipart/form-data">
    <input type="file" name="avatar" accept="image/*">
    <button type="submit">Upload Avatar</button>
</form>

<!-- Display avatar -->
<img th:if="${user.avatar}" th:src="@{/uploads/avatars/{file}(file=${user.avatar})}"
     alt="Avatar" width="100">
```

---

## 10. Internationalization (i18n)

```properties
# messages.properties (default — English)
app.title=My Application
user.name=Name
user.email=Email
user.create=Create User
user.edit=Edit User
validation.name.required=Name is required

# messages_es.properties (Spanish)
app.title=Mi Aplicación
user.name=Nombre
user.email=Correo electrónico
user.create=Crear Usuario
```

```html
<!-- Use in Thymeleaf -->
<h1 th:text="#{app.title}">My App</h1>
<label th:text="#{user.name}">Name</label>
```

---

## 11. Error Pages & Exception Handling

```html
<!-- templates/error/404.html -->
<!DOCTYPE html>
<html>
<body>
    <h1>404 — Page Not Found</h1>
    <p>The page you're looking for doesn't exist.</p>
    <a href="/">Go Home</a>
</body>
</html>
```

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public String handleNotFound(ResourceNotFoundException ex, Model model) {
        model.addAttribute("message", ex.getMessage());
        return "error/404";
    }
    
    @ExceptionHandler(DuplicateResourceException.class)
    public String handleDuplicate(DuplicateResourceException ex, 
                                   RedirectAttributes redirectAttributes) {
        redirectAttributes.addFlashAttribute("errorMessage", ex.getMessage());
        return "redirect:/users/new";
    }
}
```

---

## 12. Session Management

```java
@Controller
@SessionAttributes("cart")  // Keep "cart" in session
public class CartController {
    
    @ModelAttribute("cart")
    public Cart initCart() {
        return new Cart();  // Created once per session
    }
    
    @PostMapping("/cart/add")
    public String addToCart(@ModelAttribute("cart") Cart cart,
                            @RequestParam Long productId) {
        Product product = productService.findById(productId);
        cart.addItem(product, 1);
        return "redirect:/cart";
    }
    
    @GetMapping("/cart")
    public String viewCart(@ModelAttribute("cart") Cart cart, Model model) {
        model.addAttribute("total", cart.getTotal());
        return "cart/view";
    }
    
    @PostMapping("/cart/clear")
    public String clearCart(SessionStatus sessionStatus) {
        sessionStatus.setComplete();  // Remove session attribute
        return "redirect:/cart";
    }
}
```

---

## 13. REST + MVC Hybrid

```java
// Same application serves both HTML pages and REST API

// MVC Controller (serves HTML)
@Controller
@RequestMapping("/users")
public class UserMvcController {
    @GetMapping
    public String list(Model model) { ... return "users/list"; }
}

// REST Controller (serves JSON)
@RestController
@RequestMapping("/api/v1/users")
public class UserApiController {
    @GetMapping
    public List<UserDTO> list() { ... return users; }
}
```

---

## 14. Testing MVC Applications

```java
@WebMvcTest(UserController.class)
class UserControllerTest {
    
    @Autowired
    private MockMvc mockMvc;
    
    @MockBean
    private UserService userService;
    
    @Test
    void shouldShowUserList() throws Exception {
        when(userService.findAll(any())).thenReturn(Page.empty());
        
        mockMvc.perform(get("/users"))
            .andExpect(status().isOk())
            .andExpect(view().name("users/list"))
            .andExpect(model().attributeExists("users"));
    }
    
    @Test
    void shouldCreateUser() throws Exception {
        mockMvc.perform(post("/users")
                .param("name", "Dilip")
                .param("email", "dilip@mail.com")
                .param("role", "USER"))
            .andExpect(status().is3xxRedirection())
            .andExpect(redirectedUrl("/users"))
            .andExpect(flash().attributeExists("successMessage"));
    }
    
    @Test
    void shouldShowValidationErrors() throws Exception {
        mockMvc.perform(post("/users")
                .param("name", "")  // Invalid — blank
                .param("email", "invalid-email"))
            .andExpect(status().isOk())
            .andExpect(view().name("users/form"))
            .andExpect(model().hasErrors());
    }
}
```

---

## 15. Deployment

```bash
# Build executable JAR
./mvnw clean package -DskipTests

# Run
java -jar target/myapp-1.0.0.jar --spring.profiles.active=prod

# With environment variables
DB_PASSWORD=secret JWT_SECRET=mysecret java -jar target/myapp.jar

# Docker
FROM eclipse-temurin:21-jre
COPY target/myapp.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

---

## 16. Complete Project Example

### application.properties

```properties
spring.application.name=user-management
server.port=8080

spring.datasource.url=jdbc:mysql://localhost:3306/userdb
spring.datasource.username=root
spring.datasource.password=password

spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true

spring.thymeleaf.cache=false
spring.servlet.multipart.max-file-size=5MB

logging.level.org.springframework.web=DEBUG
```

---

## 17. Interview Questions & Answers (50+)

### Beginner

**Q1: Difference between @Controller and @RestController?** @Controller returns view names. @RestController returns data (JSON) directly. @RestController = @Controller + @ResponseBody.

**Q2: What is Thymeleaf?** Server-side Java template engine. Generates HTML from templates with data. Natural templates (valid HTML without server).

**Q3: What is Model in Spring MVC?** Object to pass data from controller to view. `model.addAttribute("key", value)`.

**Q4: What is @ModelAttribute?** Binds form data to a Java object. Auto-maps form fields to object properties.

**Q5: What is RedirectAttributes?** Passes data to redirected URL using flash attributes (survive one redirect).

**Q6: What is PRG pattern?** Post-Redirect-Get. After POST, redirect to GET to prevent duplicate submissions on refresh.

**Q7: What is BindingResult?** Contains validation errors from @Valid. Must be IMMEDIATELY after the validated parameter.

**Q8: How to serve static files?** Place in `src/main/resources/static/`. Access via `@{/css/style.css}`.

---

### Intermediate

**Q9: What is `th:field`?** Thymeleaf attribute that binds form field to object property. Sets name, id, and value.

**Q10: What are Thymeleaf fragments?** Reusable HTML chunks. Define with `th:fragment`, include with `th:insert` or `th:replace`.

**Q11: What is ViewResolver?** Resolves view name to actual template. Thymeleaf's resolver maps "users/list" → templates/users/list.html.

**Q12: What is @SessionAttributes?** Stores model attributes in HTTP session across requests. Used for multi-step forms, shopping carts.

**Q13: What is content negotiation in MVC?** Same URL returns HTML or JSON based on Accept header or URL extension.

**Q14: What is a flash attribute?** Temporary attribute that survives one redirect. Used for success/error messages after form submission.

**Q15: What is @ControllerAdvice?** Global handler for exceptions, model attributes, and data binding across all controllers.

---

### Advanced

**Q16: How does Spring MVC handle form validation?** @Valid triggers bean validation, errors go to BindingResult. If errors exist, return form view with errors displayed via Thymeleaf.

**Q17: What is `WebMvcConfigurer`?** Interface to customize Spring MVC configuration: CORS, interceptors, view controllers, resource handlers.

**Q18: What is a HandlerInterceptor?** Intercepts requests before/after controller execution. preHandle, postHandle, afterCompletion. Like filters but Spring-specific.

**Q19: How to implement file upload?** MultipartFile parameter, configure multipart properties, save to filesystem or cloud storage.

**Q20: How to implement i18n?** MessageSource bean, messages_xx.properties files, LocaleResolver, `#{key}` in Thymeleaf.

---

### Rapid-Fire (Q21–Q50)

**Q21: `th:text` vs `th:utext`?** th:text escapes HTML. th:utext doesn't (use for trusted HTML only).

**Q22: `th:each` syntax?** `th:each="item, stat : ${list}"`. stat has index, count, even, odd, first, last.

**Q23: `th:if` vs `th:unless`?** th:if renders if true. th:unless renders if false.

**Q24: `@{...}` in Thymeleaf?** URL expression. Handles context path, parameters automatically.

**Q25: `${...}` in Thymeleaf?** Variable expression. Accesses model attributes.

**Q26: `*{...}` in Thymeleaf?** Selection expression. Used with `th:object` for form binding.

**Q27: `#{...}` in Thymeleaf?** Message expression. Accesses i18n messages.

**Q28: `~{...}` in Thymeleaf?** Fragment expression. References template fragments.

**Q29: What is `th:switch/th:case`?** Switch statement in Thymeleaf templates.

**Q30: How to format dates?** `${#temporals.format(date, 'yyyy-MM-dd')}`.

**Q31: What is `th:classappend`?** Adds CSS class conditionally without replacing existing classes.

**Q32: What is `th:attr`?** Sets arbitrary HTML attributes dynamically.

**Q33: What is `th:href` vs `href`?** th:href is processed by Thymeleaf. Regular href is static.

**Q34: What is `@WebMvcTest`?** Test slice for MVC controllers. Auto-configures MockMvc. No service/repo beans.

**Q35: What is MockMvc?** Testing utility for controllers without starting HTTP server.

**Q36: What is `model().attributeExists()`?** MockMvc assertion checking model contains an attribute.

**Q37: What is `view().name()`?** MockMvc assertion checking returned view name.

**Q38: What is `flash().attributeExists()`?** MockMvc assertion for flash attributes.

**Q39: Static resources default location?** `src/main/resources/static/`, `public/`, `resources/`.

**Q40: Thymeleaf cache in dev?** Set `spring.thymeleaf.cache=false` for hot reload.

**Q41: What is `server.servlet.context-path`?** URL prefix for entire app: `/myapp`.

**Q42: What is `@InitBinder`?** Customizes data binding per controller (date formats, custom editors).

**Q43: What is `WebDataBinder`?** Binds request parameters to Java objects with type conversion.

**Q44: What is `Converter` in Spring?** Converts between types (String → Enum, String → Date).

**Q45: What is `Formatter`?** Like Converter but for String ↔ Object (print + parse).

**Q46: What is `@RequestMapping` produces/consumes?** produces: response content type. consumes: expected request content type.

**Q47: What is `multipart/form-data`?** Form encoding for file uploads.

**Q48: What is `SessionStatus.setComplete()`?** Clears @SessionAttributes data.

**Q49: What is `th:action`?** Sets form action URL dynamically.

**Q50: MVC vs REST when?** MVC: server-rendered HTML (admin panels, simple apps). REST: API-first (SPA, mobile apps, microservices).

---

## 📚 References

- [Spring MVC Documentation](https://docs.spring.io/spring-framework/reference/web/webmvc.html)
- [Thymeleaf Documentation](https://www.thymeleaf.org/documentation.html)
- [Baeldung Spring MVC](https://www.baeldung.com/spring-mvc-tutorial)
- [Spring Boot Web Guides](https://spring.io/guides/gs/serving-web-content/)

---

> **Previous Topic:** [← 15 - Spring Data JPA](../15-spring-data-jpa/README.md)  
> **Next Topic:** [17 - Spring Security →](../17-spring-security/README.md)
