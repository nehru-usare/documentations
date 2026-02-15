# Chapter 20: JWT, OAuth2, and OIDC Integration

## 0. Learning Objectives

- **🟢 Beginner**: Understand the basic concepts of a Token and the difference between OAuth2 and OpenID Connect (OIDC).
- **🟡 Professional**: Master the configuration of a "Resource Server" using JWT, and learn how to use the `spring-boot-starter-oauth2-client` for social login (Google/GitHub).
- **🔴 Architect**: Deep dive into the `JwtDecoder` internals, understand the "Token Exchange" pattern, and design a secure CI/CD and Microservice identity flow using **Shared Secrets vs. PKI (Public Key Infrastructure)**.

---

## 1. Why This Topic Exists

### Real-World Business Problem
In a world of microservices and mobile apps, **Session-based** security (Stateful) is a bottleneck. If you have 100 services, you don't want each one asking a central "Session Database" if a user is logged in for every single request. You need a "Passport" that the user carries with them, which any service can verify without checking a central database.

### Technical Limitations Solved
- **Scalability**: JSON Web Tokens (JWT) are stateless. The service can verify the token using a mathematical signature (Public Key) entirely offline.
- **Interoperability**: OAuth2 and OIDC are industry standards. They allow your app to talk to Google, Azure AD, Okta, or Keycloak without writing custom code for each provider.

---

## 2. Big Picture Architecture View

Token-based security moves the "Identity Responsibility" from the **Application Server** to an **Identity Provider (IdP)**.

### Interaction with Other Modules
- **Spring Security**: Provides the `BearerTokenAuthenticationFilter`.
- **API Gateway**: Usually the place where tokens are first validated or "swapped" for internal tokens.
- **Microservices**: Act as "Resource Servers" that trust tokens issued by the IdP.

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **JWT (JSON Web Token)**: A compact, URL-safe way of representing claims between two parties.
- **OAuth2**: A framework for **Authorization** (Allowing an app to access your data).
- **OIDC (OpenID Connect)**: A layer on top of OAuth2 for **Authentication** (Verifying who you are).

### Simple Explanation
Think of a **Concert**.
- **OIDC**: You show your ID at the gate to prove you are "John Smith" (Identity).
- **OAuth2**: The ticket checker gives you a **Wristband**. The red wristband lets you into the "VIP Lounge" but not the "Stage Area". You don't need to show your ID again; just the wristband.
- **JWT**: The wristband has a physical "Hologram" (Signature) that proves it was issued by the concert organizers and not made by a counterfeiter.

### Minimal Working Example
Adding a dependency for a Resource Server:
```yaml
# application.yml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://my-auth-server.com/realms/master
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Anatomy of a JWT
1. **Header**: Algorithm and Type.
2. **Payload (Claims)**: `sub` (user), `exp` (expiry), `roles`.
3. **Signature**: Cryptographic proof that the header and payload haven't changed.

### OAuth2 Client Flow (Login)
1. User clicks "Login with Google".
2. App redirects to Google.
3. User logs in at Google.
4. Google sends a **Code** back to the App.
5. App swaps the Code for an **Access Token** and an **ID Token**.

### Resource Server Flow (API)
The client sends the token in the header:
`Authorization: Bearer <token_here>`
Spring Security automatically decodes it, verifies the signature, and populates the `SecurityContext`.

---

## 5. Internal Mechanics (🔴 Advanced Level)

### JwtDecoder Mechanism
The `NimbusJwtDecoder` is the default.
1. **JWKS (JSON Web Key Set)**: On startup, Spring calls the IdP's `well-known` endpoint to fetch the **Public Keys**.
2. **Offline Validation**: For every request, Spring uses those public keys to verify the token signature locally in the JVM. No network call needed!
3. **Caching**: The public keys are cached to ensure low latency.

### Token Scopes vs. Authorities
- **Scopes**: What the *App* is allowed to do (e.g., `read:profile`).
- **Authorities**: What the *User* is allowed to do (e.g., `ROLE_ADMIN`).
Spring Security's `JwtAuthenticationConverter` can be customized to map JWT scopes/claims into Spring Authorities.

---

## 6. Under the Hood

### The Role of `spring-security-oauth2-jose`
This library handles the "JOSE" (Javascript Object Signing and Encryption) standards. It manages JWS (Signed), JWE (Encrypted), and JWK (Keys).

---

## 7. Real-World Use Cases

- **Mobile App**: A React Native app that logs in once via OIDC and then uses a JWT to call 20 different microservices.
- **B2B Integration**: Allowing a partner company to call your API using their own "Client Credentials" (Service-to-Service).

---

## 8. Production & Performance Considerations

- **Token Expiry**: Keep JWT expiry short (e.g., 15 minutes). Use **Refresh Tokens** to get new ones. If a JWT lasts for 1 year and is stolen, the attacker is in for 1 year.
- **Blacklisting**: You cannot "Logout" a JWT because it's stateless. 
  - **Solution**: If a token is stolen, you must store its ID in a **Redis Blacklist** and check it on every request (reintroducing state) or wait for it to expire.

---

## 9. Architect-Level Best Practices

- **Use Asymmetric Keys (RS256)**: Never use Symmetric (HS256) keys for distributed systems. RS256 allows the IdP to keep the Private Key secret while everyone else uses the Public Key to verify.
- **Validate "aud" (Audience)**: Ensure the JWT was meant for YOUR service. If Service A's JWT is stolen, it shouldn't be valid for Service B.
- **Centralized Logout**: Implement a "BFF" (Backend For Frontend) pattern where tokens are stored in secure HttpOnly cookies on the server, not in the browser's LocalStorage (vulnerable to XSS).

---

## 10. Common Mistakes & Anti-Patterns

- **Sensitive Data in Payload**: Putting a user's password or home address inside a JWT. **JWT is NOT encrypted; it is only SIGNED.** Anyone can decode it at `jwt.io`.
- **Ignoring "iat" (Issued At)**: Not checking if a token was issued *before* a user changed their password or was banned.

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **`jwt.io`**: The first step. Copy the token and see if the claims are what you expect.
- **`logging.level.org.springframework.security.oauth2=DEBUG`**: Shows the key fetch phase and validation errors.
- **401 Unauthorized**: Often means the `issuer-uri` in your config doesn't exactly match the `iss` claim in the token. (Case sensitive!).

---

## 12. Comparisons

### Statefull (Sessions) vs. Stateless (JWT)
| Feature | Sessions | JWT |
| :--- | :--- | :--- |
| **Storage** | Server RAM/DB | Client Handsets |
| **Revocation** | Instant | Hard (needs blacklist) |
| **Scalability** | Lower (needs sticky sessions) | Infinite |
| **Security** | Opaque (more secure) | Transparent (contents visible) |
| **Recommendation** | Internal monoliths | **Microservices / Public APIs** |

---

## 13. Interview Questions

### 🟢 Basic
1. What is the difference between OAuth2 and OIDC?
2. What are the three parts of a JWT?

### 🟡 Intermediate
1. How does a Resource Server verify a token without calling the Auth server?
2. What is a "Refresh Token"?

### 🔴 Advanced
1. Describe the OIDC "Authorization Code" flow with PKCE (Proof Key for Code Exchange).
2. What is a JWKS endpoint and why is it used?

### 🔥 Tricky
1. If your server clock is 5 minutes behind the Auth server, will the JWT validation fail? (Yes, it will look like the token "Is not yet valid" or already "Expired").

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You are building a Microservice system with 5,000 requests per second. Every service needs the user's "Full Name" and "Employee ID". Do you put this in the JWT or have each service fetch it from the DB? (Put it in the JWT as non-sensitive claims to avoid 5,000 DB calls per second).
2. **Security**: A hacker stole a developer's Laptop which had an active long-lived JWT. How do you revoke that specific developer's access immediately? (Rotate the signing keys—nuclear option—or add the specific token `jti` to a Redis blacklist).

---

## 15. Summary & Key Takeaways

- **Core Insight**: JWT is a **Portable Proof of Identity**.
- **Architect Mindset**: Standardize and outsource. Don't build your own Auth Server; use Keycloak, Okta, or Cognito.
- **Production Reminder**: Keep your tokens small. A 10KB JWT in a request header will slow down every single HTTP call.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 4: Chapter 20**
