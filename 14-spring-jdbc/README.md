# 🗃️ Spring JDBC — Complete In-Depth Guide

> **"Spring JDBC simplifies raw JDBC by eliminating boilerplate code while keeping you in full control of SQL. JdbcTemplate is the bridge between raw JDBC and full ORM."**

---

## 📑 Table of Contents

1. [Introduction & Why Spring JDBC](#1-introduction--why-spring-jdbc)
2. [JdbcTemplate](#2-jdbctemplate)
3. [NamedParameterJdbcTemplate](#3-namedparameterjdbctemplate)
4. [RowMapper & ResultSetExtractor](#4-rowmapper--resultsetextractor)
5. [CRUD Operations](#5-crud-operations)
6. [Batch Operations](#6-batch-operations)
7. [SimpleJdbcInsert & SimpleJdbcCall](#7-simplejdbcinsert--simplejdbccall)
8. [Transaction Management](#8-transaction-management)
9. [Exception Handling](#9-exception-handling)
10. [Spring JDBC vs JPA](#10-spring-jdbc-vs-jpa)
11. [Best Practices](#11-best-practices)
12. [Interview Questions & Answers (50+)](#12-interview-questions--answers-50)

---

## 1. Introduction & Why Spring JDBC

### The Problem with Raw JDBC

```java
// ❌ Raw JDBC: 20+ lines for a simple query!
public User findById(Long id) {
    Connection conn = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;
    try {
        conn = dataSource.getConnection();          // Get connection
        pstmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
        pstmt.setLong(1, id);                       // Set parameters
        rs = pstmt.executeQuery();                   // Execute
        if (rs.next()) {
            User user = new User();                  // Manual mapping
            user.setId(rs.getLong("id"));
            user.setName(rs.getString("name"));
            return user;
        }
        return null;
    } catch (SQLException e) {
        throw new RuntimeException(e);               // Exception handling
    } finally {
        if (rs != null) try { rs.close(); } catch (SQLException e) {}
        if (pstmt != null) try { pstmt.close(); } catch (SQLException e) {}
        if (conn != null) try { conn.close(); } catch (SQLException e) {}
    }
}

// ✅ Spring JDBC: 3 lines!
public User findById(Long id) {
    return jdbcTemplate.queryForObject(
        "SELECT * FROM users WHERE id = ?",
        new UserRowMapper(), id
    );
}
```

### What Spring JDBC Handles For You

| Responsibility | Raw JDBC | Spring JDBC |
|----------------|----------|-------------|
| Get/close connection | YOU | Spring ✅ |
| Create statement | YOU | Spring ✅ |
| Set parameters | YOU | Spring ✅ |
| Execute query | YOU | Spring ✅ |
| Iterate ResultSet | YOU | Spring ✅ |
| Handle exceptions | YOU | Spring ✅ |
| Close resources | YOU | Spring ✅ |
| **Write SQL** | **YOU** | **YOU** |
| **Map results** | **YOU** | **YOU** |

---

## 2. JdbcTemplate

### Setup

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
</dependency>
```

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
```

### Core Methods

```java
@Repository
public class UserJdbcRepository {
    
    private final JdbcTemplate jdbcTemplate;
    
    public UserJdbcRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;  // Auto-configured by Spring Boot
    }
    
    // ═══ QUERY SINGLE VALUE ═══
    public int count() {
        return jdbcTemplate.queryForObject("SELECT COUNT(*) FROM users", Integer.class);
    }
    
    public String findNameById(Long id) {
        return jdbcTemplate.queryForObject(
            "SELECT name FROM users WHERE id = ?", String.class, id);
    }
    
    // ═══ QUERY SINGLE ROW → Object ═══
    public User findById(Long id) {
        return jdbcTemplate.queryForObject(
            "SELECT * FROM users WHERE id = ?",
            (rs, rowNum) -> new User(
                rs.getLong("id"),
                rs.getString("name"),
                rs.getString("email")
            ),
            id  // Parameter value for ?
        );
    }
    
    // ═══ QUERY MULTIPLE ROWS → List ═══
    public List<User> findAll() {
        return jdbcTemplate.query(
            "SELECT * FROM users ORDER BY name",
            (rs, rowNum) -> new User(
                rs.getLong("id"),
                rs.getString("name"),
                rs.getString("email")
            )
        );
    }
    
    // ═══ INSERT ═══
    public int save(User user) {
        return jdbcTemplate.update(
            "INSERT INTO users (name, email) VALUES (?, ?)",
            user.getName(), user.getEmail()
        );
    }
    
    // ═══ INSERT with generated key ═══
    public Long saveAndGetId(User user) {
        KeyHolder keyHolder = new GeneratedKeyHolder();
        
        jdbcTemplate.update(connection -> {
            PreparedStatement ps = connection.prepareStatement(
                "INSERT INTO users (name, email) VALUES (?, ?)",
                Statement.RETURN_GENERATED_KEYS
            );
            ps.setString(1, user.getName());
            ps.setString(2, user.getEmail());
            return ps;
        }, keyHolder);
        
        return keyHolder.getKey().longValue();
    }
    
    // ═══ UPDATE ═══
    public int update(User user) {
        return jdbcTemplate.update(
            "UPDATE users SET name = ?, email = ? WHERE id = ?",
            user.getName(), user.getEmail(), user.getId()
        );
    }
    
    // ═══ DELETE ═══
    public int deleteById(Long id) {
        return jdbcTemplate.update("DELETE FROM users WHERE id = ?", id);
    }
    
    // ═══ CHECK EXISTS ═══
    public boolean existsById(Long id) {
        Integer count = jdbcTemplate.queryForObject(
            "SELECT COUNT(*) FROM users WHERE id = ?", Integer.class, id);
        return count != null && count > 0;
    }
}
```

---

## 3. NamedParameterJdbcTemplate

```java
// Named parameters are MORE READABLE than ? placeholders!

@Repository
public class UserNamedRepository {
    
    private final NamedParameterJdbcTemplate namedTemplate;
    
    public UserNamedRepository(NamedParameterJdbcTemplate namedTemplate) {
        this.namedTemplate = namedTemplate;
    }
    
    // ═══ Using Map for parameters ═══
    public User findById(Long id) {
        String sql = "SELECT * FROM users WHERE id = :id";
        Map<String, Object> params = Map.of("id", id);
        
        return namedTemplate.queryForObject(sql, params, new UserRowMapper());
    }
    
    // ═══ Using MapSqlParameterSource ═══
    public List<User> findByNameAndAge(String name, int minAge) {
        String sql = "SELECT * FROM users WHERE name LIKE :name AND age >= :minAge";
        
        MapSqlParameterSource params = new MapSqlParameterSource()
            .addValue("name", "%" + name + "%")
            .addValue("minAge", minAge);
        
        return namedTemplate.query(sql, params, new UserRowMapper());
    }
    
    // ═══ Using BeanPropertySqlParameterSource ═══
    public int save(User user) {
        String sql = "INSERT INTO users (name, email, age) VALUES (:name, :email, :age)";
        // Automatically maps bean properties to named parameters!
        BeanPropertySqlParameterSource params = new BeanPropertySqlParameterSource(user);
        return namedTemplate.update(sql, params);
    }
    
    // ═══ IN clause ═══
    public List<User> findByIds(List<Long> ids) {
        String sql = "SELECT * FROM users WHERE id IN (:ids)";
        MapSqlParameterSource params = new MapSqlParameterSource("ids", ids);
        return namedTemplate.query(sql, params, new UserRowMapper());
    }
}
```

---

## 4. RowMapper & ResultSetExtractor

### RowMapper (One Row → One Object)

```java
// Reusable RowMapper (recommended — extract to separate class)
public class UserRowMapper implements RowMapper<User> {
    @Override
    public User mapRow(ResultSet rs, int rowNum) throws SQLException {
        User user = new User();
        user.setId(rs.getLong("id"));
        user.setName(rs.getString("name"));
        user.setEmail(rs.getString("email"));
        user.setAge(rs.getInt("age"));
        user.setActive(rs.getBoolean("active"));
        user.setCreatedAt(rs.getTimestamp("created_at").toLocalDateTime());
        return user;
    }
}

// Usage:
List<User> users = jdbcTemplate.query("SELECT * FROM users", new UserRowMapper());
User user = jdbcTemplate.queryForObject("SELECT * FROM users WHERE id = ?", new UserRowMapper(), 1L);

// Lambda RowMapper (for simple cases):
List<User> users = jdbcTemplate.query("SELECT * FROM users",
    (rs, rowNum) -> new User(rs.getLong("id"), rs.getString("name"), rs.getString("email"))
);

// BeanPropertyRowMapper (auto-maps by column name → property name)
List<User> users = jdbcTemplate.query("SELECT * FROM users",
    new BeanPropertyRowMapper<>(User.class));
// Column "user_name" → property "userName" (snake_case → camelCase automatic!)
```

### ResultSetExtractor (Entire ResultSet → Custom Object)

```java
// When you need to process ALL rows together (e.g., one-to-many)

public class UserWithOrdersExtractor implements ResultSetExtractor<List<User>> {
    @Override
    public List<User> extractData(ResultSet rs) throws SQLException {
        Map<Long, User> userMap = new LinkedHashMap<>();
        
        while (rs.next()) {
            Long userId = rs.getLong("user_id");
            
            User user = userMap.computeIfAbsent(userId, id -> {
                try {
                    User u = new User();
                    u.setId(id);
                    u.setName(rs.getString("user_name"));
                    u.setOrders(new ArrayList<>());
                    return u;
                } catch (SQLException e) { throw new RuntimeException(e); }
            });
            
            // Add order to user (from same row)
            if (rs.getLong("order_id") != 0) {
                Order order = new Order();
                order.setId(rs.getLong("order_id"));
                order.setProduct(rs.getString("product"));
                user.getOrders().add(order);
            }
        }
        return new ArrayList<>(userMap.values());
    }
}

// Usage:
String sql = "SELECT u.id AS user_id, u.name AS user_name, " +
             "o.id AS order_id, o.product " +
             "FROM users u LEFT JOIN orders o ON u.id = o.user_id";
List<User> usersWithOrders = jdbcTemplate.query(sql, new UserWithOrdersExtractor());
```

---

## 5. CRUD Operations

### Complete Repository Example

```java
@Repository
public class ProductRepository {
    
    private final JdbcTemplate jdbc;
    
    public ProductRepository(JdbcTemplate jdbc) {
        this.jdbc = jdbc;
    }
    
    private final RowMapper<Product> rowMapper = (rs, rowNum) -> new Product(
        rs.getLong("id"),
        rs.getString("name"),
        rs.getBigDecimal("price"),
        rs.getInt("stock"),
        rs.getString("category"),
        rs.getTimestamp("created_at").toLocalDateTime()
    );
    
    public List<Product> findAll() {
        return jdbc.query("SELECT * FROM products ORDER BY name", rowMapper);
    }
    
    public Optional<Product> findById(Long id) {
        try {
            Product product = jdbc.queryForObject(
                "SELECT * FROM products WHERE id = ?", rowMapper, id);
            return Optional.ofNullable(product);
        } catch (EmptyResultDataAccessException e) {
            return Optional.empty();
        }
    }
    
    public List<Product> findByCategory(String category) {
        return jdbc.query("SELECT * FROM products WHERE category = ?", rowMapper, category);
    }
    
    public List<Product> search(String keyword) {
        return jdbc.query(
            "SELECT * FROM products WHERE name LIKE ? OR category LIKE ?",
            rowMapper, "%" + keyword + "%", "%" + keyword + "%");
    }
    
    public Product save(Product product) {
        KeyHolder keyHolder = new GeneratedKeyHolder();
        jdbc.update(conn -> {
            PreparedStatement ps = conn.prepareStatement(
                "INSERT INTO products (name, price, stock, category) VALUES (?, ?, ?, ?)",
                Statement.RETURN_GENERATED_KEYS);
            ps.setString(1, product.getName());
            ps.setBigDecimal(2, product.getPrice());
            ps.setInt(3, product.getStock());
            ps.setString(4, product.getCategory());
            return ps;
        }, keyHolder);
        product.setId(keyHolder.getKey().longValue());
        return product;
    }
    
    public int update(Product product) {
        return jdbc.update(
            "UPDATE products SET name=?, price=?, stock=?, category=? WHERE id=?",
            product.getName(), product.getPrice(), product.getStock(),
            product.getCategory(), product.getId());
    }
    
    public int deleteById(Long id) {
        return jdbc.update("DELETE FROM products WHERE id = ?", id);
    }
}
```

---

## 6. Batch Operations

```java
// Batch INSERT — much faster than individual inserts!
public int[] batchInsert(List<User> users) {
    return jdbcTemplate.batchUpdate(
        "INSERT INTO users (name, email) VALUES (?, ?)",
        new BatchPreparedStatementSetter() {
            @Override
            public void setValues(PreparedStatement ps, int i) throws SQLException {
                ps.setString(1, users.get(i).getName());
                ps.setString(2, users.get(i).getEmail());
            }
            @Override
            public int getBatchSize() {
                return users.size();
            }
        }
    );
}

// Batch with NamedParameterJdbcTemplate (cleaner)
public int[] batchInsertNamed(List<User> users) {
    SqlParameterSource[] batch = users.stream()
        .map(BeanPropertySqlParameterSource::new)
        .toArray(SqlParameterSource[]::new);
    
    return namedTemplate.batchUpdate(
        "INSERT INTO users (name, email) VALUES (:name, :email)", batch);
}
```

---

## 7. SimpleJdbcInsert & SimpleJdbcCall

```java
// SimpleJdbcInsert — Easy inserts without writing SQL!
@Repository
public class UserSimpleRepository {
    
    private final SimpleJdbcInsert insert;
    
    public UserSimpleRepository(JdbcTemplate jdbcTemplate) {
        this.insert = new SimpleJdbcInsert(jdbcTemplate)
            .withTableName("users")
            .usingGeneratedKeyColumns("id");
    }
    
    public Long save(User user) {
        Map<String, Object> params = new HashMap<>();
        params.put("name", user.getName());
        params.put("email", user.getEmail());
        
        return insert.executeAndReturnKey(params).longValue();
    }
}

// SimpleJdbcCall — Call stored procedures
SimpleJdbcCall call = new SimpleJdbcCall(jdbcTemplate)
    .withProcedureName("get_user_by_id");

Map<String, Object> result = call.execute(Map.of("user_id", 1));
```

---

## 8. Transaction Management

```java
// Declarative transactions (recommended)
@Service
@Transactional
public class TransferService {
    
    private final JdbcTemplate jdbc;
    
    @Transactional  // If ANY exception → rollback ALL changes
    public void transfer(Long fromId, Long toId, BigDecimal amount) {
        // Debit
        int debitRows = jdbc.update(
            "UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?",
            amount, fromId, amount);
        
        if (debitRows == 0) {
            throw new InsufficientFundsException("Insufficient balance");
        }
        
        // Credit
        jdbc.update("UPDATE accounts SET balance = balance + ? WHERE id = ?", amount, toId);
        
        // Log
        jdbc.update("INSERT INTO transfers (from_id, to_id, amount) VALUES (?, ?, ?)",
            fromId, toId, amount);
        
        // If exception here → ALL three operations are rolled back!
    }
}
```

---

## 9. Exception Handling

```
Spring JDBC translates SQL exceptions to a consistent hierarchy:

SQLException (JDBC)
    ↓ Spring translates to:
DataAccessException (Spring)
    ├── DataIntegrityViolationException  (unique constraint, FK violation)
    ├── DuplicateKeyException            (duplicate primary key)
    ├── EmptyResultDataAccessException   (expected row not found)
    ├── IncorrectResultSizeDataAccessException (wrong number of results)
    ├── BadSqlGrammarException           (syntax error in SQL)
    ├── DataAccessResourceFailureException (can't connect to DB)
    └── DeadlockLoserDataAccessException (deadlock detected)

These are UNCHECKED exceptions (RuntimeException) — no try-catch needed!
```

---

## 10. Spring JDBC vs JPA

| Feature | Spring JDBC | Spring Data JPA |
|---------|------------|-----------------|
| SQL control | Full (write your SQL) | Limited (generated) |
| Boilerplate | Medium | Minimal |
| Performance | Excellent (you optimize SQL) | Good (may generate suboptimal) |
| Relationships | Manual JOINs | Automatic navigation |
| Caching | None | First + second level |
| Learning curve | Lower | Higher |
| Use when | Complex SQL, performance-critical | Standard CRUD, rapid dev |

---

## 11. Best Practices

1. **Use NamedParameterJdbcTemplate** — More readable than `?` placeholders
2. **Extract RowMappers** — Reusable, testable, DRY
3. **Use BeanPropertyRowMapper** for simple cases — Auto-maps snake_case to camelCase
4. **Handle EmptyResultDataAccessException** — Return Optional instead of null
5. **Use batch operations** for bulk inserts — Orders of magnitude faster
6. **Use @Transactional** — Declarative transaction management
7. **Never concatenate SQL** — Always use parameterized queries
8. **Use SimpleJdbcInsert** for simple inserts — No SQL needed
9. **Use KeyHolder** to get generated IDs

---

## 12. Interview Questions & Answers (50+)

### Beginner

**Q1: What is JdbcTemplate?** Spring class that simplifies JDBC by handling connection management, statement creation, exception handling, and resource cleanup.

**Q2: JdbcTemplate vs raw JDBC?** JdbcTemplate eliminates boilerplate (connection/statement/ResultSet management), translates exceptions, and manages resources automatically.

**Q3: What is RowMapper?** Interface that maps a single ResultSet row to a Java object. Implement `mapRow(ResultSet, int)`.

**Q4: Difference between `query` and `queryForObject`?** `query` returns List. `queryForObject` returns single object (throws exception if 0 or >1 rows).

**Q5: What is `update()` method?** Executes INSERT, UPDATE, DELETE. Returns number of rows affected.

**Q6: How to get auto-generated key after insert?** Use `KeyHolder` with `update()` callback.

**Q7: What is BeanPropertyRowMapper?** Auto-maps ResultSet columns to Java bean properties by name convention (snake_case → camelCase).

**Q8: What is NamedParameterJdbcTemplate?** JdbcTemplate variant using named parameters (:name) instead of positional (?) for better readability.

---

### Intermediate

**Q9: RowMapper vs ResultSetExtractor?** RowMapper: one row → one object (called per row). ResultSetExtractor: entire ResultSet → custom result (called once).

**Q10: What is SqlParameterSource?** Interface for named parameter values. Implementations: MapSqlParameterSource, BeanPropertySqlParameterSource.

**Q11: How does Spring translate SQL exceptions?** Uses `SQLExceptionTranslator` to convert JDBC `SQLException` to Spring's `DataAccessException` hierarchy.

**Q12: What is SimpleJdbcInsert?** Simplifies INSERT without writing SQL. Auto-discovers table metadata. Supports generated key retrieval.

**Q13: How to handle batch operations?** `batchUpdate()` with `BatchPreparedStatementSetter` or `SqlParameterSource[]`. Much faster than individual inserts.

**Q14: What is DataAccessException?** Root of Spring's data access exception hierarchy. Unchecked (RuntimeException). Consistent across JDBC, JPA, etc.

**Q15: How does @Transactional work with JdbcTemplate?** Spring creates proxy, begins transaction before method, commits on success, rolls back on RuntimeException.

---

### Advanced

**Q16: When to use Spring JDBC vs JPA?** JDBC: complex queries, performance-critical, reporting, legacy databases. JPA: standard CRUD, object navigation, rapid development.

**Q17: How to map one-to-many with JdbcTemplate?** Use `ResultSetExtractor` with a Map to group parent entities and collect children.

**Q18: What is JdbcClient (Spring 6.1+)?** Fluent API replacing JdbcTemplate. Modern, readable: `jdbcClient.sql("...").param("id", 1).query(User.class).single()`.

**Q19: How to call stored procedures?** `SimpleJdbcCall` or `jdbcTemplate.call()`.

**Q20: What is SQLExceptionTranslator?** Translates vendor-specific SQL error codes to Spring exceptions.

---

### Rapid-Fire (Q21–Q50)

**Q21: Default connection pool in Spring Boot?** HikariCP.

**Q22: `queryForList()` returns?** List of Maps (each map = one row, keys = column names).

**Q23: `queryForMap()` returns?** Map for a single row (key = column name, value = column value).

**Q24: What is `EmptyResultDataAccessException`?** Thrown by `queryForObject` when query returns no rows.

**Q25: How to test JdbcTemplate?** Use `@JdbcTest` (auto-configures embedded H2 + JdbcTemplate).

**Q26: `@JdbcTest` vs `@DataJpaTest`?** @JdbcTest for Spring JDBC. @DataJpaTest for JPA/Hibernate.

**Q27: How to use IN clause?** NamedParameterJdbcTemplate: `WHERE id IN (:ids)` + `MapSqlParameterSource("ids", list)`.

**Q28: What is `DataSourceUtils`?** Utility for getting/releasing connections with Spring transaction awareness.

**Q29: What is `JdbcDaoSupport`?** Base class providing JdbcTemplate. Deprecated — use constructor injection instead.

**Q30: Can JdbcTemplate handle LOBs?** Yes, using `LobHandler` and `LobCreator`.

**Q31: What is `RowCallbackHandler`?** Like RowMapper but void return — used for streaming/side effects.

**Q32: Thread safety of JdbcTemplate?** Thread-safe! Can be shared across multiple threads/DAOs.

**Q33: What is `JdbcTemplate.execute()`?** Executes arbitrary SQL (DDL, etc.).

**Q34: Difference between `batchUpdate` return values?** Array of row counts per statement.

**Q35: How to paginate with JdbcTemplate?** Add `LIMIT ? OFFSET ?` to SQL manually.

**Q36: What is `SqlRowSet`?** Disconnected ResultSet wrapper. Use `queryForRowSet()`.

**Q37: Multiple DataSources?** Configure two DataSource beans, use @Qualifier to inject specific one.

**Q38: What is `@Sql` annotation?** Runs SQL scripts before/after test methods.

**Q39: `PreparedStatementCreator` purpose?** Creates PreparedStatement with custom configuration.

**Q40: `PreparedStatementSetter` purpose?** Sets parameters on an existing PreparedStatement.

**Q41: Connection pooling with Spring JDBC?** HikariCP auto-configured. Configure via `spring.datasource.hikari.*`.

**Q42: Schema initialization?** `spring.sql.init.mode=always` + `schema.sql` and `data.sql` in resources.

**Q43: What is `@Transactional(propagation)`?** REQUIRED (default), REQUIRES_NEW, NESTED, SUPPORTS, etc.

**Q44: What is `@Transactional(isolation)`?** READ_UNCOMMITTED, READ_COMMITTED, REPEATABLE_READ, SERIALIZABLE.

**Q45: What is `@Transactional(rollbackFor)`?** Specify which exceptions trigger rollback (default: RuntimeException).

**Q46: What is `TransactionTemplate`?** Programmatic transaction management (alternative to @Transactional).

**Q47: What is read-only transaction?** `@Transactional(readOnly = true)` — optimization hint for database.

**Q48: SimpleJdbcInsert vs JdbcTemplate.update?** SimpleJdbcInsert: no SQL needed, auto-discovers columns. update: you write SQL.

**Q49: How to log SQL in Spring JDBC?** `logging.level.org.springframework.jdbc.core=DEBUG`.

**Q50: JdbcClient fluent API example?** `jdbcClient.sql("SELECT...").param("id",1).query(User.class).optional()`.

---

## 📚 References

- [Spring JDBC Documentation](https://docs.spring.io/spring-framework/reference/data-access/jdbc.html)
- [Spring Boot Data JDBC](https://docs.spring.io/spring-boot/docs/current/reference/html/data.html)
- [Baeldung Spring JDBC](https://www.baeldung.com/spring-jdbc-jdbctemplate)

---

> **Previous Topic:** [← 13 - Spring REST API](../13-spring-rest-api-springboot/README.md)  
> **Next Topic:** [15 - Spring Data JPA →](../15-spring-data-jpa/README.md)
