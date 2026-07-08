# ⚙️ Gradle — Complete In-Depth Guide (Beginner to Expert)

> **"Gradle combines the power and flexibility of Ant with the dependency management and conventions of Maven into a more effective build tool."**

---

## 📑 Table of Contents

1. [Introduction & Overview](#1-introduction--overview)
2. [Installation & Setup](#2-installation--setup)
3. [Gradle Architecture & Core Concepts](#3-gradle-architecture--core-concepts)
4. [Build Scripts: Groovy DSL vs Kotlin DSL](#4-build-scripts-groovy-dsl-vs-kotlin-dsl)
5. [Tasks](#5-tasks)
6. [Dependency Management](#6-dependency-management)
7. [Plugins](#7-plugins)
8. [Java Plugin & Spring Boot Plugin](#8-java-plugin--spring-boot-plugin)
9. [Multi-Project Builds](#9-multi-project-builds)
10. [Build Lifecycle & Task Graph](#10-build-lifecycle--task-graph)
11. [Gradle Wrapper](#11-gradle-wrapper)
12. [Performance Optimization](#12-performance-optimization)
13. [Testing with Gradle](#13-testing-with-gradle)
14. [Publishing & Distribution](#14-publishing--distribution)
15. [Gradle vs Maven](#15-gradle-vs-maven)
16. [Best Practices](#16-best-practices)
17. [Common Mistakes & Pitfalls](#17-common-mistakes--pitfalls)
18. [Interview Questions & Answers (50+)](#18-interview-questions--answers-50)

---

## 1. Introduction & Overview

### What is Gradle?

Gradle is a **build automation tool** that uses a **Groovy or Kotlin DSL** (Domain-Specific Language) instead of XML. It combines the best of Apache Ant (flexibility) and Apache Maven (conventions) into a modern, high-performance build system.

### Key Features

- **Groovy/Kotlin DSL** — Expressive, programmable build scripts (not verbose XML)
- **Incremental builds** — Only rebuilds what changed
- **Build cache** — Reuses outputs from previous builds (even across machines)
- **Daemon process** — Stays in memory for fast subsequent builds
- **Parallel execution** — Builds independent modules concurrently
- **Convention over configuration** — Sensible defaults, override when needed
- **Rich plugin ecosystem** — Hundreds of official and community plugins
- **Composite builds** — Combine multiple independent builds

### Who Uses Gradle?

- **Android** — Official build system (required)
- **Spring Boot** — Fully supported (Spring Initializr offers Gradle option)
- **Netflix, LinkedIn, Uber** — Large-scale enterprise adoption
- **Kotlin projects** — Natural fit with Kotlin DSL
- **Multi-language projects** — Supports Java, Kotlin, Groovy, Scala, C/C++

---

## 2. Installation & Setup

### Installation

```bash
# macOS (Homebrew)
brew install gradle

# Windows (Chocolatey)
choco install gradle

# Linux (SDKMAN — recommended)
sdk install gradle

# Manual Installation
# 1. Download from https://gradle.org/releases/
# 2. Extract to /opt/gradle
# 3. Set environment variables:
export GRADLE_HOME=/opt/gradle
export PATH=$GRADLE_HOME/bin:$PATH
```

### Verify Installation

```bash
gradle --version
# Gradle 8.7
# Build time: 2024-03-22
# Kotlin:     1.9.22
# Groovy:     3.0.17
# JVM:        21.0.2 (Oracle)
```

### Initialize a New Project

```bash
# Interactive project initialization
gradle init

# Select type of project:
# 1: basic
# 2: application
# 3: library
# 4: Gradle plugin

# Select implementation language:
# 1: Java
# 2: Kotlin
# 3: Groovy
# 4: Scala
# 5: Swift
# 6: C++

# Select build script DSL:
# 1: Kotlin
# 2: Groovy
```

### Generated Project Structure

```
my-project/
├── gradle/
│   └── wrapper/
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
├── gradlew                    # Unix wrapper script
├── gradlew.bat                # Windows wrapper script
├── settings.gradle            # Settings (project name, modules)
├── build.gradle               # Build configuration
└── src/
    ├── main/
    │   ├── java/
    │   └── resources/
    └── test/
        ├── java/
        └── resources/
```

---

## 3. Gradle Architecture & Core Concepts

### Architecture

```
┌─────────────────────────────────────────────┐
│               Gradle Build                   │
│                                              │
│  ┌────────────────────────────────────────┐  │
│  │         Build Lifecycle                │  │
│  │                                        │  │
│  │  1. Initialization Phase               │  │
│  │     └─ settings.gradle                 │  │
│  │     └─ Determine which projects        │  │
│  │                                        │  │
│  │  2. Configuration Phase                │  │
│  │     └─ build.gradle for each project   │  │
│  │     └─ Build task dependency graph     │  │
│  │                                        │  │
│  │  3. Execution Phase                    │  │
│  │     └─ Execute requested tasks         │  │
│  │     └─ Follow dependency graph         │  │
│  └────────────────────────────────────────┘  │
│                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  Tasks   │  │ Plugins  │  │  Deps    │  │
│  │          │  │          │  │          │  │
│  │ compile  │  │ java     │  │ Maven    │  │
│  │ test     │  │ spring   │  │ Central  │  │
│  │ jar      │  │ docker   │  │ Local    │  │
│  │ custom   │  │ custom   │  │ Custom   │  │
│  └──────────┘  └──────────┘  └──────────┘  │
└─────────────────────────────────────────────┘
```

### Key Concepts

| Concept | Description |
|---------|-------------|
| **Project** | A thing to be built (maps to a `build.gradle` file) |
| **Task** | A single unit of work (compile, test, jar, etc.) |
| **Plugin** | Adds tasks, configurations, and conventions to a project |
| **Configuration** | A named set of dependencies (implementation, testImplementation) |
| **Dependency** | A library your project needs |
| **Repository** | A source for dependencies (Maven Central, Jcenter, custom) |
| **Build Script** | `build.gradle` (Groovy) or `build.gradle.kts` (Kotlin) |
| **Settings Script** | `settings.gradle` defines which projects are in the build |

### Three Build Phases

1. **Initialization** — Gradle determines which projects are part of the build (reads `settings.gradle`)
2. **Configuration** — Gradle executes build scripts for all projects, creating and configuring tasks (builds the task DAG)
3. **Execution** — Gradle executes the selected tasks based on the command line arguments and the task dependency graph

```groovy
// This code runs during CONFIGURATION phase
println "This is executed during configuration"

task myTask {
    // This block runs during CONFIGURATION phase
    println "Configuring myTask"
    
    doFirst {
        // This runs during EXECUTION phase (before task action)
        println "doFirst: about to execute"
    }
    
    doLast {
        // This runs during EXECUTION phase (the task action)
        println "doLast: task is executing"
    }
}
```

---

## 4. Build Scripts: Groovy DSL vs Kotlin DSL

### Groovy DSL (`build.gradle`)

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.0'
    id 'io.spring.dependency-management' version '1.1.5'
}

group = 'com.example'
version = '1.0.0-SNAPSHOT'

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    
    runtimeOnly 'com.mysql:mysql-connector-j'
    
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

### Kotlin DSL (`build.gradle.kts`)

```kotlin
plugins {
    java
    id("org.springframework.boot") version "3.3.0"
    id("io.spring.dependency-management") version "1.1.5"
}

group = "com.example"
version = "1.0.0-SNAPSHOT"

java {
    sourceCompatibility = JavaVersion.VERSION_21
    targetCompatibility = JavaVersion.VERSION_21
}

repositories {
    mavenCentral()
}

dependencies {
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    
    runtimeOnly("com.mysql:mysql-connector-j")
    
    compileOnly("org.projectlombok:lombok")
    annotationProcessor("org.projectlombok:lombok")
    
    testImplementation("org.springframework.boot:spring-boot-starter-test")
}

tasks.withType<Test> {
    useJUnitPlatform()
}
```

### Groovy vs Kotlin DSL Comparison

| Feature | Groovy DSL | Kotlin DSL |
|---------|-----------|------------|
| File extension | `.gradle` | `.gradle.kts` |
| Strings | Single or double quotes | Double quotes only |
| IDE support | Good | Excellent (auto-complete, refactoring) |
| Type safety | Dynamic (runtime errors) | Static (compile-time errors) |
| Learning curve | Lower | Higher (Kotlin knowledge needed) |
| Community | Larger (older) | Growing (recommended for new projects) |
| Syntax | `id 'java'` | `id("java")` or `java` |

---

## 5. Tasks

### Built-in Tasks (from Java Plugin)

```bash
# List all tasks
gradle tasks
gradle tasks --all  # Including hidden tasks

# Common tasks
gradle build         # Full build (compile + test + jar)
gradle clean         # Delete build directory
gradle test          # Run unit tests
gradle jar           # Create JAR
gradle compileJava   # Compile Java sources
gradle bootRun       # Run Spring Boot application
gradle bootJar       # Create executable JAR
gradle dependencies  # Show dependency tree
```

### Custom Tasks

```groovy
// Simple task
task hello {
    description = 'Prints a greeting'
    group = 'custom'
    
    doLast {
        println 'Hello, Gradle!'
    }
}

// Task with inputs and outputs (enables incremental builds)
task generateConfig(type: Copy) {
    from 'src/main/templates'
    into "$buildDir/generated"
    expand(
        version: project.version,
        buildTime: new Date().format('yyyy-MM-dd HH:mm:ss')
    )
}

// Task with dependencies
task fullBuild {
    dependsOn 'clean', 'build', 'generateDocs'
    
    doLast {
        println 'Full build completed!'
    }
}

// Task ordering (without dependency)
task taskA {
    doLast { println 'Task A' }
}

task taskB {
    mustRunAfter taskA  // If both run, B runs after A
    // shouldRunAfter taskA  // Weaker ordering hint
    doLast { println 'Task B' }
}

// Typed tasks
task copyDocs(type: Copy) {
    from 'docs/'
    into "$buildDir/docs"
    include '**/*.md'
    rename { filename ->
        filename.replace('.md', '.txt')
    }
}

task createArchive(type: Zip) {
    archiveFileName = "my-app-${version}.zip"
    destinationDirectory = file("$buildDir/distributions")
    from "$buildDir/libs"
    from 'config/'
}

task runApp(type: JavaExec) {
    mainClass = 'com.example.Application'
    classpath = sourceSets.main.runtimeClasspath
    args = ['--server.port=8080']
    jvmArgs = ['-Xmx512m', '-Dspring.profiles.active=dev']
}

// Task with conditions
task conditionalTask {
    onlyIf { project.hasProperty('runConditional') }
    doLast { println 'This only runs when -PrunConditional is set' }
}

// Kotlin DSL equivalent
tasks.register("hello") {
    description = "Prints a greeting"
    group = "custom"
    doLast {
        println("Hello, Gradle!")
    }
}
```

### Task Types Reference

| Task Type | Purpose |
|-----------|---------|
| `Copy` | Copy files |
| `Delete` | Delete files/directories |
| `Zip`, `Tar`, `Jar`, `War` | Create archives |
| `JavaExec` | Run a Java application |
| `Exec` | Run any command |
| `JavaCompile` | Compile Java sources |
| `Test` | Run tests |
| `ProcessResources` | Process resource files |

---

## 6. Dependency Management

### Configurations (Scopes)

```groovy
dependencies {
    // api — exposed to consumers (transitive)
    // Only available with 'java-library' plugin
    api 'com.google.guava:guava:32.1.3-jre'
    
    // implementation — internal dependency (NOT exposed to consumers)
    implementation 'org.springframework.boot:spring-boot-starter-web'
    
    // compileOnly — compile time only, NOT at runtime (like Maven 'provided')
    compileOnly 'org.projectlombok:lombok'
    
    // runtimeOnly — runtime only, NOT at compile time (like Maven 'runtime')
    runtimeOnly 'com.mysql:mysql-connector-j'
    
    // annotationProcessor — annotation processing only
    annotationProcessor 'org.projectlombok:lombok'
    
    // testImplementation — test compile + test runtime
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.2'
    
    // testRuntimeOnly — test runtime only
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
    
    // testCompileOnly — test compile time only
    testCompileOnly 'org.projectlombok:lombok'
}
```

### Configuration Hierarchy

```
api                  → implementation
                           │
                     compileOnly   runtimeOnly
                           │           │
                    testImplementation
                           │
                  testCompileOnly  testRuntimeOnly
```

### `api` vs `implementation`

```groovy
// java-library plugin required for 'api'
plugins {
    id 'java-library'
}

dependencies {
    // api: Consumer of this library can use Guava directly
    // Guava appears on consumer's compile classpath
    api 'com.google.guava:guava:32.1.3-jre'
    
    // implementation: Consumer CANNOT use Jackson directly
    // Jackson is only on this project's compile classpath
    implementation 'com.fasterxml.jackson.core:jackson-databind:2.17.0'
}

// Rule of thumb:
// Use 'api' if the dependency is part of your public API (method signatures, return types)
// Use 'implementation' for everything else (internal use only)
// Prefer 'implementation' — it reduces recompilation and compile classpath size
```

### Dependency Declaration Formats

```groovy
dependencies {
    // String notation (most common)
    implementation 'group:artifact:version'
    implementation 'org.springframework.boot:spring-boot-starter-web:3.3.0'
    
    // Map notation
    implementation group: 'org.springframework.boot',
                   name: 'spring-boot-starter-web',
                   version: '3.3.0'
    
    // File dependencies (local JARs)
    implementation files('libs/my-custom-lib.jar')
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    
    // Project dependencies (multi-module)
    implementation project(':common')
    implementation project(':service')
    
    // Platform/BOM (version management)
    implementation platform('org.springframework.boot:spring-boot-dependencies:3.3.0')
    implementation 'org.springframework.boot:spring-boot-starter-web' // No version needed!
    
    // Enforced platform (overrides transitive versions)
    implementation enforcedPlatform('org.springframework.boot:spring-boot-dependencies:3.3.0')
}
```

### Version Catalogs (Gradle 7.0+, Recommended)

```toml
# gradle/libs.versions.toml (TOML format)
[versions]
spring-boot = "3.3.0"
spring-dependency-management = "1.1.5"
lombok = "1.18.32"
mapstruct = "1.5.5.Final"
junit = "5.10.2"

[libraries]
spring-boot-starter-web = { module = "org.springframework.boot:spring-boot-starter-web" }
spring-boot-starter-data-jpa = { module = "org.springframework.boot:spring-boot-starter-data-jpa" }
spring-boot-starter-test = { module = "org.springframework.boot:spring-boot-starter-test" }
spring-boot-starter-security = { module = "org.springframework.boot:spring-boot-starter-security" }
mysql-connector = { module = "com.mysql:mysql-connector-j" }
lombok = { module = "org.projectlombok:lombok", version.ref = "lombok" }
mapstruct = { module = "org.mapstruct:mapstruct", version.ref = "mapstruct" }
mapstruct-processor = { module = "org.mapstruct:mapstruct-processor", version.ref = "mapstruct" }
junit-jupiter = { module = "org.junit.jupiter:junit-jupiter", version.ref = "junit" }

[bundles]
spring-web = ["spring-boot-starter-web", "spring-boot-starter-security"]
testing = ["spring-boot-starter-test", "junit-jupiter"]

[plugins]
spring-boot = { id = "org.springframework.boot", version.ref = "spring-boot" }
spring-dependency-management = { id = "io.spring.dependency-management", version.ref = "spring-dependency-management" }
```

```groovy
// build.gradle — Using version catalog
plugins {
    alias(libs.plugins.spring.boot)
    alias(libs.plugins.spring.dependency.management)
}

dependencies {
    implementation libs.spring.boot.starter.web
    implementation libs.spring.boot.starter.data.jpa
    implementation libs.bundles.spring.web  // Use a bundle
    runtimeOnly libs.mysql.connector
    compileOnly libs.lombok
    annotationProcessor libs.lombok
    testImplementation libs.bundles.testing  // Use a bundle
}
```

### Excluding Transitive Dependencies

```groovy
dependencies {
    // Exclude specific transitive dependency
    implementation('org.springframework.boot:spring-boot-starter-web') {
        exclude group: 'org.springframework.boot', module: 'spring-boot-starter-tomcat'
    }
    
    // Exclude from all configurations
    configurations.all {
        exclude group: 'commons-logging', module: 'commons-logging'
    }
    
    // Force a specific version
    implementation('com.google.guava:guava') {
        version {
            strictly '32.1.3-jre' // Fails build if conflicting version
        }
    }
}
```

### Viewing Dependencies

```bash
# Full dependency tree
gradle dependencies

# Dependencies for specific configuration
gradle dependencies --configuration implementation
gradle dependencies --configuration runtimeClasspath

# Dependency insight (why was this version chosen?)
gradle dependencyInsight --dependency spring-core
gradle dependencyInsight --dependency slf4j --configuration compileClasspath
```

---

## 7. Plugins

### Applying Plugins

```groovy
// Plugins DSL (preferred — Groovy)
plugins {
    id 'java'                                           // Core plugin
    id 'java-library'                                   // Core plugin
    id 'application'                                    // Core plugin
    id 'org.springframework.boot' version '3.3.0'      // Community plugin
    id 'io.spring.dependency-management' version '1.1.5'
    id 'com.google.cloud.tools.jib' version '3.4.1'
}

// Kotlin DSL
plugins {
    java
    `java-library`
    application
    id("org.springframework.boot") version "3.3.0"
}

// Legacy apply syntax (avoid for new projects)
apply plugin: 'java'
apply plugin: 'org.springframework.boot'
```

### Important Plugins

| Plugin | Purpose |
|--------|---------|
| `java` | Compiles Java, runs tests, creates JAR |
| `java-library` | Like java, but adds `api` configuration |
| `application` | Adds `run` task, distribution creation |
| `org.springframework.boot` | Boot JAR, run, buildImage |
| `io.spring.dependency-management` | Maven-like BOM import |
| `com.google.cloud.tools.jib` | Docker image without Dockerfile |
| `jacoco` | Code coverage reports |
| `checkstyle` | Code style checking |
| `pmd` | Static code analysis |
| `com.diffplug.spotless` | Code formatting |
| `com.github.ben-manes.versions` | Dependency update checking |

### What the Java Plugin Provides

```
Tasks:
  compileJava           — Compile src/main/java
  processResources      — Copy src/main/resources to build/
  classes               — compileJava + processResources
  compileTestJava       — Compile src/test/java
  processTestResources  — Copy src/test/resources to build/
  testClasses           — compileTestJava + processTestResources
  jar                   — Create JAR from classes
  test                  — Run unit tests
  javadoc               — Generate Javadoc
  clean                 — Delete build/ directory

Source Sets:
  main — src/main/java, src/main/resources
  test — src/test/java, src/test/resources

Configurations:
  implementation, compileOnly, runtimeOnly
  testImplementation, testCompileOnly, testRuntimeOnly
  annotationProcessor
```

---

## 8. Java Plugin & Spring Boot Plugin

### Spring Boot Plugin Configuration

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.3.0'
    id 'io.spring.dependency-management' version '1.1.5'
}

group = 'com.example'
version = '1.0.0-SNAPSHOT'

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21) // Use Java 21
    }
}

springBoot {
    buildInfo()  // Generate build-info.properties for /actuator/info
    mainClass = 'com.example.Application'
}

bootJar {
    archiveFileName = 'my-app.jar'
    layered {
        enabled = true  // Enable layered JARs for Docker optimization
    }
}

bootRun {
    jvmArgs = ['-Dspring.profiles.active=dev', '-Xmx512m']
    args = ['--server.port=8080']
    // Pass system properties
    systemProperty 'spring.profiles.active', findProperty('profile') ?: 'dev'
}

// Spring Boot dependency management (versions from BOM)
dependencyManagement {
    imports {
        mavenBom "org.springframework.cloud:spring-cloud-dependencies:2023.0.1"
    }
}

repositories {
    mavenCentral()
    maven { url 'https://repo.spring.io/milestone' }
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
    implementation 'org.springframework.boot:spring-boot-starter-validation'
    
    // Spring Cloud (version from BOM)
    implementation 'org.springframework.cloud:spring-cloud-starter-netflix-eureka-client'
    
    runtimeOnly 'com.mysql:mysql-connector-j'
    
    compileOnly 'org.projectlombok:lombok'
    annotationProcessor 'org.projectlombok:lombok'
    
    developmentOnly 'org.springframework.boot:spring-boot-devtools'
    
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

### Spring Boot Gradle Commands

```bash
# Run the application
gradle bootRun
gradle bootRun --args='--server.port=9090'
gradle bootRun -Pprofile=prod

# Build executable JAR
gradle bootJar

# Build Docker image (using Buildpacks)
gradle bootBuildImage

# Custom image name
gradle bootBuildImage --imageName=myregistry/my-app:latest
```

---

## 9. Multi-Project Builds

### Structure

```
my-app/
├── settings.gradle             # Root settings
├── build.gradle                # Root build script
├── gradle/
│   └── libs.versions.toml      # Version catalog
├── common/
│   ├── build.gradle
│   └── src/main/java/
├── api/
│   ├── build.gradle
│   └── src/main/java/
├── service/
│   ├── build.gradle
│   └── src/main/java/
└── persistence/
    ├── build.gradle
    └── src/main/java/
```

### Settings File

```groovy
// settings.gradle
rootProject.name = 'my-app'

include 'common'
include 'persistence'
include 'service'
include 'api'
```

### Root Build Script

```groovy
// build.gradle (root)
plugins {
    id 'java' apply false  // Don't apply to root
    id 'org.springframework.boot' version '3.3.0' apply false
    id 'io.spring.dependency-management' version '1.1.5' apply false
}

// Configuration shared across ALL sub-projects
allprojects {
    group = 'com.example'
    version = '1.0.0-SNAPSHOT'
    
    repositories {
        mavenCentral()
    }
}

// Configuration shared across ALL sub-projects (but NOT root)
subprojects {
    apply plugin: 'java'
    apply plugin: 'io.spring.dependency-management'
    
    java {
        sourceCompatibility = JavaVersion.VERSION_21
    }
    
    dependencyManagement {
        imports {
            mavenBom "org.springframework.boot:spring-boot-dependencies:3.3.0"
        }
    }
    
    dependencies {
        testImplementation 'org.springframework.boot:spring-boot-starter-test'
    }
    
    tasks.named('test') {
        useJUnitPlatform()
    }
}
```

### Sub-Project Build Scripts

```groovy
// api/build.gradle
plugins {
    id 'org.springframework.boot'
}

dependencies {
    implementation project(':service')
    implementation project(':common')
    
    implementation 'org.springframework.boot:spring-boot-starter-web'
    implementation 'org.springframework.boot:spring-boot-starter-actuator'
}

bootJar {
    mainClass = 'com.example.api.ApiApplication'
}

// service/build.gradle
dependencies {
    implementation project(':persistence')
    implementation project(':common')
    
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa'
}

// Only create a plain JAR (not an executable Boot JAR)
// This is a library module, not a runnable application
jar {
    enabled = true
}

// persistence/build.gradle
plugins {
    id 'java-library'  // Exposes 'api' configuration
}

dependencies {
    api project(':common')  // Exposed to consumers
    
    api 'org.springframework.boot:spring-boot-starter-data-jpa'
    runtimeOnly 'com.mysql:mysql-connector-j'
}

jar {
    enabled = true
}
```

### Building Multi-Project Builds

```bash
# Build everything
gradle build

# Build specific project
gradle :api:build
gradle :service:test

# List projects
gradle projects

# Show dependency tree for a specific project
gradle :api:dependencies
```

---

## 10. Build Lifecycle & Task Graph

### Build Phases in Detail

```groovy
// settings.gradle — runs during INITIALIZATION
println "=== Initialization Phase ==="
rootProject.name = 'demo'

// build.gradle — The script body runs during CONFIGURATION
println "=== Configuration Phase ==="

task example {
    println "Configuring example task" // CONFIGURATION phase
    
    doFirst {
        println "Executing example task (first)" // EXECUTION phase
    }
    
    doLast {
        println "Executing example task (last)" // EXECUTION phase
    }
}

// Output when running 'gradle example':
// === Initialization Phase ===
// === Configuration Phase ===
// Configuring example task
// > Task :example
// Executing example task (first)
// Executing example task (last)
```

### Task Dependencies and Ordering

```groovy
// Task dependency graph
task compile {
    doLast { println 'Compiling...' }
}

task test {
    dependsOn compile
    doLast { println 'Testing...' }
}

task packageJar {
    dependsOn test
    doLast { println 'Packaging...' }
}

task deploy {
    dependsOn packageJar
    doLast { println 'Deploying...' }
}

// Graph: compile → test → packageJar → deploy
// Running 'gradle deploy' executes all four in order
```

### Task Rules

```groovy
// Dynamic task creation based on a pattern
tasks.addRule("Pattern: ping<Server>") { String taskName ->
    if (taskName.startsWith("ping")) {
        task(taskName) {
            doLast {
                def server = taskName - "ping"
                println "Pinging server: $server"
            }
        }
    }
}

// Now you can run:
// gradle pingProduction
// gradle pingStaging
```

---

## 11. Gradle Wrapper

### Creating the Wrapper

```bash
# Generate wrapper with specific version
gradle wrapper --gradle-version 8.7

# Files created:
# gradlew              — Unix wrapper script
# gradlew.bat          — Windows wrapper script
# gradle/wrapper/
#   gradle-wrapper.jar        — Wrapper bootstrap JAR
#   gradle-wrapper.properties — Wrapper configuration
```

### Wrapper Properties

```properties
# gradle/wrapper/gradle-wrapper.properties
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.7-bin.zip
networkTimeout=10000
validateDistributionUrl=true
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
```

### Using the Wrapper

```bash
# Always use wrapper instead of 'gradle'
./gradlew build          # Unix/Mac
gradlew.bat build        # Windows

# Upgrade wrapper version
./gradlew wrapper --gradle-version 8.8
```

**Always commit wrapper files to version control!**

---

## 12. Performance Optimization

### Gradle Daemon

```properties
# gradle.properties
# The daemon keeps a JVM running in the background
org.gradle.daemon=true

# Maximum memory for the daemon
org.gradle.jvmargs=-Xmx2g -XX:+HeapDumpOnOutOfMemoryError

# Parallel project execution
org.gradle.parallel=true

# Build caching
org.gradle.caching=true

# Configuration cache (Gradle 8+)
org.gradle.configuration-cache=true

# File system watching (detects changes without scanning)
org.gradle.vfs.watch=true

# Number of workers
org.gradle.workers.max=4
```

### Build Cache

```groovy
// settings.gradle
buildCache {
    local {
        directory = new File(rootDir, '.gradle/build-cache')
        removeUnusedEntriesAfterDays = 30
    }
    
    remote(HttpBuildCache) {
        url = 'https://cache.example.com/cache/'
        push = true // Enable pushing to remote cache (CI only)
        credentials {
            username = 'admin'
            password = 'secret'
        }
    }
}
```

### Performance Tips

```bash
# Profile your build
gradle build --profile
# Opens build/reports/profile/profile-*.html

# Build scan (detailed analysis)
gradle build --scan
# Uploads to scans.gradle.com (or Develocity server)

# Only compile changed files
gradle compileJava  # Incremental compilation is automatic

# Skip unnecessary tasks
gradle build -x test            # Skip tests
gradle build -x javadoc         # Skip javadoc
gradle build --no-build-cache   # Disable cache for debugging

# Parallel execution
gradle build --parallel

# Use configuration cache
gradle build --configuration-cache
```

---

## 13. Testing with Gradle

### JUnit 5 Configuration

```groovy
dependencies {
    testImplementation 'org.junit.jupiter:junit-jupiter:5.10.2'
    testImplementation 'org.mockito:mockito-core:5.11.0'
    testImplementation 'org.mockito:mockito-junit-jupiter:5.11.0'
    testImplementation 'org.assertj:assertj-core:3.25.3'
    
    testRuntimeOnly 'org.junit.platform:junit-platform-launcher'
}

tasks.named('test') {
    useJUnitPlatform()
    
    // Test filtering
    filter {
        includeTestsMatching '*Test'
        includeTestsMatching '*Tests'
        excludeTestsMatching '*IntegrationTest'
    }
    
    // JVM settings for tests
    jvmArgs = ['-Xmx512m']
    
    // Enable test logging
    testLogging {
        events 'passed', 'skipped', 'failed'
        showExceptions true
        showCauses true
        showStackTraces true
        exceptionFormat 'full'
    }
    
    // Parallel test execution
    maxParallelForks = Runtime.runtime.availableProcessors().intdiv(2) ?: 1
    
    // Fail on no test sources
    failOnNoMatchingTests = true
    
    // Generate reports
    reports {
        html.required = true
        junitXml.required = true
    }
    
    // Retry flaky tests
    retry {
        maxRetries = 2
        maxFailures = 5
    }
}
```

### Integration Tests (Separate Source Set)

```groovy
// Create a separate source set for integration tests
sourceSets {
    integrationTest {
        java {
            srcDir 'src/integrationTest/java'
        }
        resources {
            srcDir 'src/integrationTest/resources'
        }
        compileClasspath += sourceSets.main.output + sourceSets.test.output
        runtimeClasspath += sourceSets.main.output + sourceSets.test.output
    }
}

configurations {
    integrationTestImplementation.extendsFrom testImplementation
    integrationTestRuntimeOnly.extendsFrom testRuntimeOnly
}

task integrationTest(type: Test) {
    description = 'Runs integration tests.'
    group = 'verification'
    
    testClassesDirs = sourceSets.integrationTest.output.classesDirs
    classpath = sourceSets.integrationTest.runtimeClasspath
    
    useJUnitPlatform()
    
    shouldRunAfter test
}

check.dependsOn integrationTest
```

### JaCoCo (Code Coverage)

```groovy
plugins {
    id 'jacoco'
}

jacoco {
    toolVersion = '0.8.12'
}

test {
    finalizedBy jacocoTestReport
}

jacocoTestReport {
    dependsOn test
    
    reports {
        xml.required = true
        csv.required = false
        html.outputLocation = layout.buildDirectory.dir('reports/jacoco')
    }
}

jacocoTestCoverageVerification {
    violationRules {
        rule {
            limit {
                counter = 'LINE'
                value = 'COVEREDRATIO'
                minimum = 0.80  // 80% line coverage minimum
            }
        }
        rule {
            element = 'CLASS'
            limit {
                counter = 'BRANCH'
                value = 'COVEREDRATIO'
                minimum = 0.70
            }
        }
    }
}

check.dependsOn jacocoTestCoverageVerification
```

---

## 14. Publishing & Distribution

### Publishing to Maven Repository

```groovy
plugins {
    id 'java-library'
    id 'maven-publish'
}

publishing {
    publications {
        mavenJava(MavenPublication) {
            from components.java
            
            groupId = 'com.example'
            artifactId = 'my-library'
            version = '1.0.0'
            
            pom {
                name = 'My Library'
                description = 'A useful library'
                url = 'https://github.com/example/my-library'
                
                licenses {
                    license {
                        name = 'MIT License'
                        url = 'https://opensource.org/licenses/MIT'
                    }
                }
                
                developers {
                    developer {
                        id = 'dilip'
                        name = 'Dilip'
                        email = 'dilip@example.com'
                    }
                }
            }
        }
    }
    
    repositories {
        maven {
            name = 'nexus'
            def releasesRepoUrl = 'https://nexus.example.com/repository/maven-releases/'
            def snapshotsRepoUrl = 'https://nexus.example.com/repository/maven-snapshots/'
            url = version.endsWith('SNAPSHOT') ? snapshotsRepoUrl : releasesRepoUrl
            
            credentials {
                username = project.findProperty('nexusUsername') ?: 'admin'
                password = project.findProperty('nexusPassword') ?: 'admin'
            }
        }
    }
}

// Publish: gradle publish
// Publish to local: gradle publishToMavenLocal
```

---

## 15. Gradle vs Maven

| Feature | Gradle | Maven |
|---------|--------|-------|
| **Configuration** | Groovy/Kotlin DSL | XML |
| **Build speed** | 2-10x faster (daemon, cache) | Slower (no caching) |
| **Incremental builds** | ✅ Built-in | ❌ Not supported |
| **Build cache** | ✅ Local + Remote | ❌ Not native |
| **Parallel builds** | ✅ Native | ✅ (with `-T`) |
| **Flexibility** | High (scripting) | Low (convention-only) |
| **Learning curve** | Higher | Lower |
| **IDE support** | Excellent | Excellent |
| **Android** | ✅ Official | ❌ Not supported |
| **Spring Boot** | ✅ Fully supported | ✅ Fully supported |
| **Multi-module** | Excellent | Good |
| **Dependency mgmt** | Better (conflict resolution, BOMs, version catalogs) | Good |
| **Community** | Growing | Larger |
| **Build scan** | ✅ Built-in | Via plugin |

### When to Choose Gradle

- Performance-critical builds (large projects)
- Complex build logic (custom tasks, conditional logic)
- Android development (required)
- Multi-language projects
- Need for incremental builds and build caching

### When to Choose Maven

- Simpler projects
- Team has Maven experience
- Corporate standardization
- Need XML-based configuration
- Larger community and more online resources

---

## 16. Best Practices

1. **Always use the Gradle Wrapper** — ensures consistent Gradle version
2. **Use Kotlin DSL for new projects** — better IDE support, type safety
3. **Use Version Catalogs** — centralize dependency versions in `libs.versions.toml`
4. **Prefer `implementation` over `api`** — reduces compile classpath, faster builds
5. **Enable build cache** — `org.gradle.caching=true` in `gradle.properties`
6. **Enable parallel execution** — `org.gradle.parallel=true`
7. **Use `buildSrc` or convention plugins** — share build logic across modules
8. **Avoid `allprojects`/`subprojects` blocks** — use convention plugins instead (Gradle 8+)
9. **Pin plugin versions** — always specify versions in `plugins {}` block
10. **Use `tasks.named()` or `tasks.register()`** — lazy task configuration
11. **Avoid `task` keyword** — use `tasks.register()` for lazy configuration
12. **Configure inputs/outputs for custom tasks** — enables caching and incremental builds
13. **Use `./gradlew build --scan`** — identify build performance issues
14. **Organize multi-project builds properly** — use flat or hierarchical structure
15. **Use `gradle.properties`** for build settings — JVM args, parallelism, caching

---

## 17. Common Mistakes & Pitfalls

```groovy
// ❌ MISTAKE 1: Using 'compile' instead of 'implementation'
// 'compile' is deprecated since Gradle 7!
compile 'org.springframework:spring-core:6.1.0' // DON'T use!
// ✅ FIX:
implementation 'org.springframework:spring-core:6.1.0'

// ❌ MISTAKE 2: Eager task creation with 'task' keyword
task myTask { // Creates and configures immediately
    doLast { println 'Hello' }
}
// ✅ FIX: Lazy registration
tasks.register('myTask') {
    doLast { println 'Hello' }
}

// ❌ MISTAKE 3: Putting logic in configuration phase
task buildApp {
    // This runs during CONFIGURATION, not execution!
    def files = fileTree('src').files
    files.each { println it }
    
    // ✅ FIX: Put logic in doLast
    doLast {
        def files = fileTree('src').files
        files.each { println it }
    }
}

// ❌ MISTAKE 4: Not using the Gradle Wrapper
// Running 'gradle build' may use a different version!
// ✅ FIX: Always use './gradlew build'

// ❌ MISTAKE 5: Using 'api' when 'implementation' is sufficient
// 'api' exposes the dependency transitively, slowing compilation
// ✅ FIX: Use 'api' ONLY when the dependency is part of your public API

// ❌ MISTAKE 6: Not configuring test platform
test {
    // Without this, JUnit 5 tests won't be discovered!
    useJUnitPlatform() // ✅ Required for JUnit 5
}

// ❌ MISTAKE 7: Hardcoding versions everywhere
dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web:3.3.0'
    implementation 'org.springframework.boot:spring-boot-starter-data-jpa:3.3.0'
}
// ✅ FIX: Use version catalogs or ext block
ext {
    springBootVersion = '3.3.0'
}
// Or better: use gradle/libs.versions.toml

// ❌ MISTAKE 8: Using allprojects/subprojects excessively
// This couples all projects together
allprojects {
    apply plugin: 'java'
    // What if a project doesn't need Java?
}
// ✅ FIX: Use convention plugins for Gradle 8+
```

---

## 18. Interview Questions & Answers (50+)

### Beginner Level

**Q1: What is Gradle and how does it differ from Maven?**

**A:** Gradle is a build automation tool that uses Groovy or Kotlin DSL instead of XML. Key differences:
- Gradle uses a programming language (not XML) for build configuration
- Gradle has incremental builds, build caching, and a daemon for speed
- Gradle is 2-10x faster than Maven
- Maven has a larger community and simpler learning curve
- Both support Spring Boot and Java projects equally well

---

**Q2: What are the three phases of the Gradle build lifecycle?**

**A:**
1. **Initialization**: Reads `settings.gradle`, determines which projects are in the build
2. **Configuration**: Evaluates `build.gradle` for all projects, builds the task DAG (Directed Acyclic Graph)
3. **Execution**: Runs the requested tasks in dependency order

---

**Q3: What is the Gradle Wrapper and why should you use it?**

**A:** The Gradle Wrapper (`gradlew`/`gradlew.bat`) is a script that downloads and runs a specific version of Gradle. Benefits:
- No need to install Gradle
- Everyone uses the same version (reproducible builds)
- CI/CD doesn't need pre-installed Gradle
- New team members can build immediately

---

**Q4: What is the difference between `implementation` and `api` configurations?**

**A:**
- `implementation`: Internal dependency. NOT exposed to consumers. Consumers can't use it directly. Faster compilation.
- `api`: Public dependency. Exposed to consumers. Consumers can use it. Requires `java-library` plugin.

Rule: Use `implementation` by default. Only use `api` when the dependency is part of your public API.

---

**Q5: How do you add a dependency in Gradle?**

**A:**
```groovy
dependencies {
    implementation 'group:artifact:version'
    implementation 'org.springframework.boot:spring-boot-starter-web:3.3.0'
}
```

---

**Q6: What is a Gradle task?**

**A:** A task is a single unit of work (compile, test, jar, etc.). Tasks can depend on other tasks, forming a DAG. They can be built-in (from plugins) or custom-defined.

---

**Q7: How do you run a Spring Boot application with Gradle?**

**A:** `./gradlew bootRun`

---

**Q8: What is `settings.gradle` used for?**

**A:** It configures the build structure during the initialization phase. It defines:
- Root project name (`rootProject.name`)
- Sub-projects to include (`include 'module-a', 'module-b'`)
- Plugin management
- Build cache configuration

---

### Intermediate Level

**Q9: Explain Gradle's dependency configurations (scopes).**

**A:**
| Configuration | Compile | Test | Runtime | Packaged |
|---------------|---------|------|---------|----------|
| `implementation` | ✅ | ✅ | ✅ | ✅ |
| `api` | ✅ | ✅ | ✅ | ✅ (transitive) |
| `compileOnly` | ✅ | ❌ | ❌ | ❌ |
| `runtimeOnly` | ❌ | ✅ | ✅ | ✅ |
| `testImplementation` | ❌ | ✅ | ❌ | ❌ |
| `annotationProcessor` | Compile only | ❌ | ❌ | ❌ |

---

**Q10: What is incremental build in Gradle?**

**A:** Gradle tracks the inputs and outputs of every task. If neither has changed since the last build, the task is skipped (marked as UP-TO-DATE). This dramatically speeds up builds since only changed parts are rebuilt.

---

**Q11: What is the Gradle build cache?**

**A:** The build cache stores task outputs keyed by their inputs. Even if you clean and rebuild, cached outputs are reused. It supports:
- **Local cache**: Shared across builds on the same machine
- **Remote cache**: Shared across machines and CI/CD (via HTTP)

---

**Q12: What is the Gradle Daemon?**

**A:** The daemon is a long-lived background process that keeps the JVM warm for subsequent builds. Benefits: faster startup, hot JIT compilation, cached class loading. Enabled by default since Gradle 3.0.

---

**Q13: How does Gradle resolve dependency conflicts?**

**A:** Gradle uses the **newest version wins** strategy by default (unlike Maven's nearest-wins). You can customize with:
- `strictly` — fail on conflict
- `prefer` — prefer a version
- `require` — require a specific version
- `reject` — reject certain versions
- Resolution strategy force

---

**Q14: What is a Version Catalog in Gradle?**

**A:** A centralized file (`gradle/libs.versions.toml`) for managing dependency versions. It defines versions, libraries, bundles, and plugins in TOML format. Provides type-safe accessors in build scripts. Available since Gradle 7.0.

---

**Q15: How do you create a custom task in Gradle?**

**A:**
```groovy
tasks.register('myTask') {
    description = 'My custom task'
    group = 'custom'
    inputs.file('input.txt')   // Declare inputs for caching
    outputs.file('output.txt') // Declare outputs for caching
    doLast {
        // Task action
    }
}
```

---

### Advanced Level

**Q16: What is a convention plugin in Gradle?**

**A:** A convention plugin (using `buildSrc` or included builds) allows sharing build logic across multiple sub-projects in a type-safe way. It replaces `allprojects`/`subprojects` blocks with properly isolated, reusable build conventions. This is the recommended approach in Gradle 8+.

---

**Q17: Explain Gradle's configuration avoidance API.**

**A:** The configuration avoidance API (`tasks.register()` vs `task`) defers task creation until the task is actually needed. This speeds up the configuration phase because unused tasks are never configured. Always use `tasks.register()` instead of `task` keyword.

---

**Q18: What is a composite build in Gradle?**

**A:** A composite build combines multiple independent builds. Instead of publishing a library to a repo and consuming it, you can substitute the dependency with a local project. Useful for developing a library and its consumer simultaneously.

```groovy
// settings.gradle
includeBuild '../my-library'
```

---

**Q19: How does Gradle's configuration cache work?**

**A:** The configuration cache serializes the task graph after the configuration phase. On subsequent builds, it skips the configuration phase entirely, loading the cached graph. This dramatically speeds up builds, especially for large projects. Enabled with `org.gradle.configuration-cache=true`.

---

**Q20: What are Gradle's resolution strategies?**

**A:**
```groovy
configurations.all {
    resolutionStrategy {
        force 'com.google.guava:guava:32.1.3-jre' // Force version
        failOnVersionConflict() // Fail build on any conflict
        preferProjectModules() // Prefer project modules over external
        cacheDynamicVersionsFor 10, 'minutes'
        cacheChangingModulesFor 0, 'seconds' // Don't cache SNAPSHOTs
    }
}
```

---

### Rapid-Fire Questions (Q21–Q50)

**Q21: What is `doFirst` and `doLast` in a task?**
`doLast` adds an action to the END of the task's action list. `doFirst` adds an action to the BEGINNING. Multiple `doFirst`/`doLast` can be added.

**Q22: How do you skip a task?**
`gradle build -x test` or using `onlyIf { condition }` in the task definition.

**Q23: What is `buildSrc`?**
A special directory treated as an included build. Code in `buildSrc` is automatically compiled and available to all build scripts. Used for custom tasks, plugins, and shared build logic.

**Q24: How do you view the dependency tree?**
`gradle dependencies` or `gradle :module:dependencies --configuration runtimeClasspath`

**Q25: What is the `plugins {}` block?**
The recommended way to apply plugins. It's a special block processed early in the build lifecycle. Provides better performance and type safety.

**Q26: Difference between `apply plugin:` and `plugins {}`?**
`plugins {}` is the modern approach (type-safe, better performance). `apply plugin:` is legacy (still works but not recommended for new projects).

**Q27: What is `gradle.properties` used for?**
Project-level properties: JVM args, parallelism, caching, custom properties. Also in `~/.gradle/gradle.properties` for user-level settings.

**Q28: How do you create a multi-module project?**
Create `settings.gradle` with `include` for each module. Each module has its own `build.gradle` and can depend on other modules via `project(':module-name')`.

**Q29: What is the `clean` task?**
Deletes the `build/` directory. Added by the `base` plugin (which `java` applies).

**Q30: How do you run tests with Gradle?**
`gradle test` — runs all tests. `gradle test --tests 'MyTestClass'` — runs specific test.

**Q31: What is `sourceSets` in Gradle?**
Defines groups of source files. Default: `main` and `test`. You can add custom source sets for integration tests.

**Q32: What is the difference between `implementation` and `compileOnly`?**
`implementation` is available at compile, test, and runtime. `compileOnly` is only available at compile time (not packaged). Like Maven's `provided` scope.

**Q33: How do you publish a library with Gradle?**
Apply `maven-publish` plugin, configure `publishing { publications { ... } repositories { ... } }`, run `gradle publish`.

**Q34: What is `dependencyInsight`?**
`gradle dependencyInsight --dependency spring-core` — shows why a specific dependency version was chosen (conflict resolution details).

**Q35: What is `ext` block used for?**
Defines extra properties on a project. Used for sharing values across build scripts:
```groovy
ext { springVersion = '6.1.0' }
```

**Q36: How does Gradle handle transitive dependencies?**
Gradle automatically includes transitive dependencies. It uses "newest version wins" for conflicts (unlike Maven's "nearest wins").

**Q37: What is `developmentOnly` configuration?**
Used for Spring Boot DevTools. Dependencies are included in bootRun classpath but not in the packaged JAR.

**Q38: What is a Gradle Build Scan?**
A shareable web-based build report (`--scan`). Shows build time, task execution, dependency resolution, test results, and performance suggestions.

**Q39: How do you force-refresh dependencies?**
`gradle build --refresh-dependencies` — re-downloads all dependencies.

**Q40: What is `enforcedPlatform` vs `platform`?**
`platform` imports version recommendations (can be overridden). `enforcedPlatform` forces versions (cannot be overridden by transitives).

**Q41: What is Gradle's task outcome meanings?**
- `UP-TO-DATE`: Inputs/outputs unchanged, skipped
- `FROM-CACHE`: Output restored from cache
- `NO-SOURCE`: No source files to process
- `SKIPPED`: Task was skipped (condition or -x flag)
- (no label): Task was executed

**Q42: What is `annotationProcessor` configuration?**
For compile-time annotation processing (Lombok, MapStruct). Separated from implementation classpath for better isolation.

**Q43: How do you set Java version in Gradle?**
```groovy
java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}
```

**Q44: What is the `init` task?**
`gradle init` creates a new Gradle project from scratch with templates for different project types.

**Q45: What is a Gradle distribution type (bin vs all)?**
`bin` — contains only the Gradle binary. `all` — includes binary + source code + documentation. Use `all` for IDE support.

**Q46: How do you enable JUnit 5 in Gradle?**
```groovy
test { useJUnitPlatform() }
```

**Q47: What is `flattened` dependency resolution?**
Gradle resolves all dependencies to a flat list, resolving conflicts globally (not just nearest path like Maven).

**Q48: How do you configure proxy in Gradle?**
In `gradle.properties`:
```
systemProp.http.proxyHost=proxy.company.com
systemProp.http.proxyPort=8080
```

**Q49: What is the difference between `bootJar` and `jar` tasks?**
`bootJar` creates an executable JAR with embedded dependencies (from Spring Boot plugin). `jar` creates a standard library JAR without embedded dependencies.

**Q50: How do you debug a Gradle build?**
`gradle build --debug` for verbose logging, `gradle build --stacktrace` for full stack traces, `gradle build --info` for info logging, `gradle build --scan` for build scan.

---

## 📚 References & Further Reading

- [Gradle Official Documentation](https://docs.gradle.org/)
- [Gradle User Manual](https://docs.gradle.org/current/userguide/userguide.html)
- [Gradle Plugin Portal](https://plugins.gradle.org/)
- [Spring Boot Gradle Plugin](https://docs.spring.io/spring-boot/docs/current/gradle-plugin/reference/htmlsingle/)
- [Gradle Performance Guide](https://docs.gradle.org/current/userguide/performance.html)
- [Gradle vs Maven Comparison](https://gradle.org/maven-vs-gradle/)
- [Baeldung Gradle Tutorials](https://www.baeldung.com/gradle)

---

> **Previous Topic:** [← 02 - Maven](../02-maven/README.md)  
> **Next Topic:** [04 - JUnit →](../04-junit/README.md)
