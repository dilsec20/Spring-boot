# ⚡ Spring WebFlux (Reactive Programming) — Complete In-Depth Guide

> **"Spring WebFlux is a non-blocking, reactive web framework. While Spring MVC blocks a thread per request, WebFlux uses an event loop, allowing you to handle massive concurrency with a tiny number of threads."**

---

## 📑 Table of Contents

1. [What is Reactive Programming?](#1-what-is-reactive-programming)
2. [Spring MVC vs Spring WebFlux](#2-spring-mvc-vs-spring-webflux)
3. [Project Reactor (Mono & Flux)](#3-project-reactor-mono--flux)
4. [Setup & Configuration](#4-setup--configuration)
5. [Annotation-based Controllers](#5-annotation-based-controllers)
6. [Functional Endpoints (Router/Handler)](#6-functional-endpoints-routerhandler)
7. [WebClient (Non-blocking HTTP Client)](#7-webclient-non-blocking-http-client)
8. [R2DBC (Reactive Databases)](#8-r2dbc-reactive-databases)
9. [Error Handling](#9-error-handling)
10. [Testing WebFlux (WebTestClient)](#10-testing-webflux-webtestclient)
11. [When NOT to use WebFlux](#11-when-not-to-use-webflux)
12. [Interview Questions & Answers (50+)](#12-interview-questions--answers-50)

---

## 1. What is Reactive Programming?

Reactive programming is an asynchronous programming paradigm focused on **data streams** and the **propagation of change**. 

It adheres to the **Reactive Manifesto**:
*   **Responsive**: Responds in a timely manner.
*   **Resilient**: Stays responsive in the face of failure.
*   **Elastic**: Stays responsive under varying workload.
*   **Message Driven**: Relies on asynchronous message-passing (Backpressure).

**Backpressure:** 
If a database produces 10,000 records/sec but the client can only process 10 records/sec, the client can tell the database: *"Slow down! Give me 10 at a time."* This prevents OutOfMemory errors.

---

## 2. Spring MVC vs Spring WebFlux

| Feature | Spring MVC | Spring WebFlux |
|---------|------------|----------------|
| **Architecture** | Thread-per-request (Synchronous) | Event Loop (Asynchronous) |
| **Server** | Tomcat, Jetty | Netty, Undertow |
| **I/O Model** | Blocking I/O | Non-blocking I/O |
| **Concurrency** | Limited by Thread Pool Size (e.g., 200) | Scales massively with few threads |
| **Data Types** | `Object`, `List<T>` | `Mono<T>`, `Flux<T>` |
| **Database** | JDBC / JPA (Blocking) | R2DBC / Spring Data Reactive |
| **Best For** | CPU-bound tasks, simple CRUD | High concurrency, streaming, microservices |

```
[ Spring MVC (Tomcat) ]
Request 1 → Thread 1 → Wait for DB (Blocked 🛑) → Return
Request 2 → Thread 2 → Wait for DB (Blocked 🛑) → Return
*If 200 requests hit at once, 200 threads are blocked. Request 201 fails.*

[ Spring WebFlux (Netty) ]
Request 1 → Event Loop → Fire DB Query → Event Loop moves to next task 🚀
Request 2 → Event Loop → Fire DB Query → Event Loop moves to next task 🚀
... DB finishes query 1 ... → Event Loop picks it up and returns response.
*1 Event Loop thread can handle 10,000+ concurrent connections!*
```

---

## 3. Project Reactor (Mono & Flux)

Spring WebFlux is built on **Project Reactor**, which implements the Reactive Streams specification.

*   `Mono<T>`: Emits **0 or 1** element. (e.g., fetching a single User).
*   `Flux<T>`: Emits **0 to N** elements. (e.g., fetching a List of Users).

```java
// ═══ Mono ═══
Mono<String> mono = Mono.just("Hello WebFlux");
Mono<String> emptyMono = Mono.empty();
Mono<String> errorMono = Mono.error(new RuntimeException("Oops"));

// ═══ Flux ═══
Flux<String> flux = Flux.just("Apple", "Banana", "Orange");
Flux<Integer> range = Flux.range(1, 10);
Flux<Long> interval = Flux.interval(Duration.ofSeconds(1)); // Streams infinitely

// ═══ Operators (Nothing happens until you SUBSCRIBE!) ═══
flux.map(String::toUpperCase)
    .filter(s -> s.startsWith("A"))
    .subscribe(System.out::println); // Prints: APPLE
```

---

## 4. Setup & Configuration

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-webflux</artifactId>
</dependency>
<!-- NEVER include spring-boot-starter-web! They conflict. -->
```

---

## 5. Annotation-based Controllers

The syntax is identical to Spring MVC, but the return types change!

```java
@RestController
@RequestMapping("/api/users")
@RequiredArgsConstructor
public class UserController {
    
    private final UserService userService;
    
    // Returns 0 or 1 item
    @GetMapping("/{id}")
    public Mono<User> getUser(@PathVariable String id) {
        return userService.findById(id);
    }
    
    // Returns multiple items
    @GetMapping
    public Flux<User> getAllUsers() {
        return userService.findAll();
    }
    
    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public Mono<User> createUser(@RequestBody Mono<User> userMono) {
        // We accept a Mono<User> because parsing the JSON is also non-blocking!
        return userMono.flatMap(userService::save);
    }
    
    // Server-Sent Events (Streaming data to browser!)
    @GetMapping(value = "/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
    public Flux<Long> streamData() {
        return Flux.interval(Duration.ofSeconds(1));
    }
}
```

---

## 6. Functional Endpoints (Router/Handler)

A modern, functional alternative to `@RestController`. Highly preferred in Kotlin, but works great in Java too.

**1. Handler (The Logic)**
```java
@Component
@RequiredArgsConstructor
public class UserHandler {
    
    private final UserService userService;
    
    public Mono<ServerResponse> getUser(ServerRequest request) {
        String id = request.pathVariable("id");
        return userService.findById(id)
            .flatMap(user -> ServerResponse.ok().bodyValue(user))
            .switchIfEmpty(ServerResponse.notFound().build());
    }
    
    public Mono<ServerResponse> getAllUsers(ServerRequest request) {
        return ServerResponse.ok().body(userService.findAll(), User.class);
    }
}
```

**2. Router (The URLs)**
```java
@Configuration
public class UserRouter {
    
    @Bean
    public RouterFunction<ServerResponse> route(UserHandler handler) {
        return RouterFunctions
            .route(GET("/api/v2/users/{id}"), handler::getUser)
            .andRoute(GET("/api/v2/users"), handler::getAllUsers);
    }
}
```

---

## 7. WebClient (Non-blocking HTTP Client)

`RestTemplate` is blocking and is in maintenance mode. `WebClient` is the modern, reactive way to make HTTP calls.

```java
@Service
public class ExternalApiService {
    
    private final WebClient webClient;
    
    public ExternalApiService(WebClient.Builder webClientBuilder) {
        this.webClient = webClientBuilder.baseUrl("https://api.github.com").build();
    }
    
    public Mono<String> getUserInfo(String username) {
        return webClient.get()
            .uri("/users/{username}", username)
            .retrieve()
            .bodyToMono(String.class); // Non-blocking network call!
    }
}
```

---

## 8. R2DBC (Reactive Databases)

JPA/JDBC are BLOCKING. If you use JPA in WebFlux, you ruin the entire non-blocking architecture. You must use **R2DBC (Reactive Relational Database Connectivity)** or reactive NoSQL drivers (Mongo, Redis, Cassandra).

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-r2dbc</artifactId>
</dependency>
<dependency>
    <groupId>io.r2dbc</groupId>
    <artifactId>r2dbc-postgresql</artifactId>
</dependency>
```

```java
// Reactive Repository!
public interface UserRepository extends ReactiveCrudRepository<User, Long> {
    
    // Returns Mono instead of Optional
    Mono<User> findByEmail(String email);
    
    // Returns Flux instead of List
    Flux<User> findByStatus(String status);
}
```

---

## 9. Error Handling

Reactive error handling uses functional operators (`onErrorResume`, `onErrorReturn`).

```java
public Mono<ServerResponse> getUserData(String id) {
    return userService.findById(id)
        .flatMap(user -> ServerResponse.ok().bodyValue(user))
        
        // 1. Return default value on error
        .onErrorReturn(CustomException.class, ServerResponse.badRequest().build())
        
        // 2. Execute alternative logic on error
        .onErrorResume(DatabaseException.class, ex -> {
            log.error("DB failed", ex);
            return ServerResponse.status(503).build();
        })
        
        // 3. 404 if empty (Mono completes without emitting)
        .switchIfEmpty(ServerResponse.notFound().build());
}
```

---

## 10. Testing WebFlux (WebTestClient)

`WebTestClient` is designed to test reactive endpoints non-blockingly.

```java
@WebFluxTest(UserController.class)
class UserControllerTest {

    @Autowired
    private WebTestClient webTestClient;

    @MockBean
    private UserService userService;

    @Test
    void shouldReturnUser() {
        User mockUser = new User("1", "John Doe");
        when(userService.findById("1")).thenReturn(Mono.just(mockUser));

        webTestClient.get().uri("/api/users/1")
            .exchange()
            .expectStatus().isOk()
            .expectBody()
            .jsonPath("$.name").isEqualTo("John Doe");
    }
}
```

---

## 11. When NOT to use WebFlux

**DO NOT use WebFlux if:**
1.  Your application heavily relies on blocking APIs (JDBC, JPA, older libraries).
2.  Your app is CPU-bound (heavy calculations, image processing). WebFlux event loops will get blocked, causing the whole app to freeze!
3.  Your team is entirely new to reactive programming and your scale is small (Spring MVC is much easier to debug and learn).

---

## 12. Interview Questions & Answers (50+)

### Beginner

**Q1: What is Spring WebFlux?** A non-blocking, reactive web framework introduced in Spring 5, built on Project Reactor and Netty.

**Q2: Spring MVC vs WebFlux?** MVC: blocking, thread-per-request, Tomcat. WebFlux: non-blocking, event-loop, Netty.

**Q3: What is Project Reactor?** A reactive library based on the Reactive Streams specification, providing `Mono` and `Flux`.

**Q4: What is a Mono?** A reactive stream that emits 0 or 1 item, then completes.

**Q5: What is a Flux?** A reactive stream that emits 0 to N items, then completes.

**Q6: What is Backpressure?** A mechanism allowing a consumer to tell a producer to slow down the data stream to avoid being overwhelmed.

**Q7: Can I use Spring Data JPA with WebFlux?** Technically yes, but you SHOULD NOT. JPA is blocking. It will block the event loop and ruin the reactive architecture. Use R2DBC instead.

**Q8: What is WebClient?** The reactive, non-blocking alternative to `RestTemplate` for making HTTP requests.

---

### Intermediate

**Q9: What happens if you run a blocking thread (like `Thread.sleep`) in WebFlux?** The event loop thread blocks. Because there are very few event loop threads (often 1 per CPU core), the entire application will hang and stop accepting new requests.

**Q10: How do you handle blocking calls if you absolutely must make one in WebFlux?** Wrap the blocking call in `Mono.fromCallable()` and run it on a separate dedicated thread pool using `.subscribeOn(Schedulers.boundedElastic())`.

**Q11: What is R2DBC?** Reactive Relational Database Connectivity. An asynchronous, non-blocking API for SQL databases.

**Q12: Functional Endpoints vs Annotated Controllers?** Annotated (`@RestController`) uses reflection and is familiar to MVC devs. Functional (`RouterFunction`) is programmatic, type-safe, and slightly faster to boot.

**Q13: What does `subscribe()` do?** Nothing happens in a reactive stream until you subscribe. It triggers the execution of the pipeline. In WebFlux controllers, Spring framework handles the subscription for you.

**Q14: `map()` vs `flatMap()` in Reactor?** `map` transforms a value synchronously. `flatMap` transforms a value asynchronously by returning another Publisher (`Mono`/`Flux`) and merging them.

**Q15: How do you handle errors in WebFlux?** Using operators like `onErrorReturn`, `onErrorResume`, or `doOnError`.

---

### Rapid-Fire (Q16–Q50)

**Q16: Default server for WebFlux?** Netty.

**Q17: Can WebFlux run on Tomcat?** Yes, if configured to use Servlet 3.1+ non-blocking I/O, but Netty is the default and preferred.

**Q18: What is the Reactive Streams specification?** A standard for asynchronous stream processing with non-blocking backpressure (Publisher, Subscriber, Subscription, Processor).

**Q19: What is a Publisher?** Emits a sequence of events to Subscribers. `Mono` and `Flux` implement Publisher.

**Q20: What is a Subscriber?** Consumes events from a Publisher.

**Q21: What is a Subscription?** Represents the 1-to-1 lifecycle of a Subscriber subscribing to a Publisher. Controls backpressure (`request(n)`).

**Q22: `Mono.empty()`?** Creates a Mono that completes immediately without emitting any value.

**Q23: `Mono.justOrEmpty()`?** Wraps an Optional or a potentially null value.

**Q24: `Mono.defer()`?** Delays the execution of the Mono creation until subscription happens.

**Q25: What is `zip()`?** Combines multiple Monos/Fluxes together, waiting for all of them to emit, then combining their results.

**Q26: What is `concat()`?** Executes Publishers sequentially.

**Q27: What is `merge()`?** Executes Publishers concurrently, interleaving their emissions.

**Q28: What is Server-Sent Events (SSE)?** Pushing real-time updates from server to client over a single HTTP connection. (`produces = TEXT_EVENT_STREAM_VALUE`).

**Q29: How to test WebFlux controllers?** Use `WebTestClient`.

**Q30: What is `StepVerifier`?** A testing utility in Project Reactor used to assert the sequence of events emitted by a Mono or Flux.

**Q31: Can I use Spring Security with WebFlux?** Yes, use `@EnableWebFluxSecurity` and configure a `SecurityWebFilterChain`.

**Q32: How do filters work in WebFlux?** Use `WebFilter`, which operates on `ServerWebExchange` and returns a `Mono<Void>`.

**Q33: What is `ServerWebExchange`?** The reactive equivalent of `HttpServletRequest` + `HttpServletResponse`.

**Q34: How to extract path variables in Functional routing?** `request.pathVariable("id")`.

**Q35: How to read body in Functional routing?** `request.bodyToMono(User.class)`.

**Q36: What is `switchIfEmpty()`?** Provides an alternative Publisher if the original Publisher completes without emitting any items (e.g., returning a 404).

**Q37: What is a hot vs cold publisher?** Cold: starts generating data anew for each subscriber (e.g., an HTTP call). Hot: emits data regardless of subscribers, subscribers share the stream (e.g., mouse clicks).

**Q38: Is `WebClient` thread-safe?** Yes, it is immutable and thread-safe.

**Q39: What is `exchangeToMono` in WebClient?** Gives you access to the raw `ClientResponse` to check status codes before parsing the body.

**Q40: `Schedulers.parallel()`?** Thread pool for CPU-bound work.

**Q41: `Schedulers.boundedElastic()`?** Thread pool for blocking I/O tasks. Grows as needed but has a cap.

**Q42: `Schedulers.immediate()`?** Runs on the current thread.

**Q43: How does Netty handle requests?** Uses an Event Loop Group (boss group accepts connections, worker group handles I/O).

**Q44: Difference between CompletableFuture and Mono?** Both represent async results, but Mono supports lazy execution, rich operators, and backpressure.

**Q45: Why is debugging WebFlux hard?** Stack traces are asynchronous and span across multiple threads.

**Q46: How to improve WebFlux debugging?** Enable Reactor debug mode: `Hooks.onOperatorDebug()` or use `checkpoint()`.

**Q47: Can I return `ResponseEntity` in WebFlux?** Yes, `Mono<ResponseEntity<T>>`.

**Q48: Does WebFlux support WebSockets?** Yes, native support via `WebSocketHandler`.

**Q49: When does a reactive stream execute?** At the moment of subscription.

**Q50: Are there Virtual Threads (Project Loom) in Spring Boot?** Yes (Boot 3.2+). Virtual threads allow you to write blocking code (MVC) that scales like WebFlux. This is reducing the need for WebFlux for simple CRUD apps!

---

## 📚 References

- [Spring WebFlux Official Docs](https://docs.spring.io/spring-framework/reference/web/webflux.html)
- [Project Reactor](https://projectreactor.io/)
- [R2DBC](https://r2dbc.io/)

---

> **Previous Topic:** [← 31 - Terraform](../31-terraform/README.md)  
> **Next Topic:** [33 - Spring for GraphQL →](../33-spring-graphql/README.md)
