# 🏗️ Apache Maven — Complete In-Depth Guide (Beginner to Expert)

> **"Maven is not just a build tool — it's a project management and comprehension tool that provides a uniform build system, quality project information, and guidelines for best practices development."**

---

## 📑 Table of Contents

1. [Introduction & Overview](#1-introduction--overview)
2. [Installation & Setup](#2-installation--setup)
3. [Maven Architecture & Core Concepts](#3-maven-architecture--core-concepts)
4. [The POM (Project Object Model)](#4-the-pom-project-object-model)
5. [Maven Build Lifecycle](#5-maven-build-lifecycle)
6. [Dependency Management](#6-dependency-management)
7. [Maven Repositories](#7-maven-repositories)
8. [Maven Plugins](#8-maven-plugins)
9. [Profiles & Environment-Specific Builds](#9-profiles--environment-specific-builds)
10. [Multi-Module Projects](#10-multi-module-projects)
11. [Maven Wrapper](#11-maven-wrapper)
12. [BOM (Bill of Materials)](#12-bom-bill-of-materials)
13. [Advanced Maven Features](#13-advanced-maven-features)
14. [Maven vs Gradle](#14-maven-vs-gradle)
15. [Best Practices](#15-best-practices)
16. [Common Mistakes & Pitfalls](#16-common-mistakes--pitfalls)
17. [Interview Questions & Answers (50+)](#17-interview-questions--answers-50)

---

## 1. Introduction & Overview

### What is Maven?

Apache Maven is a **build automation and project management tool** primarily used for Java projects. The name "Maven" comes from Yiddish, meaning **"accumulator of knowledge."**

Maven provides:
- **Build automation** — Compile, test, package, deploy with one command
- **Dependency management** — Automatically downloads and manages libraries
- **Project structure convention** — Standard directory layout
- **Lifecycle management** — Predefined build phases
- **Plugin architecture** — Extensible with thousands of plugins
- **Multi-module support** — Manage complex projects with multiple sub-projects

### Why Maven for Spring Boot?

- Spring Boot projects are typically created with Maven or Gradle
- Spring Initializr generates Maven POM by default
- Maven Central hosts all Spring dependencies
- `spring-boot-starter-parent` provides a curated Maven BOM
- Most Spring Boot tutorials and documentation use Maven

### Convention over Configuration

Maven follows the principle of **"Convention over Configuration"** — if you follow Maven's standard conventions, you need minimal configuration. For example:
- Source code goes in `src/main/java`
- Test code goes in `src/test/java`
- Resources go in `src/main/resources`
- Output goes in `target/`

---

## 2. Installation & Setup

### Prerequisites

- **Java JDK** 8 or later must be installed
- `JAVA_HOME` environment variable must be set

### Installation on Different Platforms

```bash
# Windows (using Chocolatey)
choco install maven

# macOS (using Homebrew)
brew install maven

# Linux (Ubuntu/Debian)
sudo apt install maven

# Linux (RHEL/CentOS)
sudo yum install maven

# Manual Installation (Any OS)
# 1. Download from https://maven.apache.org/download.cgi
# 2. Extract to a directory (e.g., /opt/maven)
# 3. Set environment variables:
export MAVEN_HOME=/opt/maven
export PATH=$MAVEN_HOME/bin:$PATH
```

### Verify Installation

```bash
mvn --version
# Output:
# Apache Maven 3.9.6
# Maven home: /opt/maven
# Java version: 21.0.2, vendor: Oracle
# Default locale: en_US, platform encoding: UTF-8
# OS name: "linux", version: "5.15.0"
```

### Settings File

Maven uses two settings files:

1. **Global settings**: `$MAVEN_HOME/conf/settings.xml`
2. **User settings**: `~/.m2/settings.xml` (overrides global)

```xml
<!-- ~/.m2/settings.xml -->
<settings xmlns="http://maven.apache.org/SETTINGS/1.2.0">
    
    <!-- Local repository location -->
    <localRepository>${user.home}/.m2/repository</localRepository>
    
    <!-- Proxy configuration (corporate networks) -->
    <proxies>
        <proxy>
            <id>company-proxy</id>
            <active>true</active>
            <protocol>https</protocol>
            <host>proxy.company.com</host>
            <port>8080</port>
            <username>proxyuser</username>
            <password>proxypass</password>
            <nonProxyHosts>localhost|*.company.com</nonProxyHosts>
        </proxy>
    </proxies>
    
    <!-- Server credentials (for deploying artifacts) -->
    <servers>
        <server>
            <id>nexus-releases</id>
            <username>admin</username>
            <password>admin123</password>
        </server>
    </servers>
    
    <!-- Mirror configuration -->
    <mirrors>
        <mirror>
            <id>company-nexus</id>
            <mirrorOf>central</mirrorOf>
            <url>https://nexus.company.com/repository/maven-central/</url>
        </mirror>
    </mirrors>
    
    <!-- Active profiles -->
    <activeProfiles>
        <activeProfile>dev</activeProfile>
    </activeProfiles>
    
</settings>
```

---

## 3. Maven Architecture & Core Concepts

### Maven Architecture Diagram

```
┌──────────────────────────────────────────────────────┐
│                    Developer                          │
│                       │                               │
│                 mvn <command>                         │
│                       │                               │
│              ┌────────▼────────┐                     │
│              │   Maven Core    │                     │
│              │  ┌────────────┐ │                     │
│              │  │ Lifecycle  │ │                     │
│              │  │ Manager    │ │                     │
│              │  └────────────┘ │                     │
│              │  ┌────────────┐ │                     │
│              │  │ Dependency │ │                     │
│              │  │ Resolver   │ │                     │
│              │  └────────────┘ │                     │
│              │  ┌────────────┐ │                     │
│              │  │ Plugin     │ │                     │
│              │  │ Manager    │ │                     │
│              │  └────────────┘ │                     │
│              └───────┬─────────┘                     │
│                      │                                │
│       ┌──────────────┼──────────────┐                │
│       ▼              ▼              ▼                │
│  ┌─────────┐  ┌───────────┐  ┌───────────┐         │
│  │  POM    │  │  Plugins  │  │ Repository│         │
│  │(pom.xml)│  │           │  │           │         │
│  │         │  │ compiler  │  │ Local     │         │
│  │ GAV     │  │ surefire  │  │ ~/.m2/    │         │
│  │ deps    │  │ jar       │  │           │         │
│  │ plugins │  │ deploy    │  │ Remote    │         │
│  │ profiles│  │ ...       │  │ Central   │         │
│  └─────────┘  └───────────┘  │ Nexus     │         │
│                              └───────────┘          │
└──────────────────────────────────────────────────────┘
```

### Key Concepts

| Concept | Description |
|---------|-------------|
| **POM** | Project Object Model — XML file that describes the project |
| **GAV** | GroupId:ArtifactId:Version — unique identifier for any project/dependency |
| **Lifecycle** | A sequence of phases that define the build process |
| **Phase** | A step in the build lifecycle (compile, test, package, etc.) |
| **Goal** | A specific task performed by a plugin (e.g., `compiler:compile`) |
| **Plugin** | A component that provides goals to execute during phases |
| **Repository** | A storage location for artifacts (local, remote, central) |
| **Dependency** | A library your project needs (defined in POM) |
| **Archetype** | A project template for creating new projects |

### Standard Directory Structure

```
my-project/
├── pom.xml                          # Project Object Model
├── src/
│   ├── main/
│   │   ├── java/                    # Application source code
│   │   │   └── com/example/
│   │   │       └── App.java
│   │   ├── resources/               # Application resources
│   │   │   ├── application.properties
│   │   │   └── logback.xml
│   │   └── webapp/                  # Web application resources (WAR projects)
│   │       ├── WEB-INF/
│   │       │   └── web.xml
│   │       └── index.jsp
│   └── test/
│       ├── java/                    # Test source code
│       │   └── com/example/
│       │       └── AppTest.java
│       └── resources/               # Test resources
│           └── test-data.json
└── target/                          # Build output (generated, not committed)
    ├── classes/                     # Compiled classes
    ├── test-classes/                # Compiled test classes
    ├── surefire-reports/            # Test reports
    └── my-project-1.0.0.jar        # Final artifact
```

---

## 4. The POM (Project Object Model)

### Minimal POM

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Project coordinates (GAV) -->
    <groupId>com.example</groupId>       <!-- Organization/group -->
    <artifactId>my-project</artifactId>   <!-- Project name -->
    <version>1.0.0-SNAPSHOT</version>     <!-- Version -->
    <packaging>jar</packaging>            <!-- jar, war, pom, ear -->
    
</project>
```

### Complete POM (Spring Boot Project)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    
    <modelVersion>4.0.0</modelVersion>
    
    <!-- Inherit from Spring Boot starter parent -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.3.0</version>
        <relativePath/>
    </parent>
    
    <!-- Project coordinates -->
    <groupId>com.example</groupId>
    <artifactId>my-spring-app</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>
    
    <!-- Project information -->
    <name>My Spring Application</name>
    <description>A sample Spring Boot application</description>
    <url>https://github.com/example/my-spring-app</url>
    
    <licenses>
        <license>
            <name>MIT License</name>
            <url>https://opensource.org/licenses/MIT</url>
        </license>
    </licenses>
    
    <developers>
        <developer>
            <name>Dilip</name>
            <email>dilip@example.com</email>
        </developer>
    </developers>
    
    <!-- Properties (variables you can reuse throughout POM) -->
    <properties>
        <java.version>21</java.version>
        <spring-cloud.version>2023.0.1</spring-cloud.version>
        <mapstruct.version>1.5.5.Final</mapstruct.version>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>
    
    <!-- Dependency Management (version control without adding dependencies) -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
    <!-- Dependencies -->
    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <!-- Database -->
        <dependency>
            <groupId>com.mysql</groupId>
            <artifactId>mysql-connector-j</artifactId>
            <scope>runtime</scope>
        </dependency>
        
        <dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>test</scope>
        </dependency>
        
        <!-- Lombok -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        
        <!-- MapStruct -->
        <dependency>
            <groupId>org.mapstruct</groupId>
            <artifactId>mapstruct</artifactId>
            <version>${mapstruct.version}</version>
        </dependency>
        
        <!-- Testing -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.security</groupId>
            <artifactId>spring-security-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
    
    <!-- Build configuration -->
    <build>
        <plugins>
            <!-- Spring Boot Maven Plugin -->
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
            
            <!-- Maven Compiler Plugin -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <configuration>
                    <source>${java.version}</source>
                    <target>${java.version}</target>
                    <annotationProcessorPaths>
                        <path>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                            <version>${lombok.version}</version>
                        </path>
                        <path>
                            <groupId>org.mapstruct</groupId>
                            <artifactId>mapstruct-processor</artifactId>
                            <version>${mapstruct.version}</version>
                        </path>
                    </annotationProcessorPaths>
                </configuration>
            </plugin>
        </plugins>
    </build>
    
</project>
```

### Super POM

Every POM inherits from a **Super POM** (Maven's default POM). It defines:
- Default repository (Maven Central)
- Default plugin versions
- Default directory structure
- Default build lifecycle bindings

You can see the effective (merged) POM:
```bash
mvn help:effective-pom
```

---

## 5. Maven Build Lifecycle

### Three Built-in Lifecycles

Maven has **three built-in lifecycles**:

#### 1. Default Lifecycle (Build & Deploy)

```
validate      → Is the project correct? All necessary info available?
compile       → Compile source code
test-compile  → Compile test source code
test          → Run unit tests (using Surefire)
package       → Package compiled code (JAR, WAR)
verify        → Run integration tests and quality checks
install       → Install artifact to local repository (~/.m2)
deploy        → Deploy artifact to remote repository
```

#### 2. Clean Lifecycle

```
pre-clean     → Pre-cleaning steps
clean         → Delete target/ directory
post-clean    → Post-cleaning steps
```

#### 3. Site Lifecycle

```
pre-site      → Pre-site generation
site          → Generate project documentation
post-site     → Post-site generation
site-deploy   → Deploy generated site
```

### How Phases Work

**Important:** When you run a phase, ALL preceding phases also run.

```bash
mvn compile     # Runs: validate → compile
mvn test        # Runs: validate → compile → test-compile → test
mvn package     # Runs: validate → compile → test-compile → test → package
mvn install     # Runs: validate → ... → package → verify → install
mvn deploy      # Runs: validate → ... → install → deploy

# Skip tests
mvn package -DskipTests                    # Compile tests but DON'T run them
mvn package -Dmaven.test.skip=true         # DON'T compile or run tests

# Clean before building
mvn clean package                          # clean lifecycle + default lifecycle
mvn clean install -DskipTests              # Common CI command
```

### Phase-to-Plugin Goal Bindings (JAR packaging)

| Phase | Plugin:Goal |
|-------|-------------|
| process-resources | `resources:resources` |
| compile | `compiler:compile` |
| process-test-resources | `resources:testResources` |
| test-compile | `compiler:testCompile` |
| test | `surefire:test` |
| package | `jar:jar` |
| install | `install:install` |
| deploy | `deploy:deploy` |

### Running Specific Plugin Goals

```bash
# Run a specific plugin goal directly
mvn compiler:compile             # Just compile
mvn surefire:test                # Just run tests
mvn dependency:tree              # Show dependency tree
mvn dependency:analyze           # Find unused/undeclared dependencies
mvn versions:display-dependency-updates  # Check for updates
mvn help:describe -Dplugin=compiler      # Describe a plugin

# Spring Boot specific
mvn spring-boot:run              # Run the application
mvn spring-boot:build-image      # Build Docker image (Buildpacks)
```

---

## 6. Dependency Management

### Dependency Coordinates (GAV)

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>    <!-- WHO -->
    <artifactId>spring-boot-starter-web</artifactId> <!-- WHAT -->
    <version>3.3.0</version>                        <!-- WHICH version -->
    <type>jar</type>                                <!-- Type (default: jar) -->
    <scope>compile</scope>                          <!-- When needed (default: compile) -->
    <optional>false</optional>                      <!-- Is it optional? -->
    <classifier>sources</classifier>                <!-- Variant (sources, javadoc) -->
</dependency>
```

### Dependency Scopes

| Scope | Compile CP | Test CP | Runtime CP | Packaged | Use Case |
|-------|-----------|---------|------------|----------|----------|
| **compile** (default) | ✅ | ✅ | ✅ | ✅ | Spring, Jackson |
| **provided** | ✅ | ✅ | ❌ | ❌ | Servlet API, Lombok |
| **runtime** | ❌ | ✅ | ✅ | ✅ | JDBC drivers, SLF4J impls |
| **test** | ❌ | ✅ | ❌ | ❌ | JUnit, Mockito, H2 |
| **system** | ✅ | ✅ | ❌ | ❌ | Local JARs (AVOID) |
| **import** | N/A | N/A | N/A | N/A | BOM imports |

```xml
<!-- Examples of each scope -->

<!-- compile (default) — needed everywhere -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <!-- scope is "compile" by default -->
</dependency>

<!-- provided — the container provides this at runtime -->
<dependency>
    <groupId>jakarta.servlet</groupId>
    <artifactId>jakarta.servlet-api</artifactId>
    <scope>provided</scope>
</dependency>

<!-- runtime — not needed for compilation, only at runtime -->
<dependency>
    <groupId>com.mysql</groupId>
    <artifactId>mysql-connector-j</artifactId>
    <scope>runtime</scope>
</dependency>

<!-- test — only for testing -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<!-- import — used in dependencyManagement to import a BOM -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-dependencies</artifactId>
    <version>2023.0.1</version>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

### Transitive Dependencies

When you add dependency A, and A depends on B, and B depends on C:
- **Transitive**: A → B → C — Maven automatically includes B and C
- **Depth**: Maven includes transitives up to a configurable depth

### Dependency Mediation

When multiple paths lead to the same dependency with different versions, Maven uses the **"nearest definition"** strategy:

```
Project
├── A:1.0
│   └── C:2.0    ← This version wins (depth 2)
└── B:1.0
    └── D:1.0
        └── C:3.0 ← Ignored (depth 3)
```

### Excluding Transitive Dependencies

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <!-- Exclude Tomcat to use Jetty instead -->
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- Add Jetty instead -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jetty</artifactId>
</dependency>
```

### Dependency Analysis Commands

```bash
# Show dependency tree (ESSENTIAL for debugging)
mvn dependency:tree
# Output:
# com.example:my-app:jar:1.0.0
# ├── org.springframework.boot:spring-boot-starter-web:jar:3.3.0
# │   ├── org.springframework.boot:spring-boot-starter:jar:3.3.0
# │   │   ├── org.springframework.boot:spring-boot:jar:3.3.0
# │   │   │   └── org.springframework:spring-core:jar:6.1.6
# │   │   └── ...
# │   ├── org.springframework.boot:spring-boot-starter-json:jar:3.3.0
# │   └── org.springframework.boot:spring-boot-starter-tomcat:jar:3.3.0

# Filter dependency tree
mvn dependency:tree -Dincludes=org.slf4j
mvn dependency:tree -Dverbose   # Show conflicts

# Find unused and undeclared dependencies
mvn dependency:analyze

# List all dependencies
mvn dependency:list

# Copy dependencies to a folder
mvn dependency:copy-dependencies -DoutputDirectory=lib/

# Download sources and javadocs
mvn dependency:sources
mvn dependency:resolve -Dclassifier=javadoc
```

---

## 7. Maven Repositories

### Repository Types

```
┌─────────────────────────────────────────────┐
│              Repository Hierarchy            │
│                                              │
│  1. Local Repository (~/.m2/repository)      │
│     └─ Maven checks here FIRST              │
│                                              │
│  2. Remote Repositories                      │
│     ├─ Maven Central (default)               │
│     │  https://repo.maven.apache.org/maven2  │
│     ├─ Company Nexus/Artifactory             │
│     │  https://nexus.company.com/repository  │
│     └─ Custom repositories                   │
│        (defined in POM or settings.xml)      │
└─────────────────────────────────────────────┘
```

### Resolution Order

1. Check **local repository** (`~/.m2/repository`)
2. Check **remote repositories** (in order defined in POM)
3. If not found → **BUILD FAILS**

### Configuring Custom Repositories

```xml
<!-- In pom.xml -->
<repositories>
    <repository>
        <id>spring-milestones</id>
        <name>Spring Milestones</name>
        <url>https://repo.spring.io/milestone</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
    <repository>
        <id>spring-snapshots</id>
        <name>Spring Snapshots</name>
        <url>https://repo.spring.io/snapshot</url>
        <releases>
            <enabled>false</enabled>
        </releases>
    </repository>
</repositories>

<!-- Plugin repositories (for Maven plugins) -->
<pluginRepositories>
    <pluginRepository>
        <id>spring-milestones</id>
        <url>https://repo.spring.io/milestone</url>
    </pluginRepository>
</pluginRepositories>
```

### SNAPSHOT vs RELEASE

| Feature | SNAPSHOT | RELEASE |
|---------|----------|---------|
| Version | `1.0.0-SNAPSHOT` | `1.0.0` |
| Mutable | Yes (updated frequently) | No (immutable) |
| Maven behavior | Checks for updates | Uses cached version |
| Use in production | ❌ Never | ✅ Always |
| Purpose | Active development | Stable releases |

```xml
<!-- SNAPSHOT — Maven checks remote repo for updates (daily by default) -->
<dependency>
    <groupId>com.example</groupId>
    <artifactId>my-lib</artifactId>
    <version>1.0.0-SNAPSHOT</version>
</dependency>

<!-- Force update check -->
<!-- mvn clean install -U  (force update snapshots) -->
```

---

## 8. Maven Plugins

### Core Plugins

| Plugin | Purpose |
|--------|---------|
| `maven-compiler-plugin` | Compiles Java source code |
| `maven-surefire-plugin` | Runs unit tests |
| `maven-failsafe-plugin` | Runs integration tests |
| `maven-jar-plugin` | Creates JAR files |
| `maven-war-plugin` | Creates WAR files |
| `maven-resources-plugin` | Copies resources |
| `maven-install-plugin` | Installs to local repo |
| `maven-deploy-plugin` | Deploys to remote repo |
| `maven-clean-plugin` | Cleans target directory |
| `maven-site-plugin` | Generates project site |

### Essential Third-Party Plugins

```xml
<!-- Spring Boot Maven Plugin -->
<plugin>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-maven-plugin</artifactId>
    <configuration>
        <mainClass>com.example.Application</mainClass>
        <layers>
            <enabled>true</enabled>
        </layers>
    </configuration>
    <executions>
        <execution>
            <goals>
                <goal>repackage</goal> <!-- Creates executable JAR -->
                <goal>build-info</goal> <!-- Generate build-info.properties -->
            </goals>
        </execution>
    </executions>
</plugin>

<!-- JaCoCo — Code Coverage -->
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.12</version>
    <executions>
        <execution>
            <id>prepare-agent</id>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
        <execution>
            <id>check</id>
            <goals>
                <goal>check</goal>
            </goals>
            <configuration>
                <rules>
                    <rule>
                        <element>BUNDLE</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum> <!-- 80% line coverage -->
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>

<!-- Spotless — Code Formatting -->
<plugin>
    <groupId>com.diffplug.spotless</groupId>
    <artifactId>spotless-maven-plugin</artifactId>
    <version>2.43.0</version>
    <configuration>
        <java>
            <googleJavaFormat/>
        </java>
    </configuration>
</plugin>

<!-- Maven Enforcer — Enforce Rules -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-enforcer-plugin</artifactId>
    <executions>
        <execution>
            <id>enforce</id>
            <goals>
                <goal>enforce</goal>
            </goals>
            <configuration>
                <rules>
                    <requireMavenVersion>
                        <version>3.8.0</version>
                    </requireMavenVersion>
                    <requireJavaVersion>
                        <version>17</version>
                    </requireJavaVersion>
                    <banDuplicatePomDependencyVersions/>
                    <dependencyConvergence/>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>

<!-- Versions Plugin — Manage dependency versions -->
<!-- mvn versions:display-dependency-updates -->
<!-- mvn versions:display-plugin-updates -->
<!-- mvn versions:use-latest-releases -->
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>versions-maven-plugin</artifactId>
</plugin>

<!-- Jib — Docker image without Dockerfile -->
<plugin>
    <groupId>com.google.cloud.tools</groupId>
    <artifactId>jib-maven-plugin</artifactId>
    <version>3.4.1</version>
    <configuration>
        <to>
            <image>registry.example.com/my-app</image>
            <tags>
                <tag>latest</tag>
                <tag>${project.version}</tag>
            </tags>
        </to>
        <from>
            <image>eclipse-temurin:21-jre-alpine</image>
        </from>
    </configuration>
</plugin>
```

---

## 9. Profiles & Environment-Specific Builds

### Defining Profiles

```xml
<profiles>
    <!-- Development Profile -->
    <profile>
        <id>dev</id>
        <activation>
            <activeByDefault>true</activeByDefault>
        </activation>
        <properties>
            <spring.profiles.active>dev</spring.profiles.active>
            <db.url>jdbc:h2:mem:devdb</db.url>
        </properties>
        <dependencies>
            <dependency>
                <groupId>com.h2database</groupId>
                <artifactId>h2</artifactId>
                <scope>runtime</scope>
            </dependency>
        </dependencies>
    </profile>
    
    <!-- Production Profile -->
    <profile>
        <id>prod</id>
        <properties>
            <spring.profiles.active>prod</spring.profiles.active>
            <db.url>jdbc:mysql://prod-db:3306/myapp</db.url>
        </properties>
        <dependencies>
            <dependency>
                <groupId>com.mysql</groupId>
                <artifactId>mysql-connector-j</artifactId>
                <scope>runtime</scope>
            </dependency>
        </dependencies>
    </profile>
    
    <!-- CI/CD Profile -->
    <profile>
        <id>ci</id>
        <build>
            <plugins>
                <plugin>
                    <groupId>org.jacoco</groupId>
                    <artifactId>jacoco-maven-plugin</artifactId>
                    <!-- Coverage required in CI -->
                </plugin>
            </plugins>
        </build>
    </profile>
    
    <!-- Profile activated by OS -->
    <profile>
        <id>windows</id>
        <activation>
            <os>
                <family>windows</family>
            </os>
        </activation>
    </profile>
    
    <!-- Profile activated by JDK version -->
    <profile>
        <id>jdk21</id>
        <activation>
            <jdk>[21,)</jdk>
        </activation>
    </profile>
    
    <!-- Profile activated by property -->
    <profile>
        <id>integration-tests</id>
        <activation>
            <property>
                <name>integration</name>
                <value>true</value>
            </property>
        </activation>
    </profile>
</profiles>
```

### Activating Profiles

```bash
# Activate by name
mvn clean install -Pprod
mvn clean install -Pdev,ci         # Multiple profiles

# Activate by property
mvn clean install -Dintegration=true

# List active profiles
mvn help:active-profiles

# List all profiles
mvn help:all-profiles
```

---

## 10. Multi-Module Projects

### Structure

```
parent-project/
├── pom.xml                    # Parent POM (packaging: pom)
├── common/
│   └── pom.xml                # Common utilities module
├── api/
│   └── pom.xml                # REST API module
├── service/
│   └── pom.xml                # Business logic module
└── persistence/
    └── pom.xml                # Data access module
```

### Parent POM

```xml
<!-- parent-project/pom.xml -->
<project>
    <modelVersion>4.0.0</modelVersion>
    
    <groupId>com.example</groupId>
    <artifactId>parent-project</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>pom</packaging>  <!-- MUST be 'pom' for parent -->
    
    <!-- Modules to build (reactor order) -->
    <modules>
        <module>common</module>
        <module>persistence</module>
        <module>service</module>
        <module>api</module>
    </modules>
    
    <!-- Centralized version management -->
    <properties>
        <java.version>21</java.version>
        <spring-boot.version>3.3.0</spring-boot.version>
    </properties>
    
    <!-- Dependency versions for ALL modules -->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            
            <!-- Internal module versions -->
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>common</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>persistence</artifactId>
                <version>${project.version}</version>
            </dependency>
            <dependency>
                <groupId>com.example</groupId>
                <artifactId>service</artifactId>
                <version>${project.version}</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

### Child Module POM

```xml
<!-- api/pom.xml -->
<project>
    <modelVersion>4.0.0</modelVersion>
    
    <parent>
        <groupId>com.example</groupId>
        <artifactId>parent-project</artifactId>
        <version>1.0.0-SNAPSHOT</version>
    </parent>
    
    <artifactId>api</artifactId>
    
    <dependencies>
        <!-- Internal dependencies (version inherited from parent) -->
        <dependency>
            <groupId>com.example</groupId>
            <artifactId>service</artifactId>
        </dependency>
        
        <!-- External dependencies (version from BOM) -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>
</project>
```

### Building Multi-Module Projects

```bash
# Build all modules
mvn clean install

# Build specific module (and its dependencies)
mvn clean install -pl api -am    # -pl = projects list, -am = also make dependencies

# Build specific module only
mvn clean install -pl service

# Resume build from a failed module
mvn clean install -rf :service   # Resume from service module
```

---

## 11. Maven Wrapper

The Maven Wrapper ensures everyone on the team uses the **same Maven version**, without requiring Maven to be installed.

```bash
# Generate Maven Wrapper files
mvn wrapper:wrapper -Dmaven=3.9.6

# This creates:
# mvnw          — Unix wrapper script
# mvnw.cmd      — Windows wrapper script
# .mvn/
#   └── wrapper/
#       ├── maven-wrapper.jar
#       └── maven-wrapper.properties

# Use wrapper instead of mvn
./mvnw clean install     # Unix/Mac
mvnw.cmd clean install   # Windows
```

**Always commit wrapper files to version control!**

---

## 12. BOM (Bill of Materials)

A BOM is a special POM that centralizes dependency version management across multiple projects.

### Using a BOM

```xml
<dependencyManagement>
    <dependencies>
        <!-- Spring Boot BOM -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>3.3.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
        
        <!-- AWS SDK BOM -->
        <dependency>
            <groupId>software.amazon.awssdk</groupId>
            <artifactId>bom</artifactId>
            <version>2.25.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<!-- Now use dependencies WITHOUT specifying versions -->
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
        <!-- No version needed! Managed by BOM -->
    </dependency>
    
    <dependency>
        <groupId>software.amazon.awssdk</groupId>
        <artifactId>s3</artifactId>
        <!-- No version needed! Managed by BOM -->
    </dependency>
</dependencies>
```

### Creating Your Own BOM

```xml
<!-- my-company-bom/pom.xml -->
<project>
    <groupId>com.company</groupId>
    <artifactId>my-company-bom</artifactId>
    <version>1.0.0</version>
    <packaging>pom</packaging>
    
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>com.company</groupId>
                <artifactId>common-utils</artifactId>
                <version>2.1.0</version>
            </dependency>
            <dependency>
                <groupId>com.company</groupId>
                <artifactId>security-lib</artifactId>
                <version>3.0.1</version>
            </dependency>
        </dependencies>
    </dependencyManagement>
</project>
```

---

## 13. Advanced Maven Features

### Resource Filtering

```xml
<!-- Enable filtering to replace ${...} placeholders in resource files -->
<build>
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
            <includes>
                <include>**/*.properties</include>
                <include>**/*.yml</include>
            </includes>
        </resource>
        <resource>
            <directory>src/main/resources</directory>
            <filtering>false</filtering>
            <excludes>
                <exclude>**/*.properties</exclude>
                <exclude>**/*.yml</exclude>
            </excludes>
        </resource>
    </resources>
</build>
```

```properties
# application.properties — ${...} will be replaced during build
app.version=${project.version}
app.name=${project.name}
db.url=${db.url}
```

### Assembly Plugin (Creating Distribution Packages)

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-assembly-plugin</artifactId>
    <configuration>
        <descriptorRefs>
            <descriptorRef>jar-with-dependencies</descriptorRef>
        </descriptorRefs>
        <archive>
            <manifest>
                <mainClass>com.example.Main</mainClass>
            </manifest>
        </archive>
    </configuration>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>single</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

### Shade Plugin (Uber JAR with Relocation)

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-shade-plugin</artifactId>
    <executions>
        <execution>
            <phase>package</phase>
            <goals>
                <goal>shade</goal>
            </goals>
            <configuration>
                <relocations>
                    <relocation>
                        <pattern>com.google.guava</pattern>
                        <shadedPattern>com.example.shaded.guava</shadedPattern>
                    </relocation>
                </relocations>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### Useful Maven Commands

```bash
# Create project from archetype
mvn archetype:generate -DgroupId=com.example \
    -DartifactId=my-app \
    -DarchetypeArtifactId=maven-archetype-quickstart \
    -DinteractiveMode=false

# Show effective POM (after inheritance & interpolation)
mvn help:effective-pom

# Show effective settings
mvn help:effective-settings

# Describe a plugin
mvn help:describe -Dplugin=compiler -Ddetail

# Evaluate expressions
mvn help:evaluate -Dexpression=project.version -q -DforceStdout

# Run in offline mode (use only local repo)
mvn clean install -o

# Run in debug mode
mvn clean install -X

# Run with specific settings file
mvn clean install -s /path/to/settings.xml

# Show reactor build order (multi-module)
mvn validate -pl api -am
```

---

## 14. Maven vs Gradle

| Feature | Maven | Gradle |
|---------|-------|--------|
| Configuration | XML (pom.xml) | Groovy/Kotlin DSL (build.gradle) |
| Learning curve | Lower (XML is straightforward) | Higher (DSL requires learning) |
| Performance | Slower (no caching) | Faster (incremental builds, caching) |
| Flexibility | Convention-based, less flexible | Highly flexible and customizable |
| IDE support | Excellent | Excellent |
| Build scripts | Declarative (XML) | Declarative + Imperative (scripting) |
| Dependency mgmt | Good | Better (conflict resolution) |
| Multi-module | Good | Better (composite builds) |
| Community | Larger (older, more projects) | Growing (Android default) |
| Spring Boot | ✅ Fully supported | ✅ Fully supported |

**When to use Maven:** Simpler projects, teams familiar with Maven, corporate environments preferring standardization.

**When to use Gradle:** Performance-critical builds, Android development, complex build logic, larger projects.

---

## 15. Best Practices

1. **Always use the Maven Wrapper** (`mvnw`) — ensures consistent Maven version across team
2. **Use BOMs for version management** — avoid specifying versions for managed dependencies
3. **Use properties for versions** — `<my.version>1.2.3</my.version>` + `${my.version}`
4. **Use `dependencyManagement`** in parent POMs — control versions without adding dependencies
5. **Run `mvn dependency:tree`** regularly — understand your dependency graph
6. **Use `mvn dependency:analyze`** — find unused and undeclared dependencies
7. **Pin plugin versions** — use `<pluginManagement>` for reproducible builds
8. **Never use SNAPSHOT versions in production** — they're mutable and unreliable
9. **Use profiles sparingly** — prefer Spring profiles or environment variables
10. **Use `mvn enforcer:enforce`** — enforce minimum Java/Maven versions
11. **Configure `<distributionManagement>`** — for deploying to Nexus/Artifactory
12. **Use `mvn versions:display-dependency-updates`** — keep dependencies current
13. **Exclude test dependencies from final artifact** — use `<scope>test</scope>`
14. **Use `<optional>true</optional>`** — for dependencies that consumers shouldn't inherit
15. **Configure encoding** — `<project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>`

---

## 16. Common Mistakes & Pitfalls

```xml
<!-- ❌ MISTAKE 1: Not specifying encoding -->
<!-- Causes: Platform-dependent builds, garbled text -->
<!-- ✅ FIX: Add to properties -->
<properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
</properties>

<!-- ❌ MISTAKE 2: Using system scope -->
<dependency>
    <groupId>com.example</groupId>
    <artifactId>my-lib</artifactId>
    <version>1.0</version>
    <scope>system</scope>
    <systemPath>/path/to/lib.jar</systemPath>
</dependency>
<!-- ✅ FIX: Install to local/remote repo or use a repository manager -->

<!-- ❌ MISTAKE 3: Not pinning plugin versions -->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-compiler-plugin</artifactId>
    <!-- No version! Maven picks latest, builds may differ -->
</plugin>
<!-- ✅ FIX: Always specify version or use pluginManagement -->

<!-- ❌ MISTAKE 4: Duplicate dependencies with different versions -->
<!-- Maven uses the LAST one declared, which may not be what you want -->
<!-- ✅ FIX: Use mvn dependency:analyze-duplicate -->

<!-- ❌ MISTAKE 5: Using SNAPSHOT versions in production -->
<!-- SNAPSHOTs are mutable — your build today may differ from tomorrow -->
<!-- ✅ FIX: Use release versions in production -->

<!-- ❌ MISTAKE 6: Not using dependencyManagement in multi-module projects -->
<!-- Child modules might use different versions of the same library -->
<!-- ✅ FIX: Centralize in parent's dependencyManagement -->

<!-- ❌ MISTAKE 7: Hardcoding versions everywhere -->
<!-- Version changes require updating in multiple places -->
<!-- ✅ FIX: Use properties -->
```

---

## 17. Interview Questions & Answers (50+)

### Beginner Level

**Q1: What is Maven and why is it used?**

**A:** Maven is a build automation and project management tool for Java. It handles:
- Compilation, testing, and packaging
- Dependency management (automatically downloads libraries)
- Standard project structure
- Build lifecycle with predefined phases
- Plugin architecture for extensibility

---

**Q2: What is a POM file?**

**A:** POM (Project Object Model) is the fundamental configuration file in Maven (pom.xml). It contains:
- Project coordinates (groupId, artifactId, version)
- Dependencies
- Build plugins
- Profiles
- Repository information
- Project metadata (name, description, URL)

---

**Q3: What is the Maven build lifecycle?**

**A:** Maven has three built-in lifecycles:
1. **Default**: validate → compile → test → package → verify → install → deploy
2. **Clean**: pre-clean → clean → post-clean
3. **Site**: pre-site → site → post-site → site-deploy

Each lifecycle consists of phases, and when you run a phase, all preceding phases run first.

---

**Q4: What is the difference between `mvn install` and `mvn deploy`?**

**A:**
- `mvn install`: Installs the artifact to the **local repository** (`~/.m2/repository`). Available only on your machine.
- `mvn deploy`: Deploys the artifact to a **remote repository** (Nexus, Artifactory). Available to other developers and CI/CD.

---

**Q5: What is a Maven archetype?**

**A:** An archetype is a project template. It generates a standard project structure with all necessary files. Example:
```bash
mvn archetype:generate -DarchetypeArtifactId=maven-archetype-quickstart
```

---

**Q6: What is the difference between `compile` and `provided` scope?**

**A:**
- `compile` (default): Available during compile, test, and runtime. Included in the final package.
- `provided`: Available during compile and test, but NOT at runtime (the container provides it). NOT included in the final package. Example: `servlet-api` is provided by Tomcat.

---

**Q7: How do you skip tests in Maven?**

**A:**
```bash
mvn package -DskipTests          # Compiles tests but doesn't run them
mvn package -Dmaven.test.skip=true  # Neither compiles nor runs tests
```

---

**Q8: What is the `target` directory?**

**A:** `target/` is Maven's output directory. It contains:
- Compiled classes (`target/classes/`)
- Test classes (`target/test-classes/`)
- Test reports (`target/surefire-reports/`)
- Final artifact (`target/myapp-1.0.0.jar`)
- `mvn clean` deletes this directory.

---

### Intermediate Level

**Q9: Explain transitive dependencies and how Maven resolves conflicts.**

**A:** Transitive dependencies are dependencies of your dependencies. If you depend on Spring Boot, and Spring Boot depends on Spring Core, then Spring Core is a transitive dependency.

**Conflict resolution strategies:**
1. **Nearest definition wins** — The shortest path in the dependency tree wins
2. **First declaration wins** — If same depth, the first one declared in POM wins
3. **Explicit declaration always wins** — Directly declaring a dependency overrides transitive

---

**Q10: What is the difference between `dependencies` and `dependencyManagement`?**

**A:**
- `<dependencies>`: Actually adds the dependency to the project. The dependency is downloaded and included.
- `<dependencyManagement>`: Only declares version/scope information. Does NOT add the dependency. Child modules can then reference the dependency without specifying a version — it inherits from dependencyManagement.

This is crucial for multi-module projects to ensure version consistency.

---

**Q11: What are Maven profiles and when would you use them?**

**A:** Profiles are a set of configuration values that customize a build for specific environments or conditions. Use cases:
- Environment-specific builds (dev, staging, prod)
- OS-specific configurations
- Optional features (enable/disable modules)
- CI/CD-specific settings (code coverage, linting)

Activated by: command line (`-Pprofile`), property, OS, JDK version, file presence, or `activeByDefault`.

---

**Q12: How does Maven's parent POM inheritance work?**

**A:** When a POM declares a `<parent>`, it inherits:
- groupId, version (if not overridden)
- Properties
- Dependencies from `dependencyManagement`
- Plugin configuration from `pluginManagement`
- Build settings
- Repositories

The child can override any inherited configuration. Effective POM = Super POM + Parent POM + Project POM.

---

**Q13: What is the Maven Wrapper and why should you use it?**

**A:** Maven Wrapper (`mvnw`) ensures that everyone building the project uses the exact same Maven version, without requiring Maven to be pre-installed. Benefits:
- Reproducible builds across environments
- No "works on my machine" issues
- CI/CD doesn't need Maven pre-installed
- New team members can build immediately

---

**Q14: What is a BOM (Bill of Materials)?**

**A:** A BOM is a POM file that centralizes dependency version management. You import it in `<dependencyManagement>` with `<scope>import</scope>` and `<type>pom</type>`. Then you can use dependencies from the BOM without specifying versions. Spring Boot Dependencies BOM manages 500+ dependency versions.

---

**Q15: What is the Maven Reactor?**

**A:** The Reactor is the mechanism that handles multi-module builds. It:
1. Reads all modules from `<modules>` section
2. Sorts them based on dependencies (topological sort)
3. Builds them in the correct order
4. Supports partial builds (`-pl`, `-am`, `-amd`, `-rf`)

---

### Advanced Level

**Q16: How does Maven resolve dependency versions when there are conflicts?**

**A:** Maven uses these strategies in order:
1. **Direct dependency always wins** over transitive
2. **Nearest definition** — shortest path in the dependency tree
3. **Declaration order** — first declared wins at the same depth
4. **`<dependencyManagement>` overrides everything** — explicitly set versions

For complex cases, use `mvn dependency:tree -Dverbose` to see conflict resolution details.

---

**Q17: Explain the difference between `<pluginManagement>` and `<plugins>`.**

**A:**
- `<pluginManagement>`: Declares plugin configuration that CHILD modules can inherit. Does NOT actually bind the plugin.
- `<plugins>`: Actually binds the plugin to the build lifecycle.

A child module must declare the plugin in `<plugins>` to activate it, but the configuration comes from parent's `<pluginManagement>`.

---

**Q18: How do you create a fat/uber JAR with Maven?**

**A:** Three approaches:
1. **Spring Boot Plugin**: `spring-boot:repackage` creates an executable JAR with embedded dependencies
2. **Assembly Plugin**: `maven-assembly-plugin` with `jar-with-dependencies` descriptor
3. **Shade Plugin**: `maven-shade-plugin` for class-level merging and package relocation

Spring Boot's approach is preferred for Spring applications.

---

**Q19: What is dependency mediation in Maven?**

**A:** When the same dependency appears multiple times in the tree with different versions, Maven uses the "nearest wins" strategy. Example:
```
A → B → C:1.0
A → D → E → C:2.0
```
C:1.0 wins because it's closer (depth 2 vs depth 3). If depths are equal, the first declaration order in pom.xml wins.

---

**Q20: How do you configure Maven for multi-environment deployments?**

**A:** Use a combination of:
1. **Profiles** — Different configurations for dev/staging/prod
2. **Resource filtering** — Replace `${property}` in config files
3. **Properties** — Environment-specific values
4. **settings.xml** — Server credentials, proxy settings
5. **CI/CD variables** — Pass values via `-D` flags

---

**Q21: How do you publish artifacts to a private repository?**

**A:** 
```xml
<!-- In pom.xml -->
<distributionManagement>
    <repository>
        <id>nexus-releases</id>
        <url>https://nexus.company.com/repository/maven-releases/</url>
    </repository>
    <snapshotRepository>
        <id>nexus-snapshots</id>
        <url>https://nexus.company.com/repository/maven-snapshots/</url>
    </snapshotRepository>
</distributionManagement>

<!-- In settings.xml — server credentials -->
<servers>
    <server>
        <id>nexus-releases</id>  <!-- Must match distributionManagement ID -->
        <username>admin</username>
        <password>password</password>
    </server>
</servers>
```
Then: `mvn clean deploy`

---

**Q22: What is the Super POM and what does it contain?**

**A:** The Super POM is Maven's default POM from which all POMs inherit. It defines:
- Maven Central as the default repository
- Default plugin versions for all lifecycle phases
- Standard directory layout (`src/main/java`, `target/`, etc.)
- Default resource directories
- Default reporting output directory

View it with: `mvn help:effective-pom` on a minimal POM.

---

**Q23: How does Maven handle SNAPSHOT versions internally?**

**A:** For SNAPSHOT versions:
1. Maven appends a timestamp (e.g., `1.0-20240101.120000-1`) when deploying
2. When resolving, Maven checks remote repo for newer timestamps (default: daily)
3. `-U` flag forces update check
4. SNAPSHOT metadata is stored in `maven-metadata.xml` in the repository
5. Local repo stores the latest downloaded snapshot

---

**Q24: What are Maven extensions and how do they differ from plugins?**

**A:** 
- **Plugins** execute specific goals during the build lifecycle
- **Extensions** modify Maven's core behavior — custom lifecycles, wagon providers (for custom repository protocols), build listeners

```xml
<build>
    <extensions>
        <extension>
            <groupId>org.apache.maven.wagon</groupId>
            <artifactId>wagon-ssh</artifactId>
            <version>3.5.3</version>
        </extension>
    </extensions>
</build>
```

---

**Q25: Explain Maven's dependency resolution algorithm in detail.**

**A:**
1. Read project POM and build dependency graph
2. For each dependency, resolve version:
   - Check `dependencyManagement` first
   - Apply nearest-wins for transitive conflicts
3. For each resolved dependency:
   - Check local repository
   - If SNAPSHOT, check remote for updates (based on update policy)
   - Download from remote repositories (in order defined)
4. Verify checksums (SHA-1, MD5)
5. Apply scope rules (compile, test, runtime, provided)
6. Build classpath for the requested scope

---

### Rapid-Fire Questions (Q26–Q50)

**Q26: What does `mvn clean` do?**
Deletes the `target/` directory and all build artifacts.

**Q27: Difference between `package` and `install` phases?**
`package` creates the JAR/WAR in `target/`. `install` copies it to `~/.m2/repository` for use by other local projects.

**Q28: What is the default Maven lifecycle for a JAR project?**
validate → compile → test → package → verify → install → deploy

**Q29: How do you run a specific test class?**
`mvn test -Dtest=MyTestClass`

**Q30: What is `<optional>true</optional>` in a dependency?**
The dependency is available for the current project but NOT propagated as a transitive dependency to projects that depend on it.

**Q31: How do you force Maven to update snapshots?**
`mvn clean install -U`

**Q32: What is the difference between `<repositories>` and `<pluginRepositories>`?**
`<repositories>` is for project dependencies. `<pluginRepositories>` is for Maven plugins.

**Q33: What command shows the effective POM?**
`mvn help:effective-pom`

**Q34: How do you find unused dependencies?**
`mvn dependency:analyze` — shows unused declared and used undeclared dependencies.

**Q35: What is `<relativePath/>` in the parent section?**
Tells Maven where to find the parent POM. Empty (`<relativePath/>`) means don't search locally, always download from repository.

**Q36: What is the reactor build order?**
It's the order Maven builds modules in a multi-module project. Determined by inter-module dependencies (topological sort).

**Q37: How do you create a project from an archetype?**
`mvn archetype:generate -DgroupId=com.example -DartifactId=my-app -DarchetypeArtifactId=maven-archetype-quickstart`

**Q38: What is the `settings.xml` file for?**
User-specific or global Maven configuration: repository mirrors, proxies, server credentials, active profiles. NOT committed to VCS.

**Q39: How do you run Maven in offline mode?**
`mvn clean install -o` — uses only local repository, no network access.

**Q40: What is the `<classifier>` element?**
A suffix appended to the artifact name to distinguish variants: `sources`, `javadoc`, `tests`, `linux-x86_64`.

**Q41: How do you exclude a transitive dependency?**
Use `<exclusions>` inside the `<dependency>` element.

**Q42: What is the default local repository location?**
`~/.m2/repository` (can be changed in `settings.xml`).

**Q43: How do you specify the Java version for compilation?**
Via `maven-compiler-plugin` configuration or `<java.version>` property with Spring Boot parent.

**Q44: What is the `<distributionManagement>` section?**
Defines where to deploy artifacts (`mvn deploy`). Specifies release and snapshot repository URLs.

**Q45: What is the difference between `goal` and `phase`?**
A **phase** is a step in the lifecycle. A **goal** is a specific task executed by a plugin. Phases are bound to goals. You can run goals directly (`mvn compiler:compile`) or through phases (`mvn compile`).

**Q46: How do you create a multi-module project?**
Create a parent POM with `<packaging>pom</packaging>` and `<modules>` listing child modules. Each child has a `<parent>` section pointing to the parent.

**Q47: What is `mvn verify` used for?**
Runs any checks on the package to verify it meets quality criteria (integration tests, code coverage checks, etc.).

**Q48: How do you pass command-line properties to Maven?**
`mvn clean install -Dproperty.name=value` — accessible as `${property.name}` in POM.

**Q49: What is the Maven Release Plugin?**
Automates the release process: removes -SNAPSHOT from version, creates a Git tag, updates to next SNAPSHOT version, deploys to remote repo.

**Q50: How does Maven handle circular dependencies?**
It doesn't — Maven will throw an error. Circular dependencies must be resolved by refactoring (extract shared code into a common module).

---

## 📚 References & Further Reading

- [Maven Official Documentation](https://maven.apache.org/guides/)
- [Maven Central Repository](https://search.maven.org/)
- [Maven POM Reference](https://maven.apache.org/pom.html)
- [Maven Lifecycle Reference](https://maven.apache.org/guides/introduction/introduction-to-the-lifecycle.html)
- [Spring Boot Maven Plugin](https://docs.spring.io/spring-boot/docs/current/maven-plugin/reference/htmlsingle/)
- [Maven by Example (Sonatype)](https://books.sonatype.com/mvnex-book/reference/)
- [Baeldung Maven Tutorials](https://www.baeldung.com/maven)

---

> **Previous Topic:** [← 01 - Core Java](../01-core-java/README.md)  
> **Next Topic:** [03 - Gradle →](../03-gradle/README.md)
