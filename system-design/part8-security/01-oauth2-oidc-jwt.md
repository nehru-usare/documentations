# OAuth2 vs OIDC vs JWT

> **Part 8: Security**  
> **Difficulty:** ⭐⭐⭐⭐ (Advanced)  
> **Status:** The Keys to the Kingdom

---

## 0. Learning Objectives
*   **Beginner**: The difference between Logging In (AuthN) and Permission (AuthZ).
*   **Developer**: Inspecting a JWT Header/Payload/Signature.
*   **Architect**: Designing a centralized Identity Provider (IdP) for all microservices.

---

## 1. Problem Context
**Why does this exist?**
*   **Old Way**: Store `username/password` in every database.
*   **Problem**: User changes password. Must update 10 services. Security nightmare.
*   **New Way**: **SSO (Single Sign-On)**.
    *   User logs in *once* to Google/Auth0.
    *   Gets a Token (Badge).
    *   Shows Badge to Service A, B, C.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. AuthN vs AuthZ
*   **Authentication (AuthN)**: Who are you? (Identity). *Protocol: OIDC*.
*   **Authorization (AuthZ)**: What can you do? (Access). *Protocol: OAuth2*.

### 2. JWT (JSON Web Token)
*   The Badge.
*   Stateless. Contains data (`sub: 123`, `role: admin`).
*   Signed by the IdP.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### OAuth2 Flows (Grants)
1.  **Authorization Code Flow** (Standard):
    *   User -> Browser -> IdP (Login) -> Code -> Backend -> IdP -> Token.
    *   *Secure*: Token never hits the Browser URL.
2.  **Client Credentials Flow** (M2M):
    *   Service A -> IdP (ID+Secret) -> Token.
    *   No Human involved.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Anatomy of a JWT
*   `Header`: Algorithm (`HS256` or `RS256`).
*   `Payload`: Claims (`exp`, `iss`, `custom_field`).
*   `Signature`: `Hash(Header + Payload + Secret)`.
*   **Verification**: Service B doesn't need to call IdP. It just verifies the signature using the Public Key.

### 2. OIDC (OpenID Connect)
*   OAuth2 is for *Access* (Getting into the hotel room).
*   OIDC adds an *ID Token* (The Passport showed at reception).
*   Standardizes `UserInfo` endpoint.

---

## 5. Trade-Off Analysis

| Feature | Session ID (Cookies) | JWT (Tokens) |
| :--- | :--- | :--- |
| **State** | Stateful (DB lookup) | Stateless (Self-contained) |
| **Revocation** | Easy (Delete user session) | Hard (Wait for expiry or Blacklist) |
| **Size** | Tiny (32 bytes) | Large (1KB+) |
| **Cross-Domain** | Hard (CORS/Cookies) | Easy (Auth Header) |

---

## 6. Scaling Considerations

### The Revocation Problem
*   JWTs are valid until they expire (e.g., 1 hour).
*   If User is banned, they still have access for 59 minutes.
*   *Fix*: **Short Expiry** (5 mins) + **Refresh Tokens**.

---

## 7. Failure Scenarios & Recovery

### 1. Key Rotation
*   IdP rotates the Signing Key.
*   All services fail to verify tokens.
*   **Fix**: Services must cache the JWKS (JSON Web Key Set) and refresh it dynamically on verification failure.

---

## 8. Security Considerations

### 1. The "None" Algorithm
*   Attacker sends JWT with `alg: none`.
*   Naive libraries accept it without checking signature.
*   **Fix**: Hardcode expected algorithms (`RS256`).

### 2. Storing Tokens
*   **Local Storage**: Vulnerable to XSS.
*   **HttpOnly Cookie**: Safe from XSS, vulnerable to CSRF.
*   *Verdict*: HttpOnly Cookie + CSRF Token is best for Web.

---

## 9. Performance Considerations

*   **Symmetric (HS256)**: Fast. Shared Secret. (Bad for Microservices).
*   **Asymmetric (RS256)**: Slower. Public/Private Key. (Good for Microservices).

---

## 10. Real Production Lessons

### Auth0 Outage
*   **Event**: Global Outage.
*   **Impact**: Nobody could log in.
*   **Lesson**: Access Tokens should be long-lived enough to survive short IdP outages, or implement a "Break Glass" local auth for Admins.

---

## 11. Interview Questions

### Basic
1.  Describe structure of JWT.
2.  Difference between 401 (Unauth) and 403 (Forbidden).
3.  What is a Refresh Token?
4.  Why not store password in plain text? (Hashing/Salting).
5.  What is standard header? (`Authorization: Bearer <token>`).

### Intermediate
1.  Explain Authorization Code Flow.
2.  AuthN vs AuthZ.
3.  Symmetric vs Asymmetric signing.
4.  Where should you store a JWT in the browser?
5.  What happens if you change a byte in the JWT payload?

### Advanced
1.  Design a "Logout" system for JWTs. (Bloom Filter Blacklist).
2.  Analyze the security impact of "Implicit Flow" (Deprecated).
3.  How does generic OAuth2 differ from OIDC?
4.  Implement "Token Exchange" pattern for microservices chain.
5.  Critique usage of Paseto tokens vs JWT.

### Architect-Level
1.  "We have 100 internal apps. Architect a Centralized IAM." (Keycloak/Okta).
2.  Design a Multi-Tenant Auth system where Tenant A cannot forge tokens for Tenant B. (Issuer Validation).
3.  Evaluate the risks of using JWT for Session Management.

---

## 12. Scenario-Based System Design Problems

### 1. Design Google Login integration
*   **Flow**: Web App redirects to Google. Google redirects back with Code. Backend exchanges Code for Token.
*   **Key**: Don't trust the client. Verify ID Token on backend.

### 2. Design Microservice-to-Microservice Auth
*   **Choice**: **mTLS** (Infrastructure level) OR **Client Credentials Flow** (App level).
*   **Verdict**: mTLS is modern standard (Service Mesh).

---

## 13. Summary & Architect Takeaways

1.  **Don't Roll Your Own Crypto**: Use OAuth2 libraries. Use Auth0/Keycloak.
2.  **Stateless is a double-edged sword**: Easy scale, hard revocation.
3.  **Rotation**: Keys rotate. Certificates expire. Automate it.
