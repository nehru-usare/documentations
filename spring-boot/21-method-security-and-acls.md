# Chapter 21: Method Security and ACLs

## 0. Learning Objectives

- **🟢 Beginner**: Understand the use of `@PreAuthorize` and `@PostAuthorize` for basic role checks.
- **🟡 Professional**: Master the use of SpEL (Spring Expression Language) in security annotations, and understand the difference between Method Security and URL-based security.
- **🔴 Architect**: Deep dive into the `MethodSecurityInterceptor` internals, understand the performance cost of **Spring Security ACLs**, and design a custom `PermissionEvaluator` for domain-specific authorization.

---

## 1. Why This Topic Exists

### Real-World Business Problem
URL-based security is limited. It only protects the "Front Door" of an application. But what if a user accesses a service method via a **Background Job**, an **RMI call**, or another service? You need security that is attached directly to the business method, not just the web path.

### Technical Limitations Solved
- **In-Depth Defense**: If a developer accidentally adds a public endpoint that calls an unprotected service, the data is at risk. Method security ensures that the business logic itself is guarded regardless of how it's called.
- **Entity-Level Permission**: "Users can edit their own profiles." This rule depends on the specific *ID* of the object, which URL security cannot easily check.

---

## 2. Big Picture Architecture View

Method Security is an **AOP-based** feature. It wraps your service beans in a security proxy.

### Interaction with Other Modules
- **Spring Context**: Security proxies are created during the Bean Lifecycle (Initialization phase).
- **Core Container**: Security metadata is stored in the bean definitions.

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **Pre-Authorization**: Checks permissions *Before* the method runs.
- **Post-Authorization**: Checks permissions *After* the method runs (useful for checking the result of the method).

### Simple Explanation
Think of a **Safety Deposit Box**.
- **URL Security**: Check your ID at the front desk of the bank. (Authentication).
- **Pre-Authorization**: The guard checks if you have your key before letting you into the vault room.
- **Post-Authorization**: The guard checks if the box you are carrying out of the room actually belongs to you before letting you leave.

### Minimal Working Example
```java
@EnableMethodSecurity
@Configuration
public class SecurityConfig { }

@Service
public class DocumentService {
    @PreAuthorize("hasRole('ADMIN')")
    public void deleteEverything() { ... }
}
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### SpEL (Spring Expression Language)
You can use powerful logic in annotations:
```java
@PreAuthorize("#contact.name == authentication.name")
public void doSomething(Contact contact);
```
- **`@PostAuthorize`**: Often used to verify the user "Owns" the result:
  `@PostAuthorize("returnObject.owner == authentication.name")`

### Security Annotations Comparison
- **`@Secured`**: Legacy, only supports a list of roles (no SpEL).
- **`@RolesAllowed`**: Java standard (JSR-250), basic role checks.
- **`@PreAuthorize`**: **Modern standard**, full SpEL support.

---

## 5. Internal Mechanics (🔴 Advanced Level)

### The Proxy Flow
1. **Intercept**: When `service.delete()` is called, the **AOP Proxy** catches it.
2. **Context Check**: It retrieves the `Authentication` from the `SecurityContextHolder`.
3. **Expression Evaluation**: It passes the annotation string to a `DefaultMethodSecurityExpressionHandler`.
4. **Outcome**: If the expression returns `false`, an `AccessDeniedException` is thrown *before* your method code ever starts.

### Spring Security ACL (Access Control List)
When you need to know "Who has access to row #541?".
- **Infrastructure**: Spring Security ACL uses 4 database tables (`ACL_CLASS`, `ACL_OBJECT_IDENTITY`, `ACL_SID`, `ACL_ENTRY`) to store every single permission for every single row.
- **Cost**: It is extremely powerful but can add a 100ms-200ms overhead to every database call due to the complex joins required to check permissions.

---

## 6. Under the Hood

### Custom PermissionEvaluator
If ACL is too heavy, architects build a custom `PermissionEvaluator`.
```java
public class MyEvaluator implements PermissionEvaluator {
    @Override
    public boolean hasPermission(Authentication auth, Object targetId, String perm) {
        // Run custom DB or logic check
    }
}
```
**Usage**: `@PreAuthorize("hasPermission(#id, 'Document', 'READ')")`

---

## 7. Real-World Use Cases

- **Healthcare APIs**: Ensuring a Doctor can only see patients assigned to their clinic.
- **Financial Systems**: Verifying that a transfer amount doesn't exceed the user's "Daily Limit" claim in their token.

---

## 8. Production & Performance Considerations

- **Proxy Overhead**: Having `@PreAuthorize` on every single method in a massive service layer can slightly impact performance due to the reflection and expression parsing.
- **Caching expressions**: Spring automatically caches the parsed SpEL expressions, but the *evaluation* still happens on every call.

---

## 9. Architect-Level Best Practices

- **Web Security First**: Use URL security for the "Bulk" of your work (blocking /admin). Use Method security for "Granular" checks (blocking row-level access).
- **Don't Over-automate**: Avoid deeply nested SpEL logic. If your `@PreAuthorize` is 5 lines long, move that logic into a Service method and call it: `@PreAuthorize("@mySecurityService.canAccess(#id)")`.
- **Prefer @PreAuthorize**: It's more intuitive to fail early. Use `@PostAuthorize` sparingly as it still executes the business logic (which might be expensive) before failing.

---

## 10. Common Mistakes & Anti-Patterns

- **Missing `@EnableMethodSecurity`**: Without this annotation on a config class, all method security annotations are silently ignored!
- **Self-Invocation**: Just like transactions, calling a method from another method in the same class skips the proxy and the security check.

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **Breakpoint**: Put a breakpoint in `AffirmativeBased.decide()`. This is where the "YES/NO" vote for authorization happens.
- **Log Exceptions**: `AccessDeniedException` is often caught and converted to 403. Check your **ExceptionHandling** logs to see the actual SpEL evaluation failure.

---

## 12. Comparisons

### URL Security vs. Method Security
| Feature | URL Security | Method Security |
| :--- | :--- | :--- |
| **Granularity** | Coarse (paths) | Fine (methods/rows) |
| **Logic** | Static patterns | Dynamic (SpEL/Params) |
| **Implementation** | Filters | AOP Proxies |
| **Recommendation** | **Public vs Private** | **Domain Permission Rules** |

---

## 13. Interview Questions

### 🟢 Basic
1. Difference between `@PreAuthorize` and `@PostAuthorize`?
2. How do you enable method security in Spring Boot?

### 🟡 Intermediate
1. Can you use method security on non-web-controller services? (Yes, it's AOP-based).
2. What is SpEL?

### 🔴 Advanced
1. Why doesn't method security work on private methods?
2. What is a `PermissionEvaluator` and how do you use it?

### 🔥 Tricky
1. If you have both URL security and Method security on the same request, who wins? (They BOTH run. If either one denies access, the user is blocked. It is an "AND" relationship).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You have a requirement where "Managers can see all employees' salaries, but regular employees can only see their own." How do you implement this? (Use `@PostAuthorize` to check if the returned User object's ID matches the logged-in user OR if the logged-in user has `ROLE_MANAGER`).
2. **Performance**: Your ACL implementation has slowed down your app significantly. You have 10 million entries in the `ACL_ENTRY` table. What is the architecture fix? (Move to a **Policy-Based** approach where permissions are derived from attributes/groups rather than individual row mappings, or use a cached bitmask).

---

## 15. Summary & Key Takeaways

- **Core Insight**: Method Security is **Logic-Aware** authorization.
- **Architect Mindset**: Don't put security in every file. Centralize your permission logic in a SecurityService and call it from the annotations.
- **Production Reminder**: Be careful with `@PostAuthorize`. It doesn't prevent a "Write" operation; it only hides the "Read" result. Use `@Pre` for anything that modifies data.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 4: Chapter 21**
