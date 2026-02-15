# Jenkins Architecture

> **Part 2: CI/CD Pipelines**  
> **Difficulty:** ⭐⭐⭐ (Legacy Standard)  
> **Status:** The Old Butler

---

## 0. Learning Objectives
*   **Beginner**: Running a "Freestyle Job".
*   **Developer**: Writing Declarative Pipelines (`Jenkinsfile`).
*   **Architect**: Scaling Jenkins with ephemeral agents on Kubernetes.

---

## 1. Context
**Jenkins** is the most popular open-source CI automation server.
*   **Pros**: Plugins for everything. Huge community.
*   **Cons**: XML-based config hell. Maintenance burden. "Jar hell".

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Master-Agent Architecture
*   **Master (Controller)**: Stores config, handles GUI, schedules jobs. *Do not run builds here*.
*   **Agent (Slave)**: Executes the build. Can be Linux, Windows, Docker.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The Jenkinsfile (Pipeline as Code)
*   **Declarative Syntax** (Recommended):
    ```groovy
    pipeline {
        agent any
        stages {
            stage('Build') {
                steps {
                    sh 'mvn clean package'
                }
            }
        }
    }
    ```
*   **Scripted Syntax**: Full Groovy power. Harder to read. Used for complex logic.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Scaling on Kubernetes
*   Install `kubernetes-plugin`.
*   When a job starts, Jenkins asks K8s for a Pod.
*   **Pod Template**: Defines the build environment (e.g., `maven:3.8-jdk-17`).
*   Job runs in Pod. Pod dies.
*   **Benefit**: Infinite scale. Zero idle cost. Clean environment.

### 2. Shared Libraries
*   Don't copy-paste `Jenkinsfile` logic across 100 repos.
*   Create a Git Repo `jenkins-shared-lib`.
*   Define global functions `buildJavaApp()`.
*   Repos just call `buildJavaApp()`.

---

## 5. Trade-Off Analysis

| Feature | Jenkins | GitHub Actions |
| :--- | :--- | :--- |
| **Hosting** | Self-Hosted (Painful) | SaaS (Easy) |
| **Config** | Groovy | YAML |
| **Scale** | Agents | Runners |
| **Cost** | Server Cost | Per Minute Billing |

---

## 6. Scaling Considerations

### The "Jenkins Master" Bottleneck
*   The Master is a Monolith.
*   If it goes down, no one can deploy.
*   **HA Setup**: Active-Passive is standard. Active-Active is hard (CloudBees Feature).

---

## 7. Failure Scenarios & Recovery

### 1. Plugin Hell
*   Upgrading one plugin breaks 5 others.
*   **Fix**: Immutable Jenkins (Configuration as Code - JCasC). Build a Docker image with pre-installed plugins.

---

## 8. Security Considerations

### 1. Script Approval
*   Jenkins runs arbitrary Groovy code.
*   Sandbox prevents malicious code. Admins must "Approve" scripts.
*   **CVEs**: Jenkins has many. Patch frequently. Don't expose to public internet.

---

## 9. Performance Considerations

*   **Workspace Cleanup**: Jobs leave 5GB files. Disk fills up.
*   **Fix**: `wsCleanup()` step. Monitor disk usage.

---

## 10. Real Production Lessons

### Enterprise Usage
*   Big banks still use Jenkins because it runs *inside* their firewall and listens to internal Git.
*   SaaS (managed CI) is banned in high-security zones.

---

## 11. Interview Questions

### Basic
1.  What is a Jenkinsfile?
2.  Difference between Master and Agent.
3.  Declarative vs Scripted pipeline.

### Intermediate
1.  How to persist data (artifacts) after a build? (Archive artifacts / Push to Nexus).
2.  What are Shared Libraries?
3.  How to secure a Jenkins instance?

### Advanced
1.  Architect a Jenkins HA setup.
2.  How to implement "Configuration as Code" (JCasC)?
3.  Compare Jenkins scaling on VM vs Kubernetes.

---

## 12. Summary & Architect Takeaways

1.  **Don't build on Master**: Always use Agents.
2.  **Pipeline as Code**: If it's not in `Jenkinsfile`, it's wrong.
3.  **Modernize**: If possible, use GitHub Actions / GitLab CI. Use Jenkins only if required by policy.
