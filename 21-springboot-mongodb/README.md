# 🍃 Spring Boot + MongoDB — Complete In-Depth Guide

> **"MongoDB is the leading NoSQL document database. Combined with Spring Data MongoDB, you get the same productive repository pattern as JPA but for flexible, schema-less data."**

---

## 📑 Table of Contents

1. [Introduction to MongoDB](#1-introduction-to-mongodb)
2. [SQL vs NoSQL](#2-sql-vs-nosql)
3. [MongoDB Concepts](#3-mongodb-concepts)
4. [Setup & Configuration](#4-setup--configuration)
5. [Document Model & Annotations](#5-document-model--annotations)
6. [MongoRepository (CRUD)](#6-mongorepository-crud)
7. [Query Methods](#7-query-methods)
8. [Custom Queries (@Query)](#8-custom-queries-query)
9. [MongoTemplate (Advanced)](#9-mongotemplate-advanced)
10. [Aggregation Pipeline](#10-aggregation-pipeline)
11. [Indexing](#11-indexing)
12. [Embedded vs Referenced Documents](#12-embedded-vs-referenced-documents)
13. [Pagination & Sorting](#13-pagination--sorting)
14. [Transactions](#14-transactions)
15. [GridFS (File Storage)](#15-gridfs-file-storage)
16. [Change Streams](#16-change-streams)
17. [Best Practices](#17-best-practices)
18. [Interview Questions & Answers (50+)](#18-interview-questions--answers-50)

---

## 1. Introduction to MongoDB

```
MongoDB is a DOCUMENT database:
  - Stores data as JSON-like documents (BSON)
  - Schema-flexible (no rigid table structure)
  - Horizontally scalable (sharding)
  - High performance for read-heavy workloads

SQL Database:              MongoDB:
Database                   Database
Table                      Collection
Row                        Document (JSON object)
Column                     Field
JOIN                       Embedded document / $lookup
PRIMARY KEY                _id (auto-generated ObjectId)
```

---

## 2. SQL vs NoSQL

| Feature | SQL (MySQL, PostgreSQL) | NoSQL (MongoDB) |
|---------|------------------------|-----------------|
| Schema | Fixed (ALTER TABLE) | Flexible (add fields anytime) |
| Data model | Tables + Rows | Documents (JSON) |
| Joins | Built-in (JOIN) | Embedded docs or $lookup |
| Transactions | Full ACID | ACID (since v4.0) |
| Scaling | Vertical (bigger server) | Horizontal (sharding) |
| Query | SQL | MongoDB Query Language |
| Best for | Complex relationships | Flexible, nested data |
| Examples | Banking, ERP | Content mgmt, IoT, catalogs |

---

## 3. MongoDB Concepts

```json
// A MongoDB DOCUMENT (like a row, but flexible):
{
    "_id": ObjectId("65a1b2c3d4e5f6a7b8c9d0e1"),
    "name": "Dilip",
    "email": "dilip@mail.com",
    "age": 25,
    "address": {
        "city": "Mumbai",
        "state": "Maharashtra",
        "zip": "400001"
    },
    "skills": ["Java", "Spring Boot", "MongoDB"],
    "orders": [
        { "product": "Laptop", "amount": 75000 },
        { "product": "Mouse", "amount": 500 }
    ],
    "createdAt": ISODate("2024-01-15T10:30:00Z")
}

// Notice:
// - Nested object (address) — no separate table needed!
// - Array (skills, orders) — no junction table needed!
// - Each document can have DIFFERENT fields
```

---

## 4. Setup & Configuration

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
</dependency>
```

```properties
# application.properties
spring.data.mongodb.uri=mongodb://localhost:27017/mydb
# Or separate properties:
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=mydb
spring.data.mongodb.username=admin
spring.data.mongodb.password=password
spring.data.mongodb.authentication-database=admin

# Docker run:
# docker run -d -p 27017:27017 --name mongo mongo:latest
```

---

## 5. Document Model & Annotations

```java
@Document(collection = "users")  // Maps to "users" collection
@Data @NoArgsConstructor @AllArgsConstructor
public class User {
    
    @Id                                  // Maps to "_id" in MongoDB
    private String id;                   // String → ObjectId auto-generated
    
    @Field("user_name")                  // Custom field name in MongoDB
    @Indexed                             // Create index on this field
    private String name;
    
    @Indexed(unique = true)              // Unique index
    private String email;
    
    private int age;
    
    @Transient                           // NOT saved to MongoDB
    private String tempCalculation;
    
    private Address address;             // Embedded document (stored inside user)
    
    private List<String> skills;         // Array of strings
    
    private List<Order> orders;          // Array of embedded documents
    
    @CreatedDate                         // Auto-set on creation
    private LocalDateTime createdAt;
    
    @LastModifiedDate                    // Auto-set on update
    private LocalDateTime updatedAt;
    
    @DBRef                              // Reference to another collection
    private Department department;       // Stores reference, not embedded
}

// Embedded document (no @Document — lives inside User)
@Data @NoArgsConstructor @AllArgsConstructor
public class Address {
    private String street;
    private String city;
    private String state;
    private String zip;
}
```

---

## 6. MongoRepository (CRUD)

```java
public interface UserRepository extends MongoRepository<User, String> {
    // INHERITED (free!):
    // save(User) → INSERT or UPDATE
    // findById(String) → Optional<User>
    // findAll() → List<User>
    // findAll(Pageable) → Page<User>
    // deleteById(String)
    // count()
    // existsById(String)
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
        user.setSkills(dto.getSkills());
        user.setAddress(dto.getAddress());
        return userRepository.save(user);  // _id auto-generated
    }
    
    public User findById(String id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new ResourceNotFoundException("User not found: " + id));
    }
    
    public User update(String id, UpdateUserDTO dto) {
        User user = findById(id);
        user.setName(dto.getName());
        user.setEmail(dto.getEmail());
        return userRepository.save(user);  // save() = upsert (insert or update)
    }
    
    public void delete(String id) {
        userRepository.deleteById(id);
    }
}
```

---

## 7. Query Methods

```java
public interface UserRepository extends MongoRepository<User, String> {
    
    // Same method-name derivation as JPA!
    List<User> findByName(String name);
    Optional<User> findByEmail(String email);
    List<User> findByAgeGreaterThan(int age);
    List<User> findByAgeBetween(int min, int max);
    List<User> findByNameContainingIgnoreCase(String keyword);
    
    // Nested field query (dot notation)
    List<User> findByAddressCity(String city);
    // → {"address.city": "Mumbai"}
    
    List<User> findByAddressState(String state);
    
    // Array queries
    List<User> findBySkillsContaining(String skill);
    // → {"skills": "Java"}
    
    // Boolean
    List<User> findByActiveTrue();
    
    // Sorting & Limiting
    List<User> findTop5ByOrderByCreatedAtDesc();
    List<User> findByAgeGreaterThanOrderByNameAsc(int age);
    
    // Count & Exists
    long countByActive(boolean active);
    boolean existsByEmail(String email);
    
    // Delete
    void deleteByEmail(String email);
}
```

---

## 8. Custom Queries (@Query)

```java
public interface UserRepository extends MongoRepository<User, String> {
    
    // MongoDB JSON query syntax
    @Query("{ 'name': ?0 }")
    List<User> findByNameCustom(String name);
    
    // Regex search
    @Query("{ 'name': { $regex: ?0, $options: 'i' } }")
    List<User> searchByName(String keyword);
    
    // Nested field
    @Query("{ 'address.city': ?0 }")
    List<User> findByCity(String city);
    
    // Multiple conditions
    @Query("{ 'age': { $gte: ?0, $lte: ?1 }, 'active': true }")
    List<User> findActiveByAgeRange(int min, int max);
    
    // Array contains
    @Query("{ 'skills': { $all: ?0 } }")
    List<User> findByAllSkills(List<String> skills);
    
    // Projection (return only specific fields)
    @Query(value = "{ 'active': true }", fields = "{ 'name': 1, 'email': 1 }")
    List<User> findActiveUsersNameAndEmail();
    
    // With Sort
    @Query(value = "{ 'age': { $gte: ?0 } }", sort = "{ 'name': 1 }")
    List<User> findByMinAgeSorted(int minAge);
    
    // Update
    @Query("{ '_id': ?0 }")
    @org.springframework.data.mongodb.repository.Update("{ $set: { 'name': ?1 } }")
    long updateName(String id, String newName);
    
    // Delete
    @Query(value = "{ 'active': false }", delete = true)
    List<User> deleteInactiveUsers();
}
```

---

## 9. MongoTemplate (Advanced)

```java
@Repository
@RequiredArgsConstructor
public class UserCustomRepository {
    
    private final MongoTemplate mongoTemplate;
    
    // Dynamic query building
    public List<User> search(String name, String city, Integer minAge, String skill) {
        Query query = new Query();
        
        if (name != null) {
            query.addCriteria(Criteria.where("name").regex(name, "i"));
        }
        if (city != null) {
            query.addCriteria(Criteria.where("address.city").is(city));
        }
        if (minAge != null) {
            query.addCriteria(Criteria.where("age").gte(minAge));
        }
        if (skill != null) {
            query.addCriteria(Criteria.where("skills").in(skill));
        }
        
        query.with(Sort.by(Sort.Direction.ASC, "name"));
        
        return mongoTemplate.find(query, User.class);
    }
    
    // Update specific fields (without loading entire document)
    public long updateEmail(String id, String newEmail) {
        Query query = Query.query(Criteria.where("_id").is(id));
        Update update = new Update()
            .set("email", newEmail)
            .set("updatedAt", LocalDateTime.now());
        
        UpdateResult result = mongoTemplate.updateFirst(query, update, User.class);
        return result.getModifiedCount();
    }
    
    // Add to array
    public void addSkill(String userId, String skill) {
        Query query = Query.query(Criteria.where("_id").is(userId));
        Update update = new Update().addToSet("skills", skill);  // $addToSet prevents duplicates
        mongoTemplate.updateFirst(query, update, User.class);
    }
    
    // Remove from array
    public void removeSkill(String userId, String skill) {
        Query query = Query.query(Criteria.where("_id").is(userId));
        Update update = new Update().pull("skills", skill);
        mongoTemplate.updateFirst(query, update, User.class);
    }
    
    // Increment field
    public void incrementAge(String userId) {
        Query query = Query.query(Criteria.where("_id").is(userId));
        Update update = new Update().inc("age", 1);
        mongoTemplate.updateFirst(query, update, User.class);
    }
    
    // Upsert (insert if not exists, update if exists)
    public void upsertByEmail(User user) {
        Query query = Query.query(Criteria.where("email").is(user.getEmail()));
        Update update = new Update()
            .set("name", user.getName())
            .set("age", user.getAge())
            .setOnInsert("createdAt", LocalDateTime.now());
        
        mongoTemplate.upsert(query, update, User.class);
    }
}
```

---

## 10. Aggregation Pipeline

```java
// MongoDB Aggregation = SQL GROUP BY + JOIN + transform on steroids

// Count users per city
public Map<String, Long> countByCity() {
    Aggregation agg = Aggregation.newAggregation(
        Aggregation.group("address.city").count().as("count"),
        Aggregation.sort(Sort.Direction.DESC, "count")
    );
    
    AggregationResults<Document> results = mongoTemplate.aggregate(agg, "users", Document.class);
    
    return results.getMappedResults().stream()
        .collect(Collectors.toMap(
            doc -> doc.getString("_id"),
            doc -> doc.getLong("count")
        ));
}

// Average age by role
public List<Document> avgAgeByRole() {
    Aggregation agg = Aggregation.newAggregation(
        Aggregation.match(Criteria.where("active").is(true)),
        Aggregation.group("role")
            .avg("age").as("averageAge")
            .count().as("count"),
        Aggregation.sort(Sort.Direction.DESC, "averageAge")
    );
    
    return mongoTemplate.aggregate(agg, "users", Document.class).getMappedResults();
}

// Pipeline stages:
// $match   → Filter documents (WHERE)
// $group   → Group by field (GROUP BY)
// $sort    → Sort results (ORDER BY)
// $project → Select/reshape fields (SELECT)
// $limit   → Limit results (LIMIT)
// $skip    → Skip results (OFFSET)
// $unwind  → Flatten arrays
// $lookup  → Join collections (LEFT JOIN)
```

---

## 11. Indexing

```java
@Document(collection = "users")
public class User {
    @Id
    private String id;
    
    @Indexed                            // Single field index
    private String name;
    
    @Indexed(unique = true)             // Unique index
    private String email;
    
    @TextIndexed                        // Full-text search index
    private String bio;
    
    @GeoSpatialIndexed(type = GeoSpatialIndexType.GEO_2DSPHERE)
    private GeoJsonPoint location;      // Geospatial index
}

// Compound index (multiple fields)
@Document(collection = "users")
@CompoundIndex(name = "name_email_idx", def = "{'name': 1, 'email': 1}")
public class User { ... }

// TTL index (auto-delete after time)
@Indexed(expireAfterSeconds = 86400)  // Delete after 24 hours
private Date tempTokenExpiry;

// Programmatic index creation
mongoTemplate.indexOps("users").ensureIndex(
    new Index().on("name", Sort.Direction.ASC).on("age", Sort.Direction.DESC)
);
```

---

## 12. Embedded vs Referenced Documents

```
EMBEDDED (denormalized):           REFERENCED (normalized):
┌─────────────────────┐           ┌──────────────┐   ┌──────────────┐
│ User                │           │ User          │   │ Department   │
│ {                   │           │ {             │   │ {            │
│   name: "Dilip",    │           │   name: "D",  │   │   _id: "d1",│
│   address: {        │           │   deptId: "d1"│──►│   name: "IT"│
│     city: "Mumbai"  │           │ }             │   │ }           │
│   }                 │           └──────────────┘   └──────────────┘
│ }                   │
└─────────────────────┘

EMBEDDED when:                    REFERENCED when:
✅ Data always read together     ✅ Data shared by many documents
✅ One-to-few relationship       ✅ One-to-many (large)
✅ Data doesn't change often     ✅ Data changes independently
✅ Better read performance       ✅ Avoids data duplication
```

```java
// Embedded (address stored INSIDE user document)
public class User {
    private Address address;       // Embedded — no @DBRef
    private List<Order> orders;    // Embedded array
}

// Referenced (department is a separate collection)
public class User {
    @DBRef                          // Stores reference: {"$ref":"departments","$id":"abc"}
    private Department department;  // Loaded separately when accessed
}
```

---

## 13. Pagination & Sorting

```java
// Same as JPA!
public interface UserRepository extends MongoRepository<User, String> {
    Page<User> findByActive(boolean active, Pageable pageable);
}

// Service
public Page<User> findAll(int page, int size, String sortBy) {
    Pageable pageable = PageRequest.of(page, size, Sort.by(sortBy).ascending());
    return userRepository.findAll(pageable);
}
```

---

## 14. Transactions

```java
// MongoDB supports multi-document transactions (v4.0+)
// Requires replica set!

@Service
@Transactional
public class OrderService {
    
    @Transactional
    public void placeOrder(String userId, OrderDTO dto) {
        User user = userRepository.findById(userId).orElseThrow();
        
        Order order = new Order();
        order.setUserId(userId);
        order.setProduct(dto.getProduct());
        order.setAmount(dto.getAmount());
        orderRepository.save(order);
        
        user.setOrderCount(user.getOrderCount() + 1);
        userRepository.save(user);
        // Both operations in single transaction
        // If exception → both rolled back
    }
}

// Configure transaction manager
@Configuration
public class MongoConfig {
    @Bean
    MongoTransactionManager transactionManager(MongoDatabaseFactory dbFactory) {
        return new MongoTransactionManager(dbFactory);
    }
}
```

---

## 15. GridFS (File Storage)

```java
// Store large files (>16MB) in MongoDB

@Service
public class FileService {
    
    @Autowired
    private GridFsTemplate gridFsTemplate;
    
    public String storeFile(MultipartFile file) throws IOException {
        ObjectId id = gridFsTemplate.store(
            file.getInputStream(),
            file.getOriginalFilename(),
            file.getContentType()
        );
        return id.toString();
    }
    
    public GridFsResource getFile(String id) {
        GridFSFile file = gridFsTemplate.findOne(
            Query.query(Criteria.where("_id").is(id))
        );
        return gridFsTemplate.getResource(file);
    }
}
```

---

## 16. Change Streams

```java
// React to real-time changes in MongoDB collections

@Component
public class UserChangeListener {
    
    @Autowired
    private MongoTemplate mongoTemplate;
    
    @PostConstruct
    public void watchChanges() {
        new Thread(() -> {
            mongoTemplate.changeStream(User.class)
                .watchCollection("users")
                .listen()
                .doOnNext(event -> {
                    System.out.println("Change: " + event.getOperationType());
                    System.out.println("Document: " + event.getBody());
                })
                .subscribe();
        }).start();
    }
}
```

---

## 17. Best Practices

1. **Design for access patterns** — Embed data you always read together
2. **Index frequently queried fields** — Use `explain()` to analyze queries
3. **Use projections** — Don't load entire documents when you need 2 fields
4. **Limit array sizes** — Don't let embedded arrays grow unbounded
5. **Use MongoTemplate for complex queries** — More control than repository
6. **Use aggregation pipelines** — Push computation to the database
7. **Enable authentication** — Never run MongoDB without auth in production
8. **Use replica sets** — High availability and transactions
9. **Monitor with MongoDB Atlas or Compass** — Query performance analysis

---

## 18. Interview Questions & Answers (50+)

### Beginner

**Q1: What is MongoDB?** NoSQL document database storing JSON-like (BSON) documents. Schema-flexible, horizontally scalable.

**Q2: SQL table vs MongoDB collection?** Table has fixed schema with rows/columns. Collection holds flexible documents that can have different fields.

**Q3: What is BSON?** Binary JSON. MongoDB's internal format. Supports more types than JSON (Date, ObjectId, Binary).

**Q4: What is ObjectId?** 12-byte unique identifier auto-generated for `_id`. Contains timestamp, machine ID, process ID, counter.

**Q5: What is MongoRepository?** Spring Data interface providing CRUD, pagination, sorting for MongoDB. Same concept as JpaRepository.

**Q6: How to query nested fields?** Dot notation: `findByAddressCity()` → `{"address.city": "Mumbai"}`.

**Q7: Embedded vs Referenced?** Embedded: data stored inside document (fast reads). Referenced: separate collection with @DBRef (normalized).

**Q8: What is Spring Data MongoDB?** Spring's abstraction for MongoDB access. Provides MongoRepository, MongoTemplate, query derivation.

---

### Intermediate

**Q9: What is MongoTemplate?** Lower-level API than MongoRepository. Supports complex queries, aggregations, updates, upserts.

**Q10: What is aggregation pipeline?** Series of stages ($match, $group, $sort, $project) that transform and analyze data. Like SQL GROUP BY on steroids.

**Q11: Does MongoDB support transactions?** Yes, since v4.0. Multi-document ACID transactions. Requires replica set.

**Q12: What is sharding?** Horizontal scaling by distributing data across multiple servers based on a shard key.

**Q13: What is a replica set?** Group of MongoDB instances maintaining same data. Primary + secondaries. Auto-failover.

**Q14: What is GridFS?** MongoDB's file storage for files >16MB. Splits files into chunks across two collections.

**Q15: What is Change Streams?** Real-time notifications when data changes. Based on MongoDB's oplog.

---

### Advanced

**Q16: When NOT to use MongoDB?** Heavy joins, strict schema requirements, complex multi-table transactions, financial systems needing strong consistency.

**Q17: What is the $lookup aggregation?** Left outer join between collections. Like SQL JOIN but in aggregation pipeline.

**Q18: How to optimize MongoDB queries?** Proper indexing, projections, covered queries, aggregation instead of app-side processing, connection pooling.

**Q19: What is a covered query?** Query answered entirely from index without reading documents. All queried and returned fields must be in the index.

**Q20: What is write concern?** Level of acknowledgment for write operations: `w:1` (primary), `w:"majority"` (majority of replicas).

---

### Rapid-Fire (Q21–Q50)

**Q21: MongoDB default port?** 27017.

**Q22: `@Document` annotation?** Maps class to MongoDB collection.

**Q23: `@Field` annotation?** Custom field name in MongoDB.

**Q24: `@Indexed` annotation?** Creates index on the field.

**Q25: `@DBRef` annotation?** Creates reference to another collection (not embedded).

**Q26: `@TextIndexed`?** Full-text search index.

**Q27: `@Transient`?** Field NOT stored in MongoDB.

**Q28: `@CompoundIndex`?** Index on multiple fields.

**Q29: TTL index?** Auto-deletes documents after specified time.

**Q30: `findBySkillsContaining`?** Queries array field for a value.

**Q31: `$set` in Update?** Sets a field's value.

**Q32: `$inc` in Update?** Increments a numeric field.

**Q33: `$push` in Update?** Adds element to array.

**Q34: `$pull` in Update?** Removes element from array.

**Q35: `$addToSet`?** Adds to array only if not already present.

**Q36: `$unwind` in aggregation?** Deconstructs array into separate documents.

**Q37: `$match` in aggregation?** Filters documents (like WHERE).

**Q38: `$group` in aggregation?** Groups by field and applies accumulators.

**Q39: `$project` in aggregation?** Reshapes documents (like SELECT).

**Q40: MongoDB Compass?** GUI tool for querying and visualizing MongoDB data.

**Q41: MongoDB Atlas?** Cloud-hosted MongoDB service.

**Q42: What is oplog?** Operations log — records all write operations. Used for replication and change streams.

**Q43: What is capped collection?** Fixed-size collection that overwrites oldest documents when full.

**Q44: `save()` vs `insert()`?** save: insert or update. insert: only insert (fails if exists).

**Q45: What is read preference?** Which replica to read from: primary, secondary, nearest.

**Q46: What is read concern?** Level of data consistency: local, majority, linearizable.

**Q47: Spring profile for MongoDB?** `spring.data.mongodb.uri` in application-{profile}.properties.

**Q48: How to test MongoDB?** Use `@DataMongoTest` + embedded MongoDB (de.flapdoodle) or Testcontainers.

**Q49: `@DataMongoTest`?** Test slice for MongoDB repositories. Auto-configures MongoTemplate + repos.

**Q50: Mongoose vs Spring Data MongoDB?** Mongoose is Node.js ODM. Spring Data MongoDB is Java ODM. Same concept, different language.

---

## 📚 References

- [Spring Data MongoDB Reference](https://docs.spring.io/spring-data/mongodb/reference/)
- [MongoDB Manual](https://www.mongodb.com/docs/manual/)
- [MongoDB University (Free Courses)](https://university.mongodb.com/)
- [Baeldung Spring MongoDB](https://www.baeldung.com/spring-data-mongodb-tutorial)

---

> **Previous Topic:** [← 20 - Logging](../20-logging-log4j/README.md)  
> **Next Topic:** [22 - Docker →](../22-docker/README.md)
