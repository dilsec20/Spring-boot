# 🔄 ORM Tools — Complete In-Depth Guide

> **"ORM (Object-Relational Mapping) bridges the gap between object-oriented Java and relational databases. Understanding ORM concepts is essential before diving into Hibernate and Spring Data JPA."**

---

## 📑 Table of Contents

1. [Introduction & The Impedance Mismatch](#1-introduction--the-impedance-mismatch)
2. [What is ORM?](#2-what-is-orm)
3. [ORM vs JDBC](#3-orm-vs-jdbc)
4. [JPA (Java Persistence API)](#4-jpa-java-persistence-api)
5. [JPA Annotations Deep Dive](#5-jpa-annotations-deep-dive)
6. [Entity Lifecycle & States](#6-entity-lifecycle--states)
7. [Entity Relationships](#7-entity-relationships)
8. [Cascade Types & Fetch Types](#8-cascade-types--fetch-types)
9. [Inheritance Mapping](#9-inheritance-mapping)
10. [JPQL & Criteria API](#10-jpql--criteria-api)
11. [Popular ORM Tools Comparison](#11-popular-orm-tools-comparison)
12. [N+1 Problem & Performance](#12-n1-problem--performance)
13. [Best Practices](#13-best-practices)
14. [Interview Questions & Answers (50+)](#14-interview-questions--answers-50)

---

## 1. Introduction & The Impedance Mismatch

### The Problem: Object-Relational Impedance Mismatch

Java thinks in **objects**. Databases think in **tables**. These two worlds don't naturally fit together.

```
JAVA WORLD (Objects):                 DATABASE WORLD (Tables):

class User {                          users table:
    Long id;                          ┌────┬────────┬─────────────────┐
    String name;                      │ id │  name  │     email       │
    String email;                     ├────┼────────┼─────────────────┤
    Address address;   ← Object!      │  1 │ Dilip  │ dilip@mail.com  │
    List<Order> orders; ← Collection! │  2 │ Alice  │ alice@mail.com  │
}                                     └────┴────────┴─────────────────┘

MISMATCHES:
1. Granularity:    Java has fine-grained objects (Address). DB has flat tables.
2. Inheritance:    Java has class hierarchy. Tables don't have inheritance.
3. Identity:       Java uses == and .equals(). DB uses primary keys.
4. Associations:   Java uses object references. DB uses foreign keys.
5. Navigation:     Java navigates: user.getAddress().getCity()
                   DB JOINs: SELECT ... FROM users JOIN addresses ON ...
```

### What Without ORM Looks Like (Raw JDBC)

```java
// ❌ Without ORM: LOTS of boilerplate for simple CRUD
public User findById(Long id) throws SQLException {
    String sql = "SELECT u.*, a.* FROM users u " +
                 "LEFT JOIN addresses a ON u.address_id = a.id " +
                 "WHERE u.id = ?";
    
    try (Connection conn = dataSource.getConnection();
         PreparedStatement pstmt = conn.prepareStatement(sql)) {
        
        pstmt.setLong(1, id);
        ResultSet rs = pstmt.executeQuery();
        
        if (rs.next()) {
            User user = new User();
            user.setId(rs.getLong("id"));
            user.setName(rs.getString("name"));
            user.setEmail(rs.getString("email"));
            
            Address address = new Address();
            address.setStreet(rs.getString("street"));
            address.setCity(rs.getString("city"));
            user.setAddress(address);
            
            return user;
        }
        return null;
    }
    // Imagine doing this for 50 entities with relationships... 😩
}
```

---

## 2. What is ORM?

**ORM (Object-Relational Mapping)** automatically maps Java objects to database tables and vice versa.

```
WITH ORM:

Java Object ←→ ORM Framework ←→ Database Table

@Entity
class User {         ORM automatically         users table
    @Id              generates SQL:             ┌────┬────────┐
    Long id;    ←─── SELECT, INSERT,  ────→     │ id │  name  │
    String name;     UPDATE, DELETE              │  1 │ Dilip  │
}                                               └────┴────────┘

// ✅ With ORM: Just one line!
User user = entityManager.find(User.class, 1L);
// ORM generates: SELECT * FROM users WHERE id = 1
// And maps the result to a User object automatically!
```

### How ORM Maps Objects to Tables

```
Java                              SQL
─────────────────────────────────────────────
@Entity class User          →     CREATE TABLE users (...)
@Id Long id                 →     id BIGINT PRIMARY KEY
String name                 →     name VARCHAR(255)
@Column(length=100)         →     VARCHAR(100)
@ManyToOne Address address  →     address_id BIGINT FOREIGN KEY
@OneToMany List<Order>      →     orders table has user_id FK

save(user)                  →     INSERT INTO users VALUES (...)
findById(1)                 →     SELECT * FROM users WHERE id = 1
user.setName("New")         →     UPDATE users SET name = 'New' WHERE id = 1
delete(user)                →     DELETE FROM users WHERE id = 1
```

---

## 3. ORM vs JDBC

| Feature | JDBC | ORM (JPA/Hibernate) |
|---------|------|---------------------|
| SQL | Write manually | Generated automatically |
| Mapping | Manual (ResultSet → Object) | Automatic (annotations) |
| Relationships | Manual JOINs | Automatic navigation |
| Transactions | Manual commit/rollback | Declarative (@Transactional) |
| Caching | None | First + second level cache |
| Lazy loading | Not available | Built-in |
| Database portability | Database-specific SQL | Database-agnostic |
| Boilerplate | High | Minimal |
| Performance control | Full control | May generate suboptimal SQL |
| Learning curve | Lower | Higher |

---

## 4. JPA (Java Persistence API)

### What is JPA?

JPA is a **specification** (set of interfaces), NOT an implementation. It defines HOW Java objects should be mapped to databases.

```
JPA Specification (Interfaces)
     ├── Hibernate (Implementation) ✅ Most popular
     ├── EclipseLink (Implementation)
     └── OpenJPA (Implementation)

Think of it like:
  JPA = JDBC Driver Interface
  Hibernate = MySQL JDBC Driver (one implementation)
  
  You code to the JPA interface.
  Hibernate provides the actual implementation.
  You can swap Hibernate for EclipseLink without changing code!
```

### JPA Architecture

```
┌──────────────────────────────────────────────┐
│              Your Application                │
│  @Entity User, UserRepository, UserService   │
├──────────────────────────────────────────────┤
│              JPA API                         │
│  EntityManager, @Entity, @Id, JPQL          │
├──────────────────────────────────────────────┤
│         JPA Implementation                   │
│  Hibernate / EclipseLink / OpenJPA          │
├──────────────────────────────────────────────┤
│              JDBC                            │
├──────────────────────────────────────────────┤
│         Database Driver                      │
│  MySQL Connector, PostgreSQL Driver          │
├──────────────────────────────────────────────┤
│           Database                           │
│  MySQL, PostgreSQL, Oracle, H2              │
└──────────────────────────────────────────────┘
```

---

## 5. JPA Annotations Deep Dive

### Entity & Table

```java
@Entity                    // This class maps to a database table
@Table(
    name = "users",        // Table name (default: class name)
    schema = "public",     // Database schema
    uniqueConstraints = @UniqueConstraint(columnNames = {"email"}),
    indexes = @Index(columnList = "name")
)
public class User {
    
    @Id                                // Primary key
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // Auto-increment
    private Long id;
    
    @Column(
        name = "user_name",           // Column name (default: field name)
        length = 100,                 // VARCHAR(100)
        nullable = false,             // NOT NULL
        unique = true,                // UNIQUE constraint
        updatable = false,            // Can't update after insert
        columnDefinition = "TEXT"     // Raw SQL type
    )
    private String name;
    
    @Column(nullable = false)
    private String email;
    
    @Enumerated(EnumType.STRING)      // Store enum as STRING (not ordinal!)
    private UserRole role;            // Stored as 'ADMIN', 'USER' in DB
    
    @Temporal(TemporalType.TIMESTAMP) // For java.util.Date
    private Date createdAt;
    
    // For Java 8+ LocalDateTime, no @Temporal needed
    private LocalDateTime updatedAt;
    
    @Transient                        // NOT mapped to database (ignored)
    private String fullName;          // Calculated field, not in DB
    
    @Lob                              // Large Object (CLOB/BLOB)
    private String biography;         // TEXT/CLOB in database
    
    @Lob
    private byte[] profilePicture;    // BLOB in database
    
    // Constructors, getters, setters...
}
```

### ID Generation Strategies

```java
// Strategy 1: IDENTITY — Database auto-increment (MySQL, PostgreSQL)
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;  // MySQL: AUTO_INCREMENT, PostgreSQL: SERIAL

// Strategy 2: SEQUENCE — Database sequence (PostgreSQL, Oracle)
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
@SequenceGenerator(name = "user_seq", sequenceName = "user_sequence", allocationSize = 1)
private Long id;

// Strategy 3: TABLE — Uses a separate table to generate IDs
@GeneratedValue(strategy = GenerationType.TABLE)
private Long id;

// Strategy 4: UUID — Universally Unique Identifier
@Id
@GeneratedValue(strategy = GenerationType.UUID)
private UUID id;

// Strategy 5: AUTO — Let JPA provider decide (default)
@GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
```

### Embedded Objects

```java
@Embeddable  // This class is NOT a separate table, it's embedded
public class Address {
    private String street;
    private String city;
    private String state;
    private String zipCode;
}

@Entity
public class User {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    
    @Embedded                    // Address fields are in the USERS table
    private Address address;     // Columns: street, city, state, zip_code
    
    @Embedded
    @AttributeOverrides({        // Override column names for second address
        @AttributeOverride(name = "street", column = @Column(name = "work_street")),
        @AttributeOverride(name = "city", column = @Column(name = "work_city"))
    })
    private Address workAddress;
}

// Resulting table:
// users (id, name, street, city, state, zip_code, work_street, work_city, ...)
```

---

## 6. Entity Lifecycle & States

```
┌────────────┐    persist()     ┌────────────┐
│   NEW      │ ───────────────► │  MANAGED   │
│ (Transient)│                  │(Persistent)│
│            │                  │            │
│ Not in DB  │                  │ In DB +    │
│ No ID      │                  │ tracked by │
│            │                  │ EntityMgr  │
└────────────┘                  └──────┬─────┘
                                   │       │
                           detach()/│       │ remove()
                           close()  │       │
                                   ▼       ▼
                            ┌──────────┐ ┌──────────┐
                            │ DETACHED │ │ REMOVED  │
                            │          │ │          │
                            │ In DB but│ │ Will be  │
                            │ NOT      │ │ deleted  │
                            │ tracked  │ │ from DB  │
                            └──────────┘ └──────────┘
```

```java
// NEW (Transient) — Just created, not in DB
User user = new User("Dilip", "dilip@mail.com");

// MANAGED (Persistent) — Tracked by EntityManager, in DB
entityManager.persist(user);  // INSERT into database
// Now any changes to 'user' are AUTO-SAVED! (dirty checking)
user.setName("Dilip Kumar");  // This UPDATE happens automatically at flush!

// DETACHED — Was managed, now disconnected
entityManager.detach(user);
user.setName("New Name");     // This change is NOT saved to DB!

// Re-attach
entityManager.merge(user);    // Now it's managed again, changes will be saved

// REMOVED — Scheduled for deletion
entityManager.remove(user);   // DELETE from database at flush/commit
```

---

## 7. Entity Relationships

### OneToOne

```java
// User has ONE Profile (and vice versa)

@Entity
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    
    @OneToOne(cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    @JoinColumn(name = "profile_id")  // FK column in users table
    private Profile profile;
}

@Entity
public class Profile {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String bio;
    
    @OneToOne(mappedBy = "profile")  // Inverse side (no FK here)
    private User user;
}

// Tables:
// users:    id | name  | profile_id (FK)
// profiles: id | bio
```

### OneToMany / ManyToOne

```java
// One User has MANY Orders. Each Order belongs to ONE User.

@Entity
public class User {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<Order> orders = new ArrayList<>();
    
    // Helper methods (keep both sides in sync!)
    public void addOrder(Order order) {
        orders.add(order);
        order.setUser(this);
    }
    
    public void removeOrder(Order order) {
        orders.remove(order);
        order.setUser(null);
    }
}

@Entity
@Table(name = "orders")
public class Order {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String product;
    private double amount;
    
    @ManyToOne(fetch = FetchType.LAZY)     // LAZY is best for ManyToOne
    @JoinColumn(name = "user_id")          // FK column in orders table
    private User user;
}

// Tables:
// users:  id | name
// orders: id | product | amount | user_id (FK → users.id)
```

### ManyToMany

```java
// Students can enroll in MANY Courses. Courses have MANY Students.

@Entity
public class Student {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String name;
    
    @ManyToMany(cascade = {CascadeType.PERSIST, CascadeType.MERGE})
    @JoinTable(
        name = "student_courses",                       // Join table name
        joinColumns = @JoinColumn(name = "student_id"), // FK to this entity
        inverseJoinColumns = @JoinColumn(name = "course_id") // FK to other entity
    )
    private Set<Course> courses = new HashSet<>();
}

@Entity
public class Course {
    @Id @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    private String title;
    
    @ManyToMany(mappedBy = "courses")  // Inverse side
    private Set<Student> students = new HashSet<>();
}

// Tables:
// students:        id | name
// courses:         id | title
// student_courses: student_id (FK) | course_id (FK)  ← JOIN TABLE
```

---

## 8. Cascade Types & Fetch Types

### Cascade Types

```java
// CASCADE: When you do something to parent, do the SAME to children

@OneToMany(cascade = CascadeType.ALL)  // All operations cascade

// Individual cascade types:
CascadeType.PERSIST   // save parent → also save children
CascadeType.MERGE     // update parent → also update children  
CascadeType.REMOVE    // delete parent → also delete children
CascadeType.REFRESH   // refresh parent → also refresh children
CascadeType.DETACH    // detach parent → also detach children
CascadeType.ALL       // All of the above

// Example:
User user = new User("Dilip");
user.addOrder(new Order("Laptop", 999.99));
user.addOrder(new Order("Phone", 699.99));

entityManager.persist(user);  
// With CascadeType.PERSIST: Saves user AND both orders!
// Without cascade: Must persist each order separately
```

### Fetch Types

```java
// EAGER: Load related data IMMEDIATELY with the parent
@ManyToOne(fetch = FetchType.EAGER)
private User user;
// SELECT * FROM orders JOIN users ON orders.user_id = users.id

// LAZY: Load related data ONLY WHEN ACCESSED
@OneToMany(fetch = FetchType.LAZY)
private List<Order> orders;
// SELECT * FROM users WHERE id = 1
// (orders are NOT loaded yet)
// user.getOrders().size(); ← NOW the orders are loaded!
// SELECT * FROM orders WHERE user_id = 1

// DEFAULTS:
// @OneToOne   → EAGER (load immediately)
// @ManyToOne  → EAGER (load immediately)
// @OneToMany  → LAZY  (load on access) ✅
// @ManyToMany → LAZY  (load on access) ✅

// BEST PRACTICE: Use LAZY everywhere, override with EAGER only when needed
@ManyToOne(fetch = FetchType.LAZY)  // Override default EAGER to LAZY
```

---

## 9. Inheritance Mapping

### Strategy 1: Single Table (Default, Best Performance)

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "payment_type", discriminatorType = DiscriminatorType.STRING)
public abstract class Payment {
    @Id @GeneratedValue private Long id;
    private double amount;
}

@Entity
@DiscriminatorValue("CREDIT")
public class CreditCardPayment extends Payment {
    private String cardNumber;
}

@Entity
@DiscriminatorValue("BANK")
public class BankTransfer extends Payment {
    private String bankAccount;
}

// ONE table for ALL types:
// payments: id | amount | payment_type | card_number | bank_account
//           1  | 100.00 | CREDIT       | 4111...     | NULL
//           2  | 200.00 | BANK         | NULL        | ACC123
```

### Strategy 2: Table Per Class

```java
@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)
// Separate table for each concrete class:
// credit_card_payments: id | amount | card_number
// bank_transfers:       id | amount | bank_account
```

### Strategy 3: Joined Table

```java
@Inheritance(strategy = InheritanceType.JOINED)
// Base table + separate tables for subclass-specific fields:
// payments:              id | amount
// credit_card_payments:  id (FK) | card_number
// bank_transfers:        id (FK) | bank_account
// Requires JOINs to query
```

---

## 10. JPQL & Criteria API

### JPQL (Java Persistence Query Language)

```java
// JPQL uses ENTITY NAMES and FIELD NAMES (not table/column names!)

// Simple query
List<User> users = entityManager
    .createQuery("SELECT u FROM User u WHERE u.name = :name", User.class)
    .setParameter("name", "Dilip")
    .getResultList();

// JOIN query
List<User> users = entityManager
    .createQuery("SELECT u FROM User u JOIN u.orders o WHERE o.amount > :amount", User.class)
    .setParameter("amount", 100.0)
    .getResultList();

// Aggregate
Long count = entityManager
    .createQuery("SELECT COUNT(u) FROM User u WHERE u.active = true", Long.class)
    .getSingleResult();

// Update (bulk)
int updated = entityManager
    .createQuery("UPDATE User u SET u.active = false WHERE u.lastLogin < :date")
    .setParameter("date", sixMonthsAgo)
    .executeUpdate();

// Named query (defined on entity)
@Entity
@NamedQuery(name = "User.findByEmail", 
            query = "SELECT u FROM User u WHERE u.email = :email")
public class User { ... }

// Use named query
User user = entityManager
    .createNamedQuery("User.findByEmail", User.class)
    .setParameter("email", "dilip@mail.com")
    .getSingleResult();
```

---

## 11. Popular ORM Tools Comparison

| ORM | Language | Pros | Cons |
|-----|----------|------|------|
| **Hibernate** | Java | Most popular, feature-rich | Complex, steep learning curve |
| **EclipseLink** | Java | JPA reference impl, good perf | Smaller community |
| **MyBatis** | Java | SQL-focused (SQL mapper) | Not full ORM |
| **jOOQ** | Java | Type-safe SQL, code generation | Not full ORM, commercial |
| **Spring Data JPA** | Java | Repository pattern, minimal code | Built on Hibernate |
| Django ORM | Python | Built-in, easy to use | Python only |
| SQLAlchemy | Python | Flexible, powerful | Complex |
| Entity Framework | C# | Microsoft ecosystem | .NET only |
| TypeORM | TypeScript | Modern, decorators | JavaScript/TS only |

---

## 12. N+1 Problem & Performance

### The N+1 Query Problem

```java
// PROBLEM: Fetching N users, then N additional queries for their orders

// This innocent-looking code:
List<User> users = entityManager
    .createQuery("SELECT u FROM User u", User.class)
    .getResultList();

for (User user : users) {
    System.out.println(user.getOrders().size());  // Triggers LAZY loading!
}

// Generates N+1 SQL queries:
// Query 1:   SELECT * FROM users                    ← 1 query
// Query 2:   SELECT * FROM orders WHERE user_id = 1 ← N queries
// Query 3:   SELECT * FROM orders WHERE user_id = 2
// ...
// Query N+1: SELECT * FROM orders WHERE user_id = N

// If you have 100 users, that's 101 queries!!! 😱
```

### Solutions

```java
// Solution 1: JOIN FETCH (load everything in ONE query)
List<User> users = entityManager
    .createQuery("SELECT u FROM User u JOIN FETCH u.orders", User.class)
    .getResultList();
// Generates: SELECT u.*, o.* FROM users u JOIN orders o ON u.id = o.user_id
// Just 1 query! ✅

// Solution 2: @EntityGraph
@EntityGraph(attributePaths = {"orders"})
List<User> findAll();

// Solution 3: @BatchSize (Hibernate-specific)
@OneToMany(mappedBy = "user")
@BatchSize(size = 20)  // Load orders in batches of 20 users at a time
private List<Order> orders;
// Instead of N queries, generates N/20 queries
```

---

## 13. Best Practices

1. **Always use LAZY fetching** — Override EAGER defaults on @ManyToOne
2. **Use JOIN FETCH** for known use cases to avoid N+1
3. **Use DTOs for projections** — Don't return entities from controllers
4. **Use @Transactional** — Mark service methods, not controllers
5. **Avoid bidirectional relationships** when unidirectional suffices
6. **Use helper methods** to keep bidirectional relationships in sync
7. **Use Set instead of List** for @ManyToMany to avoid duplicates
8. **Enable SQL logging in dev** — `spring.jpa.show-sql=true`
9. **Use database migrations** — Flyway or Liquibase
10. **Don't use CascadeType.ALL blindly** — Think about each operation

---

## 14. Interview Questions & Answers (50+)

### Beginner Level

**Q1: What is ORM?**
**A:** Object-Relational Mapping — automatic mapping between Java objects and database tables. Eliminates manual JDBC code for CRUD operations.

**Q2: What is JPA?**
**A:** Java Persistence API — a specification defining standard ORM interfaces. Hibernate, EclipseLink are implementations. You code to JPA, swap implementations freely.

**Q3: What is the difference between JPA and Hibernate?**
**A:** JPA is the specification (interfaces). Hibernate is an implementation. JPA defines `@Entity`, `EntityManager`. Hibernate provides the actual code that executes SQL.

**Q4: What is an Entity?**
**A:** A Java class annotated with `@Entity` that maps to a database table. Each instance represents a row.

**Q5: What is `@Id`?**
**A:** Marks a field as the primary key of the entity. Required for every entity.

**Q6: What are the four entity states?**
**A:** Transient (new, not in DB), Managed (tracked by EntityManager), Detached (was managed, now disconnected), Removed (scheduled for deletion).

**Q7: What is lazy loading?**
**A:** Related data is loaded ONLY when first accessed, not when the parent is loaded. Reduces initial query size. Default for @OneToMany and @ManyToMany.

**Q8: What is the N+1 problem?**
**A:** Loading N entities triggers N additional queries for lazy-loaded relationships. Solution: JOIN FETCH, @EntityGraph, @BatchSize.

---

### Intermediate Level

**Q9: Explain cascade types.**
**A:** PERSIST (save with parent), MERGE (update), REMOVE (delete), REFRESH, DETACH, ALL. Cascade propagates operations from parent to children.

**Q10: What is `mappedBy`?**
**A:** Indicates the inverse (non-owning) side of a bidirectional relationship. The side WITHOUT `mappedBy` owns the foreign key.

**Q11: What is `@Embeddable` vs `@Entity`?**
**A:** @Entity maps to its own table with its own lifecycle. @Embeddable is embedded INTO another entity's table — no separate table, no own ID.

**Q12: What is JPQL?**
**A:** JPA Query Language — SQL-like but uses entity/field names instead of table/column names. Database-agnostic.

**Q13: Explain inheritance mapping strategies.**
**A:** SINGLE_TABLE (one table, discriminator column — best performance). JOINED (base + subclass tables, requires JOINs). TABLE_PER_CLASS (separate tables, no JOINs).

**Q14: What is `orphanRemoval = true`?**
**A:** When a child is removed from the parent's collection, it's automatically deleted from DB. Used with @OneToMany.

**Q15: What is dirty checking?**
**A:** EntityManager tracks managed entities. If any field changes, it automatically generates an UPDATE at flush time. No explicit save needed!

---

### Advanced Level

**Q16: What is the difference between `persist()` and `merge()`?**
**A:** `persist()`: makes a NEW entity managed (INSERT). `merge()`: copies state of detached entity into managed entity (INSERT or UPDATE). persist throws if entity has ID, merge doesn't.

**Q17: First-level vs second-level cache?**
**A:** First-level: per EntityManager/session, automatic, cleared on close. Second-level: shared across sessions, configurable (EhCache, Redis), must be enabled.

**Q18: What is optimistic locking?**
**A:** Uses @Version field. Before update, check if version matches. If someone else updated (version changed), throw OptimisticLockException. No database locks held.

**Q19: What is `@EntityGraph`?**
**A:** Defines which associations to eagerly fetch for a specific query. Overrides LAZY fetching without changing the entity mapping.

**Q20: What is `@NaturalId`?**
**A:** A business key (like email, ISBN) used for lookups instead of surrogate ID. Hibernate provides `byNaturalId()` for efficient lookups.

---

### Rapid-Fire (Q21–Q50)

**Q21: Default fetch type for @ManyToOne?** EAGER. Override to LAZY!

**Q22: Default fetch type for @OneToMany?** LAZY (good default).

**Q23: What is `@JoinColumn`?** Specifies the foreign key column name.

**Q24: What is `@JoinTable`?** Specifies the join table for @ManyToMany.

**Q25: What is `@GeneratedValue`?** Auto-generates primary key values (IDENTITY, SEQUENCE, TABLE, UUID, AUTO).

**Q26: What is `@Transient`?** Field is NOT mapped to database. Ignored by JPA.

**Q27: What is `@Enumerated`?** Maps Java enum to DB. STRING stores name, ORDINAL stores position number.

**Q28: Why use STRING over ORDINAL for enums?** ORDINAL breaks if enum order changes. STRING is readable and safe.

**Q29: What is `@Temporal`?** Specifies date precision: DATE, TIME, TIMESTAMP. Not needed for Java 8 LocalDate/LocalDateTime.

**Q30: What is `@Lob`?** Maps to CLOB (String) or BLOB (byte[]) for large data.

**Q31: What is `EntityManager`?** JPA interface for CRUD operations: persist, find, merge, remove, createQuery.

**Q32: What is `EntityManagerFactory`?** Factory that creates EntityManager instances. One per persistence unit.

**Q33: What is a persistence unit?** Configuration for JPA defined in persistence.xml or Spring properties.

**Q34: What is `flush()`?** Synchronizes EntityManager with database. Writes pending changes to DB.

**Q35: What is `clear()`?** Detaches all managed entities. Clears first-level cache.

**Q36: What is `@Transactional`?** Spring annotation for declarative transaction management.

**Q37: What is the difference between `save()` and `saveAndFlush()`?** save() may delay DB write. saveAndFlush() writes immediately.

**Q38: What is `@Version`?** Enables optimistic locking. Auto-incremented on each update.

**Q39: What is `@MappedSuperclass`?** Base class with common fields. Not an entity, no table. Subclasses inherit fields.

**Q40: What is `@Query` in Spring Data JPA?** Custom JPQL/native SQL on repository methods.

**Q41: What is a DTO projection?** Returning only needed fields via constructor expression or interface projection. Avoids loading full entities.

**Q42: What is `@Modifying`?** Required for UPDATE/DELETE @Query methods in Spring Data JPA.

**Q43: What is Spring Data JPA repository?** Interface with auto-generated CRUD methods: findById, save, delete, findAll.

**Q44: What is `@CreatedDate`?** Auto-populates creation timestamp. Requires @EnableJpaAuditing.

**Q45: What is `@LastModifiedDate`?** Auto-populates last update timestamp.

**Q46: What is native query?** Raw SQL (not JPQL). Use `@Query(nativeQuery = true)`.

**Q47: Can JPA generate tables?** Yes. `spring.jpa.hibernate.ddl-auto=update` (dev only!).

**Q48: What is `spring.jpa.show-sql`?** Logs generated SQL statements. Enable in dev.

**Q49: What is `@Table(name = "...")`?** Customizes table name. Default is class name.

**Q50: What is `@Column(name = "...")`?** Customizes column name. Default is field name.

---

## 📚 References

- [JPA Specification](https://jakarta.ee/specifications/persistence/)
- [Hibernate Documentation](https://hibernate.org/orm/documentation/)
- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/)
- [Baeldung JPA Tutorials](https://www.baeldung.com/learn-jpa-hibernate)
- [Vlad Mihalcea Blog](https://vladmihalcea.com/) — JPA/Hibernate expert

---

> **Previous Topic:** [← 09 - REST API](../09-rest-api-and-web-services/README.md)  
> **Next Topic:** [11 - Hibernate →](../11-hibernate/README.md)
