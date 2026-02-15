# 🗄️ SQL Basics - Layered Engineering Guide

> **Document Level:** Beginner → Intermediate → Senior → Architect  
> **Last Updated:** February 2026 | **Target:** 0-10 Year Experience Range

---

## 📋 Table of Contents

1. [Layer 1: Simple Understanding (Beginner)](#layer-1-simple-understanding)
2. [Layer 2: Practical Engineering (Intermediate)](#layer-2-practical-engineering)
3. [Layer 3: Advanced Engineering (Senior)](#layer-3-advanced-engineering)
4. [Layer 4: Architect Thinking](#layer-4-architect-thinking)
5. [Historical Evolution](#historical-evolution)
6. [Performance & Scalability](#performance--scalability)
7. [Security Considerations](#security-considerations)
8. [Common Mistakes & Anti-Patterns](#common-mistakes--anti-patterns)
9. [Production Examples](#production-examples)
10. [Interview Questions](#interview-questions)

---

## Layer 1: Simple Understanding (Beginner)

### What is SQL?

**SQL (Structured Query Language)** is a standardized language for managing and querying relational databases. It allows you to:
- Store data in tables
- Retrieve data using queries
- Update and delete data
- Create database structures

### Why Does SQL Exist?

Before SQL, accessing data required custom programs. SQL provides a **declarative** way to work with data:
- You describe **what** you want, not **how** to get it
- The database engine figures out the optimal execution plan

### Basic SQL Operations (CRUD)

```sql
-- CREATE: Insert new data
INSERT INTO users (name, email, age) 
VALUES ('Alice', 'alice@example.com', 28);

-- READ: Query data
SELECT name, email FROM users WHERE age > 25;

-- UPDATE: Modify existing data
UPDATE users SET age = 29 WHERE name = 'Alice';

-- DELETE: Remove data
DELETE FROM users WHERE age < 18;
```

### Core Concepts

| Concept | Description |
|---------|-------------|
| **Table** | Collection of rows and columns (like Excel sheet) |
| **Row** | Single record in a table |
| **Column** | Attribute or field of data |
| **Primary Key** | Unique identifier for each row |
| **Foreign Key** | Reference to another table's primary key |

### Where is SQL Used?

- Web applications (user accounts, products, orders)
- Banking systems (transactions, accounts)
- E-commerce platforms (inventory, customers)
- Analytics and reporting
- Content management systems

---

## Layer 2: Practical Engineering (Intermediate)

### Real Use Cases

#### 1. User Authentication System

```sql
-- Create users table
CREATE TABLE users (
    user_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_login TIMESTAMP,
    is_active BOOLEAN DEFAULT TRUE
);

-- Create index for faster lookups
CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_username ON users(username);
```

#### 2. E-commerce Orders

```sql
-- Orders with foreign key relationships
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    total_amount DECIMAL(10, 2) NOT NULL,
    order_status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

CREATE TABLE order_items (
    item_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    order_id BIGINT NOT NULL,
    product_id BIGINT NOT NULL,
    quantity INT NOT NULL,
    unit_price DECIMAL(10, 2) NOT NULL,
    FOREIGN KEY (order_id) REFERENCES orders(order_id) ON DELETE CASCADE
);
```

### Common Mistakes (Intermediate Level)

#### ❌ Mistake 1: SELECT *

```sql
-- BAD: Retrieves unnecessary data
SELECT * FROM users WHERE user_id = 123;

-- GOOD: Only get what you need
SELECT user_id, username, email FROM users WHERE user_id = 123;
```

**Why bad?**
- Network overhead
- Application memory waste
- Breaks when table schema changes

#### ❌ Mistake 2: No WHERE Clause on Updates

```sql
-- DANGEROUS: Updates ALL rows
UPDATE users SET is_active = FALSE;

-- SAFE: Always use WHERE
UPDATE users SET is_active = FALSE WHERE user_id = 123;
```

#### ❌ Mistake 3: N+1 Query Problem

```sql
-- BAD: 1 query for orders + N queries for users
SELECT * FROM orders; -- Returns 100 orders
-- Then in application code:
-- for each order: SELECT * FROM users WHERE user_id = ?

-- GOOD: Use JOIN to fetch in one query
SELECT o.*, u.username, u.email
FROM orders o
JOIN users u ON o.user_id = u.user_id;
```

### Best Practices

| Practice | Why Important |
|----------|---------------|
| Use **prepared statements** | Prevents SQL injection |
| Always use **transactions** for multi-step operations | Ensures data consistency |
| Add **indexes** on WHERE/JOIN columns | Dramatically improves query speed |
| Use **LIMIT** for pagination | Avoids loading millions of rows |
| Use **appropriate data types** | Saves storage and improves performance |

### Performance Basics

```sql
-- Use EXPLAIN to understand query execution
EXPLAIN SELECT * FROM users WHERE email = 'alice@example.com';

-- Output shows:
-- - Table scanned
-- - Index used (or not)
-- - Rows examined
-- - Query cost
```

---

## Layer 3: Advanced Engineering (Senior)

### Internal Implementation Details

#### How SELECT Works Internally

```
1. PARSING: SQL text → Parse tree → Validate syntax
2. OPTIMIZATION: Query optimizer evaluates multiple execution plans
3. EXECUTION: Chosen plan executed by storage engine
4. RESULT: Data retrieved and returned
```

#### Index Internals (B-Tree)

```
Without Index:
Table scan: O(n) - Must check every row

With B-Tree Index:
Lookup: O(log n) - Binary search through tree structure

        [50]
       /    \
    [25]    [75]
    /  \    /  \
  [10][35][60][90]
```

**Index Trade-offs:**
- **Pros:** Faster SELECT, WHERE, ORDER BY
- **Cons:** Slower INSERT/UPDATE/DELETE, storage overhead

### Query Optimizer Behavior

```sql
-- Example: Optimizer chooses different plans based on data distribution

-- Plan A: Index scan (if user_id is selective)
SELECT * FROM orders WHERE user_id = 123;

-- Plan B: Full table scan (if status has few distinct values)
SELECT * FROM orders WHERE order_status = 'pending';

-- Plan C: Index merge
SELECT * FROM orders 
WHERE user_id = 123 AND order_status = 'pending';
```

### Transaction Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read | Performance |
|-------|------------|---------------------|--------------|-------------|
| **READ UNCOMMITTED** | ✅ Yes | ✅ Yes | ✅ Yes | Fastest |
| **READ COMMITTED** | ❌ No | ✅ Yes | ✅ Yes | Default (most RDBMS) |
| **REPEATABLE READ** | ❌ No | ❌ No | ✅ Yes | MySQL default |
| **SERIALIZABLE** | ❌ No | ❌ No | ❌ No | Slowest |

```sql
-- Example: Race condition without proper isolation
-- Session 1
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 1; -- balance = 1000

-- Session 2 (concurrent)
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
COMMIT;

-- Session 1 continues
UPDATE accounts SET balance = balance + 500 WHERE account_id = 1;
COMMIT; -- Lost update! Should be 1400, but will be 1500
```

**Solution:** Use `SELECT ... FOR UPDATE` for pessimistic locking:

```sql
START TRANSACTION;
SELECT balance FROM accounts WHERE account_id = 1 FOR UPDATE; -- Row locked
-- Other transactions wait here
UPDATE accounts SET balance = balance + 500 WHERE account_id = 1;
COMMIT; -- Lock released
```

### Deadlock Scenarios

```sql
-- Transaction 1
START TRANSACTION;
UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
-- Waiting to lock account_id = 2
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- Transaction 2 (concurrent)
START TRANSACTION;
UPDATE accounts SET balance = balance - 50 WHERE account_id = 2;
-- Waiting to lock account_id = 1 → DEADLOCK!
UPDATE accounts SET balance = balance + 50 WHERE account_id = 1;
```

**Detection:** Database detects cycle and rolls back one transaction.

**Prevention:**
- Always lock resources in same order
- Keep transactions short
- Use timeout settings

### Connection Pooling Impact

```java
// Without pooling: 100ms per connection establishment
for (int i = 0; i < 1000; i++) {
    Connection conn = DriverManager.getConnection(url); // Expensive!
    // Execute query
    conn.close();
}

// With pooling: <1ms to get connection from pool
HikariConfig config = new HikariConfig();
config.setMaximumPoolSize(20);
config.setMinimumIdle(5);
config.setConnectionTimeout(30000);
HikariDataSource ds = new HikariDataSource(config);

Connection conn = ds.getConnection(); // Reuses existing connection
```

---

## Layer 4: Architect Thinking

### System Design Implications

#### Scaling Strategy: Read vs Write

```
Scenario: E-commerce platform
- 95% reads (product browsing)
- 5% writes (orders, updates)

Architecture:
┌─────────────┐
│ Application │
└──────┬──────┘
       │
       ├──────────────────────────┐
       │                          │
   ┌───▼────┐               ┌────▼─────┐
   │ Master │──replicate──> │ Read     │
   │ (Write)│               │ Replica  │
   └────────┘               └──────────┘
                                  │
                            ┌─────▼──────┐
                            │ Read       │
                            │ Replica 2  │
                            └────────────┘
```

**Trade-offs:**
- **Replication Lag:** Slaves might be 100-500ms behind
- **Eventual Consistency:** User updates profile → can't see it immediately on slave
- **Failover Complexity:** Master dies → promote slave → reconfigure app

#### What Breaks at Scale?

| Scale | Problem | Solution |
|-------|---------|----------|
| **1K users** | Single database handles fine | Standard RDBMS with indexes |
| **100K users** | Read pressure increases | Add read replicas |
| **1M users** | Write contention, single DB bottleneck | Shard by user_id or region |
| **10M+ users** | Complex queries slow, join overhead | Denormalize, use caching (Redis), CQRS pattern |

#### Database Sharding Example

```sql
-- Horizontal sharding by user_id
-- Shard 1: user_id % 4 = 0
-- Shard 2: user_id % 4 = 1
-- Shard 3: user_id % 4 = 2
-- Shard 4: user_id % 4 = 3

-- Application logic determines shard
int shard = user_id % 4;
Connection conn = getShardConnection(shard);
```

**Challenges:**
- Cross-shard joins impossible
- Re-sharding requires downtime or complex migration
- Distributed transactions difficult

#### Database Migration Strategies (Architect Level)

In a professional Java/Spring Boot environment, manual SQL execution is a major anti-pattern. You must use automated migration tools.

| Tool | Approach | Best For |
|------|----------|----------|
| **Flyway** | SQL-based migrations (`V1__initial.sql`) | Developers who prefer raw SQL control |
| **Liquibase** | Logical migrations (XML, YAML, JSON, SQL) | Complex enterprise environments with multiple DB types |

**The "Expand-Contract" Pattern for Zero-Downtime:**
1. **Expand**: Add new columns/tables (backward compatible).
2. **Migrate**: Copy data and update application logic to write to both.
3. **Contract**: Remove old columns/tables (clean up).

**Architect Tip:** Never perform destructive changes (DROP/RENAME) in a single deployment. Always use a multi-phase approach to avoid breaking live traffic.

#### CAP Theorem in SQL Context

```
Traditional SQL (PostgreSQL, MySQL):
- Consistency: ✅ ACID guarantees
- Availability: ⚠️ Master fails → downtime until failover
- Partition Tolerance: ❌ Network split → can't function

NoSQL Alternative (Cassandra):
- Consistency: ⚠️ Eventual
- Availability: ✅ Multi-master, always writable
- Partition Tolerance: ✅ Designed for distributed systems
```

**When to use SQL:**
- Financial transactions (consistency critical)
- Complex queries with joins
- Strong schema enforcement
- ACID guarantees required

**When to consider NoSQL:**
- Massive write throughput (millions/sec)
- Geographic distribution
- Flexible schema
- Eventual consistency acceptable

### Cost Trade-offs

#### Storage Cost

```sql
-- Inefficient: VARCHAR(255) for everything
CREATE TABLE users (
    name VARCHAR(255),      -- Wastes space if name is "Bob"
    email VARCHAR(255),
    country VARCHAR(255)    -- Use ENUM or lookup table
);

-- Optimized
CREATE TABLE users (
    name VARCHAR(100),      -- Reasonable max
    email VARCHAR(100),
    country_id SMALLINT,    -- 2 bytes vs 255 bytes
    FOREIGN KEY (country_id) REFERENCES countries(id)
);
```

**Impact at scale:**
- 10M users × 200 bytes saved per row = **2GB saved**
- Faster queries (less I/O)
- Better cache hit rates

#### Query Cost (Cloud)

```
AWS RDS pricing influenced by:
- Instance size (determined by query workload)
- IOPS (Input/Output Operations Per Second)
- Storage (affected by data types and indexes)

Poor indexing → Slow queries → Larger instance needed → Higher cost
```

### Security Implications

#### SQL Injection

```java
// VULNERABLE
String query = "SELECT * FROM users WHERE username = '" + userInput + "'";
// If userInput = "admin' OR '1'='1"
// Result: SELECT * FROM users WHERE username = 'admin' OR '1'='1'
// Returns ALL users!!!

// SAFE: Prepared statements
PreparedStatement stmt = conn.prepareStatement(
    "SELECT * FROM users WHERE username = ?"
);
stmt.setString(1, userInput); // Automatically escaped
```

#### Least Privilege Principle

```sql
-- DON'T give application DB user full permissions
GRANT ALL PRIVILEGES ON *.* TO 'app_user'@'%'; -- DANGEROUS

-- DO: Grant minimum required permissions
GRANT SELECT, INSERT, UPDATE ON app_db.* TO 'app_user'@'app_server_ip';
REVOKE DELETE ON app_db.users FROM 'app_user'@'app_server_ip';
```

### Observability & Debugging

#### Slow Query Log

```sql
-- Enable slow query logging (MySQL)
SET GLOBAL slow_query_log = 'ON';
SET GLOBAL long_query_time = 1; -- Log queries taking >1 second

-- Analyze logs
mysqldumpslow /var/log/mysql/slow-query.log

-- Common findings:
-- 1. Missing indexes on WHERE clauses
-- 2. Inefficient JOINs
-- 3. Full table scans
```

#### Monitoring Metrics (Production)

```
Key Metrics to Track:
1. Query latency (p50, p95, p99)
2. Connection pool utilization
3. Slow query count
4. Deadlock frequency
5. Replication lag
6. Disk I/O and CPU usage
7. Cache hit ratio
```

#### Connection Pool Tuning (HikariCP)

For high-throughput Java applications, the database connection pool is often the first bottleneck.

**Calculated Pool Size Formula:**
`connections = ((core_count * 2) + effective_spindle_count)`

**Key Configurations for Architects:**
- `maximumPoolSize`: Heavily depends on your DB's `max_connections`. Don't set too high (causes context switching).
- `minimumIdle`: Keep equal to `maximumPoolSize` in production to avoid "ramp-up" latency.
- `connectionTimeout`: Set low (e.g., 5-10s) to fail fast during DB saturation.
- `leakDetectionThreshold`: Set to slightly above your p99 query time (e.g., 2s) to catch unclosed connections.

**Observability Integration:**
Expose pool metrics via **Micrometer/Prometheus**. Monitor `hikaricp_connections_pending` — if this spikes, your pool is saturated!

---

## Historical Evolution

### Timeline

| Era | Technology | Key Features |
|-----|------------|--------------|
| **1970s** | SQL created at IBM | Relational model (E.F. Codd) |
| **1980s** | Oracle, MySQL born | Commercial RDBMS |
| **1990s** | PostgreSQL, SQL Server | ACID, transactions, stored procedures |
| **2000s** | MySQL dominates web | LAMP stack era |
| **2010s** | NoSQL movement | MongoDB, Cassandra challenge SQL |
| **2020s** | NewSQL era | CockroachDB, Spanner (distribute SQL) |

### Modern Best Practices (2024+)

✅ **Use ORM cautiously** - Great for simple CRUD, but understand generated SQL  
✅ **Connection pooling mandatory** - HikariCP (Java), pgBouncer (Postgres)  
✅ **Prepared statements always** - Security and performance  
✅ **Read replicas for scaling reads** - Master-slave architecture  
✅ **Monitoring and alerting** - Prometheus + Grafana  
✅ **Database migrations** - Flyway or Liquibase for version control  

❌ **Avoid:** ORM for complex queries (write raw SQL)  
❌ **Avoid:** Storing large BLOBs in database (use S3/cloud storage)  
❌ **Avoid:** Lack of indexes on foreign keys  

---

## Performance & Scalability

### Benchmarking Strategy

```sql
-- Use EXPLAIN ANALYZE (PostgreSQL) for actual execution metrics
EXPLAIN ANALYZE
SELECT u.username, COUNT(o.order_id) as order_count
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY u.user_id
HAVING COUNT(o.order_id) > 5;

-- Output shows:
-- - Actual time: 123.456 ms
-- - Rows: 10,000
-- - Planning time: 0.234 ms
-- - Index usage: idx_orders_user_id
```

### Optimization Techniques

#### 1. Index Optimization

```sql
-- BEFORE: Slow query (table scan)
SELECT * FROM orders WHERE created_at > '2024-01-01' AND user_id = 123;
-- Execution time: 5000ms (500K orders scanned)

-- ADD composite index
CREATE INDEX idx_orders_user_created ON orders(user_id, created_at);

-- AFTER: Fast query (index scan)
-- Execution time: 5ms
```

#### 2. Query Rewriting

```sql
-- INEFFICIENT: Subquery in SELECT
SELECT 
    u.username,
    (SELECT COUNT(*) FROM orders WHERE user_id = u.user_id) as order_count
FROM users u;
-- Executes subquery for EACH user

-- EFFICIENT: Use JOIN
SELECT 
    u.username,
    COUNT(o.order_id) as order_count
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY u.user_id;
```

#### 3. Pagination Done Right

```sql
-- BAD: OFFSET becomes slow with large offsets
SELECT * FROM products ORDER BY product_id LIMIT 50 OFFSET 1000000;
-- Database still reads 1M rows, then skips them

-- GOOD: Keyset pagination
SELECT * FROM products 
WHERE product_id > 1000050  -- last_seen_id from previous page
ORDER BY product_id 
LIMIT 50;
```

---

## Security Considerations

### Input Validation

```java
// Layer 1: Application validation
if (!email.matches("^[A-Za-z0-9+_.-]+@(.+)$")) {
    throw new ValidationException("Invalid email");
}

// Layer 2: Database constraint
ALTER TABLE users ADD CONSTRAINT email_format 
CHECK (email LIKE '%_@__%.__%');

// Layer 3: Prepared statement (SQL injection prevention)
PreparedStatement stmt = conn.prepareStatement(
    "INSERT INTO users (email) VALUES (?)"
);
stmt.setString(1, email);
```

### Encryption

```sql
-- Encrypt sensitive fields at rest
CREATE TABLE credit_cards (
    card_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    encrypted_card_number VARBINARY(256), -- Encrypted with AES-256
    card_hash VARCHAR(64), -- For lookups, never decrypt
    created_at TIMESTAMP
);

-- Application handles encryption
byte[] encrypted = AES.encrypt(cardNumber, encryptionKey);
stmt.setBytes(1, encrypted);
```

---

## Common Mistakes & Anti-Patterns

### Anti-Pattern 1: God Table

```sql
-- BAD: Everything in one table
CREATE TABLE everything (
    id BIGINT,
    user_name VARCHAR(100),
    user_email VARCHAR(100),
    order_id BIGINT,
    product_name VARCHAR(200),
    product_price DECIMAL(10,2),
    payment_method VARCHAR(50)
    -- 50 more columns...
);

-- GOOD: Normalized design
CREATE TABLE users (...);
CREATE TABLE orders (...);
CREATE TABLE products (...);
CREATE TABLE payments (...);
```

### Anti-Pattern 2: Magic Values

```sql
-- BAD: Hard-coded status values
SELECT * FROM orders WHERE status = 1; -- What is 1?

-- GOOD: Use ENUM or constants
CREATE TABLE orders (
    status ENUM('pending', 'processing', 'shipped', 'delivered', 'cancelled')
);

SELECT * FROM orders WHERE status = 'delivered'; -- Clear intent
```

### Anti-Pattern 3: Premature Optimization

```sql
-- DON'T: Add 20 indexes "just in case"
-- Each index:
-- - Slows INSERT/UPDATE/DELETE
-- - Uses disk space
-- - Can confuse query optimizer

-- DO: Add indexes based on actual query patterns
-- Use slow query log to identify bottlenecks first
```

---

## Production Examples

### Example 1: User Session Management

```sql
-- Basic (BAD for production)
CREATE TABLE sessions (
    session_id VARCHAR(64) PRIMARY KEY,
    user_id BIGINT,
    created_at TIMESTAMP,
    expires_at TIMESTAMP
);

-- Production-grade (GOOD)
CREATE TABLE sessions (
    session_id VARCHAR(64) PRIMARY KEY,
    user_id BIGINT NOT NULL,
    user_agent VARCHAR(255),
    ip_address VARCHAR(45),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_activity TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    expires_at TIMESTAMP NOT NULL,
    is_revoked BOOLEAN DEFAULT FALSE,
    
    INDEX idx_user_active (user_id, is_revoked, expires_at),
    INDEX idx_cleanup (expires_at, is_revoked),
    
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
) ENGINE=InnoDB 
  ROW_FORMAT=COMPRESSED
  PARTITION BY RANGE (UNIX_TIMESTAMP(created_at)) (
      PARTITION p_2024_01 VALUES LESS THAN (UNIX_TIMESTAMP('2024-02-01')),
      PARTITION p_2024_02 VALUES LESS THAN (UNIX_TIMESTAMP('2024-03-01'))
      -- Auto-cleanup old partitions
  );
```

### Example 2: Audit Logging

```sql
-- Capture all changes for compliance
CREATE TABLE audit_log (
    log_id BIGINT PRIMARY KEY AUTO_INCREMENT,
    table_name VARCHAR(64) NOT NULL,
    operation ENUM('INSERT', 'UPDATE', 'DELETE') NOT NULL,
    record_id BIGINT NOT NULL,
    old_values JSON,
    new_values JSON,
    changed_by BIGINT,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_table_record (table_name, record_id),
    INDEX idx_changed_at (changed_at),
    INDEX idx_user (changed_by)
) PARTITION BY RANGE (YEAR(changed_at));

-- Trigger to auto-populate
CREATE TRIGGER users_audit_update
AFTER UPDATE ON users
FOR EACH ROW
INSERT INTO audit_log (table_name, operation, record_id, old_values, new_values, changed_by)
VALUES ('users', 'UPDATE', NEW.user_id, 
        JSON_OBJECT('email', OLD.email, 'name', OLD.name),
        JSON_OBJECT('email', NEW.email, 'name', NEW.name),
        @current_user_id);
```

---

## Interview Questions

### Beginner Level

**Q1: What is the difference between DELETE and TRUNCATE?**

| Feature | DELETE | TRUNCATE |
|---------|--------|----------|
| Type | DML | DDL |
| WHERE clause | ✅ Supported | ❌ Not supported |
| Rollback | ✅ Can rollback | ❌ Cannot rollback (in most RDBMS) |
| Triggers | ✅ Fires DELETE triggers | ❌ No triggers |
| Speed | Slower (row-by-row) | Faster (drops and recreates table) |

**Q2: Explain PRIMARY KEY vs UNIQUE constraint.**

- **PRIMARY KEY:** NOT NULL + UNIQUE, only one per table, used as default relationship target
- **UNIQUE:** Can be NULL (in most RDBMS), multiple allowed per table

### Intermediate Level

**Q3: What causes a full table scan? How do you avoid it?**

**Causes:**
- No WHERE clause
- WHERE column not indexed
- Using functions on indexed column: `WHERE YEAR(created_at) = 2024`
- Type mismatch: `WHERE user_id = '123'` (string vs int)

**Solutions:**
- Add appropriate indexes
- Rewrite queries: `WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01'`
- Ensure type matches

**Q4: Explain the N+1 query problem and solutions.**

**Problem:**
```java
List<Order> orders = orderRepository.findAll(); // 1 query
for (Order order : orders) {
    User user = userRepository.findById(order.getUserId()); // N queries
}
```

**Solutions:**
- Use JOIN fetch in query
- Use batch loading
- Use ORM eager loading strategically

### Senior Level

**Q5: Design a database schema for a Twitter-like social network. Consider:**
- User following relationships (bidirectional)
- Posts/tweets
- Scalability to 100M users
- Fast feed generation

**Answer highlights:**
- User table with indexes on username
- Following table: (follower_id, followee_id) composite primary key
- Posts table: partitioned by user_id or time
- Feed generation: fan-out on write vs fan-out on read trade-off
- Caching layer essential (Redis)
- Denormalization for read-heavy operations

**Q6: Explain how you'd debug a query that's slow in production but fast in staging.**

**Investigation steps:**
1. Check data volume difference (1K rows vs 10M rows)
2. Compare EXPLAIN plans between environments
3. Check index statistics (ANALYZE TABLE)
4. Review concurrent load (production has 1000 concurrent queries)
5. Examine cache hit rates
6. Check for lock contention
7. Review query pattern changes (new feature deployed?)

### Architect Level

**Q7: Your database has 500M rows and growing 10M/month. Queries are slowing down. Design a migration strategy.**

**Solution approach:**
1. **Immediate:** Add indexes, optimize queries, scale vertically
2. **Short-term:** Read replicas for read scalability
3. **Medium-term:** Archive old data (move to data warehouse)
4. **Long-term:** Shard database horizontally
   - Partition by user_id ranges
   - Use database middleware (Vitess, Citus)
   - Plan for cross-shard query limitations
5. **Alternative:** Migrate to distributed SQL (CockroachDB)

**Q8: Compare SAGA pattern vs 2PC for distributed transactions.**

| Aspect | SAGA | 2PC (Two-Phase Commit) |
|--------|------|------------------------|
| **Consistency** | Eventual | Strong |
| **Availability** | High (no global lock) | Lower (coordinator bottleneck) |
| **Complexity** | High (compensating transactions) | Medium (protocol built-in) |
| **Failure handling** | Rollback via compensation | Automatic rollback |
| **Use case** | Microservices, cross-service | Single database cluster |

---

## Cross-References

### Related Documentation

- **[02-joins.md](./02-joins.md)** - Deep dive into JOIN algorithms and optimization
- **[03-normalization.md](./03-normalization.md)** - Database design theory and normal forms
- **[04-queries.md](./04-queries.md)** - Advanced query patterns and window functions
- **[Spring Boot Data](../springboot/spring-data.md)** - JPA, Hibernate, and Spring Data integration
- **[System Design](../system-design/database-design.md)** - Database architecture patterns
- **[Performance](../java/14-memory-management-and-jvm-tuning.md)** - JVM tuning for database connections

---

## Summary

### Learning Path

```
Beginner (0-2 years)
├─ Master basic CRUD operations
├─ Understand tables, keys, constraints
└─ Learn simple SELECT queries

Intermediate (2-5 years)
├─ Use JOINs effectively
├─ Understand transactions and isolation
├─ Optimize with indexes
└─ Avoid common anti-patterns

Senior (5-8 years)
├─ Deep dive into query optimizer
├─ Handle deadlocks and race conditions
├─ Design schema for scale
└─ Debug production performance issues

Architect (8-10 years)
├─ Design distributed database systems
├─ Choose SQL vs NoSQL strategically
├─ Plan sharding and replication
├─ Optimize for cost and scale
└─ Lead database migration projects
```

---

**Next:** [02-joins.md →](./02-joins.md) Deep dive into JOIN types, algorithms, and optimization strategies

**Author:** Nehru Usare  
**Contributing Architects:** Welcome to contribute via pull requests  
**Version:** 1.0 | February 2026
