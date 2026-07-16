# 🏗️ System Design — Spring Boot Implementations for 15-25 LPA Interviews

> **"Theory gets you past the first round. Implementation gets you the offer."**
> Every concept below includes working Spring Boot code you can run and explain in an interview.

---

## 📑 Table of Contents
1. [Rate Limiter (Token Bucket + Sliding Window)](#1-rate-limiter)
2. [Distributed ID Generator (Snowflake)](#2-snowflake-id-generator)
3. [Redis — Distributed Lock, Cache, Pub/Sub, Leaderboard](#3-redis-patterns)
4. [RabbitMQ — Message Queue Patterns](#4-rabbitmq)
5. [Circuit Breaker (Resilience4j)](#5-circuit-breaker)
6. [Database Indexing (B-Tree, Composite, Covering)](#6-database-indexing)
7. [Sharding & Partitioning](#7-sharding--partitioning)
8. [Consistent Hashing](#8-consistent-hashing)
9. [API Gateway & Load Balancing](#9-api-gateway--load-balancing)
10. [URL Shortener — Full Implementation](#10-url-shortener)
11. [Notification Service (Fan-Out Pattern)](#11-notification-service)
12. [Event Sourcing & CQRS](#12-event-sourcing--cqrs)
13. [Idempotency (Preventing Duplicate Requests)](#13-idempotency)
14. [Distributed Transactions (Saga Pattern)](#14-saga-pattern)
15. [Top 20 Interview Questions with Answers](#15-interview-questions)

---

## 1. Rate Limiter

### Why?
Prevent abuse. Protect APIs from DDoS. Enforce fair usage (e.g., 100 requests/minute per user).

### Algorithm 1: Token Bucket (Simple, In-Memory)

```
How it works:
  - Bucket holds MAX tokens (e.g., 10)
  - Tokens are added at a fixed rate (e.g., 1 per second)
  - Each request consumes 1 token
  - If bucket is empty → REJECT (429 Too Many Requests)

  [████████░░]  ← 8 tokens left
  Request → consume 1 → [███████░░░]
  ...
  [░░░░░░░░░░]  ← 0 tokens → REJECTED!
```

```java
// ═══════════════════════════════════════════════
// Token Bucket Rate Limiter (In-Memory)
// ═══════════════════════════════════════════════
@Component
public class TokenBucketRateLimiter {
    
    private final ConcurrentHashMap<String, Bucket> buckets = new ConcurrentHashMap<>();
    
    private static class Bucket {
        double tokens;
        long lastRefillTimestamp;
        final double maxTokens;
        final double refillRate; // tokens per second
        
        Bucket(double maxTokens, double refillRate) {
            this.tokens = maxTokens;
            this.maxTokens = maxTokens;
            this.refillRate = refillRate;
            this.lastRefillTimestamp = System.nanoTime();
        }
    }
    
    public synchronized boolean tryConsume(String clientId) {
        Bucket bucket = buckets.computeIfAbsent(clientId, 
            k -> new Bucket(10, 1)); // 10 max tokens, 1 token/sec refill
        
        long now = System.nanoTime();
        double elapsed = (now - bucket.lastRefillTimestamp) / 1_000_000_000.0;
        
        // Refill tokens
        bucket.tokens = Math.min(bucket.maxTokens, bucket.tokens + elapsed * bucket.refillRate);
        bucket.lastRefillTimestamp = now;
        
        if (bucket.tokens >= 1) {
            bucket.tokens -= 1;
            return true;  // ✅ ALLOWED
        }
        return false;     // ❌ RATE LIMITED
    }
}
```

### Algorithm 2: Sliding Window Counter (Redis-based, Distributed)

```java
// ═══════════════════════════════════════════════
// Sliding Window Rate Limiter (Redis-based)
// ═══════════════════════════════════════════════
@Component
public class RedisSlidingWindowRateLimiter {
    
    @Autowired
    private StringRedisTemplate redisTemplate;
    
    /**
     * @param key     Unique key (e.g., "rate_limit:user:123")
     * @param limit   Max requests allowed
     * @param windowSeconds  Window duration in seconds
     */
    public boolean isAllowed(String key, int limit, int windowSeconds) {
        long now = System.currentTimeMillis();
        long windowStart = now - (windowSeconds * 1000L);
        
        String redisKey = "rate_limit:" + key;
        
        // Use Redis Sorted Set (ZSET)
        // Score = timestamp, Value = unique request ID
        redisTemplate.opsForZSet().removeRangeByScore(redisKey, 0, windowStart); // Remove expired
        Long count = redisTemplate.opsForZSet().zCard(redisKey);                // Count remaining
        
        if (count != null && count >= limit) {
            return false; // ❌ RATE LIMITED
        }
        
        // Add current request
        redisTemplate.opsForZSet().add(redisKey, now + ":" + UUID.randomUUID(), now);
        redisTemplate.expire(redisKey, Duration.ofSeconds(windowSeconds + 1));
        
        return true; // ✅ ALLOWED
    }
}
```

### Rate Limiter Filter (Apply to ALL endpoints)

```java
@Component
@Order(1)
public class RateLimitFilter implements Filter {
    
    @Autowired
    private TokenBucketRateLimiter rateLimiter;
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
            throws IOException, ServletException {
        
        HttpServletRequest req = (HttpServletRequest) request;
        String clientId = req.getRemoteAddr(); // Or extract from JWT/API key
        
        if (!rateLimiter.tryConsume(clientId)) {
            HttpServletResponse res = (HttpServletResponse) response;
            res.setStatus(429); // Too Many Requests
            res.getWriter().write("{\"error\": \"Rate limit exceeded. Try again later.\"}");
            return;
        }
        
        chain.doFilter(request, response);
    }
}
```

### Rate Limiter via Annotation (Custom AOP)

```java
// Custom annotation
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RateLimit {
    int requests() default 10;    // Max requests
    int seconds() default 60;     // Per time window
}

// AOP Aspect
@Aspect
@Component
public class RateLimitAspect {
    
    @Autowired
    private RedisSlidingWindowRateLimiter rateLimiter;
    
    @Around("@annotation(rateLimit)")
    public Object enforce(ProceedingJoinPoint joinPoint, RateLimit rateLimit) throws Throwable {
        // Extract user ID from SecurityContext
        String userId = SecurityContextHolder.getContext().getAuthentication().getName();
        String key = joinPoint.getSignature().getName() + ":" + userId;
        
        if (!rateLimiter.isAllowed(key, rateLimit.requests(), rateLimit.seconds())) {
            throw new ResponseStatusException(HttpStatus.TOO_MANY_REQUESTS, "Rate limit exceeded");
        }
        
        return joinPoint.proceed();
    }
}

// Usage
@RestController
public class ApiController {
    
    @RateLimit(requests = 5, seconds = 60)  // 5 requests per minute
    @GetMapping("/api/data")
    public String getData() {
        return "Here's your data!";
    }
}
```

---

## 2. Snowflake ID Generator

### Why?
Auto-increment IDs don't work in distributed systems (two servers could generate same ID). UUID is too long (128 bits, not sortable). **Snowflake** generates unique, sortable, 64-bit IDs.

```
Snowflake ID structure (64 bits):
┌─────────────────────────────────────────────────────────────┐
│  0  │   41 bits timestamp   │ 10 bits machine │ 12 bits seq │
│sign │  (69 years of IDs)    │   (1024 nodes)  │  (4096/ms)  │
└─────────────────────────────────────────────────────────────┘

- Timestamp: milliseconds since custom epoch → IDs are TIME-SORTABLE!
- Machine ID: identifies which server generated it → NO COLLISIONS!
- Sequence: counter within same millisecond → 4096 IDs/ms per machine
```

### Spring Boot Implementation

```java
@Component
public class SnowflakeIdGenerator {
    
    // Custom epoch (e.g., 2024-01-01 00:00:00 UTC)
    private static final long EPOCH = 1704067200000L;
    
    // Bit allocation
    private static final long MACHINE_ID_BITS = 10;
    private static final long SEQUENCE_BITS = 12;
    
    // Max values
    private static final long MAX_MACHINE_ID = (1L << MACHINE_ID_BITS) - 1;  // 1023
    private static final long MAX_SEQUENCE = (1L << SEQUENCE_BITS) - 1;      // 4095
    
    // Bit shifts
    private static final long MACHINE_ID_SHIFT = SEQUENCE_BITS;              // 12
    private static final long TIMESTAMP_SHIFT = SEQUENCE_BITS + MACHINE_ID_BITS; // 22
    
    private final long machineId;
    private long sequence = 0L;
    private long lastTimestamp = -1L;
    
    public SnowflakeIdGenerator(@Value("${snowflake.machine-id:1}") long machineId) {
        if (machineId < 0 || machineId > MAX_MACHINE_ID) {
            throw new IllegalArgumentException("Machine ID must be between 0 and " + MAX_MACHINE_ID);
        }
        this.machineId = machineId;
    }
    
    public synchronized long nextId() {
        long timestamp = System.currentTimeMillis();
        
        if (timestamp < lastTimestamp) {
            throw new RuntimeException("Clock moved backwards!");
        }
        
        if (timestamp == lastTimestamp) {
            // Same millisecond → increment sequence
            sequence = (sequence + 1) & MAX_SEQUENCE;
            if (sequence == 0) {
                // Sequence exhausted → wait for next millisecond
                timestamp = waitNextMillis(lastTimestamp);
            }
        } else {
            sequence = 0; // New millisecond → reset sequence
        }
        
        lastTimestamp = timestamp;
        
        // Compose the ID
        return ((timestamp - EPOCH) << TIMESTAMP_SHIFT)
             | (machineId << MACHINE_ID_SHIFT)
             | sequence;
    }
    
    private long waitNextMillis(long lastTs) {
        long ts = System.currentTimeMillis();
        while (ts <= lastTs) {
            ts = System.currentTimeMillis();
        }
        return ts;
    }
    
    // Decode a Snowflake ID
    public Map<String, Long> decode(long id) {
        long timestamp = (id >> TIMESTAMP_SHIFT) + EPOCH;
        long machine = (id >> MACHINE_ID_SHIFT) & MAX_MACHINE_ID;
        long seq = id & MAX_SEQUENCE;
        
        return Map.of(
            "timestamp", timestamp,
            "machineId", machine,
            "sequence", seq
        );
    }
}
```

```java
// Usage in Entity
@Entity
public class Order {
    @Id
    private Long id; // Snowflake ID (not auto-generated)
    
    private String product;
    private BigDecimal amount;
    
    @PrePersist
    public void generateId() {
        if (this.id == null) {
            this.id = snowflakeIdGenerator.nextId();
        }
    }
}
```

---

## 3. Redis Patterns

### pom.xml dependency
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

### Pattern 1: Distributed Lock (Prevent Double Processing)

```
Scenario: Two servers try to process the same payment simultaneously.
Without lock: Payment processed TWICE!
With Redis lock: Only ONE server processes it.
```

```java
@Component
public class RedisDistributedLock {
    
    @Autowired
    private StringRedisTemplate redisTemplate;
    
    /**
     * Try to acquire a lock.
     * @return true if lock acquired, false if already locked
     */
    public boolean tryLock(String lockKey, String requestId, long expireSeconds) {
        Boolean result = redisTemplate.opsForValue()
            .setIfAbsent("lock:" + lockKey, requestId, Duration.ofSeconds(expireSeconds));
        return Boolean.TRUE.equals(result);
    }
    
    /**
     * Release lock (only if we own it — prevents deleting someone else's lock)
     */
    public boolean releaseLock(String lockKey, String requestId) {
        String script = """
            if redis.call('get', KEYS[1]) == ARGV[1] then
                return redis.call('del', KEYS[1])
            else
                return 0
            end
            """;
        
        Long result = redisTemplate.execute(
            new DefaultRedisScript<>(script, Long.class),
            List.of("lock:" + lockKey),
            requestId
        );
        return Long.valueOf(1).equals(result);
    }
}

// Usage
@Service
public class PaymentService {
    
    @Autowired private RedisDistributedLock lock;
    
    public void processPayment(String orderId) {
        String requestId = UUID.randomUUID().toString();
        
        if (!lock.tryLock("payment:" + orderId, requestId, 30)) {
            throw new RuntimeException("Payment already being processed!");
        }
        
        try {
            // ... process payment safely (only one server executes this)
        } finally {
            lock.releaseLock("payment:" + orderId, requestId);
        }
    }
}
```

### Pattern 2: Redis Pub/Sub (Real-time Notifications)

```java
// ═══ Publisher ═══
@Service
public class NotificationPublisher {
    @Autowired
    private StringRedisTemplate redisTemplate;
    
    public void publish(String channel, String message) {
        redisTemplate.convertAndSend(channel, message);
    }
}

// ═══ Subscriber ═══
@Component
public class NotificationSubscriber implements MessageListener {
    
    @Override
    public void onMessage(Message message, byte[] pattern) {
        String channel = new String(message.getChannel());
        String body = new String(message.getBody());
        System.out.println("Received on " + channel + ": " + body);
    }
}

// ═══ Configuration ═══
@Configuration
public class RedisConfig {
    
    @Bean
    public RedisMessageListenerContainer container(RedisConnectionFactory factory,
                                                    NotificationSubscriber subscriber) {
        RedisMessageListenerContainer container = new RedisMessageListenerContainer();
        container.setConnectionFactory(factory);
        container.addMessageListener(subscriber, new ChannelTopic("notifications"));
        return container;
    }
}
```

### Pattern 3: Redis Leaderboard (Sorted Set)

```java
@Service
public class LeaderboardService {
    
    @Autowired
    private StringRedisTemplate redisTemplate;
    
    private static final String KEY = "game:leaderboard";
    
    // Add/Update score
    public void addScore(String player, double score) {
        redisTemplate.opsForZSet().incrementScore(KEY, player, score);
    }
    
    // Get rank (0-indexed, lowest score = rank 0)
    public Long getRank(String player) {
        return redisTemplate.opsForZSet().reverseRank(KEY, player); // Descending
    }
    
    // Top N players
    public Set<ZSetOperations.TypedTuple<String>> getTopN(int n) {
        return redisTemplate.opsForZSet().reverseRangeWithScores(KEY, 0, n - 1);
    }
    
    // Get score
    public Double getScore(String player) {
        return redisTemplate.opsForZSet().score(KEY, player);
    }
}
```

### Pattern 4: Redis as Cache (Cache-Aside Pattern)

```java
@Service
public class ProductService {
    
    @Autowired private ProductRepository repo;
    @Autowired private RedisTemplate<String, Product> redisTemplate;
    
    public Product getProduct(Long id) {
        String key = "product:" + id;
        
        // 1. Check cache first
        Product cached = redisTemplate.opsForValue().get(key);
        if (cached != null) return cached; // Cache HIT
        
        // 2. Cache MISS → query DB
        Product product = repo.findById(id).orElseThrow();
        
        // 3. Store in cache with TTL
        redisTemplate.opsForValue().set(key, product, Duration.ofMinutes(30));
        
        return product;
    }
    
    public Product updateProduct(Long id, Product updated) {
        Product saved = repo.save(updated);
        
        // INVALIDATE cache (Write-Through would update cache instead)
        redisTemplate.delete("product:" + id);
        
        return saved;
    }
}
```

---

## 4. RabbitMQ

### pom.xml
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
```

### application.yml
```yaml
spring:
  rabbitmq:
    host: localhost
    port: 5672
    username: guest
    password: guest
```

### Pattern 1: Simple Queue (Point-to-Point)

```java
// ═══ Configuration ═══
@Configuration
public class RabbitConfig {
    
    @Bean
    public Queue orderQueue() {
        return new Queue("order-queue", true); // durable = true
    }
    
    @Bean
    public Queue emailQueue() {
        return new Queue("email-queue", true);
    }
}

// ═══ Producer ═══
@Service
public class OrderProducer {
    
    @Autowired
    private RabbitTemplate rabbitTemplate;
    
    public void sendOrder(OrderEvent order) {
        rabbitTemplate.convertAndSend("order-queue", order);
        System.out.println("Order sent to queue: " + order.getOrderId());
    }
}

// ═══ Consumer ═══
@Component
public class OrderConsumer {
    
    @RabbitListener(queues = "order-queue")
    public void processOrder(OrderEvent order) {
        System.out.println("Processing order: " + order.getOrderId());
        // Process order... save to DB, update inventory, etc.
    }
}
```

### Pattern 2: Fanout Exchange (Broadcast to multiple queues)

```
Scenario: When an order is placed, notify:
  1. Inventory Service (reduce stock)
  2. Email Service (send confirmation)
  3. Analytics Service (track metrics)

  Producer → [Fanout Exchange] → Queue1 → Inventory Consumer
                                → Queue2 → Email Consumer
                                → Queue3 → Analytics Consumer
```

```java
@Configuration
public class FanoutConfig {
    
    @Bean
    public FanoutExchange orderExchange() {
        return new FanoutExchange("order-exchange");
    }
    
    @Bean public Queue inventoryQueue() { return new Queue("inventory-queue"); }
    @Bean public Queue emailQueue() { return new Queue("email-queue"); }
    @Bean public Queue analyticsQueue() { return new Queue("analytics-queue"); }
    
    @Bean
    public Binding inventoryBinding() {
        return BindingBuilder.bind(inventoryQueue()).to(orderExchange());
    }
    @Bean
    public Binding emailBinding() {
        return BindingBuilder.bind(emailQueue()).to(orderExchange());
    }
    @Bean
    public Binding analyticsBinding() {
        return BindingBuilder.bind(analyticsQueue()).to(orderExchange());
    }
}

// Producer sends to exchange (not directly to queue)
@Service
public class OrderService {
    @Autowired private RabbitTemplate rabbitTemplate;
    
    public void placeOrder(Order order) {
        // Save to DB...
        
        // Broadcast event to all consumers
        rabbitTemplate.convertAndSend("order-exchange", "", order);
    }
}

// Each consumer listens to its own queue
@Component
public class InventoryConsumer {
    @RabbitListener(queues = "inventory-queue")
    public void updateInventory(Order order) {
        System.out.println("Reducing stock for: " + order.getProductId());
    }
}

@Component
public class EmailConsumer {
    @RabbitListener(queues = "email-queue")
    public void sendEmail(Order order) {
        System.out.println("Sending confirmation email for order: " + order.getId());
    }
}
```

### Pattern 3: Topic Exchange (Route by pattern)

```java
// Route messages based on routing key pattern
// "order.created" → goes to order queue
// "order.cancelled" → goes to refund queue
// "order.*" → goes to analytics queue (matches all order events)

@Configuration
public class TopicConfig {
    @Bean public TopicExchange topicExchange() { return new TopicExchange("events"); }
    
    @Bean public Queue orderCreatedQueue() { return new Queue("order-created-queue"); }
    @Bean public Queue refundQueue() { return new Queue("refund-queue"); }
    @Bean public Queue allOrdersQueue() { return new Queue("all-orders-queue"); }
    
    @Bean
    public Binding orderCreatedBinding() {
        return BindingBuilder.bind(orderCreatedQueue()).to(topicExchange()).with("order.created");
    }
    @Bean
    public Binding refundBinding() {
        return BindingBuilder.bind(refundQueue()).to(topicExchange()).with("order.cancelled");
    }
    @Bean
    public Binding allOrdersBinding() {
        return BindingBuilder.bind(allOrdersQueue()).to(topicExchange()).with("order.*"); // Wildcard!
    }
}

// Send with routing key
rabbitTemplate.convertAndSend("events", "order.created", orderEvent);
rabbitTemplate.convertAndSend("events", "order.cancelled", cancelEvent);
```

### Dead Letter Queue (Handle Failed Messages)

```java
@Configuration
public class DLQConfig {
    
    @Bean
    public Queue mainQueue() {
        return QueueBuilder.durable("main-queue")
            .deadLetterExchange("dlx-exchange")    // Send failed messages here
            .deadLetterRoutingKey("dlq-routing-key")
            .ttl(30000)                              // 30 sec TTL
            .build();
    }
    
    @Bean public DirectExchange dlxExchange() { return new DirectExchange("dlx-exchange"); }
    @Bean public Queue deadLetterQueue() { return new Queue("dead-letter-queue"); }
    
    @Bean
    public Binding dlqBinding() {
        return BindingBuilder.bind(deadLetterQueue()).to(dlxExchange()).with("dlq-routing-key");
    }
}

// Consumer that might fail
@RabbitListener(queues = "main-queue")
public void process(Message msg) {
    try {
        // Process...
        if (somethingWrong) throw new RuntimeException("Processing failed");
    } catch (Exception e) {
        throw new AmqpRejectAndDontRequeueException("Sending to DLQ"); // Goes to DLQ
    }
}
```

---

## 5. Circuit Breaker (Resilience4j)

### Why?
When Service B is down, Service A keeps calling it → threads pile up → Service A also crashes! **Circuit Breaker** stops calling the failing service after N failures.

```
States:
  CLOSED  → Normal (requests flow through)
  OPEN    → Service is down (requests fail fast, don't even try)
  HALF_OPEN → Test with limited requests to check if service recovered
```

### pom.xml
```xml
<dependency>
    <groupId>io.github.resilience4j</groupId>
    <artifactId>resilience4j-spring-boot3</artifactId>
</dependency>
```

### application.yml
```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        slidingWindowSize: 10
        failureRateThreshold: 50        # Open after 50% failures
        waitDurationInOpenState: 10s     # Stay open for 10 seconds
        permittedNumberOfCallsInHalfOpenState: 3
  retry:
    instances:
      paymentService:
        maxAttempts: 3
        waitDuration: 1s
```

```java
@Service
public class PaymentService {
    
    @Autowired
    private RestTemplate restTemplate;
    
    @CircuitBreaker(name = "paymentService", fallbackMethod = "paymentFallback")
    @Retry(name = "paymentService")
    public PaymentResponse processPayment(PaymentRequest request) {
        // Call external payment gateway
        return restTemplate.postForObject(
            "https://payment-gateway.com/charge", request, PaymentResponse.class);
    }
    
    // Fallback when circuit is OPEN or service fails
    public PaymentResponse paymentFallback(PaymentRequest request, Throwable throwable) {
        // Queue for later processing
        rabbitTemplate.convertAndSend("payment-retry-queue", request);
        
        return new PaymentResponse("PENDING", "Payment queued for processing");
    }
}
```

---

## 6. Database Indexing

### Types of Indexes (MySQL/PostgreSQL)

```sql
-- ═══ 1. B-Tree Index (DEFAULT — most common) ═══
-- Good for: =, <, >, <=, >=, BETWEEN, ORDER BY, LIKE 'abc%'
CREATE INDEX idx_users_email ON users(email);

-- ═══ 2. Composite Index (Multi-column) ═══
-- Order matters! (city, age) works for: city, city+age. NOT for age alone.
CREATE INDEX idx_city_age ON users(city, age);

-- ═══ 3. Covering Index (includes all columns query needs) ═══
-- Query doesn't need to touch the actual table (index-only scan)
CREATE INDEX idx_covering ON orders(customer_id, status) INCLUDE (total_amount);

-- ═══ 4. Unique Index (enforces uniqueness) ═══
CREATE UNIQUE INDEX idx_unique_email ON users(email);

-- ═══ 5. Full-Text Index (for text search) ═══
CREATE FULLTEXT INDEX idx_fulltext ON articles(title, body);

-- ═══ 6. Hash Index (only for = comparisons, not range queries) ═══
CREATE INDEX idx_hash ON users USING HASH (email);
```

### JPA Index Annotations

```java
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_email", columnList = "email", unique = true),
    @Index(name = "idx_city_age", columnList = "city, age"),
    @Index(name = "idx_created", columnList = "createdAt")
})
public class User {
    @Id @GeneratedValue
    private Long id;
    
    @Column(unique = true)
    private String email;
    
    private String city;
    private int age;
    private LocalDateTime createdAt;
}
```

### When to Index (and When NOT to)

```
✅ INDEX when:
  - Column is in WHERE clause frequently
  - Column is in JOIN condition
  - Column is in ORDER BY / GROUP BY
  - High cardinality (many unique values, like email)
  - Read-heavy tables

❌ DON'T INDEX when:
  - Small tables (< 1000 rows — full scan is faster)
  - Low cardinality (like boolean, gender — only 2 values)
  - Write-heavy tables (indexes slow down INSERT/UPDATE)
  - Column rarely used in queries

EXPLAIN ANALYZE your queries to verify index usage!
```

```sql
-- Check if your query uses the index
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'john@example.com';

-- Show all indexes on a table
SHOW INDEX FROM users;
```

---

## 7. Sharding & Partitioning

### Partitioning (Single Database, Split Tables)

```
Partitioning splits ONE table into smaller pieces within the SAME database.

Types:
  Range Partition:  orders_2024, orders_2025 (by date range)
  List Partition:   orders_india, orders_usa (by country)
  Hash Partition:   orders_0, orders_1, orders_2 (by hash of ID)
```

```sql
-- MySQL Range Partitioning
CREATE TABLE orders (
    id BIGINT,
    amount DECIMAL(10,2),
    created_at DATE,
    PRIMARY KEY (id, created_at)
) PARTITION BY RANGE (YEAR(created_at)) (
    PARTITION p2023 VALUES LESS THAN (2024),
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION pFuture VALUES LESS THAN MAXVALUE
);
```

### Sharding (Multiple Databases)

```
Sharding splits data across MULTIPLE database servers.

User ID 1-1M     → Shard 1 (Server A)
User ID 1M-2M    → Shard 2 (Server B)
User ID 2M-3M    → Shard 3 (Server C)
```

```java
// ═══ Shard-Aware DataSource Router ═══
public class ShardRoutingDataSource extends AbstractRoutingDataSource {
    
    @Override
    protected Object determineCurrentLookupKey() {
        return ShardContext.getCurrentShard(); // ThreadLocal
    }
}

// ThreadLocal to hold current shard
public class ShardContext {
    private static final ThreadLocal<String> currentShard = new ThreadLocal<>();
    
    public static void setShard(String shard) { currentShard.set(shard); }
    public static String getCurrentShard() { return currentShard.get(); }
    public static void clear() { currentShard.remove(); }
}

// Configuration
@Configuration
public class ShardingConfig {
    
    @Bean
    public DataSource dataSource() {
        ShardRoutingDataSource routingDS = new ShardRoutingDataSource();
        
        Map<Object, Object> shards = new HashMap<>();
        shards.put("shard_0", createDataSource("jdbc:mysql://shard0:3306/db"));
        shards.put("shard_1", createDataSource("jdbc:mysql://shard1:3306/db"));
        shards.put("shard_2", createDataSource("jdbc:mysql://shard2:3306/db"));
        
        routingDS.setTargetDataSources(shards);
        routingDS.setDefaultTargetDataSource(shards.get("shard_0"));
        
        return routingDS;
    }
    
    private DataSource createDataSource(String url) {
        HikariDataSource ds = new HikariDataSource();
        ds.setJdbcUrl(url);
        ds.setUsername("root");
        ds.setPassword("password");
        return ds;
    }
}

// Shard selection logic
@Service
public class UserService {
    
    @Autowired private UserRepository userRepo;
    
    public User getUser(Long userId) {
        String shard = "shard_" + (userId % 3); // Hash-based sharding
        ShardContext.setShard(shard);
        
        try {
            return userRepo.findById(userId).orElseThrow();
        } finally {
            ShardContext.clear();
        }
    }
}
```

### Sharding Strategies Comparison

| Strategy | How | Pros | Cons |
|:---|:---|:---|:---|
| **Range** | userId 1-1M → Shard1 | Simple, range queries easy | Hotspots (new users all hit last shard) |
| **Hash** | `userId % N` → ShardN | Even distribution | Adding shards requires reshuffling |
| **Directory** | Lookup table maps key→shard | Flexible | Lookup table is single point of failure |
| **Geo** | Region → shard | Data locality | Cross-region queries are hard |

---

## 8. Consistent Hashing

### Why?
With simple hash (`userId % 3`), adding a 4th server reshuffles EVERYTHING. Consistent hashing minimizes data movement.

```java
public class ConsistentHash<T> {
    
    private final TreeMap<Long, T> ring = new TreeMap<>();
    private final int virtualNodes;
    private final MessageDigest md;
    
    public ConsistentHash(int virtualNodes, Collection<T> nodes) {
        this.virtualNodes = virtualNodes;
        try { this.md = MessageDigest.getInstance("MD5"); } 
        catch (Exception e) { throw new RuntimeException(e); }
        
        for (T node : nodes) addNode(node);
    }
    
    public void addNode(T node) {
        for (int i = 0; i < virtualNodes; i++) {
            long hash = hash(node.toString() + "#" + i);
            ring.put(hash, node);
        }
    }
    
    public void removeNode(T node) {
        for (int i = 0; i < virtualNodes; i++) {
            long hash = hash(node.toString() + "#" + i);
            ring.remove(hash);
        }
    }
    
    // Find which node handles this key
    public T getNode(String key) {
        if (ring.isEmpty()) throw new RuntimeException("No nodes in ring!");
        
        long hash = hash(key);
        Map.Entry<Long, T> entry = ring.ceilingEntry(hash); // First node clockwise
        
        if (entry == null) {
            entry = ring.firstEntry(); // Wrap around
        }
        return entry.getValue();
    }
    
    private long hash(String key) {
        md.reset();
        byte[] digest = md.digest(key.getBytes());
        return ((long)(digest[0] & 0xFF) << 24)
             | ((long)(digest[1] & 0xFF) << 16)
             | ((long)(digest[2] & 0xFF) << 8)
             | (digest[3] & 0xFF);
    }
}

// Usage
List<String> servers = List.of("server-1", "server-2", "server-3");
ConsistentHash<String> ch = new ConsistentHash<>(150, servers); // 150 virtual nodes each

String server = ch.getNode("user:12345"); // → "server-2"
ch.addNode("server-4");                    // Only ~25% of keys move!
```

---

## 9. API Gateway & Load Balancing

### Spring Cloud Gateway (API Gateway)

```java
// application.yml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://USER-SERVICE          # Load balanced
          predicates:
            - Path=/api/users/**
          filters:
            - StripPrefix=1
            - name: CircuitBreaker
              args:
                name: userServiceCB
                fallbackUri: forward:/fallback/users
        
        - id: order-service
          uri: lb://ORDER-SERVICE
          predicates:
            - Path=/api/orders/**
          filters:
            - StripPrefix=1
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 10
                redis-rate-limiter.burstCapacity: 20
```

### Client-Side Load Balancing (Spring Cloud LoadBalancer)

```java
// Round Robin is the default. Custom:
@Configuration
public class LoadBalancerConfig {
    
    @Bean
    public ReactorLoadBalancer<ServiceInstance> randomLoadBalancer(
            ServiceInstanceListSupplier supplier) {
        return new RandomLoadBalancer(supplier, "my-service");
    }
}

// Using @LoadBalanced RestTemplate
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}

// Now this auto-resolves and load balances:
restTemplate.getForObject("http://USER-SERVICE/api/users/1", User.class);
// "USER-SERVICE" is resolved to an actual server via service discovery (Eureka)
```

---

## 10. URL Shortener — Full Implementation

```java
@Entity
@Table(indexes = @Index(name = "idx_short_code", columnList = "shortCode", unique = true))
public class UrlMapping {
    @Id @GeneratedValue
    private Long id;
    
    @Column(unique = true, length = 8)
    private String shortCode;
    
    @Column(length = 2048)
    private String originalUrl;
    
    private long clickCount;
    private LocalDateTime createdAt;
    private LocalDateTime expiresAt;
}

@Service
public class UrlShortenerService {
    
    @Autowired private UrlMappingRepository repo;
    @Autowired private StringRedisTemplate redis;
    
    private static final String BASE62 = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz";
    
    // Shorten URL
    public String shorten(String originalUrl) {
        // Check if already shortened
        return repo.findByOriginalUrl(originalUrl)
            .map(UrlMapping::getShortCode)
            .orElseGet(() -> {
                String code = generateCode();
                UrlMapping mapping = new UrlMapping();
                mapping.setShortCode(code);
                mapping.setOriginalUrl(originalUrl);
                mapping.setCreatedAt(LocalDateTime.now());
                mapping.setExpiresAt(LocalDateTime.now().plusYears(1));
                repo.save(mapping);
                
                // Cache in Redis
                redis.opsForValue().set("url:" + code, originalUrl, Duration.ofDays(30));
                
                return code;
            });
    }
    
    // Resolve short URL
    public String resolve(String shortCode) {
        // Check Redis cache first
        String cached = redis.opsForValue().get("url:" + shortCode);
        if (cached != null) return cached;
        
        // Cache miss → DB
        UrlMapping mapping = repo.findByShortCode(shortCode)
            .orElseThrow(() -> new ResponseStatusException(HttpStatus.NOT_FOUND));
        
        // Increment click count asynchronously
        repo.incrementClickCount(shortCode);
        
        // Cache for future
        redis.opsForValue().set("url:" + shortCode, mapping.getOriginalUrl(), Duration.ofDays(30));
        
        return mapping.getOriginalUrl();
    }
    
    private String generateCode() {
        // Base62 encoding of a counter or random number
        long num = System.nanoTime();
        StringBuilder sb = new StringBuilder();
        while (num > 0 && sb.length() < 7) {
            sb.append(BASE62.charAt((int)(num % 62)));
            num /= 62;
        }
        return sb.reverse().toString();
    }
}

@RestController
@RequestMapping("/api")
public class UrlController {
    
    @Autowired private UrlShortenerService service;
    
    @PostMapping("/shorten")
    public Map<String, String> shorten(@RequestBody Map<String, String> request) {
        String shortCode = service.shorten(request.get("url"));
        return Map.of("shortUrl", "https://short.ly/" + shortCode);
    }
    
    @GetMapping("/{code}")
    public ResponseEntity<Void> redirect(@PathVariable String code) {
        String originalUrl = service.resolve(code);
        return ResponseEntity.status(HttpStatus.MOVED_PERMANENTLY)
            .header("Location", originalUrl)
            .build();
    }
}
```

---

## 11. Notification Service (Fan-Out Pattern)

```java
// When user posts, notify all followers
@Service
public class NotificationService {
    
    @Autowired private RabbitTemplate rabbitTemplate;
    @Autowired private RedisTemplate<String, String> redisTemplate;
    
    // Fan-out write: Push notification to each follower's feed
    public void notifyFollowers(Long userId, String message) {
        Set<String> followers = redisTemplate.opsForSet().members("followers:" + userId);
        
        if (followers != null) {
            for (String followerId : followers) {
                // Push to each follower's notification list
                redisTemplate.opsForList().leftPush("notifications:" + followerId, message);
                redisTemplate.opsForList().trim("notifications:" + followerId, 0, 99); // Keep last 100
                
                // Also send real-time via WebSocket/RabbitMQ
                rabbitTemplate.convertAndSend("notifications-exchange", 
                    "user." + followerId, message);
            }
        }
    }
    
    // Get user's notifications
    public List<String> getNotifications(Long userId, int page, int size) {
        long start = (long) page * size;
        long end = start + size - 1;
        return redisTemplate.opsForList().range("notifications:" + userId, start, end);
    }
}
```

---

## 12. Event Sourcing & CQRS

```java
// ═══ Event Store ═══
@Entity
public class DomainEvent {
    @Id @GeneratedValue
    private Long id;
    private String aggregateId;       // e.g., "order:123"
    private String eventType;         // e.g., "ORDER_CREATED"
    
    @Column(columnDefinition = "TEXT")
    private String payload;           // JSON data
    
    private LocalDateTime occurredAt;
    private int version;              // For optimistic locking
}

// ═══ Command Side (Write) ═══
@Service
public class OrderCommandService {
    
    @Autowired private DomainEventRepository eventRepo;
    @Autowired private RabbitTemplate rabbitTemplate;
    
    @Transactional
    public void createOrder(CreateOrderCommand cmd) {
        // 1. Create event (not the entity directly!)
        DomainEvent event = new DomainEvent();
        event.setAggregateId("order:" + cmd.getOrderId());
        event.setEventType("ORDER_CREATED");
        event.setPayload(objectMapper.writeValueAsString(cmd));
        event.setOccurredAt(LocalDateTime.now());
        
        // 2. Save event to event store
        eventRepo.save(event);
        
        // 3. Publish for read-side to consume
        rabbitTemplate.convertAndSend("events", "order.created", event);
    }
}

// ═══ Query Side (Read) — Separate optimized read model ═══
@Component
public class OrderProjection {
    
    @Autowired private OrderReadRepository readRepo; // Denormalized read model
    
    @RabbitListener(queues = "order-projection-queue")
    public void onOrderCreated(DomainEvent event) {
        if ("ORDER_CREATED".equals(event.getEventType())) {
            // Build denormalized read model
            OrderView view = new OrderView();
            // ... populate from event payload
            readRepo.save(view);
        }
    }
}
```

---

## 13. Idempotency (Preventing Duplicate Requests)

```java
@Component
public class IdempotencyFilter implements Filter {
    
    @Autowired private StringRedisTemplate redis;
    
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        
        HttpServletRequest request = (HttpServletRequest) req;
        
        // Only for mutating requests
        if (!"POST".equals(request.getMethod()) && !"PUT".equals(request.getMethod())) {
            chain.doFilter(req, res);
            return;
        }
        
        String idempotencyKey = request.getHeader("Idempotency-Key");
        if (idempotencyKey == null) {
            chain.doFilter(req, res);
            return;
        }
        
        String redisKey = "idempotency:" + idempotencyKey;
        
        // Check if this request was already processed
        String cachedResponse = redis.opsForValue().get(redisKey);
        if (cachedResponse != null) {
            // Return cached response (don't process again!)
            HttpServletResponse response = (HttpServletResponse) res;
            response.setContentType("application/json");
            response.getWriter().write(cachedResponse);
            return;
        }
        
        // Process the request and cache the response
        ContentCachingResponseWrapper wrappedResponse = 
            new ContentCachingResponseWrapper((HttpServletResponse) res);
        chain.doFilter(req, wrappedResponse);
        
        // Cache response for 24 hours
        String responseBody = new String(wrappedResponse.getContentAsByteArray());
        redis.opsForValue().set(redisKey, responseBody, Duration.ofHours(24));
        wrappedResponse.copyBodyToResponse();
    }
}

// Client sends:
// POST /api/payments
// Idempotency-Key: abc-123-def
// → First call: processes payment, caches response
// → Second call (retry): returns cached response (no double charge!)
```

---

## 14. Saga Pattern (Distributed Transactions)

```
Problem: Order Service needs to:
  1. Create Order (Order DB)
  2. Reserve Inventory (Inventory DB)
  3. Charge Payment (Payment DB)
  
If step 3 fails, we must UNDO steps 1 and 2!

Choreography Saga: Each service publishes events, others react.
Orchestration Saga: A central orchestrator coordinates the steps.
```

```java
// ═══ Orchestrator-based Saga ═══
@Service
public class OrderSagaOrchestrator {
    
    @Autowired private OrderService orderService;
    @Autowired private InventoryClient inventoryClient;
    @Autowired private PaymentClient paymentClient;
    
    @Transactional
    public OrderResult createOrder(CreateOrderRequest request) {
        String sagaId = UUID.randomUUID().toString();
        
        try {
            // Step 1: Create Order (PENDING)
            Order order = orderService.createPendingOrder(request);
            
            // Step 2: Reserve Inventory
            boolean reserved = inventoryClient.reserve(order.getProductId(), order.getQuantity());
            if (!reserved) {
                orderService.cancelOrder(order.getId(), "Out of stock");
                return OrderResult.failure("Out of stock");
            }
            
            // Step 3: Process Payment
            boolean paid = paymentClient.charge(order.getUserId(), order.getAmount());
            if (!paid) {
                // COMPENSATE: Release inventory
                inventoryClient.release(order.getProductId(), order.getQuantity());
                orderService.cancelOrder(order.getId(), "Payment failed");
                return OrderResult.failure("Payment failed");
            }
            
            // All steps succeeded!
            orderService.confirmOrder(order.getId());
            return OrderResult.success(order);
            
        } catch (Exception e) {
            // Compensate all completed steps
            compensate(sagaId);
            throw new RuntimeException("Saga failed", e);
        }
    }
}
```

---

## 15. Top 20 Interview Questions with Answers

| # | Question | Key Answer |
|:---:|:---|:---|
| 1 | How do you implement Rate Limiting? | Token Bucket (in-memory) or Redis Sliding Window (distributed). Use Filter/AOP. |
| 2 | How do you generate unique IDs in distributed systems? | Snowflake: 41-bit timestamp + 10-bit machine + 12-bit sequence = 64-bit sortable ID |
| 3 | Redis vs Memcached? | Redis: data structures (list, set, sorted set, pub/sub, persistence). Memcached: simple K-V, multi-threaded. |
| 4 | How to prevent double payment? | Idempotency key in header + Redis cache of processed requests |
| 5 | Explain Cache-Aside pattern | Check cache → miss → query DB → store in cache. Invalidate on write. |
| 6 | When to use RabbitMQ vs Kafka? | RabbitMQ: task queues, complex routing. Kafka: event streaming, high throughput, replay. |
| 7 | What is a Circuit Breaker? | Stops calling failing service. States: CLOSED→OPEN→HALF_OPEN. Resilience4j in Spring. |
| 8 | Sharding vs Partitioning? | Partitioning: one DB, split tables. Sharding: multiple DBs across servers. |
| 9 | How does Consistent Hashing work? | Nodes on a ring. Key mapped to nearest node clockwise. Adding node moves ~1/N keys. |
| 10 | Explain the Saga pattern | Distributed transaction via compensating actions. Orchestration (central) vs Choreography (events). |
| 11 | What indexes to add? | Columns in WHERE, JOIN, ORDER BY. Composite index order matters. EXPLAIN to verify. |
| 12 | How to design a URL shortener? | Base62 encode ID → 7-char code. Redis cache + DB. 301 redirect. |
| 13 | Fan-out on Write vs Read? | Write: push to all followers on post (fast read, slow write). Read: pull at read time (fast write, slow read). |
| 14 | How to handle distributed locks? | Redis SETNX with expiry. Lua script to release. Redisson for production. |
| 15 | Event Sourcing vs CRUD? | Event Sourcing: store events (audit trail, replay). CRUD: store current state (simpler). |
| 16 | What is CQRS? | Separate read/write models. Write: normalized. Read: denormalized for fast queries. |
| 17 | How to do DB migration without downtime? | Expand-Contract: add new column → backfill → switch code → drop old column. |
| 18 | What is a Dead Letter Queue? | Failed messages go to DLQ for manual review instead of being lost. |
| 19 | How to handle hot partitions? | Add salt to partition key. Consistent hashing with virtual nodes. |
| 20 | Explain Write-Behind vs Write-Through cache | Write-Through: update cache + DB synchronously. Write-Behind: update cache, async write to DB. |
