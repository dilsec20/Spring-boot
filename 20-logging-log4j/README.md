# 📝 Logging & Log4j — Complete In-Depth Guide

> **"Logging is the eyes and ears of your application in production. Without proper logging, debugging is guesswork."**

---

## 📑 Table of Contents

1. [Why Logging Matters](#1-why-logging-matters)
2. [Java Logging Landscape](#2-java-logging-landscape)
3. [SLF4J (Logging Facade)](#3-slf4j-logging-facade)
4. [Logback (Spring Boot Default)](#4-logback-spring-boot-default)
5. [Log Levels](#5-log-levels)
6. [Logging Configuration](#6-logging-configuration)
7. [Logback Configuration (logback-spring.xml)](#7-logback-configuration-logback-springxml)
8. [Log4j2 Configuration](#8-log4j2-configuration)
9. [Structured Logging (JSON)](#9-structured-logging-json)
10. [MDC (Mapped Diagnostic Context)](#10-mdc-mapped-diagnostic-context)
11. [Logging Best Practices](#11-logging-best-practices)
12. [Common Patterns & Anti-Patterns](#12-common-patterns--anti-patterns)
13. [ELK Stack (Elasticsearch, Logstash, Kibana)](#13-elk-stack)
14. [Interview Questions & Answers (50+)](#14-interview-questions--answers-50)

---

## 1. Why Logging Matters

```
WITHOUT proper logging:
  Production error → "Something broke" → Spend hours guessing 😩

WITH proper logging:
  Production error → Check logs → "NullPointerException at UserService:42, 
  userId=null, requestId=abc123" → Fixed in 5 minutes! ✅
```

---

## 2. Java Logging Landscape

```
FACADE (Interface):           IMPLEMENTATION (Actual logging):
┌─────────────┐              ┌─────────────────────┐
│   SLF4J     │──────────────│  Logback (default)  │  ← Spring Boot default ✅
│   (API)     │              │  Log4j2             │  ← High performance
│             │              │  java.util.logging  │  ← JDK built-in
└─────────────┘              └─────────────────────┘

SLF4J = Simple Logging Facade for Java
  You code to SLF4J → Swap implementations without code changes!

Spring Boot uses: SLF4J + Logback (by default)
```

---

## 3. SLF4J (Logging Facade)

```java
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

@Service
public class UserService {
    
    // Create logger for this class
    private static final Logger log = LoggerFactory.getLogger(UserService.class);
    
    // Or with Lombok: @Slf4j on the class (auto-generates the above line)
    
    public User findById(Long id) {
        log.trace("Entering findById with id={}", id);     // Most detailed
        log.debug("Looking up user with id={}", id);       // Debug details
        log.info("Fetching user id={}", id);               // Normal operations
        
        User user = userRepository.findById(id).orElse(null);
        
        if (user == null) {
            log.warn("User not found: id={}", id);         // Warning
            throw new ResourceNotFoundException("User not found");
        }
        
        log.info("Found user: id={}, name={}", user.getId(), user.getName());
        return user;
    }
    
    public void deleteUser(Long id) {
        try {
            userRepository.deleteById(id);
            log.info("Deleted user id={}", id);
        } catch (Exception e) {
            log.error("Failed to delete user id={}", id, e);  // Error + stack trace
            throw e;
        }
    }
}
```

### Parameterized Logging (Important!)

```java
// ✅ CORRECT: Parameterized logging (lazy evaluation)
log.debug("Processing user: id={}, name={}", userId, userName);
// If DEBUG is disabled, toString() is NEVER called → no performance cost!

// ❌ WRONG: String concatenation (always evaluated)
log.debug("Processing user: id=" + userId + ", name=" + userName);
// String concatenation happens even if DEBUG is disabled → wasted CPU!

// ✅ For expensive operations, check level first:
if (log.isDebugEnabled()) {
    log.debug("Full user details: {}", expensiveToString(user));
}
```

---

## 4. Logback (Spring Boot Default)

Spring Boot uses Logback automatically. No extra dependencies needed.

```
Default log format:
2024-01-15 10:30:45.123  INFO 12345 --- [main] c.e.myapp.UserService : Found user: id=1, name=Dilip
│          │              │     │        │      │                       │
│          │              │     │        │      │                       └── Message
│          │              │     │        │      └── Logger name (class)
│          │              │     │        └── Thread name
│          │              │     └── PID (Process ID)
│          │              └── Log Level
│          └── Timestamp
└── Date
```

---

## 5. Log Levels

```
TRACE  ─── Most detailed (method entry/exit, variable values)
  │
DEBUG  ─── Development details (queries, cache hits)
  │
INFO   ─── Normal operations (startup, user actions, milestones)  ← Default
  │
WARN   ─── Potential problems (deprecated API, retry, near limit)
  │
ERROR  ─── Failures that need attention (exceptions, data loss)
  │
FATAL  ─── Application can't continue (Log4j2 only, not in SLF4J)

Each level includes all levels ABOVE it:
  Level=INFO → Shows INFO + WARN + ERROR
  Level=DEBUG → Shows DEBUG + INFO + WARN + ERROR
  Level=TRACE → Shows EVERYTHING

Production:  INFO (or WARN for noisy services)
Development: DEBUG (or TRACE for deep debugging)
```

---

## 6. Logging Configuration

### application.properties

```properties
# ═══ Basic Configuration ═══

# Root log level (applies to ALL loggers)
logging.level.root=INFO

# Package-specific levels
logging.level.com.example.myapp=DEBUG
logging.level.com.example.myapp.repository=TRACE
logging.level.org.springframework.web=DEBUG
logging.level.org.hibernate.SQL=DEBUG
logging.level.org.hibernate.orm.jdbc.bind=TRACE

# ═══ Log File ═══
logging.file.name=logs/application.log
logging.file.max-size=10MB
logging.file.max-history=30
logging.file.total-size-cap=1GB

# ═══ Log Pattern ═══
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss.SSS} %clr(%-5level) [%thread] %logger{36} - %msg%n
logging.pattern.file=%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{36} - %msg%n

# ═══ Useful for debugging ═══
# Show SQL queries
logging.level.org.hibernate.SQL=DEBUG
# Show SQL parameter values
logging.level.org.hibernate.orm.jdbc.bind=TRACE
# Show Spring Security decisions
logging.level.org.springframework.security=DEBUG
# Show web request mapping
logging.level.org.springframework.web.servlet.mvc.method.annotation=TRACE
```

---

## 7. Logback Configuration (logback-spring.xml)

```xml
<!-- src/main/resources/logback-spring.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    
    <!-- ═══ CONSOLE APPENDER ═══ -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>
                %d{yyyy-MM-dd HH:mm:ss.SSS} %highlight(%-5level) [%thread] %cyan(%logger{36}) - %msg%n
            </pattern>
        </encoder>
    </appender>
    
    <!-- ═══ FILE APPENDER (Rolling) ═══ -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/application.log</file>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/application.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>1GB</totalSizeCap>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- ═══ ERROR-ONLY FILE ═══ -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/error.log</file>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <rollingPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedRollingPolicy">
            <fileNamePattern>logs/error.%d{yyyy-MM-dd}.%i.log.gz</fileNamePattern>
            <maxFileSize>10MB</maxFileSize>
            <maxHistory>90</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- ═══ PROFILE-SPECIFIC CONFIG ═══ -->
    <springProfile name="dev">
        <root level="DEBUG">
            <appender-ref ref="CONSOLE" />
        </root>
    </springProfile>
    
    <springProfile name="prod">
        <root level="INFO">
            <appender-ref ref="CONSOLE" />
            <appender-ref ref="FILE" />
            <appender-ref ref="ERROR_FILE" />
        </root>
    </springProfile>
    
    <!-- ═══ PACKAGE-SPECIFIC LEVELS ═══ -->
    <logger name="com.example.myapp" level="DEBUG" />
    <logger name="org.springframework.web" level="INFO" />
    <logger name="org.hibernate.SQL" level="DEBUG" />
    
</configuration>
```

---

## 8. Log4j2 Configuration

### Switch from Logback to Log4j2

```xml
<!-- pom.xml: Exclude Logback, add Log4j2 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-logging</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-log4j2</artifactId>
</dependency>
```

### log4j2-spring.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="WARN">
    <Properties>
        <Property name="LOG_PATTERN">%d{yyyy-MM-dd HH:mm:ss.SSS} %-5level [%thread] %logger{36} - %msg%n</Property>
    </Properties>
    
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <PatternLayout pattern="${LOG_PATTERN}" />
        </Console>
        
        <RollingFile name="File" fileName="logs/app.log"
                     filePattern="logs/app-%d{yyyy-MM-dd}-%i.log.gz">
            <PatternLayout pattern="${LOG_PATTERN}" />
            <Policies>
                <SizeBasedTriggeringPolicy size="10MB" />
                <TimeBasedTriggeringPolicy interval="1" />
            </Policies>
            <DefaultRolloverStrategy max="30" />
        </RollingFile>
    </Appenders>
    
    <Loggers>
        <Logger name="com.example.myapp" level="DEBUG" />
        <Logger name="org.springframework" level="INFO" />
        
        <Root level="INFO">
            <AppenderRef ref="Console" />
            <AppenderRef ref="File" />
        </Root>
    </Loggers>
</Configuration>
```

### Logback vs Log4j2

| Feature | Logback | Log4j2 |
|---------|---------|--------|
| Spring Boot default | ✅ Yes | No (must configure) |
| Performance | Good | Better (async, garbage-free) |
| Async logging | Supported | Built-in (LMAX Disruptor) |
| Configuration | logback.xml | log4j2.xml |
| Hot reload | Yes | Yes |

---

## 9. Structured Logging (JSON)

```xml
<!-- JSON logging for ELK/cloud (Logback) -->
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>7.4</version>
</dependency>
```

```xml
<appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
    <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        <includeMdcKeyName>requestId</includeMdcKeyName>
        <includeMdcKeyName>userId</includeMdcKeyName>
    </encoder>
</appender>
```

```json
// Output:
{
    "@timestamp": "2024-01-15T10:30:45.123Z",
    "level": "INFO",
    "thread_name": "http-nio-8080-exec-1",
    "logger_name": "com.example.UserService",
    "message": "User created: id=42",
    "requestId": "abc-123-def",
    "userId": "dilip@mail.com"
}
```

---

## 10. MDC (Mapped Diagnostic Context)

```java
// MDC adds context to ALL log messages in a thread
// Perfect for request tracing!

@Component
public class RequestIdFilter extends OncePerRequestFilter {
    
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                     HttpServletResponse response,
                                     FilterChain chain) throws ServletException, IOException {
        
        String requestId = UUID.randomUUID().toString().substring(0, 8);
        
        MDC.put("requestId", requestId);
        MDC.put("userId", getCurrentUserId());
        MDC.put("clientIP", request.getRemoteAddr());
        
        response.setHeader("X-Request-Id", requestId);
        
        try {
            chain.doFilter(request, response);
        } finally {
            MDC.clear();  // ALWAYS clear to prevent memory leaks!
        }
    }
}

// Now ALL log messages in this request automatically include requestId:
// 2024-01-15 10:30:45 INFO [abc123] UserService - Creating user
// 2024-01-15 10:30:45 INFO [abc123] EmailService - Sending welcome email
// 2024-01-15 10:30:46 ERROR [abc123] UserService - Failed to create user

// Pattern: %d %-5level [%X{requestId}] %logger - %msg%n
```

---

## 11. Logging Best Practices

```java
// 1. Use parameterized messages
log.info("User {} logged in from {}", username, ipAddress);  // ✅
log.info("User " + username + " logged in");                  // ❌

// 2. Log at the RIGHT level
log.trace("Variable x = {}", x);           // Very detailed debugging
log.debug("Query returned {} results", count); // Development
log.info("User {} registered", email);       // Business events
log.warn("Retry attempt {} for service {}", attempt, service);
log.error("Failed to process order {}", orderId, exception); // Include stack trace

// 3. Include context
log.info("Order processed: orderId={}, userId={}, amount={}", orderId, userId, amount);
// Not: log.info("Order processed");  ← useless!

// 4. Log exceptions properly
try { ... } catch (Exception e) {
    log.error("Failed to send email to {}", email, e);  // ✅ Pass exception as LAST arg
    // NOT: log.error("Failed: " + e.getMessage());      // ❌ Loses stack trace!
}

// 5. Don't log sensitive data
log.info("User login: email={}", email);           // ✅
log.info("User login: password={}", password);     // ❌ NEVER!
log.info("Payment: card=****{}", last4Digits);     // ✅ Masked

// 6. Don't log in loops
for (User user : users) {
    // log.debug("Processing user {}", user);  // ❌ 10000 log lines!
}
log.info("Processed {} users", users.size());       // ✅ One summary line
```

---

## 12. Common Patterns & Anti-Patterns

```java
// ❌ Anti-pattern: System.out.println
System.out.println("User created");  // No level, no timestamp, no class!

// ❌ Anti-pattern: Logging and throwing
try { ... } catch (Exception e) {
    log.error("Error", e);
    throw e;  // Double-logged! Caller will also log it.
}
// ✅ Fix: Either log OR throw, not both

// ❌ Anti-pattern: Empty catch
try { ... } catch (Exception e) {
    // Swallowed! Bug hiding in plain sight.
}
// ✅ Fix: At minimum log.warn

// ❌ Anti-pattern: toString() in log
log.debug("User: " + user.toString());  // toString() always called!
// ✅ Fix: log.debug("User: {}", user);

// ✅ Pattern: Aspect-based logging
@Aspect
@Component
public class LoggingAspect {
    @Around("execution(* com.example.service.*.*(..))")
    public Object logMethod(ProceedingJoinPoint pjp) throws Throwable {
        log.info("→ {}.{}()", pjp.getTarget().getClass().getSimpleName(),
                pjp.getSignature().getName());
        long start = System.currentTimeMillis();
        Object result = pjp.proceed();
        log.info("← {}.{}() took {}ms", pjp.getTarget().getClass().getSimpleName(),
                pjp.getSignature().getName(), System.currentTimeMillis() - start);
        return result;
    }
}
```

---

## 13. ELK Stack

```
ELK = Elasticsearch + Logstash + Kibana

Application → JSON logs → Logstash → Elasticsearch → Kibana (Dashboard)

Elasticsearch: Stores and indexes logs (search engine)
Logstash:      Collects, transforms, and ships logs
Kibana:        Visualizes logs (dashboards, queries)

Alternative: EFK (Elasticsearch + Fluentd + Kibana)
Cloud: AWS CloudWatch, Google Cloud Logging, Datadog
```

---

## 14. Interview Questions & Answers (50+)

### Beginner

**Q1: Why is logging important?** Debugging production issues, auditing, monitoring, performance analysis. Without logs, you're blind.

**Q2: What is SLF4J?** Simple Logging Facade for Java. API that abstracts logging implementations. Code to SLF4J, swap Logback/Log4j2 freely.

**Q3: What is Logback?** Logging implementation. Spring Boot default. Successor to Log4j. Created by same author (Ceki Gülcü).

**Q4: What are the log levels?** TRACE < DEBUG < INFO < WARN < ERROR. Each level includes all higher levels.

**Q5: What level for production?** INFO or WARN. DEBUG/TRACE only for specific packages when debugging.

**Q6: What is Log4j2?** Logging implementation by Apache. Better async performance than Logback.

**Q7: How to change log level in Spring Boot?** `logging.level.com.example=DEBUG` in application.properties.

**Q8: What is a log appender?** Destination for log output: Console, File, Database, Network (ELK).

---

### Intermediate

**Q9: What is MDC?** Mapped Diagnostic Context. Thread-local map adding context (requestId, userId) to all log messages automatically.

**Q10: Why use parameterized logging?** `log.info("x={}", x)` is lazy — toString() only called if level is enabled. String concatenation always evaluates.

**Q11: What is rolling file appender?** Creates new log file based on size or time. Archives old files. Prevents disk full.

**Q12: What is structured logging?** JSON format instead of plain text. Easier to parse, search, and analyze in tools like ELK.

**Q13: SLF4J vs Log4j2?** SLF4J is a facade (API). Log4j2 is an implementation. You use SLF4J API with Log4j2 implementation.

**Q14: How to log SQL queries in Spring Boot?** `logging.level.org.hibernate.SQL=DEBUG` for queries, `logging.level.org.hibernate.orm.jdbc.bind=TRACE` for parameters.

**Q15: What is async logging?** Log messages are queued and written by a background thread. Prevents logging from slowing down the application.

---

### Rapid-Fire (Q16–Q50)

**Q16: Default Spring Boot logging framework?** SLF4J + Logback.

**Q17: @Slf4j annotation from?** Lombok. Generates `private static final Logger log = LoggerFactory.getLogger(...)`.

**Q18: Log to file in Spring Boot?** `logging.file.name=app.log` in application.properties.

**Q19: logback.xml vs logback-spring.xml?** logback-spring.xml supports Spring profiles (`<springProfile>`). Preferred.

**Q20: What is log4j2-spring.xml?** Log4j2 config with Spring profile support.

**Q21: How to exclude Logback?** Exclude `spring-boot-starter-logging` from starter dependency.

**Q22: What is LoggerFactory?** SLF4J factory to create Logger instances.

**Q23: What is log.isDebugEnabled()?** Check if DEBUG is active before expensive operations.

**Q24: What is %d in pattern?** Date/time pattern.

**Q25: What is %level?** Log level (INFO, DEBUG, etc.).

**Q26: What is %logger?** Logger name (usually class name).

**Q27: What is %msg?** Log message.

**Q28: What is %n?** Newline.

**Q29: What is %thread?** Thread name.

**Q30: What is %X{key}?** MDC value for given key.

**Q31: Max log file size best practice?** 10-100MB per file. Roll on size or daily.

**Q32: How long to keep logs?** 30-90 days typically. Depends on compliance requirements.

**Q33: What is Logstash?** Log collection and transformation pipeline. Part of ELK stack.

**Q34: What is Kibana?** Log visualization dashboard. Query and visualize logs from Elasticsearch.

**Q35: What is correlation ID?** Unique ID tracking a request across multiple services. Set in MDC.

**Q36: What is GC logging?** JVM garbage collection logs. Enabled with `-Xlog:gc`.

**Q37: What is log aggregation?** Collecting logs from multiple services into one place (ELK, Splunk).

**Q38: Access log vs application log?** Access log: HTTP requests (Tomcat). Application log: your code's log statements.

**Q39: How to change log level at runtime?** Spring Boot Actuator: POST `/actuator/loggers/com.example` with `{"configuredLevel":"DEBUG"}`.

**Q40: What is Sentry?** Error tracking platform. Captures exceptions with context.

**Q41: What is Grafana Loki?** Log aggregation system. Like ELK but lighter.

**Q42: Should you log in catch block?** Only if you're handling the exception. If re-throwing, don't log (avoid duplicates).

**Q43: What is audit logging?** Recording WHO did WHAT and WHEN for compliance/security.

**Q44: What is %highlight in Logback?** Colors log output by level in console.

**Q45: What is additive="false" in Logback?** Prevents logger from inheriting parent's appenders (avoids duplicate logs).

**Q46: What is a filter in logging?** Includes/excludes log messages based on criteria (level, marker, MDC).

**Q47: What is a Marker in SLF4J?** Named tag for categorizing log messages: `log.info(SECURITY, "Login attempt")`.

**Q48: Synchronous vs async logging?** Sync: blocks until written. Async: queues message, writes in background.

**Q49: What is LMAX Disruptor?** High-performance inter-thread messaging library used by Log4j2 for async logging.

**Q50: Log sensitive data compliance?** GDPR/HIPAA prohibit logging PII. Mask or exclude sensitive fields.

---

## 📚 References

- [Spring Boot Logging](https://docs.spring.io/spring-boot/docs/current/reference/html/features.html#features.logging)
- [Logback Documentation](https://logback.qos.ch/documentation.html)
- [Log4j2 Documentation](https://logging.apache.org/log4j/2.x/)
- [SLF4J Manual](https://www.slf4j.org/manual.html)

---

> **Previous Topic:** [← 19 - OAuth2](../19-oauth2/README.md)  
> **Next Topic:** [21 - Spring Boot MongoDB →](../21-springboot-mongodb/README.md)
