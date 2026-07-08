# 🗄️ JDBC (Java Database Connectivity) — Complete In-Depth Guide

> **"JDBC is the foundation of all Java database access. Even if you use Spring Data JPA or Hibernate, understanding JDBC is essential — they all build on top of it."**

---

## 📑 Table of Contents

1. [Introduction & Overview](#1-introduction--overview)
2. [JDBC Architecture](#2-jdbc-architecture)
3. [JDBC Drivers](#3-jdbc-drivers)
4. [Setting Up JDBC](#4-setting-up-jdbc)
5. [Connection Management](#5-connection-management)
6. [Statements](#6-statements)
7. [ResultSet](#7-resultset)
8. [CRUD Operations](#8-crud-operations)
9. [PreparedStatement (Preventing SQL Injection)](#9-preparedstatement-preventing-sql-injection)
10. [CallableStatement (Stored Procedures)](#10-callablestatement-stored-procedures)
11. [Batch Processing](#11-batch-processing)
12. [Transaction Management](#12-transaction-management)
13. [Connection Pooling](#13-connection-pooling)
14. [RowSet & DataSource](#14-rowset--datasource)
15. [JDBC with Spring Boot](#15-jdbc-with-spring-boot)
16. [Best Practices](#16-best-practices)
17. [Common Mistakes & Pitfalls](#17-common-mistakes--pitfalls)
18. [Interview Questions & Answers (50+)](#18-interview-questions--answers-50)

---

## 1. Introduction & Overview

### What is JDBC?

JDBC (Java Database Connectivity) is a **Java API** that provides a standard interface for connecting Java applications to relational databases. It is part of the Java SE platform and defined in the `java.sql` and `javax.sql` packages.

### Why Learn JDBC?

- **Foundation** — Every Java ORM (Hibernate, JPA, MyBatis) uses JDBC under the hood
- **Control** — Direct SQL gives you full control over queries and performance
- **Debugging** — Understanding JDBC helps debug ORM issues
- **Interviews** — Commonly asked in Java backend interviews
- **Spring JDBC** — Spring's JdbcTemplate simplifies JDBC but still requires understanding
- **Legacy systems** — Many enterprise apps still use raw JDBC

### JDBC vs ORM

| Feature | JDBC | ORM (Hibernate/JPA) |
|---------|------|---------------------|
| Abstraction | Low-level | High-level |
| SQL | Write manually | Generated automatically |
| Learning curve | Lower | Higher |
| Performance | Potentially faster (optimized SQL) | May generate suboptimal queries |
| Boilerplate | Lots of repetitive code | Minimal |
| Object mapping | Manual | Automatic |
| Caching | None built-in | First/second level cache |
| Portability | Database-specific SQL | Database-agnostic (mostly) |

---

## 2. JDBC Architecture

### Two-Tier Architecture

```
┌──────────────────┐         ┌──────────────────┐
│  Java Application│         │    Database       │
│                  │  JDBC   │                   │
│  ┌────────────┐  │◄───────►│  MySQL / Oracle   │
│  │ JDBC API   │  │  Driver │  PostgreSQL / H2  │
│  └────────────┘  │         │                   │
└──────────────────┘         └──────────────────┘
```

### Three-Tier Architecture (Web Applications)

```
┌────────┐     ┌──────────────┐     ┌────────────┐     ┌──────────┐
│ Client │────►│  Web Server  │────►│ App Server │────►│ Database │
│(Browser)│    │ (Controller) │     │ (Service)  │ JDBC│          │
└────────┘     └──────────────┘     └────────────┘     └──────────┘
```

### JDBC API Components

```
┌─────────────────────────────────────────────────┐
│                  JDBC API                        │
│                                                  │
│  ┌──────────────┐  ┌──────────────────────────┐ │
│  │ java.sql     │  │ javax.sql                │ │
│  │              │  │                           │ │
│  │ DriverManager│  │ DataSource               │ │
│  │ Connection   │  │ ConnectionPoolDataSource  │ │
│  │ Statement    │  │ RowSet                    │ │
│  │ PreparedStmt │  │ XADataSource (distributed)│ │
│  │ CallableStmt │  │                           │ │
│  │ ResultSet    │  │                           │ │
│  │ SQLException │  │                           │ │
│  └──────────────┘  └──────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

### Key Interfaces

| Interface | Purpose |
|-----------|---------|
| `DriverManager` | Manages database drivers, creates connections |
| `Connection` | Represents a database session |
| `Statement` | Executes static SQL queries |
| `PreparedStatement` | Executes parameterized SQL (prevents SQL injection) |
| `CallableStatement` | Executes stored procedures |
| `ResultSet` | Holds the results of a query |
| `DataSource` | Factory for connections (preferred over DriverManager) |
| `SQLException` | Exception for database errors |

---

## 3. JDBC Drivers

### Four Types of JDBC Drivers

| Type | Name | Description | Performance |
|------|------|-------------|-------------|
| Type 1 | JDBC-ODBC Bridge | Uses ODBC driver (deprecated in Java 8) | Slow |
| Type 2 | Native-API | Uses native DB client libraries | Medium |
| Type 3 | Network Protocol | Middleware translates to DB protocol | Medium |
| Type 4 | Thin/Pure Java | Direct Java-to-DB protocol | **Fast** ✅ |

**Type 4 is the standard** — used by MySQL Connector/J, PostgreSQL JDBC, Oracle JDBC, etc.

### Common JDBC Drivers (Maven Dependencies)

```xml
<!-- MySQL -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <version>8.3.0</version>
</dependency>

<!-- PostgreSQL -->
<dependency>
    <groupId>org.postgresql</groupId>
    <artifactId>postgresql</artifactId>
    <version>42.7.3</version>
</dependency>

<!-- H2 (In-Memory — for testing) -->
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>2.2.224</version>
</dependency>

<!-- Oracle -->
<dependency>
    <groupId>com.oracle.database.jdbc</groupId>
    <artifactId>ojdbc11</artifactId>
    <version>23.3.0.23.09</version>
</dependency>

<!-- SQL Server -->
<dependency>
    <groupId>com.microsoft.sqlserver</groupId>
    <artifactId>mssql-jdbc</artifactId>
    <version>12.6.1.jre11</version>
</dependency>
```

### JDBC URL Formats

```java
// MySQL
String url = "jdbc:mysql://localhost:3306/mydb?useSSL=false&serverTimezone=UTC";

// PostgreSQL
String url = "jdbc:postgresql://localhost:5432/mydb";

// H2 (In-Memory)
String url = "jdbc:h2:mem:testdb;DB_CLOSE_DELAY=-1";

// H2 (File-based)
String url = "jdbc:h2:file:./data/mydb";

// Oracle
String url = "jdbc:oracle:thin:@localhost:1521:orcl";

// SQL Server
String url = "jdbc:sqlserver://localhost:1433;databaseName=mydb";

// SQLite
String url = "jdbc:sqlite:mydb.db";
```

---

## 4. Setting Up JDBC

### Step-by-Step JDBC Connection

```java
import java.sql.*;

public class JDBCDemo {
    
    // JDBC connection parameters
    private static final String URL = "jdbc:mysql://localhost:3306/mydb";
    private static final String USER = "root";
    private static final String PASSWORD = "password";
    
    public static void main(String[] args) {
        
        // Step 1: Load the JDBC Driver (optional since JDBC 4.0 / Java 6)
        // The driver auto-registers via META-INF/services/java.sql.Driver
        // Class.forName("com.mysql.cj.jdbc.Driver"); // Not needed anymore!
        
        // Step 2: Establish Connection
        // Step 3: Create Statement
        // Step 4: Execute Query
        // Step 5: Process Results
        // Step 6: Close Resources
        
        try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery("SELECT * FROM users")) {
            
            while (rs.next()) {
                int id = rs.getInt("id");
                String name = rs.getString("name");
                String email = rs.getString("email");
                System.out.printf("ID: %d, Name: %s, Email: %s%n", id, name, email);
            }
            
        } catch (SQLException e) {
            System.err.println("SQL State: " + e.getSQLState());
            System.err.println("Error Code: " + e.getErrorCode());
            System.err.println("Message: " + e.getMessage());
            e.printStackTrace();
        }
        // Connection, Statement, and ResultSet are auto-closed by try-with-resources
    }
}
```

---

## 5. Connection Management

### Connection Properties

```java
// Method 1: URL with parameters
Connection conn = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/mydb?useSSL=false&autoReconnect=true",
    "root", "password"
);

// Method 2: Properties object
Properties props = new Properties();
props.setProperty("user", "root");
props.setProperty("password", "password");
props.setProperty("useSSL", "false");
props.setProperty("autoReconnect", "true");
props.setProperty("characterEncoding", "UTF-8");
props.setProperty("connectionTimeout", "5000");

Connection conn = DriverManager.getConnection(
    "jdbc:mysql://localhost:3306/mydb", props
);

// Connection metadata
DatabaseMetaData metaData = conn.getMetaData();
System.out.println("Database: " + metaData.getDatabaseProductName());
System.out.println("Version: " + metaData.getDatabaseProductVersion());
System.out.println("Driver: " + metaData.getDriverName());
System.out.println("URL: " + metaData.getURL());
System.out.println("Max Connections: " + metaData.getMaxConnections());
System.out.println("Supports Transactions: " + metaData.supportsTransactions());
```

### Connection Lifecycle

```java
// ❌ BAD: Connection leak (never closed!)
Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
Statement stmt = conn.createStatement();
ResultSet rs = stmt.executeQuery("SELECT * FROM users");
// If exception occurs here, resources are NEVER closed!

// ✅ GOOD: try-with-resources (auto-close guaranteed)
try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
     PreparedStatement pstmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?")) {
    
    pstmt.setInt(1, userId);
    
    try (ResultSet rs = pstmt.executeQuery()) {
        while (rs.next()) {
            // Process results
        }
    }
} // conn, pstmt, rs all closed automatically (in reverse order)
```

---

## 6. Statements

### Three Types of Statements

```java
// 1. Statement — for static SQL (NO parameters)
// ⚠️ VULNERABLE to SQL injection if you concatenate user input!
try (Statement stmt = conn.createStatement()) {
    ResultSet rs = stmt.executeQuery("SELECT * FROM users");
    int rows = stmt.executeUpdate("INSERT INTO users (name) VALUES ('Dilip')");
    boolean hasResult = stmt.execute("SELECT * FROM users"); // Returns true if ResultSet
}

// 2. PreparedStatement — for parameterized SQL (PREVENTS SQL injection) ✅
// Pre-compiled, faster for repeated execution
try (PreparedStatement pstmt = conn.prepareStatement(
        "SELECT * FROM users WHERE name = ? AND age > ?")) {
    pstmt.setString(1, "Dilip");  // Set first parameter
    pstmt.setInt(2, 18);          // Set second parameter
    ResultSet rs = pstmt.executeQuery();
}

// 3. CallableStatement — for stored procedures
try (CallableStatement cstmt = conn.prepareCall("{call get_user_by_id(?)}")) {
    cstmt.setInt(1, userId);
    ResultSet rs = cstmt.executeQuery();
}
```

### Statement Methods

| Method | Purpose | Returns |
|--------|---------|---------|
| `executeQuery(sql)` | SELECT statements | `ResultSet` |
| `executeUpdate(sql)` | INSERT, UPDATE, DELETE, DDL | `int` (rows affected) |
| `execute(sql)` | Any SQL statement | `boolean` (true if ResultSet) |
| `executeBatch()` | Execute batch of statements | `int[]` (rows per statement) |

---

## 7. ResultSet

### Navigating ResultSet

```java
try (PreparedStatement pstmt = conn.prepareStatement("SELECT * FROM users");
     ResultSet rs = pstmt.executeQuery()) {
    
    // Forward-only iteration (default)
    while (rs.next()) {
        // Get by column index (1-based)
        int id = rs.getInt(1);
        String name = rs.getString(2);
        
        // Get by column name (preferred — more readable)
        int id2 = rs.getInt("id");
        String name2 = rs.getString("name");
        String email = rs.getString("email");
        double salary = rs.getDouble("salary");
        Date joinDate = rs.getDate("join_date");
        Timestamp created = rs.getTimestamp("created_at");
        boolean active = rs.getBoolean("active");
        byte[] photo = rs.getBytes("photo");
        
        // Check for NULL values
        rs.getInt("nullable_column");
        if (rs.wasNull()) {
            System.out.println("Value was NULL");
        }
    }
}
```

### Scrollable ResultSet

```java
// Create scrollable, read-only ResultSet
try (Statement stmt = conn.createStatement(
        ResultSet.TYPE_SCROLL_INSENSITIVE,  // Scrollable
        ResultSet.CONCUR_READ_ONLY)) {      // Read-only
    
    ResultSet rs = stmt.executeQuery("SELECT * FROM users");
    
    rs.first();          // Move to first row
    rs.last();           // Move to last row
    rs.absolute(5);      // Move to row 5
    rs.relative(-2);     // Move 2 rows backward
    rs.previous();       // Move to previous row
    rs.beforeFirst();    // Move before first row
    rs.afterLast();      // Move after last row
    
    int rowCount = rs.getRow(); // Current row number
    boolean isFirst = rs.isFirst();
    boolean isLast = rs.isLast();
}

// ResultSet Types:
// TYPE_FORWARD_ONLY     — Default, can only call next()
// TYPE_SCROLL_INSENSITIVE — Scrollable, doesn't see DB changes
// TYPE_SCROLL_SENSITIVE   — Scrollable, sees DB changes

// Concurrency:
// CONCUR_READ_ONLY — Default, read-only
// CONCUR_UPDATABLE — Can update through ResultSet
```

### Updatable ResultSet

```java
try (Statement stmt = conn.createStatement(
        ResultSet.TYPE_SCROLL_INSENSITIVE,
        ResultSet.CONCUR_UPDATABLE)) {
    
    ResultSet rs = stmt.executeQuery("SELECT id, name, salary FROM employees");
    
    // Update existing row
    while (rs.next()) {
        if (rs.getString("name").equals("Dilip")) {
            rs.updateDouble("salary", 90000.00);
            rs.updateRow(); // Commit the update
        }
    }
    
    // Insert new row
    rs.moveToInsertRow();
    rs.updateString("name", "New Employee");
    rs.updateDouble("salary", 50000.00);
    rs.insertRow();
    rs.moveToCurrentRow();
    
    // Delete row
    rs.absolute(3);
    rs.deleteRow();
}
```

---

## 8. CRUD Operations

### Complete CRUD Example

```java
public class UserDAO {
    
    private static final String URL = "jdbc:mysql://localhost:3306/mydb";
    private static final String USER = "root";
    private static final String PASSWORD = "password";
    
    // CREATE
    public int createUser(String name, String email, int age) throws SQLException {
        String sql = "INSERT INTO users (name, email, age) VALUES (?, ?, ?)";
        
        try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
             PreparedStatement pstmt = conn.prepareStatement(sql, 
                 Statement.RETURN_GENERATED_KEYS)) {
            
            pstmt.setString(1, name);
            pstmt.setString(2, email);
            pstmt.setInt(3, age);
            
            int rowsAffected = pstmt.executeUpdate();
            
            // Get auto-generated ID
            if (rowsAffected > 0) {
                try (ResultSet generatedKeys = pstmt.getGeneratedKeys()) {
                    if (generatedKeys.next()) {
                        return generatedKeys.getInt(1);
                    }
                }
            }
            return -1;
        }
    }
    
    // READ (Single)
    public User findById(int id) throws SQLException {
        String sql = "SELECT * FROM users WHERE id = ?";
        
        try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setInt(1, id);
            
            try (ResultSet rs = pstmt.executeQuery()) {
                if (rs.next()) {
                    return mapRowToUser(rs);
                }
                return null;
            }
        }
    }
    
    // READ (All)
    public List<User> findAll() throws SQLException {
        String sql = "SELECT * FROM users ORDER BY id";
        List<User> users = new ArrayList<>();
        
        try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
             Statement stmt = conn.createStatement();
             ResultSet rs = stmt.executeQuery(sql)) {
            
            while (rs.next()) {
                users.add(mapRowToUser(rs));
            }
        }
        return users;
    }
    
    // UPDATE
    public boolean updateUser(int id, String name, String email) throws SQLException {
        String sql = "UPDATE users SET name = ?, email = ? WHERE id = ?";
        
        try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setString(1, name);
            pstmt.setString(2, email);
            pstmt.setInt(3, id);
            
            return pstmt.executeUpdate() > 0;
        }
    }
    
    // DELETE
    public boolean deleteUser(int id) throws SQLException {
        String sql = "DELETE FROM users WHERE id = ?";
        
        try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
             PreparedStatement pstmt = conn.prepareStatement(sql)) {
            
            pstmt.setInt(1, id);
            return pstmt.executeUpdate() > 0;
        }
    }
    
    // Helper: Map ResultSet row to User object
    private User mapRowToUser(ResultSet rs) throws SQLException {
        User user = new User();
        user.setId(rs.getInt("id"));
        user.setName(rs.getString("name"));
        user.setEmail(rs.getString("email"));
        user.setAge(rs.getInt("age"));
        user.setCreatedAt(rs.getTimestamp("created_at").toLocalDateTime());
        return user;
    }
}
```

---

## 9. PreparedStatement (Preventing SQL Injection)

### SQL Injection Attack

```java
// ❌ VULNERABLE: String concatenation with user input
String username = "admin'; DROP TABLE users; --";
String sql = "SELECT * FROM users WHERE username = '" + username + "'";
// Resulting SQL: SELECT * FROM users WHERE username = 'admin'; DROP TABLE users; --'
// This DROPS the entire users table!

// ✅ SAFE: PreparedStatement with parameters
String sql = "SELECT * FROM users WHERE username = ?";
PreparedStatement pstmt = conn.prepareStatement(sql);
pstmt.setString(1, username);
// The driver treats the entire input as a literal string value
// No SQL injection possible!
```

### PreparedStatement Parameter Types

```java
PreparedStatement pstmt = conn.prepareStatement(
    "INSERT INTO products (name, price, quantity, description, created, active, image) " +
    "VALUES (?, ?, ?, ?, ?, ?, ?)"
);

pstmt.setString(1, "Laptop");          // VARCHAR
pstmt.setDouble(2, 999.99);            // DOUBLE / DECIMAL
pstmt.setInt(3, 50);                   // INTEGER
pstmt.setNull(4, Types.VARCHAR);       // NULL value
pstmt.setTimestamp(5, Timestamp.valueOf(LocalDateTime.now())); // TIMESTAMP
pstmt.setBoolean(6, true);             // BOOLEAN
pstmt.setBytes(7, imageBytes);         // BLOB / BINARY

// Date types
pstmt.setDate(1, java.sql.Date.valueOf(LocalDate.now()));
pstmt.setTime(2, java.sql.Time.valueOf(LocalTime.now()));
pstmt.setTimestamp(3, Timestamp.valueOf(LocalDateTime.now()));

// Large objects
pstmt.setClob(1, new StringReader("large text"));  // CLOB (Character LOB)
pstmt.setBlob(2, new FileInputStream("file.pdf")); // BLOB (Binary LOB)

pstmt.executeUpdate();
```

### PreparedStatement vs Statement Performance

```java
// Statement: SQL is parsed and compiled EVERY time
for (int i = 0; i < 1000; i++) {
    stmt.executeUpdate("INSERT INTO logs (msg) VALUES ('Log " + i + "')");
    // 1000 parse + compile + execute cycles
}

// PreparedStatement: SQL is parsed and compiled ONCE, executed many times
PreparedStatement pstmt = conn.prepareStatement("INSERT INTO logs (msg) VALUES (?)");
for (int i = 0; i < 1000; i++) {
    pstmt.setString(1, "Log " + i);
    pstmt.executeUpdate();
    // 1 parse + compile, 1000 execute cycles (MUCH faster)
}
```

---

## 10. CallableStatement (Stored Procedures)

```java
// MySQL stored procedure:
// DELIMITER //
// CREATE PROCEDURE get_user(IN user_id INT, OUT user_name VARCHAR(100))
// BEGIN
//     SELECT name INTO user_name FROM users WHERE id = user_id;
// END //
// DELIMITER ;

// Call stored procedure with IN and OUT parameters
try (CallableStatement cstmt = conn.prepareCall("{CALL get_user(?, ?)}")) {
    
    cstmt.setInt(1, 1);                            // IN parameter
    cstmt.registerOutParameter(2, Types.VARCHAR);   // OUT parameter
    
    cstmt.execute();
    
    String userName = cstmt.getString(2);           // Get OUT parameter
    System.out.println("User name: " + userName);
}

// Stored procedure that returns a ResultSet
try (CallableStatement cstmt = conn.prepareCall("{CALL get_all_users()}")) {
    try (ResultSet rs = cstmt.executeQuery()) {
        while (rs.next()) {
            System.out.println(rs.getString("name"));
        }
    }
}

// Function call
try (CallableStatement cstmt = conn.prepareCall("{? = CALL count_users()}")) {
    cstmt.registerOutParameter(1, Types.INTEGER);
    cstmt.execute();
    int count = cstmt.getInt(1);
}
```

---

## 11. Batch Processing

```java
// Batch INSERT — much faster than individual inserts
public void batchInsert(List<User> users) throws SQLException {
    String sql = "INSERT INTO users (name, email, age) VALUES (?, ?, ?)";
    
    try (Connection conn = DriverManager.getConnection(URL, USER, PASSWORD);
         PreparedStatement pstmt = conn.prepareStatement(sql)) {
        
        conn.setAutoCommit(false); // Start transaction
        
        int batchSize = 1000;
        int count = 0;
        
        for (User user : users) {
            pstmt.setString(1, user.getName());
            pstmt.setString(2, user.getEmail());
            pstmt.setInt(3, user.getAge());
            pstmt.addBatch(); // Add to batch
            
            count++;
            if (count % batchSize == 0) {
                pstmt.executeBatch(); // Execute batch every 1000 records
                pstmt.clearBatch();
            }
        }
        
        pstmt.executeBatch(); // Execute remaining
        conn.commit();        // Commit transaction
        
    } catch (SQLException e) {
        conn.rollback();      // Rollback on error
        throw e;
    }
}

// For MySQL, add rewriteBatchedStatements=true to URL for maximum performance:
// jdbc:mysql://localhost:3306/mydb?rewriteBatchedStatements=true
// This rewrites individual INSERTs into multi-value INSERT (10x+ faster)
```

---

## 12. Transaction Management

### ACID Properties

| Property | Description |
|----------|-------------|
| **Atomicity** | All operations succeed or all fail (all-or-nothing) |
| **Consistency** | Database moves from one valid state to another |
| **Isolation** | Concurrent transactions don't interfere |
| **Durability** | Committed changes survive system failures |

### Transaction Example

```java
public void transferMoney(int fromId, int toId, double amount) throws SQLException {
    Connection conn = null;
    
    try {
        conn = DriverManager.getConnection(URL, USER, PASSWORD);
        conn.setAutoCommit(false); // START TRANSACTION
        
        // Step 1: Debit from sender
        try (PreparedStatement debit = conn.prepareStatement(
                "UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?")) {
            debit.setDouble(1, amount);
            debit.setInt(2, fromId);
            debit.setDouble(3, amount);
            int rows = debit.executeUpdate();
            if (rows == 0) {
                throw new SQLException("Insufficient funds or account not found");
            }
        }
        
        // Step 2: Credit to receiver
        try (PreparedStatement credit = conn.prepareStatement(
                "UPDATE accounts SET balance = balance + ? WHERE id = ?")) {
            credit.setDouble(1, amount);
            credit.setInt(2, toId);
            int rows = credit.executeUpdate();
            if (rows == 0) {
                throw new SQLException("Receiver account not found");
            }
        }
        
        // Step 3: Log the transfer
        try (PreparedStatement log = conn.prepareStatement(
                "INSERT INTO transfers (from_id, to_id, amount, transfer_date) VALUES (?, ?, ?, ?)")) {
            log.setInt(1, fromId);
            log.setInt(2, toId);
            log.setDouble(3, amount);
            log.setTimestamp(4, Timestamp.valueOf(LocalDateTime.now()));
            log.executeUpdate();
        }
        
        conn.commit(); // COMMIT — all three operations succeed
        System.out.println("Transfer successful!");
        
    } catch (SQLException e) {
        if (conn != null) {
            conn.rollback(); // ROLLBACK — undo ALL operations
            System.err.println("Transfer failed, rolled back: " + e.getMessage());
        }
        throw e;
    } finally {
        if (conn != null) {
            conn.setAutoCommit(true); // Restore default
            conn.close();
        }
    }
}
```

### Transaction Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|-------|-----------|-------------------|--------------|-------------|
| READ_UNCOMMITTED | ✅ Yes | ✅ Yes | ✅ Yes | Fastest |
| READ_COMMITTED | ❌ No | ✅ Yes | ✅ Yes | Fast |
| REPEATABLE_READ | ❌ No | ❌ No | ✅ Yes | Medium |
| SERIALIZABLE | ❌ No | ❌ No | ❌ No | Slowest |

```java
conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
conn.setTransactionIsolation(Connection.TRANSACTION_REPEATABLE_READ);
conn.setTransactionIsolation(Connection.TRANSACTION_SERIALIZABLE);
```

### Savepoints

```java
conn.setAutoCommit(false);
Savepoint sp1 = conn.setSavepoint("step1");

try {
    // Step 1
    stmt.executeUpdate("INSERT INTO orders ...");
    
    Savepoint sp2 = conn.setSavepoint("step2");
    
    try {
        // Step 2
        stmt.executeUpdate("INSERT INTO order_items ...");
    } catch (SQLException e) {
        conn.rollback(sp2); // Rollback only step 2
    }
    
    conn.commit();
} catch (SQLException e) {
    conn.rollback(sp1); // Rollback everything
}
```

---

## 13. Connection Pooling

### Why Connection Pooling?

Creating a database connection is **expensive** (TCP handshake, authentication, protocol negotiation). Connection pooling maintains a pool of reusable connections.

```
Without pooling:                    With pooling:
Request → Create Connection         Request → Borrow Connection
          → Execute Query                     → Execute Query
          → Close Connection                  → Return Connection to pool
          (100ms+ per request)               (< 1ms per request)
```

### HikariCP (Fastest Connection Pool — Default in Spring Boot)

```xml
<!-- Maven dependency -->
<dependency>
    <groupId>com.zaxxer</groupId>
    <artifactId>HikariCP</artifactId>
    <version>5.1.0</version>
</dependency>
```

```java
import com.zaxxer.hikari.HikariConfig;
import com.zaxxer.hikari.HikariDataSource;

public class ConnectionPool {
    
    private static HikariDataSource dataSource;
    
    static {
        HikariConfig config = new HikariConfig();
        config.setJdbcUrl("jdbc:mysql://localhost:3306/mydb");
        config.setUsername("root");
        config.setPassword("password");
        
        // Pool configuration
        config.setMaximumPoolSize(10);          // Max connections in pool
        config.setMinimumIdle(5);               // Min idle connections
        config.setConnectionTimeout(30000);     // 30 seconds to get connection
        config.setIdleTimeout(600000);          // 10 minutes idle before closing
        config.setMaxLifetime(1800000);         // 30 minutes max connection lifetime
        config.setLeakDetectionThreshold(60000); // Log if connection not returned in 60s
        
        // Performance
        config.addDataSourceProperty("cachePrepStmts", "true");
        config.addDataSourceProperty("prepStmtCacheSize", "250");
        config.addDataSourceProperty("prepStmtCacheSqlLimit", "2048");
        config.addDataSourceProperty("useServerPrepStmts", "true");
        
        config.setPoolName("MyAppPool");
        
        dataSource = new HikariDataSource(config);
    }
    
    public static Connection getConnection() throws SQLException {
        return dataSource.getConnection(); // Borrow from pool
    }
    
    public static void close() {
        if (dataSource != null && !dataSource.isClosed()) {
            dataSource.close();
        }
    }
}

// Usage:
try (Connection conn = ConnectionPool.getConnection()) {
    // Use connection
} // Connection is RETURNED to pool (not actually closed)
```

### Spring Boot HikariCP Configuration

```properties
# application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb
spring.datasource.username=root
spring.datasource.password=password
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver

# HikariCP settings
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.minimum-idle=5
spring.datasource.hikari.connection-timeout=30000
spring.datasource.hikari.idle-timeout=600000
spring.datasource.hikari.max-lifetime=1800000
spring.datasource.hikari.pool-name=MyAppPool
spring.datasource.hikari.leak-detection-threshold=60000
```

---

## 14. RowSet & DataSource

### DataSource (Preferred over DriverManager)

```java
// DataSource is the preferred way to get connections
// It supports connection pooling, distributed transactions, etc.

// Using MySQL DataSource directly
MysqlDataSource ds = new MysqlDataSource();
ds.setURL("jdbc:mysql://localhost:3306/mydb");
ds.setUser("root");
ds.setPassword("password");

Connection conn = ds.getConnection();

// In Spring Boot, DataSource is auto-configured
@Autowired
private DataSource dataSource;

Connection conn = dataSource.getConnection();
```

### RowSet

```java
// JdbcRowSet — connected, scrollable, updatable wrapper around ResultSet
JdbcRowSet jrs = RowSetProvider.newFactory().createJdbcRowSet();
jrs.setUrl("jdbc:mysql://localhost:3306/mydb");
jrs.setUsername("root");
jrs.setPassword("password");
jrs.setCommand("SELECT * FROM users WHERE age > ?");
jrs.setInt(1, 18);
jrs.execute();

while (jrs.next()) {
    System.out.println(jrs.getString("name"));
}
jrs.close();

// CachedRowSet — disconnected, serializable
CachedRowSet crs = RowSetProvider.newFactory().createCachedRowSet();
crs.setUrl("jdbc:mysql://localhost:3306/mydb");
crs.setUsername("root");
crs.setPassword("password");
crs.setCommand("SELECT * FROM users");
crs.execute(); // Fetches data and disconnects

// Connection is closed, but data is still available!
while (crs.next()) {
    System.out.println(crs.getString("name"));
}
```

---

## 15. JDBC with Spring Boot

### Spring JdbcTemplate

```java
@Repository
public class UserRepository {
    
    private final JdbcTemplate jdbcTemplate;
    
    public UserRepository(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }
    
    // Query single object
    public User findById(Long id) {
        String sql = "SELECT * FROM users WHERE id = ?";
        return jdbcTemplate.queryForObject(sql, new UserRowMapper(), id);
    }
    
    // Query list
    public List<User> findAll() {
        return jdbcTemplate.query("SELECT * FROM users", new UserRowMapper());
    }
    
    // Insert
    public int save(User user) {
        String sql = "INSERT INTO users (name, email, age) VALUES (?, ?, ?)";
        return jdbcTemplate.update(sql, user.getName(), user.getEmail(), user.getAge());
    }
    
    // Update
    public int update(User user) {
        String sql = "UPDATE users SET name = ?, email = ? WHERE id = ?";
        return jdbcTemplate.update(sql, user.getName(), user.getEmail(), user.getId());
    }
    
    // Delete
    public int deleteById(Long id) {
        return jdbcTemplate.update("DELETE FROM users WHERE id = ?", id);
    }
    
    // Count
    public int count() {
        return jdbcTemplate.queryForObject("SELECT COUNT(*) FROM users", Integer.class);
    }
    
    // RowMapper
    private static class UserRowMapper implements RowMapper<User> {
        @Override
        public User mapRow(ResultSet rs, int rowNum) throws SQLException {
            User user = new User();
            user.setId(rs.getLong("id"));
            user.setName(rs.getString("name"));
            user.setEmail(rs.getString("email"));
            user.setAge(rs.getInt("age"));
            return user;
        }
    }
}
```

---

## 16. Best Practices

1. **Always use PreparedStatement** — prevents SQL injection, better performance
2. **Always use try-with-resources** — guarantees resource cleanup
3. **Always use connection pooling** — HikariCP is the standard
4. **Close resources in reverse order** — ResultSet → Statement → Connection
5. **Use DataSource instead of DriverManager** — supports pooling, JNDI
6. **Handle SQLExceptions properly** — log the SQL state, error code, message
7. **Use batch operations** for bulk inserts/updates — orders of magnitude faster
8. **Set appropriate fetch size** — `rs.setFetchSize(100)` for large result sets
9. **Use transactions** for multiple related operations — ensure ACID
10. **Avoid `SELECT *`** — only select columns you need
11. **Use connection timeout** — don't wait forever for a connection
12. **Monitor connection pool** — watch for leaks, pool exhaustion
13. **Use getGeneratedKeys()** for auto-increment IDs
14. **Don't store passwords in code** — use environment variables or secret managers
15. **Use the right isolation level** — balance between consistency and performance

---

## 17. Common Mistakes & Pitfalls

```java
// ❌ MISTAKE 1: SQL Injection vulnerability
String sql = "SELECT * FROM users WHERE name = '" + userInput + "'";
// ✅ FIX: Use PreparedStatement with parameters

// ❌ MISTAKE 2: Resource leak (not closing connections)
Connection conn = DriverManager.getConnection(url, user, pass);
Statement stmt = conn.createStatement();
// If exception here, conn and stmt are never closed!
// ✅ FIX: Use try-with-resources

// ❌ MISTAKE 3: Creating a new connection for every query
for (User user : users) {
    Connection conn = DriverManager.getConnection(url, user, pass); // Expensive!
    // ...
}
// ✅ FIX: Use connection pooling (HikariCP)

// ❌ MISTAKE 4: Not using transactions for related operations
pstmt1.executeUpdate("UPDATE accounts SET balance = balance - 100 WHERE id = 1");
// If crash here, money is deducted but not credited!
pstmt2.executeUpdate("UPDATE accounts SET balance = balance + 100 WHERE id = 2");
// ✅ FIX: Wrap in transaction with setAutoCommit(false) + commit/rollback

// ❌ MISTAKE 5: ResultSet column index starting at 0
rs.getString(0); // WRONG! Columns are 1-indexed
// ✅ FIX: rs.getString(1) or rs.getString("column_name")

// ❌ MISTAKE 6: Not checking wasNull() for primitive types
int value = rs.getInt("nullable_column"); // Returns 0 for NULL!
// ✅ FIX: Check rs.wasNull() after reading

// ❌ MISTAKE 7: Using Statement for repeated queries
for (int i = 0; i < 1000; i++) {
    stmt.executeUpdate("INSERT INTO logs VALUES ('" + msg + "')");
}
// ✅ FIX: Use PreparedStatement + batch processing

// ❌ MISTAKE 8: Not setting connection timeout
// Connection hangs forever if database is unreachable
// ✅ FIX: Set connectionTimeout in pool configuration
```

---

## 18. Interview Questions & Answers (50+)

### Beginner Level

**Q1: What is JDBC?**
**A:** JDBC (Java Database Connectivity) is a Java API for connecting to relational databases. It provides interfaces (Connection, Statement, ResultSet) for executing SQL queries, processing results, and managing transactions. Part of java.sql and javax.sql packages.

**Q2: What are the steps to connect to a database using JDBC?**
**A:** 1) Load driver (auto since JDBC 4.0), 2) Get Connection via DriverManager or DataSource, 3) Create Statement/PreparedStatement, 4) Execute query, 5) Process ResultSet, 6) Close resources.

**Q3: What is the difference between Statement and PreparedStatement?**
**A:** Statement executes static SQL (compiled every time, vulnerable to SQL injection). PreparedStatement executes parameterized SQL (pre-compiled once, reused multiple times, prevents SQL injection). Always use PreparedStatement.

**Q4: What is SQL injection and how does PreparedStatement prevent it?**
**A:** SQL injection is an attack where malicious SQL is inserted via user input. PreparedStatement prevents it by separating SQL structure from data — parameters are always treated as literal values, never as SQL code.

**Q5: What is a ResultSet?**
**A:** ResultSet holds the results of a SQL query. It provides a cursor to iterate over rows (initially before the first row). Use next() to move forward, and getXxx() methods to retrieve column values.

**Q6: What is the difference between executeQuery(), executeUpdate(), and execute()?**
**A:** executeQuery() for SELECT (returns ResultSet). executeUpdate() for INSERT/UPDATE/DELETE/DDL (returns row count). execute() for any SQL (returns boolean — true if ResultSet).

**Q7: What is connection pooling?**
**A:** Maintaining a pool of pre-created database connections that are reused. Creating connections is expensive (~100ms). Pool connections are borrowed and returned, not created and destroyed. HikariCP is the fastest Java pool.

**Q8: What is auto-commit in JDBC?**
**A:** By default, JDBC auto-commits every SQL statement (each statement is its own transaction). Set `conn.setAutoCommit(false)` for manual transaction management, then explicitly call `conn.commit()` or `conn.rollback()`.

---

### Intermediate Level

**Q9: What are the four types of JDBC drivers?**
**A:** Type 1: JDBC-ODBC Bridge (deprecated). Type 2: Native-API (uses DB client libs). Type 3: Network Protocol (middleware). Type 4: Thin/Pure Java (direct protocol, fastest, most common).

**Q10: What is the difference between DriverManager and DataSource?**
**A:** DriverManager is a simple factory for connections (no pooling). DataSource is a factory that can provide connection pooling, distributed transactions, and JNDI lookup. DataSource is always preferred in production.

**Q11: Explain transaction isolation levels.**
**A:** READ_UNCOMMITTED (dirty reads possible), READ_COMMITTED (no dirty reads), REPEATABLE_READ (no non-repeatable reads), SERIALIZABLE (no phantom reads, slowest). Higher isolation = more consistent but slower.

**Q12: What is batch processing in JDBC?**
**A:** Grouping multiple SQL statements into a batch and executing them at once. Much faster than individual execution because it reduces network round-trips. Use `addBatch()` and `executeBatch()`.

**Q13: What is a Savepoint?**
**A:** A point within a transaction that you can roll back to without rolling back the entire transaction. Created with `conn.setSavepoint()`, rolled back with `conn.rollback(savepoint)`.

**Q14: How do you get auto-generated keys?**
**A:** Pass `Statement.RETURN_GENERATED_KEYS` to `prepareStatement()`, then call `pstmt.getGeneratedKeys()` after executeUpdate() to get a ResultSet containing the generated keys.

**Q15: What is the difference between scrollable and non-scrollable ResultSet?**
**A:** Non-scrollable (TYPE_FORWARD_ONLY, default): can only move forward with next(). Scrollable (TYPE_SCROLL_INSENSITIVE/SENSITIVE): can move forward, backward, to absolute/relative positions.

---

### Advanced Level

**Q16: How does HikariCP achieve high performance?**
**A:** Bytecode-level optimizations, lock-free collections (ConcurrentBag), minimal overhead proxy (using Javassist), optimal defaults, connection state tracking without querying the database.

**Q17: What is a connection leak and how do you detect it?**
**A:** A connection leak occurs when a connection is borrowed from the pool but never returned (not closed). HikariCP detects leaks via `leakDetectionThreshold` — logs a warning with the stack trace of where the connection was obtained.

**Q18: Explain the N+1 query problem.**
**A:** When you query N items and then make N additional queries to fetch related data. Example: fetch 100 orders, then 100 separate queries for customer details. Solution: JOIN queries, batch fetching, or JPA fetch strategies.

**Q19: What is database metadata in JDBC?**
**A:** `DatabaseMetaData` (from `conn.getMetaData()`) provides information about the database: product name, version, supported features, tables, columns, primary keys, foreign keys, indexes.

**Q20: What is RowSet and how does it differ from ResultSet?**
**A:** RowSet extends ResultSet. JdbcRowSet is a connected wrapper with JavaBeans support. CachedRowSet is disconnected (works offline after fetching). WebRowSet can be serialized to XML. RowSet supports event listeners.

---

### Rapid-Fire (Q21–Q50)

**Q21: What package contains JDBC core classes?** `java.sql` (core) and `javax.sql` (extensions like DataSource, RowSet).

**Q22: Is JDBC driver loading mandatory since Java 6?** No, JDBC 4.0+ auto-loads drivers via ServiceLoader (META-INF/services).

**Q23: What is the default ResultSet type?** TYPE_FORWARD_ONLY with CONCUR_READ_ONLY.

**Q24: What does rs.wasNull() return?** True if the last column read was SQL NULL.

**Q25: Are ResultSet column indexes 0-based or 1-based?** 1-based.

**Q26: What is setFetchSize()?** Hints to the driver how many rows to fetch from the database at a time. Reduces memory for large result sets.

**Q27: What is CLOB and BLOB?** CLOB: Character Large Object (text). BLOB: Binary Large Object (images, files).

**Q28: What happens if you don't close a Connection?** Connection leak — the connection is never returned to the pool, eventually exhausting the pool and causing application failure.

**Q29: What is the CallableStatement used for?** Calling stored procedures and functions in the database.

**Q30: What is the maximum number of connections in HikariCP default?** 10 (maximumPoolSize default).

**Q31: How do you handle multiple result sets?** Use `execute()`, then `getResultSet()` and `getMoreResults()` to iterate through multiple ResultSets.

**Q32: What is JNDI DataSource?** A DataSource registered in a JNDI directory (e.g., application server). Looked up by name. Common in Java EE/Jakarta EE applications.

**Q33: What is DatabaseMetaData.getTables()?** Returns a ResultSet listing all tables in the database matching the specified criteria (catalog, schema, name pattern, types).

**Q34: How do you enable MySQL batch rewriting?** Add `rewriteBatchedStatements=true` to the JDBC URL. Converts individual INSERTs to multi-value INSERT.

**Q35: What is connection validation?** Checking if a borrowed connection is still valid. HikariCP uses `Connection.isValid()` or a test query.

**Q36: What is the difference between `conn.commit()` and `conn.setAutoCommit(true)`?** commit() commits the current transaction. setAutoCommit(true) commits AND switches back to auto-commit mode.

**Q37: What is JDBC's ParameterMetaData?** Provides information about the parameters in a PreparedStatement (type, precision, scale, nullability).

**Q38: What is the difference between TYPE_SCROLL_INSENSITIVE and TYPE_SCROLL_SENSITIVE?** INSENSITIVE doesn't see changes made by other transactions. SENSITIVE does see them (reflects current DB state).

**Q39: How do you handle large result sets efficiently?** Use setFetchSize(), stream results, use pagination (LIMIT/OFFSET), or use server-side cursors.

**Q40: What is XA transaction?** A distributed transaction that spans multiple databases/resources. Managed by a transaction manager using two-phase commit.

**Q41: What is `conn.createStatement(int, int)` parameters?** ResultSet type (FORWARD_ONLY, SCROLL_INSENSITIVE, SCROLL_SENSITIVE) and concurrency (READ_ONLY, UPDATABLE).

**Q42: What is connection timeout vs socket timeout?** Connection timeout: max wait to establish TCP connection. Socket timeout: max wait for a response after connection is established.

**Q43: What does pstmt.clearParameters() do?** Clears all parameter values set on a PreparedStatement so it can be reused with new values.

**Q44: What is the default transaction isolation level in MySQL?** REPEATABLE_READ. In PostgreSQL: READ_COMMITTED. In Oracle: READ_COMMITTED.

**Q45: How does connection pool manage idle connections?** Idle connections are periodically validated and closed if idle beyond `idleTimeout`. Minimum idle connections are maintained.

**Q46: What is batch update return values?** `executeBatch()` returns an int array where each element is the row count for that statement, or SUCCESS_NO_INFO, or EXECUTE_FAILED.

**Q47: How do you set query timeout?** `stmt.setQueryTimeout(30)` — query fails if not completed within 30 seconds.

**Q48: What is ResultSetMetaData?** Provides information about the columns in a ResultSet: column count, names, types, table name, precision.

**Q49: Can PreparedStatement be used for DDL (CREATE TABLE)?** Yes, but it provides no benefit since DDL doesn't have parameters. Use Statement for DDL.

**Q50: What is the connection pool leak detection threshold in HikariCP?** `leakDetectionThreshold` — if a connection is not returned within this time (ms), HikariCP logs a warning with the allocation stack trace. Default is 0 (disabled).

---

## 📚 References & Further Reading

- [JDBC Official Tutorial (Oracle)](https://docs.oracle.com/javase/tutorial/jdbc/)
- [HikariCP GitHub](https://github.com/brettwooldridge/HikariCP)
- [MySQL Connector/J Documentation](https://dev.mysql.com/doc/connector-j/en/)
- [Spring JdbcTemplate Documentation](https://docs.spring.io/spring-framework/reference/data-access/jdbc.html)
- [Baeldung JDBC Tutorials](https://www.baeldung.com/java-jdbc)
- [JDBC API Specification](https://download.oracle.com/otn-pub/jcp/jdbc-4_3-mrel3-spec/jdbc4.3-fr-spec.pdf)

---

> **Previous Topic:** [← 06 - DSA](../06-dsa/README.md)  
> **Next Topic:** [08 - Servlets and JSP →](../08-servlets-and-jsp/README.md)
