# 🚀 Spring Boot Mastery: Beginner to Expert (31-Topic Guide)

Welcome to the definitive **Spring Boot Mastery** repository! This is a complete, in-depth, production-ready guide designed to take you from absolute zero (Core Java) to a DevOps-proficient Spring Boot Architect. 

Every single topic is designed like a "mini-book" containing theory, architecture diagrams, extensive code examples, best practices, and **50+ rapid-fire interview questions**.

---

## 📚 Curriculum Index

### 🟢 Batch 1: Foundations (Java & Build Tools)
Before conquering Spring Boot, you must master its foundation.

1. [Core Java Essentials](01-core-java/README.md) *(OOP, Collections, Streams, Multithreading)*
2. [Maven](02-maven/README.md) *(POM, Lifecycles, Dependencies)*
3. [Gradle](03-gradle/README.md) *(Groovy/Kotlin DSL, Tasks)*
4. [JUnit 5 & Mockito](04-junit/README.md) *(TDD, Assertions, Mocking)*
5. [Git & Version Control](05-git/README.md) *(Branching, Rebasing, Workflows)*
6. [Data Structures & Algorithms (DSA)](06-dsa/README.md) + [Advanced DSA](06-dsa/ADVANCED.md) *(Arrays to DP, Segment Trees, CP Tricks)*

### 🔵 Batch 2: Web & Database Fundamentals
Understanding how the web and databases work under the hood.

7. [JDBC](07-jdbc/README.md) *(Connections, Statements, Connection Pooling)*
8. [Servlets & JSP](08-servlets-and-jsp/README.md) *(The raw web layer before Spring MVC)*
9. [REST APIs & Web Services](09-rest-api-and-web-services/README.md) *(HTTP, Status Codes, Best Practices)*
10. [ORM Concepts](10-orm-tools/README.md) *(Object-Relational Impedance Mismatch)*
11. [Hibernate](11-hibernate/README.md) *(Session, Caching, HQL)*

### 🟡 Batch 3: Spring Boot Core
The heart of the framework.

12. [Spring Framework Core](12-spring-framework/README.md) *(IoC, DI, ApplicationContext, AOP)*
13. [Spring Boot Basics & REST](13-spring-rest-api-springboot/README.md) *(Auto-configuration, Controllers, Actuator)*
14. [Spring JDBC](14-spring-jdbc/README.md) *(JdbcTemplate, NamedParameterJdbcTemplate)*
15. [Spring Data JPA](15-spring-data-jpa/README.md) *(Repositories, Derived Queries, Entity Relationships)*
16. [Spring MVC Project](16-project-springboot-mvc/README.md) *(Thymeleaf, Form Handling, Validation)*

### 🟠 Batch 4: Security & Production Readiness
Securing your application for the real world.

17. [Spring Security](17-spring-security/README.md) *(Authentication, Authorization, Filters)*
18. [JWT (JSON Web Tokens)](18-jwt/README.md) *(Stateless Auth, Signing, Validation)*
19. [OAuth2 & OIDC](19-oauth2/README.md) *(Social Login, Authorization Server)*
20. [Logging (Log4j / Logback)](20-logging-log4j/README.md) *(Appenders, Levels, MDC, Distributed Tracing)*

### 🔴 Batch 5: Advanced Data & Modern Tech
Scaling out and adopting modern paradigms.

21. [MongoDB & NoSQL](21-springboot-mongodb/README.md) *(Document DBs, MongoTemplate)*
22. [Docker](22-docker/README.md) *(Containerization, Dockerfile, Compose)*
23. [Cloud Deployment](23-cloud-deployment/README.md) *(AWS, GCP, Azure, PaaS)*
24. [Spring AI](24-spring-ai/README.md) *(LLMs, RAG, Prompt Engineering)*
25. [DeepSeek & Ollama](25-deepseek-ollama-spring/README.md) *(Running Local Private AI Models)*

### 🟣 Batch 6: Distributed Systems
Building at scale.

26. [Microservices](26-microservices/README.md) *(Eureka, Gateway, Circuit Breaker, Saga)*
27. [Apache Kafka](27-springboot-kafka/README.md) *(Event Streaming, Producers, Consumer Groups)*

### ⚫ Batch 7: DevOps & Infrastructure
Automating and provisioning.

28. [Linux Essentials](28-linux/README.md) *(Commands, Permissions, Systemd)*
29. [Ansible](29-ansible/README.md) *(Configuration Management, Playbooks, Roles)*
30. [Jenkins](30-jenkins/README.md) *(CI/CD, Pipelines, Groovy)*
31. [Terraform](31-terraform/README.md) *(Infrastructure as Code, State, Providers)*

### 🚀 Batch 8: Master Level (Bonus)
Bleeding-edge topics for modern enterprise apps.

32. [Spring WebFlux](32-spring-webflux/README.md) *(Reactive Programming, Project Reactor)*
33. [Spring for GraphQL](33-spring-graphql/README.md) *(Queries, Mutations, Subscriptions)*
34. [Caching & Redis](34-caching-redis/README.md) *(@Cacheable, Performance Optimization)*
35. [Advanced Testing](35-advanced-testing/README.md) *(Testcontainers, @WebMvcTest, Integration Tests)*
36. [GraalVM Native Image](36-graalvm-native/README.md) *(AOT Compilation, Millisecond Startup)*
37. [WebSockets](37-websockets/README.md) *(Real-Time Communication, STOMP)*
38. [Scenario-Based Interviews](38-scenario-based-interviews/README.md) *(Knowing vs Mastering, Architecture Scenarios)*

---

## 📖 How to Use This Repository

1. **Sequential Learning:** If you are a beginner, go from Topic 01 to 31 in order.
2. **Reference Material:** If you are a professional, jump directly to the topic you need (e.g., [Microservices](26-microservices/README.md) or [Kafka](27-springboot-kafka/README.md)).
3. **Interview Preparation:** Scroll to the bottom of *any* topic's `README.md` to find **50+ categorized interview questions and answers** (Beginner, Intermediate, Rapid-Fire).

---

## 🗓️ 6-Month Mastery Study Plan (15-20 hours/week)

To truly master this curriculum and reach the 20+ LPA (Architect/Senior) level, follow this project-driven timeline:

### Month 1: The Foundation (Batches 1 & 2)
*   **Focus:** Core Java, Git, Maven/Gradle, JDBC, Servlets, Hibernate.
*   **Milestone:** Build a pure Java CLI application connected to a MySQL database using raw Hibernate.
*   **Daily Habit:** 1 LeetCode problem (Topic 06). Don't ignore DSA.

### Month 2: Spring Boot Core (Batch 3)
*   **Focus:** Spring Core (IoC/DI), REST APIs, Spring Data JPA, MVC.
*   **Milestone:** Build a robust single-monolith REST API (e.g., an E-Commerce backend) with full CRUD and proper exception handling.

### Month 3: Security & Advanced Data (Batches 4 & 5)
*   **Focus:** Spring Security, JWT, OAuth2, MongoDB, Docker.
*   **Milestone:** Add JWT Authentication to your API, Dockerize it, and write a `docker-compose.yml` to spin up your API alongside a Postgres database.

### Month 4: The Microservices Transition (Batch 6)
*   **Focus:** Eureka, API Gateway, Circuit Breakers, Apache Kafka.
*   **Milestone:** Break your monolith into 3 microservices (e.g., `OrderService`, `InventoryService`, `NotificationService`). Have `OrderService` send a Kafka message to `NotificationService` when an order is placed.

### Month 5: DevOps & Cloud (Batch 7)
*   **Focus:** Linux, Ansible, Jenkins, Terraform.
*   **Milestone:** Write a Terraform script to create an AWS EC2 instance, and write a Jenkins pipeline that builds your Dockerized microservices and deploys them to EC2 automatically.

### Month 6: Master Level & Interview Prep (Batch 8 + Topic 38)
*   **Focus:** WebFlux, GraphQL, Redis Caching, Advanced Testing, GraalVM.
*   **Milestone:** Add a Redis cache to solve a Thundering Herd problem, write Testcontainers integration tests, and drill the **38-scenario-based-interviews** folder.

### 💡 3 Rules for Success
1. **Never copy-paste code.** Type it out. Muscle memory is real.
2. **Break things on purpose.** Once it works, change a config, break it, and read the stack trace.
3. **Keep an "Error Log" notebook.** Document tough bugs. Interviewers love hearing about your debugging sessions.

## 💡 Philosophy
> *"Don't just learn the syntax; understand the architecture. Tools change, concepts remain."*

Happy coding! 🚀
