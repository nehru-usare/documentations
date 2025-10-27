# 🛡️ Java 21 — Production Hardening and Reliability Engineering Guide

> “Stability isn’t built by avoiding failure — it’s built by surviving it.”  
> — *Principle of Chaos Engineering*

This document focuses on **hardening Java 21 services for production** — ensuring reliability, resilience, and security through proactive design and testing.

It’s written for **senior developers and SREs** responsible for maintaining high-availability, mission-critical Java applications.

---

## 🧭 Table of Contents

1. [Production Hardening Overview](#1-production-hardening-overview)
2. [Resilience Patterns](#2-resilience-patterns)
3. [Retries, Timeouts, and Circuit Breakers](#3-retries-timeouts-and-circuit-breakers)
4. [Graceful Shutdown and Restart Strategies](#4-graceful-shutdown-and-restart-strategies)
5. [Resource and Connection Management](#5-resource-and-connection-management)
6. [Rate Limiting and Throttling](#6-rate-limiting-and-throttling)
7. [Chaos and Fault Injection Testing](#7-chaos-and-fault-injection-testing)
8. [Secure Configuration and Secrets Management](#8-secure-configuration-and-secrets-management)
9. [Observability-Driven Reliability](#9-observability-driven-reliability)
10. [Disaster Recovery and Failover Patterns](#10-disaster-recovery-and-failover-patterns)
11. [Production Checklists](#11-production-checklists)
12. [Official Resources and References](#12-official-resources-and-references)
13. [Summary](#13-summary)

---

## 1️⃣ Production Hardening Overview

Production hardening is about preparing for **failure, stress, and unpredictability**.

### Goals:
✅ Predictable behavior under load  
✅ Quick recovery from failure  
✅ Data integrity and consistency  
✅ Observability and traceability  
✅ Minimal downtime during updates  

### Core Pillars:
1. **Resilience** — Survive failures gracefully  
2. **Security** — Protect data and configs  
3. **Scalability** — Handle growth dynamically  
4. **Observability** — Detect, diagnose, act  

---

## 2️⃣ Resilience Patterns

| Pattern | Purpose | Example |
|----------|----------|----------|
| **Retry** | Reattempt transient failures | API call retries |
| **Circuit Breaker** | Prevent cascading failures | Downstream outages |
| **Bulkhead** | Isolate resource pools | Thread pools per subsystem |
| **Timeout** | Fail fast on slow ops | Network latency |
| **Fallback** | Provide defaults | Cached response |

> ⚙️ Implement with **Resilience4j** or **Spring Cloud Circuit Breaker**.

---

## 3️⃣ Retries, Timeouts, and Circuit Breakers

### Example using Resilience4j:
```java
@Retry(name = "orderService")
@CircuitBreaker(name = "orderService", fallbackMethod = "fallbackOrder")
public Order fetchOrder(String id) {
    return restTemplate.getForObject("/orders/" + id, Order.class);
}

public Order fallbackOrder(String id, Throwable ex) {
    log.warn("Fallback for {}", id);
    return new Order(id, "Fallback order", 0);
}
````

### Config Example:

```yaml
resilience4j:
  circuitbreaker:
    instances:
      orderService:
        slidingWindowSize: 10
        failureRateThreshold: 50
        waitDurationInOpenState: 10s
  retry:
    instances:
      orderService:
        maxAttempts: 3
        waitDuration: 500ms
```

✅ Add timeouts at all I/O layers (DB, HTTP).
✅ Fail fast, recover gracefully.

---

## 4️⃣ Graceful Shutdown and Restart Strategies

When deploying updates or scaling down pods, your service must **complete ongoing work safely**.

### Spring Boot Example:

```java
@Bean
public GracefulShutdown gracefulShutdown() {
    return new GracefulShutdown();
}
```

### Kubernetes Deployment:

```yaml
terminationGracePeriodSeconds: 30
lifecycle:
  preStop:
    exec:
      command: ["/bin/sh", "-c", "sleep 10"]
```

✅ Handle `SIGTERM` signal properly.
✅ Wait for inflight requests to finish.
✅ Close DB and message broker connections cleanly.

---

## 5️⃣ Resource and Connection Management

### Database:

* Use **HikariCP** with limits:

  ```yaml
  spring.datasource.hikari:
    maximum-pool-size: 20
    idle-timeout: 30000
  ```
* Monitor pool exhaustion with Micrometer.

### Threading:

* Avoid unbounded thread pools.
* Use **Virtual Threads** for I/O scalability:

  ```java
  var executor = Executors.newVirtualThreadPerTaskExecutor();
  ```

### Caching:

* Limit cache sizes with **Caffeine**:

  ```java
  Caffeine.newBuilder().maximumSize(1000).build();
  ```

---

## 6️⃣ Rate Limiting and Throttling

Prevents abuse and protects downstream services.

### Using Bucket4j:

```java
Bucket bucket = Bucket4j.builder()
    .addLimit(Bandwidth.classic(100, Refill.intervally(100, Duration.ofMinutes(1))))
    .build();

if (bucket.tryConsume(1)) {
    processRequest();
} else {
    throw new RateLimitException("Too Many Requests");
}
```

✅ Global + per-user rate limiting.
✅ Apply at gateway or API level.

---

## 7️⃣ Chaos and Fault Injection Testing

**Chaos Engineering** tests how systems behave under failure.

### Tools:

* [Chaos Monkey for Spring Boot](https://github.com/codecentric/chaos-monkey-spring-boot)
* [Gremlin](https://www.gremlin.com/)
* [LitmusChaos](https://litmuschaos.io/)

### Example:

```yaml
chaos.monkey:
  watcher:
    controller: true
    service: true
  assaults:
    latency-active: true
    latency-range-start: 1000
    latency-range-end: 3000
```

✅ Inject latency, exceptions, and restarts during staging.
✅ Validate resilience before production.

> 💡 If your system can’t survive chaos in staging, it won’t in production.

---

## 8️⃣ Secure Configuration and Secrets Management

✅ Never hardcode credentials or secrets.
✅ Use:

* **Spring Cloud Config**
* **HashiCorp Vault**
* **Kubernetes Secrets**

### Example:

```yaml
spring:
  config:
    import: vault://
```

### Environment Variables:

```bash
export DB_PASSWORD=$(vault kv get -field=password secret/db)
```

✅ Rotate secrets regularly.
✅ Use TLS/SSL for all external communication.
✅ Restrict sensitive logs.

---

## 9️⃣ Observability-Driven Reliability

Hardening = Monitoring + Alerts + Traces.

### Micrometer Metrics:

```java
registry.gauge("order.queue.size", queue, Queue::size);
```

### Prometheus Alert Rules:

```yaml
- alert: HighErrorRate
  expr: rate(http_server_requests_seconds_count{status=~"5.."}[5m]) > 0.05
  for: 1m
  labels:
    severity: warning
```

✅ Define **SLIs**, **SLOs**, **SLAs**.
✅ Correlate incidents via trace IDs (OpenTelemetry).
✅ Enable alert-driven feedback loops.

---

## 🔟 Disaster Recovery and Failover Patterns

| Pattern                    | Description                     | Tool                    |
| -------------------------- | ------------------------------- | ----------------------- |
| **Active-Active**          | Both data centers serve traffic | Multi-region deployment |
| **Active-Passive**         | Failover standby                | Cloud Load Balancer     |
| **Blue-Green Deployments** | Parallel environments           | ArgoCD / Spinnaker      |
| **Canary Releases**        | Gradual rollout                 | Istio / Linkerd         |
| **Data Replication**       | Sync/async DB replication       | PostgreSQL / MongoDB    |
| **Backups**                | Regular snapshotting            | S3 / Velero             |

✅ Automate failover testing quarterly.
✅ Validate data recovery steps.

---

## 11️⃣ Production Checklists

### ✅ Pre-Deployment

* [ ] Liveness & readiness probes configured
* [ ] Graceful shutdown enabled
* [ ] Retry & timeout policies set
* [ ] Secrets loaded from Vault/K8s
* [ ] Circuit breakers tested

### ✅ Post-Deployment

* [ ] Observability dashboards green
* [ ] JFR running in background
* [ ] Metrics scraped by Prometheus
* [ ] Alerts tested in Grafana
* [ ] Chaos simulation validated

---

## 12️⃣ Official Resources and References

📘 **Resilience & Reliability**

* [Resilience4j](https://resilience4j.readme.io/)
* [Spring Boot Resilience Patterns](https://docs.spring.io/spring-cloud-circuitbreaker/docs/current/reference/html/)
* [Chaos Monkey for Spring Boot](https://github.com/codecentric/chaos-monkey-spring-boot)
* [LitmusChaos](https://litmuschaos.io/)
* [Netflix Chaos Engineering](https://netflixtechblog.com/tagged/chaos-engineering)

📊 **Security & Config**

* [Spring Cloud Config Docs](https://spring.io/projects/spring-cloud-config)
* [HashiCorp Vault](https://developer.hashicorp.com/vault)
* [Kubernetes Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
* [OWASP Java Security Guidelines](https://owasp.org/www-project-top-ten/)

📈 **Observability**

* [Micrometer Metrics](https://micrometer.io/)
* [OpenTelemetry Java](https://opentelemetry.io/docs/instrumentation/java/)
* [Prometheus Alerting Rules](https://prometheus.io/docs/alerting/latest/configuration/)

---

## 13️⃣ Summary

You’ve now learned how to **harden Java 21 services for production**:

✅ Build resilient services with retries, timeouts, and circuit breakers
✅ Handle graceful shutdowns and resource cleanup
✅ Enforce secure configuration and secret management
✅ Simulate real-world failures with chaos testing
✅ Monitor and recover automatically via observability-driven design

> 🧠 “Reliability isn’t a feature — it’s a culture.”
> Java 21 gives you the tools to run **mission-critical, cloud-native systems** with confidence.

> 🧭 **Next (Final Advanced Section):**
> [26-java-operational-excellence.md → Operational Excellence, Automation, and SRE Practices for Java 21 Systems](./26-java-operational-excellence.md)