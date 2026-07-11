# 🐬 Spring Boot + MySQL — Complete In-Depth Guide

> **"MySQL is the world's most popular open-source relational database. Combined with Spring Data JPA & Hibernate, it powers the majority of enterprise Java applications."**

---

## 📑 Table of Contents

1. [Introduction to MySQL](#1-introduction-to-mysql)
2. [SQL vs NoSQL Recap](#2-sql-vs-nosql-recap)
3. [MySQL Core Concepts](#3-mysql-core-concepts)
4. [Setup & Configuration](#4-setup--configuration)
5. [Entity Model & JPA Annotations](#5-entity-model--jpa-annotations)
6. [JpaRepository (CRUD)](#6-jparepository-crud)
7. [Query Methods (Derived)](#7-query-methods-derived)
8. [Custom Queries (@Query — JPQL & Native)](#8-custom-queries-query--jpql--native)
9. [Relationships (One-To-One, One-To-Many, Many-To-Many)](#9-relationships)
10. [Pagination & Sorting](#10-pagination--sorting)
11. [Transactions](#11-transactions)
12. [Indexing & Performance](#12-indexing--performance)
13. [Schema Management (DDL Auto, Flyway, Liquibase)](#13-schema-management)
14. [Connection Pooling (HikariCP)](#14-connection-pooling-hikaricp)
15. [Stored Procedures & Native Queries](#15-stored-procedures--native-queries)
16. [Auditing (Created/Updated timestamps)](#16-auditing)
17. [Best Practices](#17-best-practices)
18. [Interview Questions & Answers (50+)](#18-interview-questions--answers-50)

---

## 1. Introduction to MySQL

```
MySQL is a RELATIONAL database:
  - Stores data in structured TABLES with rows & columns
  - Strict schema (DDL: CREATE TABLE, ALTER TABLE)
  - ACID-compliant transactions
  - Powerful JOINs for related data
  - Vertically scalable (bigger server) + read replicas

Key terminology:
  Database             → Schema / Database
  Table                → Collection of rows with fixed columns
  Row                  → Single record
  Column               → Field with defined data type
  PRIMARY KEY          → Unique identifier for each row
  FOREIGN KEY          → Reference to another table's primary key
  INDEX                → Speed up lookups on columns
  JOIN                 → Combine rows from multiple tables
```

---

## 2. SQL vs NoSQL Recap

| Feature | SQL (MySQL) | NoSQL (MongoDB) |
|---------|-------------|-----------------|
| Schema | Fixed (ALTER TABLE to change) | Flexible (add fields anytime) |
| Data model | Tables + Rows + Columns | Documents (JSON) |
| Joins | Built-in (JOIN, subquery) | Embedded docs or $lookup |
| Transactions | Full ACID (default) | ACID (since v4.0, replica set) |
| Scaling | Vertical + read replicas | Horizontal (sharding) |
| Query | SQL (standard) | MongoDB Query Language |
| Best for | Complex relationships, reporting | Flexible, nested, high-volume data |
| Examples | Banking, ERP, E-commerce | Content mgmt, IoT, catalogs |

---

## 3. MySQL Core Concepts

```sql
-- DATABASE
CREATE DATABASE mydb;
USE mydb;

-- TABLE (strict schema)
CREATE TABLE users (
    id          BIGINT AUTO_INCREMENT PRIMARY KEY,
    name        VARCHAR(100) NOT NULL,
    email       VARCHAR(150) UNIQUE NOT NULL,
    age         INT,
    active      BOOLEAN DEFAULT TRUE,
    created_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at  TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- INDEX
CREATE INDEX idx_users_email ON users(email);

-- INSERT
INSERT INTO users (name, email, age) VALUES ('Dilip', 'dilip@mail.com', 25);

-- SELECT with JOIN
SELECT u.name, o.product, o.amount
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.active = TRUE;

-- Data types cheat sheet:
-- INT, BIGINT, SMALLINT, TINYINT       → Integers
-- DECIMAL(10,2), FLOAT, DOUBLE         → Decimals
-- VARCHAR(n), TEXT, LONGTEXT            → Strings
-- DATE, TIME, DATETIME, TIMESTAMP      → Date/Time
-- BOOLEAN (TINYINT(1))                 → True/False
-- BLOB, LONGBLOB                       → Binary data
-- JSON                                 → JSON (MySQL 5.7+)
-- ENUM('A','B','C')                    → Fixed choices
```

---

## 4. Setup & Configuration

```xml
<!-- pom.xml -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>
```

```properties
# application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# JPA / Hibernate
spring.jpa.hibernate.ddl-auto=update
# Options: none | validate | update | create | create-drop
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect

# Connection pool (HikariCP — default in Spring Boot)
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000

# Docker run:
# docker run -d -p 3306:3306 --name mysql -e MYSQL_ROOT_PASSWORD=password -e MYSQL_DATABASE=mydb mysql:8
```

---

## 5. Entity Model & JPA Annotations

```java
@Entity                                   // Marks class as JPA entity (maps to table)
@Table(name = "users",                    // Custom table name
       uniqueConstraints = @UniqueConstraint(columnNames = "email"))
@Data @NoArgsConstructor @AllArgsConstructor
public class User {

    @Id                                   // Primary key
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // AUTO_INCREMENT
    private Long id;

    @Column(name = "user_name",           // Custom column name
            nullable = false,             // NOT NULL
            length = 100)                 // VARCHAR(100)
    private String name;

    @Column(unique = true, nullable = false, length = 150)
    private String email;

    private int age;

    @Column(columnDefinition = "BOOLEAN DEFAULT TRUE")
    private boolean active = true;

    @Transient                            // NOT saved to database
    private String tempCalculation;

    @Enumerated(EnumType.STRING)          // Store enum as VARCHAR
    private Role role;                    // Enum: ADMIN, USER, MODERATOR

    @Lob                                  // Large object (TEXT / BLOB)
    private String bio;

    @CreatedDate                          // Auto-set on creation (needs @EnableJpaAuditing)
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate                     // Auto-set on update
    private LocalDateTime updatedAt;
}

// Enum
public enum Role {
    ADMIN, USER, MODERATOR
}

// Enable auditing
@Configuration
@EnableJpaAuditing
public class JpaConfig { }
```

### ID Generation Strategies

```java
// IDENTITY — MySQL AUTO_INCREMENT (most common for MySQL)
@GeneratedValue(strategy = GenerationType.IDENTITY)

// SEQUENCE — Uses DB sequence (PostgreSQL, Oracle — NOT native MySQL)
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
@SequenceGenerator(name = "user_seq", sequenceName = "user_sequence", allocationSize = 1)

// TABLE — Uses a table to simulate sequences (portable but slow)
@GeneratedValue(strategy = GenerationType.TABLE)

// UUID — Generate UUID as primary key
@Id
@GeneratedValue(strategy = GenerationType.UUID)
private String id;

// AUTO — Let Hibernate pick (usually TABLE for MySQL)
@GeneratedValue(strategy = GenerationType.AUTO)
```

---

## 6. JpaRepository (CRUD)

```java
public interface UserRepository extends JpaRepository<User, Long> {
    // INHERITED (free!):
    // save(User)           → INSERT or UPDATE
    // findById(Long)       → Optional<User>
    // findAll()            → List<User>
    // findAll(Pageable)    → Page<User>
    // deleteById(Long)
    // count()
    // existsById(Long)
    // saveAll(List<User>)  → Batch insert/update
    // flush()              → Force sync to DB
}

// Service
@Service
@RequiredArgsConstructor
public class UserService {

    private final UserRepository userRepository;

    public User create(CreateUserDTO dto) {
        User user = new User();
        user.setName(dto.getName());
        user.setEmail(dto.getEmail());
        user.setAge(dto.getAge());
        user.setRole(dto.getRole());
        return userRepository.save(user);  // id auto-generated by MySQL
    }

    public User findById(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found: " + id));
    }

    public User update(Long id, UpdateUserDTO dto) {
        User user = findById(id);
        user.setName(dto.getName());
        user.setEmail(dto.getEmail());
        return userRepository.save(user);  // save() on existing entity = UPDATE
    }

    public void delete(Long id) {
        userRepository.deleteById(id);
    }

    public List<User> findAll() {
        return userRepository.findAll();
    }
}
```

---

## 7. Query Methods (Derived)

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // Spring Data derives SQL from method name!
    List<User> findByName(String name);
    // → SELECT * FROM users WHERE name = ?

    Optional<User> findByEmail(String email);
    // → SELECT * FROM users WHERE email = ?

    List<User> findByAgeGreaterThan(int age);
    // → SELECT * FROM users WHERE age > ?

    List<User> findByAgeBetween(int min, int max);
    // → SELECT * FROM users WHERE age BETWEEN ? AND ?

    List<User> findByNameContainingIgnoreCase(String keyword);
    // → SELECT * FROM users WHERE LOWER(name) LIKE LOWER('%keyword%')

    List<User> findByActiveTrue();
    // → SELECT * FROM users WHERE active = TRUE

    List<User> findByRole(Role role);
    // → SELECT * FROM users WHERE role = ?

    // Sorting & Limiting
    List<User> findTop5ByOrderByCreatedAtDesc();
    List<User> findByAgeGreaterThanOrderByNameAsc(int age);

    // Count & Exists
    long countByActive(boolean active);
    boolean existsByEmail(String email);

    // Delete
    void deleteByEmail(String email);

    // Multiple conditions
    List<User> findByNameAndAge(String name, int age);
    List<User> findByNameOrEmail(String name, String email);
    List<User> findByAgeGreaterThanAndActiveTrue(int age);

    // IN clause
    List<User> findByRoleIn(List<Role> roles);

    // NULL check
    List<User> findByBioIsNull();
    List<User> findByBioIsNotNull();
}
```

---

## 8. Custom Queries (@Query — JPQL & Native)

```java
public interface UserRepository extends JpaRepository<User, Long> {

    // ──── JPQL (entity-based, database-independent) ────

    @Query("SELECT u FROM User u WHERE u.name = :name")
    List<User> findByNameCustom(@Param("name") String name);

    @Query("SELECT u FROM User u WHERE u.age BETWEEN :min AND :max AND u.active = true")
    List<User> findActiveByAgeRange(@Param("min") int min, @Param("max") int max);

    @Query("SELECT u FROM User u WHERE LOWER(u.name) LIKE LOWER(CONCAT('%', :keyword, '%'))")
    List<User> searchByName(@Param("keyword") String keyword);

    // Projection (return specific fields)
    @Query("SELECT u.name, u.email FROM User u WHERE u.active = true")
    List<Object[]> findActiveUsersNameAndEmail();

    // DTO projection
    @Query("SELECT new com.example.dto.UserSummaryDTO(u.name, u.email) FROM User u WHERE u.active = true")
    List<UserSummaryDTO> findActiveUserSummaries();

    // Update (requires @Modifying + @Transactional)
    @Modifying
    @Transactional
    @Query("UPDATE User u SET u.name = :name WHERE u.id = :id")
    int updateName(@Param("id") Long id, @Param("name") String name);

    // Delete
    @Modifying
    @Transactional
    @Query("DELETE FROM User u WHERE u.active = false")
    int deleteInactiveUsers();


    // ──── NATIVE SQL (MySQL-specific) ────

    @Query(value = "SELECT * FROM users WHERE email = :email", nativeQuery = true)
    User findByEmailNative(@Param("email") String email);

    @Query(value = "SELECT * FROM users WHERE age > :age ORDER BY name LIMIT :limit",
           nativeQuery = true)
    List<User> findTopByAge(@Param("age") int age, @Param("limit") int limit);

    // Native with pagination
    @Query(value = "SELECT * FROM users WHERE active = true",
           countQuery = "SELECT COUNT(*) FROM users WHERE active = true",
           nativeQuery = true)
    Page<User> findActiveUsersNative(Pageable pageable);
}
```

---

## 9. Relationships

### One-To-One

```java
@Entity
@Table(name = "users")
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "profile_id", referencedColumnName = "id")
    private Profile profile;
}

@Entity
@Table(name = "profiles")
public class Profile {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String bio;
    private String avatarUrl;

    @OneToOne(mappedBy = "profile")   // Bidirectional — inverse side
    private User user;
}

// Table structure:
// users:    id | name | profile_id (FK)
// profiles: id | bio  | avatar_url
```

### One-To-Many / Many-To-One

```java
@Entity
@Table(name = "departments")
public class Department {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<User> users = new ArrayList<>();
}

@Entity
@Table(name = "users")
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @ManyToOne(fetch = FetchType.LAZY)         // LAZY is best practice
    @JoinColumn(name = "department_id")        // FK column in users table
    private Department department;
}

// Table structure:
// departments: id | name
// users:       id | name | department_id (FK → departments.id)
```

### Many-To-Many

```java
@Entity
@Table(name = "users")
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "user_roles",                           // Junction table
        joinColumns = @JoinColumn(name = "user_id"),   // FK to this entity
        inverseJoinColumns = @JoinColumn(name = "role_id")  // FK to other entity
    )
    private Set<Role> roles = new HashSet<>();
}

@Entity
@Table(name = "roles")
public class Role {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;

    @ManyToMany(mappedBy = "roles")   // Inverse side
    private Set<User> users = new HashSet<>();
}

// Table structure:
// users:      id | name
// roles:      id | name
// user_roles: user_id (FK) | role_id (FK)   ← junction table
```

### Cascade & Fetch Summary

```
CASCADE TYPES:
  CascadeType.PERSIST   → Save child when parent is saved
  CascadeType.MERGE     → Update child when parent is updated
  CascadeType.REMOVE    → Delete child when parent is deleted
  CascadeType.ALL       → All of the above + DETACH + REFRESH

FETCH TYPES:
  FetchType.EAGER   → Load related data immediately (avoid!)
  FetchType.LAZY    → Load related data on access (preferred)

  @ManyToOne  default: EAGER  ← Change to LAZY!
  @OneToOne   default: EAGER  ← Change to LAZY!
  @OneToMany  default: LAZY   ✅
  @ManyToMany default: LAZY   ✅
```

---

## 10. Pagination & Sorting

```java
// Repository — already has it from JpaRepository!
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByActive(boolean active, Pageable pageable);
    Slice<User> findByRole(Role role, Pageable pageable);  // Lighter than Page
}

// Service
public Page<User> findAll(int page, int size, String sortBy, String direction) {
    Sort sort = direction.equalsIgnoreCase("desc")
        ? Sort.by(sortBy).descending()
        : Sort.by(sortBy).ascending();

    Pageable pageable = PageRequest.of(page, size, sort);
    return userRepository.findAll(pageable);
}

// Multiple sort fields
Pageable pageable = PageRequest.of(0, 10,
    Sort.by("name").ascending().and(Sort.by("age").descending()));

// Controller
@GetMapping("/users")
public Page<User> getUsers(
    @RequestParam(defaultValue = "0") int page,
    @RequestParam(defaultValue = "10") int size,
    @RequestParam(defaultValue = "id") String sortBy,
    @RequestParam(defaultValue = "asc") String direction
) {
    return userService.findAll(page, size, sortBy, direction);
}

// Response includes: content, totalElements, totalPages, size, number, sort
```

---

## 11. Transactions

```java
// Spring @Transactional — wraps method in a DB transaction
// If exception → all changes rolled back

@Service
@RequiredArgsConstructor
public class OrderService {

    private final UserRepository userRepository;
    private final OrderRepository orderRepository;
    private final PaymentRepository paymentRepository;

    @Transactional   // All-or-nothing
    public void placeOrder(Long userId, OrderDTO dto) {
        User user = userRepository.findById(userId).orElseThrow();

        // 1. Create order
        Order order = new Order();
        order.setUser(user);
        order.setProduct(dto.getProduct());
        order.setAmount(dto.getAmount());
        orderRepository.save(order);

        // 2. Process payment
        Payment payment = new Payment();
        payment.setOrder(order);
        payment.setAmount(dto.getAmount());
        paymentRepository.save(payment);

        // 3. Update user
        user.setOrderCount(user.getOrderCount() + 1);
        userRepository.save(user);

        // If ANY step fails → ALL 3 saves rolled back!
    }

    @Transactional(readOnly = true)     // Optimization for read-only queries
    public List<User> getActiveUsers() {
        return userRepository.findByActiveTrue();
    }

    @Transactional(
        isolation = Isolation.READ_COMMITTED,   // Isolation level
        propagation = Propagation.REQUIRED,     // Default propagation
        timeout = 30,                           // Timeout in seconds
        rollbackFor = Exception.class           // Rollback on checked exceptions too
    )
    public void complexOperation() { ... }
}

// PROPAGATION TYPES:
// REQUIRED (default) — Join existing or create new
// REQUIRES_NEW       — Always create new (suspend existing)
// NESTED             — Nested transaction with savepoint
// SUPPORTS           — Use existing if available, else non-transactional
// NOT_SUPPORTED      — Always non-transactional
// MANDATORY          — Must have existing transaction or throw
// NEVER              — Must NOT have existing transaction or throw

// ISOLATION LEVELS:
// READ_UNCOMMITTED   — Dirty reads possible
// READ_COMMITTED     — No dirty reads (MySQL default)
// REPEATABLE_READ    — No non-repeatable reads (InnoDB default)
// SERIALIZABLE       — Full isolation (slowest)
```

---

## 12. Indexing & Performance

```sql
-- Single column index
CREATE INDEX idx_users_name ON users(name);

-- Composite (multi-column) index
CREATE INDEX idx_users_name_age ON users(name, age);

-- Unique index
CREATE UNIQUE INDEX idx_users_email ON users(email);

-- Full-text index (for text search)
CREATE FULLTEXT INDEX idx_users_bio ON users(bio);

-- Check indexes
SHOW INDEX FROM users;

-- Analyze query performance
EXPLAIN SELECT * FROM users WHERE name = 'Dilip';
EXPLAIN ANALYZE SELECT * FROM users WHERE age > 25 ORDER BY name;
```

```java
// JPA index annotations
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_name", columnList = "name"),
    @Index(name = "idx_email", columnList = "email", unique = true),
    @Index(name = "idx_name_age", columnList = "name, age")  // Composite
})
public class User {
    // ...
}
```

### N+1 Problem & Solutions

```java
// PROBLEM: Loading 100 users → 1 query for users + 100 queries for departments!
List<User> users = userRepository.findAll();   // SELECT * FROM users
for (User u : users) {
    u.getDepartment().getName();                // SELECT * FROM departments WHERE id = ? (×100!)
}

// SOLUTION 1: JOIN FETCH (JPQL)
@Query("SELECT u FROM User u JOIN FETCH u.department WHERE u.active = true")
List<User> findActiveUsersWithDepartment();

// SOLUTION 2: @EntityGraph
@EntityGraph(attributePaths = {"department"})
List<User> findByActiveTrue();

// SOLUTION 3: @BatchSize (Hibernate)
@ManyToOne(fetch = FetchType.LAZY)
@BatchSize(size = 25)   // Load 25 departments at a time instead of 1
private Department department;
```

---

## 13. Schema Management

### DDL Auto (Development only!)

```properties
# NEVER use in production!
spring.jpa.hibernate.ddl-auto=update

# Options:
# none          → Do nothing (production)
# validate      → Validate schema matches entities (production)
# update        → Add new columns/tables (dev only — never drops!)
# create        → Drop & recreate every startup
# create-drop   → Drop on shutdown
```

### Flyway (Production migrations)

```xml
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-core</artifactId>
</dependency>
<dependency>
    <groupId>org.flywaydb</groupId>
    <artifactId>flyway-mysql</artifactId>
</dependency>
```

```properties
spring.flyway.enabled=true
spring.flyway.locations=classpath:db/migration
spring.jpa.hibernate.ddl-auto=none   # Let Flyway manage schema!
```

```sql
-- src/main/resources/db/migration/V1__create_users_table.sql
CREATE TABLE users (
    id         BIGINT AUTO_INCREMENT PRIMARY KEY,
    name       VARCHAR(100) NOT NULL,
    email      VARCHAR(150) UNIQUE NOT NULL,
    age        INT,
    active     BOOLEAN DEFAULT TRUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- V2__add_role_column.sql
ALTER TABLE users ADD COLUMN role VARCHAR(20) DEFAULT 'USER';

-- V3__create_orders_table.sql
CREATE TABLE orders (
    id         BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id    BIGINT NOT NULL,
    product    VARCHAR(200) NOT NULL,
    amount     DECIMAL(10,2) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

## 14. Connection Pooling (HikariCP)

```properties
# HikariCP is the DEFAULT connection pool in Spring Boot

spring.datasource.hikari.maximum-pool-size=10      # Max connections
spring.datasource.hikari.minimum-idle=5             # Min idle connections
spring.datasource.hikari.connection-timeout=30000   # Wait for connection (ms)
spring.datasource.hikari.idle-timeout=600000        # Close idle connections after (ms)
spring.datasource.hikari.max-lifetime=1800000       # Max connection lifetime (ms)
spring.datasource.hikari.pool-name=MyHikariPool     # Pool name for monitoring
spring.datasource.hikari.leak-detection-threshold=60000  # Detect connection leaks
```

```
WHY CONNECTION POOLING?
  Without pool: Open connection → query → close connection (SLOW! ~200ms overhead each time)
  With pool:    Reuse pre-opened connections from pool (< 1ms to acquire)

  ┌─────────────┐         ┌────────────────────────┐
  │ Application │ ──────► │ HikariCP Pool          │ ──────► MySQL
  │             │ ◄────── │ [conn1][conn2][conn3].. │ ◄────── Server
  └─────────────┘         └────────────────────────┘
```

---

## 15. Stored Procedures & Native Queries

```java
// Call stored procedure
@Entity
@NamedStoredProcedureQuery(
    name = "User.findByCity",
    procedureName = "sp_find_users_by_city",
    parameters = {
        @StoredProcedureParameter(name = "city", type = String.class, mode = ParameterMode.IN)
    }
)
public class User { ... }

// Repository
@Procedure(name = "User.findByCity")
List<User> findByCity(@Param("city") String city);

// Using EntityManager directly
@Repository
@RequiredArgsConstructor
public class UserCustomRepository {

    private final EntityManager entityManager;

    public List<User> callProcedure(String city) {
        StoredProcedureQuery query = entityManager
            .createStoredProcedureQuery("sp_find_users_by_city", User.class);
        query.registerStoredProcedureParameter("city", String.class, ParameterMode.IN);
        query.setParameter("city", city);
        return query.getResultList();
    }

    // Native query with EntityManager
    public List<User> findByAgeNative(int minAge) {
        return entityManager.createNativeQuery(
            "SELECT * FROM users WHERE age > :age ORDER BY name", User.class)
            .setParameter("age", minAge)
            .getResultList();
    }
}
```

---

## 16. Auditing

```java
// Automatic created/updated timestamps on all entities

@MappedSuperclass   // Not a table — base class for entities
@EntityListeners(AuditingEntityListener.class)
@Data
public abstract class BaseEntity {

    @CreatedDate
    @Column(updatable = false)
    private LocalDateTime createdAt;

    @LastModifiedDate
    private LocalDateTime updatedAt;

    @CreatedBy
    @Column(updatable = false)
    private String createdBy;

    @LastModifiedBy
    private String updatedBy;
}

// User entity extends BaseEntity
@Entity
@Table(name = "users")
public class User extends BaseEntity {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;
}

// Provide current user for @CreatedBy / @LastModifiedBy
@Configuration
@EnableJpaAuditing
public class JpaConfig {
    @Bean
    AuditorAware<String> auditorProvider() {
        return () -> Optional.ofNullable(
            SecurityContextHolder.getContext().getAuthentication())
            .map(Authentication::getName);
    }
}
```

---

## 17. Best Practices

1. **Always use `FetchType.LAZY`** — Change `@ManyToOne` and `@OneToOne` defaults to LAZY
2. **Avoid N+1** — Use `JOIN FETCH`, `@EntityGraph`, or `@BatchSize`
3. **Use DTOs for responses** — Don't expose entities directly in REST APIs
4. **Use Flyway/Liquibase** — Never use `ddl-auto=update` in production
5. **Index frequently queried columns** — Use `EXPLAIN` to analyze slow queries
6. **Use `@Transactional(readOnly = true)`** — For read-only methods (Hibernate optimization)
7. **Use pagination** — Never `findAll()` on large tables without `Pageable`
8. **Validate input** — Use `@Valid` + Bean Validation annotations
9. **Use connection pooling** — HikariCP (default) with proper pool size
10. **Soft delete over hard delete** — Add `active`/`deleted` flag instead of DELETE

---

## 18. Interview Questions & Answers (50+)

### Beginner

**Q1: What is MySQL?** Open-source relational database. Stores data in tables with rows and columns. Uses SQL for querying.

**Q2: MySQL vs MongoDB?** MySQL = SQL, fixed schema, JOINs, vertical scaling. MongoDB = NoSQL, flexible schema, embedded docs, horizontal scaling.

**Q3: What is JPA?** Java Persistence API — specification for ORM (Object-Relational Mapping). Hibernate is the default JPA implementation in Spring Boot.

**Q4: What is Hibernate?** ORM framework that implements JPA. Maps Java objects to database tables. Generates SQL automatically.

**Q5: What is `spring.jpa.hibernate.ddl-auto`?** Controls schema generation: `none`, `validate`, `update`, `create`, `create-drop`. Use `none` or `validate` in production.

**Q6: What is `@Entity`?** Marks a Java class as a JPA entity mapped to a database table.

**Q7: What is `@Id` and `@GeneratedValue`?** `@Id` marks primary key. `@GeneratedValue(strategy = GenerationType.IDENTITY)` uses MySQL's AUTO_INCREMENT.

**Q8: What is `JpaRepository`?** Spring Data interface providing CRUD, pagination, sorting. Extend it with your entity type and ID type.

---

### Intermediate

**Q9: What is the N+1 problem?** Loading N entities triggers N additional queries for lazy-loaded relationships. Fix: JOIN FETCH, @EntityGraph, @BatchSize.

**Q10: JPQL vs Native SQL?** JPQL operates on entities (portable). Native SQL operates on tables (MySQL-specific). Use JPQL unless you need MySQL-specific features.

**Q11: What is `@Transactional`?** Wraps method in a database transaction. If exception → rollback all changes. Use `readOnly = true` for queries.

**Q12: What is `CascadeType.ALL`?** Propagates all operations (persist, merge, remove, detach, refresh) from parent to child entity.

**Q13: `FetchType.LAZY` vs `EAGER`?** LAZY: load related data on access (preferred). EAGER: load immediately with parent (can cause performance issues).

**Q14: What is `@MappedSuperclass`?** Base class not mapped to its own table. Subclasses inherit its fields. Used for common fields like `createdAt`, `updatedAt`.

**Q15: What is HikariCP?** Default connection pool in Spring Boot. Reuses database connections for better performance.

**Q16: What is Flyway?** Database migration tool. Version-controlled SQL scripts (`V1__`, `V2__`) applied in order. Production-safe schema management.

---

### Advanced

**Q17: What is the difference between `save()` and `saveAndFlush()`?** `save()` queues changes in persistence context. `saveAndFlush()` immediately writes to DB. Use `saveAndFlush()` when you need the generated ID right away.

**Q18: What is the persistence context (first-level cache)?** Cache of managed entities within a transaction. Same entity loaded twice returns same instance. Flushed on transaction commit.

**Q19: What is second-level cache?** Shared cache across sessions. Hibernate supports Ehcache, Redis. Must be explicitly enabled. Use for read-heavy, rarely changing data.

**Q20: What is optimistic locking?** Uses `@Version` field. Throws `OptimisticLockException` if another transaction modified the entity. No database locks held.

**Q21: What is pessimistic locking?** Uses `SELECT ... FOR UPDATE`. Holds database lock until transaction completes. Use for high-contention scenarios.

```java
// Optimistic locking
@Entity
public class User {
    @Version
    private Long version;   // Auto-incremented on update
}

// Pessimistic locking
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT u FROM User u WHERE u.id = :id")
Optional<User> findByIdWithLock(@Param("id") Long id);
```

---

### Rapid-Fire (Q22–Q50)

**Q22: MySQL default port?** 3306.

**Q23: `@Column` annotation?** Customize column name, length, nullable, unique, columnDefinition.

**Q24: `@Table` annotation?** Customize table name, indexes, unique constraints.

**Q25: `@Enumerated(EnumType.STRING)`?** Store enum as readable string (VARCHAR). `ORDINAL` stores as integer (fragile — don't use).

**Q26: `@Lob`?** Maps to TEXT or BLOB for large data.

**Q27: `@Transient`?** Field NOT persisted to database.

**Q28: `@JoinColumn`?** Specifies the foreign key column in a relationship.

**Q29: `@JoinTable`?** Specifies the junction table for @ManyToMany.

**Q30: `mappedBy`?** Marks the inverse (non-owning) side of a bidirectional relationship.

**Q31: `@Modifying`?** Required with `@Query` for UPDATE/DELETE JPQL queries.

**Q32: `@Param`?** Binds method parameter to named parameter in `@Query`.

**Q33: `Page` vs `Slice`?** `Page` includes total count (extra COUNT query). `Slice` only knows if there's a next page (lighter).

**Q34: What is `EntityManager`?** JPA's core API for managing entities. Lower-level than `JpaRepository`.

**Q35: What is `@EntityGraph`?** Defines which associations to fetch eagerly for a specific query. Solves N+1 without JOIN FETCH.

**Q36: `spring.jpa.show-sql=true`?** Logs generated SQL to console. Use `format_sql=true` to pretty-print.

**Q37: What is `@NamedQuery`?** Pre-defined JPQL query on entity class. Validated at startup.

**Q38: What is `@SqlResultSetMapping`?** Maps native query results to entities or DTOs.

**Q39: What is dirty checking?** Hibernate automatically detects changes to managed entities and generates UPDATE on flush. No explicit `save()` needed inside `@Transactional`.

**Q40: What is `Specification`?** Spring Data's type-safe, dynamic query builder. Alternative to building JPQL strings.

**Q41: What is `@DataJpaTest`?** Test slice for JPA repositories. Auto-configures in-memory DB + EntityManager + repos.

**Q42: InnoDB vs MyISAM?** InnoDB: ACID, row-level locking, foreign keys (default). MyISAM: faster reads, table-level locking, no transactions (legacy).

**Q43: `VARCHAR` vs `TEXT`?** VARCHAR: up to 65535 bytes, indexed. TEXT: up to 64KB, not directly indexable (needs prefix index).

**Q44: What is a composite key?** Primary key spanning multiple columns. Use `@EmbeddedId` or `@IdClass` in JPA.

**Q45: What is `@Embeddable`?** Marks a class whose fields are embedded into the parent entity's table (not a separate table).

**Q46: What is `GenerationType.IDENTITY` vs `SEQUENCE`?** IDENTITY: MySQL AUTO_INCREMENT (insert to get ID). SEQUENCE: DB sequence object (PostgreSQL/Oracle, batch-friendly).

**Q47: How to use MySQL JSON column?** `@Column(columnDefinition = "JSON")` with `String` or a custom `AttributeConverter`.

**Q48: What is `spring.datasource.url`?** JDBC URL for MySQL connection: `jdbc:mysql://host:port/database`.

**Q49: What is `@Query(nativeQuery = true)`?** Executes raw SQL instead of JPQL. Use for MySQL-specific features.

**Q50: Testcontainers for MySQL?** Spins up real MySQL in Docker for integration tests. More accurate than H2.

```java
// Testcontainers example
@Testcontainers
@SpringBootTest
class UserRepositoryTest {
    @Container
    static MySQLContainer<?> mysql = new MySQLContainer<>("mysql:8")
        .withDatabaseName("testdb");

    @DynamicPropertySource
    static void props(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", mysql::getJdbcUrl);
        registry.add("spring.datasource.username", mysql::getUsername);
        registry.add("spring.datasource.password", mysql::getPassword);
    }
}
```

---

## 📚 References

- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/)
- [MySQL Documentation](https://dev.mysql.com/doc/)
- [Hibernate ORM Guide](https://hibernate.org/orm/documentation/)
- [Baeldung Spring JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)
- [Flyway Documentation](https://documentation.red-gate.com/flyway/)
- [HikariCP GitHub](https://github.com/brettwooldridge/HikariCP)

---

> **See also:** [MongoDB (NoSQL) Guide →](./README.md)
