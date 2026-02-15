# Helm Charts (Package Manager)

> **Part 4: Container Orchestration**  
> **Difficulty:** ⭐⭐⭐ (Standard)  
> **Status:** Installing...

---

## 0. Learning Objectives
*   **Beginner**: `helm install my-app`.
*   **Developer**: Parameterizing K8s YAMLs using `values.yaml`.
*   **Architect**: Managing standard application templates for the org.

---

## 1. Context
**Helm**: The Package Manager for Kubernetes (Like `apt` or `npm` but for K8s apps).
*   **Problem**: K8s YAMLs are hardcoded. To deploy `app-v1` and `app-v2`, you need to copy-paste files.
*   **Solution**: Templates.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. The Structure
*   `Chart.yaml`: Metadata (Name, Version).
*   `values.yaml`: Default configuration variables.
*   `templates/`: The K8s YAMLs with placeholders.

### 2. Templating
*   **Classic YAML**: `image: nginx:1.14`
*   **Helm Template**: `image: {{ .Values.image.repository }}:{{ .Values.image.tag }}`

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The Release Lifecycle
1.  **Install**: `helm install release-name ./my-chart`.
2.  **Upgrade**: Change values -> `helm upgrade release-name ./my-chart`.
3.  **Rollback**: `helm rollback release-name 1`.
4.  **Uninstall**: `helm uninstall release-name`.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. How Helm Stores State
*   Helm 3 stores release history as **Kubernetes Secrets** in the namespace.
*   No external database needed.
*   It compares New Chart vs Old Release Secret to calculate patch.

### 2. Dependency Management
*   Charts can depend on other charts (Subcharts).
*   *Application Chart* depends on *Redis Chart*.
*   `helm dependency update`.

---

## 5. Trade-Off Analysis

| Approach | Pros | Cons |
| :--- | :--- | :--- |
| **Raw YAML** | Simple, No tools | Duplication, Hard to manage Env diffs |
| **Helm** | Templating, Packaging | Go Template syntax is ugly |
| **Kustomize** | Overlay model (No templates) | Verbose directory structure |

---

## 6. Scaling Considerations

### The "Library Chart" Pattern
*   Don't let every team write their own Deployment YAML.
*   Create a **Global Library Chart** that defines the "Company Standard Deployment" (Logs, Metrics, Security built-in).
*   Teams just provide `values.yaml` and inherit the Library.

---

## 7. Failure Scenarios & Recovery

### 1. Failed Upgrade
*   `helm upgrade` fails midway.
*   Helm tries to revert, but sometimes leaves resources in "Pending".
*   **Fix**: `helm rollback`.

---

## 8. Security Considerations

### 1. Image Provenance
*   Helm charts pull docker images.
*   Ensure `values.yaml` points to trusted registries, not public DockerHub.

---

## 9. Performance Considerations

*   **Rendering**:
    *   `helm template .` renders locally. Fast debugging.
    *   Use this in CI to validate YAMLs before deploying.

---

## 10. Real Production Lessons

### Config Drift
*   If someone edits a Helm-managed resource via `kubectl edit`.
*   Helm *might* overwrite it on next upgrade, or ignore it (3-way merge).
*   **Advice**: If using Helm, ONLY use Helm.

---

## 11. Interview Questions

### Basic
1.  What is Helm?
2.  Purpose of `values.yaml`.
3.  How to rollback a release?

### Intermediate
1.  Helm vs Kustomize.
2.  Where does Helm store state? (Secrets).
3.  How to debug a template? (`--dry-run --debug`).

### Advanced
1.  Architect a "Library Chart" strategy.
2.  How to handle Secrets in Helm? (Don't put in values.yaml. Intergate with External Secrets Operator).
3.  Explain the Helm upgrade 3-way merge logic.

---

## 12. Summary & Architect Takeaways

1.  **Don't Hardcode**: If a value changes between Dev and Prod, it belongs in `values.yaml`.
2.  **Version Everything**: Version the Chart *and* the App.
3.  **Standardize**: Use Helm to enforce best practices (Labels, Probes) across all teams.
