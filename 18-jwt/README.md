# 🎫 JWT (JSON Web Tokens) — Complete In-Depth Guide

> **"JWT is the standard for stateless authentication in modern REST APIs. Understand how it works inside-out to build secure, scalable systems."**

---

## 📑 Table of Contents

1. [What is JWT?](#1-what-is-jwt)
2. [JWT Structure](#2-jwt-structure)
3. [How JWT Authentication Works](#3-how-jwt-authentication-works)
4. [JWT Implementation in Spring Boot](#4-jwt-implementation-in-spring-boot)
5. [JWT Filter](#5-jwt-filter)
6. [Auth Controller (Login/Register)](#6-auth-controller-loginregister)
7. [Refresh Tokens](#7-refresh-tokens)
8. [Token Storage (Client-Side)](#8-token-storage-client-side)
9. [JWT Security Concerns](#9-jwt-security-concerns)
10. [Best Practices](#10-best-practices)
11. [Interview Questions & Answers (50+)](#11-interview-questions--answers-50)

---

## 1. What is JWT?

**JWT (JSON Web Token)** is a compact, URL-safe token that contains claims (user info) and is cryptographically signed. Used for **stateless authentication** — no session on server.

```
Session-Based Auth (Stateful):          JWT Auth (Stateless):
┌────────┐      ┌──────────┐           ┌────────┐      ┌──────────┐
│ Client │──→   │  Server  │           │ Client │──→   │  Server  │
│        │      │          │           │        │      │          │
│ Cookie:│      │ Session: │           │ Token: │      │ No state!│
│JSESSIONID=abc │{abc:Dilip}│          │JWT=eyJ..│      │ Validate │
│        │      │          │           │        │      │ token    │
└────────┘      └──────────┘           └────────┘      └──────────┘
  Server stores sessions                Server is STATELESS ✅
  Hard to scale (sticky sessions)       Easy to scale (any server works)
```

---

## 2. JWT Structure

```
A JWT has THREE parts separated by dots:

eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJkaWxpcEBtYWlsLmNvbSIsInJvbGUiOiJBRE1JTiIsImlhdCI6MTcwNTMxMjgwMCwiZXhwIjoxNzA1Mzk5MjAwfQ.abc123signature

[    HEADER    ].[     PAYLOAD     ].[  SIGNATURE  ]

═══ PART 1: HEADER (Algorithm + Type) ═══
Base64 decoded:
{
    "alg": "HS256",    // Signing algorithm (HMAC SHA-256)
    "typ": "JWT"       // Token type
}

═══ PART 2: PAYLOAD (Claims — User Data) ═══
Base64 decoded:
{
    "sub": "dilip@mail.com",  // Subject (user identifier)
    "role": "ADMIN",          // Custom claim
    "iat": 1705312800,        // Issued At (timestamp)
    "exp": 1705399200         // Expiration (timestamp)
}

═══ PART 3: SIGNATURE (Verification) ═══
HMACSHA256(
    base64UrlEncode(header) + "." + base64UrlEncode(payload),
    SECRET_KEY
)

⚠️ IMPORTANT: The payload is NOT encrypted! It's just Base64 encoded.
Anyone can READ the payload. But they can't MODIFY it (signature would break).
Don't put sensitive data (passwords, SSN) in the payload!
```

### Standard JWT Claims

| Claim | Name | Description |
|-------|------|-------------|
| `sub` | Subject | User identifier (email, user ID) |
| `iss` | Issuer | Who created the token |
| `aud` | Audience | Who the token is for |
| `exp` | Expiration | Token expiry time (Unix timestamp) |
| `iat` | Issued At | When the token was created |
| `nbf` | Not Before | Token not valid before this time |
| `jti` | JWT ID | Unique token identifier |

---

## 3. How JWT Authentication Works

```
═══ STEP 1: LOGIN ═══
Client: POST /api/auth/login
        Body: {"email": "dilip@mail.com", "password": "secret123"}

Server: 1. Verify credentials (check database)
        2. Generate JWT with user info
        3. Return JWT to client

Response: {"accessToken": "eyJhbG...", "refreshToken": "eyJhbG..."}


═══ STEP 2: ACCESS PROTECTED RESOURCE ═══
Client: GET /api/users/me
        Header: Authorization: Bearer eyJhbG...

Server: 1. Extract JWT from Authorization header
        2. Validate signature (not tampered?)
        3. Check expiration (not expired?)
        4. Extract user info from claims
        5. Load user & set SecurityContext
        6. Process request


═══ STEP 3: TOKEN EXPIRED ═══
Client: GET /api/users/me
        Header: Authorization: Bearer eyJhbG...(expired)

Server: Returns 401 Unauthorized

Client: POST /api/auth/refresh
        Body: {"refreshToken": "eyJhbG..."}

Server: 1. Validate refresh token
        2. Generate NEW access token
        3. Return new access token


═══ VISUAL FLOW ═══

┌────────┐                         ┌──────────┐
│ Client │   POST /auth/login      │  Server  │
│        │────────────────────────►│          │
│        │   {email, password}     │ Verify   │
│        │◄────────────────────────│ Generate │
│        │   {accessToken, refresh}│ JWT      │
│        │                         │          │
│        │   GET /api/users        │          │
│        │   Auth: Bearer eyJ...   │          │
│        │────────────────────────►│ Validate │
│        │◄────────────────────────│ JWT      │
│        │   {user data}           │ Process  │
└────────┘                         └──────────┘
```

---

## 4. JWT Implementation in Spring Boot

### Dependencies

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.12.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.12.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.12.5</version>
    <scope>runtime</scope>
</dependency>
```

### JwtService (Token Generation & Validation)

```java
@Service
public class JwtService {
    
    @Value("${app.jwt.secret}")
    private String secretKey;
    
    @Value("${app.jwt.access-token-expiration}")
    private long accessTokenExpiration;  // e.g., 86400000 (24 hours)
    
    @Value("${app.jwt.refresh-token-expiration}")
    private long refreshTokenExpiration; // e.g., 604800000 (7 days)
    
    // ═══════════════════════════════════════════
    // GENERATE Access Token
    // ═══════════════════════════════════════════
    public String generateAccessToken(UserDetails userDetails) {
        Map<String, Object> claims = new HashMap<>();
        
        if (userDetails instanceof CustomUserDetails customUser) {
            claims.put("userId", customUser.getUserId());
            claims.put("role", customUser.getUser().getRole().name());
        }
        
        return buildToken(claims, userDetails.getUsername(), accessTokenExpiration);
    }
    
    // ═══════════════════════════════════════════
    // GENERATE Refresh Token
    // ═══════════════════════════════════════════
    public String generateRefreshToken(UserDetails userDetails) {
        return buildToken(new HashMap<>(), userDetails.getUsername(), refreshTokenExpiration);
    }
    
    // ═══════════════════════════════════════════
    // BUILD Token (internal)
    // ═══════════════════════════════════════════
    private String buildToken(Map<String, Object> claims, String subject, long expiration) {
        return Jwts.builder()
            .claims(claims)                                    // Custom claims
            .subject(subject)                                  // User identifier
            .issuedAt(new Date())                              // Current time
            .expiration(new Date(System.currentTimeMillis() + expiration)) // Expiry
            .signWith(getSigningKey())                         // Sign with secret
            .compact();                                        // Build the JWT string
    }
    
    // ═══════════════════════════════════════════
    // VALIDATE Token
    // ═══════════════════════════════════════════
    public boolean isTokenValid(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return username.equals(userDetails.getUsername()) && !isTokenExpired(token);
    }
    
    // ═══════════════════════════════════════════
    // EXTRACT Username from Token
    // ═══════════════════════════════════════════
    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }
    
    // Extract any claim
    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }
    
    // Parse and verify the token
    private Claims extractAllClaims(String token) {
        return Jwts.parser()
            .verifyWith(getSigningKey())   // Verify signature
            .build()
            .parseSignedClaims(token)      // Parse token
            .getPayload();                 // Get claims
    }
    
    private boolean isTokenExpired(String token) {
        return extractClaim(token, Claims::getExpiration).before(new Date());
    }
    
    private SecretKey getSigningKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secretKey);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```

---

## 5. JWT Filter

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;
    
    public JwtAuthenticationFilter(JwtService jwtService, UserDetailsService userDetailsService) {
        this.jwtService = jwtService;
        this.userDetailsService = userDetailsService;
    }
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                     HttpServletResponse response,
                                     FilterChain filterChain)
            throws ServletException, IOException {
        
        // Step 1: Extract Authorization header
        final String authHeader = request.getHeader("Authorization");
        
        // Step 2: Check if it's a Bearer token
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);  // No token → continue without auth
            return;
        }
        
        // Step 3: Extract the JWT token (remove "Bearer " prefix)
        final String jwt = authHeader.substring(7);
        
        try {
            // Step 4: Extract username from token
            final String username = jwtService.extractUsername(jwt);
            
            // Step 5: If username exists and user is not already authenticated
            if (username != null && SecurityContextHolder.getContext().getAuthentication() == null) {
                
                // Step 6: Load user from database
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                
                // Step 7: Validate token
                if (jwtService.isTokenValid(jwt, userDetails)) {
                    
                    // Step 8: Create authentication token
                    UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(
                            userDetails,
                            null,
                            userDetails.getAuthorities()
                        );
                    
                    authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                    
                    // Step 9: Set authentication in SecurityContext
                    SecurityContextHolder.getContext().setAuthentication(authToken);
                }
            }
        } catch (ExpiredJwtException e) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.getWriter().write("{\"error\": \"Token expired\"}");
            return;
        } catch (JwtException e) {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            response.getWriter().write("{\"error\": \"Invalid token\"}");
            return;
        }
        
        // Step 10: Continue filter chain
        filterChain.doFilter(request, response);
    }
}
```

---

## 6. Auth Controller (Login/Register)

```java
@RestController
@RequestMapping("/api/auth")
@RequiredArgsConstructor
public class AuthController {
    
    private final AuthenticationManager authenticationManager;
    private final UserService userService;
    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;
    
    @PostMapping("/register")
    public ResponseEntity<AuthResponse> register(@Valid @RequestBody RegisterRequest request) {
        User user = userService.register(request);
        UserDetails userDetails = userDetailsService.loadUserByUsername(user.getEmail());
        
        String accessToken = jwtService.generateAccessToken(userDetails);
        String refreshToken = jwtService.generateRefreshToken(userDetails);
        
        return ResponseEntity.status(HttpStatus.CREATED)
            .body(new AuthResponse(accessToken, refreshToken));
    }
    
    @PostMapping("/login")
    public ResponseEntity<AuthResponse> login(@Valid @RequestBody LoginRequest request) {
        // Authenticate (throws BadCredentialsException if wrong)
        authenticationManager.authenticate(
            new UsernamePasswordAuthenticationToken(request.getEmail(), request.getPassword())
        );
        
        UserDetails userDetails = userDetailsService.loadUserByUsername(request.getEmail());
        
        String accessToken = jwtService.generateAccessToken(userDetails);
        String refreshToken = jwtService.generateRefreshToken(userDetails);
        
        return ResponseEntity.ok(new AuthResponse(accessToken, refreshToken));
    }
    
    @PostMapping("/refresh")
    public ResponseEntity<AuthResponse> refresh(@RequestBody RefreshRequest request) {
        String username = jwtService.extractUsername(request.getRefreshToken());
        UserDetails userDetails = userDetailsService.loadUserByUsername(username);
        
        if (!jwtService.isTokenValid(request.getRefreshToken(), userDetails)) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED).build();
        }
        
        String newAccessToken = jwtService.generateAccessToken(userDetails);
        return ResponseEntity.ok(new AuthResponse(newAccessToken, request.getRefreshToken()));
    }
}

// Request/Response DTOs
public record LoginRequest(@NotBlank String email, @NotBlank String password) {}
public record RegisterRequest(@NotBlank String name, @Email String email, 
                                @Size(min = 8) String password) {}
public record RefreshRequest(@NotBlank String refreshToken) {}
public record AuthResponse(String accessToken, String refreshToken) {}
```

---

## 7. Refresh Tokens

```
WHY REFRESH TOKENS?

Access Token:  Short-lived (15 min – 24 hours)
  If stolen → attacker has limited time window
  
Refresh Token: Long-lived (7 – 30 days)  
  Used ONLY to get a new access token
  Stored more securely (httpOnly cookie)

Flow:
  1. Login → Get access token (15 min) + refresh token (7 days)
  2. Use access token for API calls
  3. Access token expires → 401
  4. Send refresh token to /auth/refresh → Get NEW access token
  5. Continue with new access token
  6. Refresh token expires → Must login again

Security:
  Access token leaked → Attacker has 15 minutes
  Without refresh token → Must know credentials to get new tokens
```

---

## 8. Token Storage (Client-Side)

```
WHERE to store JWT on the client?

1. localStorage:
   ✅ Easy to use
   ❌ Vulnerable to XSS (JavaScript can read it)
   
2. sessionStorage:
   ✅ Cleared when tab closes
   ❌ Still vulnerable to XSS

3. httpOnly Cookie: (RECOMMENDED ✅)
   ✅ JavaScript CAN'T read it (httpOnly flag)
   ✅ Sent automatically with requests
   ❌ Vulnerable to CSRF (but can be mitigated)
   
4. In-Memory (JavaScript variable):
   ✅ Most secure (can't be accessed by other scripts)
   ❌ Lost on page refresh
   
Best Practice: Access token in memory + Refresh token in httpOnly cookie
```

---

## 9. JWT Security Concerns

```
1. Token Theft: If someone gets your JWT, they ARE you until it expires.
   Mitigation: Short expiration, HTTPS only, httpOnly cookies.

2. No Revocation: Can't invalidate a JWT before it expires.
   Mitigation: Short expiration + token blacklist (Redis) + refresh tokens.

3. Payload is NOT encrypted: Anyone can Base64-decode and read claims.
   Mitigation: Don't put sensitive data in JWT payload.

4. Algorithm Confusion: Attacker changes alg to "none" to bypass signature.
   Mitigation: Always validate algorithm, use library that prevents this.

5. Secret Key Exposure: If secret is leaked, all tokens can be forged.
   Mitigation: Use strong keys, rotate periodically, use environment variables.
```

---

## 10. Best Practices

1. **Short-lived access tokens** — 15 minutes to 24 hours max
2. **Use refresh tokens** — Long-lived, stored in httpOnly cookies
3. **Use HTTPS** — Always! Tokens are credentials.
4. **Strong secret key** — At least 256 bits for HS256
5. **Don't store sensitive data in JWT** — Payload is readable
6. **Validate everything** — Algorithm, signature, expiration, issuer
7. **Use environment variables** for secrets — Never hardcode
8. **Implement token blacklist** for logout — Redis works well
9. **Set issuer and audience** claims — Prevent token misuse
10. **Use RS256 for microservices** — Public key verification without sharing secret

---

## 11. Interview Questions & Answers (50+)

### Beginner

**Q1: What is JWT?** JSON Web Token — a compact, signed token containing user claims. Used for stateless authentication.

**Q2: What are the three parts of JWT?** Header (algorithm), Payload (claims/data), Signature (verification hash). Separated by dots.

**Q3: Is JWT encrypted?** No! It's signed (integrity) but NOT encrypted (confidentiality). Anyone can read the payload. Don't store secrets in it.

**Q4: What is the difference between session and JWT authentication?** Session: server stores state, uses cookies. JWT: server is stateless, token contains all info.

**Q5: What is a Bearer token?** Token sent in Authorization header: `Authorization: Bearer eyJ...`. Server "bears" trust in whoever presents the token.

**Q6: How to send JWT with a request?** In the Authorization header: `Authorization: Bearer <token>`.

**Q7: What is token expiration?** The `exp` claim — Unix timestamp when token becomes invalid. Server rejects expired tokens.

**Q8: What is a refresh token?** Long-lived token used to get new access tokens without re-login.

---

### Intermediate

**Q9: HS256 vs RS256?** HS256: symmetric (same secret for sign + verify). RS256: asymmetric (private key signs, public key verifies). RS256 better for microservices.

**Q10: How to logout with JWT?** JWT can't be invalidated. Solutions: token blacklist (Redis), short expiration, delete token on client.

**Q11: Where to store JWT on client?** Best: access token in memory, refresh token in httpOnly cookie. Avoid localStorage (XSS risk).

**Q12: What is token revocation?** Invalidating a token before expiry. JWT doesn't support this natively. Use blacklist in Redis.

**Q13: Can JWT be used for authorization?** Yes! Store roles/permissions in claims. Server reads claims to check access.

**Q14: What is the `sub` claim?** Subject — identifies the user. Usually email or user ID.

**Q15: How does JWT prevent tampering?** Signature is calculated from header + payload + secret. If payload changes, signature won't match.

---

### Advanced

**Q16: What is JWT algorithm confusion attack?** Attacker changes algorithm to "none" — no signature needed. Prevention: always validate algorithm on server.

**Q17: JWS vs JWE?** JWS (JSON Web Signature): signed, NOT encrypted (standard JWT). JWE (JSON Web Encryption): encrypted. Use JWE for sensitive payload.

**Q18: How to implement token rotation?** Each refresh generates new access + refresh tokens. Old refresh token is invalidated. Limits damage from stolen refresh token.

**Q19: What is JWKS (JSON Web Key Set)?** Public key endpoint for RS256. Auth server publishes keys at `/.well-known/jwks.json`. Resource servers fetch keys to verify JWTs.

**Q20: How to handle JWT in microservices?** API Gateway validates JWT and forwards claims. Or: each service validates using shared public key (RS256).

---

### Rapid-Fire (Q21–Q50)

**Q21: Default JWT library for Spring?** jjwt (io.jsonwebtoken) or spring-security-oauth2-jose.

**Q22: What is `Claims` in JJWT?** Payload of the JWT containing key-value pairs.

**Q23: What is `SignWith()`?** Method to specify signing key and algorithm.

**Q24: What is `OncePerRequestFilter`?** Base class ensuring filter executes once per request.

**Q25: Where to add JWT filter in chain?** Before `UsernamePasswordAuthenticationFilter`.

**Q26: SessionCreationPolicy for JWT?** `STATELESS` — no HttpSession created.

**Q27: CSRF needed with JWT?** No! JWT is not cookie-based. CSRF only affects cookies.

**Q28: What if JWT secret is compromised?** All tokens can be forged. Rotate secret immediately. All existing tokens become invalid.

**Q29: JWT size limit?** No hard limit, but should be small. Headers have practical limits (~8KB).

**Q30: Can JWT carry user permissions?** Yes, as custom claims: `"roles": ["ADMIN", "USER"]`.

**Q31: What is `iat` claim?** Issued At — timestamp when token was created.

**Q32: What is `nbf` claim?** Not Before — token not valid before this time.

**Q33: What is `jti` claim?** JWT ID — unique identifier to prevent replay attacks.

**Q34: What is `iss` claim?** Issuer — who created the token.

**Q35: What is `aud` claim?** Audience — who the token is intended for.

**Q36: Base64 vs Base64URL?** Base64URL replaces +/= with -/_ for URL safety. JWT uses Base64URL.

**Q37: Can JWT be used in WebSocket?** Yes, send JWT as query param or first message.

**Q38: What is token sliding expiration?** Extending token expiry on each use (not standard with JWT).

**Q39: JWT vs API key?** JWT: user-specific, expiring, contains claims. API key: service-level, long-lived, no user info.

**Q40: JWT vs OAuth2?** JWT is a token format. OAuth2 is an authorization framework. OAuth2 often USES JWT tokens.

**Q41: What is opaque token?** Random string (not JWT). Server must look up token in database. Opposite of self-contained JWT.

**Q42: What is self-contained token?** Token contains all info needed (like JWT). No database lookup needed.

**Q43: Symmetric vs asymmetric signing?** Symmetric (HS256): one shared secret. Asymmetric (RS256): private key signs, public key verifies.

**Q44: What is key rotation?** Periodically changing signing keys. New tokens use new key. Old tokens verified with old key until they expire.

**Q45: How to decode JWT without verifying?** Base64-decode header and payload. Libraries provide methods for this.

**Q46: What is `compact()` in JJWT?** Builds the final JWT string from the builder.

**Q47: What is `parseSignedClaims()`?** Parses and verifies a JWT, returns claims.

**Q48: Can JWT contain arrays?** Yes: `"roles": ["ADMIN", "USER"]` is valid JSON.

**Q49: Max recommended JWT expiration?** Access token: 15 min – 1 hour. Refresh token: 7 – 30 days.

**Q50: How to test JWT endpoints?** Use Postman/cURL with Authorization header. In tests, generate test tokens or use `@WithMockUser`.

---

## 📚 References

- [JWT.io (Debugger)](https://jwt.io/)
- [RFC 7519 (JWT Spec)](https://tools.ietf.org/html/rfc7519)
- [JJWT Library](https://github.com/jwtk/jjwt)
- [Spring Security JWT](https://docs.spring.io/spring-security/reference/servlet/oauth2/resource-server/jwt.html)

---

> **Previous Topic:** [← 17 - Spring Security](../17-spring-security/README.md)  
> **Next Topic:** [19 - OAuth2 →](../19-oauth2/README.md)
