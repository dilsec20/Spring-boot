# 📊 Spring for GraphQL — Complete In-Depth Guide

> **"GraphQL is a query language for your API. Instead of multiple REST endpoints returning fixed data structures, GraphQL exposes a single endpoint that allows the client to ask for exactly what they need, nothing more, nothing less."**

---

## 📑 Table of Contents

1. [GraphQL vs REST](#1-graphql-vs-rest)
2. [What is Spring for GraphQL?](#2-what-is-spring-for-graphql)
3. [Setup & Configuration](#3-setup--configuration)
4. [The Schema Definition (`.graphqls`)](#4-the-schema-definition-graphqls)
5. [Queries (`@QueryMapping`)](#5-queries-querymapping)
6. [Mutations (`@MutationMapping`)](#6-mutations-mutationmapping)
7. [Schema Mapping (Nested Objects)](#7-schema-mapping-nested-objects)
8. [Subscriptions (Real-time updates)](#8-subscriptions-real-time-updates)
9. [GraphiQL UI](#9-graphiql-ui)
10. [Error Handling](#10-error-handling)
11. [The N+1 Problem & DataLoader](#11-the-n1-problem--dataloader)
12. [Interview Questions & Answers (50+)](#12-interview-questions--answers-50)

---

## 1. GraphQL vs REST

| Feature | REST | GraphQL |
|---------|------|---------|
| **Endpoints** | Multiple (`/users`, `/orders`) | Single (`/graphql`) |
| **Data Fetching** | Over-fetching or Under-fetching | Exact data fetching |
| **Structure** | Fixed by server | Defined by client query |
| **Versioning** | `v1`, `v2` endpoints | Deprecate fields, no versioning needed |
| **Methods** | GET, POST, PUT, DELETE | Query (Read), Mutation (Write), Subscription |

**The Over-fetching Problem (REST):**
I just need a user's `name`.
REST `/users/1` returns: `{ id, name, email, address, phone, avatar, ... }` (Wasted bandwidth!)

**The Solution (GraphQL):**
Client asks for: `query { user(id: 1) { name } }`
Server returns: `{ "user": { "name": "John" } }`

---

## 2. What is Spring for GraphQL?

Spring for GraphQL is the official integration of GraphQL Java into the Spring ecosystem. It provides annotation-based programming models, data fetchers, and easy integration with Spring Data and Spring Security.

---

## 3. Setup & Configuration

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-graphql</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

```properties
# application.properties
spring.graphql.graphiql.enabled=true      # Enable the visual IDE
spring.graphql.graphiql.path=/graphiql    # Path for the IDE
spring.graphql.path=/graphql              # The actual API endpoint
```

---

## 4. The Schema Definition (`.graphqls`)

GraphQL requires a strict schema. Place this file in `src/main/resources/graphql/schema.graphqls`.

```graphql
# src/main/resources/graphql/schema.graphqls

# 1. Define custom types
type User {
    id: ID!          # ! means required (non-null)
    name: String!
    email: String!
    orders: [Order]  # Array of Orders
}

type Order {
    id: ID!
    product: String!
    amount: Float!
}

# 2. Define Queries (Read operations)
type Query {
    getUserById(id: ID!): User
    getAllUsers: [User]
}

# 3. Define Mutations (Write operations)
type Mutation {
    createUser(name: String!, email: String!): User
    deleteUser(id: ID!): Boolean
}

# 4. Define Subscriptions (Real-time streams)
type Subscription {
    userCreated: User
}
```

---

## 5. Queries (`@QueryMapping`)

Controllers in GraphQL handle mappings to the schema fields.

```java
@Controller
@RequiredArgsConstructor
public class UserController {
    
    private final UserRepository userRepository;

    // Matches 'getUserById' in schema.graphqls
    @QueryMapping
    public User getUserById(@Argument Long id) {
        return userRepository.findById(id).orElse(null);
    }

    // Matches 'getAllUsers' in schema.graphqls
    @QueryMapping
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
}
```

**Client Query:**
```graphql
query {
  getUserById(id: 1) {
    name
    email
  }
}
```

---

## 6. Mutations (`@MutationMapping`)

Mutations alter data (Create, Update, Delete).

```java
@Controller
@RequiredArgsConstructor
public class UserMutationController {
    
    private final UserRepository userRepository;

    @MutationMapping
    public User createUser(@Argument String name, @Argument String email) {
        User user = new User(name, email);
        return userRepository.save(user);
    }
    
    @MutationMapping
    public Boolean deleteUser(@Argument Long id) {
        userRepository.deleteById(id);
        return true;
    }
}
```

**Client Mutation:**
```graphql
mutation {
  createUser(name: "Alice", email: "alice@demo.com") {
    id
    name
  }
}
```

---

## 7. Schema Mapping (Nested Objects)

GraphQL only resolves nested objects if the client asks for them. This is where `@SchemaMapping` shines.

Assume `User` has `List<Order> orders`. The `orders` are stored in a different database table.

```java
@Controller
@RequiredArgsConstructor
public class UserOrderController {
    
    private final OrderRepository orderRepository;

    // This method is ONLY called if the client explicitly requests 'orders' inside 'User'
    @SchemaMapping(typeName = "User", field = "orders")
    public List<Order> getOrdersForUser(User user) {
        // Fetch orders for this specific user
        return orderRepository.findByUserId(user.getId());
    }
}
```

**Client Query:**
```graphql
query {
  getUserById(id: 1) {
    name
    orders {       # This triggers getOrdersForUser()!
      product
      amount
    }
  }
}
```
*If the client omits `orders`, `getOrdersForUser()` is NEVER executed, saving a database call!*

---

## 8. Subscriptions (Real-time updates)

Subscriptions use WebSockets to stream data to clients.

```java
@Controller
public class UserSubscriptionController {
    
    // Using Project Reactor Flux for the stream
    @SubscriptionMapping
    public Flux<User> userCreated() {
        // Emits a new User every 2 seconds (simulate real-time stream)
        return Flux.interval(Duration.ofSeconds(2))
                   .map(i -> new User(i, "User-" + i, "user@mail.com"));
    }
}
```

**Client Subscription:**
```graphql
subscription {
  userCreated {
    id
    name
  }
}
```

---

## 9. GraphiQL UI

GraphiQL is a built-in visual IDE for executing GraphQL queries.
1. Enable it in `application.properties`.
2. Start Spring Boot.
3. Open browser: `http://localhost:8080/graphiql`
4. Use the interface to write queries, explore the schema documentation automatically generated from your `.graphqls` file, and see real-time results.

---

## 10. Error Handling

Spring for GraphQL provides a `DataFetcherExceptionResolver` to map Java Exceptions to GraphQL Errors.

```java
@Component
public class GraphQLExceptionResolver extends DataFetcherExceptionResolverAdapter {

    @Override
    protected GraphQLError resolveToSingleError(Throwable ex, DataFetchingEnvironment env) {
        if (ex instanceof UserNotFoundException) {
            return GraphqlErrorBuilder.newError()
              .errorType(ErrorType.NOT_FOUND)
              .message(ex.getMessage())
              .path(env.getExecutionStepInfo().getPath())
              .location(env.getField().getSourceLocation())
              .build();
        }
        return null; // Fallback to default handling
    }
}
```

---

## 11. The N+1 Problem & DataLoader

**The Problem:**
If you fetch 50 Users, and the client requests their `orders`, the `@SchemaMapping` for `orders` will execute 50 times (1 query for users + 50 queries for orders = 51 queries). This kills performance.

**The Solution:** `BatchMapping` (DataLoader). It groups the 50 user IDs and fetches all their orders in ONE single query.

```java
@Controller
@RequiredArgsConstructor
public class UserOrderBatchController {
    
    private final OrderRepository orderRepository;

    // Replaces @SchemaMapping. Fetches all orders for a list of users at once!
    @BatchMapping(typeName = "User", field = "orders")
    public Map<User, List<Order>> getOrdersForUsers(List<User> users) {
        
        List<Long> userIds = users.stream().map(User::getId).toList();
        
        // ONE single database query: SELECT * FROM orders WHERE user_id IN (1, 2, 3...)
        List<Order> allOrders = orderRepository.findByUserIdIn(userIds);
        
        // Group orders by User
        return allOrders.stream()
                        .collect(Collectors.groupingBy(Order::getUser));
    }
}
```

---

## 12. Interview Questions & Answers (50+)

### Beginner

**Q1: What is GraphQL?** A query language for APIs that allows clients to request exactly the data they need from a single endpoint.

**Q2: GraphQL vs REST?** REST has multiple endpoints returning fixed structures. GraphQL has one endpoint returning flexible, client-defined structures.

**Q3: What is a Query in GraphQL?** Equivalent to a GET request in REST. Used for fetching data.

**Q4: What is a Mutation?** Equivalent to POST/PUT/DELETE in REST. Used for modifying data.

**Q5: What is a Subscription?** Real-time data fetching over WebSockets. Pushes data to the client when events happen.

**Q6: What is a Schema in GraphQL?** A strong type definition of all available queries, mutations, and types the API exposes. Written in SDL (Schema Definition Language).

**Q7: Default endpoint for Spring GraphQL?** `/graphql` (Accepts POST requests).

**Q8: What is GraphiQL?** An in-browser IDE for exploring GraphQL APIs and executing queries.

---

### Intermediate

**Q9: What is `@QueryMapping`?** Spring annotation that maps a Java method to a GraphQL Query defined in the schema.

**Q10: What is `@SchemaMapping`?** Maps a Java method to a specific field on a GraphQL Type (useful for resolving nested relations like `User.orders`).

**Q11: What is `@Argument`?** Spring annotation to bind a GraphQL query argument to a Java method parameter (like `@RequestParam` in MVC).

**Q12: What is the N+1 problem in GraphQL?** Fetching a list of N parent objects, then individually fetching child objects for each parent, resulting in N+1 database queries.

**Q13: How does Spring resolve the N+1 problem?** Using `@BatchMapping` (which uses GraphQL Java's `DataLoader` under the hood) to batch child queries into a single IN clause query.

**Q14: Where do you place the `.graphqls` file?** In `src/main/resources/graphql/`.

**Q15: Does GraphQL use HTTP Status Codes for errors?** Usually no. It returns HTTP 200 OK, but includes an `"errors": [...]` array in the JSON response payload.

---

### Rapid-Fire (Q16–Q50)

**Q16: Can GraphQL return JSON?** Yes, the response is standard JSON.

**Q17: What does `!` mean in a GraphQL schema (e.g., `String!`)?** It means the field is Non-Nullable (required).

**Q18: How do you pass an object in a Mutation?** Using `input` types in the schema.

**Q19: What is an `input` type?** A special type used only for passing complex objects as arguments into queries/mutations.

**Q20: What is an `enum` in GraphQL?** A restricted set of string values.

**Q21: What is a `scalar`?** Primitive types: `Int`, `Float`, `String`, `Boolean`, `ID`. You can also define custom scalars (like `Date`).

**Q22: Does Spring GraphQL support Spring Security?** Yes. You can use `@PreAuthorize` directly on `@QueryMapping` methods.

**Q23: How do you test GraphQL endpoints in Spring?** Using `GraphQlTester` or `HttpGraphQlTester`.

**Q24: Can you use GraphQL with Spring WebFlux?** Yes, Spring GraphQL works with both Spring MVC and Spring WebFlux.

**Q25: What is a DataFetcher?** The core component in GraphQL Java that provides the logic to fetch data for a specific field. Spring's annotations wrap DataFetchers.

**Q26: What is the `ID` type in GraphQL?** A unique identifier, serialized as a String, but signifies it is not intended to be human-readable.

**Q27: Can you cache GraphQL requests easily?** Harder than REST because everything is a POST request to `/graphql`. Requires client-side caching (Apollo) or persisted queries.

**Q28: What is Over-fetching?** Getting more data than you need (common in REST).

**Q29: What is Under-fetching?** Not getting enough data in one request, requiring multiple subsequent API calls (common in REST).

**Q30: What is a GraphQL Fragment?** A reusable set of fields that can be included in multiple queries to reduce duplication.

**Q31: What is Introspection?** The ability of a GraphQL server to describe its own schema. Tools like GraphiQL use this to provide autocomplete.

**Q32: Should you disable Introspection in production?** Yes, generally, to hide your schema from potential attackers.

**Q33: Can GraphQL queries be nested infinitely?** Yes, which is a security risk.

**Q34: How do you prevent deeply nested queries?** Configure a Max Query Depth limit in the GraphQL execution engine.

**Q35: How to handle file uploads in GraphQL?** Not natively supported by standard GraphQL. Typically done via a separate REST endpoint, or using the `graphql-multipart-request-spec`.

**Q36: What is a Union type in GraphQL?** A type that can be one of several object types (e.g., `union SearchResult = Human | Droid | Starship`).

**Q37: What is an Interface in GraphQL?** An abstract type that includes a certain set of fields that a type must include to implement the interface.

**Q38: Does GraphQL support pagination?** Yes, usually through cursor-based pagination (Connections, Edges, Nodes) as defined by the Relay specification.

**Q39: What is Apollo?** A popular suite of tools (Client and Server) for GraphQL. Spring Boot replaces Apollo Server for Java backends.

**Q40: Does Spring Data support GraphQL?** Yes! `spring-boot-starter-data-jpa` + `graphql` allows you to expose Repositories directly to GraphQL via `@GraphQlRepository`.

**Q41: What is `@GraphQlRepository`?** Auto-generates DataFetchers for a Spring Data repository, similar to Spring Data REST.

**Q42: Can mutations return data?** Yes, it is best practice for a mutation to return the object it just modified.

**Q43: What is the `DataFetchingEnvironment`?** An object passed to DataFetchers containing context, arguments, and the selection set of the query.

**Q44: What are GraphQL directives?** Identifiers preceded by `@` (like `@deprecated`) that provide instructions to the GraphQL engine.

**Q45: How to format dates in GraphQL?** Define a custom `Scalar` for Date, or return it as a String.

**Q46: Why is REST still used if GraphQL is so good?** REST is simpler, utilizes HTTP caching natively, better for file uploads, and easier to rate-limit.

**Q47: Can I use both REST and GraphQL in the same Spring Boot app?** Absolutely.

**Q48: How do you handle authentication in subscriptions (WebSockets)?** Since WebSockets don't easily send HTTP headers post-connection, auth tokens are usually passed in the `connection_init` payload.

**Q49: What is the `ExecutionResult`?** The final object output by the GraphQL Java engine, containing `data` and `errors`.

**Q50: Does GraphQL require a specific database?** No, it is database-agnostic. You resolve data from anywhere (SQL, NoSQL, external APIs).

---

## 📚 References

- [Spring for GraphQL Reference](https://docs.spring.io/spring-graphql/reference/)
- [GraphQL Official Website](https://graphql.org/)
- [GraphQL Java](https://www.graphql-java.com/)

---

> **Previous Topic:** [← 32 - Spring WebFlux](../32-spring-webflux/README.md)  
> **Next Topic:** [34 - Caching & Redis →](../34-caching-redis/README.md)
