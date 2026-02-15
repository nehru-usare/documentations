# 02. OAuth2 & OIDC (Keycloak, JWT)

> **Part 6: Security**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Architect)  
> **Status:** Standard Practice

---

## 0. Learning Objectives

| Level | Goal |
|:---|:---|
| **Beginner** | Understand that you shouldn't build your own Login form. |
| **Developer** | Configure Spring Security as a Resource Server. |
| **Architect** | Design a centralized IAM system (Keycloak/Okta). |

---

## 1. Why This Topic Exists

### The Password Problem
In Monoliths, we stored hashes in the `users` table.
In Microservices, we don't want every service to check the password.
**Solution**: Authenticate *once* (Identity Provider), get a **Token** (Passport), and show that Token to everyone else.

---

## 3. Core Concepts (🟢 Beginner Level)

### OAuth2 vs OIDC
*   **OAuth2**: Authorization. "I give this app permission to access my Photos."
*   **OIDC (OpenID Connect)**: Authentication on top of OAuth2. "I am John Doe."

### JWT (JSON Web Token)
A stateless token containing claims.
*   **Header**: Algorithm (`HS256`).
*   **Payload**: User data (`sub: 123`, `role: admin`, `exp: 10:00`).
*   **Signature**: Proof that Keycloak signed it.

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Spring Security Resource Server
The microservice doesn't log you in. It just validates the Token.

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/public/**").permitAll()
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt()); // Validates Signature
        return http.build();
    }
}
```

*Config (`application.yml`)*:
```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://keycloak.mycompany.com/realms/myrealm
```

---

## 5. Internal Mechanics (🔴 Architect Level)

### Grant Types
1.  **Authorization Code Flow with PKCE**: For Frontend Apps (React/Mobile). User is redirected to Login Page.
2.  **Client Credentials Flow**: For Machine-to-Machine. Service A calls Service B. No User involved.

### Token Relay
When Service A calls Service B, it must forward the User's JWT.
*   *Spring Cloud Gateway*: Does this automatically with `TokenRelay` filter.
*   *Feign*: Needs an `RequestInterceptor` to copy the header.

---

## 14. Summary & Architect Takeaways

*   **Stateless**: The beauty of JWT is that Service B validates it *without* calling Keycloak (using Public Key).
*   **Short Lived**: Access tokens should expire in 5-15 mins. Refresh tokens live longer.
