# CI/CD Fundamentals

> **Part 2: CI/CD Pipelines**  
> **Difficulty:** ⭐ (Concept)  
> **Status:** Integrating...

---

## 0. Learning Objectives
*   **Beginner**: Difference between Delivery and Deployment.
*   **Developer**: Why "It works on my machine" is an invalid excuse.
*   **Architect**: Designing a pipeline that is fast, secure, and reliable.

---

## 1. Core Concepts (🟢 Beginner Level)

### 1. The Acronyms
*   **CI (Continuous Integration)**:
    *   *Trigger*: Git Push.
    *   *Action*: Build -> Unit Test -> Static Analysis.
    *   *Output*: Verified Artifact (Docker Image / JAR).
*   **CD (Continuous Delivery)**:
    *   *Action*: Auto-deploy to Staging. Manual approval to Production.
*   **CD (Continuous Deployment)**:
    *   *Action*: Auto-deploy to Production (No humans involved).
    *   *Requires*: Elite testing maturity.

---

## 2. Architecture Breakdown (🟡 Developer Level)

### The Anatomy of a Pipeline
1.  **Source Stage**: Watch Git Repo.
2.  **Build Stage**: Compile Code. Download Dependencies (Maven/npm).
3.  **Test Stage**: Run Unit Tests (`mvn test`).
4.  **Scan Stage**: SAST (SonarQube), Container Scan (Trivy).
5.  **Package Stage**: Build Docker Image. Push to Registry (Nexus/ECR).
6.  **Deploy Stage**: Update Kubernetes Manifest (Helm Upgrade).

---

## 3. Internal Mechanics (🔴 Architect Level)

### 1. Artifact Management
*   **Anti-Pattern**: Rebuilding the binary for Production.
*   **Pattern**: Build **Once**. Promote the **Same Binary** from Dev -> QA -> Prod.
*   *Why?* Rebuilding risks introducing differences (e.g., a dependency version changed in the last hour).

### 2. Ephemeral Build Agents
*   Don't run builds on a static Jenkins server.
*   Spin up a **Docker Container** as a build agent. Run build. Destroy container.
*   Ensures a clean environment every time.

---

## 4. Trade-Off Analysis

| Strategy | Speed | Safety |
| :--- | :--- | :--- |
| **Manual Deploy** | Slow | Low (Human error) |
| **Continuous Delivery** | Fast | High (Human check) |
| **Continuous Deployment**| **Instant** | **High** (If tests are good) |

---

## 5. Scaling Considerations

### Pipeline as Code
*   Don't click buttons in Jenkins UI.
*   Write a `Jenkinsfile` or `.github/workflows/deploy.yml`.
*   Version control your pipeline logic.

---

## 6. Failure Scenarios & Recovery

### 1. Broken Build
*   **Rule**: If the build is broken, fixing it is the top priority for the *entire team*.
*   Do not push new features on top of a broken build.

---

## 7. Security Considerations

### 1. Secrets Management
*   **Never** commit passwords to `Jenkinsfile`.
*   Use Jenkins Credentials Binding or GitHub Secrets.
*   Inject them as Environment Variables at runtime.

---

## 8. Performance Considerations

*   **Caching**:
    *   Downloading Maven/NPM dependencies takes 50% of build time.
    *   Cache the `~/.m2` or `node_modules` folder between builds.

---

## 9. Real Production Lessons

### Facebook
*   **Mainline Development**: Everyone commits to `main`.
*   No long-lived feature branches.
*   Requires powerful CI to prevent breakage.

---

## 10. Interview Questions

### Basic
1.  Difference between CI and CD?
2.  What is an Artifact Repository?
3.  Why "Build Once"?

### Intermediate
1.  Explain "Pipeline as Code".
2.  How to speed up a slow Maven build?
3.  What is a "Flaky Test"? How to handle it?

### Advanced
1.  Architect a secure pipeline for a Banking App (Compliance gates).
2.  How to implement "Fan-in" and "Fan-out" in pipelines?
3.  Design a CI system for a Monorepo (Google style).

---

## 11. Summary & Architect Takeaways

1.  **Consistency**: The pipeline ensures process consistency.
2.  **Fast Feedback**: Is it broken? Tell me in 5 minutes, not 5 days.
3.  **Artifacts are Immutable**: If it passed QA, that exact byte-stream goes to Prod.
