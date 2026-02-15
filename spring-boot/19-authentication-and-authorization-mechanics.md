# Chapter 19: Authentication & Authorization Mechanics

## 0. Learning Objectives

- **🟢 Beginner**: Understand the difference between "Who are you?" (Authentication) and "What can you do?" (Authorization).
- **🟡 Professional**: Master the `AuthenticationProvider` interface, `GrantedAuthority`, and custom role-based access control (RBAC).
- **🔴 Architect**: Deep dive into the `AuthenticationManager` delegation, understand the `SecurityContext` thread propogation, and design a dynamic authorization system using **Voters** and custom **Expression Handlers**.

---

## 1. Why This Topic Exists

### Real-World Business Problem
A banking app has three types of people: **Customers**, **Bank Tellers**, and **Admins**.
- A **Customer** can see their own balance but shouldn't see anyone else's.
- A **Teller** can see balances for everyone but can't delete accounts.
- An **Admin** can do everything.
Without a robust AuthN/AuthZ system, you would have thousands of `if` statements scattered throughout your code, leading to massive security leaks.

### Technical Limitations Solved
- **Abstraction**: It allows your code to say "Only Admins can enter here" without having to manually check databases or LDAP servers in every method.
- **Unified Identity**: Once a user is authenticated, their identity is available globally across all layers (Web, Service, Data).

---

## 2. Big Picture Architecture View

Authentication happens **First**, usually in a Filter. Authorization happens **Last**, usually right before the controller or service method is actually called.

### Interaction with Other Modules
- **Data Layer**: Usually where the `User` and `Role` records are stored.
- **Microservices**: Authentication details are often passed via JWT tokens in the headers.

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **Authentication (AuthN)**: The process of verifying a user's identity (e.g., Username/Password).
- **Authorization (AuthZ)**: The process of verifying if the authenticated user has permission to access a specific resource.

### Simple Explanation
Think of a **Hotel**.
- **AuthN**: You go to the front desk and show your ID (Passport). They verify it's you and give you a **Key Card**.
- **AuthZ**: You take the key card to Room 201. The door lock checks the card. If it's for 201, it opens. If it's for 202, it stays locked. The key card is your "Authentication Token"; the door lock is the "Authorization Check".

### Minimal Working Example
```java
// AuthZ check in SecurityFilterChain
http.authorizeHttpRequests(auth -> auth
    .requestMatchers("/admin/**").hasRole("ADMIN")
    .anyRequest().authenticated()
);
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### The `Authentication` Object
Found in the `SecurityContext`. It contains:
- **Principal**: The user (usually a `UserDetails` object).
- **Credentials**: Password (usually cleared after authentication for security).
- **Authorities**: A list of `GrantedAuthority` (e.g., `ROLE_ADMIN`).

### UserDetailsService
The most common way to load a user.
```java
@Service
public class MyUserDetailsService implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) {
        UserEntity user = userRepo.findByUsername(username);
        return new User(user.getUsername(), user.getPassword(), getAuthorities(user));
    }
}
```

---

## 5. Internal Mechanics (🔴 Advanced Level)

### AuthenticationManager & ProviderManager
The `AuthenticationManager` is the brain. It usually uses a `ProviderManager` implementation:
1. **Chain of Command**: It has a list of `AuthenticationProvider`s.
2. **Delegation**: It asks the first provider: "Can you handle this `UsernamePasswordAuthenticationToken`?"
3. **Result**: If a provider says "Yes" and the password is correct, it returns a fully-populated `Authentication` object.

### AccessDecisionManager (Voters)
Authorization is a "Vote".
- **`RoleVoter`**: Votes YES if the user has the required ROLE.
- **`AuthenticatedVoter`**: Votes YES if the user is logged in.
- **`AffirmativeBased`**: One "YES" vote is enough to grant access (Spring's default).
- **`UnanimousBased`**: Everyone must vote YES.

---

## 6. Under the Hood

### Password Hashing (BCrypt)
Spring Security mandates the use of a `PasswordEncoder`. 
- **Internal**: `BCryptPasswordEncoder` uses a random **Salt** for every password. This prevents "Rainbow Table" attacks where hackers use pre-calculated hashes to find passwords.

---

## 7. Real-World Use Cases

- **SSO (Single Sign-On)**: A custom `AuthenticationProvider` that checks a third-party SAML or CAS ticket instead of a database.
- **Dynamic Permissions**: A system where permissions are stored in the DB as strings (e.g., `user:read`, `user:write`) and mapped to `GrantedAuthority`.

---

## 8. Production & Performance Considerations

- **User Loading Bottleneck**: Calling the database to load the user on *every single request* is slow.
  - **Architect Tip**: Use **Caching (Redis)** for your `UserDetailsService` results to keep authentication latency under 5ms.
- **Session Bloat**: Storing a 50KB User object in the Servlet Session will kill your server's RAM as users grow. Store only the `ID` and `Roles`.

---

## 9. Architect-Level Best Practices

- **Never use "ROLE_" prefix in code**: If you use `hasRole("ADMIN")`, Spring automatically adds the `ROLE_` prefix. If you use `hasAuthority("ADMIN")`, it doesn't. Be consistent.
- **User Immutable Principals**: Ensure your `UserDetails` implementation is an immutable record or POJO to prevent accidental side-effects during a request.
- **Separation of Concerns**: Keep your `UserEntity` (DB model) separate from your `UserDetails` (Security model) to avoid leaking internal DB fields to the security layer.

---

## 10. Common Mistakes & Anti-Patterns

- **Hardcoding Passwords**: Storing your `admin` password in `application.yml`. Use environment variables.
- **Logic in AccessDecisionVoters**: Don't put heavy DB calls in a voter. Voters run for every request to an endpoint. Keep them fast and purely logic-based.

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **`AuthenticationEventPublisher`**: Listen to `AuthenticationSuccessEvent` or `AbstractAuthenticationFailureEvent` to log exactly who is failing to log in and why (bad password vs. locked account).
- **SecurityContext Insight**: In a debugger, inspect `SecurityContextHolder.getContext().getAuthentication()`. If it's null, your filter wasn't triggered.

---

## 12. Comparisons

### hasRole vs. hasAuthority
| Feature | hasRole("XYZ") | hasAuthority("XYZ") |
| :--- | :--- | :--- |
| **Prefix** | Adds "ROLE_" automatically | Matches exact string |
| **Philosophy** | "Who are you?" (Persona) | "What can you do?" (Permission) |
| **Recommendation** | Standard user categories | **Granular permission systems** |

---

## 13. Interview Questions

### 🟢 Basic
1. What is the difference between `@Secured` and `@PreAuthorize`?
2. What is a "Principal" in Spring Security?

### 🟡 Intermediate
1. How does `BCrypt` password hashing work?
2. What is the role of the `AuthenticationProvider`?

### 🔴 Advanced
1. Explain the delegation model inside the `ProviderManager`.
2. How do you implement a custom `AccessDecisionVoter`?

### 🔥 Tricky
1. If you successfully log in, but then change the user's roles in the database, does the current user's role update immediately? (No, not until their session is refreshed or their token is re-validated).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You need to implement "Multi-Factor Authentication" (MFA) in a Spring Boot app. How do you integrate the second step into the `AuthenticationManager`? (Create a custom `AuthenticationFilter` that checks for a session flag or a specialized MFA token after the initial log in).
2. **Security**: Your app allows users to log in with either an **Email** or a **Phone Number**. How do you handle this in `UserDetailsService`? (The `loadUserByUsername` method should use a repository method that checks both fields: `findByEmailOrPhone`).

---

## 15. Summary & Key Takeaways

- **Core Insight**: Authentication is **Identity**; Authorization is **Policy**.
- **Architect Mindset**: Deny by default. Every "Open" endpoint is a liability.
- **Production Reminder**: Use **BCrypt** or **Argon2** for passwords. Never, ever store plain text or MD5 hashes.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 4: Chapter 19**
