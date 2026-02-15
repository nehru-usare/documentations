# PostgreSQL Internals (MVCC, VACUUM, WAL)

> **Part 2: Internals**  
> **Difficulty:** ⭐⭐⭐⭐⭐ (Deep Dive)  
> **Status:** Vacuuming...

---

## 0. Learning Objectives
*   **Beginner**: Why `DELETE` doesn't free up disk space immediately.
*   **Developer**: Why long-running transactions kill performance.
*   **Architect**: Tuning Autovacuum for terabyte-scale tables.

---

## 1. Core Concepts (🟢 Beginner Level)

### MVCC (Multi-Version Concurrency Control)
*   **Rule**: Readers don't block Writers. Writers don't block Readers.
*   **Implementation**:
    *   When you `UPDATE` a row, Postgres does **not** overwrite it.
    *   It marks old row as "Dead" (xmin/xmax).
    *   It inserts a "New" version of the row.
*   *Consequence*: Tables accumulate "Dead Tuples" (Garbage).

---

## 2. Architecture Breakdown (The Cleanup)

### VACUUM
*   The Garbage Collector.
*   Scans table for dead tuples. Marks space as "free" for reuse.
*   **VACUUM FULL**: Rewrites the whole table. Locks it exclusive. Repatriates disk space to OS. (Dangerous).
*   **Autovacuum**: Daemon that runs in background. Tunes itself.

### Transaction ID Wraparound
*   Transactions have 32-bit IDs. They run out after 4 Billion.
*   **Freeze**: Postgres must "Freeze" old rows (set XID to special value) to prevent ID wrap panic.
*   *Critical*: If Autovacuum fails to freeze, DB shuts down to prevent data corruption.

---

## 3. Internal Mechanics (🔴 Architect Level)

### 1. The WAL (Write Ahead Log)
*   Before modifying shared buffers (RAM), change is written to WAL (Disk).
*   **Checkpoints**: Periodically, dirty pages in RAM are flushed to Data Files.
*   **Tuning**: `checkpoint_timeout` (default 5m). Increase to 30m for write-heavy loads to reduce I/O spikes.

### 2. TOAST (The Oversized Attribute Storage Technique)
*   Postgres page size is 8KB.
*   If a row is > 2KB (e.g., big JSONB), it's compressed and moved to a separate TOAST table.
*   *Performance*: Accessing TOAST is slower but keeps the main table small/fast.

---

## 4. Scaling Considerations

### Connection Pooling (Pgbouncer)
*   Postgres uses "Process per Connection". High RAM overhead per user.
*   **Limit**: ~100-300 active connections per core is max.
*   **Solution**: Use **Pgbouncer** (Transaction Pooling) to map 10,000 Users -> 100 DB Connections.

---

## 5. Failure Scenarios & Recovery

### 1. Bloat
*   Table size = 100GB. Live Data = 10GB. Dead Tuples = 90GB.
*   **Cause**: Aggressive Updates/Deletes + Autovacuum too slow.
*   **Fix**: Tune `autovacuum_vacuum_scale_factor`. Use `pg_repack` to reclaim space without locking.

---

## 6. Real Production Lessons

### "The Long Transaction"
*   If User A starts a transaction at 9:00 AM and leaves it open...
*   Postgres cannot Vacuum *any* row updated after 9:00 AM (because User A *might* need to see the old version).
*   **Result**: Massive Bloat.
*   **Fix**: Set `idle_in_transaction_session_timeout`.

---

## 7. Interview Questions

### Basic
1.  Does DELETE free disk space in Postgres? (No, marks dead).
2.  What is VACUUM?
3.  Why use Connection Pooling?

### Intermediate
1.  Explain MVCC.
2.  What happens during Transaction ID Wraparound?
3.  Difference between `VACUUM` and `VACUUM FULL`.

### Advanced
1.  How to tune Autovacuum for a high-write table? (Aggressive settings).
2.  Explain the relationship between Checkpoints and Crash Recovery time.
3.  How does Postgres store a 1MB text field? (TOAST).

---

## 8. Summary & Architect Takeaways

1.  **Monitor Bloat**: Use `pgstattuple` or queries to track dead tuple ratio.
2.  **Kill Idle Transactions**: An open transaction is a liability.
3.  **Pgbouncer is mandatory**: Never expose Postgres directly to the web without a pooler.
