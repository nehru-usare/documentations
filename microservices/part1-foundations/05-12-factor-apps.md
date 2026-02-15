# 05. The 12-Factor App Principles

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐ (Developer)  
> **Status:** Industry Standard

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Learn the rules for building "Cloud Native" apps. |
| **Developer** | Apply these rules to Spring Boot (Profiles, Env Vars). |
| **Architect** | Enforce these principles to ensure portability (K8s ready). |

---

## 1. Why This Topic Exists

### The "Works on my Machine" Problem
In the old days, we SSH'd into servers, edited config files manually, and relied on "Snowflake Servers".
In the Cloud (AWS/K8s), servers are **Cattle, not Pets**. They die and restart.
The **12-Factor Methodology** (Heroku) is a set of rules to make apps portable, scalable, and resilient.

---

## 3. Core Concepts (The 12 Factors)

### 1. Codebase
**One codebase tracked in revision control, many deploys.**
*   *Rule*: One Git Repo = One App.
*   *Anti-Pattern*: Multiple apps in one repo (unless Monorepo with strict tooling).

### 2. Dependencies
**Explicitly declare and isolate dependencies.**
*   *Do*: Maven `pom.xml`, Gradle `build.gradle`.
*   *Don't*: Rely on system-wide JARs or `CLASSPATH`.

### 3. Config
**Store config in the environment.**
*   *Rule*: Code stays same. Config chances per environment (Dev/QA/Prod).
*   *Spring Boot*: `application.properties` reading `${DB_PASSWORD}` from OS Env.
*   *Anti-Pattern*: Hardcoding passwords or IPs.

### 4. Backing Services
**Treat backing services as attached resources.**
*   *Concept*: Database, RabbitMQ, SMTP are just URLs.
*   *Benefit*: You can swap a local MySQL for AWS RDS by just changing the URL (Config).

### 5. Build, Release, Run
**Strictly separate build and run stages.**
*   **Build**: Compile code -> Artifact (JAR/Docker Image).
*   **Release**: Artifact + Config -> Release v1.
*   **Run**: Execute Release v1.
*   *Rule*: You cannot change code at Runtime.

### 6. Processes
**Execute the app as one or more stateless processes.**
*   *Rule*: No Sticky Sessions. No local file storage (it vanishes on restart).
*   *Store state*: In Redis or DB.

### 7. Port Binding
**Export services via port binding.**
*   *Rule*: App should listen on `PORT` (e.g., 8080).
*   *Spring Boot*: Built-in Tomcat makes this easy.

### 8. Concurrency
**Scale out via the process model.**
*   *Rule*: Scale X-axis. Run 10 copies of the process.

### 9. Disposability
**Maximize robustness with fast startup and graceful shutdown.**
*   *Rule*: App should start in seconds.
*   *SIGTERM*: App should finish current requests and exit cleanly.

### 10. Dev/Prod Parity
**Keep development, staging, and production as similar as possible.**
*   *Don't*: Use SQLite in Dev and Oracle in Prod. Use Docker Compose to run Oracle in Dev.

### 11. Logs
**Treat logs as event streams.**
*   *Rule*: App writes to `STDOUT`. Infrastructure (Fluentd) collects/routes it.
*   *Don't*: App writing to `/var/log/app.log`.

### 12. Admin Processes
**Run admin/management tasks as one-off processes.**
*   *Example*: Database migrations (`Flyway`).

---

## 9. Architect-Level Best Practices

1.  **Strict Env Var Policy**: Reject any PR that hardcodes config.
2.  **Docker is the Enforcer**: Docker containers naturally enforce many factors (Port Binding, Dependencies, Processes).
3.  **Externalize Everything**: If AWS Region goes down, I should be able to deploy to GCP just by changing Config Envs.

---

## 12. Interview Questions

1.  Why should config be stored in Environment Variables?
2.  What is a "Backing Service"?
3.  Why are "Sticky Sessions" bad for scaling?
4.  Explain "Graceful Shutdown".
