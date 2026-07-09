# 🔗 REST API & Web Services — Complete In-Depth Guide

> **"REST APIs are the backbone of modern applications. Every microservice, mobile app, and SPA communicates through REST APIs."**

---

## 📑 Table of Contents

1. [Introduction & Overview](#1-introduction--overview)
2. [REST Architecture & Principles](#2-rest-architecture--principles)
3. [HTTP Protocol Deep Dive](#3-http-protocol-deep-dive)
4. [HTTP Methods (CRUD)](#4-http-methods-crud)
5. [HTTP Status Codes](#5-http-status-codes)
6. [URL Design & Best Practices](#6-url-design--best-practices)
7. [Request & Response Structure](#7-request--response-structure)
8. [JSON & Data Formats](#8-json--data-formats)
9. [Authentication & Authorization](#9-authentication--authorization)
10. [API Versioning](#10-api-versioning)
11. [Pagination, Filtering, Sorting](#11-pagination-filtering-sorting)
12. [HATEOAS](#12-hateoas)
13. [SOAP vs REST vs GraphQL vs gRPC](#13-soap-vs-rest-vs-graphql-vs-grpc)
14. [API Documentation (Swagger/OpenAPI)](#14-api-documentation-swaggeropenapi)
15. [Error Handling](#15-error-handling)
16. [CORS (Cross-Origin Resource Sharing)](#16-cors-cross-origin-resource-sharing)
17. [Rate Limiting & Throttling](#17-rate-limiting--throttling)
18. [API Security Best Practices](#18-api-security-best-practices)
19. [Testing REST APIs](#19-testing-rest-apis)
20. [Interview Questions & Answers (50+)](#20-interview-questions--answers-50)

---

## 1. Introduction & Overview

### What is an API?

**API (Application Programming Interface)** — A set of rules that allows one software application to talk to another.

```
Your App                     Twitter API
┌──────────┐    HTTP Request   ┌──────────────┐
│          │──────────────────►│              │
│  Client  │   GET /tweets     │   Server     │
│          │◄──────────────────│              │
│          │    JSON Response   │              │
└──────────┘                   └──────────────┘
```

### What is REST?

**REST (Representational State Transfer)** — An architectural style for designing APIs. It uses HTTP protocol and is based on **resources** (data entities).

### What is a Web Service?

A web service is an API accessible over the network via HTTP. Types:
- **SOAP** — XML-based, strict, WS-Security (enterprise/legacy)
- **REST** — JSON-based, flexible, lightweight (modern standard ✅)
- **GraphQL** — Query language, client specifies data needed
- **gRPC** — Binary protocol (Protocol Buffers), very fast

---

## 2. REST Architecture & Principles

### 6 REST Constraints

| # | Constraint | Meaning |
|---|-----------|---------|
| 1 | **Client-Server** | Client and server are separate, communicate via HTTP |
| 2 | **Stateless** | Server doesn't store client state. Each request has ALL info needed |
| 3 | **Cacheable** | Responses can be cached to improve performance |
| 4 | **Uniform Interface** | Consistent URL patterns, HTTP methods, response formats |
| 5 | **Layered System** | Client doesn't know if it talks to server directly or via proxy |
| 6 | **Code on Demand** | (Optional) Server can send executable code to client |

### Statelessness — What It Means

```
STATEFUL (Server remembers):
  Request 1: "Login as Dilip"     → Server: "OK, I remember you're Dilip"
  Request 2: "Show my profile"    → Server: "You're Dilip, here's your profile"

STATELESS (REST — Server doesn't remember):
  Request 1: "Login as Dilip"     → Server: "Here's a token: xyz123"
  Request 2: "Show my profile"    → MUST include token!
             Header: "Authorization: Bearer xyz123"
             → Server: "Token valid for Dilip, here's your profile"
  
  Each request is INDEPENDENT. No "session" on server.
```

---

## 3. HTTP Protocol Deep Dive

### HTTP Request Structure

```
POST /api/users HTTP/1.1                    ← Request Line
Host: api.example.com                       ← Headers
Content-Type: application/json
Authorization: Bearer eyJhbGciOi...
Accept: application/json
                                            ← Empty Line
{                                           ← Body (payload)
    "name": "Dilip",
    "email": "dilip@example.com"
}
```

### HTTP Response Structure

```
HTTP/1.1 201 Created                        ← Status Line
Content-Type: application/json              ← Headers
Location: /api/users/42
Cache-Control: no-cache
                                            ← Empty Line
{                                           ← Body
    "id": 42,
    "name": "Dilip",
    "email": "dilip@example.com",
    "createdAt": "2024-01-15T10:30:00Z"
}
```

---

## 4. HTTP Methods (CRUD)

| Method | CRUD | Description | Idempotent | Safe |
|--------|------|-------------|------------|------|
| **GET** | Read | Fetch resource(s) | ✅ Yes | ✅ Yes |
| **POST** | Create | Create new resource | ❌ No | ❌ No |
| **PUT** | Update | Replace entire resource | ✅ Yes | ❌ No |
| **PATCH** | Update | Partial update | ❌ No | ❌ No |
| **DELETE** | Delete | Remove resource | ✅ Yes | ❌ No |
| HEAD | — | Like GET but no body | ✅ Yes | ✅ Yes |
| OPTIONS | — | Get supported methods | ✅ Yes | ✅ Yes |

```
Idempotent: Calling it multiple times has the SAME effect as calling it once.
  PUT /users/1 {name: "Dilip"} → Always sets name to "Dilip"
  DELETE /users/1 → Deletes user 1 (calling again still results in "no user 1")
  
  POST /users {name: "Dilip"} → Creates NEW user each time! Not idempotent.

Safe: Doesn't modify data on the server. Only GET and HEAD are safe.
```

### CRUD Mapping Example

```
Resource: Users

GET    /api/users          → Get ALL users (list)
GET    /api/users/42       → Get user with ID 42
POST   /api/users          → Create a new user
PUT    /api/users/42       → Replace user 42 entirely
PATCH  /api/users/42       → Update specific fields of user 42
DELETE /api/users/42       → Delete user 42

Nested Resource: User's Orders
GET    /api/users/42/orders       → Get all orders for user 42
GET    /api/users/42/orders/7     → Get order 7 of user 42
POST   /api/users/42/orders       → Create new order for user 42
```

---

## 5. HTTP Status Codes

### Status Code Categories

| Range | Category | Meaning |
|-------|----------|---------|
| 1xx | Informational | Request received, continuing |
| **2xx** | **Success** | Request succeeded ✅ |
| **3xx** | **Redirection** | Further action needed |
| **4xx** | **Client Error** | Bad request ❌ |
| **5xx** | **Server Error** | Server failed 💥 |

### Most Important Status Codes

```
═══ 2xx Success ═══
200 OK           → GET successful, data in response body
201 Created      → POST successful, new resource created
204 No Content   → DELETE successful, no body to return
202 Accepted     → Request accepted for async processing

═══ 3xx Redirection ═══
301 Moved Permanently  → Resource moved, use new URL always
302 Found              → Temporary redirect
304 Not Modified       → Cached version is still valid

═══ 4xx Client Errors ═══
400 Bad Request        → Invalid request body/params (validation error)
401 Unauthorized       → Not authenticated (no token or expired)
403 Forbidden          → Authenticated but not authorized (no permission)
404 Not Found          → Resource doesn't exist
405 Method Not Allowed → Using POST on a GET-only endpoint
409 Conflict           → Conflict (duplicate email, version mismatch)
415 Unsupported Media  → Wrong Content-Type header
422 Unprocessable      → Validation error (semantically wrong)
429 Too Many Requests  → Rate limit exceeded

═══ 5xx Server Errors ═══
500 Internal Server Error  → Unhandled exception on server
502 Bad Gateway            → Upstream server error (proxy/load balancer)
503 Service Unavailable    → Server overloaded or maintenance
504 Gateway Timeout        → Upstream server didn't respond in time
```

---

## 6. URL Design & Best Practices

### Good vs Bad URL Design

```
✅ GOOD (RESTful):
GET    /api/users                      Plural nouns for collections
GET    /api/users/42                   Resource ID in path
GET    /api/users/42/orders            Nested resources
GET    /api/users?status=active        Query params for filtering
GET    /api/users?page=2&size=20       Query params for pagination
POST   /api/users                      Create (no ID in URL)
PUT    /api/users/42                   Update (ID in URL)
DELETE /api/users/42                   Delete (ID in URL)

❌ BAD:
GET /api/getUser                       Don't use verbs!
GET /api/user/42                       Use PLURAL (users, not user)
POST /api/createUser                   HTTP method IS the verb
GET /api/users/delete/42               Don't use action words in URL
GET /api/Users/42                      Use lowercase
GET /api/user_list                     Use hyphens, not underscores
```

### URL Design Rules

```
1. Use NOUNS, not verbs          → /users not /getUsers
2. Use PLURAL                     → /users not /user
3. Use LOWERCASE                  → /users not /Users
4. Use HYPHENS for readability    → /order-items not /orderItems
5. Use PATH for identity          → /users/42 (resource ID)
6. Use QUERY for filtering        → /users?role=admin
7. Nest resources logically       → /users/42/orders (user's orders)
8. Don't nest too deep            → Max 2 levels: /users/42/orders
9. Version your API               → /api/v1/users
```

---

## 7. Request & Response Structure

### Typical API Request Examples

```
═══ GET — Fetch a user ═══
GET /api/v1/users/42 HTTP/1.1
Host: api.example.com
Accept: application/json
Authorization: Bearer eyJhbGciOi...

═══ POST — Create a user ═══
POST /api/v1/users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer eyJhbGciOi...

{
    "name": "Dilip",
    "email": "dilip@example.com",
    "password": "secret123",
    "role": "USER"
}

═══ PUT — Replace a user ═══
PUT /api/v1/users/42 HTTP/1.1
Content-Type: application/json
Authorization: Bearer eyJhbGciOi...

{
    "name": "Dilip Kumar",
    "email": "dilip@example.com",
    "role": "ADMIN"
}

═══ PATCH — Partial update ═══
PATCH /api/v1/users/42 HTTP/1.1
Content-Type: application/json

{
    "role": "ADMIN"
}
```

### Standard Response Format

```json
// Success Response
{
    "status": "success",
    "data": {
        "id": 42,
        "name": "Dilip",
        "email": "dilip@example.com",
        "role": "ADMIN",
        "createdAt": "2024-01-15T10:30:00Z",
        "updatedAt": "2024-03-20T14:45:00Z"
    }
}

// List Response with Pagination
{
    "status": "success",
    "data": [
        {"id": 1, "name": "Alice"},
        {"id": 2, "name": "Bob"}
    ],
    "pagination": {
        "page": 1,
        "size": 20,
        "totalPages": 5,
        "totalElements": 100
    }
}

// Error Response
{
    "status": "error",
    "error": {
        "code": "VALIDATION_ERROR",
        "message": "Validation failed",
        "details": [
            {"field": "email", "message": "Email is not valid"},
            {"field": "name", "message": "Name is required"}
        ]
    },
    "timestamp": "2024-01-15T10:30:00Z",
    "path": "/api/v1/users"
}
```

---

## 8. JSON & Data Formats

### JSON Syntax

```json
{
    "string": "Hello",
    "number": 42,
    "decimal": 3.14,
    "boolean": true,
    "null_value": null,
    "array": [1, 2, 3],
    "object": {
        "key": "value"
    },
    "nested_array": [
        {"id": 1, "name": "Alice"},
        {"id": 2, "name": "Bob"}
    ]
}
```

### JSON vs XML

```
JSON (Modern ✅):                     XML (Legacy):
{                                     <user>
    "name": "Dilip",                    <name>Dilip</name>
    "age": 25,                          <age>25</age>
    "skills": ["Java", "Spring"]        <skills>
}                                         <skill>Java</skill>
                                          <skill>Spring</skill>
                                        </skills>
                                      </user>

JSON: Lighter, easier to read/parse, used by REST APIs
XML: Verbose, supports schemas, used by SOAP
```

---

## 9. Authentication & Authorization

### Common Authentication Methods

```
1. API Key:
   GET /api/users
   X-API-Key: abc123def456

2. Basic Auth:
   GET /api/users
   Authorization: Basic dXNlcjpwYXNz    (base64 of "user:pass")

3. Bearer Token (JWT) — Most Common for REST APIs ✅
   GET /api/users
   Authorization: Bearer eyJhbGciOi...

4. OAuth 2.0:
   GET /api/users
   Authorization: Bearer <access_token_from_oauth_flow>
```

### Authentication vs Authorization

```
Authentication (AuthN): "WHO are you?"
  → Login, verify identity (username/password, JWT, OAuth)

Authorization (AuthZ): "WHAT can you do?"
  → Check permissions (admin can delete, user can only read)

Flow:
  1. User sends credentials → Server verifies → Issues JWT token
  2. User sends JWT with every request
  3. Server validates JWT → Extracts user/roles
  4. Server checks if this role can access this endpoint
```

---

## 10. API Versioning

### Four Versioning Strategies

```
1. URL Path Versioning (Most Common ✅):
   GET /api/v1/users
   GET /api/v2/users

2. Query Parameter:
   GET /api/users?version=1
   GET /api/users?version=2

3. Header Versioning:
   GET /api/users
   Accept: application/vnd.myapp.v1+json

4. Media Type:
   GET /api/users
   Accept: application/vnd.myapp.v1+json
```

---

## 11. Pagination, Filtering, Sorting

### Pagination

```
GET /api/users?page=0&size=20

Response:
{
    "content": [...],
    "pageable": {
        "pageNumber": 0,
        "pageSize": 20
    },
    "totalPages": 5,
    "totalElements": 100,
    "first": true,
    "last": false
}
```

### Filtering

```
GET /api/users?status=active&role=admin&age=25
GET /api/users?minAge=18&maxAge=65
GET /api/users?name=Dilip
GET /api/products?category=electronics&minPrice=100&maxPrice=500
```

### Sorting

```
GET /api/users?sort=name,asc
GET /api/users?sort=createdAt,desc
GET /api/users?sort=name,asc&sort=age,desc    (multi-sort)
```

---

## 12. HATEOAS

**HATEOAS (Hypermedia As The Engine Of Application State)** — Response includes links to related actions.

```json
{
    "id": 42,
    "name": "Dilip",
    "email": "dilip@example.com",
    "_links": {
        "self": {"href": "/api/users/42"},
        "orders": {"href": "/api/users/42/orders"},
        "update": {"href": "/api/users/42", "method": "PUT"},
        "delete": {"href": "/api/users/42", "method": "DELETE"}
    }
}
```

---

## 13. SOAP vs REST vs GraphQL vs gRPC

| Feature | SOAP | REST | GraphQL | gRPC |
|---------|------|------|---------|------|
| Protocol | XML over HTTP/SMTP | JSON over HTTP | JSON over HTTP | Protobuf over HTTP/2 |
| Speed | Slow | Medium | Medium | Very Fast |
| Contract | WSDL (strict) | OpenAPI (optional) | Schema (strict) | .proto (strict) |
| Flexibility | Rigid | Flexible | Very Flexible | Rigid |
| Use Case | Enterprise/banking | Web APIs | Complex queries | Microservices |
| Over-fetching | Yes | Yes | No (client specifies) | No |
| Learning Curve | High | Low | Medium | Medium |
| Caching | Difficult | Easy (HTTP cache) | Complex | N/A |

---

## 14. API Documentation (Swagger/OpenAPI)

### Spring Boot Setup

```xml
<dependency>
    <groupId>org.springdoc</groupId>
    <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
    <version>2.3.0</version>
</dependency>
```

```yaml
# application.yml
springdoc:
  api-docs:
    path: /api-docs
  swagger-ui:
    path: /swagger-ui.html
```

```java
@RestController
@RequestMapping("/api/v1/users")
@Tag(name = "Users", description = "User management APIs")
public class UserController {
    
    @Operation(summary = "Get all users", description = "Returns paginated list of users")
    @ApiResponses({
        @ApiResponse(responseCode = "200", description = "Success"),
        @ApiResponse(responseCode = "401", description = "Unauthorized")
    })
    @GetMapping
    public Page<UserDTO> getUsers(@RequestParam(defaultValue = "0") int page,
                                   @RequestParam(defaultValue = "20") int size) {
        // ...
    }
}
```

---

## 15. Error Handling

```java
// Consistent error response
public class ApiError {
    private int status;
    private String error;
    private String message;
    private String path;
    private LocalDateTime timestamp;
    private List<FieldError> fieldErrors;
}

// Global exception handler
@RestControllerAdvice
public class GlobalExceptionHandler {
    
    @ExceptionHandler(ResourceNotFoundException.class)
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ApiError handleNotFound(ResourceNotFoundException ex, HttpServletRequest req) {
        return new ApiError(404, "Not Found", ex.getMessage(), req.getRequestURI());
    }
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ApiError handleValidation(MethodArgumentNotValidException ex) {
        List<FieldError> errors = ex.getBindingResult().getFieldErrors()
            .stream()
            .map(e -> new FieldError(e.getField(), e.getDefaultMessage()))
            .collect(Collectors.toList());
        
        return new ApiError(400, "Validation Error", "Request validation failed", errors);
    }
}
```

---

## 16. CORS (Cross-Origin Resource Sharing)

```
PROBLEM: Browser blocks requests from http://localhost:3000 (React)
         to http://localhost:8080 (Spring Boot) — different ORIGINS!

SOLUTION: Server tells browser "it's OK to accept requests from other origins"
```

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins("http://localhost:3000")
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowedHeaders("*")
            .allowCredentials(true)
            .maxAge(3600);
    }
}
```

---

## 17. Rate Limiting & Throttling

```
Rate limiting prevents abuse by limiting requests per time period.

Common strategies:
  Fixed window:   100 requests per minute
  Sliding window: 100 requests in any 60-second window
  Token bucket:   Tokens replenish at fixed rate

Response when rate limited:
  HTTP/1.1 429 Too Many Requests
  Retry-After: 60
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 0
  X-RateLimit-Reset: 1705312800
```

---

## 18. API Security Best Practices

```
1. Always use HTTPS (never HTTP in production)
2. Authenticate every request (JWT, OAuth2)
3. Authorize based on roles/permissions
4. Validate ALL input (never trust client data)
5. Use rate limiting to prevent abuse
6. Don't expose internal IDs (use UUIDs)
7. Don't return sensitive data (passwords, tokens)
8. Use CORS properly (don't allow *)
9. Log all API access for auditing
10. Use API keys for machine-to-machine
11. Set proper Content-Type headers
12. Handle errors consistently (don't leak stack traces)
13. Version your API
14. Use OWASP guidelines
```

---

## 19. Testing REST APIs

### Tools for Testing

| Tool | Type | Best For |
|------|------|----------|
| **Postman** | GUI | Manual testing, collections |
| **cURL** | CLI | Quick tests, scripts |
| **HTTPie** | CLI | Human-friendly cURL alternative |
| **JUnit + MockMvc** | Code | Automated testing in Spring |
| **RestAssured** | Code | BDD-style API testing |

### cURL Examples

```bash
# GET
curl -X GET http://localhost:8080/api/users

# POST with JSON
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name": "Dilip", "email": "dilip@example.com"}'

# PUT with auth
curl -X PUT http://localhost:8080/api/users/42 \
  -H "Authorization: Bearer eyJhbGci..." \
  -H "Content-Type: application/json" \
  -d '{"name": "Dilip Kumar"}'

# DELETE
curl -X DELETE http://localhost:8080/api/users/42 \
  -H "Authorization: Bearer eyJhbGci..."
```

---

## 20. Interview Questions & Answers (50+)

### Beginner Level

**Q1: What is REST?**
**A:** REST (Representational State Transfer) is an architectural style for designing web APIs using HTTP methods (GET, POST, PUT, DELETE) to perform CRUD operations on resources identified by URLs.

**Q2: What is the difference between REST and SOAP?**
**A:** REST: lightweight, JSON, flexible, easy to learn. SOAP: heavyweight, XML, strict contracts (WSDL), WS-Security. REST is the modern standard.

**Q3: What are the HTTP methods used in REST?**
**A:** GET (read), POST (create), PUT (full update), PATCH (partial update), DELETE (remove). HEAD (headers only), OPTIONS (allowed methods).

**Q4: What does stateless mean?**
**A:** Each request must contain ALL information needed to process it. Server doesn't store client state between requests. Use tokens (JWT) for authentication.

**Q5: What is the difference between PUT and PATCH?**
**A:** PUT replaces the ENTIRE resource. PATCH updates only the specified fields. PUT is idempotent, PATCH may not be.

**Q6: What is the difference between PUT and POST?**
**A:** POST creates a NEW resource (server assigns ID). PUT replaces/creates a resource at a SPECIFIC URL. POST is not idempotent, PUT is.

**Q7: What does idempotent mean?**
**A:** Multiple identical requests have the same effect as a single request. GET, PUT, DELETE are idempotent. POST is NOT (creates new resource each time).

**Q8: What is JSON?**
**A:** JavaScript Object Notation — lightweight data format with key-value pairs. Standard format for REST APIs. Easier to read and parse than XML.

---

### Intermediate Level

**Q9: What is HATEOAS?**
**A:** Hypermedia As The Engine Of Application State — REST responses include links to related actions. Clients discover available operations from links rather than hardcoding URLs.

**Q10: Explain HTTP status code categories.**
**A:** 1xx: Informational. 2xx: Success. 3xx: Redirection. 4xx: Client error. 5xx: Server error. Most important: 200 (OK), 201 (Created), 400 (Bad Request), 401 (Unauthorized), 404 (Not Found), 500 (Internal Error).

**Q11: Difference between 401 and 403?**
**A:** 401 Unauthorized: not authenticated (no token or invalid token). 403 Forbidden: authenticated but not authorized (no permission for this resource).

**Q12: How do you version a REST API?**
**A:** URL path (/api/v1/users — most common), query param (?version=1), header (Accept: application/vnd.app.v1+json), media type.

**Q13: What is CORS?**
**A:** Cross-Origin Resource Sharing — browser security that blocks requests to different origins. Server must send Access-Control-Allow-Origin header to allow cross-origin requests.

**Q14: What is content negotiation?**
**A:** Client specifies desired response format via Accept header. Server responds in that format. Accept: application/json → JSON response.

**Q15: What is the Richardson Maturity Model?**
**A:** Level 0: Single URL, single method. Level 1: Multiple URLs (resources). Level 2: Multiple HTTP methods. Level 3: HATEOAS. Higher level = more RESTful.

---

### Advanced Level

**Q16: How would you design a REST API for a social media app?**
**A:** Resources: /users, /posts, /comments, /likes. Nested: /users/{id}/posts. Use proper HTTP methods. Paginate lists. Use JWT auth. Consistent error format.

**Q17: What is API rate limiting?**
**A:** Limiting the number of requests a client can make per time period. Prevents abuse. Returns 429 (Too Many Requests). Headers: X-RateLimit-Limit, X-RateLimit-Remaining.

**Q18: REST vs GraphQL — when to use each?**
**A:** REST: simple CRUD, caching important, well-defined resources. GraphQL: complex nested data, multiple client types needing different data shapes, avoiding over/under-fetching.

**Q19: How do you handle partial updates?**
**A:** Use PATCH with only the fields to update. Validate that at least one field is present. Return the full updated resource in response.

**Q20: What is API gateway?**
**A:** Entry point for all API requests. Handles routing, rate limiting, authentication, load balancing, logging. Examples: Kong, AWS API Gateway, Spring Cloud Gateway.

---

### Rapid-Fire (Q21–Q50)

**Q21: What is a resource in REST?** Any entity that can be named and addressed: User, Product, Order.

**Q22: What is content type?** Header indicating the format of the request/response body: application/json, text/html.

**Q23: What is idempotency key?** Unique key sent with POST to prevent duplicate processing. Server returns same response for same key.

**Q24: What is ETag?** Entity tag for caching — hash of the response. Client sends If-None-Match, server returns 304 if unchanged.

**Q25: What is pagination cursor-based vs offset-based?** Offset: page=2&size=20 (can skip/miss records). Cursor: after=lastId (consistent, no duplicates).

**Q26: What is OpenAPI?** Standard specification for documenting REST APIs. Swagger UI renders it as interactive docs.

**Q27: Safe vs unsafe HTTP methods?** Safe: GET, HEAD (don't modify data). Unsafe: POST, PUT, DELETE (modify data).

**Q28: What is a webhook?** Server pushes events to client URL (reverse API). Instead of client polling, server notifies client.

**Q29: What is multipart/form-data?** Content type for file uploads. Allows mixing file data with form fields.

**Q30: What is Accept header?** Client tells server what format it wants: Accept: application/json.

**Q31: What is Content-Type header?** Tells receiver the format of the body: Content-Type: application/json.

**Q32: What is HTTPS?** HTTP + TLS encryption. All production APIs must use HTTPS.

**Q33: What is URL encoding?** Converting special characters to %XX format: "hello world" → "hello%20world".

**Q34: What is a query parameter?** Key-value pairs after `?` in URL: /users?age=25&role=admin.

**Q35: What is a path parameter?** Variable part of URL path: /users/{id} → /users/42.

**Q36: What is request body?** Data sent in the body of POST/PUT/PATCH requests, usually JSON.

**Q37: What is response header?** Metadata sent back by server: Content-Type, Cache-Control, Set-Cookie.

**Q38: What is 204 No Content?** Success with no response body. Used after DELETE.

**Q39: What is 409 Conflict?** Request conflicts with current state (duplicate email, version mismatch).

**Q40: What is 422 Unprocessable Entity?** Request is syntactically correct but semantically invalid (validation error).

**Q41: What is Bearer token?** Authentication token sent in Authorization header: "Bearer eyJhbG..."

**Q42: What is REST maturity model Level 3?** Full HATEOAS — responses include hyperlinks to available actions.

**Q43: What is API throttling?** Limiting request rate. Different from rate limiting in granularity.

**Q44: What is REST controller in Spring?** @RestController = @Controller + @ResponseBody. Returns JSON directly, no view resolution.

**Q45: What is @RequestMapping?** Maps HTTP requests to handler methods. @GetMapping, @PostMapping are shortcuts.

**Q46: What is @PathVariable?** Extracts value from URL path: @GetMapping("/users/{id}") → @PathVariable Long id.

**Q47: What is @RequestParam?** Extracts query parameter: /users?name=Dilip → @RequestParam String name.

**Q48: What is @RequestBody?** Binds JSON request body to Java object. Used with POST/PUT.

**Q49: What is ResponseEntity?** Spring class to return response with status code, headers, and body.

**Q50: What is @Valid?** Triggers bean validation on request body. Returns 400 if validation fails.

---

## 📚 References

- [REST API Tutorial](https://restfulapi.net/)
- [HTTP Status Codes](https://httpstatuses.com/)
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Spring REST Documentation](https://spring.io/guides/tutorials/rest/)
- [JSON Specification](https://www.json.org/)
- [Postman Learning Center](https://learning.postman.com/)

---

> **Previous Topic:** [← 08 - Servlets and JSP](../08-servlets-and-jsp/README.md)  
> **Next Topic:** [10 - ORM Tools →](../10-orm-tools/README.md)
