# Bloom Filters

> **Part 6: Patterns**  
> **Difficulty:** ⭐⭐⭐ (Intermediate)  
> **Status:** Maybe Yes, Definitely No

---

## 0. Learning Objectives
*   **Beginner**: Understand how to check if a username is taken without hitting the DB.
*   **Developer**: Implement a Bloom Filter using `BitSet` and MurmurHash.
*   **Architect**: Optimize storage costs by 90% using probabilistic structures.

---

## 1. Problem Context
**Why does this exist?**
You have a Database of 1 Billion "Bad URLs" (Phishing).
User visits `google.com`.
*   **Option 1**: Query DB. (Too slow for every page load).
*   **Option 2**: Cache all URLs in RAM. (1 Billion Strings * 100 bytes = 100GB RAM). Too expensive.
**Bloom Filter**: Store 1 Billion items in 1GB RAM.

---

## 2. Core Concepts (🟢 Beginner Level)

### 1. The Promise
*   Query: "Is X in the set?"
*   Answer: **"No"** (100% Certain) OR **"Maybe"** (Small chance of error).
*   **Never** returns "No" if it is actually "Yes" (No False Negatives).

### 2. The Bit Array
*   An array of $m$ bits, all set to 0.
*   $k$ Hash functions.

---

## 3. Architecture Breakdown (🟡 Developer Level)

### How it works
1.  **Add("Apple")**:
    *   Hash1("Apple") -> 5. Set Bit[5] = 1.
    *   Hash2("Apple") -> 12. Set Bit[12] = 1.
    *   Hash3("Apple") -> 9. Set Bit[9] = 1.
2.  **Check("Apple")**:
    *   Are bits 5, 12, and 9 all 1? **Yes**. (Maybe Present).
3.  **Check("Banana")**:
    *   Hashes to 5, 20, 22.
    *   Bit 20 is 0. **No**. (Definitely Not Present).

---

## 4. Internal Mechanics (🔴 Architect Level)

### 1. False Positive Rate
*   Depends on size of bit array ($m$) and number of inserted elements ($n$).
*   Formula: $P \approx (1 - e^{-kn/m})^k$.
*   **Trade-off**: Larger array = Lower error rate.

### 2. Deletion Impossible
*   You cannot delete "Apple".
*   Why? If you set bit 5 to 0, you might accidentally remove "Orange" which also mapped to bit 5.
*   *Fix*: **Counting Bloom Filter** (Uses counters instead of bits, but uses 4x memory).

---

## 5. Trade-Off Analysis

| Strategy | Space | Time | Precision |
| :--- | :--- | :--- | :--- |
| **HashSet** | High (O(N)) | O(1) | 100% |
| **Bloom Filter** | Low (Bits) | O(k) | < 100% (False Positives) |
| **Linear Scan** | None | O(N) | 100% |

---

## 6. Scaling Considerations

### Sizing BitSet
*   For 1 Billion items with 1% error rate:
*   Standard Hash Set: 100 GB.
*   Bloom Filter: 1.2 GB.

---

## 7. Failure Scenarios & Recovery

### 1. Saturation
*   If you fill the Bloom Filter beyond capacity ($n > m$), nearly all bits become 1.
*   **Result**: False Positive rate goes to 100%. The filter becomes useless.
*   **Fix**: Scalable Bloom Filter (Stacking filters).

---

## 8. Security Considerations

### 1. No Encryption
*   Bloom Filters do not hide the data perfectly.
*   An attacker can brute-force the filter to enumerate the set.

---

## 9. Performance Considerations

*   **Hash Function**: Must be fast and uniform.
*   **Good**: MurmurHash, FNV.
*   **Bad**: SHA-256 (Too slow), MD5 (Too slow).

---

## 10. Real Production Lessons

### Google Chrome
*   **Use Case**: Malicious URL check.
*   **Flow**: Browser checks local Bloom Filter.
    *   If "No" -> Safe to browse. (Fast path).
    *   If "Maybe" -> Call Google API to verify. (Slow path).

---

## 11. Interview Questions

### Basic
1.  What is a Bloom Filter?
2.  Can a Bloom Filter give a False Negative? (No).
3.  Can a Bloom Filter give a False Positive? (Yes).
4.  Can you delete from a Bloom Filter? (No).
5.  Why use multiple hash functions?

### Intermediate
1.  Calculate required bits for N items and P probability.
2.  What is a Counting Bloom Filter?
3.  Bloom Filter vs Cuckoo Filter. (Cuckoo supports deletion).
4.  Why not use Java's `hashCode()`? (Not uniform enough, implementation varies).
5.  How do you handle saturation?

### Advanced
1.  Design a "Scalable Bloom Filter" that grows dynamically.
2.  Analyze the CPU cache efficiency of Bloom Filters (Random access = Bad).
3.  Critique the use of Bloom Filters in Cassandra (SSTable checking).
4.  How to use Bloom Filters for "Set Intersection" estimation.
5.  Explain "Quotient Filter" as an alternative.

### Architect-Level
1.  "We have a DB with 50TB on disk. Writes are slow due to disk seeks checking for duplicates." Architect a solution. (Bloom Filter in RAM).
2.  Design a Distributed Bloom Filter for a P2P network.
3.  Evaluate the network bandwidth savings of sending a Bloom Filter vs the full set for synchronization (Set Reconciliation).

---

## 12. Scenario-Based System Design Problems

### 1. Design Username Availability
*   **Req**: "Is 'bob' taken?"
*   **Solution**: Bloom Filter on valid usernames.
*   **Logic**: If BF says "Maybe", check DB. If "No", say "Available".
*   **Edge Case**: If BF says "Maybe" but DB says "Available", you just did an extra DB call. Acceptable.

### 2. Design CDN Cache Avoidance
*   **Req**: Don't cache Object unless requested at least twice ("One Hit Wonder" problem).
*   **Solution**:
    *   Req 1: Check BF. Not present. Add to BF. Pass to Origin.
    *   Req 2: Check BF. Present. Cache it.

---

## 13. Summary & Architect Takeaways

1.  **Memory Efficiency**: The go-to tool for reducing RAM usage for large sets.
2.  **IO Savings**: Use it to avoid expensive Disk/Network calls.
3.  **Know the limits**: Don't use it if you need 100% precision or deletions (use Cuckoo Filter).
