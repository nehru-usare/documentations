# Docker Image Optimization

> **Part 2: Best Practices**  
> **Difficulty:** ⭐⭐⭐ (Optimization)  
> **Status:** Compressed

---

## 0. Learning Objectives
*   **Beginner**: Why your `Hello World` image is 800MB.
*   **Developer**: Reducing image size by 90% using Multi-Stage builds.
*   **Architect**: Securing supply chain with Distroless images.

---

## 1. Core Concepts (🟢 Beginner Level)

### The Layer Cake
*   `FROM ubuntu` (Layer 1 - 70MB)
*   `RUN apt-get install python` (Layer 2 - 50MB)
*   `COPY . .` (Layer 3 - 2MB)
*   **Total Size**: Sum of all layers.
*   **Cache**: Docker caches layers. If Layer 3 changes, Layer 1 & 2 are reused.

---

## 2. Architecture Breakdown (Techniques)

### 1. Multi-Stage Builds (The Silver Bullet)
*   **Problem**: You need Maven/GCC to build, but only the JAR/Binary to run.
*   **Solution**:
    ```dockerfile
    # Stage 1: Build
    FROM maven AS builder
    WORKDIR /app
    COPY . .
    RUN mvn package

    # Stage 2: Run
    FROM openjdk:jre-alpine
    COPY --from=builder /app/target/app.jar .
    CMD ["java", "-jar", "app.jar"]
    ```
*   **Result**: Image drops from ~600MB (Maven) to ~80MB (JRE + Jar).

### 2. Distroless Images (Google)
*   Images that contain **only** your app and its runtime dependencies.
*   No Shell (`/bin/sh`). No Package Manager (`apt`).
*   *Security Win*: Hacker can't prevent `bash`, because `bash` isn't there.

---

## 3. Internal Mechanics (🔴 Architect Level)

### 1. The Cache Invalidating Order
*   **Bad**:
    ```dockerfile
    COPY . .
    RUN npm install
    ```
    (Every code change invalidates `npm install`. Slow builds).

*   **Good**:
    ```dockerfile
    COPY package.json .
    RUN npm install
    COPY . .
    ```
    (Code change only invalidates last step. fast builds).

---

## 4. Trade-Off Analysis

| Base Image | Size | Compat | Security |
| :--- | :--- | :--- | :--- |
| **Ubuntu/Debian** | Large (100MB+) | High | Med (Large attack surface) |
| **Alpine** | Tiny (5MB) | Med (Musl libc quirks) | High |
| **Distroless** | Tiny (20MB) | Low (No shell) | Very High |

---

## 5. Scaling Considerations

### Network Bandwidth
*   Small images = Faster `docker pull`.
*   Crucial for Autoscaling (K8s spins up new node -> Pulls image -> Starts Pod).
*   Big image = Slow scale up.

---

## 6. Real Production Lessons

### "The Alpine DNS Issue"
*   Alpine uses `musl` libc instead of `glibc`.
*   It handles DNS slightly differently. Can cause issues in some environments.
*   **Lesson**: Test Alpine thoroughly before Prod.

---

## 7. Interview Questions

### Basic
1.  Why are small images better? (Security, Speed, Storage).
2.  What is `.dockerignore`? (Like .gitignore, prevents sending huge `node_modules` to build context).
3.  Difference between `COPY` and `ADD`. (ADD is dangerous, can unzip/fetch URL).

### Intermediate
1.  Explain Multi-Stage builds.
2.  Why sort `apt-get install` arguments?
3.  How to flatten an image? (`docker export | docker import`).

### Advanced
1.  Explain the difference between `musl` and `glibc`.
2.  How to debug a container that has no shell (Distroless)? (`kubectl debug` ephemeral containers).
3.  What is a Dangling Image? (`<none>`).

---

## 8. Summary & Architect Takeaways

1.  **Order Matters**: Optimize Dockerfile specific to least specific.
2.  **Combine RUNs**: `RUN apt-get update && apt-get install -y foo && rm -rf /var/lib/apt/lists/*`.
3.  **Don't run as Root**: Always `USER appuser` at the end.
