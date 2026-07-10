# 🏗️ Microservices — Complete In-Depth Guide

> **"Microservices is an architectural style where an application is built as a collection of small, independent services. Each service runs in its own process, communicates via APIs, and can be deployed independently."**

---

## 📑 Table of Contents

1. [Monolith vs Microservices](#1-monolith-vs-microservices)
2. [Microservices Architecture](#2-microservices-architecture)
3. [Spring Cloud Ecosystem](#3-spring-cloud-ecosystem)
4. [Service Discovery (Eureka)](#4-service-discovery-eureka)
5. [API Gateway (Spring Cloud Gateway)](#5-api-gateway-spring-cloud-gateway)
6. [Load Balancing](#6-load-balancing)
7. [Inter-Service Communication](#7-inter-service-communication)
8. [Circuit Breaker (Resilience4j)](#8-circuit-breaker-resilience4j)
9. [Distributed Configuration (Config Server)](#9-distributed-configuration-config-server)
10. [Distributed Tracing](#10-distributed-tracing)
11. [Event-Driven Architecture](#11-event-driven-architecture)
12. [Saga Pattern](#12-saga-pattern)
13. [Docker + Kubernetes Deployment](#13-docker--kubernetes-deployment)
14. [Design Patterns](#14-design-patterns)
15. [Best Practices](#15-best-practices)
16. [Interview Questions & Answers (50+)](#16-interview-questions--answers-50)

---

## 1. Monolith vs Microservices

```
MONOLITH:                              MICROSERVICES:
┌─────────────────────────┐           ┌──────┐ ┌──────┐ ┌──────┐
│        One Big App       │           │ User │ │Order │ │Payment│
│  ┌─────┐ ┌─────┐ ┌────┐│           │ Svc  │ │ Svc  │ │ Svc   │
│  │User │ │Order│ │Pay ││           └──┬───┘ └──┬───┘ └──┬────┘
│  │Mod  │ │Mod  │ │Mod ││              │        │        │
│  └─────┘ └─────┘ └────┘│           ┌──▼───┐ ┌──▼───┐ ┌──▼────┐
│  ┌─────────────────────┐│           │UserDB│ │OrdDB │ │PayDB  │
│  │   Single Database    ││           └──────┘ └──────┘ └───────┘
│  └─────────────────────┘│
└─────────────────────────┘           Each service:
                                       ✅ Own database
One deploy = everything               ✅ Own deployment
One failure = everything down          ✅ Own technology
One team = everyone                    ✅ Own team
```

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| Deployment | One unit | Independent services |
| Scaling | Scale everything | Scale specific service |
| Technology | One stack | Polyglot (mix Java, Python, etc.) |
| Team | One large team | Small, autonomous teams |
| Failure | Single point of failure | Isolated failures |
| Complexity | Simple to start | Operationally complex |
| Data | Shared database | Database per service |
| Best for | Small apps, startups | Large, complex systems |

---

## 2. Microservices Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        CLIENTS                               │
│              (Web, Mobile, Third-party)                      │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│                   API GATEWAY                                │
│              (Spring Cloud Gateway)                          │
│  Routing, Auth, Rate Limiting, Load Balancing                │
└──────────┬────────────┬────────────┬────────────────────────┘
           │            │            │
    ┌──────▼─────┐ ┌────▼─────┐ ┌───▼──────┐
    │ User Svc   │ │ Order Svc│ │Payment Svc│
    │ :8081      │ │ :8082    │ │ :8083     │
    │ ┌────────┐ │ │┌────────┐│ │┌────────┐│
    │ │User DB │ │ ││Order DB││ ││Pay DB  ││
    │ │(MySQL) │ │ ││(Postgres)│ ││(Mongo) ││
    │ └────────┘ │ │└────────┘│ │└────────┘│
    └──────┬─────┘ └────┬─────┘ └───┬──────┘
           │            │            │
    ┌──────▼────────────▼────────────▼──────┐
    │        Service Discovery (Eureka)      │
    │        Config Server                   │
    │        Message Broker (Kafka/RabbitMQ) │
    └───────────────────────────────────────┘
```

---

## 3. Spring Cloud Ecosystem

| Component | Purpose | Technology |
|-----------|---------|-----------|
| Service Discovery | Find service instances | Eureka, Consul |
| API Gateway | Single entry point, routing | Spring Cloud Gateway |
| Load Balancer | Distribute requests | Spring Cloud LoadBalancer |
| Circuit Breaker | Handle failures gracefully | Resilience4j |
| Config Server | Centralized configuration | Spring Cloud Config |
| Distributed Tracing | Track requests across services | Micrometer Tracing, Zipkin |
| Message Broker | Async communication | Kafka, RabbitMQ |

---

## 4. Service Discovery (Eureka)

### Eureka Server

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaServerApplication {
    public static void main(String[] args) {
        SpringApplication.run(EurekaServerApplication.class, args);
    }
}
```

```properties
# Eureka Server: application.properties
server.port=8761
eureka.client.register-with-eureka=false
eureka.client.fetch-registry=false
```

### Eureka Client (Each Microservice)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

```properties
# User Service: application.properties
spring.application.name=user-service
server.port=8081
eureka.client.service-url.defaultZone=http://localhost:8761/eureka/
```

```
How Service Discovery Works:

1. Service starts → registers with Eureka: "I'm user-service at 192.168.1.5:8081"
2. Eureka stores: {user-service: [192.168.1.5:8081, 192.168.1.6:8081]}
3. Order service needs user service → asks Eureka: "Where is user-service?"
4. Eureka responds: "192.168.1.5:8081"
5. Order service calls user service directly
6. Heartbeat every 30 seconds. No heartbeat → removed from registry.
```

---

## 5. API Gateway (Spring Cloud Gateway)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-gateway</artifactId>
</dependency>
```

```yaml
# Gateway application.yml
server:
  port: 8080

spring:
  application:
    name: api-gateway
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service        # lb = load balanced via Eureka
          predicates:
            - Path=/api/users/**
          filters:
            - StripPrefix=1

        - id: order-service
          uri: lb://order-service
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1

        - id: payment-service
          uri: lb://payment-service
          predicates:
            - Path=/api/payments/**
```

```
Client → Gateway:8080/api/users/1
Gateway routes to → user-service:8081/users/1

All services behind ONE URL! Client doesn't know individual service addresses.
```

### Gateway Filter (Authentication)

```java
@Component
public class AuthFilter implements GlobalFilter, Ordered {
    
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        String token = exchange.getRequest().getHeaders().getFirst("Authorization");
        
        if (token == null || !token.startsWith("Bearer ")) {
            exchange.getResponse().setStatusCode(HttpStatus.UNAUTHORIZED);
            return exchange.getResponse().setComplete();
        }
        
        // Validate JWT token
        // Add user info to headers for downstream services
        
        return chain.filter(exchange);
    }
    
    @Override
    public int getOrder() { return -1; }
}
```

---

## 6. Load Balancing

```java
// Spring Cloud LoadBalancer (client-side)
// Automatically distributes requests across instances

@Bean
@LoadBalanced  // Enable load balancing
public RestTemplate restTemplate() {
    return new RestTemplate();
}

// Now use service NAME instead of URL:
restTemplate.getForObject("http://user-service/users/1", User.class);
// Load balancer resolves "user-service" to actual instances via Eureka
// Round-robin by default
```

---

## 7. Inter-Service Communication

### Synchronous (REST — OpenFeign)

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

```java
@EnableFeignClients
@SpringBootApplication
public class OrderServiceApplication { }

// Declarative REST client (like JPA for REST!)
@FeignClient(name = "user-service")  // Service name in Eureka
public interface UserClient {
    
    @GetMapping("/users/{id}")
    UserDTO getUserById(@PathVariable Long id);
    
    @GetMapping("/users")
    List<UserDTO> getAllUsers();
}

// Usage — just inject and call!
@Service
@RequiredArgsConstructor
public class OrderService {
    
    private final UserClient userClient;  // Injected like any Spring bean
    
    public OrderDTO createOrder(CreateOrderDTO dto) {
        UserDTO user = userClient.getUserById(dto.getUserId());  // REST call!
        // Create order...
    }
}
```

### Asynchronous (Message Broker)

```
Sync (REST):              Async (Messaging):
A ──request──► B          A ──message──► Queue ──► B
A ◄──response── B         A doesn't wait!
A waits for B             B processes when ready

Use Sync when: You need immediate response
Use Async when: Fire-and-forget, long operations, decoupling
```

---

## 8. Circuit Breaker (Resilience4j)

```
The Problem:
  Order Service → User Service (DOWN!) → timeout... timeout... timeout...
  All Order Service threads blocked waiting → Order Service also goes DOWN!
  Cascading failure! 💥

The Solution: Circuit Breaker
  CLOSED:  Normal operation, requests go through
  OPEN:    Service is down, fail FAST (don't even try)
  HALF-OPEN: Try a few requests, if OK → CLOSED, if fail → OPEN

Timeline:
  ✅✅✅❌❌❌❌❌ → Circuit OPENS → ⚡fail fast⚡ → try one → ❌ → still OPEN
  ⚡⚡⚡⚡⚡ → try one → ✅ → Circuit CLOSES → ✅✅✅
```

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-circuitbreaker-resilience4j</artifactId>
</dependency>
```

```java
@Service
public class OrderService {
    
    private final UserClient userClient;
    
    @CircuitBreaker(name = "userService", fallbackMethod = "getUserFallback")
    @Retry(name = "userService", fallbackMethod = "getUserFallback")
    public UserDTO getUser(Long userId) {
        return userClient.getUserById(userId);
    }
    
    // Fallback — called when circuit is OPEN or retries exhausted
    private UserDTO getUserFallback(Long userId, Exception ex) {
        log.warn("User service unavailable, returning cached/default user for id={}", userId);
        return new UserDTO(userId, "Unknown User", "N/A");
    }
}
```

```yaml
# application.yml
resilience4j:
  circuitbreaker:
    instances:
      userService:
        sliding-window-size: 10
        failure-rate-threshold: 50        # Open if 50% of last 10 calls failed
        wait-duration-in-open-state: 10s  # Wait 10s before trying again
        permitted-number-of-calls-in-half-open-state: 3
  retry:
    instances:
      userService:
        max-attempts: 3
        wait-duration: 1s
```

---

## 9. Distributed Configuration (Config Server)

```
Problem: 10 microservices × 3 environments (dev, staging, prod) = 30 config files!
Solution: ONE Config Server reading from Git repository.

Config Server ← reads ← Git Repo (application.yml files)
     ↑
All services fetch config from Config Server at startup
```

```java
// Config Server
@SpringBootApplication
@EnableConfigServer
public class ConfigServerApplication { }
```

```yaml
# Config Server application.yml
spring:
  cloud:
    config:
      server:
        git:
          uri: https://github.com/myorg/config-repo
```

---

## 10. Distributed Tracing

```
Request: GET /api/orders/42

Gateway → Order Service → User Service → Payment Service
  ↓           ↓               ↓              ↓
  TraceID: abc-123 (SAME across all services!)
  SpanID:  span-1  span-2    span-3        span-4

Zipkin UI shows the entire request journey:
  [Gateway: 5ms] → [Order: 15ms] → [User: 8ms]
                                  → [Payment: 20ms]
  Total: 48ms
```

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>
<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-brave</artifactId>
</dependency>
```

---

## 11. Event-Driven Architecture

```java
// Order Service publishes event
@Service
public class OrderService {
    
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;
    
    public Order createOrder(CreateOrderDTO dto) {
        Order order = orderRepository.save(mapToOrder(dto));
        
        // Publish event — other services react!
        kafkaTemplate.send("order-events", new OrderEvent("ORDER_CREATED", order.getId()));
        
        return order;
    }
}

// Payment Service listens for event
@Service
public class PaymentListener {
    
    @KafkaListener(topics = "order-events", groupId = "payment-service")
    public void handleOrderEvent(OrderEvent event) {
        if ("ORDER_CREATED".equals(event.getType())) {
            paymentService.processPayment(event.getOrderId());
        }
    }
}

// Notification Service also listens (independently!)
@KafkaListener(topics = "order-events", groupId = "notification-service")
public void handleOrderEvent(OrderEvent event) {
    notificationService.sendOrderConfirmation(event.getOrderId());
}
```

---

## 12. Saga Pattern

```
Problem: Distributed transaction across multiple services.
  Order → Payment → Inventory → Shipping
  If Payment fails, how to undo the Order?

Saga = Sequence of local transactions with compensating actions.

CHOREOGRAPHY (event-based):
  Order Created → Payment Charged → Inventory Reserved → Shipping Started
  If Inventory fails → Payment Refunded → Order Cancelled

ORCHESTRATION (central coordinator):
  Saga Orchestrator tells each service what to do and handles failures.
```

---

## 13. Docker + Kubernetes Deployment

```yaml
# docker-compose.yml for microservices
version: '3.8'
services:
  eureka:
    build: ./eureka-server
    ports: ["8761:8761"]

  gateway:
    build: ./api-gateway
    ports: ["8080:8080"]
    depends_on: [eureka]
    environment:
      EUREKA_CLIENT_SERVICE_URL_DEFAULTZONE: http://eureka:8761/eureka/

  user-service:
    build: ./user-service
    depends_on: [eureka, user-db]
    environment:
      EUREKA_CLIENT_SERVICE_URL_DEFAULTZONE: http://eureka:8761/eureka/
      SPRING_DATASOURCE_URL: jdbc:mysql://user-db:3306/userdb

  user-db:
    image: mysql:8
    environment:
      MYSQL_DATABASE: userdb
      MYSQL_ROOT_PASSWORD: password
```

---

## 14. Design Patterns

| Pattern | Problem | Solution |
|---------|---------|----------|
| **API Gateway** | Multiple entry points | Single entry, routing |
| **Service Discovery** | Service locations change | Dynamic registry |
| **Circuit Breaker** | Cascading failures | Fail fast, fallback |
| **Saga** | Distributed transactions | Compensating transactions |
| **CQRS** | Complex read/write | Separate read/write models |
| **Event Sourcing** | Audit trail | Store events, not state |
| **Strangler Fig** | Migrate monolith | Gradually replace modules |
| **Sidecar** | Cross-cutting concerns | Proxy container alongside service |
| **BFF** | Multiple clients | Backend per frontend |

---

## 15. Best Practices

1. **One service = one responsibility** — Don't make micro-monoliths
2. **Database per service** — No shared databases
3. **API versioning** — /api/v1/users
4. **Use async where possible** — Kafka/RabbitMQ for decoupling
5. **Implement circuit breakers** — Prevent cascading failures
6. **Centralize logging** — ELK stack, correlation IDs
7. **Distributed tracing** — Zipkin/Jaeger
8. **Health checks** — /actuator/health on every service
9. **Contract testing** — Pact for API contracts between services
10. **Start monolith, extract later** — Don't start with microservices

---

## 16. Interview Questions & Answers (50+)

### Beginner

**Q1: What are microservices?** Architectural style where app = collection of small, independent services. Each has its own DB and deployment.

**Q2: Monolith vs Microservices?** Monolith: one deployable unit, simpler. Microservices: independent services, complex but scalable.

**Q3: What is Service Discovery?** Mechanism for services to find each other dynamically. Eureka, Consul.

**Q4: What is API Gateway?** Single entry point for all clients. Handles routing, auth, rate limiting. Spring Cloud Gateway.

**Q5: What is a Circuit Breaker?** Prevents cascading failures. If service is down, fail fast instead of waiting.

**Q6: What is Spring Cloud?** Collection of tools for building microservices: Eureka, Gateway, Config, Circuit Breaker.

**Q7: What is inter-service communication?** Sync (REST/gRPC) or Async (Kafka/RabbitMQ) communication between services.

**Q8: What is database per service?** Each microservice owns its database. No sharing.

---

### Intermediate

**Q9: What is the Saga pattern?** Manages distributed transactions via sequence of local transactions with compensating actions.

**Q10: What is CQRS?** Command Query Responsibility Segregation. Separate models for reads and writes.

**Q11: What is Event Sourcing?** Store state changes as events, not current state. Replay events to rebuild state.

**Q12: What is OpenFeign?** Declarative REST client. Define interface with annotations, Spring generates implementation.

**Q13: What is distributed tracing?** Tracking a request across multiple services. TraceID links all spans.

**Q14: What is centralized configuration?** One Config Server manages configs for all services. Git-backed.

**Q15: What is the Strangler Fig pattern?** Gradually replace monolith by routing requests to new microservices one by one.

---

### Rapid-Fire (Q16–Q50)

**Q16: Eureka heartbeat interval?** 30 seconds by default.

**Q17: What is client-side load balancing?** Client chooses which instance to call (Spring Cloud LoadBalancer). No external LB.

**Q18: What is @LoadBalanced?** Enables RestTemplate to resolve service names via Eureka + load balance.

**Q19: What is @FeignClient?** Declarative REST client annotation. Specify service name, Spring generates HTTP calls.

**Q20: What is Resilience4j?** Fault tolerance library: circuit breaker, retry, rate limiter, bulkhead.

**Q21: Circuit breaker states?** CLOSED (normal), OPEN (failing fast), HALF-OPEN (testing recovery).

**Q22: What is a fallback method?** Alternative logic when primary call fails (circuit breaker/retry exhausted).

**Q23: What is @Retry?** Automatically retries failed calls with configurable attempts and delay.

**Q24: What is @RateLimiter?** Limits number of calls in a time period. Prevents overload.

**Q25: What is @Bulkhead?** Limits concurrent calls to a service. Prevents resource exhaustion.

**Q26: What is Spring Cloud Config?** Centralized external configuration server backed by Git.

**Q27: What is Spring Cloud Bus?** Broadcasts config changes to all services via message broker.

**Q28: What is Zipkin?** Distributed tracing system. Visualizes request flow across services.

**Q29: What is correlation ID?** Unique ID propagated across services for request tracking in logs.

**Q30: What is gRPC?** High-performance RPC framework. Binary protocol, faster than REST.

**Q31: REST vs gRPC for microservices?** REST: simple, HTTP/JSON. gRPC: fast, binary, streaming. gRPC for internal, REST for external.

**Q32: What is service mesh?** Infrastructure layer handling service-to-service communication. Istio, Linkerd.

**Q33: What is a sidecar proxy?** Proxy running alongside each service handling networking concerns.

**Q34: What is BFF pattern?** Backend for Frontend. Separate API layer per client type (web, mobile).

**Q35: What is DDD?** Domain-Driven Design. Bounded contexts align well with microservice boundaries.

**Q36: What is a bounded context?** DDD concept — logical boundary around a domain model. Maps to microservice.

**Q37: What is eventual consistency?** Data will be consistent eventually, not immediately. Common in distributed systems.

**Q38: What is CAP theorem?** Distributed system can guarantee only 2 of 3: Consistency, Availability, Partition tolerance.

**Q39: What is idempotency?** Same request produces same result regardless of how many times it's called.

**Q40: What is API composition?** API Gateway combines responses from multiple services into one response.

**Q41: What is blue-green deployment?** Two identical environments. Switch traffic between them for zero-downtime deployment.

**Q42: What is canary deployment?** Route small percentage of traffic to new version, gradually increase.

**Q43: What is contract testing?** Verify API contract between consumer and provider. Pact framework.

**Q44: What is consumer-driven contract?** Consumer defines expected API, provider verifies it meets expectations.

**Q45: How many services is too many?** Depends on team size. Rule: 2-pizza team per service. Don't over-decompose.

**Q46: What is a death star architecture?** Too many inter-service dependencies. Services tightly coupled = worse than monolith.

**Q47: What is choreography vs orchestration?** Choreography: services react to events. Orchestration: central coordinator directs services.

**Q48: What is the outbox pattern?** Ensure database write + event publish are atomic. Write event to outbox table, publish later.

**Q49: What is health check?** Endpoint reporting service status. /actuator/health. Used by discovery, load balancer.

**Q50: When NOT to use microservices?** Small teams, simple domains, startups, when you don't need independent scaling.

---

## 📚 References

- [Spring Cloud Documentation](https://spring.io/projects/spring-cloud)
- [Microservices.io Patterns](https://microservices.io/patterns/)
- [Building Microservices (Sam Newman)](https://samnewman.io/books/building_microservices_2nd_edition/)
- [Spring Cloud Netflix](https://spring.io/projects/spring-cloud-netflix)

---

> **Previous Topic:** [← 25 - DeepSeek + Ollama](../25-deepseek-ollama-spring/README.md)  
> **Next Topic:** [27 - Kafka →](../27-springboot-kafka/README.md)
