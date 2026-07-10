# 📊 Spring Data JPA — Complete In-Depth Guide

> **"Spring Data JPA eliminates 90% of data access code. Define an interface, Spring writes the implementation. It's the most productive way to work with databases in Java."**

---

## 📑 Table of Contents

1. [Introduction & Setup](#1-introduction--setup)
2. [Repository Hierarchy](#2-repository-hierarchy)
3. [Query Methods (Derived Queries)](#3-query-methods-derived-queries)
4. [Custom Queries (@Query)](#4-custom-queries-query)
5. [Pagination & Sorting](#5-pagination--sorting)
6. [Specifications (Dynamic Queries)](#6-specifications-dynamic-queries)
7. [Projections & DTOs](#7-projections--dtos)
8. [Auditing](#8-auditing)
9. [Custom Repository Implementation](#9-custom-repository-implementation)
10. [Query by Example (QBE)](#10-query-by-example-qbe)
11. [Transactions](#11-transactions)
12. [Entity Relationships in Practice](#12-entity-relationships-in-practice)
13. [Testing Repositories](#13-testing-repositories)
14. [Performance Optimization](#14-performance-optimization)
15. [Best Practices](#15-best-practices)
16. [Interview Questions & Answers (50+)](#16-interview-questions--answers-50)

---

## 1. Introduction & Setup

### What is Spring Data JPA?

```
Without Spring Data JPA:
  1. Write Entity class
  2. Write DAO interface
  3. Write DAO implementation (100+ lines of EntityManager code)
  4. Handle transactions manually

With Spring Data JPA:
  1. Write Entity class
  2. Write Repository interface (just extend JpaRepository)
  3. DONE! Spring generates the implementation automatically! ✅
```

### Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

### Entity Example

```java
@Entity
@Table(name = "users")
@Getter @Setter @NoArgsConstructor
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(nullable = false, length = 100)
    private String name;
    
    @Column(nullable = false, unique = true)
    private String email;
    
    @Enumerated(EnumType.STRING)
    private UserRole role;
    
    private boolean active = true;
    
    private LocalDateTime createdAt;
    private LocalDateTime updatedAt;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
    
    @PrePersist
    protected void onCreate() {
        createdAt = LocalDateTime.now();
        updatedAt = LocalDateTime.now();
    }
    
    @PreUpdate
    protected void onUpdate() {
        updatedAt = LocalDateTime.now();
    }
}
```

---

## 2. Repository Hierarchy

```
Repository<T, ID>                    (marker interface)
    │
    ├── CrudRepository<T, ID>        (basic CRUD: save, findById, delete, count)
    │       │
    │       ├── ListCrudRepository    (returns List instead of Iterable)
    │       │
    │       └── PagingAndSortingRepository (+ findAll(Pageable), findAll(Sort))
    │               │
    │               └── JpaRepository<T, ID>  (+ flush, saveAll, deleteInBatch)
    │
    └── JpaSpecificationExecutor<T>   (dynamic queries with Specifications)
```

```java
// Most common: extend JpaRepository (includes EVERYTHING)
public interface UserRepository extends JpaRepository<User, Long> {
    // INHERITED METHODS (no code needed!):
    // save(User) → INSERT or UPDATE
    // saveAll(List<User>) → Batch save
    // findById(Long) → Optional<User>
    // findAll() → List<User>
    // findAll(Pageable) → Page<User>
    // findAll(Sort) → List<User>
    // count() → long
    // deleteById(Long)
    // delete(User)
    // deleteAll()
    // existsById(Long) → boolean
    // flush()
    // saveAndFlush(User)
}
```

---

## 3. Query Methods (Derived Queries)

Spring Data JPA generates SQL from **method names**!

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // ═══ FIND BY SINGLE FIELD ═══
    List<User> findByName(String name);
    // → SELECT * FROM users WHERE name = ?
    
    Optional<User> findByEmail(String email);
    // → SELECT * FROM users WHERE email = ?
    
    List<User> findByRole(UserRole role);
    // → SELECT * FROM users WHERE role = ?
    
    // ═══ FIND BY MULTIPLE FIELDS ═══
    List<User> findByNameAndEmail(String name, String email);
    // → SELECT * FROM users WHERE name = ? AND email = ?
    
    List<User> findByNameOrEmail(String name, String email);
    // → SELECT * FROM users WHERE name = ? OR email = ?
    
    // ═══ COMPARISON ═══
    List<User> findByAgeGreaterThan(int age);
    // → WHERE age > ?
    
    List<User> findByAgeLessThanEqual(int age);
    // → WHERE age <= ?
    
    List<User> findByAgeBetween(int start, int end);
    // → WHERE age BETWEEN ? AND ?
    
    // ═══ STRING MATCHING ═══
    List<User> findByNameContaining(String keyword);
    // → WHERE name LIKE '%keyword%'
    
    List<User> findByNameStartingWith(String prefix);
    // → WHERE name LIKE 'prefix%'
    
    List<User> findByNameEndingWith(String suffix);
    // → WHERE name LIKE '%suffix'
    
    List<User> findByNameContainingIgnoreCase(String keyword);
    // → WHERE LOWER(name) LIKE LOWER('%keyword%')
    
    // ═══ NULL CHECKS ═══
    List<User> findByEmailIsNull();
    // → WHERE email IS NULL
    
    List<User> findByEmailIsNotNull();
    // → WHERE email IS NOT NULL
    
    // ═══ BOOLEAN ═══
    List<User> findByActiveTrue();
    // → WHERE active = true
    
    List<User> findByActiveFalse();
    // → WHERE active = false
    
    // ═══ IN CLAUSE ═══
    List<User> findByRoleIn(List<UserRole> roles);
    // → WHERE role IN (?, ?, ?)
    
    // ═══ ORDERING ═══
    List<User> findByActiveOrderByNameAsc(boolean active);
    // → WHERE active = ? ORDER BY name ASC
    
    List<User> findByRoleOrderByCreatedAtDesc(UserRole role);
    // → WHERE role = ? ORDER BY created_at DESC
    
    // ═══ LIMITING ═══
    List<User> findTop5ByOrderByCreatedAtDesc();
    // → SELECT * FROM users ORDER BY created_at DESC LIMIT 5
    
    User findFirstByOrderByNameAsc();
    // → SELECT * FROM users ORDER BY name ASC LIMIT 1
    
    // ═══ COUNT, EXISTS, DELETE ═══
    long countByActive(boolean active);
    // → SELECT COUNT(*) FROM users WHERE active = ?
    
    boolean existsByEmail(String email);
    // → SELECT COUNT(*) > 0 FROM users WHERE email = ?
    
    void deleteByEmail(String email);
    // → DELETE FROM users WHERE email = ?
    
    // ═══ DISTINCT ═══
    List<User> findDistinctByRole(UserRole role);
    // → SELECT DISTINCT * FROM users WHERE role = ?
}
```

### Method Name Keywords Reference

| Keyword | SQL | Example |
|---------|-----|---------|
| `And` | AND | `findByNameAndEmail` |
| `Or` | OR | `findByNameOrEmail` |
| `Is`, `Equals` | = | `findByName` / `findByNameIs` |
| `Between` | BETWEEN | `findByAgeBetween` |
| `LessThan` | < | `findByAgeLessThan` |
| `GreaterThan` | > | `findByAgeGreaterThan` |
| `Like` | LIKE | `findByNameLike` |
| `Containing` | LIKE %x% | `findByNameContaining` |
| `StartingWith` | LIKE x% | `findByNameStartingWith` |
| `In` | IN | `findByRoleIn` |
| `True`/`False` | = true/false | `findByActiveTrue` |
| `IsNull` | IS NULL | `findByEmailIsNull` |
| `OrderBy` | ORDER BY | `findByActiveOrderByName` |
| `Not` | != | `findByRoleNot` |
| `IgnoreCase` | LOWER() | `findByNameIgnoreCase` |

---

## 4. Custom Queries (@Query)

```java
public interface UserRepository extends JpaRepository<User, Long> {
    
    // ═══ JPQL (Entity/field names) ═══
    @Query("SELECT u FROM User u WHERE u.email = :email")
    Optional<User> findByEmailCustom(@Param("email") String email);
    
    @Query("SELECT u FROM User u WHERE u.name LIKE %:keyword% AND u.active = true")
    List<User> searchActive(@Param("keyword") String keyword);
    
    @Query("SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :id")
    Optional<User> findByIdWithOrders(@Param("id") Long id);
    
    @Query("SELECT u FROM User u WHERE u.age BETWEEN :min AND :max ORDER BY u.age")
    Page<User> findByAgeRange(@Param("min") int min, @Param("max") int max, Pageable pageable);
    
    // ═══ NATIVE SQL (table/column names) ═══
    @Query(value = "SELECT * FROM users WHERE MATCH(name, email) AGAINST(:query IN BOOLEAN MODE)",
           nativeQuery = true)
    List<User> fullTextSearch(@Param("query") String query);
    
    @Query(value = "SELECT * FROM users WHERE created_at >= DATE_SUB(NOW(), INTERVAL :days DAY)",
           nativeQuery = true)
    List<User> findRecentUsers(@Param("days") int days);
    
    // ═══ MODIFYING QUERIES (UPDATE/DELETE) ═══
    @Modifying
    @Query("UPDATE User u SET u.active = false WHERE u.lastLogin < :date")
    int deactivateInactiveUsers(@Param("date") LocalDateTime date);
    
    @Modifying
    @Query("DELETE FROM User u WHERE u.active = false AND u.createdAt < :date")
    int purgeInactiveUsers(@Param("date") LocalDateTime date);
    
    // ═══ DTO PROJECTIONS ═══
    @Query("SELECT new com.example.dto.UserSummary(u.id, u.name, u.email) FROM User u")
    List<UserSummary> findAllSummaries();
    
    // ═══ AGGREGATE QUERIES ═══
    @Query("SELECT u.role, COUNT(u) FROM User u GROUP BY u.role")
    List<Object[]> countByRole();
    
    @Query("SELECT AVG(u.age) FROM User u WHERE u.active = true")
    Double getAverageAge();
}
```

---

## 5. Pagination & Sorting

```java
// ═══ REPOSITORY ═══
public interface UserRepository extends JpaRepository<User, Long> {
    Page<User> findByActive(boolean active, Pageable pageable);
    List<User> findByRole(UserRole role, Sort sort);
    Slice<User> findByNameContaining(String name, Pageable pageable);
}

// ═══ SERVICE ═══
@Service
@Transactional(readOnly = true)
public class UserService {
    
    public Page<UserDTO> findAll(int page, int size, String sortBy, String direction) {
        Sort sort = direction.equalsIgnoreCase("desc")
            ? Sort.by(sortBy).descending()
            : Sort.by(sortBy).ascending();
        
        Pageable pageable = PageRequest.of(page, size, sort);
        
        return userRepository.findByActive(true, pageable)
            .map(this::toDTO);
    }
    
    // Multi-field sorting
    public Page<UserDTO> findAllMultiSort(int page, int size) {
        Sort sort = Sort.by(
            Sort.Order.asc("name"),
            Sort.Order.desc("createdAt")
        );
        return userRepository.findAll(PageRequest.of(page, size, sort)).map(this::toDTO);
    }
}

// ═══ CONTROLLER ═══
@GetMapping
public ResponseEntity<Page<UserDTO>> getAll(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(defaultValue = "id") String sortBy,
        @RequestParam(defaultValue = "asc") String direction) {
    return ResponseEntity.ok(userService.findAll(page, size, sortBy, direction));
}

// Request: GET /api/users?page=0&size=20&sortBy=name&direction=asc
```

### Page vs Slice vs List

```
Page<User>:  Contains data + total count (extra COUNT query)
  - page.getContent()        → List<User>
  - page.getTotalElements()  → Total records (e.g., 100)
  - page.getTotalPages()     → Total pages (e.g., 5)
  - page.getNumber()         → Current page (e.g., 0)
  - page.hasNext()           → true/false

Slice<User>: Contains data + hasNext (NO total count — faster!)
  - slice.getContent()       → List<User>
  - slice.hasNext()          → true/false (loads size+1 to check)
  - No getTotalElements()!

List<User>:  Just the data, no pagination metadata.

Use Page when UI needs "Page 1 of 5 (100 results)".
Use Slice for infinite scroll (just "load more").
```

---

## 6. Specifications (Dynamic Queries)

```java
// For complex, DYNAMIC queries (like search filters)
// Implement JpaSpecificationExecutor

public interface UserRepository extends JpaRepository<User, Long>,
                                         JpaSpecificationExecutor<User> { }

// Specification builder
public class UserSpecifications {
    
    public static Specification<User> hasName(String name) {
        return (root, query, cb) -> 
            name == null ? null : cb.like(cb.lower(root.get("name")), "%" + name.toLowerCase() + "%");
    }
    
    public static Specification<User> hasRole(UserRole role) {
        return (root, query, cb) -> 
            role == null ? null : cb.equal(root.get("role"), role);
    }
    
    public static Specification<User> isActive() {
        return (root, query, cb) -> cb.isTrue(root.get("active"));
    }
    
    public static Specification<User> ageGreaterThan(Integer age) {
        return (root, query, cb) -> 
            age == null ? null : cb.greaterThan(root.get("age"), age);
    }
    
    public static Specification<User> createdAfter(LocalDateTime date) {
        return (root, query, cb) -> 
            date == null ? null : cb.greaterThan(root.get("createdAt"), date);
    }
}

// Usage in service — combine dynamically!
public Page<User> search(String name, UserRole role, Integer minAge, Pageable pageable) {
    
    Specification<User> spec = Specification
        .where(UserSpecifications.isActive())
        .and(UserSpecifications.hasName(name))      // Only added if name != null
        .and(UserSpecifications.hasRole(role))       // Only added if role != null
        .and(UserSpecifications.ageGreaterThan(minAge));
    
    return userRepository.findAll(spec, pageable);
}
```

---

## 7. Projections & DTOs

### Interface-Based Projection

```java
// Only load specific fields (lightweight!)
public interface UserSummary {
    Long getId();
    String getName();
    String getEmail();
    
    @Value("#{target.name + ' (' + target.email + ')'}")  // SpEL computed
    String getDisplayName();
}

public interface UserRepository extends JpaRepository<User, Long> {
    List<UserSummary> findByActive(boolean active);
    // → SELECT id, name, email FROM users WHERE active = ?
    // Only 3 columns loaded instead of ALL columns! ✅
}
```

### Class-Based Projection (DTO)

```java
// Using constructor expression in @Query
public record UserDTO(Long id, String name, String email) {}

@Query("SELECT new com.example.dto.UserDTO(u.id, u.name, u.email) FROM User u WHERE u.active = true")
List<UserDTO> findAllActiveAsDTO();
```

---

## 8. Auditing

```java
// Auto-populate createdBy, createdDate, lastModifiedBy, lastModifiedDate

@Configuration
@EnableJpaAuditing
public class JpaConfig {
    @Bean
    public AuditorAware<String> auditorProvider() {
        return () -> Optional.of(SecurityContextHolder.getContext()
            .getAuthentication().getName());
    }
}

@MappedSuperclass
@EntityListeners(AuditingEntityListener.class)
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

@Entity
public class User extends BaseEntity {
    // Inherits createdAt, updatedAt, createdBy, updatedBy!
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
}
```

---

## 9. Custom Repository Implementation

```java
// When derived queries and @Query aren't enough

// 1. Define custom interface
public interface UserRepositoryCustom {
    List<User> complexSearch(UserSearchCriteria criteria);
}

// 2. Implement it (MUST end with "Impl")
@Repository
public class UserRepositoryCustomImpl implements UserRepositoryCustom {
    
    @PersistenceContext
    private EntityManager em;
    
    @Override
    public List<User> complexSearch(UserSearchCriteria criteria) {
        CriteriaBuilder cb = em.getCriteriaBuilder();
        CriteriaQuery<User> cq = cb.createQuery(User.class);
        Root<User> root = cq.from(User.class);
        
        List<Predicate> predicates = new ArrayList<>();
        
        if (criteria.getName() != null) {
            predicates.add(cb.like(root.get("name"), "%" + criteria.getName() + "%"));
        }
        // ... more conditions
        
        cq.where(predicates.toArray(new Predicate[0]));
        return em.createQuery(cq).getResultList();
    }
}

// 3. Extend both interfaces
public interface UserRepository extends JpaRepository<User, Long>, UserRepositoryCustom {
    // Now has JpaRepository methods + complexSearch()!
}
```

---

## 10. Query by Example (QBE)

```java
// Search by providing an example entity

User probe = new User();
probe.setRole(UserRole.ADMIN);
probe.setActive(true);

ExampleMatcher matcher = ExampleMatcher.matching()
    .withIgnoreNullValues()
    .withStringMatcher(ExampleMatcher.StringMatcher.CONTAINING)
    .withIgnoreCase();

Example<User> example = Example.of(probe, matcher);

List<User> results = userRepository.findAll(example);
// → WHERE role = 'ADMIN' AND active = true
```

---

## 11. Transactions

```java
@Service
public class OrderService {
    
    // READ operations: readOnly optimization
    @Transactional(readOnly = true)
    public List<OrderDTO> findAll() {
        return orderRepository.findAll().stream().map(this::toDTO).toList();
    }
    
    // WRITE operations: full transaction
    @Transactional
    public OrderDTO createOrder(CreateOrderDTO dto) {
        // All operations in single transaction
        User user = userRepository.findById(dto.getUserId())
            .orElseThrow(() -> new ResourceNotFoundException("User", "id", dto.getUserId()));
        
        Order order = new Order();
        order.setUser(user);
        order.setProduct(dto.getProduct());
        order.setAmount(dto.getAmount());
        
        Order saved = orderRepository.save(order);
        
        // Update user stats
        user.setOrderCount(user.getOrderCount() + 1);
        // Auto-saved by dirty checking (no explicit save needed!)
        
        return toDTO(saved);
        // COMMIT happens here (on successful return)
        // If exception → ROLLBACK everything
    }
    
    // Custom rollback rules
    @Transactional(rollbackFor = Exception.class,        // Rollback on ALL exceptions
                   noRollbackFor = EmailException.class)  // But NOT on EmailException
    public void processOrder(Long orderId) {
        // ...
    }
}
```

---

## 12. Entity Relationships in Practice

### Complete Example: E-Commerce

```java
@Entity
@Table(name = "users")
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    private String email;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
    
    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "profile_id")
    private UserProfile profile;
}

@Entity
@Table(name = "orders")
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private BigDecimal totalAmount;
    private LocalDateTime orderDate;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false)
    private User user;
    
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
}

@Entity
public class OrderItem {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private int quantity;
    private BigDecimal price;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "product_id")
    private Product product;
}

// Repositories
public interface OrderRepository extends JpaRepository<Order, Long> {
    
    @Query("SELECT o FROM Order o JOIN FETCH o.items i JOIN FETCH i.product WHERE o.user.id = :userId")
    List<Order> findByUserIdWithItems(@Param("userId") Long userId);
    
    List<Order> findByUserIdOrderByOrderDateDesc(Long userId);
    
    @Query("SELECT SUM(o.totalAmount) FROM Order o WHERE o.user.id = :userId")
    BigDecimal getTotalSpentByUser(@Param("userId") Long userId);
}
```

---

## 13. Testing Repositories

```java
@DataJpaTest  // Auto-configures: embedded H2 + JPA + repositories
class UserRepositoryTest {
    
    @Autowired
    private UserRepository userRepository;
    
    @Autowired
    private TestEntityManager entityManager;
    
    @Test
    void shouldFindByEmail() {
        // Given
        User user = new User();
        user.setName("Dilip");
        user.setEmail("dilip@mail.com");
        user.setActive(true);
        entityManager.persistAndFlush(user);
        
        // When
        Optional<User> found = userRepository.findByEmail("dilip@mail.com");
        
        // Then
        assertThat(found).isPresent();
        assertThat(found.get().getName()).isEqualTo("Dilip");
    }
    
    @Test
    void shouldReturnEmptyForNonExistentEmail() {
        Optional<User> found = userRepository.findByEmail("nonexistent@mail.com");
        assertThat(found).isEmpty();
    }
    
    @Test
    void shouldFindActiveUsers() {
        entityManager.persist(new User("Active", "a@mail.com", true));
        entityManager.persist(new User("Inactive", "i@mail.com", false));
        entityManager.flush();
        
        List<User> active = userRepository.findByActiveTrue();
        assertThat(active).hasSize(1);
        assertThat(active.get(0).getName()).isEqualTo("Active");
    }
}
```

---

## 14. Performance Optimization

```java
// 1. N+1 Problem → JOIN FETCH
@Query("SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :id")
User findByIdWithOrders(@Param("id") Long id);

// 2. Use @EntityGraph for flexible fetching
@EntityGraph(attributePaths = {"orders", "profile"})
Optional<User> findById(Long id);

// 3. Use projections — load only needed columns
List<UserSummary> findByActive(boolean active);  // Only id, name, email

// 4. Use Slice instead of Page (no COUNT query)
Slice<User> findByRole(UserRole role, Pageable pageable);

// 5. Bulk operations (skip entity lifecycle)
@Modifying
@Query("UPDATE User u SET u.active = false WHERE u.lastLogin < :date")
int bulkDeactivate(@Param("date") LocalDateTime date);

// 6. Read-only transactions
@Transactional(readOnly = true)
public List<UserDTO> findAll() { ... }
```

---

## 15. Best Practices

1. **Use derived queries** for simple lookups — Clean and type-safe
2. **Use @Query** for complex queries — JPQL for portability
3. **Always use LAZY fetch** — Override EAGER defaults
4. **Use projections** — Don't load entire entities when you need 2 fields
5. **Use Pageable** — Never return all records
6. **Use Specifications** for dynamic search — Avoid string concatenation
7. **Use @Transactional(readOnly=true)** for reads — Performance boost
8. **Use @EntityGraph** or JOIN FETCH — Solve N+1
9. **Use @DataJpaTest** for repository tests — Fast, embedded DB
10. **Use auditing** — Track who changed what and when

---

## 16. Interview Questions & Answers (50+)

### Beginner

**Q1: What is Spring Data JPA?** Abstraction over JPA that generates repository implementations from interface method names. Eliminates boilerplate DAO code.

**Q2: What is JpaRepository?** Interface providing CRUD, pagination, sorting, batch operations. Extend it for your entity.

**Q3: How do derived query methods work?** Spring parses method name: `findByNameAndEmail` → `WHERE name = ? AND email = ?`. Method name IS the query.

**Q4: What is `@Query`?** Custom JPQL or native SQL on repository methods. Used when method name gets too complex.

**Q5: What is `@Modifying`?** Required for UPDATE/DELETE @Query methods. Marks query as modifying (not SELECT).

**Q6: What is Pageable?** Interface representing pagination info (page number, size, sort). Use `PageRequest.of()`.

**Q7: What is Page vs Slice?** Page: data + total count (extra COUNT query). Slice: data + hasNext (faster, no COUNT).

**Q8: What is `findById` return type?** `Optional<T>`. Use `.orElseThrow()` for not-found handling.

---

### Intermediate

**Q9: What are Specifications?** Type-safe, dynamic query building using Criteria API. Combine with `and()`/`or()`. Good for search filters.

**Q10: What is a Projection?** Loading only specific fields. Interface-based (auto-proxy) or class-based (constructor expression).

**Q11: What is auditing?** Auto-populating createdAt, updatedAt, createdBy via `@CreatedDate`, `@LastModifiedDate`, `@EnableJpaAuditing`.

**Q12: How to implement soft delete?** Add `deleted` boolean, override `findAll` with `@Where(clause="deleted=false")`, use `@SQLDelete` for custom DELETE.

**Q13: What is `@EntityGraph`?** Defines which associations to eagerly fetch. Overrides entity-level LAZY without changing mapping.

**Q14: How to add custom methods to repository?** Create `XxxRepositoryCustom` interface, implement in `XxxRepositoryCustomImpl`, extend both.

**Q15: What is `saveAndFlush()` vs `save()`?** `save()` may delay SQL. `saveAndFlush()` writes to DB immediately.

---

### Advanced

**Q16: How to optimize batch inserts?** Set `hibernate.jdbc.batch_size`, use `saveAll()`, call `flush()`+`clear()` in chunks.

**Q17: What is Query by Example?** Create probe entity with desired values, Spring generates WHERE clause matching non-null fields.

**Q18: How does `@Transactional(readOnly=true)` improve performance?** Hibernate skips dirty checking (no need to detect changes). Database may use read replicas.

**Q19: How to handle MultipleBagFetchException?** Use `Set` instead of `List` for collections, or use separate queries instead of multiple JOIN FETCH.

**Q20: What is `@Lock`?** Specifies locking mode: `@Lock(LockModeType.PESSIMISTIC_WRITE)` for `SELECT ... FOR UPDATE`.

---

### Rapid-Fire (Q21–Q50)

**Q21: `CrudRepository` vs `JpaRepository`?** JpaRepository adds flush, batch, and JPA-specific methods.

**Q22: How to use native SQL?** `@Query(value = "SELECT...", nativeQuery = true)`.

**Q23: What is `@Param`?** Binds method parameter to named query parameter.

**Q24: `existsBy` return type?** `boolean`.

**Q25: `countBy` return type?** `long`.

**Q26: `deleteBy` return type?** `void` or `long` (count deleted).

**Q27: Can method names have `OrderBy`?** Yes: `findByActiveOrderByNameAsc`.

**Q28: `findTop5By` does what?** `LIMIT 5`.

**Q29: `findFirstBy` does what?** `LIMIT 1`.

**Q30: What is `@DataJpaTest`?** Test slice: configures embedded DB + JPA repos. No web layer.

**Q31: What is `TestEntityManager`?** Test helper for persisting/finding entities directly.

**Q32: `Iterable` vs `List` return?** `CrudRepository` returns `Iterable`. `ListCrudRepository/JpaRepository` returns `List`.

**Q33: What is `flush()`?** Forces pending SQL to execute immediately.

**Q34: What is `deleteInBatch()`?** Single DELETE SQL (faster than one-by-one).

**Q35: What is `deleteAllInBatch()`?** `DELETE FROM table` (no WHERE, very fast).

**Q36: What is `getReferenceById()`?** Returns proxy (like Hibernate `load()`). No DB hit until access.

**Q37: What is method query keywords `Between`?** `findByAgeBetween(18, 30)` → `WHERE age BETWEEN 18 AND 30`.

**Q38: What is `@EnableJpaRepositories`?** Enables repo scanning. Auto-configured in Spring Boot.

**Q39: Can repos return Stream?** Yes: `Stream<User> findAll()`. Must use in @Transactional.

**Q40: What is `Example.of()`?** Creates Query by Example from a probe entity.

**Q41: What is `ExampleMatcher`?** Configures QBE matching: containing, ignoreCase, etc.

**Q42: `@Transactional` on repo vs service?** Service (handle business transaction). Repo only for individual queries.

**Q43: Can @Query return DTO?** Yes: `SELECT new com.dto.UserDTO(u.id, u.name) FROM User u`.

**Q44: What is `@QueryHints`?** JPA query hints: timeout, fetch size, cache mode.

**Q45: What is Specification `toPredicate`?** Method that builds a JPA Predicate for the WHERE clause.

**Q46: What is `ScrollPosition`?** Spring Data 3.1+ for keyset/cursor pagination.

**Q47: Method name limit?** No hard limit, but keep readable. Use @Query for complex queries.

**Q48: Automatic table creation?** `spring.jpa.hibernate.ddl-auto=update` (dev only).

**Q49: Custom ID generator?** Implement `IdentifierGenerator`, use `@GenericGenerator`.

**Q50: What is `@NamedEntityGraph`?** Entity-level definition of fetch plan, referenced by `@EntityGraph(value="...")`.

---

## 📚 References

- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/)
- [Spring Data JPA Guides](https://spring.io/guides/gs/accessing-data-jpa/)
- [Baeldung Spring Data JPA](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)
- [Query Methods Docs](https://docs.spring.io/spring-data/jpa/reference/jpa/query-methods.html)

---

> **Previous Topic:** [← 14 - Spring JDBC](../14-spring-jdbc/README.md)  
> **Next Topic:** [16 - Spring Boot MVC Project →](../16-project-springboot-mvc/README.md)
