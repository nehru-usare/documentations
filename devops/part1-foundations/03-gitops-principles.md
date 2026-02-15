# GitOps Principles

> **Part 1: Foundations**  
> **Difficulty:** ⭐⭐⭐ (Modern Standard)  
> **Status:** Git push production

---

## 0. Learning Objectives
*   **Beginner**: Why you shouldn't edit Kubernetes YAMLs manually.
*   **Developer**: How `git merge` can deploy your app.
*   **Architect**: Designing a secure, audited, and rollback-friendly deployment pipeline using ArgoCD/Flux.

---

## 1. What is GitOps?
**GitOps** is a set of practices to manage infrastructure and application configurations using **Git**.
*   **Core Idea**: The Git repository is the **Single Source of Truth**.
*   If it's not in Git, it doesn't exist.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Declarative vs Imperative
*   **Imperative**: "Run this script to upgrade the server." (kubectl apply -f app.yaml).
*   **Declarative**: "The server *should* look like this YAML." (Kubernetes Reconciler).

### 2. The Reconciliation Loop
*   An agent (e.g., ArgoCD) sits inside the cluster.
*   It checks Git: "V2 is desired".
*   It checks Cluster: "V1 is running".
*   **Action**: Sync (Upgrade V1 -> V2).

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Push vs Pull Architecture
#### 1. CIOps (Push) - The Old Way
*   Jenkins Pipeline: `Build -> Test -> kubectl apply`.
*   *Risk*: Jenkins needs `admin` access to the Cluster. Security flaw.

#### 2. GitOps (Pull) - The New Way
*   Jenkins: `Build -> Test -> Push Docker Image -> Update Git Config (v1 -> v2)`.
*   **ArgoCD (Inside Cluster)**: Sees Git change. Pulls new config. Applies it.
*   *Security*: Cluster doesn't know Jenkins exists. No external credentials needed.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Drift Detection
*   What if someone manually edits the Cluster (`kubectl edit deployment`)?
*   **Push**: Jenkins doesn't know. Config and Prod diverge.
*   **GitOps (Pull)**: ArgoCD detects "Out of Sync".
    *   *Option A*: Alert (Drift detected).
    *   *Option B*: **Auto-Heal** (Immediately revert manual change).

### 2. Multi-Environment Promotion
*   **Branch Strategy**: `main` = Prod, `develop` = Staging. (Bad practice).
*   **Folder Strategy**: `/overlays/staging`, `/overlays/prod` (Kustomize). (Best practice).
    *   Promote by copying YAML from Staging folder to Prod folder via PR.

---

## 5. Trade-Off Analysis

| Feature | CIOps (Jenkins Push) | GitOps (ArgoCD Pull) |
| :--- | :--- | :--- |
| **Security** | Low (Credentials leaked) | **High** (Cluster pulls) |
| **Visibility** | Pipeline logs | Git History |
| **Rollback** | Re-run old pipeline | `git revert` |
| **Complexity** | Low | Medium (New tool) |

---

## 6. Scaling Considerations

### Managing 100 Clusters
*   With GitOps, you can update 100 clusters by updating 1 Git Repo.
*   Fleet management becomes trivial.

---

## 7. Failure Scenarios & Recovery

### 1. Bad Config Commit
*   Dev pushes bad YAML -> ArgoCD syncs -> App crashes.
*   **Fix**: `git revert`. ArgoCD syncs back to previous working state.
*   **Mean Time To Recovery (MTTR)**: Seconds.

---

## 8. Security Considerations

### 1. Access Control
*   Developers do NOT need `kubectl` access to Production.
*   They only need `git push` access.
*   Code Review = Operations Review.

---

## 9. Performance Considerations

*   **Sync Latency**: ArgoCD usually polls Git every 3 mins.
*   **Optimization**: Configure Webhooks from GitHub to ArgoCD for instant sync.

---

## 10. Real Production Lessons

### Intuit (TurboTax)
*   Created ArgoCD because Jenkins pipelines were unmanageable at scale.
*   Moved thousands of services to GitOps. Deployment incidents dropped.

---

## 11. Interview Questions

### Basic
1.  What is the "Single Source of Truth"?
2.  Difference between Push and Pull deployment?
3.  Why is manual `kubectl` bad?

### Intermediate
1.  Explain Drift Detection.
2.  How to handle Secrets in GitOps? (Sealed Secrets, Vault).
3.  How to rollback a failed deployment in GitOps?

### Advanced
1.  Architect a Multi-Cluster GitOps solution.
2.  How to handle Database Migrations in GitOps? (Pre-sync hooks).
3.  Compare Flux vs ArgoCD.

---

## 12. Summary & Architect Takeaways

1.  **Git is the UI**: The best UI for developers is Git.
2.  **Audit Trail**: Git History *is* your Deployment Audit log.
3.  **Security**: Stop giving CI servers root access to your cluster.
