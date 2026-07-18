# 🏛️ Software Engineering Concepts — Interview & Certification Guide

> **"Coding gets you in. Software Engineering concepts get you promoted. Every HackerRank/HackerEarth SE certification and FAANG interview tests these."**

---

## 📑 Table of Contents
1. [SDLC (Software Development Life Cycle)](#1-sdlc)
2. [Waterfall Model](#2-waterfall-model)
3. [Agile Methodology](#3-agile-methodology)
4. [Scrum Framework (Deep Dive)](#4-scrum-framework)
5. [Kanban](#5-kanban)
6. [DevOps & CI/CD](#6-devops--cicd)
7. [Software Design Principles (SOLID)](#7-solid-principles)
8. [Design Patterns (Gang of Four)](#8-design-patterns)
9. [Software Architecture Patterns](#9-architecture-patterns)
10. [UML Diagrams](#10-uml-diagrams)
11. [Testing Types & Strategies](#11-testing-types)
12. [Version Control (Git Workflows)](#12-version-control)
13. [Code Review Best Practices](#13-code-review)
14. [Estimation Techniques](#14-estimation-techniques)
15. [Requirement Engineering](#15-requirement-engineering)
16. [Software Quality & Metrics](#16-software-quality)
17. [Risk Management](#17-risk-management)
18. [Coupling & Cohesion](#18-coupling--cohesion)
19. [API Design Principles](#19-api-design)
20. [12-Factor App](#20-twelve-factor-app)
21. [CAP Theorem & Distributed Systems](#21-cap-theorem)
22. [Interview Questions (50+)](#22-interview-questions)

---

## 1. SDLC (Software Development Life Cycle)

### What is SDLC?
A **structured process** for planning, creating, testing, and deploying a software system. Every company follows some form of SDLC.

```
┌──────────────────────────────────────────────────────────────┐
│                    SDLC PHASES                                │
│                                                               │
│  1. PLANNING        → What are we building? Why? Budget?     │
│  2. REQUIREMENTS    → What exactly should it do?             │
│  3. DESIGN          → How will we build it? Architecture?    │
│  4. IMPLEMENTATION  → Write the code                         │
│  5. TESTING         → Does it work? Find bugs.               │
│  6. DEPLOYMENT      → Release to users                       │
│  7. MAINTENANCE     → Fix bugs, add features, monitor        │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

### SDLC Models Comparison

| Model | Flow | Best For | Risk |
|:---|:---|:---|:---|
| **Waterfall** | Linear, sequential | Small, well-defined projects | High (late testing) |
| **Agile** | Iterative, incremental | Evolving requirements | Low (early feedback) |
| **Spiral** | Risk-driven iterations | Large, risky projects | Medium |
| **V-Model** | Verification & Validation | Safety-critical systems | Medium |
| **Prototype** | Build prototype first | Unclear requirements | Low |
| **RAD** | Rapid Application Dev | Time-critical projects | Medium |
| **Iterative** | Repeat cycles | Large systems | Low |
| **Big Bang** | No formal process | Very small projects | Very High |

---

## 2. Waterfall Model

```
┌──────────┐
│ Require- │
│  ments   │
└────┬─────┘
     ▼
┌──────────┐
│  Design  │
└────┬─────┘
     ▼
┌──────────┐
│  Coding  │     Each phase MUST complete before the next begins.
└────┬─────┘     No going back! (like a waterfall — water doesn't flow uphill)
     ▼
┌──────────┐
│ Testing  │
└────┬─────┘
     ▼
┌──────────┐
│ Deploy & │
│ Maintain │
└──────────┘
```

### Characteristics
- **Sequential** — one phase at a time, no overlap
- **Documentation-heavy** — detailed docs before coding starts
- **No changes** — once requirements are signed off, no modifications
- **Late testing** — bugs found only in testing phase (expensive to fix!)

### When to Use
- Requirements are **100% clear and fixed** (e.g., government contracts)
- Short projects with **no ambiguity**
- Regulatory/compliance projects needing **documentation**

### Advantages & Disadvantages

| ✅ Advantages | ❌ Disadvantages |
|:---|:---|
| Simple to understand and manage | No flexibility for changes |
| Clear milestones and deadlines | Customer sees product only at the end |
| Works for well-defined projects | Bugs found late = expensive |
| Heavy documentation for compliance | High risk of building wrong product |

---

## 3. Agile Methodology

### The Agile Manifesto (2001)

```
We value:
  ✅ Individuals and interactions     OVER  processes and tools
  ✅ Working software                 OVER  comprehensive documentation
  ✅ Customer collaboration           OVER  contract negotiation
  ✅ Responding to change             OVER  following a plan

(Items on the right still have value, but we value items on the left MORE)
```

### 12 Agile Principles (Know at least 5 for interviews!)

```
1.  Highest priority: satisfy customer through EARLY and CONTINUOUS delivery
2.  Welcome CHANGING requirements, even late in development
3.  Deliver working software FREQUENTLY (weeks, not months)
4.  Business + developers must work TOGETHER daily
5.  Build projects around MOTIVATED individuals, give them support and trust
6.  Face-to-face conversation is the most effective communication
7.  WORKING SOFTWARE is the primary measure of progress
8.  Sustainable development — maintain constant pace indefinitely
9.  Continuous attention to TECHNICAL EXCELLENCE and good design
10. SIMPLICITY — maximize the amount of work NOT done
11. Best architectures emerge from SELF-ORGANIZING teams
12. Team regularly REFLECTS on how to become more effective
```

### Agile vs Waterfall

| Aspect | Waterfall | Agile |
|:---|:---|:---|
| **Approach** | Sequential, linear | Iterative, incremental |
| **Requirements** | Fixed upfront | Evolving, flexible |
| **Customer Involvement** | Only at start and end | Continuous |
| **Delivery** | One big release at end | Frequent small releases |
| **Testing** | After development | Continuous (every sprint) |
| **Documentation** | Heavy | Lightweight, just enough |
| **Team Size** | Large, specialized | Small, cross-functional |
| **Change** | Expensive, discouraged | Welcomed, expected |
| **Risk** | High (late feedback) | Low (early feedback) |
| **Best For** | Well-defined, stable requirements | Dynamic, evolving products |

### Agile Frameworks

```
AGILE (umbrella term / philosophy)
  ├── Scrum (most popular — 58% of Agile teams)
  ├── Kanban (visual workflow)
  ├── XP (Extreme Programming — engineering practices)
  ├── SAFe (Scaled Agile Framework — for large orgs)
  ├── Lean (eliminate waste)
  └── Crystal, DSDM, FDD (less common)
```

---

## 4. Scrum Framework (Deep Dive)

### Scrum Overview

```
┌───────────────────────────────────────────────────────────────┐
│                     SCRUM FRAMEWORK                            │
│                                                                │
│  ROLES:                                                        │
│  ├── Product Owner (WHAT to build — prioritizes backlog)      │
│  ├── Scrum Master (HOW to work — removes blockers)            │
│  └── Dev Team (5-9 people — builds the product)               │
│                                                                │
│  ARTIFACTS:                                                    │
│  ├── Product Backlog (ordered list of ALL features)           │
│  ├── Sprint Backlog (features selected for THIS sprint)       │
│  └── Increment (working software at end of sprint)            │
│                                                                │
│  CEREMONIES (Events):                                          │
│  ├── Sprint Planning (what will we do this sprint?)           │
│  ├── Daily Standup (15 min — what did I do, will do, blockers)│
│  ├── Sprint Review (demo to stakeholders)                     │
│  └── Sprint Retrospective (what went well, improve?)          │
│                                                                │
│  SPRINT: Fixed time-box (usually 2 weeks)                     │
│                                                                │
└───────────────────────────────────────────────────────────────┘
```

### Sprint Lifecycle

```
          Sprint (2 weeks)
    ┌─────────────────────────────────┐
    │                                 │
┌───┴───┐  ┌─────┐  ┌─────┐  ┌──────┴──┐  ┌─────────┐
│Sprint │  │Daily│  │Daily│  │ Sprint   │  │ Sprint  │
│Planning│  │Stand│  │Stand│  │ Review   │  │ Retro-  │
│(4 hrs) │  │(15m)│  │(15m)│  │(Demo 2h) │  │spective │
│        │  │     │  │     │  │          │  │ (1.5h)  │
└────────┘  └─────┘  └─────┘  └──────────┘  └─────────┘
  Day 1     Day 2-9  Day 10    Day 10        Day 10
                     (repeat)  (end of sprint)
```

### User Stories

```
Format: "As a [role], I want [feature], so that [benefit]"

Example:
  "As a customer, I want to filter products by price, so that I can find 
   items within my budget."

Acceptance Criteria:
  ✅ Price range slider appears on product listing page
  ✅ Products update in real-time when slider moves
  ✅ Min price defaults to ₹0, max to highest product price
  ✅ Works on mobile and desktop
```

### Story Points & Estimation

```
Story Points: Relative effort, NOT hours!
  Use Fibonacci: 1, 2, 3, 5, 8, 13, 21

  1 point  = Trivially simple (change button color)
  2 points = Simple (add a new field to a form)
  3 points = Moderate (create a new API endpoint)
  5 points = Complex (build a search feature with filters)
  8 points = Very complex (integrate payment gateway)
  13 points = Needs to be broken down further!

Velocity: Total story points completed per sprint
  Sprint 1: 25 points
  Sprint 2: 30 points
  Sprint 3: 28 points
  Average velocity: 28 points/sprint
  
  → "We can deliver ~28 points of work every 2 weeks"
```

---

## 5. Kanban

```
┌──────────┬──────────┬─────────────┬──────────┬──────────┐
│ BACKLOG  │   TODO   │ IN PROGRESS │ TESTING  │   DONE   │
│          │          │   (WIP: 3)  │ (WIP: 2) │          │
├──────────┼──────────┼─────────────┼──────────┼──────────┤
│ Feature E│ Feature C│ Feature A   │Feature B │Feature X │
│ Feature F│ Feature D│ Bug Fix #42 │          │Feature Y │
│ Bug #55  │          │ API Update  │          │Bug #30   │
│          │          │             │          │          │
└──────────┴──────────┴─────────────┴──────────┴──────────┘
                         ↑ WIP Limit!
                    Max 3 items in progress
```

### Scrum vs Kanban

| Aspect | Scrum | Kanban |
|:---|:---|:---|
| **Iterations** | Fixed sprints (2 weeks) | Continuous flow |
| **Roles** | PO, SM, Dev Team | No prescribed roles |
| **Board** | Reset every sprint | Continuous, never reset |
| **WIP Limits** | Sprint backlog = limit | Explicit per column |
| **Changes** | Not during sprint | Anytime |
| **Metrics** | Velocity | Lead time, cycle time |
| **Best For** | Product development | Maintenance, support, ops |

---

## 6. DevOps & CI/CD

### DevOps Culture

```
BEFORE DevOps:
  Dev team: "Code works on my machine! Throwing over the wall to Ops."
  Ops team: "It's broken in production. Not our problem, Dev wrote it."
  → Blame game, slow releases, frustrated teams

AFTER DevOps:
  Dev + Ops = ONE team responsible for the ENTIRE lifecycle
  → Build it, run it, own it
  → Automate everything
  → Release multiple times per day
```

### CI/CD Pipeline

```
┌────────┐   ┌───────┐   ┌───────┐   ┌────────┐   ┌──────────┐   ┌────────┐
│ COMMIT │──▶│ BUILD │──▶│ TEST  │──▶│ QUALITY│──▶│  DEPLOY  │──▶│MONITOR │
│ (Git)  │   │(Maven)│   │(JUnit)│   │  GATE  │   │(K8s/AWS) │   │(Grafana│
│        │   │       │   │       │   │(SonarQ)│   │          │   │Datadog)│
└────────┘   └───────┘   └───────┘   └────────┘   └──────────┘   └────────┘
   CI ─────────────────────────────────▶│◀──── CD ────────────────────────▶

CI = Continuous Integration: Every commit is automatically built & tested
CD = Continuous Delivery: Code is always in a deployable state
CD = Continuous Deployment: Every passing build is auto-deployed to production
```

### CI vs CD vs CD

| Term | What | Example |
|:---|:---|:---|
| **CI** (Integration) | Auto build + test on every commit | Jenkins runs tests on PR |
| **CD** (Delivery) | Auto deploy to staging. Manual approval for prod. | One-click deploy to prod |
| **CD** (Deployment) | Auto deploy to production. No manual step. | Netflix deploys 1000x/day |

---

## 7. SOLID Principles

### S — Single Responsibility Principle

```
"A class should have only ONE reason to change."

❌ BAD:
class UserService {
    void createUser() { ... }
    void sendEmail() { ... }      // Email is NOT user's responsibility!
    void generateReport() { ... } // Reporting is NOT user's responsibility!
}

✅ GOOD:
class UserService { void createUser() { ... } }
class EmailService { void sendEmail() { ... } }
class ReportService { void generateReport() { ... } }
```

### O — Open/Closed Principle

```
"Open for EXTENSION, closed for MODIFICATION."
Add new features by adding new code, NOT changing existing code.

❌ BAD: Adding new shape requires modifying AreaCalculator
class AreaCalculator {
    double area(Object shape) {
        if (shape instanceof Circle) return π * r²;
        if (shape instanceof Square) return s²;
        // Must modify this class for every new shape!
    }
}

✅ GOOD: Each shape handles its own area
interface Shape { double area(); }
class Circle implements Shape { double area() { return π * r²; } }
class Square implements Shape { double area() { return s²; } }
// New shape? Just add a new class. No existing code changes!
```

### L — Liskov Substitution Principle

```
"Subtypes must be substitutable for their base types without breaking the program."

❌ BAD: Square extends Rectangle but breaks setWidth/setHeight contract
class Rectangle { 
    void setWidth(int w); 
    void setHeight(int h); 
}
class Square extends Rectangle {
    void setWidth(int w) { width = w; height = w; }  // Changes BOTH!
    // Breaks: rect.setWidth(5); rect.setHeight(10); area should be 50, but Square gives 100!
}

✅ GOOD: Use separate classes or a common Shape interface
interface Shape { double area(); }
```

### I — Interface Segregation Principle

```
"Clients should not be forced to depend on interfaces they don't use."

❌ BAD: One fat interface
interface Worker {
    void work();
    void eat();      // Robots don't eat!
    void sleep();    // Robots don't sleep!
}
class Robot implements Worker { 
    void eat() { throw new UnsupportedOperationException(); } // Forced to implement!
}

✅ GOOD: Split into smaller interfaces
interface Workable { void work(); }
interface Eatable { void eat(); }
interface Sleepable { void sleep(); }
class Robot implements Workable { void work() { ... } } // Only what it needs!
class Human implements Workable, Eatable, Sleepable { ... }
```

### D — Dependency Inversion Principle

```
"High-level modules should NOT depend on low-level modules. Both should depend on abstractions."

❌ BAD: Controller directly depends on MySQLDatabase
class OrderController {
    private MySQLDatabase db = new MySQLDatabase(); // Tightly coupled!
}

✅ GOOD: Depend on interface, inject implementation
class OrderController {
    private Database db; // Interface
    OrderController(Database db) { this.db = db; } // Injected!
}
// Now can swap MySQL for PostgreSQL without changing OrderController!
// This is exactly what Spring's @Autowired does!
```

---

## 8. Design Patterns (Gang of Four)

### Creational Patterns (How objects are created)

```
┌────────────────┬──────────────────────────────────────────────────────┐
│ Pattern        │ When to Use                                          │
├────────────────┼──────────────────────────────────────────────────────┤
│ Singleton      │ Only ONE instance (DB connection pool, logger)       │
│ Factory        │ Create objects without specifying exact class        │
│ Abstract       │ Create families of related objects                   │
│  Factory       │                                                      │
│ Builder        │ Complex object with many optional params             │
│ Prototype      │ Clone existing object instead of creating from scratch│
└────────────────┴──────────────────────────────────────────────────────┘
```

```java
// ═══ SINGLETON ═══
public class DatabaseConnection {
    private static volatile DatabaseConnection instance;
    private DatabaseConnection() {} // Private constructor
    
    public static DatabaseConnection getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                if (instance == null) instance = new DatabaseConnection();
            }
        }
        return instance;
    }
}
// Spring: Every @Bean is Singleton by default!

// ═══ FACTORY ═══
interface Notification { void send(String message); }
class EmailNotification implements Notification { ... }
class SMSNotification implements Notification { ... }
class PushNotification implements Notification { ... }

class NotificationFactory {
    static Notification create(String type) {
        return switch (type) {
            case "email" -> new EmailNotification();
            case "sms" -> new SMSNotification();
            case "push" -> new PushNotification();
            default -> throw new IllegalArgumentException("Unknown: " + type);
        };
    }
}

// ═══ BUILDER ═══
User user = User.builder()
    .name("Dilip")
    .email("dilip@example.com")
    .age(25)
    .city("Bangalore")  // Optional
    .build();
// Spring: WebClient.builder(), RestClient.builder()

// ═══ OBSERVER ═══ (Pub/Sub pattern)
// Spring: @EventListener, ApplicationEventPublisher
@Component
public class OrderEventListener {
    @EventListener
    public void onOrderPlaced(OrderPlacedEvent event) {
        // React to order being placed
        emailService.sendConfirmation(event.getOrderId());
    }
}
```

### Structural Patterns (How objects are composed)

```
┌────────────────┬──────────────────────────────────────────────────────┐
│ Adapter        │ Make incompatible interfaces work together           │
│ Decorator      │ Add behavior dynamically (wrapping)                 │
│ Proxy          │ Control access to an object                          │
│ Facade         │ Simple interface to complex subsystem               │
│ Composite      │ Tree structure (folder → files/subfolders)          │
└────────────────┴──────────────────────────────────────────────────────┘
```

```java
// ═══ PROXY ═══ (Spring AOP = Proxy pattern!)
// @Transactional creates a proxy around your service:
// Proxy intercepts call → starts transaction → calls your method → commits/rollbacks

// ═══ DECORATOR ═══ (Java I/O streams)
InputStream is = new BufferedInputStream(     // Decorator: adds buffering
                    new FileInputStream("file.txt")); // Base: reads file

// ═══ FACADE ═══
// Your OrderService is a facade:
class OrderService {
    void placeOrder(OrderRequest req) {
        inventoryService.reserve(req);       // Complex subsystem 1
        paymentService.charge(req);          // Complex subsystem 2
        notificationService.notify(req);     // Complex subsystem 3
        shippingService.schedule(req);       // Complex subsystem 4
    }
}
// Caller just does orderService.placeOrder() — doesn't know internals!
```

### Behavioral Patterns (How objects communicate)

```
┌────────────────┬──────────────────────────────────────────────────────┐
│ Observer       │ Notify multiple objects of state changes (events)    │
│ Strategy       │ Swap algorithms at runtime                           │
│ Template Method│ Define skeleton, let subclasses fill steps           │
│ Command        │ Encapsulate request as object (undo/redo)           │
│ Iterator       │ Access elements sequentially without exposing struct │
│ State          │ Object behavior changes with state (State Machine)  │
│ Chain of Resp. │ Pass request along a chain of handlers              │
└────────────────┴──────────────────────────────────────────────────────┘
```

```java
// ═══ STRATEGY ═══ (Most asked in interviews!)
interface PaymentStrategy { void pay(int amount); }
class CreditCardPayment implements PaymentStrategy { ... }
class UPIPayment implements PaymentStrategy { ... }
class WalletPayment implements PaymentStrategy { ... }

class ShoppingCart {
    void checkout(PaymentStrategy strategy) {
        strategy.pay(totalAmount); // Strategy is swapped at runtime!
    }
}

cart.checkout(new UPIPayment());        // Pay via UPI
cart.checkout(new CreditCardPayment()); // Pay via Card
// Spring: Comparator in Collections.sort() is Strategy pattern!

// ═══ CHAIN OF RESPONSIBILITY ═══
// Spring Security filter chain:
// Request → AuthFilter → CorsFilter → RateLimitFilter → Controller
// Each filter decides: process & pass, or reject
```

---

## 9. Architecture Patterns

### Monolith vs Microservices vs Serverless

```
MONOLITH:
  ┌──────────────────────────────┐
  │     ONE BIG APPLICATION       │
  │  ┌─────┐ ┌─────┐ ┌───────┐  │
  │  │Users│ │Order│ │Payment│  │    Single deployment unit
  │  └─────┘ └─────┘ └───────┘  │    One database
  │  ┌──────┐ ┌────────────┐    │    Easy to develop initially
  │  │Search│ │Notification│    │    Hard to scale one part
  │  └──────┘ └────────────┘    │
  └──────────────────────────────┘

MICROSERVICES:
  ┌───────┐  ┌───────┐  ┌─────────┐
  │ User  │  │ Order │  │ Payment │    Each service = own deployment
  │Service│  │Service│  │ Service │    Own database (DB per service)
  │  DB1  │  │  DB2  │  │   DB3   │    Independent scaling
  └───┬───┘  └───┬───┘  └────┬────┘    Complex infrastructure
      └──────────┴───────────┘
           API Gateway / Message Queue

SERVERLESS:
  [Function A] ←── API Gateway trigger
  [Function B] ←── S3 upload trigger         No servers to manage
  [Function C] ←── Scheduled trigger         Pay per execution
  [Function D] ←── Queue message trigger     Auto-scales to zero
```

| Aspect | Monolith | Microservices | Serverless |
|:---|:---|:---|:---|
| **Complexity** | Low | High | Medium |
| **Scaling** | Scale everything | Scale per service | Auto-scale |
| **Deployment** | All or nothing | Independent | Per function |
| **Cost (small)** | Low | High (infra overhead) | Very low |
| **Cost (large)** | Moderate | Optimized | Can get expensive |
| **Best For** | Startups, MVPs | Large teams, complex products | Event-driven, variable load |

### MVC (Model-View-Controller)

```
┌──────────┐     ┌──────────────┐     ┌──────────┐
│   VIEW   │◀───▶│  CONTROLLER  │◀───▶│  MODEL   │
│ (HTML/UI)│     │ (Logic/Route)│     │ (Data/DB)│
└──────────┘     └──────────────┘     └──────────┘

Spring MVC:
  View       = Thymeleaf / React / HTML
  Controller = @RestController
  Model      = @Entity + @Service + @Repository
```

### Layered Architecture (N-Tier)

```
┌─────────────────────────────────┐
│      Presentation Layer          │  Controllers, DTOs
├─────────────────────────────────┤
│       Business Logic Layer       │  Services, validation
├─────────────────────────────────┤
│       Data Access Layer          │  Repositories, DAOs
├─────────────────────────────────┤
│         Database Layer           │  MySQL, PostgreSQL
└─────────────────────────────────┘

Spring Boot follows this:
  @Controller / @RestController  →  Presentation
  @Service                       →  Business Logic
  @Repository                    →  Data Access
  @Entity                        →  Database mapping
```

---

## 10. UML Diagrams

### Use Case Diagram

```
                ┌─────────────────────────┐
                │    E-Commerce System      │
                │                           │
  ┌──────┐      │  (Browse Products)       │
  │      │──────│  (Add to Cart)           │
  │ User │──────│  (Place Order)           │      ┌───────┐
  │      │──────│  (Make Payment) ─────────│──────│Payment │
  └──────┘      │  (Track Order)           │      │Gateway │
                │                           │      └───────┘
  ┌──────┐      │  (Manage Products)       │
  │Admin │──────│  (View Reports)          │
  └──────┘      │  (Manage Users)          │
                └─────────────────────────┘
```

### Class Diagram (Relationships)

```
┌──────────────┐        ┌──────────────┐
│    Animal     │        │   Flyable    │  ← Interface
│──────────────│        │──────────────│
│ - name       │        │ + fly()      │
│ - age        │        └──────┬───────┘
│──────────────│               │ implements
│ + eat()      │        ┌──────┴───────┐
│ + sleep()    │        │     Bird     │
└──────┬───────┘        │──────────────│
       │ extends        │ + fly()      │
┌──────┴───────┐        │ + sing()     │
│     Dog      │        └──────────────┘
│──────────────│
│ + bark()     │
└──────────────┘

Relationships:
  ──────▶  Association (has-a, uses)
  ◆──────▶ Composition (part-of, lifecycle dependent: Car ◆── Engine)
  ◇──────▶ Aggregation (has-a, independent lifecycle: Dept ◇── Employee)
  ─ ─ ─ ▶ Dependency (uses temporarily)
  ────────▷ Inheritance (extends)
  - - - - ▷ Implementation (implements interface)
```

### Sequence Diagram

```
  User          Controller       Service        Repository       Database
   │                │               │               │               │
   │──POST /order──▶│               │               │               │
   │                │──createOrder─▶│               │               │
   │                │               │──save(order)─▶│               │
   │                │               │               │──INSERT──────▶│
   │                │               │               │◀──── OK ──────│
   │                │               │◀── order ─────│               │
   │                │◀──OrderDTO────│               │               │
   │◀── 201 JSON ───│               │               │               │
```

---

## 11. Testing Types & Strategies

### Testing Pyramid

```
            ╱  ╲
           ╱ UI ╲               Few (slow, expensive, brittle)
          ╱ Tests╲              E2E / Selenium
         ╱────────╲
        ╱Integration╲          Some (medium speed)
       ╱   Tests     ╲         API tests, DB tests
      ╱────────────────╲
     ╱   Unit Tests     ╲      Many (fast, cheap, reliable)
    ╱                    ╲     JUnit, Mockito
   ╱──────────────────────╲
```

### Types of Testing

| Type | What | Tool | Level |
|:---|:---|:---|:---|
| **Unit** | Test single method/class in isolation | JUnit, Mockito | Developer |
| **Integration** | Test components working together | Spring Test, TestContainers | Developer |
| **E2E** | Test full user flow | Selenium, Cypress, Playwright | QA |
| **Performance** | Test speed and load handling | JMeter, Gatling, k6 | DevOps |
| **Regression** | Ensure old features still work | Automated test suite | CI/CD |
| **Smoke** | Basic checks after deployment | Quick API calls | Post-deploy |
| **Acceptance** | Does it meet business requirements? | Cucumber (BDD) | Business |
| **Security** | Find vulnerabilities | OWASP ZAP, Snyk | Security |
| **Mutation** | Test quality of your tests | PITest | Developer |

### Testing Best Practices

```
1. AAA Pattern:
   Arrange → Set up test data
   Act     → Execute the method
   Assert  → Verify the result

2. FIRST:
   Fast      → Tests run in milliseconds
   Isolated  → No dependency between tests
   Repeatable→ Same result every time
   Self-validating → Pass or fail, no manual check
   Timely    → Write tests BEFORE or WITH code (not after)

3. Test Coverage:
   80% is generally good enough
   100% is a waste (diminishing returns)
   Focus on: critical paths, edge cases, business logic
```

---

## 12. Version Control (Git Workflows)

### Git Flow

```
main (production) ──●──────────────────────●──────────────── (releases)
                    │                      ▲
develop ────────●───┼──●──●──●─────●───────┤──●──●── (integration)
                │   │  ▲     │     ▲       │
feature/login ──┘   │  │     │     │       │
feature/cart ───────┘  │     │     │       │
                       │     │     │       │
hotfix/critical ───────┘     │     │       │
release/v1.0 ────────────────┘     │       │
release/v1.1 ──────────────────────┘       │
```

### GitHub Flow (Simpler — Most companies use this)

```
main ──────●──────────────●───────────── (always deployable)
           │              ▲
           │    PR + Review + Merge
           │              │
feature ───┴──●──●──●─────┘

1. Branch from main
2. Add commits
3. Open Pull Request
4. Code Review
5. Merge to main
6. Deploy
```

### Trunk-Based Development (Google/Netflix style)

```
main ──●──●──●──●──●──●──●──●──●──── (everyone commits to main)
       │     │     │     │
       ▼     ▼     ▼     ▼
    Deploy Deploy Deploy Deploy (multiple times per day)

Rules:
- Short-lived branches (< 1 day)
- Feature flags instead of long-lived branches
- Everyone commits to main/trunk
```

---

## 13. Code Review Best Practices

```
WHAT TO CHECK:
  ✅ Correctness — Does it actually work?
  ✅ Readability — Can someone else understand this in 6 months?
  ✅ Performance — Any O(N²) where O(N) is possible?
  ✅ Security — SQL injection? XSS? Auth bypass?
  ✅ Error handling — What if this API call fails?
  ✅ Tests — Are there adequate test cases?
  ✅ Edge cases — Empty list? Null input? Max integer?
  ✅ Naming — Are variables/methods descriptive?
  ✅ DRY — Is code duplicated anywhere?
  ✅ SOLID — Any principle violations?

HOW TO GIVE FEEDBACK:
  ❌ "This code is bad."
  ✅ "Consider using a HashMap here instead of nested loops — it would reduce 
      time complexity from O(n²) to O(n)."

  ❌ "Why did you do it this way?"
  ✅ "I'm curious about this approach. Have you considered using the Strategy 
      pattern here? It might make it easier to add new payment methods later."
```

---

## 14. Estimation Techniques

| Technique | How | Best For |
|:---|:---|:---|
| **Story Points** | Fibonacci (1,2,3,5,8,13) — relative effort | Agile teams |
| **Planning Poker** | Team votes with cards, discuss outliers | Sprint planning |
| **T-Shirt Sizing** | S, M, L, XL — quick rough estimate | Roadmap planning |
| **Three-Point** | (Optimistic + 4×Most Likely + Pessimistic) / 6 | Risk-aware estimation |
| **Function Points** | Count inputs, outputs, queries, files | Large waterfall projects |
| **COCOMO** | Formula-based (lines of code estimate) | Academic / government |

---

## 15. Requirement Engineering

### Types of Requirements

```
FUNCTIONAL Requirements (WHAT the system does):
  - User can register with email and password
  - System sends OTP for verification
  - Admin can view all orders with filters
  - Payment must support UPI, Card, and Net Banking

NON-FUNCTIONAL Requirements (HOW the system performs):
  - Response time < 200ms for 95th percentile
  - System handles 10,000 concurrent users
  - 99.9% uptime (max 8.7 hours downtime/year)
  - Data encrypted at rest and in transit
  - System recovers within 5 minutes of failure
  
  Categories: Performance, Scalability, Security, Reliability, 
              Usability, Maintainability, Portability
```

---

## 16. Software Quality & Metrics

### Key Metrics

| Metric | What | Good Target |
|:---|:---|:---|
| **Code Coverage** | % of code tested | 80%+ |
| **Cyclomatic Complexity** | Number of independent paths | < 10 per method |
| **Technical Debt** | Cost to fix shortcuts | Track and reduce |
| **Defect Density** | Bugs per 1000 lines | < 1 |
| **MTTR** | Mean Time To Recovery | < 1 hour |
| **MTBF** | Mean Time Between Failures | > 720 hours |
| **Lead Time** | Commit to production | < 1 day |
| **Deployment Frequency** | How often you deploy | Daily+ |
| **Change Failure Rate** | % of deploys causing incidents | < 15% |

### DORA Metrics (Google's DevOps Research)

```
4 Key Metrics that separate Elite teams from Low performers:

                    Elite        Low
Deployment Freq:    Multiple/day  < Monthly
Lead Time:          < 1 hour      > 6 months
Change Failure:     0-15%         46-60%
MTTR:               < 1 hour      > 6 months
```

---

## 17. Risk Management

```
Risk = Probability × Impact

Risk Matrix:
                    Low Impact    Medium Impact   High Impact
High Probability    Medium Risk   High Risk       CRITICAL ⚠️
Medium Probability  Low Risk      Medium Risk     High Risk
Low Probability     Negligible    Low Risk        Medium Risk

Risk Response Strategies:
  AVOID    → Change plan to eliminate the risk
  MITIGATE → Reduce probability or impact
  TRANSFER → Shift risk to third party (insurance, SLA)
  ACCEPT   → Acknowledge and monitor
```

---

## 18. Coupling & Cohesion

```
COUPLING: How much one module depends on another
  Tight Coupling (BAD):  Class A directly creates Class B, calls internals
  Loose Coupling (GOOD): Class A depends on interface, B is injected

  ❌ class OrderService { private MySQLDB db = new MySQLDB(); }
  ✅ class OrderService { private Database db; OrderService(Database db) {...} }

COHESION: How related are the responsibilities within a module
  Low Cohesion (BAD):  UserService does: create user + send email + generate PDF
  High Cohesion (GOOD): UserService does: ONLY user CRUD operations

  Goal: HIGH Cohesion + LOW Coupling
  = Each module does ONE thing well, with minimal dependencies
```

---

## 19. API Design Principles

### REST API Best Practices

```
✅ Use nouns, not verbs:     GET /users (not GET /getUsers)
✅ Use plural nouns:          /users (not /user)
✅ Use proper HTTP methods:   GET (read), POST (create), PUT (update), DELETE (delete)
✅ Use HTTP status codes:     200 OK, 201 Created, 400 Bad Request, 404 Not Found
✅ Version your API:          /api/v1/users
✅ Use pagination:            /users?page=1&size=20
✅ Use filtering:             /users?status=active&city=mumbai
✅ Use HATEOAS links:         Include links to related resources
✅ Return consistent format:  Always { "data": ..., "error": ..., "meta": ... }
✅ Use proper error response: { "error": "Validation failed", "details": [...] }
```

### API Versioning Strategies

| Strategy | Example | Pros | Cons |
|:---|:---|:---|:---|
| **URL Path** | `/api/v1/users` | Simple, visible | URL changes |
| **Header** | `Accept: application/vnd.api.v1+json` | Clean URLs | Hidden |
| **Query Param** | `/api/users?version=1` | Easy to use | Cluttered |

---

## 20. 12-Factor App

```
The 12-Factor methodology for building modern, scalable apps:

 1. CODEBASE        One codebase in version control, many deploys
 2. DEPENDENCIES    Explicitly declare and isolate (pom.xml / build.gradle)
 3. CONFIG          Store in environment variables (not in code!)
 4. BACKING SERVICES Treat DBs, queues, caches as attached resources
 5. BUILD, RELEASE, RUN  Strictly separate these stages
 6. PROCESSES       Stateless processes (no sticky sessions)
 7. PORT BINDING    Export services via port binding (embedded Tomcat)
 8. CONCURRENCY     Scale out via process model (horizontal scaling)
 9. DISPOSABILITY   Fast startup, graceful shutdown
10. DEV/PROD PARITY Keep dev, staging, production as similar as possible
11. LOGS            Treat logs as event streams (stdout, not files)
12. ADMIN PROCESSES Run admin/management tasks as one-off processes

Spring Boot follows most of these naturally!
  Factor 2: Maven/Gradle
  Factor 3: application.yml + @Value + env vars
  Factor 4: Spring Data, Spring AMQP, Spring Cache
  Factor 7: Embedded Tomcat on port 8080
  Factor 9: Fast startup, @PreDestroy for graceful shutdown
  Factor 11: Logback to stdout (Docker captures it)
```

---

## 21. CAP Theorem & Distributed Systems

### CAP Theorem

```
In a distributed system, you can only guarantee 2 out of 3:

     Consistency
       /     \
      /       \
     /   PICK  \
    /    TWO    \
   /             \
Availability ─── Partition
                 Tolerance

C = Consistency:         Every read gets the most recent write
A = Availability:        Every request gets a response (even if stale)
P = Partition Tolerance:  System works even if network fails between nodes

Real-world: P is NOT optional (networks WILL fail)
So you choose: CP or AP

CP (Consistency + Partition):  Banking, inventory
   → If network fails, system refuses requests (to maintain consistency)
   → Examples: MongoDB (configurable), HBase, Redis Cluster

AP (Availability + Partition): Social media feeds, DNS
   → If network fails, system returns possibly stale data
   → Examples: Cassandra, DynamoDB, CouchDB
```

### BASE vs ACID

| ACID (Traditional DB) | BASE (Distributed Systems) |
|:---|:---|
| **A**tomicity — all or nothing | **B**asically **A**vailable — always responds |
| **C**onsistency — valid state always | **S**oft state — may be temporarily inconsistent |
| **I**solation — transactions don't interfere | **E**ventually consistent — will converge |
| **D**urability — committed = permanent | |
| MySQL, PostgreSQL | Cassandra, DynamoDB |

---

## 22. Interview Questions (50+)

### SDLC & Methodologies

| # | Question | Answer |
|:---:|:---|:---|
| 1 | What is SDLC? | Structured process: Planning → Requirements → Design → Code → Test → Deploy → Maintain |
| 2 | Waterfall vs Agile? | Waterfall: sequential, fixed reqs, late testing. Agile: iterative, flexible, continuous feedback. |
| 3 | What is Scrum? | Agile framework: sprints (2wk), daily standups, PO, SM, Dev Team, backlog, retrospective. |
| 4 | Sprint vs Kanban? | Sprint: fixed time-box. Kanban: continuous flow with WIP limits. |
| 5 | What is velocity? | Story points completed per sprint. Used for forecasting. |
| 6 | What is a burndown chart? | Graph showing remaining work vs time in a sprint. |
| 7 | Definition of Done? | Agreed checklist: coded + tested + reviewed + documented + deployed. |
| 8 | What is technical debt? | Shortcuts taken now that cost more to fix later. Manage it, don't eliminate it. |
| 9 | What are story points? | Relative effort measure using Fibonacci (1,2,3,5,8,13). Not hours! |
| 10 | Explain CI/CD | CI: auto build+test on commit. CD: auto deploy to staging/prod. |

### Design Principles & Patterns

| # | Question | Answer |
|:---:|:---|:---|
| 11 | Explain SOLID principles | S: single responsibility. O: open/closed. L: Liskov substitution. I: interface segregation. D: dependency inversion. |
| 12 | What is Singleton? | Only one instance of a class. Spring @Bean is singleton by default. |
| 13 | Factory vs Builder? | Factory: create different types. Builder: construct complex object step by step. |
| 14 | What is Strategy pattern? | Swap algorithms at runtime. Example: different payment methods. |
| 15 | What is Observer pattern? | Publish-subscribe. Spring's @EventListener. RabbitMQ is a distributed observer. |
| 16 | DRY vs KISS vs YAGNI? | DRY: Don't Repeat Yourself. KISS: Keep It Simple. YAGNI: You Aren't Gonna Need It. |
| 17 | What is coupling vs cohesion? | Coupling: inter-module dependency (low = good). Cohesion: intra-module relatedness (high = good). |
| 18 | Explain MVC | Model (data), View (UI), Controller (logic). Spring MVC follows this. |
| 19 | Monolith vs Microservices? | Monolith: one deployment. Microservices: independent services, own DBs, complex infra. |
| 20 | When to choose microservices? | Large team (>20), need independent scaling, polyglot tech stack, clear domain boundaries. |

### Testing

| # | Question | Answer |
|:---:|:---|:---|
| 21 | Unit vs Integration vs E2E? | Unit: single method. Integration: components together. E2E: full user flow. |
| 22 | What is mocking? | Replace real dependencies with fake ones (Mockito). Test class in isolation. |
| 23 | What is TDD? | Test-Driven Development: Write test first → Write code → Refactor. Red-Green-Refactor. |
| 24 | What is BDD? | Behavior-Driven Development: Given-When-Then format. Cucumber framework. |
| 25 | What is regression testing? | Re-running tests to ensure new changes don't break existing features. |
| 26 | What code coverage is ideal? | 80% is good. 100% is diminishing returns. Focus on critical paths. |
| 27 | What is the testing pyramid? | Many unit tests, some integration, few E2E. Fast base, slow top. |

### Version Control & DevOps

| # | Question | Answer |
|:---:|:---|:---|
| 28 | Git rebase vs merge? | Merge: creates merge commit, preserves history. Rebase: linear history, rewrites commits. |
| 29 | What is GitFlow? | Branching model: main, develop, feature/*, release/*, hotfix/* branches. |
| 30 | What is trunk-based development? | Everyone commits to main. Short-lived branches. Feature flags. |
| 31 | What is a feature flag? | Toggle features on/off without deploying. Gradual rollout (1% → 10% → 100%). |
| 32 | Blue-Green deployment? | Two identical envs. Route traffic from Blue to Green. Instant rollback. |
| 33 | Canary deployment? | Deploy to 5% of users first. Monitor. If OK, roll to 100%. |
| 34 | What is Infrastructure as Code? | Manage infra via code files (Terraform, CloudFormation). Version controlled, repeatable. |

### Architecture & System Design

| # | Question | Answer |
|:---:|:---|:---|
| 35 | What is CAP theorem? | Distributed systems can have 2 of 3: Consistency, Availability, Partition tolerance. |
| 36 | ACID vs BASE? | ACID: strong consistency (SQL). BASE: eventually consistent (NoSQL). |
| 37 | What is 12-Factor App? | 12 principles for modern apps: env config, stateless, disposable, dev/prod parity, etc. |
| 38 | Horizontal vs Vertical scaling? | Horizontal: add more machines. Vertical: add more RAM/CPU to one machine. |
| 39 | What is load balancing? | Distribute traffic across servers. Round robin, least connections, weighted. |
| 40 | What is caching? | Store frequently accessed data in fast storage (Redis). Cache-aside, write-through. |
| 41 | SQL vs NoSQL? | SQL: structured, ACID, joins. NoSQL: flexible schema, horizontal scaling, eventual consistency. |
| 42 | What is eventual consistency? | Data will be consistent eventually (not immediately). Used in distributed systems. |

### General SE Concepts

| # | Question | Answer |
|:---:|:---|:---|
| 43 | What is refactoring? | Restructuring code without changing behavior. Improve readability, reduce complexity. |
| 44 | What is code smell? | Signs of deeper problems: long methods, god class, duplicate code, magic numbers. |
| 45 | What is API versioning? | Support multiple API versions. URL path (/v1, /v2), header, or query param. |
| 46 | What is idempotency? | Same request called multiple times produces same result. Critical for payments, APIs. |
| 47 | What is SLA vs SLO vs SLI? | SLA: contract (99.9% uptime). SLO: target (99.95%). SLI: actual measurement (99.97%). |
| 48 | Functional vs Non-functional reqs? | Functional: what system does (features). Non-functional: how system performs (speed, security). |
| 49 | What is a design review? | Team reviews architecture/design before coding. Catch issues early. |
| 50 | What is the difference between concurrency and parallelism? | Concurrency: managing multiple tasks (interleaving). Parallelism: executing multiple tasks simultaneously (multi-core). |
