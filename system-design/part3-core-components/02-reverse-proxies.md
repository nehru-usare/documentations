# Reverse Proxies

> **Part 3: Core Components**  
> **Difficulty:** ⭐⭐ (Basic)  
> **Status:** The Gatekeeper

---

## 0. Learning Objectives
*   **Beginner**: Distinguish Forward Proxy (VPN) vs Reverse Proxy (Nginx).
*   **Developer**: Use a Reverse Proxy to serve static files and compress API responses.
*   **Architect**: Design a DMZ using Reverse Proxies to hide internal topology.

---

## 1. Problem Context
**Why does this exist?**
You don't want to expose your Java/Node.js application server directly to the internet.
*   It's not great at SSL.
*   It's not great at serving `image.jpg`.
*   It exposes your internal IP structure.
You put a "Face" in front of it. That face is the **Reverse Proxy**.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Forward Proxy (The Client's Guard)
*   Sits *before* the Client.
*   Hides the Client identity.
*   **Example**: VPN, Corporate Firewall (blocks Facebook).

### 2. Reverse Proxy (The Server's Guard)
*   Sits *before* the Server.
*   Hides the Server identity.
*   **Example**: Nginx, Cloudflare.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Features
1.  **Security**: Hide internal IPs. Block malicious IPs.
2.  **SSL Termination**: Handle certs in one place.
3.  **Caching**: Cache static content (`style.css`).
4.  **Compression**: Gzip/Brotli responses.

```nginx
server {
    listen 80;
    location / {
        proxy_pass http://localhost:8080; # Java App
    }
    location /static {
        root /var/www/html; # Serve files directly
    }
}
```

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Buffering
*   Slow Clients (Mobile 3G) keep the connection open for seconds.
*   Reverse Proxy buffers the response from the fast Backend and spoon-feeds it to the slow Client.
*   *Benefit*: Frees up Backend threads immediately.

### 2. Request Queuing
*   If Backend is 100% busy, Reverse Proxy queues the request (up to a limit) instead of dropping it.

---

## 5. Trade-Off Analysis

| Strategy | Pros | Cons |
| :--- | :--- | :--- |
| **Direct Exposure** | Low Latency. Simple. | Security Risk. App Server handles SSL/Compression (inefficient). |
| **Reverse Proxy** | Security. Caching. Performance features. | Added Hop (Latency). Config complexity. |

---

## 6. Scaling Considerations

### The Sidecar Pattern
In Kubernetes/Microservices, the Reverse Proxy moved closer to the app.
*   **Old**: One giant Nginx for 100 services.
*   **New**: One tiny Envoy proxy *per pod*. (Service Mesh).

---

## 7. Failure Scenarios & Recovery

### 1. The "502 Bad Gateway"
*   **Meaning**: The Reverse Proxy is fine, but the Backend refused connection.
*   **Cause**: Backend crashed, or Firewall blocks Proxy -> Backend.

### 2. The "504 Gateway Timeout"
*   **Meaning**: The Backend accepted connection but took too long to reply.
*   **Cause**: Slow DB query.

---

## 8. Security Considerations

### 1. Information Leaking
*   By default, Nginx might send a header `Server: nginx/1.14.0`.
*   *Security*: Remove version numbers. `server_tokens off;`.

### 2. IP Masking
*   The Backend sees the Proxy's IP, not the User's IP.
*   *Fix*: Proxy must add `X-Forwarded-For` header. Backend must trust this header.

---

## 9. Performance Considerations

*   **Compression**: CPU intensive.
*   **Optimization**: Compress *once* and cache the compressed version. Or use a separate CDN.

---

## 10. Real Production Lessons

### The "Header Size" Exploit
*   **Scenario**: Attacker sends a 100KB HTTP Header.
*   **Result**: Reverse Proxy buffer overflows or crashes.
*   **Fix**: Limit `client_header_buffer_size`.

---

## 11. Interview Questions

### Basic
1.  Difference between Forward and Reverse Proxy.
2.  Name 2 popular Reverse Proxy softwares.
3.  Why send static files via Nginx instead of Tomcat?
4.  What is `X-Forwarded-For`?
5.  What is a 502 error?

### Intermediate
1.  How does a Reverse Proxy help with slow clients? (Buffering).
2.  Explain SSL Offloading.
3.  Can a Reverse Proxy do Load Balancing? (Yes).
4.  How do you handle WebSocket timeouts in Nginx?
5.  What is the benefit of Gzip compression?

### Advanced
1.  Design a cache purging strategy for a Reverse Proxy.
2.  Analyze the overhead of Envoy in a Service Mesh.
3.  How do you secure the link between Proxy and Backend? (mTLS).
4.  Explain "Anycast" mechanism for global proxies (Cloudflare).
5.  How to prevent "Bypassing the Proxy" attacks? (Firewall rules).

### Architect-Level
1.  Evaluate Nginx vs Apache Traffic Server for a high-traffic media site.
2.  Design a Zero-Trust architecture where the Proxy authenticates every request via OIDC.
3.  Critique the "Single Proxy" vs "Sidecar Proxy" architecture.

---

## 12. Scenario-Based System Design Problems

### 1. Design a Secure API Exposure
*   **Requirement**: Block SQL Injection.
*   **Choice**: Reverse Proxy with ModSecurity (WAF).

### 2. Design a Static Site Hosting
*   **Requirement**: Fast.
*   **Choice**: Nginx serving files from disk + In-memory caching + Gzip.

---

## 13. Summary & Architect Takeaways

1.  **Never expose App Servers**: It's unprofessional and dangerous. Always use a Proxy.
2.  **Swiss Army Knife**: It's a Cache, a LB, a WAF, and a Compressor. Use the features.
3.  **Watch the Timeouts**: 80% of outages are 504 Timeouts. Config them correctly.
