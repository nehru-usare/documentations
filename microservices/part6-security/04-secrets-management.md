# 04. Secrets Management (Vault, K8s Secrets)

> **Part 6: Security**  
> **Difficulty:** ⭐⭐⭐⭐ (DevSecOps)  
> **Status:** Mandatory

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Never commit passwords to Git. |
| **Developer** | Connect Spring Boot to HashiCorp Vault. |
| **Architect** | Design a rotation policy for credentials. |

---

## 1. Why This Topic Exists

### The Leak
If `DB_PASSWORD=secret` is in `application.properties` in Git, anyone with Repo access owns your database.
Even if repo is private, contractors, ex-employees, and CI logs have access.

---

## 3. Core Concepts (🟢 Beginner Level)

### Kubernetes Secrets
Native object.
*   *Issue*: It's just Base64. Anyone with `kubectl get secrets` can decode it.
*   *Fix*: Enable "Encryption at Rest" in Kubernetes (ETCD).

### External Secret Store
AWS Secrets Manager, Azure KeyVault, HashiCorp Vault.
*   Central secure storage.
*   Audit logs ("Who read the password?").

---

## 4. Developer Deep Dive (🟡 Professional Level)

### HashiCorp Vault + Spring Boot
Vault can generate **Dynamic Secrets**.
1.  App asks Vault for Postgres Creds.
2.  Vault creates `user_123` in Postgres with permission.
3.  Vault gives `user_123` to App.
4.  After 1 hour, Vault deletes `user_123`.
*   *Security*: Even if leaked, it expires.

---

## 14. Summary & Architect Takeaways

*   **Rotation**: Static passwords are a risk. Rotate them every 30-90 days.
*   **Git is Public**: Assume your Git repo will be leaked. If secrets are there, you are dead.
