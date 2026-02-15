# Terraform State & Modules

> **Part 3: Infrastructure as Code**  
> **Difficulty:** ⭐⭐⭐ (Standard)  
> **Status:** Plan Applied

---

## 0. Learning Objectives
*   **Beginner**: Writing your first `main.tf`.
*   **Developer**: Understanding why `terraform.tfstate` is dangerous.
*   **Architect**: Designing reusable Modules for the whole company.

---

## 1. Context
**Terraform**: The industry standard for provisioning cloud infrastructure.
*   **Multi-Cloud**: Works with AWS, Azure, GCP, Kubernetes, etc.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. HCL (HashiCorp Config Language)
*   Declarative syntax.
    ```hcl
    resource "aws_s3_bucket" "my_bucket" {
      bucket = "my-app-logs"
    }
    ```

### 2. The Lifecycle
1.  `terraform init`: Download providers (plugins).
2.  `terraform plan`: Dry run. Shows what *will* happen. (+1 to add, -1 to destroy).
3.  `terraform apply`: Execute the changes. Updates State file.
4.  `terraform destroy`: Kill everything.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The State File (`terraform.tfstate`)
*   **Purpose**: Maps your HCL code to real world resource IDs (e.g., `i-123456`).
*   **JSON format**: Contains sensitive data (DB passwords). **Encrypt it**.
*   **Location**:
    *   *Local*: Bad. (Laptop drives die).
    *   *Remote (Standard)*: AWS S3 Bucket (Storage) + DynamoDB Table (Locking).

### Why Locking?
*   Prevents two developers from running `apply` at the same time and corrupting the state.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Modules (DRY Principle)
*   Don't copy-paste the "EC2 + Security Group + IAM Role" block 50 times.
*   **Create a Module**: `modules/standard-web-server`.
*   **Usage**:
    ```hcl
    module "web" {
      source = "./modules/standard-web-server"
      instance_count = 3
    }
    ```

### 2. Dependency Graph
*   Terraform builds a DAG (Directed Acyclic Graph) of resources.
*   It knows: "VPC must exist *before* Subnet". "Subnet *before* EC2".
*   It parallelizes independent creations (making it fast).

---

## 5. Trade-Off Analysis

| State Strategy | Pros | Cons |
| :--- | :--- | :--- |
| **Local State** | Easy, Fast | No collab, Risk of loss |
| **Remote State (S3)** | Safe, Locking | Setup complexity, Network dep |
| **Terraform Cloud** | UI, History, Policy | Money |

---

## 6. Scaling Considerations

### Workspaces / Folder Structure
*   **Workspaces**: `terraform workspace new prod`. (Same code, different state).
    *   *Risk*: Easy to accidentally apply to Prod thinking it's Dev.
*   **Folder Structure (Recommended)**:
    *   `envs/dev/main.tf`
    *   `envs/prod/main.tf`
    *   Explicit separation.

---

## 7. Failure Scenarios & Recovery

### 1. Corrupted State
*   You manually deleted an EC2, but State thinks it exists.
*   `terraform apply` fails: "Resource not found".
*   **Fix**: `terraform refresh` (Syncs state with reality) or `terraform state rm` (Manually edit state).

---

## 8. Security Considerations

### 1. Sensitive Outputs
*   `output "db_password"` prints to console.
*   Use `sensitive = true` to mask it.
*   *Note*: It is still visible in the `.tfstate` JSON file! Restrict S3 access.

---

## 9. Performance Considerations

*   **Plan File**:
    *   Always use `terraform plan -out=tfplan`.
    *   `terraform apply tfplan`.
    *   Guarantees that what you saw in Plan is exactly what gets Applied (prevents race conditions).

---

## 10. Real Production Lessons

### Monolithic State Limit
*   Putting 1000 resources in one state file makes `plan` take 20 minutes.
*   **Blast Radius**: One error breaks everything.
*   **Fix**: Split state by Domain (Network State, App State, Data State).

---

## 11. Interview Questions

### Basic
1.  What is `terraform plan`?
2.  Why do we need a State file?
3.  What is a Provider?

### Intermediate
1.  How to handle locking in S3 backend? (DynamoDB).
2.  Explain Modules.
3.  Output Variables purpose?

### Advanced
1.  Architect a Directory Structure for Multi-Region Multi-Env Terraform.
2.  How to import existing manually created resources into Terraform (`terraform import`)?
3.  Critique Terraform vs CloudFormation. (TF is cloud-agnostic, CF is AWS native but safer rollback).

---

## 12. Summary & Architect Takeaways

1.  **Read the Plan**: Never auto-approve. The Plan tells you if you are about to delete the Production Database.
2.  **Modules**: Build a library of "Approved Patterns".
3.  **State Hygiene**: Protect the state file with your life (Encryption + Versioning).
