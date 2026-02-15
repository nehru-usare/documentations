# Docker Compose Production Patterns

> **Part 2: Best Practices**  
> **Difficulty:** ⭐⭐⭐ (Ops)  
> **Status:** Orchestrated

---

## 0. Learning Objectives
*   **Beginner**: Why `docker-compose up` is magic.
*   **Developer**: managing Dev/Test/Prod flavors in one file.
*   **Architect**: Using Compose as a poor man's Kubernetes.

---

## 1. Core Concepts (🟢 Beginner Level)

### The Manifest
*   `docker-compose.yml`: Defines Services, Networks, and Volumes.
*   **Declarative**: "I want 3 services". Docker makes it happen.

---

## 2. Architecture Breakdown (Patterns)

### 1. Extension Fields (DRY)
*   Don't copy-paste `environment`, `restart: always` for 10 services.
*   Use `x-common-config`:
    ```yaml
    x-logging: &default-logging
      driver: "json-file"
      options:
        max-size: "10m"

    services:
      web:
        <<: *default-logging
      db:
        <<: *default-logging
    ```

### 2. Profiles (Dev vs Prod)
*   You don't need `Prometheus` and `Grafana` in local dev.
*   Tag services with `profiles: ["monitoring"]`.
*   `docker-compose --profile monitoring up`.

### 3. Healthchecks & Dependency
*   **Problem**: App starts before DB is ready -> Crash.
*   **Fix**:
    ```yaml
    services:
      db:
        healthcheck:
          test: ["CMD", "pg_isready"]
          interval: 10s
      app:
        depends_on:
          db:
            condition: service_healthy
    ```

---

## 3. Internal Mechanics (🔴 Architect Level)

### 1. Override Files
*   `docker-compose.yml` (Base).
*   `docker-compose.override.yml` (Local Dev - mounts volumes).
*   `docker-compose.prod.yml` (Production - restart policies).
*   **Command**: `docker-compose -f docker-compose.yml -f docker-compose.prod.yml up`.

---

## 4. Trade-Off Analysis

| Feature | Compose | Kubernetes |
| :--- | :--- | :--- |
| **Simplicity** | High (Single binary) | Low (Control Plane) |
| **Scaling** | Single Node only | Multi-Node |
| **Rolling Update**| Basic (`up -d`) | Native |

---

## 5. Scenarios

### The "Local Stack"
*   Run everything (Redis, Kafka, Postgres, App) with one command.
*   Wait for Healthchecks.
*   Run Integration Tests.
*   Tear down.
*   *Perfect for CI/CD*.

---

## 6. Interview Questions

### Basic
1.  What is `docker-compose.yml`?
2.  How to scale a service? (`--scale web=3`).
3.  How to persist data? (Volumes).

### Intermediate
1.  Explain `depends_on`. Does it wait for TCP port? (No, unless `condition: service_healthy`).
2.  How to share variables between services? (.env file).
3.  What are "Projects" in compose?

### Advanced
1.  How to run docker-compose inside a container (DinD) for CI?
2.  Explain Network Aliases.
3.  Use `tmpfs` mounts for speed in testing.

---

## 7. Summary & Architect Takeaways

1.  **Keep it Dry**: Use Anchors (`&`) and Aliases (`*`).
2.  **Healthchecks**: Mandatory for reliable startup order.
3.  **Env Files**: Never hardcode secrets in YAML. Use `.env`.
