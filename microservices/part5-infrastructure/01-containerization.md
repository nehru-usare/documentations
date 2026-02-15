# 01. Containerization (Docker, Jib, Multi-stage builds)

> **Part 5: Infrastructure & DevOps**  
> **Difficulty:** ⭐⭐⭐ (Developer)  
> **Status:** Industry Standard

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Run a Java app inside a Container. |
| **Developer** | Write efficient `Dockerfile` (Multi-stage). |
| **Architect** | Optimize Security (Distroless) and Build Speed (Layer Caching). |

---

## 1. Why This Topic Exists

### The Matrix from Hell
*   **Dev**: Java 17, Ubuntu.
*   **QA**: Java 11, CentOS.
*   **Prod**: Java 8, RHEL.
*   **Result**: "It works on my machine".

### The Solution: Containerization
Package the OS + JVM + App + Libs into one artifact (Image).
Run it anywhere (Laptop, AWs, Azure). It behaves exactly the same.

---

## 2. Big Picture Architecture View

```mermaid
graph TD
    Code[Java Code] -->|Compile| JAR[App.jar]
    
    subgraph "Docker Build"
        Base[Base Image (OpenJDK)] --> Layer1
        JAR --> Layer2[App Layer]
        Layer1 --> Final[Docker Image]
        Layer2 --> Final
    end
    
    Final -->|Push| Registry[Docker Hub / ECR]
    Registry -->|Pull| Server[Kubernetes Node]
```

---

## 3. Core Concepts (🟢 Beginner Level)

### Image vs Container
*   **Image**: The Class. (ReadOnly Template).
*   **Container**: The Object. (Running Instance).

### Layers
Docker uses a Copy-on-Write Union File System.
*   Layer 1: OS (Ubuntu) - 100MB
*   Layer 2: JDK - 200MB
*   Layer 3: App.jar - 50MB
*   *Advantage*: If you update App.jar, you only re-download Layer 3.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Best Practice: Multi-Stage Build
Don't ship Maven/Gradle in Production.

```dockerfile
# Stage 1: Build
FROM maven:3.8-openjdk-17 AS builder
WORKDIR /app
COPY . .
RUN mvn clean package -DskipTests

# Stage 2: Run (Distroless / Slim)
FROM openjdk:17-slim
WORKDIR /app
COPY --from=builder /app/target/myapp.jar app.jar
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Google Jib (No Dockerfile)
Builds Java images *without* a Docker Daemon.
*   *Plugin*: `jib-maven-plugin`.
*   *Command*: `mvn jib:build`.
*   *Pros*: Fast, reproducible, splits dependencies into separate layers automatically.

---

## 5. Security & Optimization (🔴 Architect Level)

### Distroless Images
Google's "Distroless" images contain **only** the Application and Runtime (JVM).
*   No Shell (`/bin/bash`).
*   No Package Manager (`apt-get`).
*   **Benefit**: If a hacker gets in, they can't run commands.

### Layer Ordering
Put least changing things first.
1.  OS (Rarely changes).
2.  Dependencies (Changes monthly).
3.  Source Code (Changes hourly).
*   This maximizes Docker Cache hits.

---

## 9. Architect-Level Best Practices

1.  **One Process Per Container**: Don't run SSH + Cron + Java. Just Java.
2.  **Immutable Tags**: Never deploy `latest` to Prod. Use SHA or Version (`v1.0.2`).
3.  **Rootless**: Run as non-root user (`USER 1000`). If container is breached, Host is safe.

---

## 12. Interview Questions

1.  Difference between VM and Container? (OS sharing).
2.  How to reduce Docker Image size?
3.  What is a Multi-Stage build?

---

## 14. Summary & Architect Takeaways

*   **Standard Unit**: The Container is the new JAR.
*   **Security**: Minimal base images = Minimal attack surface.
*   **Speed**: Optimize for caching. Developers hate slow builds.
