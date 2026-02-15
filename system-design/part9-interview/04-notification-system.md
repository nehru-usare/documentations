# Design a Notification System

> **Part 9: Interview Scenarios**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** Ding!

---

## 0. Learning Objectives
*   **Beginner**: Understanding the mess of 3rd party providers (Twilio, SendGrid, FCM).
*   **Developer**: Building a retry mechanism with Exponential Backoff.
*   **Architect**: Designing a centralized "Notification Platform" for all microservices to use.

---

## 1. Problem Context
**The Ask**: Centralize notifications.
*   **Inputs**: From Order Service ("Order Shipped"), Marketing ("Promo"), Security ("OTP").
*   **Channels**: Email, SMS, iOS Push, Android Push, Slack Webhook.
*   **Scale**: 10M notifications/day.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Types of Notifications
*   **Transactional**: Critical. OTP, Pwd Reset. (Low Latency, High Reliability).
*   **Promotional**: Bulk. Newsletter. (High Latency, High Throughput).

### 2. Providers
*   **Email**: SendGrid, Amazon SES.
*   **SMS**: Twilio, Nexmo.
*   **Push**: FCM (Firebase - Android), APNS (Apple - iOS).
*   **Note**: You rarely build your own Email Server. You allowlisting IPs is a nightmare. Use Vendors.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Components
1.  **Notification Service**: API Gateway (`POST /send`).
2.  **Message Queue**: Buffer requests. Decouple senders from vendors.
3.  **Workers**: Read from Queue -> Call Vendor API.
4.  **Database**: Audit log (`user_id`, `status`, `created_at`).
5.  **Preference Service**: "User X opted out of Marketing SMS".

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Priority Queues
*   One Queue is dangerous. If Marketing sends 1M emails, the Queue fills up.
*   **OTP Service** cannot send Verification Code because Queue is full.
*   **Fix**: Separate Queues.
    *   `critical_queue`: OTPs. (High priority workers).
    *   `promo_queue`: Marketing. (Low priority workers).

### 2. Rate Limiting (Outbound)
*   Prevent spamming users.
*   Rule: "Max 3 SMS per user per hour".
*   Rule: "Max 1000 calls to Twilio per second" (Vendor limit).
*   Impl: Redis Token Bucket.

---

## 5. Trade-Off Analysis

| Feature | Direct Vendor Call | Queued Async System |
| :--- | :--- | :--- |
| **Latency** | Low | Medium |
| **Reliability** | Low (Vendor down = Fail) | High (Retry later) |
| **Flow Control** | None | Rate Limiting possible |
| **Verdict** | Prototype | **Production** |

---

## 6. Scaling Considerations

### Deduplication
*   Retry Logic might send duplicate emails.
*   **Idempotency**: Check `msg_id` in Redis. If processed, skip.
*   **Business Logic**: If "Welcome Email" triggered twice, only send once.

---

## 7. Failure Scenarios & Recovery

### 1. Vendor Outage
*   Twilio is down.
*   **Circuit Breaker**: Open circuit. Stop calling Twilio.
*   **Failover**: Switch to Nexmo (Secondary Provider).
*   *Requires*: Abstraction layer over vendors.

---

## 8. Security Considerations

### 1. Hardcoded Credentials
*   Do not put API Keys in code. Use Vault / Secrets Manager.

### 2. Phishing Protection
*   Ensure DKIM / SPF records are set up so emails land in Inbox, not Spam.

---

## 9. Performance Considerations

*   **Bulk Sending**: Vendors often have "Batch APIs".
*   Instead of 1000 HTTP calls, send 1 HTTP call with 1000 emails.
*   worker should aggregate messages.

---

## 10. Real Production Lessons

### LinkedIn
*   **Air Traffic Controller**: A system specifically designed to prevent "Notification Fatigue".
*   If User received 3 emails today -> Drop the 4th one.
*   Increases engagement by *reducing* volume.

---

## 11. Interview Questions

### Basic
1.  Why use a Queue?
2.  List common notification providers.
3.  How to prevent spamming a user?
4.  JSON payload structure for a notification request.
5.  What is a Webhook?

### Intermediate
1.  Design the Preference Center DB schema (Opt-in/Opt-out).
2.  How to handle Retries? (Exponential Backoff).
3.  How to track "Open Rates"? (Pixel tracking).
4.  Security of API Keys.
5.  Priority Queue implementation.

### Advanced
1.  Design a templating system. (Mustache/Jinja2 stored in DB, injected with content).
2.  Architect a Failover strategy for SMS providers.
3.  How to handle "Timezones"? (Don't SMS user at 3 AM).
4.  Critique "Fire and Forget" vs "Guaranteed Delivery".
5.  Scaling to 1 Billion push notifications (The fan-out problem).

### Architect-Level
1.  "Marketing wants to send 10M emails in 1 hour. Twilio limit is 100/sec." Architect the throttling and scaling solution.
2.  Design 'In-App Notifications' (Feed) alongside Push. (Requires Persistent Storage + WebSocket).
3.  Evaluate building vs buying a Notification Infrastructure (Courier/Novu).

---

## 12. Scenario-Based System Design Problems

### 1. Design OTP System
*   **Req**: < 5s delivery.
*   **Arch**: Critical Queue. Fallback (SMS -> Voice Call).

### 2. Design Newsletter System
*   **Req**: 1M subscribers.
*   **Arch**: Batch processing. Cron Job -> Queue -> Batch Send.

---

## 13. Summary & Architect Takeaways

1.  **Don't Spam**: The fastest way to lose users is annoying notifications.
2.  **Abstraction**: Isolate your code from Vendor APIs. Vendors change/fail.
3.  **Audit**: "I didn't get the email" is the #1 support ticket. Log everything.
