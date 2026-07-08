# ☕ Core Java — Complete In-Depth Guide (Beginner to Expert)

> **"Java is not just a language; it's a platform, an ecosystem, and a philosophy of writing robust, portable, and scalable software."**

---

## 📑 Table of Contents

1. [Introduction & History](#1-introduction--history)
2. [JVM, JRE, and JDK Architecture](#2-jvm-jre-and-jdk-architecture)
3. [Data Types & Variables](#3-data-types--variables)
4. [Operators & Control Flow](#4-operators--control-flow)
5. [Object-Oriented Programming (OOP)](#5-object-oriented-programming-oop)
6. [Constructors & Initialization Blocks](#6-constructors--initialization-blocks)
7. [Strings & String Handling](#7-strings--string-handling)
8. [Arrays & Varargs](#8-arrays--varargs)
9. [Exception Handling](#9-exception-handling)
10. [Collections Framework](#10-collections-framework)
11. [Generics](#11-generics)
12. [Lambda Expressions & Functional Interfaces](#12-lambda-expressions--functional-interfaces)
13. [Streams API](#13-streams-api)
14. [Multithreading & Concurrency](#14-multithreading--concurrency)
15. [Java I/O & NIO](#15-java-io--nio)
16. [Java Memory Model & Garbage Collection](#16-java-memory-model--garbage-collection)
17. [Java 8–21 Feature Evolution](#17-java-821-feature-evolution)
18. [Sealed Classes, Records, and Pattern Matching](#18-sealed-classes-records-and-pattern-matching)
19. [Best Practices & Code Conventions](#19-best-practices--code-conventions)
20. [Common Mistakes & Pitfalls](#20-common-mistakes--pitfalls)
21. [Interview Questions & Answers (50+)](#21-interview-questions--answers-50)

---

## 1. Introduction & History

### What is Java?

Java is a **high-level, class-based, object-oriented programming language** designed to have as few implementation dependencies as possible. It follows the principle of **WORA** — Write Once, Run Anywhere — meaning compiled Java code can run on all platforms that support Java without the need for recompilation.

### Brief History

| Year | Milestone |
|------|-----------|
| 1991 | James Gosling starts "Green Project" at Sun Microsystems |
| 1995 | Java 1.0 released publicly |
| 1998 | Java 2 (J2SE 1.2) — Collections Framework introduced |
| 2004 | Java 5 — Generics, Annotations, Enums, Autoboxing |
| 2006 | Java becomes open-source (OpenJDK) |
| 2014 | Java 8 — Lambdas, Streams, Optional (GAME CHANGER) |
| 2017 | Java 9 — Modules (Project Jigsaw) |
| 2018 | 6-month release cadence begins |
| 2021 | Java 17 LTS — Sealed classes, pattern matching |
| 2023 | Java 21 LTS — Virtual threads, record patterns, sequenced collections |
| 2025 | Java 25 LTS — Latest long-term support |

### Why Java for Spring Boot?

- **Mature ecosystem** — Thousands of battle-tested libraries
- **Strong typing** — Catches errors at compile time
- **Enterprise adoption** — Banks, healthcare, government all use Java
- **Spring Boot is Java-first** — Designed to leverage Java's strengths
- **Backward compatibility** — Code from Java 8 still runs on Java 21

---

## 2. JVM, JRE, and JDK Architecture

### The Java Platform Stack

```
┌──────────────────────────────────────────┐
│                  JDK                      │
│  ┌───────────────────────────────────┐   │
│  │              JRE                   │   │
│  │  ┌────────────────────────────┐   │   │
│  │  │           JVM              │   │   │
│  │  │  ┌──────────────────────┐  │   │   │
│  │  │  │   Class Loader       │  │   │   │
│  │  │  │   Bytecode Verifier  │  │   │   │
│  │  │  │   Execution Engine   │  │   │   │
│  │  │  │   ├─ Interpreter     │  │   │   │
│  │  │  │   ├─ JIT Compiler    │  │   │   │
│  │  │  │   └─ GC              │  │   │   │
│  │  │  │   Runtime Data Areas │  │   │   │
│  │  │  │   ├─ Method Area     │  │   │   │
│  │  │  │   ├─ Heap            │  │   │   │
│  │  │  │   ├─ Stack           │  │   │   │
│  │  │  │   ├─ PC Register     │  │   │   │
│  │  │  │   └─ Native Method   │  │   │   │
│  │  │  └──────────────────────┘  │   │   │
│  │  └────────────────────────────┘   │   │
│  │  + Java Standard Libraries        │   │
│  └───────────────────────────────────┘   │
│  + Development Tools (javac, jar, etc.)  │
└──────────────────────────────────────────┘
```

### JVM (Java Virtual Machine)

The JVM is an **abstract computing machine** that enables a computer to run Java programs. It does three main things:

1. **Loads** bytecode (Class Loader)
2. **Verifies** bytecode (Bytecode Verifier)
3. **Executes** bytecode (Execution Engine)

#### Class Loading Process

```
Loading → Linking (Verify → Prepare → Resolve) → Initialization
```

**Three built-in class loaders:**

```java
// Bootstrap ClassLoader - loads rt.jar (core Java classes)
// Extension ClassLoader - loads from jre/lib/ext
// Application ClassLoader - loads from classpath

public class ClassLoaderDemo {
    public static void main(String[] args) {
        // Application ClassLoader
        System.out.println(ClassLoaderDemo.class.getClassLoader());
        // sun.misc.Launcher$AppClassLoader

        // Extension ClassLoader
        System.out.println(ClassLoaderDemo.class.getClassLoader().getParent());
        // sun.misc.Launcher$ExtClassLoader

        // Bootstrap ClassLoader (returns null because it's native)
        System.out.println(String.class.getClassLoader());
        // null
    }
}
```

#### JVM Memory Areas

| Area | Purpose | Thread-Shared? |
|------|---------|---------------|
| **Method Area** | Stores class metadata, static variables, constant pool | Yes |
| **Heap** | Object instances and arrays | Yes |
| **Stack** | Local variables, method calls, partial results | No (per thread) |
| **PC Register** | Address of current instruction | No (per thread) |
| **Native Method Stack** | Native method information | No (per thread) |

### JRE (Java Runtime Environment)

JRE = JVM + Standard Libraries (java.lang, java.util, java.io, etc.)

It is what **end users** need to run Java applications. It does NOT include development tools.

### JDK (Java Development Kit)

JDK = JRE + Development Tools (javac, jar, javadoc, jdb, etc.)

It is what **developers** need to write, compile, and debug Java programs.

```bash
# Check your JDK version
java -version
javac -version

# Compile a Java file
javac HelloWorld.java

# Run the compiled class
java HelloWorld

# Create a JAR file
jar cf myapp.jar *.class
```

---

## 3. Data Types & Variables

### Primitive Data Types

Java has **8 primitive data types**:

| Type | Size | Default | Range | Example |
|------|------|---------|-------|---------|
| `byte` | 1 byte | 0 | -128 to 127 | `byte b = 100;` |
| `short` | 2 bytes | 0 | -32,768 to 32,767 | `short s = 10000;` |
| `int` | 4 bytes | 0 | -2^31 to 2^31-1 | `int i = 100000;` |
| `long` | 8 bytes | 0L | -2^63 to 2^63-1 | `long l = 100000L;` |
| `float` | 4 bytes | 0.0f | ~7 decimal digits | `float f = 3.14f;` |
| `double` | 8 bytes | 0.0d | ~16 decimal digits | `double d = 3.14159;` |
| `char` | 2 bytes | '\u0000' | 0 to 65,535 | `char c = 'A';` |
| `boolean` | ~1 bit | false | true or false | `boolean b = true;` |

### Type Casting

```java
// Widening (Implicit) — No data loss
int myInt = 100;
long myLong = myInt;        // int → long (automatic)
double myDouble = myLong;    // long → double (automatic)

// Narrowing (Explicit) — Potential data loss
double d = 9.78;
int i = (int) d;            // d becomes 9 (truncated, not rounded!)

// Overflow example
byte b = (byte) 130;        // b becomes -126 (wraps around!)
```

### Variable Types

```java
public class VariableTypes {

    // 1. Instance Variables (Non-Static Fields)
    // - Belong to an instance of the class
    // - Created when object is created, destroyed when object is garbage collected
    // - Have default values
    private String name;
    private int age;

    // 2. Class Variables (Static Fields)
    // - Belong to the class itself, shared among all instances
    // - Created when class is loaded, destroyed when class is unloaded
    static int instanceCount = 0;

    // 3. Local Variables
    // - Declared inside methods, constructors, or blocks
    // - NO default values — must be initialized before use
    // - Exist only within the enclosing block
    public void greet() {
        String greeting = "Hello!"; // local variable
        System.out.println(greeting);
    }

    // 4. Parameters
    // - Variables in method declarations
    public void setName(String name) { // 'name' is a parameter
        this.name = name;
    }
}
```

### `var` (Local Variable Type Inference — Java 10+)

```java
// The compiler infers the type from the right-hand side
var name = "Dilip";           // Inferred as String
var numbers = List.of(1, 2, 3); // Inferred as List<Integer>
var map = new HashMap<String, List<Integer>>(); // Much cleaner!

// ❌ Cannot use var for:
// var x;                    // No initializer
// var x = null;             // Cannot infer type from null
// var x = {1, 2, 3};       // Array initializer needs explicit type
// var x = () -> "hello";   // Lambda needs target type
```

---

## 4. Operators & Control Flow

### Operators

```java
// Arithmetic: + - * / %
int result = 10 % 3;    // 1 (modulus/remainder)

// Comparison: == != > < >= <=
boolean isEqual = (5 == 5);  // true

// Logical: && || !
boolean both = (true && false); // false
boolean either = (true || false); // true

// Bitwise: & | ^ ~ << >> >>>
int a = 5;           // 0101 in binary
int b = 3;           // 0011 in binary
int and = a & b;     // 0001 = 1
int or = a | b;      // 0111 = 7
int xor = a ^ b;     // 0110 = 6
int leftShift = a << 1;  // 1010 = 10 (multiply by 2)
int rightShift = a >> 1; // 0010 = 2 (divide by 2)

// Ternary: condition ? valueIfTrue : valueIfFalse
String status = (age >= 18) ? "Adult" : "Minor";

// instanceof (with Pattern Matching — Java 16+)
if (obj instanceof String s) {
    System.out.println(s.toUpperCase()); // s is already cast!
}
```

### Control Flow

```java
// Enhanced switch (Java 14+)
String dayType = switch (day) {
    case MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY -> "Weekday";
    case SATURDAY, SUNDAY -> "Weekend";
};

// Switch with pattern matching (Java 21+)
static String formatter(Object obj) {
    return switch (obj) {
        case Integer i -> String.format("int %d", i);
        case Long l    -> String.format("long %d", l);
        case Double d  -> String.format("double %f", d);
        case String s  -> String.format("String %s", s);
        case null      -> "null";
        default        -> obj.toString();
    };
}

// For-each loop
for (String item : list) {
    System.out.println(item);
}

// Labeled break/continue
outer:
for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 10; j++) {
        if (j == 5) break outer; // breaks out of BOTH loops
    }
}
```

---

## 5. Object-Oriented Programming (OOP)

### The Four Pillars of OOP

### 5.1 Encapsulation

Encapsulation means **bundling data (fields) and methods that operate on that data into a single unit (class)**, and restricting direct access to some of the object's components.

```java
public class BankAccount {
    // Private fields — cannot be accessed directly from outside
    private String accountNumber;
    private double balance;
    private String ownerName;

    // Constructor
    public BankAccount(String accountNumber, String ownerName, double initialBalance) {
        this.accountNumber = accountNumber;
        this.ownerName = ownerName;
        if (initialBalance < 0) {
            throw new IllegalArgumentException("Initial balance cannot be negative");
        }
        this.balance = initialBalance;
    }

    // Public getter — controlled read access
    public double getBalance() {
        return balance;
    }

    // Public method — controlled write access with validation
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Deposit amount must be positive");
        }
        this.balance += amount;
    }

    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Withdrawal amount must be positive");
        }
        if (amount > balance) {
            throw new InsufficientFundsException("Insufficient balance");
        }
        this.balance -= amount;
    }

    // No setter for accountNumber — it's immutable after creation
    public String getAccountNumber() {
        return accountNumber;
    }
}
```

**Why Encapsulation matters:**
- **Data validation** — You control what values are acceptable
- **Flexibility** — You can change internal implementation without affecting external code
- **Security** — Sensitive data is protected from unauthorized access

### 5.2 Inheritance

Inheritance allows a class to **inherit fields and methods from another class**, promoting code reuse.

```java
// Base class (Parent/Superclass)
public class Vehicle {
    protected String brand;
    protected int year;
    protected double speed;

    public Vehicle(String brand, int year) {
        this.brand = brand;
        this.year = year;
        this.speed = 0;
    }

    public void accelerate(double amount) {
        this.speed += amount;
        System.out.println(brand + " accelerating to " + speed + " km/h");
    }

    public void brake() {
        this.speed = Math.max(0, speed - 10);
    }

    @Override
    public String toString() {
        return brand + " (" + year + ") - Speed: " + speed + " km/h";
    }
}

// Derived class (Child/Subclass)
public class ElectricCar extends Vehicle {
    private int batteryLevel;

    public ElectricCar(String brand, int year, int batteryLevel) {
        super(brand, year); // Call parent constructor
        this.batteryLevel = batteryLevel;
    }

    // Method specific to ElectricCar
    public void charge() {
        this.batteryLevel = 100;
        System.out.println(brand + " fully charged!");
    }

    @Override
    public void accelerate(double amount) {
        if (batteryLevel <= 0) {
            System.out.println("Battery dead! Cannot accelerate.");
            return;
        }
        super.accelerate(amount); // Call parent's accelerate
        batteryLevel -= 2; // Battery drains when accelerating
    }
}
```

**Key Rules of Inheritance:**
- Java supports **single inheritance** only (one parent class)
- Use `extends` keyword
- All classes implicitly extend `java.lang.Object`
- `super` is used to call parent class constructor/methods
- Constructors are NOT inherited
- `private` members are NOT inherited (but they exist in the object)

### 5.3 Polymorphism

Polymorphism means **"many forms"** — the ability of objects to take different forms.

#### Compile-Time Polymorphism (Method Overloading)

```java
public class Calculator {
    // Same method name, different parameter lists
    public int add(int a, int b) {
        return a + b;
    }

    public double add(double a, double b) {
        return a + b;
    }

    public int add(int a, int b, int c) {
        return a + b + c;
    }

    public String add(String a, String b) {
        return a + b; // String concatenation
    }
}
```

#### Runtime Polymorphism (Method Overriding)

```java
public class Shape {
    public double area() {
        return 0;
    }
}

public class Circle extends Shape {
    private double radius;

    public Circle(double radius) {
        this.radius = radius;
    }

    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

public class Rectangle extends Shape {
    private double width, height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public double area() {
        return width * height;
    }
}

// Runtime polymorphism in action
Shape shape1 = new Circle(5);      // Shape reference, Circle object
Shape shape2 = new Rectangle(4, 6); // Shape reference, Rectangle object

System.out.println(shape1.area()); // 78.54 — Circle's area() is called
System.out.println(shape2.area()); // 24.0 — Rectangle's area() is called
// The JVM decides WHICH method to call at RUNTIME based on the actual object type
```

### 5.4 Abstraction

Abstraction means **hiding implementation details and showing only the essential features**.

#### Abstract Classes

```java
public abstract class Database {
    // Abstract method — no implementation, MUST be overridden
    public abstract void connect();
    public abstract void disconnect();
    public abstract List<Map<String, Object>> executeQuery(String sql);

    // Concrete method — has implementation, can be inherited as-is
    public void executeTransaction(List<String> queries) {
        connect();
        try {
            for (String query : queries) {
                executeQuery(query);
            }
            System.out.println("Transaction committed");
        } catch (Exception e) {
            System.out.println("Transaction rolled back");
        } finally {
            disconnect();
        }
    }
}

public class MySQLDatabase extends Database {
    @Override
    public void connect() {
        System.out.println("Connected to MySQL via JDBC");
    }

    @Override
    public void disconnect() {
        System.out.println("Disconnected from MySQL");
    }

    @Override
    public List<Map<String, Object>> executeQuery(String sql) {
        // MySQL-specific implementation
        return new ArrayList<>();
    }
}
```

#### Interfaces

```java
// Interface — 100% abstraction (before Java 8)
public interface Sortable<T> {
    void sort(List<T> items);
    boolean isSorted(List<T> items);

    // Default method (Java 8+) — provides default implementation
    default void sortAndPrint(List<T> items) {
        sort(items);
        items.forEach(System.out::println);
    }

    // Static method (Java 8+)
    static <T extends Comparable<T>> boolean isListSorted(List<T> list) {
        for (int i = 0; i < list.size() - 1; i++) {
            if (list.get(i).compareTo(list.get(i + 1)) > 0) return false;
        }
        return true;
    }

    // Private method (Java 9+) — helper for default methods
    private void log(String message) {
        System.out.println("[Sortable] " + message);
    }
}

// A class can implement MULTIPLE interfaces
public class Student implements Comparable<Student>, Serializable, Cloneable {
    // ...
}
```

#### Abstract Class vs Interface

| Feature | Abstract Class | Interface |
|---------|---------------|-----------|
| Methods | Abstract + Concrete | Abstract + Default + Static + Private |
| Fields | Any type | Only `public static final` |
| Constructors | Yes | No |
| Inheritance | Single (`extends`) | Multiple (`implements`) |
| Access Modifiers | Any | `public` (methods), `public static final` (fields) |
| When to use | IS-A with shared state | CAN-DO capability |

---

## 6. Constructors & Initialization Blocks

```java
public class Employee {
    private static int totalEmployees;
    private String name;
    private int id;

    // Static initialization block — runs ONCE when class is loaded
    static {
        totalEmployees = 0;
        System.out.println("Static block executed");
    }

    // Instance initialization block — runs EVERY TIME an object is created
    // Runs BEFORE the constructor
    {
        this.id = ++totalEmployees;
        System.out.println("Instance block executed for employee #" + id);
    }

    // No-argument constructor
    public Employee() {
        this("Unknown"); // Constructor chaining
    }

    // Parameterized constructor
    public Employee(String name) {
        this.name = name;
        System.out.println("Constructor executed for " + name);
    }

    // Copy constructor
    public Employee(Employee other) {
        this.name = other.name;
        // id is set by instance initializer block
    }
}

// Execution order:
// 1. Static block (once, when class loads)
// 2. Instance initializer block (each time)
// 3. Constructor body (each time)
```

---

## 7. Strings & String Handling

### String Immutability

```java
// Strings are IMMUTABLE in Java — once created, they CANNOT be changed
String s1 = "Hello";
String s2 = s1.concat(" World"); // Creates a NEW string object
System.out.println(s1); // "Hello" — s1 is unchanged!
System.out.println(s2); // "Hello World"

// String Pool (String Intern Pool)
String a = "Java";        // Created in String Pool
String b = "Java";        // Points to SAME object in pool
String c = new String("Java"); // Created in HEAP (not pool)

System.out.println(a == b);      // true (same reference in pool)
System.out.println(a == c);      // false (different objects)
System.out.println(a.equals(c)); // true (same content)

// Intern manually
String d = c.intern();   // Now points to the pool object
System.out.println(a == d); // true
```

### String vs StringBuilder vs StringBuffer

```java
// String — Immutable, thread-safe (because immutable)
String s = "Hello";
s = s + " World"; // Creates new object every time! Slow in loops.

// StringBuilder — Mutable, NOT thread-safe, FAST
StringBuilder sb = new StringBuilder("Hello");
sb.append(" World");     // Modifies in-place
sb.insert(5, ",");       // "Hello, World"
sb.delete(5, 6);         // "Hello World"
sb.reverse();            // "dlroW olleH"
String result = sb.toString();

// StringBuffer — Mutable, thread-safe (synchronized), SLOWER than StringBuilder
StringBuffer sbf = new StringBuffer("Hello");
sbf.append(" World"); // Same API as StringBuilder, but thread-safe

// Performance comparison in loops:
// String concatenation in loop: O(n²) — creates n objects
// StringBuilder in loop: O(n) — modifies in place
```

### Important String Methods

```java
String s = "Hello, World!";

s.length();                    // 13
s.charAt(0);                   // 'H'
s.indexOf("World");            // 7
s.lastIndexOf('l');            // 10
s.substring(7);                // "World!"
s.substring(0, 5);            // "Hello"
s.toLowerCase();               // "hello, world!"
s.toUpperCase();               // "HELLO, WORLD!"
s.trim();                      // Removes leading/trailing whitespace
s.strip();                     // Like trim() but Unicode-aware (Java 11+)
s.replace('l', 'L');           // "HeLLo, WorLd!"
s.replaceAll("[aeiou]", "*");  // Regex replacement
s.contains("World");           // true
s.startsWith("Hello");         // true
s.endsWith("!");               // true
s.isEmpty();                   // false
s.isBlank();                   // false (Java 11+, checks whitespace too)
s.split(",");                  // ["Hello", " World!"]
s.toCharArray();               // char array
String.join("-", "a", "b");    // "a-b"
s.chars();                     // IntStream of characters
s.repeat(3);                   // "Hello, World!Hello, World!Hello, World!" (Java 11+)

// Text Blocks (Java 15+)
String json = """
        {
            "name": "Dilip",
            "role": "Developer"
        }
        """;

// Formatted Strings
String formatted = String.format("Name: %s, Age: %d", "Dilip", 25);
String formatted2 = "Name: %s, Age: %d".formatted("Dilip", 25); // Java 15+
```

---

## 8. Arrays & Varargs

### Arrays

```java
// Declaration and initialization
int[] numbers = new int[5];              // Default values: 0
int[] nums = {1, 2, 3, 4, 5};          // Inline initialization
int[][] matrix = new int[3][4];          // 2D array
int[][] jagged = {{1, 2}, {3, 4, 5}};  // Jagged array

// Array operations
Arrays.sort(nums);                       // Sort in place
Arrays.fill(numbers, 42);               // Fill all elements with 42
int idx = Arrays.binarySearch(nums, 3); // Binary search (array must be sorted)
int[] copy = Arrays.copyOf(nums, 10);   // Copy with new length
boolean eq = Arrays.equals(nums, copy);  // Compare arrays
String str = Arrays.toString(nums);      // "[1, 2, 3, 4, 5]"
String str2D = Arrays.deepToString(matrix); // For multi-dimensional

// Convert to List
List<Integer> list = Arrays.asList(1, 2, 3); // Fixed-size list (backed by array)
List<Integer> mutableList = new ArrayList<>(Arrays.asList(1, 2, 3));

// Stream from array
IntStream stream = Arrays.stream(nums);
```

### Varargs (Variable Arguments)

```java
public static int sum(int... numbers) {
    // 'numbers' is treated as an array inside the method
    int total = 0;
    for (int n : numbers) {
        total += n;
    }
    return total;
}

// Can be called with any number of arguments
sum();           // 0
sum(1);          // 1
sum(1, 2, 3);   // 6
sum(1, 2, 3, 4, 5); // 15

// Rules:
// - Only ONE varargs parameter per method
// - Must be the LAST parameter
// public void method(String name, int... scores) // ✅ Valid
// public void method(int... scores, String name) // ❌ Invalid
```

---

## 9. Exception Handling

### Exception Hierarchy

```
java.lang.Throwable
├── java.lang.Error (Unchecked — DON'T catch these)
│   ├── OutOfMemoryError
│   ├── StackOverflowError
│   └── VirtualMachineError
└── java.lang.Exception (Checked — MUST handle)
    ├── IOException
    ├── SQLException
    ├── ClassNotFoundException
    └── java.lang.RuntimeException (Unchecked — optional handling)
        ├── NullPointerException
        ├── ArrayIndexOutOfBoundsException
        ├── ArithmeticException
        ├── ClassCastException
        ├── IllegalArgumentException
        └── NumberFormatException
```

### try-catch-finally

```java
public String readFile(String path) {
    BufferedReader reader = null;
    try {
        reader = new BufferedReader(new FileReader(path));
        return reader.readLine();
    } catch (FileNotFoundException e) {
        System.err.println("File not found: " + path);
        return null;
    } catch (IOException e) {
        System.err.println("Error reading file: " + e.getMessage());
        return null;
    } finally {
        // ALWAYS executes (even if return is in try/catch)
        if (reader != null) {
            try {
                reader.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

### try-with-resources (Java 7+)

```java
// AutoCloseable resources are automatically closed
public String readFile(String path) {
    try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
        return reader.readLine();
    } catch (IOException e) {
        System.err.println("Error: " + e.getMessage());
        return null;
    }
    // reader.close() is called automatically!
}

// Multiple resources
try (
    FileInputStream fis = new FileInputStream("input.txt");
    FileOutputStream fos = new FileOutputStream("output.txt");
    BufferedReader reader = new BufferedReader(new InputStreamReader(fis))
) {
    // Use resources
} // All three are closed in reverse order
```

### Custom Exceptions

```java
// Checked Exception
public class InsufficientFundsException extends Exception {
    private final double amount;
    private final double balance;

    public InsufficientFundsException(double amount, double balance) {
        super(String.format("Cannot withdraw %.2f. Balance: %.2f", amount, balance));
        this.amount = amount;
        this.balance = balance;
    }

    public double getDeficit() {
        return amount - balance;
    }
}

// Unchecked Exception
public class InvalidOrderException extends RuntimeException {
    public InvalidOrderException(String message) {
        super(message);
    }

    public InvalidOrderException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

### Multi-catch and Exception Chaining

```java
// Multi-catch (Java 7+)
try {
    // risky operation
} catch (IOException | SQLException | ClassNotFoundException e) {
    // Handle all three the same way
    logger.error("Operation failed", e);
}

// Exception chaining — preserving the root cause
try {
    connectToDatabase();
} catch (SQLException e) {
    throw new ServiceException("Failed to initialize service", e); // 'e' is the cause
}
```

---

## 10. Collections Framework

### Collections Hierarchy

```
Iterable
└── Collection
    ├── List (ordered, allows duplicates)
    │   ├── ArrayList   — Dynamic array, fast random access O(1)
    │   ├── LinkedList  — Doubly-linked list, fast insert/delete O(1)
    │   ├── Vector      — Synchronized ArrayList (legacy, avoid)
    │   └── Stack       — LIFO (legacy, use Deque instead)
    ├── Set (no duplicates)
    │   ├── HashSet     — Unordered, O(1) operations
    │   ├── LinkedHashSet — Insertion-ordered
    │   └── TreeSet     — Sorted (Red-Black tree), O(log n)
    └── Queue
        ├── PriorityQueue — Heap-based, O(log n) operations
        ├── ArrayDeque    — Resizable array deque
        └── LinkedList    — Also implements Deque

Map (NOT part of Collection interface)
├── HashMap       — Unordered, O(1) operations, allows null key
├── LinkedHashMap  — Insertion-ordered
├── TreeMap        — Sorted by keys (Red-Black tree), O(log n)
├── Hashtable      — Synchronized (legacy, use ConcurrentHashMap)
└── ConcurrentHashMap — Thread-safe, high-concurrency
```

### ArrayList Deep Dive

```java
// ArrayList — backed by a dynamic array
// Default capacity: 10
// Growth: 50% increase (newCapacity = oldCapacity + (oldCapacity >> 1))

List<String> list = new ArrayList<>(); // Initial capacity 10
List<String> list2 = new ArrayList<>(100); // Pre-allocate for performance

list.add("Java");           // Append: O(1) amortized
list.add(0, "Spring");     // Insert at index: O(n) — shifts elements
list.get(0);                // Random access: O(1)
list.set(0, "Spring Boot"); // Replace: O(1)
list.remove("Java");        // Remove by object: O(n)
list.remove(0);             // Remove by index: O(n)
list.contains("Java");      // Search: O(n)
list.indexOf("Java");       // Find index: O(n)
list.size();                 // Size: O(1)
list.isEmpty();              // Check empty: O(1)
list.clear();                // Remove all: O(n)
list.sort(Comparator.naturalOrder()); // Sort: O(n log n)

// Immutable lists (Java 9+)
List<String> immutable = List.of("a", "b", "c"); // Cannot add/remove/modify
// immutable.add("d"); // throws UnsupportedOperationException!

// Immutable copy (Java 10+)
List<String> copy = List.copyOf(list);
```

### HashMap Deep Dive

```java
// HashMap internals:
// - Uses array of Node<K,V>[] (buckets)
// - Default capacity: 16
// - Default load factor: 0.75
// - When size > capacity * loadFactor, it REHASHES (doubles capacity)
// - Java 8+: When bucket has 8+ entries, linked list → Red-Black tree (O(log n))
// - When bucket drops to 6 entries, tree → linked list

Map<String, Integer> map = new HashMap<>();

map.put("Java", 1);         // Insert: O(1) average
map.get("Java");             // Get: O(1) average
map.getOrDefault("Python", 0); // Get with default
map.containsKey("Java");    // Check key: O(1)
map.containsValue(1);       // Check value: O(n)
map.remove("Java");         // Remove: O(1)
map.size();                  // Size: O(1)

// Iteration
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}

map.forEach((key, value) -> System.out.println(key + " = " + value));

// Java 8+ methods
map.putIfAbsent("Java", 1);
map.computeIfAbsent("count", k -> 0);
map.computeIfPresent("count", (k, v) -> v + 1);
map.merge("count", 1, Integer::sum); // Increment or set to 1

// Immutable maps (Java 9+)
Map<String, Integer> immutable = Map.of("a", 1, "b", 2);
Map<String, Integer> immutable2 = Map.ofEntries(
    Map.entry("a", 1),
    Map.entry("b", 2)
);
```

### Comparable vs Comparator

```java
// Comparable — natural ordering (implemented BY the class itself)
public class Student implements Comparable<Student> {
    private String name;
    private double gpa;

    @Override
    public int compareTo(Student other) {
        return Double.compare(this.gpa, other.gpa); // Sort by GPA
    }
}

// Comparator — custom ordering (external to the class)
Comparator<Student> byName = Comparator.comparing(Student::getName);
Comparator<Student> byGpaDesc = Comparator.comparing(Student::getGpa).reversed();
Comparator<Student> byNameThenGpa = Comparator
    .comparing(Student::getName)
    .thenComparing(Student::getGpa);

List<Student> students = new ArrayList<>();
students.sort(byNameThenGpa); // Sort using custom comparator
Collections.sort(students, byGpaDesc); // Alternative
```

### equals() and hashCode() Contract

```java
public class Employee {
    private int id;
    private String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;                   // Same reference
        if (o == null || getClass() != o.getClass()) return false; // Null or different class
        Employee employee = (Employee) o;
        return id == employee.id && Objects.equals(name, employee.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(id, name);
    }

    // THE CONTRACT:
    // 1. If a.equals(b), then a.hashCode() == b.hashCode()
    // 2. If a.hashCode() != b.hashCode(), then !a.equals(b)
    // 3. If a.hashCode() == b.hashCode(), a.equals(b) MAY or MAY NOT be true (collision)
    // RULE: Always override BOTH or NEITHER
}
```

---

## 11. Generics

### Why Generics?

```java
// WITHOUT Generics (pre-Java 5) — unsafe!
List list = new ArrayList();
list.add("Hello");
list.add(42); // No compile error! Mixed types
String s = (String) list.get(1); // ClassCastException at RUNTIME!

// WITH Generics — type-safe!
List<String> list = new ArrayList<>();
list.add("Hello");
// list.add(42); // COMPILE ERROR! Type mismatch
String s = list.get(0); // No cast needed
```

### Generic Classes, Interfaces, and Methods

```java
// Generic Class
public class Pair<K, V> {
    private K key;
    private V value;

    public Pair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() { return key; }
    public V getValue() { return value; }
}

Pair<String, Integer> pair = new Pair<>("Age", 25);

// Generic Interface
public interface Repository<T, ID> {
    T findById(ID id);
    List<T> findAll();
    T save(T entity);
    void deleteById(ID id);
}

// Generic Method
public static <T extends Comparable<T>> T findMax(List<T> list) {
    if (list.isEmpty()) throw new IllegalArgumentException("Empty list");
    T max = list.get(0);
    for (T item : list) {
        if (item.compareTo(max) > 0) {
            max = item;
        }
    }
    return max;
}
```

### Bounded Type Parameters & Wildcards

```java
// Upper bound — T must be a Number or its subclass
public static <T extends Number> double sum(List<T> list) {
    return list.stream().mapToDouble(Number::doubleValue).sum();
}

// Multiple bounds
public static <T extends Comparable<T> & Serializable> void process(T item) {
    // T must implement BOTH Comparable AND Serializable
}

// Wildcards
List<?> anything = new ArrayList<String>(); // Unknown type (read-only effectively)
List<? extends Number> numbers = new ArrayList<Integer>(); // Upper-bounded wildcard
List<? super Integer> integers = new ArrayList<Number>(); // Lower-bounded wildcard

// PECS: Producer Extends, Consumer Super
// Use "? extends T" when you READ from the collection (producer)
// Use "? super T" when you WRITE to the collection (consumer)
public static <T> void copy(List<? extends T> source, List<? super T> destination) {
    for (T item : source) {
        destination.add(item);
    }
}
```

### Type Erasure

```java
// At compile time: List<String>, List<Integer> are different types
// At runtime: Both become just List (raw type) — this is TYPE ERASURE

// Consequences:
// 1. Cannot create generic arrays: new T[10] ❌
// 2. Cannot use instanceof with generics: obj instanceof List<String> ❌
// 3. Cannot create instances of type parameter: new T() ❌
// 4. Static fields cannot use class type parameter

// Bridge methods are generated by the compiler to maintain polymorphism
```

---

## 12. Lambda Expressions & Functional Interfaces

### Functional Interfaces

A **functional interface** has exactly **one abstract method**. It can have default and static methods.

```java
@FunctionalInterface
public interface Transformer<T, R> {
    R transform(T input); // Single abstract method

    default Transformer<T, R> andThen(Transformer<R, R> after) {
        return input -> after.transform(this.transform(input));
    }
}
```

### Built-in Functional Interfaces (java.util.function)

| Interface | Method | Signature | Use Case |
|-----------|--------|-----------|----------|
| `Predicate<T>` | `test(T)` | `T → boolean` | Filtering |
| `Function<T, R>` | `apply(T)` | `T → R` | Transformation |
| `Consumer<T>` | `accept(T)` | `T → void` | Side effects |
| `Supplier<T>` | `get()` | `() → T` | Factory/lazy creation |
| `UnaryOperator<T>` | `apply(T)` | `T → T` | Same type transform |
| `BinaryOperator<T>` | `apply(T, T)` | `(T, T) → T` | Combining two values |
| `BiFunction<T, U, R>` | `apply(T, U)` | `(T, U) → R` | Two-arg function |
| `BiPredicate<T, U>` | `test(T, U)` | `(T, U) → boolean` | Two-arg test |

### Lambda Syntax

```java
// Full syntax
(parameters) -> { statements; return value; }

// Simplified forms
(String s) -> { return s.length(); }  // Full
(s) -> { return s.length(); }         // Type inference
s -> { return s.length(); }           // Single parameter: no parentheses
s -> s.length()                       // Single expression: no braces, no return

// Examples
Predicate<String> isLong = s -> s.length() > 10;
Function<String, Integer> toLength = String::length; // Method reference
Consumer<String> printer = System.out::println; // Method reference
Supplier<List<String>> listFactory = ArrayList::new; // Constructor reference
BinaryOperator<Integer> add = Integer::sum;
Comparator<String> byLength = Comparator.comparingInt(String::length);

// Composing functions
Function<String, String> trim = String::trim;
Function<String, String> upper = String::toUpperCase;
Function<String, String> trimAndUpper = trim.andThen(upper);

Predicate<String> notEmpty = Predicate.not(String::isEmpty);
Predicate<String> shortString = s -> s.length() < 5;
Predicate<String> valid = notEmpty.and(shortString); // Combining predicates
```

### Method References

```java
// Four types of method references:

// 1. Static method reference
Function<String, Integer> parseInt = Integer::parseInt;
// Equivalent: s -> Integer.parseInt(s)

// 2. Instance method of a particular object
String prefix = "Hello";
Predicate<String> startsWith = prefix::startsWith;
// Equivalent: s -> prefix.startsWith(s)

// 3. Instance method of an arbitrary object of a particular type
Function<String, String> toUpper = String::toUpperCase;
// Equivalent: s -> s.toUpperCase()

// 4. Constructor reference
Supplier<ArrayList<String>> listMaker = ArrayList::new;
// Equivalent: () -> new ArrayList<>()
Function<Integer, ArrayList<String>> sizedList = ArrayList::new;
// Equivalent: size -> new ArrayList<>(size)
```

---

## 13. Streams API

### What are Streams?

Streams are **a sequence of elements supporting sequential and parallel aggregate operations**. They don't store data — they process data from a source (collection, array, I/O) through a pipeline of operations.

```
Source → Intermediate Operations → Terminal Operation → Result
```

### Creating Streams

```java
// From Collection
List<String> list = List.of("a", "b", "c");
Stream<String> stream = list.stream();
Stream<String> parallelStream = list.parallelStream();

// From values
Stream<String> stream = Stream.of("a", "b", "c");

// From array
int[] arr = {1, 2, 3, 4, 5};
IntStream intStream = Arrays.stream(arr);

// Infinite streams
Stream<Integer> infinite = Stream.iterate(0, n -> n + 2); // 0, 2, 4, 6, ...
Stream<Double> randoms = Stream.generate(Math::random); // Random doubles
Stream<Integer> bounded = Stream.iterate(0, n -> n < 100, n -> n + 2); // Java 9+

// From string
IntStream chars = "Hello".chars(); // IntStream of char values

// Range
IntStream range = IntStream.range(0, 10);       // 0 to 9
IntStream rangeClosed = IntStream.rangeClosed(1, 10); // 1 to 10
```

### Intermediate Operations (Lazy — not executed until terminal operation)

```java
List<Employee> employees = getEmployees();

// filter — keep elements matching predicate
employees.stream()
    .filter(e -> e.getSalary() > 50000)

// map — transform each element
    .map(Employee::getName) // Employee → String

// flatMap — flatten nested structures
List<List<String>> nested = List.of(List.of("a", "b"), List.of("c", "d"));
nested.stream().flatMap(Collection::stream); // Stream<String>: a, b, c, d

// sorted — sort elements
    .sorted() // Natural order
    .sorted(Comparator.comparing(Employee::getSalary).reversed())

// distinct — remove duplicates (uses equals())
    .distinct()

// peek — debug/inspect without modifying
    .peek(System.out::println) // DON'T use for side effects in production

// limit — take first N elements
    .limit(10)

// skip — skip first N elements
    .skip(5)

// takeWhile / dropWhile (Java 9+)
    .takeWhile(e -> e.getSalary() < 100000) // Take while condition is true
    .dropWhile(e -> e.getSalary() < 30000)  // Drop while condition is true

// mapMulti (Java 16+) — alternative to flatMap
    .mapMulti((e, consumer) -> {
        consumer.accept(e.getFirstName());
        consumer.accept(e.getLastName());
    });
```

### Terminal Operations (Eager — trigger processing)

```java
// forEach — perform action on each element
employees.stream().forEach(System.out::println);

// collect — accumulate into a collection
List<String> names = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.toList());

// toList() — Java 16+ shorthand (returns unmodifiable list)
List<String> names = employees.stream()
    .map(Employee::getName)
    .toList();

// reduce — combine all elements into one value
int totalSalary = employees.stream()
    .mapToInt(Employee::getSalary)
    .reduce(0, Integer::sum);

Optional<Integer> max = numbers.stream()
    .reduce(Integer::max);

// count
long count = employees.stream()
    .filter(e -> e.getDepartment().equals("Engineering"))
    .count();

// min / max
Optional<Employee> highestPaid = employees.stream()
    .max(Comparator.comparing(Employee::getSalary));

// anyMatch / allMatch / noneMatch
boolean anyRich = employees.stream().anyMatch(e -> e.getSalary() > 200000);
boolean allAdults = employees.stream().allMatch(e -> e.getAge() >= 18);

// findFirst / findAny
Optional<Employee> first = employees.stream()
    .filter(e -> e.getDepartment().equals("HR"))
    .findFirst();

// toArray
Employee[] array = employees.stream().toArray(Employee[]::new);
```

### Collectors

```java
// Grouping
Map<String, List<Employee>> byDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// Grouping with counting
Map<String, Long> countByDept = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment, Collectors.counting()));

// Grouping with summing
Map<String, Integer> salaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.summingInt(Employee::getSalary)
    ));

// Partitioning (split into two groups)
Map<Boolean, List<Employee>> partitioned = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.getSalary() > 50000));

// Joining strings
String names = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.joining(", ", "[", "]")); // [Alice, Bob, Charlie]

// Statistics
IntSummaryStatistics stats = employees.stream()
    .collect(Collectors.summarizingInt(Employee::getSalary));
stats.getAverage(); stats.getMax(); stats.getMin(); stats.getCount(); stats.getSum();

// toMap
Map<Integer, String> idToName = employees.stream()
    .collect(Collectors.toMap(Employee::getId, Employee::getName));

// toMap with merge function (handle duplicate keys)
Map<String, Integer> deptMaxSalary = employees.stream()
    .collect(Collectors.toMap(
        Employee::getDepartment,
        Employee::getSalary,
        Integer::max // Keep the higher salary if duplicate department
    ));

// toUnmodifiableList, toUnmodifiableSet, toUnmodifiableMap (Java 10+)
List<String> immutable = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.toUnmodifiableList());

// Teeing collector (Java 12+) — two collectors in one pass
var result = employees.stream()
    .collect(Collectors.teeing(
        Collectors.averagingInt(Employee::getSalary),
        Collectors.counting(),
        (avg, count) -> "Average: " + avg + ", Count: " + count
    ));
```

---

## 14. Multithreading & Concurrency

### Creating Threads

```java
// Method 1: Extend Thread class
public class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread running: " + Thread.currentThread().getName());
    }
}

MyThread t = new MyThread();
t.start(); // start() creates new thread; run() would execute in current thread!

// Method 2: Implement Runnable (PREFERRED — allows extending other classes)
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Runnable running: " + Thread.currentThread().getName());
    }
}

Thread t = new Thread(new MyRunnable());
t.start();

// Method 3: Lambda (simplest)
Thread t = new Thread(() -> System.out.println("Lambda thread"));
t.start();

// Method 4: Callable + Future (returns a result)
Callable<Integer> task = () -> {
    Thread.sleep(2000);
    return 42;
};
ExecutorService executor = Executors.newSingleThreadExecutor();
Future<Integer> future = executor.submit(task);
Integer result = future.get(); // Blocks until result is ready (or throws exception)
executor.shutdown();
```

### Thread Lifecycle

```
NEW → RUNNABLE → RUNNING → BLOCKED/WAITING/TIMED_WAITING → TERMINATED
                    ↑              ↓
                    └──────────────┘
```

### Synchronization

```java
// Problem: Race condition
public class Counter {
    private int count = 0;

    // Without synchronization — NOT thread-safe
    public void increment() {
        count++; // Read-modify-write is NOT atomic!
    }
}

// Solution 1: synchronized method
public class SynchronizedCounter {
    private int count = 0;

    public synchronized void increment() { // Locks on 'this'
        count++;
    }

    public synchronized int getCount() {
        return count;
    }
}

// Solution 2: synchronized block (more granular)
public class Counter {
    private final Object lock = new Object();
    private int count = 0;

    public void increment() {
        synchronized (lock) { // Lock only the critical section
            count++;
        }
    }
}

// Solution 3: AtomicInteger (lock-free, best for simple counters)
public class AtomicCounter {
    private final AtomicInteger count = new AtomicInteger(0);

    public void increment() {
        count.incrementAndGet(); // Thread-safe without locks!
    }

    public int getCount() {
        return count.get();
    }
}
```

### ExecutorService & Thread Pools

```java
// Fixed thread pool — fixed number of threads
ExecutorService fixed = Executors.newFixedThreadPool(4);

// Cached thread pool — creates threads as needed, reuses idle threads
ExecutorService cached = Executors.newCachedThreadPool();

// Single thread executor — one thread, tasks execute sequentially
ExecutorService single = Executors.newSingleThreadExecutor();

// Scheduled thread pool — for delayed/periodic tasks
ScheduledExecutorService scheduled = Executors.newScheduledThreadPool(2);
scheduled.scheduleAtFixedRate(() -> System.out.println("Tick"), 0, 1, TimeUnit.SECONDS);

// Submit tasks
Future<String> future = fixed.submit(() -> {
    Thread.sleep(1000);
    return "Result";
});

// Submit multiple tasks
List<Callable<String>> tasks = List.of(
    () -> "Task 1",
    () -> "Task 2",
    () -> "Task 3"
);
List<Future<String>> futures = fixed.invokeAll(tasks); // Wait for ALL
String firstResult = fixed.invokeAny(tasks); // Return FIRST completed

// Shutdown
fixed.shutdown(); // Graceful — finish current tasks
fixed.shutdownNow(); // Forceful — interrupt running tasks
fixed.awaitTermination(10, TimeUnit.SECONDS); // Wait for shutdown
```

### CompletableFuture (Java 8+)

```java
// Async computation
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // Runs in ForkJoinPool.commonPool()
    return fetchDataFromAPI();
});

// Chaining
CompletableFuture<String> result = CompletableFuture
    .supplyAsync(() -> fetchUser(userId))           // Step 1: Fetch user
    .thenApply(user -> user.getEmail())             // Step 2: Extract email
    .thenApplyAsync(email -> fetchOrders(email))    // Step 3: Fetch orders (async)
    .thenApply(orders -> formatOrders(orders))       // Step 4: Format
    .exceptionally(ex -> "Error: " + ex.getMessage()); // Handle errors

// Combining futures
CompletableFuture<String> userFuture = CompletableFuture.supplyAsync(() -> fetchUser());
CompletableFuture<String> orderFuture = CompletableFuture.supplyAsync(() -> fetchOrders());

CompletableFuture<String> combined = userFuture
    .thenCombine(orderFuture, (user, orders) -> user + ": " + orders);

// Wait for all
CompletableFuture<Void> allOf = CompletableFuture.allOf(future1, future2, future3);

// Wait for any
CompletableFuture<Object> anyOf = CompletableFuture.anyOf(future1, future2, future3);
```

### Virtual Threads (Java 21+)

```java
// Virtual threads are lightweight threads managed by the JVM (not OS)
// You can create MILLIONS of virtual threads

// Method 1: Thread.ofVirtual()
Thread vThread = Thread.ofVirtual().start(() -> {
    System.out.println("Virtual thread: " + Thread.currentThread());
});

// Method 2: ExecutorService
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    for (int i = 0; i < 100_000; i++) {
        executor.submit(() -> {
            Thread.sleep(Duration.ofSeconds(1));
            return "Done";
        });
    }
} // All 100,000 tasks run concurrently with minimal memory!

// Key benefits:
// - Extremely lightweight (few KB vs ~1MB for platform threads)
// - Can create millions of them
// - Perfect for I/O-bound tasks (HTTP calls, DB queries)
// - Managed by JVM scheduler, not OS
```

---

## 15. Java I/O & NIO

### Traditional I/O (java.io)

```java
// Byte Streams (binary data)
try (FileInputStream fis = new FileInputStream("input.bin");
     FileOutputStream fos = new FileOutputStream("output.bin")) {
    byte[] buffer = new byte[8192];
    int bytesRead;
    while ((bytesRead = fis.read(buffer)) != -1) {
        fos.write(buffer, 0, bytesRead);
    }
}

// Character Streams (text data)
try (BufferedReader reader = new BufferedReader(new FileReader("input.txt"));
     BufferedWriter writer = new BufferedWriter(new FileWriter("output.txt"))) {
    String line;
    while ((line = reader.readLine()) != null) {
        writer.write(line);
        writer.newLine();
    }
}

// Serialization
public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    private String name;
    private transient String password; // transient = not serialized

    // Serialize
    try (ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream("user.ser"))) {
        oos.writeObject(new User("Dilip", "secret"));
    }

    // Deserialize
    try (ObjectInputStream ois = new ObjectInputStream(new FileInputStream("user.ser"))) {
        User user = (User) ois.readObject();
    }
}
```

### NIO (java.nio — New I/O)

```java
// Path & Files API (Java 7+)
Path path = Path.of("data", "users.txt"); // data/users.txt
Path absolute = path.toAbsolutePath();
Path parent = path.getParent();
Path fileName = path.getFileName();

// File operations
Files.exists(path);
Files.isDirectory(path);
Files.isRegularFile(path);
Files.size(path);

// Read entire file
String content = Files.readString(path);                    // Java 11+
List<String> lines = Files.readAllLines(path);
byte[] bytes = Files.readAllBytes(path);

// Write file
Files.writeString(path, "Hello, World!");                   // Java 11+
Files.write(path, List.of("line1", "line2"));
Files.write(path, "append".getBytes(), StandardOpenOption.APPEND);

// Copy, Move, Delete
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
Files.move(source, target, StandardCopyOption.ATOMIC_MOVE);
Files.delete(path);
Files.deleteIfExists(path);

// Create directories
Files.createDirectory(path);
Files.createDirectories(path); // Creates parent dirs too

// Walk directory tree
try (Stream<Path> walk = Files.walk(Paths.get("."))) {
    walk.filter(Files::isRegularFile)
        .filter(p -> p.toString().endsWith(".java"))
        .forEach(System.out::println);
}

// Find files
try (Stream<Path> found = Files.find(Paths.get("."), 10,
        (p, attr) -> p.toString().endsWith(".java") && attr.size() > 1000)) {
    found.forEach(System.out::println);
}
```

---

## 16. Java Memory Model & Garbage Collection

### Memory Areas

```
┌─────────────────────────────────────┐
│            JVM Memory               │
├──────────────┬──────────────────────┤
│    Stack     │        Heap          │
│  (per thread)│    (shared)          │
│              │                      │
│ - Local vars │  ┌───────────────┐   │
│ - Method     │  │  Young Gen    │   │
│   calls      │  │  ├─ Eden      │   │
│ - Partial    │  │  ├─ S0        │   │
│   results    │  │  └─ S1        │   │
│              │  ├───────────────┤   │
│              │  │  Old Gen      │   │
│              │  │  (Tenured)    │   │
│              │  ├───────────────┤   │
│              │  │  Metaspace    │   │
│              │  │  (class meta) │   │
│              │  └───────────────┘   │
└──────────────┴──────────────────────┘
```

### Garbage Collection Process

```
1. New objects → Eden Space
2. Minor GC: Eden full → live objects move to S0/S1 (survivor spaces)
3. Objects surviving multiple minor GCs → Old Generation (tenured)
4. Major GC (Full GC): Old Gen full → collects entire heap (STOP-THE-WORLD)
```

### GC Algorithms

| Algorithm | Flag | Best For |
|-----------|------|----------|
| Serial GC | `-XX:+UseSerialGC` | Small apps, single core |
| Parallel GC | `-XX:+UseParallelGC` | Throughput-focused |
| G1 GC | `-XX:+UseG1GC` | Balanced (default Java 9+) |
| ZGC | `-XX:+UseZGC` | Ultra-low latency (<1ms pauses) |
| Shenandoah | `-XX:+UseShenandoahGC` | Low latency (OpenJDK) |

### Memory Tuning

```bash
# Heap size
-Xms512m          # Initial heap size
-Xmx2g            # Maximum heap size

# Young generation
-Xmn256m          # Young generation size

# Metaspace
-XX:MetaspaceSize=128m
-XX:MaxMetaspaceSize=512m

# GC logging
-Xlog:gc*:file=gc.log:time,level,tags

# Stack size per thread
-Xss512k
```

### Memory Leaks in Java

```java
// Common causes of memory leaks:

// 1. Static collections that grow forever
private static final List<Object> cache = new ArrayList<>(); // Never cleared!

// 2. Unclosed resources
Connection conn = DriverManager.getConnection(url); // Never closed!

// 3. Listeners/callbacks not deregistered
button.addActionListener(this); // Never removed!

// 4. Inner classes holding reference to outer class
public class Outer {
    byte[] data = new byte[10_000_000]; // 10MB
    class Inner { // Holds implicit reference to Outer.this
        // Even if Outer is no longer needed, Inner prevents GC
    }
}

// 5. Improper equals/hashCode in HashMap keys
// Objects become "lost" in the map — can't be retrieved or removed
```

---

## 17. Java 8–21 Feature Evolution

### Java 8 (2014) — THE Biggest Release

- Lambda Expressions
- Stream API
- Optional
- Default & Static methods in interfaces
- java.time API (LocalDate, LocalDateTime, etc.)
- CompletableFuture

### Java 9 (2017)

- Module System (Project Jigsaw)
- JShell (REPL)
- Collection factory methods: `List.of()`, `Set.of()`, `Map.of()`
- Private methods in interfaces
- `Optional.ifPresentOrElse()`, `Optional.or()`, `Optional.stream()`

### Java 10 (2018)

- `var` (Local Variable Type Inference)
- `List.copyOf()`, `Set.copyOf()`, `Map.copyOf()`
- `Collectors.toUnmodifiableList()`

### Java 11 (2018) — LTS

- `String.isBlank()`, `String.lines()`, `String.strip()`, `String.repeat()`
- `Files.readString()`, `Files.writeString()`
- `HttpClient` (standard)
- `var` in lambda parameters

### Java 14 (2020)

- Switch Expressions (standard)
- Records (preview)
- `NullPointerException` with helpful messages

### Java 16 (2021)

- Records (standard)
- Pattern Matching for `instanceof` (standard)
- `Stream.toList()`

### Java 17 (2021) — LTS

- Sealed Classes (standard)
- Pattern Matching for switch (preview)
- Text Blocks (standard since 15)

### Java 21 (2023) — LTS

- Virtual Threads (standard)
- Sequenced Collections
- Record Patterns (standard)
- Pattern Matching for switch (standard)
- String Templates (preview)

---

## 18. Sealed Classes, Records, and Pattern Matching

### Records (Java 16+)

```java
// Records are immutable data carriers — auto-generate:
// - Constructor, getters, equals(), hashCode(), toString()
public record Point(double x, double y) {
    // Compact constructor (validation)
    public Point {
        if (x < 0 || y < 0) {
            throw new IllegalArgumentException("Coordinates must be positive");
        }
    }

    // Custom method
    public double distanceTo(Point other) {
        return Math.sqrt(Math.pow(this.x - other.x, 2) + Math.pow(this.y - other.y, 2));
    }
}

Point p = new Point(3.0, 4.0);
System.out.println(p.x());        // 3.0 (NOT getX())
System.out.println(p);             // Point[x=3.0, y=4.0]
```

### Sealed Classes (Java 17+)

```java
// Sealed classes restrict which classes can extend them
public sealed class Shape permits Circle, Rectangle, Triangle {
    public abstract double area();
}

public final class Circle extends Shape {
    private final double radius;

    public Circle(double radius) { this.radius = radius; }

    @Override
    public double area() { return Math.PI * radius * radius; }
}

public final class Rectangle extends Shape {
    private final double width, height;

    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }

    @Override
    public double area() { return width * height; }
}

public non-sealed class Triangle extends Shape {
    // non-sealed: any class can extend Triangle
    private final double base, height;

    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }

    @Override
    public double area() { return 0.5 * base * height; }
}
```

### Pattern Matching (Java 21+)

```java
// Pattern matching for switch — exhaustive, type-safe
public double calculateArea(Shape shape) {
    return switch (shape) {
        case Circle c -> Math.PI * c.radius() * c.radius();
        case Rectangle r -> r.width() * r.height();
        case Triangle t -> 0.5 * t.base() * t.height();
        // No default needed — sealed class, all cases covered!
    };
}

// Record patterns (destructuring)
record Point(int x, int y) {}
record Line(Point start, Point end) {}

void printLine(Line line) {
    if (line instanceof Line(Point(var x1, var y1), Point(var x2, var y2))) {
        System.out.println("Line from (%d,%d) to (%d,%d)".formatted(x1, y1, x2, y2));
    }
}

// Guarded patterns
String describe(Object obj) {
    return switch (obj) {
        case Integer i when i > 0 -> "Positive integer: " + i;
        case Integer i when i < 0 -> "Negative integer: " + i;
        case Integer i            -> "Zero";
        case String s when s.length() > 10 -> "Long string";
        case String s             -> "Short string: " + s;
        case null                 -> "null";
        default                   -> "Unknown: " + obj;
    };
}
```

---

## 19. Best Practices & Code Conventions

### Naming Conventions

| Element | Convention | Example |
|---------|-----------|---------|
| Package | lowercase, reverse domain | `com.company.project` |
| Class | PascalCase, noun | `UserService`, `OrderRepository` |
| Interface | PascalCase, adjective or noun | `Serializable`, `UserRepository` |
| Method | camelCase, verb | `getUserById()`, `calculateTotal()` |
| Variable | camelCase, meaningful | `firstName`, `orderCount` |
| Constant | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT`, `DEFAULT_TIMEOUT` |
| Enum | PascalCase (type), UPPER_SNAKE_CASE (values) | `Status.ACTIVE` |

### SOLID Principles

```java
// S — Single Responsibility Principle
// A class should have ONE reason to change
// ❌ BAD: UserService that handles auth, email, and CRUD
// ✅ GOOD: UserService, AuthService, EmailService (separate concerns)

// O — Open/Closed Principle
// Open for extension, closed for modification
// ❌ BAD: Adding new shape requires modifying AreaCalculator
// ✅ GOOD: Shape interface → each shape implements its own area()

// L — Liskov Substitution Principle
// Subtypes must be substitutable for their base types
// ❌ BAD: Square extends Rectangle (changing width changes height — violates contract)
// ✅ GOOD: Both implement Shape interface

// I — Interface Segregation Principle
// Clients shouldn't depend on methods they don't use
// ❌ BAD: One huge interface with 20 methods
// ✅ GOOD: Multiple focused interfaces (Readable, Writable, Closeable)

// D — Dependency Inversion Principle
// Depend on abstractions, not concrete implementations
// ❌ BAD: UserService directly creates new MySQLUserRepository()
// ✅ GOOD: UserService depends on UserRepository interface (injected)
```

### Effective Java Key Rules (Joshua Bloch)

1. **Use static factory methods instead of constructors** — `Optional.of()`, `List.of()`
2. **Use builders for many constructor parameters** — `User.builder().name("Dilip").build()`
3. **Enforce singleton with enum** — `enum Singleton { INSTANCE; }`
4. **Prefer composition over inheritance** — Use delegation, not `extends`
5. **Design for inheritance or prohibit it** — Use `final` class or sealed
6. **Prefer interfaces over abstract classes** — More flexible
7. **Use Optional instead of null** — Explicit absence
8. **Make defensive copies** — Protect mutable state
9. **Minimize mutability** — Use `final`, records, immutable collections
10. **Favor `List.of()` over `Arrays.asList()`** — Truly immutable

---

## 20. Common Mistakes & Pitfalls

```java
// ❌ MISTAKE 1: String comparison with ==
String a = new String("hello");
String b = new String("hello");
if (a == b) { } // FALSE! Compares references
if (a.equals(b)) { } // ✅ TRUE! Compares content

// ❌ MISTAKE 2: Integer caching gotcha
Integer x = 127;
Integer y = 127;
System.out.println(x == y); // true (cached: -128 to 127)
Integer p = 128;
Integer q = 128;
System.out.println(p == q); // FALSE! Not cached

// ❌ MISTAKE 3: ConcurrentModificationException
List<String> list = new ArrayList<>(List.of("a", "b", "c"));
for (String s : list) {
    list.remove(s); // ConcurrentModificationException!
}
// ✅ Use iterator.remove() or removeIf()
list.removeIf(s -> s.equals("b"));

// ❌ MISTAKE 4: Floating point comparison
double result = 0.1 + 0.2;
System.out.println(result == 0.3); // FALSE! (result is 0.30000000000000004)
// ✅ Use BigDecimal for precise calculations or compare with epsilon

// ❌ MISTAKE 5: Modifying unmodifiable collections
List<String> immutable = List.of("a", "b");
immutable.add("c"); // UnsupportedOperationException!

// ❌ MISTAKE 6: Forgetting to close resources
Connection conn = DriverManager.getConnection(url);
// If exception occurs, conn is NEVER closed → connection leak!
// ✅ Use try-with-resources

// ❌ MISTAKE 7: Catching Exception/Throwable
try {
    riskyOperation();
} catch (Exception e) { // Too broad — catches RuntimeExceptions too
    // Might swallow important errors
}
// ✅ Catch specific exceptions

// ❌ MISTAKE 8: Empty catch blocks
try {
    Integer.parseInt("abc");
} catch (NumberFormatException e) {
    // Silently swallowed — debugging nightmare!
}
// ✅ At minimum, log the error

// ❌ MISTAKE 9: Using Date instead of LocalDate
Date date = new Date(); // Mutable, poorly designed (month starts at 0!)
// ✅ Use java.time API
LocalDate today = LocalDate.now();
LocalDateTime now = LocalDateTime.now();

// ❌ MISTAKE 10: Synchronizing on wrong object
private Integer count = 0;
synchronized (count) { // Integer is immutable — new object each time!
    count++; // Synchronizing on different objects!
}
// ✅ Use a dedicated lock object
private final Object lock = new Object();
```

---

## 21. Interview Questions & Answers (50+)

### Beginner Level

**Q1: What is the difference between JDK, JRE, and JVM?**

**A:** 
- **JVM** (Java Virtual Machine) is an abstract machine that executes Java bytecode. It provides the runtime environment.
- **JRE** (Java Runtime Environment) = JVM + standard libraries. It's what end users need to RUN Java applications.
- **JDK** (Java Development Kit) = JRE + development tools (javac, jar, javadoc, jdb). It's what developers need to WRITE and COMPILE Java.

---

**Q2: Why is Java platform-independent?**

**A:** Java source code is compiled into **bytecode** (.class files) by `javac`, not into machine-specific native code. This bytecode runs on the **JVM**, which is platform-specific. So the bytecode is portable — write once, run anywhere (WORA). The JVM handles translating bytecode to native instructions for the specific OS/hardware.

---

**Q3: What is the difference between `==` and `.equals()`?**

**A:**
- `==` compares **references** (memory addresses) — are these the exact same object?
- `.equals()` compares **content/value** — do these objects have the same data?

For primitives, `==` compares values (there are no references). For objects, always use `.equals()` for value comparison. The `String` class overrides `.equals()` to compare character sequences.

---

**Q4: What are wrapper classes and what is autoboxing?**

**A:** Wrapper classes represent primitives as objects: `int → Integer`, `double → Double`, etc. **Autoboxing** is automatic conversion from primitive to wrapper (`int → Integer`), and **unboxing** is the reverse (`Integer → int`). This allows primitives to be used with Collections (`List<Integer>`).

```java
Integer x = 5;        // Autoboxing: int → Integer
int y = x;            // Unboxing: Integer → int
List<Integer> list = new ArrayList<>();
list.add(42);         // Autoboxing
int val = list.get(0); // Unboxing
```

---

**Q5: What is the `final` keyword used for?**

**A:**
- `final` variable → Cannot be reassigned (constant). For objects, the reference is constant, but the object's state can change.
- `final` method → Cannot be overridden by subclasses.
- `final` class → Cannot be extended (e.g., `String`, `Integer`).
- `final` parameter → Cannot be reassigned within the method.

---

**Q6: Explain method overloading vs method overriding.**

**A:**
| Feature | Overloading | Overriding |
|---------|-------------|------------|
| What | Same name, different parameters | Same signature, different class |
| When | Compile-time polymorphism | Runtime polymorphism |
| Where | Same class or subclass | Subclass only |
| Return type | Can be different | Same or covariant |
| Access | Can be different | Same or wider |
| static/final | Can be overloaded | Cannot be overridden |

---

**Q7: What is the difference between `String`, `StringBuilder`, and `StringBuffer`?**

**A:**
| Feature | String | StringBuilder | StringBuffer |
|---------|--------|---------------|--------------|
| Mutability | Immutable | Mutable | Mutable |
| Thread-safe | Yes (immutable) | No | Yes (synchronized) |
| Performance | Slow for concatenation | Fast | Slower than StringBuilder |
| Use when | Few modifications | Single-threaded string manipulation | Multi-threaded string manipulation |

---

### Intermediate Level

**Q8: What is the difference between `ArrayList` and `LinkedList`?**

**A:**
| Feature | ArrayList | LinkedList |
|---------|-----------|------------|
| Data structure | Dynamic array | Doubly-linked list |
| Random access | O(1) | O(n) |
| Insert/delete at beginning | O(n) | O(1) |
| Insert/delete at end | O(1) amortized | O(1) |
| Memory | Less (contiguous) | More (node overhead) |
| Cache performance | Better (locality) | Worse (scattered) |

**In practice, `ArrayList` is almost always better** due to CPU cache locality. Use `LinkedList` only when you need constant-time insertion/deletion at both ends AND never need random access.

---

**Q9: How does HashMap work internally?**

**A:**
1. `HashMap` uses an **array of buckets** (default 16).
2. When you `put(key, value)`:
   - `hashCode()` of key → bucket index: `index = hash & (capacity - 1)`
   - If bucket is empty → store the entry
   - If bucket has entries → check `equals()` for duplicate key
   - If duplicate → update value
   - If no duplicate → add to the bucket's chain (linked list or tree)
3. **Java 8+**: When a bucket has 8+ entries, the linked list is converted to a **Red-Black tree** (O(log n) instead of O(n)).
4. When `size > capacity * loadFactor (0.75)`, the map **rehashes** — doubles capacity, redistributes entries.

---

**Q10: What is the difference between `Comparable` and `Comparator`?**

**A:**
- `Comparable` defines the **natural ordering** of a class. The class implements `Comparable<T>` and overrides `compareTo()`. There can be only ONE natural ordering.
- `Comparator` defines **custom/external ordering**. It's a separate class/lambda that can define MULTIPLE different orderings. Used when you want to sort by different criteria.

```java
// Comparable: Student implements Comparable<Student>
// Comparator: Comparator.comparing(Student::getName)
```

---

**Q11: What are checked vs unchecked exceptions?**

**A:**
- **Checked exceptions** (subclass of `Exception` but not `RuntimeException`): Must be declared in `throws` or caught. Examples: `IOException`, `SQLException`. The compiler enforces handling.
- **Unchecked exceptions** (subclass of `RuntimeException`): Not required to declare or catch. Examples: `NullPointerException`, `ArrayIndexOutOfBoundsException`. Usually indicate programming bugs.

---

**Q12: Explain the `Optional` class.**

**A:** `Optional<T>` is a container that may or may not contain a non-null value. It's designed to prevent `NullPointerException` and make the absence of a value explicit.

```java
Optional<User> user = userRepository.findById(id);

// ❌ BAD
if (user.isPresent()) {
    return user.get().getName();
}

// ✅ GOOD
return user.map(User::getName).orElse("Unknown");
return user.orElseThrow(() -> new UserNotFoundException(id));
```

Key rule: **Never use Optional as a field, parameter, or collection element.** It's designed for return types only.

---

**Q13: What is the diamond problem and how does Java solve it?**

**A:** The diamond problem occurs when a class inherits from two classes that have a method with the same signature. Java solves this by:
1. **Not allowing multiple class inheritance** — `extends` only one class
2. For **interfaces with default methods**, if two interfaces have conflicting default methods, the implementing class MUST override the method to resolve the ambiguity.

```java
interface A { default void hello() { System.out.println("A"); } }
interface B { default void hello() { System.out.println("B"); } }

class C implements A, B {
    @Override
    public void hello() {
        A.super.hello(); // Explicitly choose A's implementation
    }
}
```

---

**Q14: What is the difference between `fail-fast` and `fail-safe` iterators?**

**A:**
- **Fail-fast**: Throws `ConcurrentModificationException` if collection is modified during iteration (e.g., `ArrayList`, `HashMap`). Uses a modification counter.
- **Fail-safe**: Works on a clone/snapshot of the collection, so modifications don't cause exceptions (e.g., `ConcurrentHashMap`, `CopyOnWriteArrayList`). May not reflect latest changes.

---

### Advanced Level

**Q15: Explain Java Memory Model (JMM).**

**A:** The JMM defines how threads interact through memory. Key concepts:
- **Visibility**: Changes made by one thread may not be visible to other threads without synchronization (due to CPU caches).
- **Ordering**: Compiler and CPU can reorder instructions for optimization.
- **Happens-before**: A set of rules guaranteeing visibility. Example: unlock of a monitor happens-before every subsequent lock of that same monitor.
- `volatile` ensures visibility and prevents reordering.
- `synchronized` ensures both visibility and atomicity.

---

**Q16: What is the `volatile` keyword?**

**A:** `volatile` ensures:
1. **Visibility**: Reads and writes go directly to main memory (bypass CPU cache).
2. **Ordering**: Prevents instruction reordering around volatile reads/writes.

It does NOT ensure **atomicity**. `count++` on a volatile variable is still not thread-safe because it's read-modify-write (3 operations).

Use `volatile` for: flags, status variables, simple read/write scenarios.
Use `AtomicInteger` or `synchronized` for: compound operations (increment, compare-and-swap).

---

**Q17: Explain the `Phaser`, `CountDownLatch`, `CyclicBarrier`, and `Semaphore`.**

**A:**
- **CountDownLatch**: One-time barrier. N threads count down, waiting threads proceed when count reaches 0. Cannot be reused.
- **CyclicBarrier**: Reusable barrier. N threads wait for each other at a barrier point, then all proceed simultaneously.
- **Semaphore**: Controls access to a resource pool. Permits N concurrent accesses. `acquire()` blocks when no permits, `release()` adds a permit.
- **Phaser**: Flexible, reusable synchronizer. Supports multiple phases, dynamic party registration/deregistration. Combines features of CountDownLatch and CyclicBarrier.

---

**Q18: What is type erasure and how does it affect generics?**

**A:** Type erasure is the process by which the Java compiler removes all generic type information at compile time, replacing type parameters with their bounds (or `Object` if unbounded). This means:
- `List<String>` and `List<Integer>` are both just `List` at runtime
- You cannot do `new T()`, `new T[]`, or `instanceof List<String>`
- Bridge methods are generated to maintain polymorphism

This was done for backward compatibility with pre-generics Java code.

---

**Q19: Explain the Fork/Join framework.**

**A:** The Fork/Join framework (Java 7+) is designed for **divide-and-conquer** parallelism:
1. **Fork**: Split a large task into smaller subtasks
2. **Compute**: Process subtasks (recursively fork more if needed)
3. **Join**: Combine results of subtasks

It uses a **work-stealing algorithm** — idle threads steal work from busy threads' queues. Uses `ForkJoinPool` and `RecursiveTask<V>` (returns result) or `RecursiveAction` (no result).

```java
class SumTask extends RecursiveTask<Long> {
    private final int[] array;
    private final int start, end;
    private static final int THRESHOLD = 1000;

    protected Long compute() {
        if (end - start <= THRESHOLD) {
            long sum = 0;
            for (int i = start; i < end; i++) sum += array[i];
            return sum;
        }
        int mid = (start + end) / 2;
        SumTask left = new SumTask(array, start, mid);
        SumTask right = new SumTask(array, mid, end);
        left.fork(); // Async compute
        return right.compute() + left.join(); // Combine
    }
}
```

---

**Q20: What are Virtual Threads and how do they differ from Platform Threads?**

**A:**
| Feature | Platform Threads | Virtual Threads |
|---------|-----------------|-----------------|
| Managed by | OS kernel | JVM |
| Memory | ~1MB stack each | Few KB each |
| Scalability | Thousands max | Millions possible |
| Context switching | Expensive (OS) | Cheap (JVM) |
| Best for | CPU-bound | I/O-bound |
| Blocking | Blocks OS thread | Only blocks virtual thread |
| Introduced | Java 1.0 | Java 21 |

Virtual threads are ideal for server applications handling many concurrent I/O operations (HTTP requests, database queries).

---

**Q21: Explain the `Proxy` pattern and `java.lang.reflect.Proxy`.**

**A:** `java.lang.reflect.Proxy` creates dynamic proxy instances at runtime that implement specified interfaces. It uses an `InvocationHandler` to intercept method calls. This is the foundation for:
- AOP (Aspect-Oriented Programming) in Spring
- Lazy loading in Hibernate
- Transaction management
- Logging/security cross-cutting concerns

```java
UserService proxy = (UserService) Proxy.newProxyInstance(
    UserService.class.getClassLoader(),
    new Class[]{UserService.class},
    (proxyObj, method, args) -> {
        System.out.println("Before: " + method.getName());
        Object result = method.invoke(realService, args);
        System.out.println("After: " + method.getName());
        return result;
    }
);
```

---

**Q22: What is the `ClassLoader` delegation model?**

**A:** Class loading follows the **parent delegation model**:
1. A class loader first delegates to its parent class loader
2. Only if the parent can't find the class does the child try
3. Order: Bootstrap → Extension → Application → Custom

This prevents: duplicate loading, security issues (can't override `java.lang.String`), and ensures consistency.

---

**Q23: Explain weak, soft, and phantom references.**

**A:**
- **Strong reference**: Normal reference (`Object obj = new Object()`). GC never collects it while reachable.
- **Soft reference**: GC collects when memory is low. Good for caches.
- **Weak reference**: GC collects at next GC cycle. Good for metadata maps (like `WeakHashMap`).
- **Phantom reference**: GC enqueues before finalization. Good for cleanup actions. Cannot retrieve the object.

```java
SoftReference<byte[]> cache = new SoftReference<>(new byte[1024 * 1024]);
WeakReference<Object> weak = new WeakReference<>(new Object());
PhantomReference<Object> phantom = new PhantomReference<>(obj, referenceQueue);
```

---

**Q24: What is the `happens-before` relationship?**

**A:** Happens-before is a guarantee in the Java Memory Model that if action A happens-before action B, then:
- A's results are visible to B
- A is ordered before B

Key happens-before rules:
1. **Program order**: Each action in a thread happens-before every subsequent action in that thread
2. **Monitor lock**: Unlock happens-before every subsequent lock of the same monitor
3. **Volatile**: Write to a volatile field happens-before every subsequent read of that field
4. **Thread start**: `Thread.start()` happens-before any action in the started thread
5. **Thread join**: All actions in a thread happen-before `Thread.join()` returns
6. **Transitivity**: If A happens-before B, and B happens-before C, then A happens-before C

---

### Expert Level

**Q25: How does `ConcurrentHashMap` achieve thread-safety without locking the entire map?**

**A:** In Java 8+, `ConcurrentHashMap` uses:
1. **CAS (Compare-And-Swap)** operations for bucket initialization and updates to empty buckets
2. **Synchronized on the first node** of each bucket for updates to occupied buckets
3. **Volatile reads** for get operations — no locking needed for reads
4. **TreeBin synchronization** for tree-structured buckets

This gives much better concurrency than `Hashtable` (which locks the entire map) or `Collections.synchronizedMap()`.

---

**Q26: Explain the String Constant Pool and String interning mechanism.**

**A:** The String Constant Pool (part of the Heap since Java 7) stores unique string literals. When you create a string literal, the JVM checks the pool:
- If found → returns existing reference (no new object)
- If not found → adds to pool and returns reference

`new String("hello")` creates TWO objects: one in the pool (the literal) and one on the heap. `String.intern()` explicitly adds a heap string to the pool and returns the pool reference.

This optimization works because strings are immutable — sharing is safe.

---

**Q27: What is a `ThreadLocal` and when should you use it?**

**A:** `ThreadLocal` provides thread-local variables — each thread has its own independent copy. No synchronization needed.

```java
private static final ThreadLocal<SimpleDateFormat> dateFormat =
    ThreadLocal.withInitial(() -> new SimpleDateFormat("yyyy-MM-dd"));

// Each thread gets its own SimpleDateFormat instance
String formatted = dateFormat.get().format(new Date());
```

Use cases: Per-request context (user info, transaction), non-thread-safe objects (SimpleDateFormat).
**Warning**: Always clean up (`remove()`) to prevent memory leaks, especially with thread pools.

---

**Q28: Explain how Java achieves "Write Once, Run Anywhere" at the bytecode level.**

**A:**
1. `javac` compiles `.java` → `.class` (bytecode)
2. Bytecode is a standardized instruction set for the JVM (stack-based)
3. Each platform has its own JVM implementation that translates bytecode to native instructions
4. The JIT (Just-In-Time) compiler optimizes hot bytecode into native code at runtime
5. The class file format is standardized — any compliant JVM can execute any valid class file

---

**Q29: What is the difference between deep copy and shallow copy?**

**A:**
- **Shallow copy**: Copies the object's fields as-is. Reference fields still point to the same objects. Change in nested object affects both copies.
- **Deep copy**: Recursively copies all objects. Each copy is completely independent.

```java
// Shallow copy (using clone)
@Override
protected Object clone() throws CloneNotSupportedException {
    return super.clone(); // Only copies top-level fields
}

// Deep copy
@Override
protected Employee clone() throws CloneNotSupportedException {
    Employee cloned = (Employee) super.clone();
    cloned.address = new Address(this.address); // Deep copy nested object
    return cloned;
}
```

---

**Q30: Explain the `ServiceLoader` mechanism in Java.**

**A:** `ServiceLoader` is Java's built-in SPI (Service Provider Interface) mechanism for discovering and loading service implementations at runtime.

```java
// 1. Define service interface
public interface PaymentProcessor { void process(Payment p); }

// 2. Create implementation
public class StripeProcessor implements PaymentProcessor { ... }

// 3. Register in META-INF/services/com.example.PaymentProcessor
com.example.StripeProcessor

// 4. Load dynamically
ServiceLoader<PaymentProcessor> loader = ServiceLoader.load(PaymentProcessor.class);
for (PaymentProcessor processor : loader) {
    processor.process(payment);
}
```

---

**Q31: How does the JIT compiler optimize Java code?**

**A:** The JIT (Just-In-Time) compiler optimizes bytecode to native code at runtime:
1. **Method inlining**: Replaces method calls with the method body
2. **Loop unrolling**: Reduces loop overhead by expanding iterations
3. **Dead code elimination**: Removes unreachable code
4. **Escape analysis**: If an object doesn't escape a method, allocate it on the stack instead of heap
5. **Devirtualization**: Converts virtual method calls to direct calls when only one implementation exists
6. **Tiered compilation**: C1 (quick compilation, basic optimizations) → C2 (slow compilation, aggressive optimizations)

Hotspot identifies "hot" methods (frequently executed) and optimizes them.

---

**Q32: What are Sealed Interfaces and when would you use them?**

**A:** Sealed interfaces (Java 17+) restrict which classes/interfaces can implement them. Combined with pattern matching, they enable exhaustive type checking:

```java
public sealed interface Result<T> permits Success, Failure {
    record Success<T>(T value) implements Result<T> {}
    record Failure<T>(Exception error) implements Result<T> {}
}

// Exhaustive switch — compiler ensures all cases are covered
return switch (result) {
    case Success<String> s -> "Got: " + s.value();
    case Failure<String> f -> "Error: " + f.error().getMessage();
};
```

---

**Q33: Explain Class Data Sharing (CDS) and AppCDS.**

**A:** CDS pre-processes class metadata into a shared archive that multiple JVM instances can memory-map. **Benefits**: Faster startup, reduced memory footprint. AppCDS (Java 10+) extends this to application classes. Very important for microservices and containerized deployments where startup time matters.

---

**Q34: What is the "Initialization-on-demand holder" idiom for singletons?**

**A:** It's a lazy, thread-safe singleton pattern without synchronization:

```java
public class Singleton {
    private Singleton() {}

    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

The inner class is not loaded until `getInstance()` is called. Class loading is thread-safe by the JLS. No synchronization, no volatile, lazy initialization.

---

**Q35 to Q55: Rapid-Fire Questions**

**Q35: Can we override static methods?**
No. Static methods are resolved at compile time based on the reference type, not the object type. You can "hide" a static method in a subclass, but it's not overriding (no polymorphism).

**Q36: What is the purpose of the `transient` keyword?**
Marks a field to be excluded from serialization. The field will have its default value after deserialization.

**Q37: Difference between `throw` and `throws`?**
`throw` is used to explicitly throw an exception. `throws` is used in a method signature to declare that the method may throw an exception.

**Q38: What is a marker interface?**
An interface with no methods (e.g., `Serializable`, `Cloneable`). It "marks" a class as having a certain capability. Modern approach: use annotations instead.

**Q39: Can a constructor be `final`?**
No. Constructors are not inherited, so `final` makes no sense for them.

**Q40: What is covariant return type?**
A subclass method can override a parent method and return a more specific (subtype) return type.

**Q41: What is the difference between `Iterable` and `Iterator`?**
`Iterable` is an interface that provides an `Iterator` via `iterator()` method. It enables for-each loop. `Iterator` has `hasNext()`, `next()`, and `remove()` methods.

**Q42: Why is `String` immutable in Java?**
Security (used in class loading, networking), thread safety (can be shared without synchronization), hashCode caching (consistent use in HashMap), and String Pool optimization.

**Q43: What is the difference between `Path.of()` and `Paths.get()`?**
They do the same thing. `Path.of()` was introduced in Java 11 and is preferred. `Paths.get()` delegates to `Path.of()` internally.

**Q44: Explain `finalize()`, `finally`, and `final`.**
- `final`: keyword for constants, preventing inheritance/overriding
- `finally`: block that always executes after try/catch
- `finalize()`: deprecated method called by GC before collecting an object (DON'T USE)

**Q45: What is a `FunctionalInterface`?**
An interface with exactly one abstract method. Can have multiple default/static methods. Can be annotated with `@FunctionalInterface` (optional but recommended). Enables lambda expression usage.

**Q46: What is the difference between `Stream.of()` and `Arrays.stream()`?**
For arrays, `Arrays.stream(int[])` creates an `IntStream`, while `Stream.of(int[])` creates a `Stream<int[]>` (single element). For object arrays, both work similarly.

**Q47: What is `try-with-resources` and what interface must resources implement?**
Automatic resource management. Resources must implement `AutoCloseable` (or `Closeable`). They are closed in reverse order of declaration, even if exceptions occur.

**Q48: What is `record` in Java?**
A compact syntax for immutable data classes (Java 16+). Auto-generates constructor, getters (`name()` not `getName()`), `equals()`, `hashCode()`, and `toString()`. Cannot extend other classes (implicitly extends `Record`).

**Q49: What is `StackWalker` (Java 9+)?**
A modern, efficient alternative to `Thread.getStackTrace()`. Supports lazy traversal and filtering of stack frames.

**Q50: What are `Scoped Values` (Java 21 Preview)?**
A modern alternative to `ThreadLocal` designed for virtual threads. Immutable, automatically cleaned up, and safely inherited by child threads. Will likely replace `ThreadLocal` for many use cases.

**Q51: Explain the `@Override` annotation's importance.**
It tells the compiler "I intend to override a method." If the method doesn't actually override anything (typo, wrong parameters), the compiler generates an error. Catches bugs at compile time.

**Q52: What is double-checked locking and why is it broken without `volatile`?**
It's a pattern for lazy initialization of singletons with minimal synchronization. Without `volatile`, due to instruction reordering, a thread might see a partially constructed object. The `volatile` keyword prevents this by establishing a happens-before relationship.

**Q53: Explain the `Spliterator` interface.**
`Spliterator` (Java 8) is a "splitting iterator" for parallel traversal. It can split itself into two halves for parallel processing. `Stream`s use `Spliterator` under the hood for parallel operations.

**Q54: What is the difference between `peek()` and `map()` in streams?**
`peek()` returns `Stream<T>` (same type), takes `Consumer<T>` (for side effects). `map()` returns `Stream<R>` (potentially different type), takes `Function<T, R>` (for transformation). `peek()` should only be used for debugging.

**Q55: What are `Gatherers` (Java 22+)?**
`Gatherers` are a new intermediate stream operation that allows custom stateful transformations. They generalize `Collector` for intermediate operations, enabling things like windowing, folding, and custom scan operations that weren't possible with the existing stream API.

---

## 📚 References & Further Reading

- [Official Java Documentation](https://docs.oracle.com/en/java/)
- [Java Language Specification (JLS)](https://docs.oracle.com/javase/specs/)
- [Effective Java by Joshua Bloch](https://www.oreilly.com/library/view/effective-java/9780134686097/)
- [Java Concurrency in Practice by Brian Goetz](https://jcip.net/)
- [OpenJDK Source Code](https://github.com/openjdk/jdk)
- [Baeldung Java Tutorials](https://www.baeldung.com/)
- [Java Design Patterns](https://java-design-patterns.com/)

---

> **Next Topic:** [02 - Maven →](../02-maven/README.md)
