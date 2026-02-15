# Disaster Recovery (RPO/RTO)

> **Part 8: Reliability**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** The Plan B

---

## 0. Learning Objectives
*   **Beginner**: Difference between a Backup and a DR Plan.
*   **Developer**: Why "US-EAST-1" going down shouldn't kill your business.
*   **Architect**: Calculating RPO/RTO and justifying the cost of Multi-Region.

---

## 1. Problem Context
**Why does this exist?**
Datacenters burn. Hurricanes happen. Ransomware hits.
**Disaster Recovery (DR)**: The art of surviving the worst case.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. RPO (Recovery Point Objective)
*   **"How much data can we lose?"**
*   Last backup was at 1:00 AM. Disaster at 2:00 AM.
*   Lost 1 hour data. RPO = 1 Hour.
*   *Ideal*: 0. (Expensive).

### 2. RTO (Recovery Time Objective)
*   **"How long until we are back online?"**
*   Disaster at 2:00 AM. System back up at 4:00 AM.
*   Down for 2 hours. RTO = 2 Hours.
*   *Ideal*: 0. (Expensive).

---

## 3. Architecture Breakdown (🟡 Developer Level)

### Strategies

1.  **Backup & Restore (Cheap)**:
    *   Daily Snapshots to S3.
    *   RPO: 24h. RTO: Hours (Restore time).

2.  **Pilot Light**:
    *   DB data replicated continuously. App servers Off.
    *   Disaster -> Turn on App Servers.
    *   RPO: Minutes. RTO: Minutes.

3.  **Warm Standby**:
    *   Scaled down version running in DR region.
    *   Disaster -> Scale up.
    *   RPO: Seconds. RTO: Minutes.

4.  **Multi-Site Active-Active (Expensive)**:
    *   Both regions taking traffic.
    *   Disaster -> Route all traffic to survivor.
    *   RPO: ~0. RTO: ~0.

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. Data Replication Physics
*   Speed of Light: NY to London = 70ms round trip.
*   **Sync Replication**: 0 RPO, but adds 70ms latency to every write.
*   **Async Replication**: Low latency, but >0 RPO (Data in flight is lost).
*   *Verdict*: Most DR uses Async. You accept small data loss to save performance.

### 2. DNS Failover
*   Update Route53 to point `api.com` from US-IP to EU-IP.
*   *Problem*: DNS Caching (TTL). Users might still hit dead region for 5 mins.

---

## 5. Trade-Off Analysis

| Strategy | Cost | RPO | RTO |
| :--- | :--- | :--- | :--- |
| **Backup** | $ | High | High |
| **Pilot Light** | $$ | Medium | Medium |
| **Active-Active** | $$$$ | Low | Low |

---

## 6. Scaling Considerations

### The "Thundering Herd" in Region B
*   If Region A handles 50% traffic and fails.
*   Region B suddenly gets 100% traffic (2x load).
*   **Risk**: Region B crashes immediately.
*   **Fix**: Over-provision Region B or have aggressive Auto-Scaling (Step Scaling).

---

## 7. Failure Scenarios & Recovery

### 1. Corrupted Data Replication
*   **Scenario**: Bug deletes `Users` table in Primary.
*   **Event**: Replication deletes `Users` table in Secondary instantly.
*   **Result**: DR is useless.
*   **Fix**: **Point-in-Time Recovery (PITR)** / Delayed Replication.

---

## 8. Security Considerations

### 1. Ransomware and Backups
*   Attackers target backups first.
*   **Fix**: **WORM Storage** (Write Once Read Many). AWS S3 Object Lock. Even Admin cannot delete backups.

---

## 9. Performance Considerations

*   **Failback**: Moving back to Primary after it heals.
*   Data synchronization (Delta sync) is hard. often easier to make Secondary the new Primary.

---

## 10. Real Production Lessons

### Delta Airlines Outage
*   **Event**: Power outage.
*   **Issue**: Non-critical systems booted first and choked the network, blocking critical systems.
*   **Lesson**: Boot Priority matters.

---

## 11. Interview Questions

### Basic
1.  Define RPO and RTO.
2.  Difference between HA (High Availability) and DR (Disaster Recovery).
3.  What is a Cold Standby?
4.  Why are backups important?
5.  What is Failover?

### Intermediate
1.  Pilot Light vs Warm Standby.
2.  How to test DR? (Game Days).
3.  Impact of DNS TTL on RTO.
4.  Sync vs Async replication trade-offs.
5.  What is Split Brain in DR?

### Advanced
1.  Design a DR plan for a Financial system (RPO=0). (Must use Sync rep + Active-Active).
2.  Analyze the cost of Active-Active vs Active-Passive.
3.  How to handle "Data Residency" laws in DR? (Cannot failover German data to US).
4.  Critique reliance on a single Cloud Provider (Multi-Cloud DR?).
5.  Explain "Chaos Engineering" relation to DR.

### Architect-Level
1.  "We have a localized outage (AZ down). Do we trigger Region Failover?" (No. Use Multi-AZ HA first. Region Failover is nuclear option).
2.  Design the failback procedure for a Multi-Master database after a partition heal.
3.  Evaluate the business impact of 1 hour downtime vs $1M/year infra cost.

---

## 12. Scenario-Based System Design Problems

### 1. Design Startup DR
*   **Budget**: Low.
*   **Strategy**: S3 Backups + Script to spin up Infrastructure (Terraform).
*   **RTO**: 4 hours. Acceptable.

### 2. Design 911 Service DR
*   **Budget**: Infinite.
*   **Strategy**: Active-Active. Over-provisioned.
*   **RTO**: 0.

---

## 13. Summary & Architect Takeaways

1.  **HA != DR**: HA handles server failure. DR handles datacenter failure.
2.  **Untested DR is a myth**: If you haven't run a Game Day (Simulated disaster), you have no DR.
3.  **Business Decision**: RPO/RTO are business metrics, not tech specs. Ask the CEO "How much money do we lose per hour?"
