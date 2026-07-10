# 🐳 Docker — Complete In-Depth Guide

> **"Docker packages your application with ALL its dependencies into a container that runs identically on any machine. 'It works on my machine' is no longer an excuse."**

---

## 📑 Table of Contents

1. [What is Docker?](#1-what-is-docker)
2. [Docker Architecture](#2-docker-architecture)
3. [Docker vs Virtual Machines](#3-docker-vs-virtual-machines)
4. [Installation & First Run](#4-installation--first-run)
5. [Docker Images](#5-docker-images)
6. [Dockerfile](#6-dockerfile)
7. [Docker Commands](#7-docker-commands)
8. [Docker Compose](#8-docker-compose)
9. [Dockerizing Spring Boot](#9-dockerizing-spring-boot)
10. [Multi-Stage Builds](#10-multi-stage-builds)
11. [Volumes & Persistence](#11-volumes--persistence)
12. [Networking](#12-networking)
13. [Docker Registry (Docker Hub)](#13-docker-registry-docker-hub)
14. [Docker Best Practices](#14-docker-best-practices)
15. [Docker in CI/CD](#15-docker-in-cicd)
16. [Interview Questions & Answers (50+)](#16-interview-questions--answers-50)

---

## 1. What is Docker?

```
WITHOUT Docker:
  Developer: "It works on MY machine!"
  Ops:       "Well, it doesn't work on the server."
  Problem:   Different OS, Java version, env variables, dependencies...

WITH Docker:
  Package: App + JDK + configs + everything → ONE container
  Run:     Same container works on laptop, server, cloud. Identical!
  Result:  "It works in the container" = "It works everywhere" ✅
```

---

## 2. Docker Architecture

```
┌──────────────────────────────────────────────┐
│                Docker Client                  │
│  (docker build, docker run, docker-compose)   │
└───────────────┬──────────────────────────────┘
                │ REST API
┌───────────────▼──────────────────────────────┐
│              Docker Daemon (dockerd)          │
│                                               │
│  ┌───────────┐  ┌───────────┐  ┌───────────┐│
│  │ Container1│  │ Container2│  │ Container3││
│  │ (Spring)  │  │ (MySQL)   │  │ (Redis)   ││
│  └───────────┘  └───────────┘  └───────────┘│
│                                               │
│  ┌───────────────────────────────────────┐   │
│  │         Docker Images (read-only)      │   │
│  │  openjdk:21, mysql:8, redis:7          │   │
│  └───────────────────────────────────────┘   │
│                                               │
│  ┌───────────────────────────────────────┐   │
│  │         Volumes (persistent data)      │   │
│  └───────────────────────────────────────┘   │
│                                               │
│  ┌───────────────────────────────────────┐   │
│  │         Networks (container comms)     │   │
│  └───────────────────────────────────────┘   │
└───────────────────────────────────────────────┘
```

### Key Concepts

| Concept | Analogy | Description |
|---------|---------|-------------|
| **Image** | Recipe | Read-only template. Blueprint for containers |
| **Container** | Running dish | Running instance of an image |
| **Dockerfile** | Cookbook | Instructions to build an image |
| **Volume** | External storage | Persist data beyond container lifecycle |
| **Network** | Phone line | Communication between containers |
| **Registry** | App Store | Repository for images (Docker Hub) |

---

## 3. Docker vs Virtual Machines

```
Virtual Machine:                 Docker Container:
┌───────────────────┐           ┌───────────────────┐
│ App 1    │ App 2  │           │ App 1   │  App 2  │
├──────────┼────────┤           ├─────────┼─────────┤
│ Bins/Libs│Bins/Lib│           │ Bins/Lib│ Bins/Lib│
├──────────┼────────┤           ├─────────┴─────────┤
│ Guest OS │Guest OS│           │   Docker Engine    │
│ (Ubuntu) │(CentOS)│           │   (shared kernel)  │
├──────────┴────────┤           ├───────────────────┤
│   Hypervisor      │           │   Host OS          │
├───────────────────┤           ├───────────────────┤
│   Host OS         │           │   Hardware         │
├───────────────────┤           └───────────────────┘
│   Hardware        │
└───────────────────┘

VM:     Heavy (GB), slow start (minutes), full OS per VM
Docker: Light (MB), fast start (seconds), shared kernel
```

---

## 4. Installation & First Run

```bash
# Verify installation
docker --version
docker info

# Run your first container
docker run hello-world

# Run an interactive Ubuntu container
docker run -it ubuntu bash

# Run Nginx web server
docker run -d -p 8080:80 --name my-nginx nginx
# -d: detached (background)
# -p 8080:80: map host port 8080 → container port 80
# --name: give it a name
# Open http://localhost:8080
```

---

## 5. Docker Images

```bash
# List local images
docker images

# Pull image from Docker Hub
docker pull openjdk:21-slim
docker pull mysql:8.0
docker pull redis:7-alpine

# Remove image
docker rmi nginx:latest

# Image naming: registry/repository:tag
docker.io/library/nginx:1.25    # Full name
nginx:1.25                      # Shorthand (docker.io/library is default)
nginx:latest                    # "latest" tag (default if not specified)
myregistry.com/myapp:v1.2.0    # Custom registry

# Image layers (each instruction in Dockerfile = 1 layer)
docker history nginx:latest
```

---

## 6. Dockerfile

```dockerfile
# ═══ Basic Dockerfile for Spring Boot ═══

# Base image (start FROM an existing image)
FROM eclipse-temurin:21-jre-alpine

# Metadata
LABEL maintainer="dilip@mail.com"
LABEL version="1.0"

# Set working directory inside container
WORKDIR /app

# Copy JAR file into container
COPY target/myapp-1.0.0.jar app.jar

# Expose port (documentation, doesn't actually publish)
EXPOSE 8080

# Environment variables
ENV SPRING_PROFILES_ACTIVE=prod
ENV JAVA_OPTS="-Xmx512m -Xms256m"

# Health check
HEALTHCHECK --interval=30s --timeout=3s --retries=3 \
  CMD curl -f http://localhost:8080/actuator/health || exit 1

# Command to run when container starts
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Dockerfile Instructions Reference

| Instruction | Purpose | Example |
|-------------|---------|---------|
| `FROM` | Base image | `FROM openjdk:21-slim` |
| `WORKDIR` | Set working directory | `WORKDIR /app` |
| `COPY` | Copy files from host | `COPY target/app.jar .` |
| `ADD` | Copy + extract archives | `ADD config.tar.gz /etc/` |
| `RUN` | Execute command during build | `RUN apt-get update` |
| `ENV` | Set environment variable | `ENV PORT=8080` |
| `EXPOSE` | Document exposed port | `EXPOSE 8080` |
| `ENTRYPOINT` | Main command (not overridable) | `ENTRYPOINT ["java", "-jar"]` |
| `CMD` | Default arguments (overridable) | `CMD ["app.jar"]` |
| `ARG` | Build-time variable | `ARG JAR_FILE=app.jar` |
| `VOLUME` | Create mount point | `VOLUME /data` |
| `USER` | Set user to run as | `USER 1001` |
| `HEALTHCHECK` | Container health check | `HEALTHCHECK CMD curl ...` |

---

## 7. Docker Commands

```bash
# ═══ CONTAINER LIFECYCLE ═══
docker run -d -p 8080:8080 --name myapp myimage    # Create & start
docker start myapp                                    # Start stopped container
docker stop myapp                                     # Graceful stop
docker restart myapp                                  # Restart
docker kill myapp                                     # Force stop
docker rm myapp                                       # Remove container
docker rm -f myapp                                    # Force remove (running)

# ═══ INSPECT & DEBUG ═══
docker ps                      # List running containers
docker ps -a                   # List ALL containers (including stopped)
docker logs myapp              # View logs
docker logs -f myapp           # Follow logs (tail)
docker logs --tail 100 myapp   # Last 100 lines
docker exec -it myapp bash     # Execute command inside container
docker exec -it myapp sh       # Shell (Alpine uses sh)
docker inspect myapp           # Detailed container info
docker stats                   # Live resource usage (CPU, memory)
docker top myapp               # Running processes inside container

# ═══ BUILD ═══
docker build -t myapp:1.0 .                    # Build image from Dockerfile
docker build -t myapp:1.0 -f Dockerfile.prod . # Custom Dockerfile
docker build --no-cache -t myapp:1.0 .         # Build without cache

# ═══ CLEANUP ═══
docker system prune            # Remove unused data
docker container prune         # Remove stopped containers
docker image prune             # Remove dangling images
docker volume prune            # Remove unused volumes
```

---

## 8. Docker Compose

```yaml
# docker-compose.yml — Define multi-container application

version: '3.8'

services:
  # ═══ Spring Boot Application ═══
  app:
    build:
      context: .
      dockerfile: Dockerfile
    container_name: myapp
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=prod
      - SPRING_DATASOURCE_URL=jdbc:mysql://db:3306/mydb
      - SPRING_DATASOURCE_USERNAME=root
      - SPRING_DATASOURCE_PASSWORD=password
      - SPRING_DATA_REDIS_HOST=redis
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - app-network
    restart: unless-stopped

  # ═══ MySQL Database ═══
  db:
    image: mysql:8.0
    container_name: mysql
    ports:
      - "3306:3306"
    environment:
      - MYSQL_ROOT_PASSWORD=password
      - MYSQL_DATABASE=mydb
    volumes:
      - mysql-data:/var/lib/mysql        # Persist data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql  # Init script
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - app-network

  # ═══ Redis Cache ═══
  redis:
    image: redis:7-alpine
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    networks:
      - app-network

  # ═══ MongoDB ═══
  mongo:
    image: mongo:7
    container_name: mongodb
    ports:
      - "27017:27017"
    environment:
      - MONGO_INITDB_ROOT_USERNAME=admin
      - MONGO_INITDB_ROOT_PASSWORD=password
    volumes:
      - mongo-data:/data/db
    networks:
      - app-network

# ═══ Named Volumes (persistent storage) ═══
volumes:
  mysql-data:
  redis-data:
  mongo-data:

# ═══ Networks ═══
networks:
  app-network:
    driver: bridge
```

```bash
# Docker Compose Commands
docker-compose up -d          # Start all services (detached)
docker-compose down           # Stop and remove containers
docker-compose down -v        # Also remove volumes (data!)
docker-compose logs -f app    # Follow app logs
docker-compose ps             # List services
docker-compose build          # Rebuild images
docker-compose restart app    # Restart specific service
docker-compose exec app sh    # Shell into container
```

---

## 9. Dockerizing Spring Boot

### Step-by-Step

```bash
# 1. Build the JAR
./mvnw clean package -DskipTests

# 2. Create Dockerfile (in project root)
# 3. Build Docker image
docker build -t myapp:latest .

# 4. Run the container
docker run -d -p 8080:8080 --name myapp \
  -e SPRING_PROFILES_ACTIVE=prod \
  -e SPRING_DATASOURCE_URL=jdbc:mysql://host.docker.internal:3306/mydb \
  myapp:latest

# 5. Check logs
docker logs -f myapp
```

---

## 10. Multi-Stage Builds

```dockerfile
# ═══ MULTI-STAGE: Build AND run in one Dockerfile ═══
# Result: smaller final image (only JRE, not JDK + Maven)

# Stage 1: BUILD (heavy — JDK + Maven, discarded after build)
FROM eclipse-temurin:21-jdk AS builder
WORKDIR /app
COPY pom.xml .
COPY src ./src

# Download dependencies first (cached if pom.xml unchanged)
RUN apt-get update && apt-get install -y maven
RUN mvn dependency:go-offline
RUN mvn clean package -DskipTests

# Stage 2: RUN (lightweight — only JRE + JAR)
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app

# Create non-root user
RUN addgroup -S spring && adduser -S spring -G spring
USER spring:spring

# Copy ONLY the built JAR from stage 1
COPY --from=builder /app/target/*.jar app.jar

EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]

# Final image: ~200MB (instead of ~800MB with full JDK + Maven)
```

---

## 11. Volumes & Persistence

```bash
# Containers are EPHEMERAL — data is lost when container is removed!
# Volumes persist data beyond container lifecycle.

# Named volume
docker run -v mysql-data:/var/lib/mysql mysql:8

# Bind mount (host directory → container)
docker run -v /host/path/logs:/app/logs myapp

# Volume types:
# Named volume: Docker manages location (docker volume create)
# Bind mount: You specify exact host path
# tmpfs: In-memory only (not persisted)
```

---

## 12. Networking

```bash
# Containers on the same network can talk using container NAMES

# Create network
docker network create app-net

# Run containers on same network
docker run -d --name db --network app-net mysql:8
docker run -d --name app --network app-net -e DB_HOST=db myapp
# app can reach MySQL at hostname "db" (container name)

# Network types:
# bridge: Default. Containers on same bridge can communicate.
# host: Container uses host's network directly.
# none: No networking.
# overlay: Multi-host (Docker Swarm/Kubernetes).
```

---

## 13. Docker Registry (Docker Hub)

```bash
# Login to Docker Hub
docker login

# Tag image
docker tag myapp:latest dilsec20/myapp:1.0.0

# Push to Docker Hub
docker push dilsec20/myapp:1.0.0

# Pull from Docker Hub
docker pull dilsec20/myapp:1.0.0

# Private registry
docker tag myapp:latest myregistry.com:5000/myapp:1.0
docker push myregistry.com:5000/myapp:1.0
```

---

## 14. Docker Best Practices

1. **Use multi-stage builds** — Smaller images, no build tools in production
2. **Use specific tags** — `openjdk:21-slim` not `openjdk:latest`
3. **Run as non-root user** — Security! `USER 1001`
4. **Use `.dockerignore`** — Exclude unnecessary files
5. **One process per container** — Don't run app + DB in same container
6. **Use HEALTHCHECK** — Proper container health monitoring
7. **Order Dockerfile layers** — Put rarely-changing layers first (caching)
8. **Use Alpine images** — Smallest base images (`-alpine`)
9. **Don't store data in containers** — Use volumes
10. **Scan images for vulnerabilities** — `docker scout`, Trivy

### .dockerignore

```
.git
.gitignore
*.md
target/
!target/*.jar
node_modules/
.env
docker-compose*.yml
```

---

## 15. Docker in CI/CD

```yaml
# GitHub Actions example
name: Build and Push Docker Image

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
      
      - name: Build with Maven
        run: mvn clean package -DskipTests
      
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_TOKEN }}
      
      - name: Build and Push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: true
          tags: dilsec20/myapp:${{ github.sha }}
```

---

## 16. Interview Questions & Answers (50+)

### Beginner

**Q1: What is Docker?** Platform for packaging apps with dependencies into containers. Containers run consistently on any machine.

**Q2: Docker image vs container?** Image: read-only template (blueprint). Container: running instance of image.

**Q3: What is a Dockerfile?** Text file with instructions to build a Docker image. FROM, COPY, RUN, EXPOSE, ENTRYPOINT.

**Q4: What is Docker Compose?** Tool for defining multi-container apps in YAML. Run entire stack with `docker-compose up`.

**Q5: Docker vs VM?** Docker: lightweight, shared kernel, seconds startup, MB size. VM: heavy, separate OS, minutes startup, GB size.

**Q6: What is Docker Hub?** Public registry for Docker images. Like GitHub for containers.

**Q7: What is a Docker volume?** Persistent storage. Data survives container deletion.

**Q8: What is `docker run`?** Creates and starts a container from an image.

---

### Intermediate

**Q9: What is multi-stage build?** Multiple FROM stages in Dockerfile. Build in first stage, copy result to final stage. Smaller images.

**Q10: ENTRYPOINT vs CMD?** ENTRYPOINT: main command (not overridden). CMD: default arguments (can be overridden). Used together: `ENTRYPOINT ["java"] CMD ["-jar", "app.jar"]`.

**Q11: COPY vs ADD?** COPY: simple file copy. ADD: copy + auto-extract tar + fetch URLs. Use COPY unless you need ADD features.

**Q12: What is Docker networking?** Containers communicate via Docker networks. Bridge (default), host, none, overlay.

**Q13: What is `depends_on`?** Start order in Docker Compose. Waits for dependency to start (not be ready).

**Q14: How to persist MySQL data?** Named volume: `volumes: - mysql-data:/var/lib/mysql`.

**Q15: What is `.dockerignore`?** Excludes files from build context. Like .gitignore for Docker.

---

### Advanced

**Q16: How does Docker layering work?** Each Dockerfile instruction = one layer. Layers are cached and reused. Unchanged layers are not rebuilt.

**Q17: What is Docker overlay network?** Network spanning multiple Docker hosts. Used in Docker Swarm and Kubernetes.

**Q18: What is Docker Swarm?** Docker's built-in orchestration. Manages cluster of Docker nodes. Simpler alternative to Kubernetes.

**Q19: Docker vs Kubernetes?** Docker: single-host container runtime. Kubernetes: multi-host container orchestration (scaling, healing, service discovery).

**Q20: What is container orchestration?** Managing deployment, scaling, networking, and health of containers across multiple hosts.

---

### Rapid-Fire (Q21–Q50)

**Q21: `docker ps` shows what?** Running containers.

**Q22: `docker ps -a`?** All containers including stopped.

**Q23: `docker exec -it bash`?** Interactive shell inside running container.

**Q24: `docker logs -f`?** Follow (tail) container logs.

**Q25: `docker stop` vs `docker kill`?** stop: graceful (SIGTERM). kill: force (SIGKILL).

**Q26: `docker system prune`?** Remove all unused containers, images, networks.

**Q27: `docker inspect`?** Detailed JSON info about container/image.

**Q28: What is Alpine Linux?** Minimal Linux distro (~5MB). Used for smallest Docker images.

**Q29: What is `host.docker.internal`?** Hostname to access host machine from inside container (Docker Desktop).

**Q30: How to reduce image size?** Alpine base, multi-stage build, .dockerignore, minimize layers.

**Q31: What is `EXPOSE`?** Documents which port the container listens on. Doesn't actually publish it.

**Q32: `-p 8080:80` means?** Map host port 8080 to container port 80.

**Q33: What is `restart: unless-stopped`?** Auto-restart container unless explicitly stopped.

**Q34: What is Docker context?** Files sent to Docker daemon during build. Controlled by .dockerignore.

**Q35: What is `docker build --no-cache`?** Rebuild all layers without using cache.

**Q36: What is `docker tag`?** Add a tag (name:version) to an image.

**Q37: What is `docker push`?** Upload image to a registry.

**Q38: What is `docker pull`?** Download image from a registry.

**Q39: What is Docker Desktop?** GUI application for Docker on Windows/Mac.

**Q40: What is Testcontainers?** Java library for using Docker containers in tests.

**Q41: What is `HEALTHCHECK`?** Instruction to check if container is healthy.

**Q42: What is `USER` instruction?** Sets the user for subsequent instructions and container runtime.

**Q43: What is `ARG` vs `ENV`?** ARG: build-time only. ENV: build-time + runtime.

**Q44: What is Docker Scout?** Security scanning tool for Docker images.

**Q45: What is distroless image?** Image with only your app + runtime. No shell, no package manager. Most secure.

**Q46: `docker-compose up -d`?** Start services in detached (background) mode.

**Q47: `docker-compose down -v`?** Stop services AND remove volumes.

**Q48: What is `docker stats`?** Live resource usage (CPU, memory, network) per container.

**Q49: What is BuildKit?** Next-gen Docker build engine. Parallel builds, better caching.

**Q50: What is OCI?** Open Container Initiative — standard for container formats and runtimes.

---

## 📚 References

- [Docker Documentation](https://docs.docker.com/)
- [Docker Hub](https://hub.docker.com/)
- [Spring Boot Docker Guide](https://spring.io/guides/topicals/spring-boot-docker)
- [Dockerfile Best Practices](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)

---

> **Previous Topic:** [← 21 - MongoDB](../21-springboot-mongodb/README.md)  
> **Next Topic:** [23 - Cloud Deployment →](../23-cloud-deployment/README.md)
