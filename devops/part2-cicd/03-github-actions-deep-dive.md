# GitHub Actions Deep Dive

> **Part 2: CI/CD Pipelines**  
> **Difficulty:** ⭐⭐ (Modern Standard)  
> **Status:** The New King

---

## 0. Learning Objectives
*   **Beginner**: Creating a `.github/workflows/main.yml`.
*   **Developer**: Using Marketplace Actions to save time.
*   **Architect**: Creating custom Composite Actions for standardization.

---

## 1. Context
**GitHub Actions**: Integrated CI/CD within GitHub.
*   **Philosophy**: "Events" happen in Repo -> "Workflows" respond.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Hierarchy
*   **Workflow**: The YAML file. Triggered by Event.
*   **Job**: A runner (VM). Runs in parallel by default.
*   **Step**: A command (`run: npm test`) or an Action (`uses: actions/checkout`).

### 2. Events
*   `on: push`: Runs on commit.
*   `on: pull_request`: Runs on PR open/sync.
*   `on: workflow_dispatch`: Manual button click.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### 1. Matrix Builds
*   Run the same job across multiple OS/Versions.
    ```yaml
    strategy:
      matrix:
        node: [14, 16, 18]
        os: [ubuntu-latest, windows-latest]
    ```
    *   Creates $3 \times 2 = 6$ parallel jobs.

### 2. Services
*   Need a DB for testing?
    ```yaml
    services:
      postgres:
        image: postgres:13
    ```

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Custom Actions (Composite)
*   **Problem**: Copy-pasting 50 lines of YAML to "Setup Java + Maven + AWS Creds".
*   **Solution**: Create a `action.yml` in a separate repo (e.g., `my-org/setup-standard-env`).
*   **Usage**: `uses: my-org/setup-standard-env@v1`.

### 2. Self-Hosted Runners
*   GitHub provides hosted runners (Ubuntu). Expensive and Public IP.
*   **Self-Hosted**: Run the agent on your own AWS EC2 / K8s.
*   *Security Risk*: Don't use self-hosted on Public Repos (Crypto mining attacks).

---

## 5. Trade-Off Analysis

| Feature | GHA Hosted | GHA Self-Hosted |
| :--- | :--- | :--- |
| **Cost** | Per minute | Infrastructure cost |
| **Security** | Isolated VMs | Inside your VPC |
| **Maintenance** | Zero | High (Update OS/Docker) |

---

## 6. Scaling Considerations

### Reusable Workflows
*   Define a "Deploy Workflow" once.
*   Call it from 50 repos using `uses: my-org/workflows/.github/workflows/deploy.yml`.
*   Pass inputs (Environment, Version).

---

## 7. Failure Scenarios & Recovery

### 1. Rate Limits
*   GitHub API has rate limits. Big monorepos hit this.
*   **Fix**: `GITHUB_TOKEN` limitation. Use a GitHub App for higher limits.

---

## 8. Security Considerations

### 1. OIDC Connect
*   Don't store AWS Keys ($AWS_ACCESS_KEY_ID$) in GitHub Secrets.
*   **Use OIDC**: GitHub assumes a Role in AWS temporarily.
    *   "I am repo X, on branch main. Give me access."
    *   No static keys to rotate/leak.

---

## 9. Performance Considerations

*   **Cache Action**:
    ```yaml
    - uses: actions/cache@v3
      with:
        path: ~/.m2/repository
        key: maven-${{ hashFiles('pom.xml') }}
    ```
    *   Saves massive bandwidth and time.

---

## 10. Real Production Lessons

### Best Practice
*   Use `dependabot` to update Action versions. `uses: actions/checkout@v2` -> `v3`.

---

## 11. Interview Questions

### Basic
1.  Where do you perform GitHub Actions? (`.github/workflows`).
2.  How to trigger on PR?
3.  What is `uses` keyword?

### Intermediate
1.  Explain Matrix Strategy.
2.  How to pass data between jobs? (Artifacts: `upload-artifact` / `download-artifact`).
3.  What happens if previous step fails? (Default: Job stops. `if: always()` to force run).

### Advanced
1.  Architect a secure OIDC connection to AWS/Azure.
2.  Design a Reusable Workflow strategy for a Microservices org.
3.  Security implications of Self-Hosted Runners on public repos.

---

## 12. Summary & Architect Takeaways

1.  **Simplicity**: YAML is easier than Groovy.
2.  **Integration**: Being inside GitHub is a huge DX (Developer Experience) win.
3.  **Marketplace**: Don't reinvent the wheel. Someone already wrote the "Upload to S3" action.
