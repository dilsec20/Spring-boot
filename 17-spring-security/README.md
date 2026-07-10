# 🔒 Spring Security — Complete In-Depth Guide

> **"Spring Security is the de facto standard for securing Java applications. It handles authentication, authorization, CSRF, CORS, and session management out of the box."**

---

## 📑 Table of Contents

1. [Introduction & Architecture](#1-introduction--architecture)
2. [Filter Chain](#2-filter-chain)
3. [Authentication](#3-authentication)
4. [Authorization](#4-authorization)
5. [SecurityFilterChain Configuration](#5-securityfilterchain-configuration)
6. [UserDetailsService](#6-userdetailsservice)
7. [Password Encoding](#7-password-encoding)
8. [Method-Level Security](#8-method-level-security)
9. [CSRF Protection](#9-csrf-protection)
10. [CORS Configuration](#10-cors-configuration)
11. [Session Management](#11-session-management)
12. [Remember Me](#12-remember-me)
13. [Form Login & Logout](#13-form-login--logout)
14. [HTTP Basic Auth](#14-http-basic-auth)
15. [Exception Handling](#15-exception-handling)
16. [Security for REST APIs](#16-security-for-rest-apis)
17. [Common Vulnerabilities & Mitigation](#17-common-vulnerabilities--mitigation)
18. [Best Practices](#18-best-practices)
19. [Interview Questions & Answers (50+)](#19-interview-questions--answers-50)

---

## 1. Introduction & Architecture

### Spring Security Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    HTTP Request                               │
│                        │                                      │
│    ┌───────────────────▼────────────────────────────────┐    │
│    │              Security Filter Chain                  │    │
│    │                                                     │    │
│    │  ┌─────────────────────────────────────────────┐   │    │
│    │  │ 1. SecurityContextPersistenceFilter         │   │    │
│    │  │    Load SecurityContext from session         │   │    │
│    │  ├─────────────────────────────────────────────┤   │    │
│    │  │ 2. CsrfFilter                              │   │    │
│    │  │    Validate CSRF token                      │   │    │
│    │  ├─────────────────────────────────────────────┤   │    │
│    │  │ 3. UsernamePasswordAuthenticationFilter     │   │    │
│    │  │    Process login form (POST /login)         │   │    │
│    │  ├─────────────────────────────────────────────┤   │    │
│    │  │ 4. BasicAuthenticationFilter               │   │    │
│    │  │    Process Basic Auth header                │   │    │
│    │  ├─────────────────────────────────────────────┤   │    │
│    │  │ 5. BearerTokenAuthenticationFilter         │   │    │
│    │  │    Process JWT Bearer token                 │   │    │
│    │  ├─────────────────────────────────────────────┤   │    │
│    │  │ 6. AuthorizationFilter                     │   │    │
│    │  │    Check if user has required role/permission│   │    │
│    │  ├─────────────────────────────────────────────┤   │    │
│    │  │ 7. ExceptionTranslationFilter              │   │    │
│    │  │    Handle auth exceptions (401, 403)        │   │    │
│    │  └─────────────────────────────────────────────┘   │    │
│    └────────────────────────────────────────────────────┘    │
│                        │                                      │
│                 ┌──────▼──────┐                               │
│                 │  Controller │                               │
│                 └─────────────┘                               │
└─────────────────────────────────────────────────────────────┘
```

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Authentication** | WHO are you? (verify identity) |
| **Authorization** | WHAT can you do? (verify permissions) |
| **Principal** | Currently logged-in user |
| **GrantedAuthority** | Permission/role assigned to user |
| **SecurityContext** | Holds Authentication object for current thread |
| **Filter Chain** | Series of filters processing each request |

---

## 2. Filter Chain

### How the Filter Chain Works

```
HTTP Request → Filter 1 → Filter 2 → ... → Filter N → Servlet (Controller)
HTTP Response ← Filter 1 ← Filter 2 ← ... ← Filter N ← Servlet

Each filter can:
  1. Pass the request to the next filter (chain.doFilter)
  2. Block the request (return 401/403)
  3. Modify the request or response

Key filters in order:
  SecurityContextHolderFilter     → Sets up SecurityContext
  CsrfFilter                     → Validates CSRF token
  LogoutFilter                   → Handles /logout
  UsernamePasswordAuthFilter     → Handles form login
  BasicAuthenticationFilter      → Handles Basic auth
  BearerTokenAuthFilter          → Handles JWT
  ExceptionTranslationFilter     → Catches auth exceptions
  AuthorizationFilter            → Checks permissions
```

---

## 3. Authentication

### Authentication Flow

```
1. User submits credentials (username + password)
2. AuthenticationFilter creates UsernamePasswordAuthenticationToken
3. AuthenticationManager delegates to AuthenticationProvider
4. AuthenticationProvider uses UserDetailsService to load user
5. UserDetailsService fetches user from database
6. AuthenticationProvider compares passwords (BCrypt)
7. If match → Authentication object stored in SecurityContext
8. If no match → AuthenticationException thrown

┌──────────┐     ┌───────────────────┐     ┌──────────────────────┐
│  Filter   │────►│AuthenticationMgr  │────►│AuthenticationProvider│
│           │     │                   │     │                      │
│ Creates   │     │ Delegates to      │     │ 1. Load user via     │
│ AuthToken │     │ provider(s)       │     │    UserDetailsService│
└──────────┘     └───────────────────┘     │ 2. Compare passwords │
                                            │ 3. Return Auth or    │
                                            │    throw exception   │
                                            └──────────────────────┘
```

### Authentication Object

```java
// After successful authentication, you can access the user:
Authentication auth = SecurityContextHolder.getContext().getAuthentication();

String username = auth.getName();                           // Username
Object principal = auth.getPrincipal();                     // UserDetails object
Collection<? extends GrantedAuthority> roles = auth.getAuthorities(); // Roles
boolean isAuthenticated = auth.isAuthenticated();

// In a controller:
@GetMapping("/me")
public String currentUser(@AuthenticationPrincipal UserDetails user) {
    return "Logged in as: " + user.getUsername();
}
```

---

## 4. Authorization

### URL-Based Authorization

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.authorizeHttpRequests(auth -> auth
            // Public endpoints (no auth required)
            .requestMatchers("/", "/home", "/register", "/css/**", "/js/**").permitAll()
            .requestMatchers("/api/public/**").permitAll()
            
            // Role-based access
            .requestMatchers("/admin/**").hasRole("ADMIN")
            .requestMatchers("/api/admin/**").hasRole("ADMIN")
            
            // Authority-based access
            .requestMatchers(HttpMethod.DELETE, "/api/users/**").hasAuthority("DELETE_USER")
            .requestMatchers(HttpMethod.POST, "/api/users/**").hasAnyRole("ADMIN", "MODERATOR")
            
            // Authenticated (any logged-in user)
            .requestMatchers("/dashboard/**").authenticated()
            
            // Everything else requires authentication
            .anyRequest().authenticated()
        );
        
        return http.build();
    }
}
```

### Role vs Authority

```
ROLE:      hasRole("ADMIN")      → Checks for "ROLE_ADMIN" authority
AUTHORITY: hasAuthority("ADMIN") → Checks for exactly "ADMIN" authority

Convention:
  Role = coarse-grained (ADMIN, USER, MODERATOR)
  Authority = fine-grained (READ_USER, WRITE_USER, DELETE_USER)

In database: Store as "ROLE_ADMIN" for roles.
  hasRole("ADMIN")           checks for "ROLE_ADMIN"
  hasAuthority("ROLE_ADMIN") checks for "ROLE_ADMIN"  (same thing!)
  hasAuthority("DELETE_USER") checks for "DELETE_USER" (no ROLE_ prefix)
```

---

## 5. SecurityFilterChain Configuration

### Complete Configuration (Spring Boot 3.x / Spring Security 6.x)

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity  // Enable @PreAuthorize, @Secured
public class SecurityConfig {
    
    private final CustomUserDetailsService userDetailsService;
    private final JwtAuthenticationFilter jwtFilter;
    
    public SecurityConfig(CustomUserDetailsService userDetailsService,
                          JwtAuthenticationFilter jwtFilter) {
        this.userDetailsService = userDetailsService;
        this.jwtFilter = jwtFilter;
    }
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            // CSRF
            .csrf(csrf -> csrf
                .ignoringRequestMatchers("/api/**")  // Disable for REST API
            )
            
            // Session management
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)  // For JWT
            )
            
            // Authorization rules
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            
            // Authentication provider
            .authenticationProvider(authenticationProvider())
            
            // Add JWT filter before UsernamePasswordAuthenticationFilter
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
            
            // Exception handling
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint((req, resp, authEx) -> {
                    resp.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
                    resp.setContentType("application/json");
                    resp.getWriter().write("{\"error\": \"Unauthorized\"}");
                })
                .accessDeniedHandler((req, resp, accessEx) -> {
                    resp.setStatus(HttpServletResponse.SC_FORBIDDEN);
                    resp.setContentType("application/json");
                    resp.getWriter().write("{\"error\": \"Access denied\"}");
                })
            );
        
        return http.build();
    }
    
    @Bean
    public AuthenticationProvider authenticationProvider() {
        DaoAuthenticationProvider provider = new DaoAuthenticationProvider();
        provider.setUserDetailsService(userDetailsService);
        provider.setPasswordEncoder(passwordEncoder());
        return provider;
    }
    
    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }
    
    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration config)
            throws Exception {
        return config.getAuthenticationManager();
    }
}
```

---

## 6. UserDetailsService

```java
// Custom UserDetailsService — loads user from YOUR database

@Service
public class CustomUserDetailsService implements UserDetailsService {
    
    private final UserRepository userRepository;
    
    public CustomUserDetailsService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        // Load from YOUR database
        User user = userRepository.findByEmail(username)
            .orElseThrow(() -> new UsernameNotFoundException("User not found: " + username));
        
        // Convert YOUR User entity to Spring Security's UserDetails
        return org.springframework.security.core.userdetails.User.builder()
            .username(user.getEmail())
            .password(user.getPassword())  // Already BCrypt encoded!
            .roles(user.getRole().name())  // ADMIN → ROLE_ADMIN
            .disabled(!user.isActive())
            .build();
    }
}
```

### Custom UserDetails (for more user info)

```java
public class CustomUserDetails implements UserDetails {
    
    private final User user;  // Your entity
    
    public CustomUserDetails(User user) {
        this.user = user;
    }
    
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority("ROLE_" + user.getRole().name()));
    }
    
    @Override
    public String getPassword() { return user.getPassword(); }
    
    @Override
    public String getUsername() { return user.getEmail(); }
    
    @Override
    public boolean isAccountNonExpired() { return true; }
    
    @Override
    public boolean isAccountNonLocked() { return true; }
    
    @Override
    public boolean isCredentialsNonExpired() { return true; }
    
    @Override
    public boolean isEnabled() { return user.isActive(); }
    
    // Custom method to access full user info
    public User getUser() { return user; }
    public Long getUserId() { return user.getId(); }
}

// Access in controller:
@GetMapping("/me")
public User getCurrentUser(@AuthenticationPrincipal CustomUserDetails userDetails) {
    return userDetails.getUser();  // Full user entity!
}
```

---

## 7. Password Encoding

```java
@Bean
public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();  // Industry standard ✅
}

// Usage:
String rawPassword = "myPassword123";
String encoded = passwordEncoder.encode(rawPassword);
// Result: $2a$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy

boolean matches = passwordEncoder.matches(rawPassword, encoded);  // true

// BCrypt is DIFFERENT each time (random salt):
// encode("hello") → $2a$10$abc...
// encode("hello") → $2a$10$xyz...  (different! but both match "hello")
```

### Available Encoders

| Encoder | Strength | Use Case |
|---------|----------|----------|
| **BCryptPasswordEncoder** | Strong ✅ | Production (default choice) |
| Argon2PasswordEncoder | Strongest | High security requirements |
| SCryptPasswordEncoder | Strong | Memory-hard hashing |
| Pbkdf2PasswordEncoder | Good | FIPS compliance |
| NoOpPasswordEncoder | None ❌ | Testing only! |

---

## 8. Method-Level Security

```java
@Configuration
@EnableMethodSecurity  // Enables @PreAuthorize, @PostAuthorize, @Secured
public class SecurityConfig { }

@Service
public class UserService {
    
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }
    
    @PreAuthorize("hasRole('ADMIN') or #id == principal.userId")
    public User getUser(Long id) {
        // ADMIN can view anyone. Users can only view themselves.
        return userRepository.findById(id).orElseThrow();
    }
    
    @PreAuthorize("hasAnyRole('ADMIN', 'MODERATOR')")
    public void banUser(Long id) { /* ... */ }
    
    @PreAuthorize("#user.email == principal.username")
    public void updateProfile(User user) { /* Only own profile */ }
    
    @PostAuthorize("returnObject.email == principal.username or hasRole('ADMIN')")
    public User findUser(Long id) {
        // Checks AFTER method execution
        return userRepository.findById(id).orElseThrow();
    }
    
    @PreAuthorize("hasAuthority('DELETE_USER')")
    public void permanentlyDeleteUser(Long id) { /* Fine-grained authority */ }
}
```

---

## 9. CSRF Protection

```
CSRF (Cross-Site Request Forgery):
  Attacker tricks a logged-in user's browser into making unwanted requests.

Example attack:
  1. User is logged into bank.com
  2. User visits evil.com
  3. evil.com has: <img src="http://bank.com/transfer?to=attacker&amount=10000">
  4. Browser sends the request WITH the user's cookies!
  5. Bank processes the transfer because cookies are valid 😱

Protection:
  Include a unique CSRF TOKEN in every form.
  Server validates the token with each POST/PUT/DELETE request.
  Attacker can't guess the token!
```

```java
// CSRF is ENABLED by default in Spring Security

// For Thymeleaf forms — token is AUTO-INCLUDED:
<form th:action="@{/transfer}" method="post">
    <!-- Thymeleaf automatically adds: -->
    <!-- <input type="hidden" name="_csrf" value="abc123..."> -->
    <input type="text" name="amount">
    <button type="submit">Transfer</button>
</form>

// For REST APIs — typically DISABLE CSRF:
http.csrf(csrf -> csrf.disable());
// Why? REST APIs use tokens (JWT), not cookies. No CSRF risk.

// Or disable only for API endpoints:
http.csrf(csrf -> csrf.ignoringRequestMatchers("/api/**"));
```

---

## 10. CORS Configuration

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .cors(cors -> cors.configurationSource(corsConfigurationSource()))
        // ... other config
    ;
    return http.build();
}

@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOrigins(List.of("http://localhost:3000"));
    config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));
    config.setAllowedHeaders(List.of("*"));
    config.setAllowCredentials(true);
    config.setMaxAge(3600L);
    
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/api/**", config);
    return source;
}
```

---

## 11. Session Management

```java
http.sessionManagement(session -> session
    // For traditional web apps (form login):
    .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
    .maximumSessions(1)                     // One session per user
    .maxSessionsPreventsLogin(true)         // Block new login (vs. expire old)
    .expiredUrl("/login?expired")
    
    // For REST APIs (JWT):
    .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
    // No session created. Each request must contain JWT.
);

// Session fixation protection (default: migrateSession)
http.sessionManagement(session -> session
    .sessionFixation().migrateSession()  // Create new session ID on login
);
```

---

## 12. Remember Me

```java
http.rememberMe(remember -> remember
    .key("uniqueAndSecretKey")             // Encryption key
    .tokenValiditySeconds(7 * 24 * 3600)   // 7 days
    .userDetailsService(userDetailsService)
    .rememberMeParameter("remember-me")    // Form checkbox name
);
```

---

## 13. Form Login & Logout

```java
http
    .formLogin(form -> form
        .loginPage("/login")                   // Custom login page
        .loginProcessingUrl("/perform-login")  // Form action URL
        .defaultSuccessUrl("/dashboard", true) // After successful login
        .failureUrl("/login?error=true")       // After failed login
        .usernameParameter("email")            // Form field name
        .passwordParameter("password")         // Form field name
        .permitAll()
    )
    .logout(logout -> logout
        .logoutUrl("/perform-logout")          // Logout URL
        .logoutSuccessUrl("/login?logout=true") // After logout
        .deleteCookies("JSESSIONID")           // Clear cookies
        .invalidateHttpSession(true)           // Invalidate session
        .clearAuthentication(true)
        .permitAll()
    );
```

---

## 14. HTTP Basic Auth

```java
// Simple but sends credentials with EVERY request (Base64 encoded)
http.httpBasic(Customizer.withDefaults());

// Request: Authorization: Basic dXNlcjpwYXNz (base64 of "user:pass")
```

---

## 15. Exception Handling

```java
http.exceptionHandling(ex -> ex
    // 401 — Not authenticated
    .authenticationEntryPoint((request, response, authException) -> {
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setContentType("application/json");
        response.getWriter().write("""
            {"status": 401, "error": "Unauthorized", "message": "Please login first"}
            """);
    })
    // 403 — Authenticated but not authorized
    .accessDeniedHandler((request, response, accessDeniedException) -> {
        response.setStatus(HttpServletResponse.SC_FORBIDDEN);
        response.setContentType("application/json");
        response.getWriter().write("""
            {"status": 403, "error": "Forbidden", "message": "You don't have permission"}
            """);
    })
);
```

---

## 16. Security for REST APIs

### Complete REST API Security Setup

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain apiFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())             // No CSRF for REST
            .cors(cors -> cors.configurationSource(corsConfig()))
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)) // No sessions
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/auth/**").permitAll()
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers(HttpMethod.GET, "/api/products/**").permitAll()
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated()
            )
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class)
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(jwtAuthEntryPoint)
            );
        
        return http.build();
    }
}
```

---

## 17. Common Vulnerabilities & Mitigation

| Vulnerability | Attack | Spring Security Mitigation |
|---------------|--------|---------------------------|
| **CSRF** | Forged requests from other sites | CSRF token (enabled by default) |
| **XSS** | Injected malicious scripts | Content-Security-Policy header |
| **SQL Injection** | Malicious SQL in input | Use parameterized queries |
| **Session Fixation** | Hijack session after login | Migrate session (default) |
| **Brute Force** | Guess passwords | Account lockout, rate limiting |
| **Clickjacking** | Hidden iframe tricks | X-Frame-Options header |

```java
// Security headers (auto-configured by Spring Security)
http.headers(headers -> headers
    .frameOptions(frame -> frame.deny())              // Prevent clickjacking
    .contentSecurityPolicy(csp -> csp.policyDirectives("default-src 'self'"))
    .httpStrictTransportSecurity(hsts -> hsts.maxAgeInSeconds(31536000))
);
```

---

## 18. Best Practices

1. **Always use BCrypt** — Never store plain text passwords
2. **Use HTTPS** in production — Encrypt all traffic
3. **Disable CSRF for REST APIs** — Use JWT tokens instead
4. **Use method-level security** — @PreAuthorize for fine-grained control
5. **Least privilege** — Give minimum required permissions
6. **Don't expose security details** — Generic error messages for 401/403
7. **Use roles AND authorities** — Roles for broad access, authorities for specific actions
8. **Session fixation protection** — Use migrateSession (default)
9. **Validate all input** — Never trust client data
10. **Log authentication events** — Audit login success/failure

---

## 19. Interview Questions & Answers (50+)

### Beginner

**Q1: What is Spring Security?** Framework for authentication, authorization, and protection against common attacks. Handles login, roles, CSRF, CORS, sessions.

**Q2: Authentication vs Authorization?** Authentication: verify WHO you are (login). Authorization: verify WHAT you can do (permissions).

**Q3: What is the Security Filter Chain?** Series of filters processing every HTTP request. Each filter handles one security concern (CSRF, auth, authorization).

**Q4: What is UserDetailsService?** Interface with one method: `loadUserByUsername()`. Returns UserDetails object. Connect to YOUR database here.

**Q5: What is BCrypt?** Password hashing algorithm with random salt. One-way: can't reverse. Same password gives different hashes (due to salt).

**Q6: What is CSRF?** Cross-Site Request Forgery. Attacker tricks user's browser into making unwanted requests. Protected by CSRF tokens.

**Q7: What is @PreAuthorize?** Method-level security. Checks condition BEFORE method executes. Example: `@PreAuthorize("hasRole('ADMIN')")`.

**Q8: Role vs Authority?** Role is coarse-grained (ADMIN, USER). Authority is fine-grained (READ_USER, DELETE_USER). Role has ROLE_ prefix.

---

### Intermediate

**Q9: How does Spring Security authentication work?** Filter creates AuthToken → AuthenticationManager delegates to AuthenticationProvider → Provider uses UserDetailsService + PasswordEncoder → Sets SecurityContext.

**Q10: What is SecurityContext?** Holds the Authentication object for the current thread. Set after successful authentication. Accessible via SecurityContextHolder.

**Q11: What is GrantedAuthority?** Represents a permission. SimpleGrantedAuthority wraps a string like "ROLE_ADMIN" or "DELETE_USER".

**Q12: `hasRole` vs `hasAuthority`?** hasRole("ADMIN") checks for "ROLE_ADMIN". hasAuthority("ADMIN") checks for exactly "ADMIN". hasRole adds ROLE_ prefix.

**Q13: What is CORS?** Cross-Origin Resource Sharing. Browser blocks requests to different origins. Server must send Access-Control-Allow-Origin header.

**Q14: SessionCreationPolicy types?** ALWAYS (always create), IF_REQUIRED (create when needed, default), NEVER (don't create but use if exists), STATELESS (no sessions — for JWT).

**Q15: What is session fixation attack?** Attacker sets victim's session ID before login. After login, attacker uses the same session. Fix: migrate session on login.

---

### Advanced

**Q16: How does Spring Security integrate with JWT?** Custom filter extracts JWT from Authorization header, validates it, creates Authentication, sets SecurityContext. Session is STATELESS.

**Q17: What is DaoAuthenticationProvider?** Uses UserDetailsService to load user and PasswordEncoder to compare passwords. The default provider.

**Q18: How to implement multi-factor authentication?** Custom AuthenticationProvider that verifies password + OTP. Or use a custom filter after primary authentication.

**Q19: What is OAuth2 Resource Server?** Validates OAuth2 access tokens (JWT or opaque). `http.oauth2ResourceServer(oauth2 -> oauth2.jwt())`.

**Q20: What is Spring Security's method security?** @PreAuthorize, @PostAuthorize, @Secured, @RolesAllowed. Uses AOP proxies to intercept method calls.

---

### Rapid-Fire (Q21–Q50)

**Q21: Default login page URL?** `/login` (auto-generated if not customized).

**Q22: What is `@EnableWebSecurity`?** Enables Spring Security's web security configuration.

**Q23: What is `@EnableMethodSecurity`?** Enables @PreAuthorize and @PostAuthorize on methods.

**Q24: What is `SecurityFilterChain` bean?** Defines security rules (URL access, auth method, CSRF, etc.).

**Q25: What is `AuthenticationManager`?** Central interface for authentication. Delegates to AuthenticationProvider(s).

**Q26: What is `AuthenticationProvider`?** Performs actual authentication logic (check credentials).

**Q27: What is `PasswordEncoder`?** Interface for encoding/matching passwords. BCryptPasswordEncoder is the standard.

**Q28: What is `permitAll()`?** Allows access without authentication.

**Q29: What is `authenticated()`?** Requires any authenticated user.

**Q30: What is `denyAll()`?** Blocks all access.

**Q31: What is `anonymous()`?** Allows only non-authenticated users.

**Q32: What is `hasAnyRole()`?** Requires ANY of the specified roles.

**Q33: What is `@AuthenticationPrincipal`?** Injects current user's UserDetails into controller method.

**Q34: What is `remember-me` authentication?** Persists login across browser sessions using a cookie/token.

**Q35: What is `X-Frame-Options`?** HTTP header preventing clickjacking (DENY or SAMEORIGIN).

**Q36: What is `Content-Security-Policy`?** HTTP header controlling allowed resource sources (prevents XSS).

**Q37: What is `HSTS`?** HTTP Strict Transport Security — forces HTTPS.

**Q38: How to disable Spring Security for tests?** `@AutoConfigureMockMvc(addFilters = false)` or mock security context.

**Q39: What is `@WithMockUser`?** Test annotation that simulates an authenticated user.

**Q40: What is `@WithUserDetails`?** Test annotation loading user from actual UserDetailsService.

**Q41: What is `OncePerRequestFilter`?** Base class for filters guaranteed to execute once per request.

**Q42: What is `DelegatingFilterProxy`?** Servlet filter that delegates to Spring-managed filter bean.

**Q43: What is `FilterChainProxy`?** Spring Security's internal filter that manages the SecurityFilterChain.

**Q44: What is `formLogin()` vs `httpBasic()`?** formLogin: HTML form-based. httpBasic: Authorization header (base64).

**Q45: Default session timeout?** 30 minutes. Configure: `server.servlet.session.timeout=30m`.

**Q46: What is `maxSessionsPreventsLogin`?** When true, new login is rejected if max sessions reached. When false, oldest session is expired.

**Q47: How to get current user in service layer?** `SecurityContextHolder.getContext().getAuthentication().getName()`.

**Q48: What is `WebSecurityCustomizer`?** Ignores security for specific paths: `web.ignoring().requestMatchers("/static/**")`.

**Q49: What is `SecurityContextHolderStrategy`?** How SecurityContext is stored: ThreadLocal (default), InheritableThreadLocal, Global.

**Q50: Spring Security 6 vs 5 changes?** Lambda DSL (no `.and()`), `requestMatchers()` replaces `antMatchers()`, `@EnableMethodSecurity` replaces `@EnableGlobalMethodSecurity`.

---

## 📚 References

- [Spring Security Reference](https://docs.spring.io/spring-security/reference/)
- [Spring Security Architecture](https://docs.spring.io/spring-security/reference/servlet/architecture.html)
- [Baeldung Spring Security](https://www.baeldung.com/security-spring)
- [OWASP Cheat Sheets](https://cheatsheetseries.owasp.org/)

---

> **Previous Topic:** [← 16 - Spring Boot MVC](../16-project-springboot-mvc/README.md)  
> **Next Topic:** [18 - JWT →](../18-jwt/README.md)
