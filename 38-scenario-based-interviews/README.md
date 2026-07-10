# 🕵️‍♂️ Scenario-Based Interviews (Knowing vs Mastering)

> **"Top-tier product companies (paying 20-40+ LPA) don't just ask 'What is an Annotation?' They ask: 'Our server crashed during the Big Billion Day sale. Walk me through how you find and fix the bottleneck.' This is the difference between Knowing and Mastering."**

---

## 📑 Table of Contents

1. [Caching (The Thundering Herd)](#1-caching-the-thundering-herd)
2. [Database (The N+1 & Connection Leaks)](#2-database-the-n1--connection-leaks)
3. [Microservices (Cascading Failures)](#3-microservices-cascading-failures)
4. [Kafka (Poison Pills & Rebalancing)](#4-kafka-poison-pills--rebalancing)
5. [Java Concurrency (Thread Pool Exhaustion)](#5-java-concurrency-thread-pool-exhaustion)
6. [Security (JWT Invalidation)](#6-security-jwt-invalidation)
7. [Spring Boot Internals (Circular Dependencies)](#7-spring-boot-internals-circular-dependencies)

---

## 1. Caching (The Thundering Herd)

### 🔴 The Scenario
You work at an e-commerce company. The homepage hits a backend API to load the "Top 10 Deals of the Day." The query is heavy, so you cached it. At exactly 12:00 PM, the cache expires (TTL ends). Suddenly, 5,000 users hit the homepage simultaneously. 

### ⚠️ The "Knowing" Answer (Junior Level)
"I will use `@Cacheable` on the method. If the cache expires, the next request will hit the database, put it back in the cache, and the rest of the users will get the cached data."

### 🏆 The "Mastering" Answer (Senior Level)
"What you described is the **Cache Stampede (Thundering Herd) problem**.
If 5,000 requests arrive exactly when the cache expires, ALL 5,000 threads will experience a 'cache miss' simultaneously because the cache hasn't been repopulated yet. All 5,000 threads will fire that heavy query to the database at the exact same millisecond. The database will choke, connections will max out, and the database will crash.

**How to fix it:**
1. **Synchronized Caching:** In Spring, I would use `@Cacheable(sync = true)`. This ensures that when the cache misses, only ONE thread is allowed to query the database and update the cache. The other 4,999 threads are blocked and wait for the cache to be updated, preventing the DB crash.
2. **Background Refresh (Write-Behind):** Instead of letting the cache expire and waiting for a user request to trigger a refresh, I'll set up a `@Scheduled` background job to refresh the cache every 5 minutes *before* the TTL expires."

---

## 2. Database (The N+1 & Connection Leaks)

### 🔴 The Scenario
You built an API that fetches 100 `Orders` and their associated `User` details. In the DEV environment (with 5 orders), the API responds in 50ms. In PROD (with 100,000 orders), the API takes 5 seconds to load just 100 orders and sometimes throws a `ConnectionPoolTimeoutException`.

### ⚠️ The "Knowing" Answer (Junior Level)
"I probably need to add an index on the database table, or maybe increase the HikariCP `maximum-pool-size` to allow more connections."

### 🏆 The "Mastering" Answer (Senior Level)
"This is a classic **N+1 Query Problem**. 
Because `User` is likely mapped as `@ManyToOne`, if it's set to `FetchType.EAGER` (the default) or even `LAZY` (but accessed in a loop), Hibernate executes 1 query to fetch the 100 orders, and then 100 separate queries to fetch the user for each order. That's 101 queries!

**How to fix it:**
1. I will write a custom JPQL query using `JOIN FETCH` (e.g., `SELECT o FROM Order o JOIN FETCH o.user`) to fetch everything in ONE single SQL query.
2. Or, I will use JPA `@EntityGraph` to dynamically specify which relationships to fetch eagerly for this specific API call.

**Regarding the `ConnectionPoolTimeoutException`:**
Increasing the pool size is a band-aid, not a cure. The timeout happens because those 101 queries are holding onto a connection for too long, starving other threads. Fixing the N+1 problem will drastically reduce the time the connection is held. If the error persists, I will check for slow, un-indexed queries or transactions (`@Transactional`) that are kept open during long API calls (e.g., doing HTTP calls inside a DB transaction, which is an anti-pattern)."

---

## 3. Microservices (Cascading Failures)

### 🔴 The Scenario
Service A calls Service B. Service B calls Service C. 
Service C's database goes down, causing Service C to take 30 seconds to respond with a timeout. 
Suddenly, Service A and Service B both crash completely, even though their databases are fine.

### ⚠️ The "Knowing" Answer (Junior Level)
"I will wrap the REST call in a try-catch block. If Service C throws an exception, I'll catch it and return a default response."

### 🏆 The "Mastering" Answer (Senior Level)
"Try-catch won't save you here. This is a **Cascading Failure resulting from Thread Pool Exhaustion**.
When Service C hangs for 30 seconds, the HTTP threads in Service B that called Service C are also blocked for 30 seconds. Quickly, all 200 Tomcat threads in Service B become blocked waiting for C. Since B is out of threads, Service A's requests to B start timing out, eventually exhausting all of A's threads too. The whole system goes down.

**How to fix it:**
1. **Circuit Breaker (Resilience4j):** I will implement a Circuit Breaker on Service B's call to C. Once it detects that C is timing out (e.g., 50% failure rate), it trips OPEN. Subsequent calls from B to C immediately fail-fast (in 1ms instead of 30s), freeing up B's threads immediately. 
2. **Aggressive Timeouts:** I will set a strict `ReadTimeout` (e.g., 2 seconds) on the `RestTemplate` or `WebClient`. Never rely on the default infinite timeout.
3. **Bulkhead Pattern:** Isolate resources so a failure in one part of the app doesn't consume all threads (e.g., dedicate only 20 threads to Service C calls)."

---

## 4. Kafka (Poison Pills & Rebalancing)

### 🔴 The Scenario
You have a Kafka Consumer reading messages from a topic. One day, a malformed message (missing a required field) arrives. Your consumer tries to parse it, throws a `NullPointerException`, and fails. The same message is read again, fails again. The partition gets stuck and consumer lag skyrockets.

### ⚠️ The "Knowing" Answer (Junior Level)
"I will wrap the processing logic in a try-catch. If it fails, I'll just print the error and ignore the message."

### 🏆 The "Mastering" Answer (Senior Level)
"What you have is a **Poison Pill**. If you don't catch it and commit the offset, Kafka will keep redelivering the same message, causing an infinite loop that halts the partition.

**How to fix it:**
1. Wrapping it in a `try-catch` and logging it *does* move the offset forward, but **silently dropping data in production is unacceptable.**
2. **Dead Letter Queue (DLQ):** I will configure a Spring Kafka `DefaultErrorHandler` with a `DeadLetterPublishingRecoverer`. If a message fails (after a configured number of retries, say 3 times with backoff), Spring will automatically forward the Poison Pill to a `topic-name.DLT` (Dead Letter Topic) and commit the offset on the main topic.
3. Our operations team can then inspect the DLT, fix the bug in the code, and eventually replay those failed messages."

---

## 5. Java Concurrency (Thread Pool Exhaustion)

### 🔴 The Scenario
You built a background service using `@Async` to send welcome emails. It works perfectly for 100 users. One day, a marketing campaign brings in 10,000 new users at once. The application crashes with an `OutOfMemoryError` (OOM).

### ⚠️ The "Knowing" Answer (Junior Level)
"I will increase the JVM heap size using `-Xmx` or switch to a better Garbage Collector."

### 🏆 The "Mastering" Answer (Senior Level)
"The OOM is likely caused by an **Unbounded Queue in the Thread Pool**.
By default, Spring's `@Async` uses a `SimpleAsyncTaskExecutor`, which does not reuse threads. It spawns a brand new thread for every single task! 10,000 users = 10,000 threads. Each thread consumes ~1MB of RAM natively, which instantly kills the JVM.

Even if you defined a custom `ThreadPoolTaskExecutor`, if you didn't configure a queue capacity, the default queue is `Integer.MAX_VALUE`. 10,000 email tasks get dumped into the memory queue, causing an OOM.

**How to fix it:**
I will explicitly configure a `ThreadPoolTaskExecutor`:
- `CorePoolSize`: 10
- `MaxPoolSize`: 50
- `QueueCapacity`: 500
- **Rejection Policy:** `CallerRunsPolicy`. If 550 emails hit the system at once, the 551st email won't crash the app or get dropped. The thread that submitted the task (the HTTP thread) will be forced to send the email itself, creating natural backpressure."

---

## 6. Security (JWT Invalidation)

### 🔴 The Scenario
You implemented stateless authentication using JWT. A user reports that their laptop was stolen while logged in. They want you to log them out immediately from all devices. 

### ⚠️ The "Knowing" Answer (Junior Level)
"I will delete the JWT token from the browser's local storage."

### 🏆 The "Mastering" Answer (Senior Level)
"Deleting the token from the browser doesn't invalidate it on the server. Because JWTs are **stateless**, the server doesn't keep a record of active tokens. As long as the thief has the token string and it hasn't expired, they can use it.

**How to fix it (Since JWTs cannot be revoked natively):**
1. **Short Expiration + Refresh Tokens:** I will make the JWT lifespan very short (e.g., 15 minutes) and issue a stateful Refresh Token (stored in the database). When the user reports the theft, I revoke the Refresh Token in the database. The thief's JWT will expire in max 15 minutes, and they won't be able to get a new one.
2. **Token Blacklist:** If immediate revocation is critical (0 minutes tolerance), I will implement a Blacklist using Redis. When a user is compromised, I store their `jti` (JWT ID) or user ID in Redis. My Spring Security filter will check Redis on every request. This makes the system slightly stateful, but with Redis, the latency hit is negligible."

---

## 7. Spring Boot Internals (Circular Dependencies)

### 🔴 The Scenario
Service A injects Service B. Service B injects Service A. The application refuses to start, throwing a `BeanCurrentlyInCreationException`. 

### ⚠️ The "Knowing" Answer (Junior Level)
"I will use `@Lazy` on one of the autowired fields, or use setter injection instead of constructor injection."

### 🏆 The "Mastering" Answer (Senior Level)
"While `@Lazy` or Setter/Field injection will technically bypass the startup error, **it is a code smell and an architectural flaw.** Circular dependencies indicate a violation of the Single Responsibility Principle. 

Spring Boot 2.6+ disabled circular dependencies by default for this exact reason.

**How to fix it:**
I will not use `@Lazy`. Instead, I will refactor the code.
If A needs B, and B needs A, it usually means there is a shared piece of logic they both rely on. I will extract that shared logic into a new, independent **Service C**. Then, both Service A and Service B will inject Service C, completely eliminating the cycle and improving the domain architecture."

---

> **Back to Root:** [Spring Boot Mastery 🚀](../../README.md)
