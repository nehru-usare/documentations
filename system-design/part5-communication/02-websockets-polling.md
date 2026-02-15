# WebSockets vs Long Polling vs SSE

> **Part 5: Communication**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** Event Driven Web

---

## 0. Learning Objectives
*   **Beginner**: Understand why hitting "Refresh" is bad for a Chat App.
*   **Developer**: Implement a WebSocket server in Node/Java.
*   **Architect**: Scale a stateful connection layer to 1 Million concurrent users.

---

## 1. Problem Context
**Why does this exist?**
HTTP is **Request-Response**. Client asks, Server answers.
Server cannot say: "Hey Client, here is a new message!" (Push).
**Real-Time Apps** (Chat, Stock Tickers) need Server Push.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. Short Polling (The Naive Way)
*   Client: "Any new msg?" (Every 1 sec).
*   Server: "No".
*   *Result*: Waste of resources. Empty responses.

### 2. Long Polling (The Hack)
*   Client: "Any new msg?"
*   Server: ... *Waits 30 seconds* ... "Yes!" (or Timeout).
*   Client: "Thanks", immediately asks again.
*   *Pros*: Unidirectional Push. Works everywhere.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### The Modern Options

1.  **WebSocket (WS)**:
    *   **Protocol**: Upgrades HTTP to a persistent TCP tunnel.
    *   **Direction**: Bi-Directional (Full Duplex).
    *   **Use**: Chat, Multiplayer Games.

2.  **Server-Sent Events (SSE)**:
    *   **Protocol**: Standard HTTP. Keeps content-type `text/event-stream`.
    *   **Direction**: Uni-Directional (Server -> Client only).
    *   **Use**: News Feed, Stock Ticker, Notifications.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. The Upgrade Handshake
*   WS starts as HTTP.
*   Client sends `Connection: Upgrade`, `Upgrade: websocket`.
*   Server sends `101 Switching Protocols`.
*   Connection stays open indefinitely.

### 2. Connection State
*   HTTP is Stateless.
*   WS is **Stateful**. Server knows "Connection 5 = User Bob".
*   *Scale Impact*: Load Balancers effectively lose their ability to balance active traffic.

---

## 5. Trade-Off Analysis

| Strategy | Bi-Directional | Complexity | Scale Limit |
| :--- | :--- | :--- | :--- |
| **Polling** | No | Low | Low (Overhead) |
| **Long Polling** | No | Medium | Medium |
| **SSE** | No | Low | High (HTTP) |
| **WebSocket** | **Yes** | High | Medium (Memory/Ports) |

---

## 6. Scaling Considerations

### The "Max Ports" Problem
*   A server has 65,535 ports.
*   One WS connection = 1 File Descriptor + 1 Port (conceptually at L4).
*   **C10k Problem**: Handling 10,000 connections.
*   **C10M Problem**: Handling 10 Million.
*   **Solution**: Tune Linux `ulimit`. Use specialized Gateway (Netty/Nginx) to offload connections.

---

## 7. Failure Scenarios & Recovery

### 1. Connection Drop
*   Mobile networks drop TCP connections constantly.
*   **Scenario**: User A sends message. Tunnel breaks. Message lost.
*   **Fix**: Application Layer Ack. "Did you get msg 5?" If no, resend. (Socket.io does this).

---

## 8. Security Considerations

### 1. CSWSH (Cross-Site WebSocket Hijacking)
*   Like CSRF.
*   Attacker site opens WS connection to `bank.com`. Browser sends cookies.
*   **Fix**: Check the `Origin` header during the Handshake.

---

## 9. Performance Considerations

*   **Heartbeats (Ping/Pong)**: Essential to keep NAT/Firewalls from closing the "Idle" connection.
*   **Latency**: WS is lower latency than HTTP (No header overhead per message).

---

## 10. Real Production Lessons

### WhatsApp's Implementation
*   **Language**: Erlang (Optimized for massive concurrency).
*   **Protocol**: XMPP (Modified).
*   **Architecture**: Optimized for millions of "Idle" connections that rarely speak.

---

## 11. Interview Questions

### Basic
1.  Why is Polling inefficient?
2.  What is full-duplex communication?
3.  Does SSE work for a Chat App? (Yes, if you use HTTP POST for sending).
4.  What port does WebSocket use? (80/443).
5.  What is Socket.io? (Library that abstracts WS/Polling).

### Intermediate
1.  Explain the WebSocket Handshake.
2.  How does a Load Balancer handle WebSockets? (Sticky Session often needed initially).
3.  Why choose SSE over WebSocket? (Simpler, Auto-Reconnect).
4.  Limit of concurrent connections on a single IP? (Ephemeral port limit).
5.  How do you auth a WebSocket? (Token in Query Param or initial message).

### Advanced
1.  Design a Pub/Sub system for a Chat App (Redis Pub/Sub + WS Servers).
2.  Analyze memory footprint of 100k WS connections in Java vs Go.
3.  How does HTTP/2 Server Push differ from SSE?
4.  Architect a "Presence" system (Who is Online) using WebSockets.
5.  Critique the use of WebTansport (QUIC based WS).

### Architect-Level
1.  "We have 10M concurrent users. We need to push a 'Goal Scored' notification to all." Architect it. (Broadcast tree).
2.  Design a fallback strategy: UDP -> WebSocket -> Long Polling.
3.  Evaluate the cost of maintaining persisting connections vs FaaS (Function as a Service) limitations.

---

## 12. Scenario-Based System Design Problems

### 1. Design Uber Live Location
*   **Req**: Driver location updates every 3 sec.
*   **Choice**: **WebSocket** (or UDP in future).
*   **Why**: High frequency, Bi-directional.

### 2. Design Facebook Notification
*   **Req**: Occasional "Ding".
*   **Choice**: **SSE** (Server Sent Events).
*   **Why**: Uni-directional. Battery efficient.

---

## 13. Summary & Architect Takeaways

1.  **Don't open a socket for everything**: It drains mobile battery. Use Push Notifications (APNS/FCM) for background apps.
2.  **Fallbacks matter**: Corporate firewalls block non-HTTP traffic. Long Polling is the robust fallback.
3.  **State is Hard**: Websockets make your server Stateful. Scaling is 10x harder than Stateless REST.
