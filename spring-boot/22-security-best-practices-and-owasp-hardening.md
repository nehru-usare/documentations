# Chapter 22: Security Best Practices & OWASP Hardening

## 0. Learning Objectives

- **🟢 Beginner**: Understand the basics of CSRF, CORS, and why you should never use raw HTTP for communication.
- **🟡 Professional**: Master the configuration of Content Security Policy (CSP), XSS protection, and securing sensitive endpoints like `/actuator`.
- **🔴 Architect**: Design a comprehensive security strategy that includes **Zero Trust architecture**, automated dependency scanning (OWASP Dependency Check), and hardening Spring Boot against sophisticated attacks like **Insecure Deserialization** and **SSRF (Server-Side Request Forgery)**.

---

## 1. Why This Topic Exists

### Real-World Business Problem
A single security breach can cost a company millions in fines, lawsuits, and lost reputation. Hackers don't just "guess passwords"; they exploit misconfigured headers, vulnerable libraries, and logical holes in your REST API.

### Technical Limitations Solved
- **Defense in Depth**: Security isn't one "Feature". It's a series of layers. Spring Security provides the tools to implement the most critical layers of the **OWASP Top 10** automatically.
- **Compliance**: For industries like Finance (PCI-DSS) or Healthcare (HIPAA), these hardening steps are a legal requirement.

---

## 2. Big Picture Architecture View

Security hardening sits as a **Global Wrapper** around your entire application.

### Interaction with Other Modules
- **Web MVC**: Hardening via HTTP Headers (CORS, CSP, HSTS).
- **Dependency Management**: Keeping your `pom.xml` / `build.gradle` clean of vulnerable jars.
- **Actuator**: Hardening the "Back door" of your application.

---

## 3. Core Concepts (🟢 Beginner Level)

### Definitions
- **CSRF (Cross-Site Request Forgery)**: An attack where a hacker tricks a user's browser into sending a request (like "Send Money") to a site where they are logged in.
- **CORS (Cross-Origin Resource Sharing)**: A mechanism that tells browsers which other domains (like `my-ui.com`) are allowed to talk to your API (`my-api.com`).

### Simple Explanation
Think of a **Nuclear Power Plant**.
- **AuthN/AuthZ**: The ID badge system.
- **Hardening**: The thick concrete walls, the lead-lined doors, and the background checks for everyone (even the janitors). It's not about stopping people from entering; it's about making sure the whole facility is resilient to explosions and sabotage.

### Minimal Working Example
By default, Spring Boot ENABLES CSRF protection. You must provide a token in your HTML forms.
```html
<input type="hidden" name="_csrf" value="abc.123.xyz" />
```

---

## 4. Developer Deep Dive (🟡 Professional Level)

### Securing the Actuator
The Actuator exposes 50+ sensitive details about your app. **Never leave it public.**
```yaml
management:
  endpoints:
    web:
      exposure:
        include: "health,info,metrics" # ONLY the safe ones
  endpoint:
    health:
      show-details: when_authorized # Hide sensitive stuff
```

### HTTP Hardening Headers
Spring Security adds these by default, but you should verify them:
- **X-Content-Type-Options: nosniff**: Prevents browsers from guessing file types (Protects against XSS).
- **X-Frame-Options: DENY**: Prevents Clickjacking (your app cannot be put inside an `<iframe>`).
- **Strict-Transport-Security (HSTS)**: Tells the browser "Only ever talk to me over HTTPS."

---

## 5. Internal Mechanics (🔴 Advanced Level)

### Zero Trust Architecture
Don't trust anything *inside* the firewall.
- **Philosophy**: Every microservice must verify the JWT of the caller, even if the call came from another service on the same network.
- **mTLS**: Designing your microservices to communicate using Mutual TLS, where both the server AND the client must provide a certificate.

### Protection against SSRF
Server-Side Request Forgery is when a hacker tricks your server into making a call to its own internal systems (like `http://localhost:8080/admin/delete`).
- **Defense**: Never allow users to provide full URLs for your server to fetch. Use a strict **Allow-list** of domains.

---

## 6. Under the Hood

### Insecure Deserialization
Java's native serialization is notoriously unsafe.
- **Spring Fix**: Spring Boot's `Jackson` configuration is hardened by default to prevent "Gadget Chain" attacks. It disallows the deserialization of arbitrary types unless they are explicitly on a whitelist.

---

## 7. Real-World Use Cases

- **Banking UI**: Implementing a strict **CORS** policy so that only `https://bank.com` can call the `https://api.bank.com` endpoints.
- **Public API**: Using **Rate Limiting** (via Bucket4j or Redis) to prevent an attacker from brute-forcing your login endpoint.

---

## 8. Production & Performance Considerations

- **Dependency Scanning**: Run `mvn owasp-dependency-check:check` in your CI/CD pipeline. It will FAIL the build if you use a library with a known security vulnerability (CVE).
- **Log Masking**: Custom library or log4j filters to ensure that credit card numbers or passwords never appear in the server logs.

---

## 9. Architect-Level Best Practices

- **Security as Code**: Define your security rules in Java, not as manual firewall settings.
- **Rotate Secrets Often**: Use **HashiCorp Vault** or **AWS Secrets Manager** to rotate DB passwords and API keys every 30-90 days automatically.
- **Minimum Dependency Footprint**: If you only need a small utility from a 50MB library, find a smaller library or write it yourself. Less code = Smaller attack surface.

---

## 10. Common Mistakes & Anti-Patterns

- **Disabling CSRF for REST APIs without a reason**: If you use cookies for authentication, you NEED CSRF. Only disable it if you use 100% stateless JWT/Token auth in headers.
- **Allowing `Access-Control-Allow-Origin: *`**: This is a security disaster. It allows ANY website to steal your data if a user is logged in. **Always use a specific domain.**

---

## 11. Debugging & Troubleshooting

### How to Diagnose Issues
- **`https://securityheaders.com`**: Put your public URL here. It will grade your app (A+ to F) based on which security headers are present.
- **Burp Suite / Zap**: Professional penetration testing tools. Run them against your app to find logical holes.

---

## 12. Comparisons

### CSRF vs. XSS
| Feature | CSRF | XSS (Cross-Site Scripting) |
| :--- | :--- | :--- |
| **Goal** | Impersonate a request | Inject malicious JS |
| **Source** | External site | User-generated content / URL |
| **Defense** | Token-based check | Sanitization / CSP Headers |
| **Scope** | State-changing actions | Entire page context |

---

## 13. Interview Questions

### 🟢 Basic
1. What is CORS and why do we need it?
2. Why is HTTPS mandatory for modern apps?

### 🟡 Intermediate
1. How do you secure Actuator endpoints in Spring Boot?
2. What are the common HTTP security headers Spring Security provides?

### 🔴 Advanced
1. Explain SSRF and how to prevent it in a Spring Boot application.
2. What is Zero Trust and how do you implement it for internal microservice communication?

### 🔥 Tricky
1. If you disable CSRF and your app uses cookies for authentication, is it vulnerable? (Yes, highly. Cookies are sent automatically by the browser, allowing an attacker to mimic requests).

---

## 14. Scenario-Based Interview Questions

1. **Architecture**: You are building a public "Image Resizer" API where users provide a URL to an image. How do you prevent a hacker from using your server to scan your internal network? (Implement a `UrlValidator` that checks the IP address of the target URL and blocks any Private IP ranges like `10.x.x.x` or `192.168.x.x`).
2. **Security Incident**: A "Dependency Scanning" report shows that your core logging library has a critical RCE (Remote Code Execution) vulnerability (like Log4Shell). How do you patch this across 500 microservices quickly? (Update the "Parent POM" or use an "Agent" like Snyk/Contrast to block the exploit path until code can be redeployed).

---

## 15. Summary & Key Takeaways

- **Core Insight**: Security is a **Process**, not a Product. 
- **Architect Mindset**: Don't ask "Is it secure?". Ask "What happens when it's breached?".
- **Production Reminder**: Run automated vulnerability scans weekly. New zero-day exploits are discovered every day.

---

**Author:** Nehru Usare | Senior Spring Architect  
**Part 4: Chapter 22**
