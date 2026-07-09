# 🐻 Hibernate — Complete In-Depth Guide

> **"Hibernate is the most popular JPA implementation and the default ORM in Spring Boot. Mastering Hibernate means mastering Java persistence."**

---

## 📑 Table of Contents

1. [Introduction & Architecture](#1-introduction--architecture)
2. [Hibernate vs JPA](#2-hibernate-vs-jpa)
3. [Configuration & Setup](#3-configuration--setup)
4. [Session & SessionFactory](#4-session--sessionfactory)
5. [CRUD Operations](#5-crud-operations)
6. [HQL (Hibernate Query Language)](#6-hql-hibernate-query-language)
7. [Criteria API & Criteria Builder](#7-criteria-api--criteria-builder)
8. [Caching (First & Second Level)](#8-caching-first--second-level)
9. [Lazy Loading & Proxy](#9-lazy-loading--proxy)
10. [Locking (Optimistic & Pessimistic)](#10-locking-optimistic--pessimistic)
11. [Native SQL Queries](#11-native-sql-queries)
12. [Hibernate Validators](#12-hibernate-validators)
13. [Hibernate in Spring Boot](#13-hibernate-in-spring-boot)
14. [Performance Tuning](#14-performance-tuning)
15. [Common Exceptions & Solutions](#15-common-exceptions--solutions)
16. [Best Practices](#16-best-practices)
17. [Interview Questions & Answers (50+)](#17-interview-questions--answers-50)

---

## 1. Introduction & Architecture

### What is Hibernate?

Hibernate is a Java **ORM framework** that maps Java objects to database tables. It implements the JPA specification and adds extra features like caching, lazy loading, and HQL.

### Hibernate Architecture

```
┌──────────────────────────────────────────────────────┐
│                  Your Application                     │
│   Entity classes, DAO/Repository, Service layer       │
├──────────────────────────────────────────────────────┤
│                 Hibernate ORM                         │
│  ┌────────────────────────────────────────────────┐  │
│  │            SessionFactory                       │  │
│  │  (One per application — thread-safe, heavy)     │  │
│  │                                                 │  │
│  │  ┌──────────┐  ┌──────────┐  ┌──────────────┐ │  │
│  │  │ Session 1 │  │ Session 2 │  │  Session N   │ │  │
│  │  │(per req.) │  │(per req.) │  │ (per req.)   │ │  │
│  │  └──────────┘  └──────────┘  └──────────────┘ │  │
│  │                                                 │  │
│  │  ┌────────────────────────────────────────────┐│  │
│  │  │ First Level Cache (per Session)            ││  │
│  │  │ Second Level Cache (shared, optional)      ││  │
│  │  │ Query Cache (shared, optional)             ││  │
│  │  └────────────────────────────────────────────┘│  │
│  │                                                 │  │
│  │  ┌────────────────────────────────────────────┐│  │
│  │  │ Transaction Manager                        ││  │
│  │  │ Connection Pool (HikariCP)                 ││  │
│  │  └────────────────────────────────────────────┘│  │
│  └────────────────────────────────────────────────┘  │
├──────────────────────────────────────────────────────┤
│                    JDBC Driver                        │
├──────────────────────────────────────────────────────┤
│                    Database                           │
└──────────────────────────────────────────────────────┘
```

### Key Components

| Component | Description |
|-----------|-------------|
| **SessionFactory** | Thread-safe factory for Sessions. One per application. Heavy to create. |
| **Session** | Single-threaded, short-lived. Wraps JDBC Connection. First-level cache. |
| **Transaction** | Wraps database transaction. Begin, commit, rollback. |
| **Query** | HQL, JPQL, Criteria, or native SQL queries. |
| **Configuration** | Database URL, dialect, mappings, properties. |

---

## 2. Hibernate vs JPA

```
JPA = Standard (Interface)         Hibernate = Implementation

EntityManager         ←→          Session
EntityManagerFactory  ←→          SessionFactory
persist()            ←→          save() / persist()
merge()              ←→          update() / merge()
remove()             ←→          delete() / remove()
find()               ←→          get() / load()
JPQL                 ←→          HQL (superset of JPQL)
@Entity              ←→          @Entity (same)
@Id                  ←→          @Id (same)
```

```java
// JPA way (portable, standard):
@PersistenceContext
private EntityManager entityManager;

entityManager.persist(user);
User user = entityManager.find(User.class, 1L);

// Hibernate way (Hibernate-specific features):
Session session = entityManager.unwrap(Session.class);

session.save(user);
User user = session.get(User.class, 1L);
session.byNaturalId(User.class).using("email", "dilip@mail.com").load();
```

---

## 3. Configuration & Setup

### Spring Boot Configuration (application.properties)

```properties
# Database connection
spring.datasource.url=jdbc:mysql://localhost:3306/mydb?useSSL=false&serverTimezone=UTC
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# Hibernate / JPA
spring.jpa.hibernate.ddl-auto=update
# Options: none, validate, update, create, create-drop
# none:        No schema changes (production)
# validate:    Validate schema matches entities (production)
# update:      Auto-update schema (development)
# create:      Drop and create on startup
# create-drop: Drop on shutdown

spring.jpa.show-sql=true
spring.jpa.properties.hibernate.format_sql=true
spring.jpa.properties.hibernate.use_sql_comments=true

# Hibernate dialect (auto-detected in Spring Boot)
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect

# Connection pool (HikariCP — Spring Boot default)
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000

# Second-level cache
spring.jpa.properties.hibernate.cache.use_second_level_cache=true
spring.jpa.properties.hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory

# Batch operations
spring.jpa.properties.hibernate.jdbc.batch_size=30
spring.jpa.properties.hibernate.order_inserts=true
spring.jpa.properties.hibernate.order_updates=true

# Statistics (for debugging)
spring.jpa.properties.hibernate.generate_statistics=true
```

---

## 4. Session & SessionFactory

### Session Operations

```java
@Service
@Transactional
public class UserService {
    
    @PersistenceContext
    private EntityManager entityManager;
    
    // Get Hibernate Session from EntityManager
    private Session getSession() {
        return entityManager.unwrap(Session.class);
    }
    
    // save() — INSERT new entity, returns generated ID
    public Long saveUser(User user) {
        Session session = getSession();
        return (Long) session.save(user);  // Returns the generated ID
    }
    
    // get() vs load()
    public User getUser(Long id) {
        Session session = getSession();
        
        // get() — Hits database IMMEDIATELY, returns null if not found
        User user = session.get(User.class, id);
        
        // load() — Returns PROXY, hits DB only when accessed
        //          Throws ObjectNotFoundException if not found
        User proxy = session.load(User.class, id);
        // proxy.getName() ← THIS triggers the actual SQL query
        
        return user;
    }
    
    // update() — UPDATE detached entity
    public void updateUser(User detachedUser) {
        Session session = getSession();
        session.update(detachedUser);  // Reattach and update
    }
    
    // saveOrUpdate() — INSERT if new, UPDATE if existing
    public void saveOrUpdate(User user) {
        Session session = getSession();
        session.saveOrUpdate(user);
    }
    
    // delete()
    public void deleteUser(Long id) {
        Session session = getSession();
        User user = session.get(User.class, id);
        if (user != null) {
            session.delete(user);
        }
    }
}
```

### get() vs load() — Key Difference

```
get(User.class, 1):
  → Hits database IMMEDIATELY
  → Returns null if not found
  → Returns the actual User object
  → Use when you need the data right now

load(User.class, 1):
  → Returns a PROXY object (no DB hit yet!)
  → Hits database when you ACCESS a property
  → Throws ObjectNotFoundException if not found
  → Use when you only need the reference (e.g., for setting foreign key)

Example:
  Order order = new Order();
  order.setUser(session.load(User.class, userId)); 
  // load() is perfect here — we don't need user data,
  // just the reference to set the foreign key!
```

---

## 5. CRUD Operations

```java
@Repository
@Transactional
public class UserRepository {
    
    @PersistenceContext
    private EntityManager em;
    
    // CREATE
    public User save(User user) {
        em.persist(user);              // INSERT
        return user;                   // user.getId() is now set
    }
    
    // READ
    public User findById(Long id) {
        return em.find(User.class, id);  // SELECT * FROM users WHERE id = ?
    }
    
    // READ ALL
    public List<User> findAll() {
        return em.createQuery("SELECT u FROM User u", User.class)
                 .getResultList();
    }
    
    // UPDATE (managed entity — automatic dirty checking)
    public User update(Long id, String newName) {
        User user = em.find(User.class, id);  // Load (now managed)
        user.setName(newName);                 // Just change it!
        // NO em.merge() needed — Hibernate detects the change automatically
        // UPDATE statement is generated at flush/commit time
        return user;
    }
    
    // UPDATE (detached entity)
    public User updateDetached(User detachedUser) {
        return em.merge(detachedUser);  // Copy state, return managed copy
    }
    
    // DELETE
    public void delete(Long id) {
        User user = em.find(User.class, id);
        if (user != null) {
            em.remove(user);  // DELETE FROM users WHERE id = ?
        }
    }
}
```

---

## 6. HQL (Hibernate Query Language)

```java
// HQL uses ENTITY and FIELD names (not table/column names)

// Basic SELECT
List<User> users = session.createQuery("FROM User", User.class).getResultList();

// WHERE clause with parameters
List<User> users = session
    .createQuery("FROM User u WHERE u.name = :name AND u.active = :active", User.class)
    .setParameter("name", "Dilip")
    .setParameter("active", true)
    .getResultList();

// JOIN with relationship
List<User> users = session
    .createQuery("SELECT DISTINCT u FROM User u JOIN u.orders o WHERE o.amount > :min", User.class)
    .setParameter("min", 100.0)
    .getResultList();

// JOIN FETCH (solves N+1!)
List<User> users = session
    .createQuery("SELECT u FROM User u JOIN FETCH u.orders", User.class)
    .getResultList();

// Pagination
List<User> users = session
    .createQuery("FROM User u ORDER BY u.name", User.class)
    .setFirstResult(0)    // offset (skip first 0)
    .setMaxResults(20)    // limit (take 20)
    .getResultList();

// Aggregate functions
Long count = session
    .createQuery("SELECT COUNT(u) FROM User u", Long.class)
    .getSingleResult();

Double avgAmount = session
    .createQuery("SELECT AVG(o.amount) FROM Order o", Double.class)
    .getSingleResult();

// GROUP BY
List<Object[]> stats = session
    .createQuery("SELECT u.role, COUNT(u) FROM User u GROUP BY u.role", Object[].class)
    .getResultList();

for (Object[] row : stats) {
    System.out.println("Role: " + row[0] + ", Count: " + row[1]);
}

// UPDATE (bulk)
int updated = session
    .createQuery("UPDATE User u SET u.active = false WHERE u.lastLogin < :date")
    .setParameter("date", sixMonthsAgo)
    .executeUpdate();

// DELETE (bulk)
int deleted = session
    .createQuery("DELETE FROM User u WHERE u.active = false")
    .executeUpdate();

// DTO projection (avoid loading full entity)
List<UserDTO> dtos = session.createQuery(
    "SELECT new com.example.dto.UserDTO(u.id, u.name, u.email) FROM User u", UserDTO.class)
    .getResultList();
```

---

## 7. Criteria API & Criteria Builder

```java
// Type-safe queries built programmatically (good for dynamic queries)

@Repository
public class UserSearchRepository {
    
    @PersistenceContext
    private EntityManager em;
    
    public List<User> search(String name, String email, Integer minAge, String role) {
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<User> cq = cb.createQuery(User.class);
        Root<User> user = cq.from(User.class);
        
        List<Predicate> predicates = new ArrayList<>();
        
        // Dynamic conditions — only add if parameter is provided
        if (name != null) {
            predicates.add(cb.like(cb.lower(user.get("name")), "%" + name.toLowerCase() + "%"));
        }
        if (email != null) {
            predicates.add(cb.equal(user.get("email"), email));
        }
        if (minAge != null) {
            predicates.add(cb.greaterThanOrEqualTo(user.get("age"), minAge));
        }
        if (role != null) {
            predicates.add(cb.equal(user.get("role"), UserRole.valueOf(role)));
        }
        
        cq.select(user)
           .where(predicates.toArray(new Predicate[0]))
           .orderBy(cb.asc(user.get("name")));
        
        return em.createQuery(cq).getResultList();
    }
}
```

---

## 8. Caching (First & Second Level)

### First Level Cache (Session Cache)

```java
// AUTOMATIC — Every Session has its own first-level cache

Session session = sessionFactory.openSession();

// First query — hits database
User user1 = session.get(User.class, 1L);
// SQL: SELECT * FROM users WHERE id = 1

// Second query — same session, same ID — CACHE HIT! No SQL!
User user2 = session.get(User.class, 1L);
// No SQL executed! Returns cached object.

System.out.println(user1 == user2);  // true — SAME object in memory!

// First level cache is CLEARED when:
session.clear();           // Manual clear
session.evict(user);       // Remove specific entity
// Or when session is closed
```

```
First Level Cache:

Session 1 Cache:          Session 2 Cache:
┌──────────────────┐     ┌──────────────────┐
│ User(1) → {Dilip}│     │ User(1) → {Dilip}│  ← Separate copy!
│ User(2) → {Alice}│     │ User(5) → {Bob}  │
└──────────────────┘     └──────────────────┘

Each session has its OWN cache. Not shared!
```

### Second Level Cache (Shared Cache)

```
Second Level Cache:
┌─────────────────────────────────────┐
│ Shared across ALL sessions           │
│                                      │
│ User(1) → {Dilip}                   │
│ User(2) → {Alice}                   │
│                                      │
│ Any session can read from here!     │
│ Reduces database queries globally.   │
└─────────────────────────────────────┘

Flow:
1. Session.get(User.class, 1)
2. Check FIRST level cache (session) → miss
3. Check SECOND level cache (shared) → hit? Return!
4. If miss → query DATABASE → store in both caches
```

```xml
<!-- pom.xml — Add EhCache dependency -->
<dependency>
    <groupId>org.hibernate.orm</groupId>
    <artifactId>hibernate-jcache</artifactId>
</dependency>
<dependency>
    <groupId>org.ehcache</groupId>
    <artifactId>ehcache</artifactId>
</dependency>
```

```java
// Enable on entity
@Entity
@Cacheable                                          // JPA standard
@org.hibernate.annotations.Cache(
    usage = CacheConcurrencyStrategy.READ_WRITE     // Hibernate-specific
)
public class User {
    // ...
}

// Cache concurrency strategies:
// READ_ONLY          — Never updated, safest, fastest
// NONSTRICT_READ_WRITE — Occasional updates OK, slight inconsistency window
// READ_WRITE         — Frequent reads, some writes. Uses soft locks.
// TRANSACTIONAL      — Full JTA transactions (most consistent, slowest)
```

### Query Cache

```java
// Cache query RESULTS (not just entities)
List<User> users = session
    .createQuery("FROM User u WHERE u.active = true", User.class)
    .setCacheable(true)           // Enable query cache for this query
    .setCacheRegion("activeUsers") // Optional: named cache region
    .getResultList();

// Properties:
spring.jpa.properties.hibernate.cache.use_query_cache=true
```

---

## 9. Lazy Loading & Proxy

### How Lazy Loading Works

```java
@Entity
public class User {
    @Id
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    private List<Order> orders;  // NOT loaded immediately!
}

// Loading user:
User user = em.find(User.class, 1L);
// SQL: SELECT * FROM users WHERE id = 1
// orders is a PROXY (placeholder), not actual data!

user.getName();           // ✅ Works fine — name was loaded
user.getOrders().size();  // ✅ NOW Hibernate loads orders!
// SQL: SELECT * FROM orders WHERE user_id = 1
```

### LazyInitializationException (The Classic Error!)

```java
// ❌ THIS WILL FAIL:
@GetMapping("/users/{id}")
public User getUser(@PathVariable Long id) {
    User user = userRepository.findById(id).get();
    return user;  // Jackson tries to serialize user.getOrders()
    // But the Session is already CLOSED!
    // → LazyInitializationException! 💥
}

// Solution 1: Use DTO (don't return entity)
@GetMapping("/users/{id}")
public UserDTO getUser(@PathVariable Long id) {
    User user = userRepository.findById(id).get();
    return new UserDTO(user.getId(), user.getName()); // No lazy fields
}

// Solution 2: JOIN FETCH in repository
@Query("SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :id")
User findByIdWithOrders(@Param("id") Long id);

// Solution 3: @EntityGraph
@EntityGraph(attributePaths = {"orders"})
Optional<User> findById(Long id);

// Solution 4: @Transactional on controller (not ideal)
@GetMapping("/users/{id}")
@Transactional(readOnly = true)  // Keeps session open
public User getUser(@PathVariable Long id) {
    // ...
}
```

---

## 10. Locking (Optimistic & Pessimistic)

### Optimistic Locking (Version-Based)

```java
@Entity
public class Product {
    @Id
    private Long id;
    private String name;
    private int stock;
    
    @Version                    // Optimistic lock version
    private int version;        // Auto-incremented by Hibernate
}
```

```
How @Version works:

1. User A reads Product (version=1)
2. User B reads Product (version=1)
3. User A updates stock: UPDATE products SET stock=5, version=2 WHERE id=1 AND version=1
   → Success! version becomes 2
4. User B updates stock: UPDATE products SET stock=3, version=2 WHERE id=1 AND version=1
   → FAILS! version is now 2, not 1
   → Throws OptimisticLockException! ❌
   → User B must refresh and retry

Timeline:
  User A: read(v1) --------→ write(v1→v2) ✅
  User B: read(v1) ----→ --------→ write(v1→v2) ❌ version mismatch!
```

### Pessimistic Locking (Database-Level)

```java
// Lock the row in the database (prevents others from reading/writing)

// PESSIMISTIC_READ (shared lock — others can read, can't write)
User user = em.find(User.class, 1L, LockModeType.PESSIMISTIC_READ);
// SQL: SELECT * FROM users WHERE id = 1 FOR SHARE

// PESSIMISTIC_WRITE (exclusive lock — others can't read or write)
User user = em.find(User.class, 1L, LockModeType.PESSIMISTIC_WRITE);
// SQL: SELECT * FROM users WHERE id = 1 FOR UPDATE

// In Spring Data JPA:
@Lock(LockModeType.PESSIMISTIC_WRITE)
@Query("SELECT u FROM User u WHERE u.id = :id")
User findByIdForUpdate(@Param("id") Long id);
```

### When to Use Which?

```
OPTIMISTIC (default choice ✅):
  - Low contention (rare conflicts)
  - High read/low write
  - Better performance (no DB locks)
  - Retry on conflict

PESSIMISTIC (special cases):
  - High contention (many concurrent writes to same data)
  - Critical operations (financial transactions)
  - Guarantees no conflicts (but may cause deadlocks)
```

---

## 11. Native SQL Queries

```java
// When HQL/JPQL can't express your query (database-specific features)

// Simple native query
List<Object[]> results = em.createNativeQuery(
    "SELECT id, name, email FROM users WHERE active = 1")
    .getResultList();

for (Object[] row : results) {
    Long id = ((Number) row[0]).longValue();
    String name = (String) row[1];
}

// Native query mapped to entity
List<User> users = em.createNativeQuery(
    "SELECT * FROM users WHERE active = 1", User.class)
    .getResultList();

// Native query with parameters
List<User> users = em.createNativeQuery(
    "SELECT * FROM users WHERE name LIKE :pattern", User.class)
    .setParameter("pattern", "%Dilip%")
    .getResultList();

// In Spring Data JPA
@Query(value = "SELECT * FROM users WHERE MATCH(name) AGAINST(:query)", 
       nativeQuery = true)
List<User> fullTextSearch(@Param("query") String query);
```

---

## 12. Hibernate Validators

```java
@Entity
public class User {
    
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be 2-100 characters")
    private String name;
    
    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    @Column(unique = true)
    private String email;
    
    @NotNull(message = "Age is required")
    @Min(value = 18, message = "Must be at least 18")
    @Max(value = 150, message = "Invalid age")
    private Integer age;
    
    @Pattern(regexp = "^\\+?[1-9]\\d{1,14}$", message = "Invalid phone number")
    private String phone;
    
    @Past(message = "Birth date must be in the past")
    private LocalDate birthDate;
    
    @Future(message = "Expiry must be in the future")
    private LocalDate subscriptionExpiry;
    
    @DecimalMin(value = "0.0", message = "Salary must be positive")
    private BigDecimal salary;
}
```

---

## 13. Hibernate in Spring Boot

### Spring Data JPA Repository

```java
// Spring Data JPA auto-generates CRUD implementations!

public interface UserRepository extends JpaRepository<User, Long> {
    
    // Method name → query generation (magic!)
    List<User> findByName(String name);
    // → SELECT * FROM users WHERE name = ?
    
    List<User> findByNameContainingIgnoreCase(String name);
    // → SELECT * FROM users WHERE LOWER(name) LIKE LOWER('%?%')
    
    List<User> findByAgeGreaterThan(int age);
    // → SELECT * FROM users WHERE age > ?
    
    List<User> findByActiveAndRole(boolean active, UserRole role);
    // → SELECT * FROM users WHERE active = ? AND role = ?
    
    List<User> findByNameOrderByCreatedAtDesc(String name);
    // → SELECT * FROM users WHERE name = ? ORDER BY created_at DESC
    
    Optional<User> findByEmail(String email);
    boolean existsByEmail(String email);
    long countByActive(boolean active);
    
    // Custom JPQL query
    @Query("SELECT u FROM User u WHERE u.age BETWEEN :min AND :max")
    List<User> findByAgeRange(@Param("min") int min, @Param("max") int max);
    
    // Modifying query
    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.lastLogin < :date")
    int deactivateInactiveUsers(@Param("date") LocalDateTime date);
    
    // Pagination
    Page<User> findByActive(boolean active, Pageable pageable);
}
```

### Service Layer

```java
@Service
@Transactional
public class UserService {
    
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public User createUser(CreateUserDTO dto) {
        if (userRepository.existsByEmail(dto.getEmail())) {
            throw new DuplicateEmailException("Email already exists");
        }
        
        User user = new User();
        user.setName(dto.getName());
        user.setEmail(dto.getEmail());
        user.setActive(true);
        
        return userRepository.save(user);
    }
    
    @Transactional(readOnly = true)  // Optimization for read operations
    public Page<User> getUsers(int page, int size) {
        return userRepository.findByActive(true, PageRequest.of(page, size, Sort.by("name")));
    }
    
    public User updateUser(Long id, UpdateUserDTO dto) {
        User user = userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
        
        user.setName(dto.getName());
        // No save() needed! Entity is managed, dirty checking handles it.
        
        return user;
    }
}
```

---

## 14. Performance Tuning

```java
// 1. Batch inserts
spring.jpa.properties.hibernate.jdbc.batch_size=50
spring.jpa.properties.hibernate.order_inserts=true

@Transactional
public void bulkInsert(List<User> users) {
    for (int i = 0; i < users.size(); i++) {
        em.persist(users.get(i));
        if (i % 50 == 0) {
            em.flush();   // Write batch to DB
            em.clear();   // Clear first-level cache (prevent memory overflow)
        }
    }
}

// 2. Read-only transactions
@Transactional(readOnly = true)  // Hibernate skips dirty checking!
public List<User> getAll() { ... }

// 3. DTO projections (don't load full entities)
@Query("SELECT new com.example.dto.UserSummary(u.id, u.name) FROM User u")
List<UserSummary> findAllSummaries();

// 4. Avoid N+1 with JOIN FETCH
@Query("SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :id")
User findWithOrders(@Param("id") Long id);

// 5. Use @BatchSize for collections
@OneToMany(mappedBy = "user")
@BatchSize(size = 20)
private List<Order> orders;

// 6. Enable statistics to find bottlenecks
spring.jpa.properties.hibernate.generate_statistics=true
// Then check: session.getSessionFactory().getStatistics()
```

---

## 15. Common Exceptions & Solutions

```java
// 1. LazyInitializationException
// "could not initialize proxy - no Session"
// CAUSE: Accessing lazy-loaded data after Session is closed
// FIX: JOIN FETCH, @EntityGraph, DTO projection, @Transactional

// 2. NonUniqueResultException
// CAUSE: Query expected single result but got multiple
// FIX: Use getResultList() instead of getSingleResult(), or fix query

// 3. OptimisticLockException
// CAUSE: Another transaction modified the entity (version mismatch)
// FIX: Catch exception, refresh entity, retry the operation

// 4. ConstraintViolationException
// CAUSE: Violating unique, not-null, or foreign key constraints
// FIX: Validate data before persisting, handle exception gracefully

// 5. TransientPropertyValueException
// "Not-null property references a transient value"
// CAUSE: Saving entity that references unsaved entity without cascade
// FIX: Add CascadeType.PERSIST or save the referenced entity first

// 6. MultipleBagFetchException
// "cannot simultaneously fetch multiple bags"
// CAUSE: JOIN FETCH on two @OneToMany List collections
// FIX: Use Set instead of List, or use @BatchSize, or separate queries

// 7. StaleStateException
// CAUSE: Trying to update/delete a row that doesn't exist
// FIX: Check if entity exists before update, handle concurrent deletes

// 8. DataIntegrityViolationException
// CAUSE: Database constraint violation (duplicate key, FK violation)
// FIX: Validate before save, use proper exception handling
```

---

## 16. Best Practices

1. **Always use LAZY fetch type** — Override EAGER defaults
2. **Use `@Transactional(readOnly = true)`** for read operations
3. **Return DTOs from API endpoints** — Never expose entities
4. **Use JOIN FETCH** to solve N+1 queries
5. **Enable batch processing** for bulk operations
6. **Use `@Version`** for optimistic locking
7. **Log SQL in development** — `show-sql=true`, `format_sql=true`
8. **Use `ddl-auto=validate`** in production — Never `update` or `create`
9. **Use Flyway/Liquibase** for database migrations
10. **Don't use bidirectional** relationships unless necessary
11. **Clear EntityManager** during bulk operations to prevent memory issues
12. **Use connection pooling** (HikariCP — Spring Boot default)

---

## 17. Interview Questions & Answers (50+)

### Beginner Level

**Q1: What is Hibernate?**
**A:** Java ORM framework that maps objects to database tables. Implements JPA specification. Generates SQL automatically, provides caching, lazy loading, and transaction management.

**Q2: What is the difference between `get()` and `load()`?**
**A:** `get()` hits database immediately, returns null if not found. `load()` returns a proxy, hits DB on access, throws ObjectNotFoundException if not found.

**Q3: What is the Hibernate Session?**
**A:** Short-lived, single-threaded object that wraps a JDBC connection. Provides CRUD operations and a first-level cache. One per request/transaction.

**Q4: What is SessionFactory?**
**A:** Thread-safe, heavy object that creates Sessions. One per application. Configured from hibernate.cfg.xml or Spring properties.

**Q5: What is HQL?**
**A:** Hibernate Query Language — object-oriented query language using entity/field names. Database-independent. Superset of JPQL.

**Q6: What are the entity states?**
**A:** Transient (new, no ID), Persistent/Managed (tracked, has ID), Detached (was persistent, session closed), Removed (scheduled for deletion).

**Q7: What is dirty checking?**
**A:** Hibernate automatically detects changes to managed entities and generates UPDATE statements. No explicit save/update needed for managed entities.

**Q8: What is the first-level cache?**
**A:** Per-Session cache. Every entity loaded in a session is cached. Second read of same entity returns cached object. Cleared when session closes.

---

### Intermediate Level

**Q9: What is the second-level cache?**
**A:** Shared cache across all sessions. Reduces database hits. Uses providers like EhCache, Redis. Must be explicitly enabled and configured.

**Q10: What is lazy loading?**
**A:** Related data loaded only when first accessed. Uses proxy objects. Default for @OneToMany/@ManyToMany. Reduces initial query size.

**Q11: What is the N+1 problem?**
**A:** Loading N entities triggers N additional queries for lazy collections. Fixed with JOIN FETCH, @EntityGraph, or @BatchSize.

**Q12: What is `@Version` used for?**
**A:** Optimistic locking. Auto-incremented on update. Prevents lost updates when multiple transactions modify same entity.

**Q13: What is the difference between `persist()` and `save()`?**
**A:** `persist()` (JPA) doesn't return ID, doesn't guarantee immediate INSERT. `save()` (Hibernate) returns generated ID, may INSERT immediately.

**Q14: What is `merge()` vs `update()`?**
**A:** `merge()` copies detached state to a managed entity (returns managed copy). `update()` reattaches the detached entity itself (throws if another managed entity with same ID exists).

**Q15: What is `@Transactional(readOnly = true)`?**
**A:** Optimization hint. Hibernate skips dirty checking for read-only transactions, improving performance.

---

### Advanced Level

**Q16: Explain Hibernate caching strategies.**
**A:** READ_ONLY (immutable), NONSTRICT_READ_WRITE (eventual consistency), READ_WRITE (read-committed, uses soft locks), TRANSACTIONAL (full ACID with JTA).

**Q17: How to solve MultipleBagFetchException?**
**A:** Can't JOIN FETCH two `List` collections simultaneously. Solutions: use `Set` instead of `List`, use `@BatchSize`, or execute separate queries.

**Q18: What is Hibernate's bytecode enhancement?**
**A:** Compile-time modification of entity bytecode for lazy field loading (not just association loading), dirty checking optimization, and bi-directional relationship management.

**Q19: What is Hibernate Envers?**
**A:** Auditing framework. Tracks entity changes (create, update, delete) in audit tables. Access historical versions with `AuditReader`.

**Q20: What is `@NaturalId`?**
**A:** Hibernate-specific. Marks a business key (email, ISBN). Enables efficient `session.byNaturalId()` lookups. Can be cached separately.

---

### Rapid-Fire (Q21–Q50)

**Q21: What is `hibernate.hbm2ddl.auto`?** Controls schema generation: none, validate, update, create, create-drop.

**Q22: What is a Hibernate Dialect?** Generates database-specific SQL. MySQLDialect, PostgreSQLDialect, etc.

**Q23: What is `session.flush()`?** Synchronizes session state with database. Writes pending SQL.

**Q24: What is `session.clear()`?** Detaches all managed entities. Clears first-level cache.

**Q25: What is `session.evict(obj)`?** Removes a specific entity from first-level cache.

**Q26: What is `@DynamicUpdate`?** Generates UPDATE with only changed columns (not all columns).

**Q27: What is `@DynamicInsert`?** Generates INSERT with only non-null columns.

**Q28: What is `@SQLDelete`?** Custom SQL for delete operations. Used for soft deletes.

**Q29: What is `@Where`?** Adds SQL WHERE clause to all queries. `@Where(clause = "deleted = false")`.

**Q30: What is `@Filter`?** Dynamic, parameterized WHERE clause that can be enabled/disabled.

**Q31: What is the Open Session in View pattern?** Keeps Hibernate Session open during view rendering. Controversial.

**Q32: What is `@Immutable`?** Marks entity as read-only. Hibernate never generates UPDATE for it.

**Q33: What is `@BatchSize`?** Loads lazy collections in batches instead of one-by-one.

**Q34: What is `@Fetch(FetchMode.SUBSELECT)`?** Loads all lazy collections for all parent entities in one subselect query.

**Q35: What is `cascade = CascadeType.REMOVE`?** Deleting parent also deletes children.

**Q36: What is `orphanRemoval = true`?** Removing child from parent's collection deletes it from DB.

**Q37: Default fetch type for @ManyToOne?** EAGER. Should be changed to LAZY!

**Q38: What is Hibernate Interceptor?** Callbacks before/after CRUD operations. `onSave`, `onFlush`, `onDelete`.

**Q39: What is `@GeneratedValue` AUTO vs IDENTITY?** AUTO lets Hibernate choose strategy. IDENTITY uses database auto-increment.

**Q40: What is SEQUENCE strategy?** Uses database sequence to generate IDs. Supports batch inserts (IDENTITY doesn't).

**Q41: What is `@Formula`?** Computed column using SQL expression. Read-only.

**Q42: What is `@SecondaryTable`?** Maps entity to multiple tables.

**Q43: What is `@OrderBy`?** Sorts collection when loaded. `@OrderBy("name ASC")`.

**Q44: What is `@SQLRestriction`?** Modern replacement for `@Where`. Adds SQL filter.

**Q45: What is StatelessSession?** No first-level cache, no dirty checking, no lazy loading. Fast for bulk operations.

**Q46: What is `session.scroll()`?** Scrollable results for processing large datasets.

**Q47: What is `@CreationTimestamp`?** Hibernate-specific. Auto-sets creation timestamp.

**Q48: What is `@UpdateTimestamp`?** Hibernate-specific. Auto-updates timestamp on modification.

**Q49: What is `@ColumnTransformer`?** SQL transformation on read/write. E.g., encryption: `read = "AES_DECRYPT(col, key)"`.

**Q50: What is multi-tenancy in Hibernate?** Single app serving multiple tenants. Strategies: separate databases, separate schemas, shared schema with discriminator.

---

## 📚 References

- [Hibernate ORM Documentation](https://hibernate.org/orm/documentation/6.4/)
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/)
- [Vlad Mihalcea — Hibernate Expert Blog](https://vladmihalcea.com/)
- [Baeldung Hibernate Tutorials](https://www.baeldung.com/learn-jpa-hibernate)
- [Hibernate Performance Tuning](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#performance)

---

> **Previous Topic:** [← 10 - ORM Tools](../10-orm-tools/README.md)  
> **Next Topic:** [12 - Spring Framework →](../12-spring-framework/README.md)
