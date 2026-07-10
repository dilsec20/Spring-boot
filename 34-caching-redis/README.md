# ⚡ Spring Boot Caching & Redis — Complete In-Depth Guide

> **"The fastest database query is the one you never make. Caching stores frequently accessed, rarely changing data in memory (RAM), reducing latency from milliseconds to microseconds and protecting your database from crushing loads."**

---

## 📑 Table of Contents

1. [Why Caching?](#1-why-caching)
2. [Spring Cache Abstraction](#2-spring-cache-abstraction)
3. [Setup & Configuration](#3-setup--configuration)
4. [The `@Cacheable` Annotation](#4-the-cacheable-annotation)
5. [The `@CachePut` Annotation](#5-the-cacheput-annotation)
6. [The `@CacheEvict` Annotation](#6-the-cacheevict-annotation)
7. [Integrating Redis (Distributed Cache)](#7-integrating-redis-distributed-cache)
8. [Cache Expiration & TTL](#8-cache-expiration--ttl)
9. [Local vs Distributed Caching](#9-local-vs-distributed-caching)
10. [Cache Eviction Strategies](#10-cache-eviction-strategies)
11. [Interview Questions & Answers (50+)](#11-interview-questions--answers-50)

---

## 1. Why Caching?

```
Without Cache:
Client → API (10ms) → DB Query (150ms) → Response = 160ms
*If 1000 users ask for the same product, the DB runs the same query 1000 times.*

With Cache:
Client → API (10ms) → Cache Hit (1ms) → Response = 11ms
*The DB is completely bypassed. App handles 100x more traffic.*

Good candidates for caching:
✅ Product catalogs, configuration data, top 10 leaderboards.
✅ Data that is READ frequently but WRITTEN rarely.

BAD candidates for caching:
❌ Real-time stock prices, banking balances.
❌ Data that changes every second.
```

---

## 2. Spring Cache Abstraction

Spring provides a transparent cache abstraction. It separates the caching logic from the business logic using AOP (Aspect-Oriented Programming). 

You write standard Java code, add an annotation, and Spring intercepts the method call to check the cache before executing the method.

Supported Cache Providers:
*   `ConcurrentMapCache` (Default, built-in, local memory)
*   `Caffeine` (High-performance local memory)
*   `Redis` (Distributed in-memory data store)
*   `Ehcache`, `Hazelcast`

---

## 3. Setup & Configuration

**1. Add Dependency**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

**2. Enable Caching**
Add `@EnableCaching` to your main application class or a configuration class.

```java
@SpringBootApplication
@EnableCaching  // CRITICAL: Caching annotations won't work without this!
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
}
```

*By default, if no specific provider is on the classpath, Spring uses a simple `ConcurrentHashMap`.*

---

## 4. The `@Cacheable` Annotation

Used for **Reading**.
*   **Behavior:** Checks the cache first. 
*   If data exists (Cache Hit) → Returns immediately. Method is NOT executed.
*   If data is missing (Cache Miss) → Executes method, puts result in cache, returns data.

```java
@Service
public class ProductService {

    // Cache name: "products", Key: the 'id' parameter
    @Cacheable(value = "products", key = "#id")
    public Product getProductById(Long id) {
        simulateSlowDbQuery();
        return productRepository.findById(id).orElseThrow();
    }
    
    // Custom key generation using SpEL (Spring Expression Language)
    @Cacheable(value = "users", key = "#user.email")
    public UserDetails loadUser(User user) { ... }
    
    // Conditional caching (Only cache if price > 100)
    @Cacheable(value = "products", condition = "#result.price > 100")
    public Product getExpensiveProduct(Long id) { ... }
}
```

---

## 5. The `@CachePut` Annotation

Used for **Updating**.
*   **Behavior:** ALWAYS executes the method. Takes the result and updates the cache.
*   Used on Update/Save methods to keep the cache fresh.

```java
@Service
public class ProductService {

    @CachePut(value = "products", key = "#product.id")
    public Product updateProduct(Product product) {
        // ALWAYS executes. Replaces the entry in the cache with the updated product.
        return productRepository.save(product);
    }
}
```
*(Warning: Never mix `@Cacheable` and `@CachePut` on the exact same method. They have conflicting behaviors).*

---

## 6. The `@CacheEvict` Annotation

Used for **Deleting**.
*   **Behavior:** Removes data from the cache.

```java
@Service
public class ProductService {

    // Removes specific product from cache
    @CacheEvict(value = "products", key = "#id")
    public void deleteProduct(Long id) {
        productRepository.deleteById(id);
    }
    
    // Removes ALL products from this cache (e.g., when massive updates happen)
    @CacheEvict(value = "products", allEntries = true)
    public void clearProductCache() {
        log.info("Product cache cleared!");
    }
}
```

---

## 7. Integrating Redis (Distributed Cache)

The default `ConcurrentHashMap` cache only lives in the RAM of ONE server. If you have 5 microservices running, they each have their own empty cache. **Redis** solves this by putting the cache in a centralized, ultra-fast, external server.

**1. Start Redis (Docker)**
```bash
docker run -d -p 6379:6379 --name my-redis redis
```

**2. Add Redis Dependency**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

**3. Configure Properties**
```properties
spring.data.redis.host=localhost
spring.data.redis.port=6379
# Spring Boot detects this and automatically switches the CacheManager to RedisCacheManager!
```

**IMPORTANT: Serialization**
By default, Redis stores data in binary format using Java Serialization. Your classes MUST implement `Serializable`. Alternatively, configure Redis to store JSON.

```java
// Better: Configure Redis to store JSON so you can read it easily via CLI
@Bean
public RedisCacheConfiguration cacheConfiguration() {
    return RedisCacheConfiguration.defaultCacheConfig()
        .entryTtl(Duration.ofMinutes(60)) // Default TTL
        .disableCachingNullValues()
        .serializeValuesWith(SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));
}
```

---

## 8. Cache Expiration & TTL

Caches shouldn't live forever (data becomes stale). You need a TTL (Time To Live).

If using Redis, configure TTLs per cache name:

```java
@Bean
public RedisCacheManagerBuilderCustomizer redisCacheManagerBuilderCustomizer() {
    return (builder) -> builder
        .withCacheConfiguration("products",
            RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofMinutes(10)))
        .withCacheConfiguration("users",
            RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofDays(1)));
}
```

---

## 9. Local vs Distributed Caching

| Feature | Local Cache (Caffeine/HashMap) | Distributed Cache (Redis/Memcached) |
|---------|--------------------------------|-------------------------------------|
| **Location** | Inside JVM Memory (RAM) | External Server |
| **Speed** | ⚡ Nanoseconds (Fastest) | 🏎️ Milliseconds (Network jump) |
| **Microservices**| ❌ Bad. Cache is isolated per instance. | ✅ Good. All instances share same cache. |
| **Persistence** | Lost on app restart | Can survive app restarts |
| **Setup** | Zero setup | Requires maintaining a Redis server |

*Pro Tip: Two-Level Caching (L1 + L2). Use Caffeine as L1 (local). If miss, check Redis (L2). If miss, hit Database.*

---

## 10. Cache Eviction Strategies

When the cache memory gets full, how does Redis decide what to delete?

*   **LRU (Least Recently Used):** Drops the data that hasn't been requested in the longest time. *(Most common)*
*   **LFU (Least Frequently Used):** Drops the data that is requested the fewest amount of times overall.
*   **FIFO (First In, First Out):** Drops the oldest data, regardless of usage.

*Configured in `redis.conf`: `maxmemory-policy allkeys-lru`*

---

## 11. Interview Questions & Answers (50+)

### Beginner

**Q1: Why do we use caching?** To improve application performance by storing frequently accessed data in fast memory, reducing database load and response times.

**Q2: What is a Cache Hit?** When the requested data is found in the cache.

**Q3: What is a Cache Miss?** When the requested data is NOT found in the cache, forcing the app to fetch it from the database.

**Q4: What annotation enables caching in Spring Boot?** `@EnableCaching`.

**Q5: What does `@Cacheable` do?** Checks if data is in the cache. If yes, returns it. If no, runs the method, puts the result in the cache, and returns it.

**Q6: What is Redis?** An open-source, in-memory, key-value data store used as a database, cache, and message broker.

**Q7: Default port for Redis?** 6379.

**Q8: Local vs Distributed cache?** Local (Caffeine) lives in the app's RAM. Distributed (Redis) lives on an external server accessible by multiple app instances.

---

### Intermediate

**Q9: Difference between `@Cacheable` and `@CachePut`?** `@Cacheable` skips method execution if the key exists. `@CachePut` *always* executes the method and updates the cache with the new result.

**Q10: What is TTL?** Time To Live. How long an item stays in the cache before expiring automatically.

**Q11: How do you clear a cache?** Using `@CacheEvict(value="myCache", allEntries=true)`.

**Q12: What happens if you call a `@Cacheable` method from within the *same* class?** The cache will NOT work. Spring Cache uses AOP proxies. Internal method calls bypass the proxy.

**Q13: How do you conditionally cache data?** Using the `condition` attribute: `@Cacheable(value="items", condition="#id > 10")`.

**Q14: What is `unless` in `@Cacheable`?** Evaluates *after* the method runs. If true, the result is NOT cached. Example: `unless="#result == null"`.

**Q15: What is the Cache Stampede (Thundering Herd) problem?** When a highly popular cache key expires, thousands of concurrent requests all get a cache miss and hit the database simultaneously, crashing it.

---

### Rapid-Fire (Q16–Q50)

**Q16: Can you cache a `void` method?** Technically yes with `@CacheEvict`, but not with `@Cacheable` (nothing to store).

**Q17: What does `@Caching` do?** Allows grouping multiple `@CacheEvict` or `@CachePut` annotations on a single method.

**Q18: What is the default cache implementation if none is specified?** `ConcurrentMapCacheManager`.

**Q19: Why use Caffeine over HashMap?** Caffeine provides eviction policies (size-based, time-based) and statistics. HashMap just grows until OutOfMemory.

**Q20: Does Redis store data permanently?** By default it is in-memory, but it can persist to disk using RDB (snapshots) or AOF (append-only file).

**Q21: How to generate a dynamic cache key?** Using SpEL: `key = "#user.id"`.

**Q22: If `key` is not specified, what is the default key?** The method parameters are used as the key.

**Q23: How to connect Spring Boot to Redis?** `spring-boot-starter-data-redis` and properties (`spring.redis.host`).

**Q24: What is `CacheManager`?** The Spring interface that manages the various Cache instances (e.g., `RedisCacheManager`).

**Q25: What is LRU?** Least Recently Used eviction policy.

**Q26: What is Cache Invalidation?** The process of removing or updating stale data in the cache.

**Q27: What is Write-Through cache?** App writes to the cache, and the cache synchronously writes to the DB.

**Q28: What is Write-Behind (Write-Back) cache?** App writes to the cache, and the cache asynchronously writes to the DB later. (Faster, but risk of data loss).

**Q29: What is Cache-Aside (Lazy Loading)?** App checks cache, if miss, app checks DB, app writes to cache. (This is how Spring `@Cacheable` works).

**Q30: How to serialize Redis data as JSON in Spring?** Use `GenericJackson2JsonRedisSerializer`.

**Q31: What exception happens if you cache an object that doesn't implement Serializable (using default Java serializer)?** `NotSerializableException`.

**Q32: Should you cache API responses containing user-specific sensitive data?** Be very careful. Ensure the cache key includes the User ID/Token, otherwise User A might see User B's data!

**Q33: Can you clear cache via Spring Boot Actuator?** Yes, `POST /actuator/caches` clears them if the endpoint is enabled.

**Q34: How to view Redis data in CLI?** `redis-cli`, then run `KEYS *` or `GET key_name`.

**Q35: What is Memcached?** Another distributed caching system, older and simpler than Redis (only supports strings, no complex data types like Lists/Hashes).

**Q36: Why does Spring use Proxies for caching?** To intercept the method call without modifying your actual byte code.

**Q37: What is `sync=true` in `@Cacheable`?** Synchronizes the cache lookup. If 10 threads get a cache miss simultaneously, only ONE thread hits the DB, the others wait for it to populate the cache. (Solves Cache Stampede!).

**Q38: Can I use multiple Cache Managers in one app?** Yes, you can define multiple beans and specify which one to use: `@Cacheable(cacheManager="redisCacheManager")`.

**Q39: How to test caching?** Mock the repository. Call the service method twice. Verify the repository was only called once using `Mockito.verify(repo, times(1))`.

**Q40: What happens if Redis goes down?** If not handled, Spring will throw exceptions and fail the request. You can configure a custom `CacheErrorHandler` to fall back to the DB gracefully.

**Q41: What is Redis Sentinel?** High availability for Redis (failover if master dies).

**Q42: What is Redis Cluster?** Data sharding across multiple Redis nodes for massive datasets.

**Q43: Can Redis be used for messaging?** Yes, it supports Pub/Sub.

**Q44: `RedisTemplate` vs `StringRedisTemplate`?** `StringRedisTemplate` is pre-configured to handle String keys and String values (very common).

**Q45: What is the `keyGenerator` attribute?** Allows you to point to a custom Bean that generates complex cache keys programmatically.

**Q46: How to handle caching large collections?** Don't cache massive lists as one key. Cache individual items, or use pagination keys (`users_page_1`).

**Q47: Is caching ACID compliant?** No. Caches sacrifice strong consistency for high availability and performance (Eventual Consistency).

**Q48: Does `@CacheEvict` run before or after the method?** After by default. Set `beforeInvocation=true` to run it before (useful if the method might throw an exception).

**Q49: What is JSR-107 (JCache)?** The standard Java caching API. Spring implements it, so you can use `@CacheResult` instead of `@Cacheable`.

**Q50: Best practice for cache keys?** Include entity names and IDs to avoid collisions (e.g., `User::123` instead of just `123`).

---

## 📚 References

- [Spring Cache Documentation](https://docs.spring.io/spring-framework/reference/integration/cache.html)
- [Redis Official Site](https://redis.io/)
- [Caffeine Cache](https://github.com/ben-manes/caffeine)

---

> **Previous Topic:** [← 33 - Spring for GraphQL](../33-spring-graphql/README.md)  
> **Next Topic:** [35 - Advanced Testing →](../35-advanced-testing/README.md)
