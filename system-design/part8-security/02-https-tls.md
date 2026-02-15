# HTTPS & TLS Handshake

> **Part 8: Security**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Expert)  
> **Status:** The Encrypted Tunnel

---

## 0. Learning Objectives
*   **Beginner**: Why the Green Lock icon matters.
*   **Developer**: Installing a Let's Encrypt certificate on Nginx.
*   **Architect**: Optimizing TLS termination for minimal latency (TLS 1.3).

---

## 1. Problem Context
**Why does this exist?**
HTTP is plain text.
*   User sends `password=123`.
*   Hacker on the Wifi (Man in the Middle) sees `password=123`.
**HTTPS** encrypts the channel using TLS (Transport Layer Security).

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Public Key Infrastructure (PKI)
*   **Private Key**: Keep secret on server.
*   **Public Key**: Give to everyone.
*   **Ceritificate**: A badge saying "I am google.com", signed by a Trusted Authority (Godaddy/Digicert).

### 2. The Chain of Trust
*   Browser trusts Root CA.
*   Root CA trusts Intermediate CA.
*   Intermediate CA signs Your Certificate.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The Handshake (TLS 1.2)
1.  **Client Hello**: "I support AES encryption. Here is a random number."
2.  **Server Hello**: "Let's use AES. Here is my Certificate + Random Number."
3.  **Key Exchange**: Client verifies Cert. Generates "Pre-Master Secret", encrypts with Server Public Key. Sends to Server.
4.  **Finished**: Both generate Session Keys. Switch to Encrypted mode.
*   **Cost**: 2 Round Trips (RTT). Slow.

### The Handshake (TLS 1.3)
*   Optimized.
*   **1 RTT**: Client sends Key Share immediately (optimistic).
*   **0 RTT**: Resumption of previous session (Fastest).

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. SNI (Server Name Indication)
*   **Problem**: One IP hosts `a.com` and `b.com`.
*   Server needs to know WHICH cert to send *before* decryption.
*   **Fix**: Client sends hostname in plain text in Client Hello.

### 2. Perfect Forward Secrecy (PFS)
*   **Scenario**: Hacker records encrypted traffic for 5 years.
*   **Event**: Hacker steals your Private Key today.
*   **Without PFS**: Hacker decrypts *all past 5 years* of traffic.
*   **With PFS**: Hacker can only decrypt *future* traffic. Ephemeral keys are used for session.

---

## 5. Trade-Off Analysis

| Feature | HTTP | HTTPS |
| :--- | :--- | :--- |
| **Privacy** | None | High |
| **Integrity** | None (Injection possible) | High (Tamper proof) |
| **Latency** | Low (TCP Handshake) | Higher (TCP + TLS Handshake) |
| **CPU** | Low | Higher (Encryption math) |

---

## 6. Scaling Considerations

### TLS Termination
*   Decryption is CPU intensive.
*   **Strategy**: Offload TLS at the Load Balancer (Nginx/ALB).
*   Traffic inside the VPC is HTTP (or weaker mTLS) to save CPU on App Servers.

---

## 7. Failure Scenarios & Recovery

### 1. Certificate Expiry
*   The classic outage.
*   cert expires. Browser shows Big Red Warning. Users flee.
*   **Fix**: Automated renewal (Certbot/ACM). Monitoring (Alert 30 days before).

---

## 8. Security Considerations

### 1. Mixed Content
*   HTTPS page loads HTTP script.
*   Browser blocks it. Site breaks.

---

## 9. Performance Considerations

*   **Session Resumption**: Don't do full handshake every time. Use Session ID / Tickets.
*   **OCSP Stapling**: Server sends "Certificate Status" (Revocation check) so browser doesn't have to call CA.

---

## 10. Real Production Lessons

### Let's Encrypt Revolution
*   Before: Certs cost $100/yr. Manual process.
*   Now: Free. Automated.
*   **Result**: 90%+ of web is now encrypted.

---

## 11. Interview Questions

### Basic
1.  Difference between HTTP and HTTPS.
2.  What is a Certificate Authority?
3.  Why does TLS slow down the first request?
4.  Green Lock vs Grey Lock.
5.  What is Port 443?

### Intermediate
1.  Explain the TLS 1.2 Handshake steps.
2.  What is Client Hello?
3.  Difference between Symmetric and Asymmetric encryption in TLS. (Asym for handshake, Sym for data).
4.  What is TLS Termination?
5.  What happens if the internal clock is wrong? (Cert validation fails).

### Advanced
1.  Analyze TLS 1.3 improvements (1-RTT, 0-RTT).
2.  How does SNI allow Virtual Hosting?
3.  Explain HSTS (Strict Transport Security). prevents Downgrade attacks.
4.  Design a Certificate Rotation system for 10,000 servers.
5.  Critique usage of Self-Signed certs in production.

### Architect-Level
1.  "We handle Payments. Architect the encryption flow." (End-to-End Encryption. Termination at LB is not enough for PCI-DSS sometimes).
2.  Evaluate the performance impact of RSA 2048 vs ECDSA (Elliptic Curve). (ECDSA is faster/smaller).
3.  Design a system to prevent "Man in the Middle" in mobile apps (Certificate Pinning).

---

## 12. Scenario-Based System Design Problems

### 1. Design Secure Banking App
*   **Req**: Zero trust.
*   **Arch**: Certificate Pinning in App. HSTS preloading. TLS 1.3.

### 2. Design CDN
*   **Req**: Serve `customer.com` traffic.
*   **Arch**: Customer uploads Cert (or Key) to CDN. CDN serves it via SNI.

---

## 13. Summary & Architect Takeaways

1.  **HTTPS Everywhere**: Even for internal tools. Network is hostile.
2.  **Automate Rotation**: Humans forget expiry dates. Bots don't.
3.  **Upgrade to TLS 1.3**: Free performance boost.
