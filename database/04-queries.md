# 🚀 Advanced SQL Queries - Layered Engineering Guide

> **Document Level:** Beginner → Intermediate → Senior → Architect  
> **Last Updated:** February 2026 | **Prerequisites:** [01-sql-basics.md](./01-sql-basics.md), [02-joins.md](./02-joins.md)

---

## 📋 Table of Contents

1. [Layer 1: Simple Understanding (Beginner)](#layer-1-simple-understanding)
2. [Layer 2: Practical Engineering (Intermediate)](#layer-2-practical-engineering)
3. [Layer 3: Advanced Engineering (Senior)](#layer-3-advanced-engineering)
4. [Layer 4: Architect Thinking](#layer-4-architect-thinking)
5. [Window Functions Deep Dive](#window-functions-deep-dive)
6. [Performance Optimization](#performance-optimization)
7. [Common Mistakes](#common-mistakes)
8. [Production Examples](#production-examples)
9. [Interview Questions](#interview-questions)

---

## Layer 1: Simple Understanding (Beginner)

### What are Advanced Queries?

Beyond basic SELECT, INSERT, UPDATE, DELETE, SQL provides powerful features for:
- **Aggregating data** (SUM, COUNT, AVG)
- **Grouping results** (GROUP BY)
- **Filtering groups** (HAVING)
- **Subqueries** (queries within queries)
- **Set operations** (UNION, INTERSECT)

### Basic Aggregation

```sql
-- Count total orders
SELECT COUNT(*) AS total_orders FROM orders;

-- Sum of all order amounts
SELECT SUM(total_amount) AS revenue FROM orders;

-- Average order value
SELECT AVG(total_amount) AS avg_order_value FROM orders;

-- Find min and max
SELECT 
    MIN(total_amount) AS smallest_order,
    MAX(total_amount) AS largest_order
FROM orders;
```

### GROUP BY Basics

```sql
-- Count orders per user
SELECT 
    user_id,
    COUNT(*) AS order_count
FROM orders
GROUP BY user_id;

-- Total revenue per user
SELECT 
    user_id,
    SUM(total_amount) AS total_spent
FROM orders
GROUP BY user_id
ORDER BY total_spent DESC;
```

### HAVING (Filter After Grouping)

```sql
-- Users who spent more than $1000
SELECT 
    user_id,
    SUM(total_amount) AS total_spent
FROM orders
GROUP BY user_id
HAVING SUM(total_amount) > 1000;

-- WHERE vs HAVING
-- WHERE: filters rows BEFORE grouping
-- HAVING: filters groups AFTER grouping

SELECT 
    user_id,
    COUNT(*) AS order_count
FROM orders
WHERE order_date >= '2024-01-01'  -- Filter rows first
GROUP BY user_id
HAVING COUNT(*) > 5;  -- Then filter groups
```

### Simple Subqueries

```sql
-- Find users who have placed orders
SELECT name 
FROM users
WHERE user_id IN (SELECT DISTINCT user_id FROM orders);

-- Find products more expensive than average
SELECT product_name, price
FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

---

## Layer 2: Practical Engineering (Intermediate)

### Common Table Expressions (CTEs)

CTEs make complex queries readable by breaking them into named steps.

```sql
-- Instead of nested subqueries
SELECT *
FROM (
    SELECT user_id, COUNT(*) AS order_count
    FROM orders
    GROUP BY user_id
) AS user_orders
WHERE order_count > 10;

-- Use CTE (clearer!)
WITH user_orders AS (
    SELECT user_id, COUNT(*) AS order_count
    FROM orders
    GROUP BY user_id
)
SELECT *
FROM user_orders
WHERE order_count > 10;
```

#### Multiple CTEs

```sql
-- Calculate customer lifetime value
WITH 
order_totals AS (
    SELECT 
        user_id,
        SUM(total_amount) AS total_spent,
        COUNT(*) AS order_count
    FROM orders
    GROUP BY user_id
),
avg_metrics AS (
    SELECT 
        AVG(total_spent) AS avg_ltv,
        AVG(order_count) AS avg_orders
    FROM order_totals
)
SELECT 
    u.name,
    ot.total_spent,
    ot.order_count,
    CASE 
        WHEN ot.total_spent > am.avg_ltv THEN 'High Value'
        WHEN ot.total_spent > am.avg_ltv * 0.5 THEN 'Medium Value'
        ELSE 'Low Value'
    END AS customer_segment
FROM users u
JOIN order_totals ot ON u.user_id = ot.user_id
CROSS JOIN avg_metrics am;
```

### Window Functions Introduction

Calculate running totals, rankings, and comparisons **without grouping**.

```sql
-- Rank users by total spent
SELECT 
    user_id,
    SUM(total_amount) AS total_spent,
    RANK() OVER (ORDER BY SUM(total_amount) DESC) AS spending_rank
FROM orders
GROUP BY user_id;

-- Running total of daily sales
SELECT 
    order_date,
    SUM(total_amount) AS daily_sales,
    SUM(SUM(total_amount)) OVER (ORDER BY order_date) AS running_total
FROM orders
GROUP BY order_date
ORDER BY order_date;
```

### CASE Statements

```sql
-- Categorize orders by size
SELECT 
    order_id,
    total_amount,
    CASE 
        WHEN total_amount >= 1000 THEN 'Large'
        WHEN total_amount >= 100 THEN 'Medium'
        ELSE 'Small'
    END AS order_size
FROM orders;

-- Conditional aggregation (pivot-like)
SELECT 
    user_id,
    SUM(CASE WHEN order_date >= '2024-01-01' THEN total_amount ELSE 0 END) AS sales_2024,
    SUM(CASE WHEN order_date >= '2023-01-01' AND order_date < '2024-01-01' THEN total_amount ELSE 0 END) AS sales_2023
FROM orders
GROUP BY user_id;
```

### UNION vs UNION ALL

```sql
-- UNION: Combines results, removes duplicates
SELECT user_id FROM orders
UNION
SELECT user_id FROM reviews;

-- UNION ALL: Combines results, keeps duplicates (faster)
SELECT user_id FROM orders
UNION ALL
SELECT user_id FROM reviews;

-- Use UNION ALL when you know there are no duplicates
```

### Correlated Subqueries

```sql
-- Find users who spent more than their country's average
SELECT 
    u.name,
    u.country,
    (SELECT SUM(total_amount) FROM orders WHERE user_id = u.user_id) AS total_spent
FROM users u
WHERE (
    SELECT SUM(total_amount) FROM orders WHERE user_id = u.user_id
) > (
    SELECT AVG(total) FROM (
        SELECT SUM(total_amount) AS total
        FROM orders o2
        JOIN users u2 ON o2.user_id = u2.user_id
        WHERE u2.country = u.country
        GROUP BY u2.user_id
    ) AS country_avg
);
```

---

## Layer 3: Advanced Engineering (Senior)

### Window Functions Deep Dive

#### ROW_NUMBER, RANK, DENSE_RANK

```sql
-- Differences between ranking functions
SELECT 
    user_id,
    order_date,
    total_amount,
    ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY order_date) AS row_num,
    RANK() OVER (PARTITION BY user_id ORDER BY total_amount DESC) AS rank,
    DENSE_RANK() OVER (PARTITION BY user_id ORDER BY total_amount DESC) AS dense_rank
FROM orders;

/*
Results example:
user_id | amount | row_num | rank | dense_rank
1       | 1000   | 1       | 1    | 1
1       | 1000   | 2       | 1    | 1  (tie)
1       | 500    | 3       | 3    | 2  (rank skips 2, dense_rank doesn't)
*/
```

#### LAG and LEAD (Access Previous/Next Rows)

```sql
-- Calculate order-to-order time difference
SELECT 
    user_id,
    order_id,
    order_date,
    LAG(order_date) OVER (PARTITION BY user_id ORDER BY order_date) AS previous_order_date,
    order_date - LAG(order_date) OVER (PARTITION BY user_id ORDER BY order_date) AS days_since_last_order
FROM orders;

-- Month-over-month growth
SELECT 
    DATE_TRUNC('month', order_date) AS month,
    SUM(total_amount) AS revenue,
    LAG(SUM(total_amount)) OVER (ORDER BY DATE_TRUNC('month', order_date)) AS prev_month_revenue,
    (SUM(total_amount) - LAG(SUM(total_amount)) OVER (ORDER BY DATE_TRUNC('month', order_date))) 
        / LAG(SUM(total_amount)) OVER (ORDER BY DATE_TRUNC('month', order_date)) * 100 AS growth_pct
FROM orders
GROUP BY DATE_TRUNC('month', order_date);
```

#### NTILE (Divide into Buckets)

```sql
-- Divide customers into quartiles by spending
SELECT 
    user_id,
    SUM(total_amount) AS total_spent,
    NTILE(4) OVER (ORDER BY SUM(total_amount) DESC) AS spending_quartile
FROM orders
GROUP BY user_id;

-- Top 10% of spenders
WITH user_spending AS (
    SELECT 
        user_id,
        SUM(total_amount) AS total_spent,
        NTILE(10) OVER (ORDER BY SUM(total_amount) DESC) AS decile
    FROM orders
    GROUP BY user_id
)
SELECT * FROM user_spending WHERE decile = 1;
```

#### Frame Specification

```sql
-- Moving average (last 7 days)
SELECT 
    order_date,
    SUM(total_amount) AS daily_sales,
    AVG(SUM(total_amount)) OVER (
        ORDER BY order_date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7day
FROM orders
GROUP BY order_date;

-- Cumulative sum with explicit frame
SELECT 
    order_date,
    SUM(total_amount) AS daily_sales,
    SUM(SUM(total_amount)) OVER (
        ORDER BY order_date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_sales
FROM orders
GROUP BY order_date;
```

### Recursive CTEs

```sql
-- Employee hierarchy (org chart)
WITH RECURSIVE employee_hierarchy AS (
    -- Base case: Top-level managers
    SELECT 
        employee_id,
        name,
        manager_id,
        1 AS level,
        name AS path
    FROM employees
    WHERE manager_id IS NULL
    
    UNION ALL
    
    -- Recursive case: Add employees under current level
    SELECT 
        e.employee_id,
        e.name,
        e.manager_id,
        eh.level + 1,
        eh.path || ' > ' || e.name
    FROM employees e
    JOIN employee_hierarchy eh ON e.manager_id = eh.employee_id
)
SELECT * FROM employee_hierarchy ORDER BY level, path;

-- Category tree (nested categories)
WITH RECURSIVE category_tree AS (
    SELECT 
        category_id,
        category_name,
        parent_id,
        1 AS depth,
        category_name AS full_path
    FROM categories
    WHERE parent_id IS NULL
    
    UNION ALL
    
    SELECT 
        c.category_id,
        c.category_name,
        c.parent_id,
        ct.depth + 1,
        ct.full_path || ' / ' || c.category_name
    FROM categories c
    JOIN category_tree ct ON c.parent_id = ct.category_id
)
SELECT * FROM category_tree;
```

### Lateral Joins (PostgreSQL)

```sql
-- Get top 3 orders per user
SELECT 
    u.user_id,
    u.name,
    recent_orders.*
FROM users u
CROSS JOIN LATERAL (
    SELECT order_id, order_date, total_amount
    FROM orders
    WHERE user_id = u.user_id
    ORDER BY order_date DESC
    LIMIT 3
) AS recent_orders;

-- This is more efficient than window functions + filtering
```

### PIVOT Tables (SQL Server) / CROSSTAB (PostgreSQL)

```sql
-- Manual pivot with CASE
SELECT 
    product_id,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 1 THEN quantity ELSE 0 END) AS jan,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 2 THEN quantity ELSE 0 END) AS feb,
    SUM(CASE WHEN EXTRACT(MONTH FROM order_date) = 3 THEN quantity ELSE 0 END) AS mar
FROM order_items oi
JOIN orders o ON oi.order_id = o.order_id
WHERE EXTRACT(YEAR FROM order_date) = 2024
GROUP BY product_id;

-- PostgreSQL crosstab (requires tablefunc extension)
SELECT *
FROM crosstab(
    'SELECT product_id, month, SUM(quantity)
     FROM monthly_sales
     ORDER BY 1, 2'
) AS ct(product_id INT, jan INT, feb INT, mar INT);
```

### Query Optimization Techniques

#### Use EXISTS Instead of IN for Large Datasets

```sql
-- SLOW: IN with subquery
SELECT * FROM users
WHERE user_id IN (SELECT user_id FROM orders WHERE total_amount > 1000);

-- FAST: EXISTS (stops at first match)
SELECT * FROM users u
WHERE EXISTS (
    SELECT 1 FROM orders o 
    WHERE o.user_id = u.user_id 
    AND o.total_amount > 1000
);
```

#### Avoid SELECT DISTINCT When Possible

```sql
-- SLOW: DISTINCT requires sorting/hashing
SELECT DISTINCT user_id FROM orders;

-- FAST: Use GROUP BY (often optimized better)
SELECT user_id FROM orders GROUP BY user_id;

-- EVEN BETTER: If you just need to check existence
SELECT user_id FROM orders LIMIT 1;
```

#### Rewrite Subqueries as JOINs

```sql
-- SLOW: Correlated subquery
SELECT 
    u.name,
    (SELECT COUNT(*) FROM orders WHERE user_id = u.user_id) AS order_count
FROM users u;

-- FAST: JOIN with aggregation
SELECT 
    u.name,
    COUNT(o.order_id) AS order_count
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY u.user_id, u.name;
```

---

## Layer 4: Architect Thinking

### Query Performance at Scale

```sql
-- Scenario: 100M orders, 10M users

-- BAD: Full table scan + aggregation (60 seconds)
SELECT 
    u.name,
    COUNT(o.order_id),
    SUM(o.total_amount)
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY u.user_id, u.name;

-- BETTER: Indexed query (5 seconds)
CREATE INDEX idx_orders_user_amount ON orders(user_id, total_amount);
-- Same query, now uses index

-- BEST: Materialized view (50ms)
CREATE MATERIALIZED VIEW user_aggregates AS
SELECT 
    u.user_id,
    u.name,
    COUNT(o.order_id) AS order_count,
    COALESCE(SUM(o.total_amount), 0) AS total_spent
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY u.user_id, u.name;

CREATE INDEX idx_user_agg_id ON user_aggregates(user_id);

-- Query materialized view
SELECT * FROM user_aggregates WHERE user_id = 123;
```

### Partitioning Impact on Queries

```sql
-- Table partitioned by order_date (monthly)
CREATE TABLE orders_2024_01 PARTITION OF orders
FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

CREATE TABLE orders_2024_02 PARTITION OF orders
FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

-- Query automatically uses partition pruning
SELECT * FROM orders
WHERE order_date >= '2024-01-15' AND order_date < '2024-01-20';
-- Only scans orders_2024_01 partition

-- Benefit: 12x faster (scans 1/12th of data)
```

### Window Functions vs Self-Joins

```sql
-- Task: Find orders with higher amount than previous order

-- Old way: Self-join (O(n²) complexity, slow)
SELECT 
    o1.order_id,
    o1.total_amount,
    o2.total_amount AS prev_amount
FROM orders o1
LEFT JOIN orders o2 ON o1.user_id = o2.user_id 
    AND o2.order_date < o1.order_date
    AND NOT EXISTS (
        SELECT 1 FROM orders o3
        WHERE o3.user_id = o1.user_id
        AND o3.order_date < o1.order_date
        AND o3.order_date > o2.order_date
    );

-- Modern way: Window function (O(n log n), fast)
WITH order_comparison AS (
    SELECT 
        order_id,
        total_amount,
        LAG(total_amount) OVER (PARTITION BY user_id ORDER BY order_date) AS prev_amount
    FROM orders
)
SELECT * FROM order_comparison
WHERE total_amount > prev_amount;
```

### Query Planning for Distributed Databases

```sql
-- Single-node PostgreSQL
SELECT u.name, COUNT(o.order_id)
FROM users u
JOIN orders o ON u.user_id = o.user_id
GROUP BY u.user_id;
-- Query planner optimizes automatically

-- Distributed SQL (CockroachDB, Spanner)
-- Same query, but:
-- 1. Data may be on different nodes
-- 2. JOIN might require network shuffle
-- 3. GROUP BY might need multi-stage aggregation

-- Optimization: Co-locate data
CREATE TABLE orders (
    ...
) PARTITION BY HASH(user_id);
-- Now joins are local, no network transfer
```

#### Read-Write Splitting Architecture

Architects must decide how to route read queries to replicas to maximize performance.

**Application Layer Routing (Spring Boot / AbstractRoutingDataSource)**:
- **Pros**: Fine-grained control. Can route specific queries (e.g., reports) to specific replicas.
- **Cons**: Code complexity. Need to manage multiple data sources.
- **Spring Pattern**: Use `@Transactional(readOnly = true)` to hint the routing proxy.

**Infrastructure Layer Routing (ProxySQL / AWS Aurora Reader Endpoint)**:
- **Pros**: Transparent to application. High availability handled by the proxy.
- **Cons**: Less granular control. Potential "Read-after-Write" consistency issues (replication lag).

**Architect Decision**: Use Infrastructure Layer routing for general scalability, but use Application Layer routing for critical, delay-sensitive report generation.

### OLAP Query Patterns

```sql
-- Typical OLAP: Multi-dimensional analysis
SELECT 
    DATE_TRUNC('month', o.order_date) AS month,
    p.category,
    u.country,
    SUM(oi.quantity * oi.unit_price) AS revenue,
    COUNT(DISTINCT o.order_id) AS order_count,
    COUNT(DISTINCT o.user_id) AS customer_count
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
JOIN users u ON o.user_id = u.user_id
WHERE o.order_date >= '2024-01-01'
GROUP BY 
    DATE_TRUNC('month', o.order_date),
    p.category,
    u.country
WITH ROLLUP;  -- Adds subtotals and grand totals
```

### Columnar Storage Benefits

```
Row-oriented (OLTP):
[order_id=1, user_id=123, amount=100, date=2024-01-01]
[order_id=2, user_id=124, amount=200, date=2024-01-02]

Columnar (OLAP):
order_id: [1, 2, ...]
user_id:  [123, 124, ...]
amount:   [100, 200, ...]  ← Only read this column for SUM!
date:     [2024-01-01, 2024-01-02, ...]

Query: SELECT SUM(amount) FROM orders;
Columnar: Read only amount column (10x faster)
Row: Must read all columns
```

### Cost-Based Optimizer Hints

```sql
-- PostgreSQL: Set statistics target
ALTER TABLE orders ALTER COLUMN user_id SET STATISTICS 1000;
ANALYZE orders;
-- Better statistics → better query plans

-- MySQL: Use hints sparingly
SELECT /*+ INDEX(orders idx_user_date) */
    user_id, SUM(total_amount)
FROM orders
WHERE user_id = 123
GROUP BY user_id;

-- General rule: Trust optimizer, only hint when proven necessary
```

---

## Window Functions Deep Dive

### Comparison: All Window Functions

| Function | Purpose | Example Use Case |
|----------|---------|------------------|
| ROW_NUMBER() | Sequential numbering | Pagination, unique ranking |
| RANK() | Ranking with gaps | Leaderboards with ties |
| DENSE_RANK() | Ranking without gaps | Category rankings |
| NTILE(n) | Divide into n buckets | Quartiles, percentiles |
| LAG() | Access previous row | Calculating deltas |
| LEAD() | Access next row | Forecasting, trends |
| FIRST_VALUE() | First in window | Baseline comparisons |
| LAST_VALUE() | Last in window | Current vs final |
| NTH_VALUE() | Nth in window | Median, arbitrary position |
| SUM/AVG/MAX/MIN | Aggregate over window | Running totals, moving averages |

### Frame Specifications

```sql
-- Different frame types
SELECT 
    order_date,
    daily_sales,
    
    -- ROWS: Physical rows
    SUM(daily_sales) OVER (
        ORDER BY order_date
        ROWS BETWEEN 2 PRECEDING AND 2 FOLLOWING
    ) AS rows_sum_5day,
    
    -- RANGE: Logical range (all rows with same value)
    SUM(daily_sales) OVER (
        ORDER BY order_date
        RANGE BETWEEN INTERVAL '2 days' PRECEDING AND INTERVAL '2 days' FOLLOWING
    ) AS range_sum_5day,
    
    -- GROUPS: Groups of rows (with PARTITION)
    SUM(daily_sales) OVER (
        PARTITION BY EXTRACT(MONTH FROM order_date)
        ORDER BY order_date
    ) AS month_cumulative
FROM daily_sales_summary;
```

### Real-World Window Function Patterns

```sql
-- Cohort analysis: Retention by signup month
WITH user_activity AS (
    SELECT 
        u.user_id,
        DATE_TRUNC('month', u.signup_date) AS cohort_month,
        DATE_TRUNC('month', o.order_date) AS activity_month,
        COUNT(DISTINCT o.order_id) AS orders
    FROM users u
    LEFT JOIN orders o ON u.user_id = o.user_id
    GROUP BY u.user_id, cohort_month, activity_month
)
SELECT 
    cohort_month,
    activity_month,
    COUNT(DISTINCT user_id) AS active_users,
    FIRST_VALUE(COUNT(DISTINCT user_id)) OVER (
        PARTITION BY cohort_month 
        ORDER BY activity_month
    ) AS cohort_size,
    COUNT(DISTINCT user_id) * 100.0 / FIRST_VALUE(COUNT(DISTINCT user_id)) OVER (
        PARTITION BY cohort_month 
        ORDER BY activity_month
    ) AS retention_pct
FROM user_activity
GROUP BY cohort_month, activity_month
ORDER BY cohort_month, activity_month;
```

---

## Performance Optimization

### Index Usage Analysis

```sql
-- Check if query uses indexes
EXPLAIN (ANALYZE, BUFFERS)
SELECT * FROM orders WHERE user_id = 123;

-- Look for:
-- ✅ "Index Scan" or "Index Only Scan" = Good
-- ❌ "Seq Scan" = Bad (full table scan)

-- Create appropriate indexes
CREATE INDEX idx_orders_user ON orders(user_id);

-- Covering index for specific query
CREATE INDEX idx_orders_user_date_amount 
ON orders(user_id, order_date, total_amount);

-- Now this query uses index-only scan (fastest!)
SELECT user_id, order_date, total_amount
FROM orders
WHERE user_id = 123;
```

### Query Result Caching

```sql
-- Application-level caching (Redis)
Cache Key: "user:123:order_count"
Cache Value: 42
TTL: 300 seconds (5 minutes)

-- Database query cache (MySQL - deprecated in 8.0)
-- Modern approach: Materialized views

-- Incremental materialized views (PostgreSQL)
CREATE MATERIALIZED VIEW user_stats AS
SELECT 
    user_id,
    COUNT(*) AS order_count,
    SUM(total_amount) AS total_spent
FROM orders
GROUP BY user_id;

-- Refresh incrementally (only new data)
REFRESH MATERIALIZED VIEW CONCURRENTLY user_stats;
```

#### Advanced Caching Patterns (Redis)

Standard DB query caching is rarely sufficient for Architect-level scale.

**1. Cache-Aside Pattern**:
- Application checks Redis first.
- If miss, query DB -> populate Redis.
- **Architect Tip**: Use a **Cache TTL** and **Randomized Expiry** to avoid "Cache Stampede" (where many users hit the DB at once when a hot key expires).

**2. Distributed Locking (Redlock)**:
- If a query is extremely expensive (e.g., 5 seconds), don't let 100 threads run it concurrently on cache miss.
- Use **Redis distributed locks** to ensure only one thread re-computes the cache while others wait.

**3. In-Memory Computing (Redis Sets/Hashes)**:
- Instead of JOINing in SQL, sometimes it's faster to store frequently accessed lookup data in **Redis Hashes**.
- **Performance**: Sub-millisecond response time vs 20-50ms SQL query.
```

### Batch Processing

```sql
-- BAD: Row-by-row processing
FOR each order IN orders:
    UPDATE inventory SET quantity = quantity - order.quantity
    WHERE product_id = order.product_id;
-- 10,000 orders = 10,000 UPDATE statements

-- GOOD: Batch update
UPDATE inventory i
SET quantity = quantity - order_totals.total_qty
FROM (
    SELECT 
        product_id,
        SUM(quantity) AS total_qty
    FROM order_items
    WHERE order_id IN (/* batch of order IDs */)
    GROUP BY product_id
) AS order_totals
WHERE i.product_id = order_totals.product_id;
-- 10,000 orders = 1 UPDATE statement
```

---

## Common Mistakes

### Mistake 1: N+1 Query Problem

```sql
-- Application code doing:
List<Order> orders = SELECT * FROM orders LIMIT 100;
FOR each order:
    User user = SELECT * FROM users WHERE user_id = order.user_id;
-- 101 queries!

-- Fix: Use JOIN or IN clause
SELECT o.*, u.*
FROM orders o
JOIN users u ON o.user_id = u.user_id
LIMIT 100;
-- 1 query!
```

### Mistake 2: Using Functions on Indexed Columns

```sql
-- BAD: Function on indexed column prevents index use
SELECT * FROM orders
WHERE YEAR(order_date) = 2024;
-- Full table scan!

-- GOOD: Rewrite without function
SELECT * FROM orders
WHERE order_date >= '2024-01-01' AND order_date < '2025-01-01';
-- Uses index!
```

### Mistake 3: Implicit Type Conversion

```sql
-- user_id is INT, but querying with string
SELECT * FROM orders WHERE user_id = '123';
-- Causes implicit conversion, doesn't use index efficiently

-- Fix: Use correct type
SELECT * FROM orders WHERE user_id = 123;
```

---

## Production Examples

### Example 1: Real-Time Analytics Dashboard

```sql
-- Dashboard showing last 30 days metrics
WITH daily_metrics AS (
    SELECT 
        DATE(order_date) AS date,
        COUNT(DISTINCT order_id) AS orders,
        COUNT(DISTINCT user_id) AS customers,
        SUM(total_amount) AS revenue
    FROM orders
    WHERE order_date >= CURRENT_DATE - INTERVAL '30 days'
    GROUP BY DATE(order_date)
)
SELECT 
    date,
    orders,
    customers,
    revenue,
    AVG(revenue) OVER (
        ORDER BY date
        ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS moving_avg_7day,
    revenue - LAG(revenue) OVER (ORDER BY date) AS day_over_day_change,
    (revenue - LAG(revenue) OVER (ORDER BY date)) * 100.0 / 
        NULLIF(LAG(revenue) OVER (ORDER BY date), 0) AS day_over_day_pct
FROM daily_metrics
ORDER BY date DESC;
```

### Example 2: Fraud Detection Query

```sql
-- Detect suspicious patterns
WITH user_behavior AS (
    SELECT 
        user_id,
        order_id,
        order_date,
        total_amount,
        LAG(order_date) OVER (PARTITION BY user_id ORDER BY order_date) AS prev_order_date,
        LAG(total_amount) OVER (PARTITION BY user_id ORDER BY order_date) AS prev_amount,
        AVG(total_amount) OVER (PARTITION BY user_id) AS avg_amount
    FROM orders
    WHERE order_date >= CURRENT_DATE - INTERVAL '90 days'
)
SELECT 
    user_id,
    order_id,
    total_amount,
    avg_amount
FROM user_behavior
WHERE 
    -- Large spike in spending
    total_amount > avg_amount * 5
    -- Multiple orders in short time
    OR (order_date - prev_order_date) < INTERVAL '5 minutes';
```

---

## Interview Questions

### Beginner Level

**Q1: What's the difference between WHERE and HAVING?**

- **WHERE:** Filters rows BEFORE aggregation
- **HAVING:** Filters groups AFTER aggregation

```sql
SELECT user_id, COUNT(*) AS orders
FROM orders
WHERE order_date >= '2024-01-01'  -- Filter rows
GROUP BY user_id
HAVING COUNT(*) > 5;  -- Filter groups
```

**Q2: When would you use a CTE instead of a subquery?**

Use CTEs when:
- Query is complex (improves readability)
- Need to reference result multiple times
- Building recursive queries

### Intermediate Level

**Q3: Write a query to find the 2nd highest salary.**

```sql
-- Using window function
WITH ranked_salaries AS (
    SELECT 
        salary,
        DENSE_RANK() OVER (ORDER BY salary DESC) AS rank
    FROM employees
)
SELECT salary FROM ranked_salaries WHERE rank = 2;

-- Using subquery
SELECT MAX(salary) FROM employees
WHERE salary < (SELECT MAX(salary) FROM employees);
```

**Q4: Calculate running total of sales.**

```sql
SELECT 
    order_date,
    daily_sales,
    SUM(daily_sales) OVER (ORDER BY order_date) AS running_total
FROM (
    SELECT order_date, SUM(total_amount) AS daily_sales
    FROM orders
    GROUP BY order_date
) AS daily;
```

### Senior Level

**Q5: Find gaps in sequential order_id.**

```sql
-- Find missing order IDs
WITH number_series AS (
    SELECT generate_series(
        (SELECT MIN(order_id) FROM orders),
        (SELECT MAX(order_id) FROM orders)
    ) AS id
)
SELECT id AS missing_order_id
FROM number_series
WHERE id NOT IN (SELECT order_id FROM orders);
```

**Q6: Optimize this slow query:**

```sql
-- Original (slow)
SELECT *
FROM orders o
WHERE EXISTS (
    SELECT 1 FROM order_items oi
    WHERE oi.order_id = o.order_id
    AND oi.product_id IN (
        SELECT product_id FROM products WHERE category = 'Electronics'
    )
);

-- Optimized
SELECT DISTINCT o.*
FROM orders o
JOIN order_items oi ON o.order_id = oi.order_id
JOIN products p ON oi.product_id = p.product_id
WHERE p.category = 'Electronics';

-- With index:
CREATE INDEX idx_products_category ON products(category);
```

### Architect Level

**Q7: Design a query strategy for a BI dashboard with 50+ charts, each requiring complex aggregations on 100M+ rows. Response time SLA is <2 seconds.**

**Solution:**

```
1. Materialized Views Strategy:
   - Create daily batch job to refresh views
   - Pre-aggregate at different granularities:
     * Hourly aggregates (detailed)
     * Daily aggregates (medium)
     * Monthly aggregates (summary)

2. Query Router:
   - Date range < 7 days: Query hourly view
   - Date range < 90 days: Query daily view
   - Date range >= 90 days: Query monthly view

3. Incremental Updates:
   - Use CONCURRENTLY option for zero downtime
   - Only refresh partitions that changed

4. Caching Layer:
   - Redis for dashboard query results (TTL: 5 minutes)
   - Warm cache during off-peak hours

5. Query Optimization:
   - Columnar storage (Parquet/ORC) for analytics
   - Partition by date
   - Cluster by frequently grouped columns

Result: 98% of queries under 200ms
```

---

## Cross-References

- **[01-sql-basics.md](./01-sql-basics.md)** - SQL fundamentals
- **[02-joins.md](./02-joins.md)** - JOIN optimization strategies
- **[03-normalization.md](./03-normalization.md)** - When to denormalize for query performance
- **[Spring Data](../springboot/spring-data.md)** - JPA query optimization
- **[Performance](../java/15-performance-and-optimization-patterns.md)** - Application-level query optimization

---

## Summary

### Key Takeaways

```
Beginner:
✓ Master GROUP BY and aggregations
✓ Understand WHERE vs HAVING
✓ Use simple subqueries

Intermediate:
✓ Write readable queries with CTEs
✓ Use CASE for conditional logic
✓ Apply basic window functions (ROW_NUMBER, RANK)

Senior:
✓ Master all window functions (LAG, LEAD, NTILE)
✓ Write recursive queries for hierarchical data
✓ Optimize queries with EXPLAIN
✓ Know when to use indexes vs materialized views

Architect:
✓ Design query strategies for billions of rows
✓ Choose OLTP vs OLAP query patterns
✓ Implement partition pruning
✓ Balance real-time queries vs batch processing
✓ Design incremental materialized view strategies
```

---

**Previous:** [← 03-normalization.md](./03-normalization.md) - Database design and normal forms  
**Next:** [README.md →](./README.md) - Database section overview

**Author:** Nehru Usare  
**Version:** 1.0 | February 2026
