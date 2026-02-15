# IaC Principles (Immutable vs Mutable)

> **Part 3: Infrastructure as Code**  
> **Difficulty:** ⭐⭐ (Concept)  
> **Status:** Applying...

---

## 0. Learning Objectives
*   **Beginner**: Why clicking buttons in AWS Console is a fireable offense.
*   **Developer**: Difference between "Configuring a Server" and "Replacing a Server".
*   **Architect**: Choosing between Terraform and Ansible.

---

## 1. Core Concepts (🟢 Beginner Level)

### 1. What is IaC?
*   Managing infrastructure (Servers, Networks, DBs) using configuration files (Code).
*   **Key Benefit**: Reproducibility. Run the code 100 times -> Get 100 identical environments.

### 2. Imperative vs Declarative
*   **Imperative (How)**: "Go to AWS. Click EC2. Select Ubuntu. Launch." (Scripting / CLI).
*   **Declarative (What)**: "I want 1 Ubuntu EC2 instance." (Terraform / K8s).
    *   The tool figures out the "How". If it already exists, it does nothing.

---

## 2. Architecture Breakdown (🟡 Developer Level)

### Mutable vs Immutable Infrastructure
#### 1. Mutable (The Pet Model)
*   **Tool**: Ansible, Chef, Puppet.
*   **Process**: Spin up VM -> SSH in -> Install Java -> Upgrade Libs.
*   **Problem**: **Configuration Drift**. Over years, manual hacks accumulate. Server becomes a "Snowflake". You are afraid to kill it.

#### 2. Immutable (The Cattle Model)
*   **Tool**: Terraform + Packer + Docker.
*   **Process**: Bake a Golden Image (AMI/Docker) with Java installed. Deploy it.
*   **Update**: Don't upgrade in-place. **Destroy** old VM. **Deploy** new VM with new Image.
*   **Benefit**: No drift. Confidence that Prod == Staging.

---

## 3. Internal Mechanics (🔴 Architect Level)

### 1. Idempotency
*   The property that running an operation multiple times has the same result as running it once.
*   `mkdir /tmp/foo` is NOT idempotent (Fails 2nd time).
*   `mkdir -p /tmp/foo` IS idempotent.
*   **Ansible/Terraform** are designed to be idempotent.

### 2. The Provisioning vs Configuration Split
*   **Provisioning (Creating Integration)**: Terraform / CloudFormation. (Create VPC, Subnet, EC2).
*   **Configuration (Setup Inside OS)**: Ansible / SaltStack. (Install Nginx, Edit `conf.d`).
*   **Modern Trend**: Use Terraform for everything + Docker for apps. Minimize Ansible (User Data scripts).

---

## 4. Trade-Off Analysis

| Feature | Mutable (Ansible) | Immutable (Terraform/AMI) |
| :--- | :--- | :--- |
| **Speed** | Fast (In-place update) | Slow (Bake image + Boot) |
| **Consistency** | Low (Drift risks) | **High** (Bit-perfect copy) |
| **Rollback** | Hard (Uninstall?) | Easy (Switch to old Image) |
| **State** | None | **State File** (Complex) |

---

## 5. Scaling Considerations

### State Management
*   Terraform tracks the state of the world in a `tfstate` file.
*   **Risk**: If you delete this file, Terraform forgets it manages your Prod DB.
*   **Fix**: Remote State (S3 + DynamoDB Locking).

---

## 6. Failure Scenarios & Recovery

### 1. Manual Changes (ClickOps)
*   Dev manually changes Security Group in AWS Console to debug.
*   **Drift**: Terraform code says Port 80. Real Cloud says Port 80 & 22.
*   **Resolution**: Run `terraform plan`. Detect drift. Overwrite manual change.

---

## 7. Security Considerations

### 1. Least Privilege
*   The CI/CD runner executing Terraform needs `AdministratorAccess` to AWS.
*   **Risk**: High value target.
*   **Mitigation**: Short-lived credentials (OIDC/AssumeRole).

---

## 8. Performance Considerations

*   **Plan Speed**: Large Terraform monorepos take 20 mins to `plan`.
*   **Fix**: Split state files. One state per environment/service.

---

## 9. Real Production Lessons

### HashiCorp
*   **Golden Rule**: "Workflow > Tool".
*   The goal isn't to use Terraform; the goal is to have a codified review process for infra changes.

---

## 10. Interview Questions

### Basic
1.  What is IaC?
2.  Difference between Ansible and Terraform.
3.  What is Idempotency?

### Intermediate
1.  Explain Immutable Infrastructure.
2.  What is Configuration Drift?
3.  Why store Terraform State remotely?

### Advanced
1.  Architect a "Zero-Touch" production environment.
2.  How to handle Secret rotation in Immutable Infrastructure?
3.  Compare "Pull Mode" (Chef) vs "Push Mode" (Ansible).

---

## 11. Summary & Architect Takeaways

1.  **Stop SSH-ing**: If you have to SSH into a server to fix it, you failed.
2.  **Cattle not Pets**: Give servers IDs, not names. Kill them often.
3.  **Code Review Infra**: Infra changes cause 80% of outages. Review them like code.
