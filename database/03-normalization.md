# 🎯 Database Normalization - Layered Engineering Guide

> **Document Level:** Beginner → Intermediate → Senior → Architect  
> **Last Updated:** February 2026 | **Prerequisites:** [01-sql-basics.md](./01-sql-basics.md), [02-joins.md](./02-joins.md)

---

## 📋 Table of Contents

1. [Layer 1: Simple Understanding (Beginner)](#layer-1-simple-understanding)
2. [Layer 2: Practical Engineering (Intermediate)](#layer-2-practical-engineering)
3. [Layer 3: Advanced Engineering (Senior)](#layer-3-advanced-engineering)
4. [Layer 4: Architect Thinking](#layer-4-architect-thinking)
5. [Denormalization Strategies](#denormalization-strategies)
6. [Performance Trade-offs](#performance-trade-offs)
7. [Common Mistakes](#common-mistakes)
8. [Production Examples](#production-examples)
9. [Interview Questions](#interview-questions)

---

## Layer 1: Simple Understanding (Beginner)

### What is Normalization?

**Normalization** is the process of organizing database tables to:
- **Reduce data redundancy** (no duplicate information)
- **Improve data integrity** (consistency when updating)
- **Make database easier to maintain**

### Why Does Normalization Exist?

**Problem: Redundant Data**

```
BAD: Unnormalized table
┌──────────┬───────────┬──────────────┬──────────┬───────────┐
│ order_id │ user_name │ user_email   │ product  │ price     │
├──────────┼───────────┼──────────────┼──────────┼───────────┤
│ 1        │ Alice     │ alice@ex.com │ Laptop   │ 1000      │
│ 2        │ Bob       │ bob@ex.com   │ Phone    │ 500       │
│ 3        │ Alice     │ alice@ex.com │ Mouse    │ 20        │
└──────────┴───────────┴──────────────┴──────────┴───────────┘
                        ↑ Alice's data duplicated!
```

**Issues:**
1. **Storage waste** - Alice's name/email stored multiple times
2. **Update anomaly** - If Alice changes email, must update multiple rows
3. **Deletion anomaly** - Delete order 1, lose Alice's data entirely
4. **Insertion anomaly** - Can't add new user without an order

**Solution: Normalized Design**

```
users table
┌─────────┬───────────┬──────────────┐
│ user_id │ name      │ email        │
├─────────┼───────────┼──────────────┤
│ 1       │ Alice     │ alice@ex.com │
│ 2       │ Bob       │ bob@ex.com   │
└─────────┴───────────┴──────────────┘

orders table
┌──────────┬─────────┬──────────┬───────┐
│ order_id │ user_id │ product  │ price │
├──────────┼─────────┼──────────┼───────┤
│ 1        │ 1       │ Laptop   │ 1000  │
│ 2        │ 2       │ Phone    │ 500   │
│ 3        │ 1       │ Mouse    │ 20    │
└──────────┴─────────┴──────────┴───────┘
```

Now Alice's data exists once. Update her email in one place!

### The Normal Forms (Simplified)

Think of normalization as **levels of organization**:

| Form | Simple Rule | Example |
|------|-------------|---------|
| **1NF** | No repeating groups, atomic values | Each cell has single value |
| **2NF** | No partial dependencies | All columns depend on full primary key |
| **3NF** | No transitive dependencies | Non-key columns don't depend on each other |
| **BCNF** | Every determinant is a candidate key | Stricter version of 3NF |

**Most applications normalize to 3NF**, which is sufficient for 95% of use cases.

### First Normal Form (1NF)

**Rule:** Each column must contain **atomic (indivisible) values**. No arrays or lists.

```sql
-- WRONG: Violates 1NF (multiple phones in one column)
CREATE TABLE users_bad (
    user_id INT,
    name VARCHAR(100),
    phone_numbers VARCHAR(255)  -- '555-1234, 555-5678, 555-9999'
);

-- CORRECT: 1NF compliant
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE user_phones (
    phone_id INT PRIMARY KEY,
    user_id INT,
    phone_number VARCHAR(20),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);
```

### Second Normal Form (2NF)

**Rule:** Must be in 1NF + **no partial dependencies** (all non-key columns depend on the ENTIRE primary key).

```sql
-- WRONG: Violates 2NF (product_name depends only on product_id, not full key)
CREATE TABLE order_items_bad (
    order_id INT,
    product_id INT,
    product_name VARCHAR(100),  -- Depends only on product_id!
    quantity INT,
    PRIMARY KEY (order_id, product_id)
);

-- CORRECT: 2NF compliant
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100)
);
```

### Third Normal Form (3NF)

**Rule:** Must be in 2NF + **no transitive dependencies** (non-key columns don't depend on other non-key columns).

```sql
-- WRONG: Violates 3NF (city depends on zip_code, not user_id)
CREATE TABLE users_bad (
    user_id INT PRIMARY KEY,
    name VARCHAR(100),
    zip_code VARCHAR(10),
    city VARCHAR(100)  -- city depends on zip_code!
);

-- CORRECT: 3NF compliant
CREATE TABLE users (
    user_id INT PRIMARY KEY,
    name VARCHAR(100),
    zip_code VARCHAR(10),
    FOREIGN KEY (zip_code) REFERENCES zip_codes(zip_code)
);

CREATE TABLE zip_codes (
    zip_code VARCHAR(10) PRIMARY KEY,
    city VARCHAR(100),
    state VARCHAR(50)
);
```

---

## Layer 2: Practical Engineering (Intermediate)

### Real-World Normalization Process

#### Step-by-Step Example: E-commerce Orders

**Starting Point: Unnormalized Spreadsheet**

```
Order Data Dump:
┌────┬──────┬───────┬────────┬─────────┬──────┬───────┬──────┐
│ ID │ Date │ Cust  │ Email  │ Product │ Qty  │ Price │ Cat  │
├────┼──────┼───────┼────────┼─────────┼──────┼───────┼──────┤
│ 1  │ 1/1  │ Alice │ a@e.c  │ Laptop  │ 1    │ 1000  │ Elec │
│ 2  │ 1/2  │ Bob   │ b@e.c  │ Phone   │ 2    │ 500   │ Elec │
│ 3  │ 1/3  │ Alice │ a@e.c  │ Mouse   │ 3    │ 20    │ Elec │
└────┴──────┴───────┴────────┴─────────┴──────┴───────┴──────┘
```

**Step 1: Achieve 1NF** (Atomic values)

Already atomic, but let's handle multiple items per order:

```sql
CREATE TABLE orders_1nf (
    order_id INT,
    order_date DATE,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    product_name VARCHAR(200),
    category VARCHAR(50),
    quantity INT,
    unit_price DECIMAL(10,2)
);
```

**Step 2: Achieve 2NF** (Remove partial dependencies)

Identify entities: Customer, Order, Product

```sql
-- Customers
CREATE TABLE customers (
    customer_id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

-- Products
CREATE TABLE products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    product_name VARCHAR(200),
    category VARCHAR(50),
    unit_price DECIMAL(10,2)
);

-- Orders
CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    customer_id INT,
    order_date DATE,
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);

-- Order Items (junction table)
CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    PRIMARY KEY (order_id, product_id),
    FOREIGN KEY (order_id) REFERENCES orders(order_id),
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);
```

**Step 3: Achieve 3NF** (Remove transitive dependencies)

Category depends on product, not product directly:

```sql
-- Categories (separate table)
CREATE TABLE categories (
    category_id INT PRIMARY KEY AUTO_INCREMENT,
    category_name VARCHAR(50) UNIQUE
);

-- Update products table
CREATE TABLE products (
    product_id INT PRIMARY KEY AUTO_INCREMENT,
    product_name VARCHAR(200),
    category_id INT,
    unit_price DECIMAL(10,2),
    FOREIGN KEY (category_id) REFERENCES categories(category_id)
);
```

### Practical Benefits

**Before Normalization:**
```sql
-- Update product price: Must update ALL order history!
UPDATE orders_bad SET unit_price = 1100 WHERE product_name = 'Laptop';
-- Affects 1000 historical orders → WRONG!
```

**After Normalization:**
```sql
-- Update product price: One row
UPDATE products SET unit_price = 1100 WHERE product_id = 5;
-- Historical orders still reference old price snapshot → CORRECT!

-- Actually, for orders, we store price at time of purchase:
ALTER TABLE order_items ADD COLUMN price_at_purchase DECIMAL(10,2);
```

### Common Normalization Decisions

| Scenario | Normalize? | Reason |
|----------|------------|--------|
| User addresses | ✅ Yes | Create `addresses` table (users may have multiple) |
| Product categories | ✅ Yes | Separate `categories` table (for categorization consistency) |
| Order total amount | ❌ No | Calculated field, but store for historical accuracy |
| User's full name | 🤔 Maybe | Split to `first_name`, `last_name` if you need to sort/search |
| Country names | ✅ Yes | Lookup table (consistent spelling, easy updates) |

### When NOT to Normalize

```sql
-- Calculated fields stored for performance
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    total_amount DECIMAL(10,2),  -- Could calculate from order_items, BUT
    tax_amount DECIMAL(10,2),     -- storing ensures historical accuracy
    -- Tax rates change over time!
    FOREIGN KEY (customer_id) REFERENCES customers(customer_id)
);
```

---

## Layer 3: Advanced Engineering (Senior)

### Boyce-Codd Normal Form (BCNF)

**Rule:** Every determinant must be a candidate key.

**Translation:** If column A determines column B, then A should be a candidate key for the table.

#### Example: Course Scheduling

```sql
-- 3NF but NOT BCNF
CREATE TABLE class_schedule (
    student_id INT,
    course_id INT,
    instructor VARCHAR(100),
    PRIMARY KEY (student_id, course_id)
);

-- Problem: instructor determines course (each course has one instructor)
-- But instructor is not a candidate key!

-- BCNF: Split into two tables
CREATE TABLE course_instructors (
    course_id INT PRIMARY KEY,
    instructor VARCHAR(100)
);

CREATE TABLE student_courses (
    student_id INT,
    course_id INT,
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (course_id) REFERENCES course_instructors(course_id)
);
```

### Fourth Normal Form (4NF) - Multi-valued Dependencies

**Rule:** No multi-valued dependencies (independent relationships in same table).

```sql
-- WRONG: Violates 4NF
CREATE TABLE employee_skills_languages (
    employee_id INT,
    skill VARCHAR(50),
    language VARCHAR(50),
    PRIMARY KEY (employee_id, skill, language)
);

-- Problem: Skills and languages are independent!
-- Employee who speaks 3 languages and has 2 skills = 6 rows!

-- CORRECT: Separate tables
CREATE TABLE employee_skills (
    employee_id INT,
    skill VARCHAR(50),
    PRIMARY KEY (employee_id, skill)
);

CREATE TABLE employee_languages (
    employee_id INT,
    language VARCHAR(50),
    PRIMARY KEY (employee_id, language)
);
```

### Fifth Normal Form (5NF) - Join Dependencies

**Rule:** No join dependencies (table can't be reconstructed from smaller tables without loss).

Rarely needed in practice. Most systems stop at 3NF/BCNF.

### Functional Dependencies

Understanding **functional dependencies** is key to normalization.

**Definition:** Column B is functionally dependent on column A if each value of A determines exactly one value of B.

```
A → B  (A determines B)

Examples:
- user_id → email  (each user has one email)
- order_id → order_date  (each order has one date)
- zip_code → city  (each zip belongs to one city)
```

**Full vs Partial Dependency:**

```sql
-- Composite key: (order_id, product_id)
-- Full dependency: quantity depends on BOTH order_id AND product_id
-- Partial dependency: product_name depends only on product_id (VIOLATES 2NF!)
```

### Normalization Algorithm

```
Input: Unnormalized relation
Output: Set of normalized tables

1. Identify all functional dependencies
2. For each FD X → Y:
   a. If X is not a superkey, create new table with X as key
   b. Move dependent attributes to new table
3. Repeat until no violations
4. Add foreign keys to maintain relationships
```

### Over-Normalization Pitfalls

```sql
-- OVER-NORMALIZED: Taken too far
CREATE TABLE users (
    user_id INT PRIMARY KEY
);

CREATE TABLE user_first_names (
    user_id INT PRIMARY KEY,
    first_name VARCHAR(50),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

CREATE TABLE user_last_names (
    user_id INT PRIMARY KEY,
    last_name VARCHAR(50),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

-- PROBLEM: Every user query requires 3 JOINs!
-- first_name and last_name should stay together in users table.
```

**Rule of Thumb:** If two attributes are **always accessed together** and have a **1:1 relationship**, keep them in the same table.

### Denormalization for Performance

**Strategic Denormalization:** Deliberately violate normalization for speed.

```sql
-- Normalized (3NF)
CREATE TABLE orders (
    order_id INT PRIMARY KEY,
    customer_id INT,
    order_date DATE
);

CREATE TABLE order_items (
    order_id INT,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2),
    PRIMARY KEY (order_id, product_id)
);

-- Denormalized: Add total to orders (avoid SUM calculation)
ALTER TABLE orders ADD COLUMN total_amount DECIMAL(10,2);

-- Maintain with trigger
CREATE TRIGGER update_order_total
AFTER INSERT ON order_items
FOR EACH ROW
UPDATE orders 
SET total_amount = (
    SELECT SUM(quantity * unit_price) 
    FROM order_items 
    WHERE order_id = NEW.order_id
)
WHERE order_id = NEW.order_id;
```

---

## Layer 4: Architect Thinking

### Normalization vs Denormalization at Scale

| Aspect | Normalized (3NF) | Denormalized | Hybrid |
|--------|------------------|--------------|--------|
| **Data Integrity** | ✅ Excellent | ⚠️ Manual enforcement | 🟡 Good |
| **Write Performance** | ✅ Fast (single update) | ⚠️ Slow (multiple updates) | 🟡 Medium |
| **Read Performance** | ⚠️ Slow (many JOINs) | ✅ Very fast (no JOINs) | 🟡 Good |
| **Storage Cost** | ✅ Minimal | ❌ High (duplication) | 🟡 Medium |
| **Complexity** | 🟡 Medium (many tables) | ✅ Simple (few tables) | ❌ High (sync logic) |
| **Best For** | OLTP, transactional | OLAP, analytics, read-heavy | Production systems |

### Decision Framework: When to Normalize

```
Normalize (3NF) when:
✓ Data integrity is critical (financial, healthcare)
✓ Write-heavy workload
✓ Storage cost is concern
✓ Schema changes frequently
✓ Team size is small (less complex to maintain)

Denormalize when:
✓ Read-heavy workload (95%+ reads)
✓ Query performance critical (< 50ms SLA)
✓ Data rarely changes
✓ Horizontal scaling needed
✓ Analytics/reporting use case
```

### OLTP vs OLAP Designs

#### OLTP (Online Transaction Processing) - Normalized

```sql
-- Highly normalized for transactional integrity
CREATE TABLE accounts (
    account_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    balance DECIMAL(15,2),
    currency CHAR(3),
    FOREIGN KEY (user_id) REFERENCES users(user_id)
);

CREATE TABLE transactions (
    transaction_id BIGINT PRIMARY KEY,
    from_account BIGINT,
    to_account BIGINT,
    amount DECIMAL(15,2),
    timestamp TIMESTAMP,
    FOREIGN KEY (from_account) REFERENCES accounts(account_id),
    FOREIGN KEY (to_account) REFERENCES accounts(account_id)
);

-- Write: Simple, fast
-- Read: Requires JOINs for analysis
```

#### OLAP (Online Analytical Processing) - Denormalized

```sql
-- Star schema: Denormalized for fast queries
CREATE TABLE fact_transactions (
    transaction_id BIGINT PRIMARY KEY,
    
    -- Denormalized dimensions
    from_user_name VARCHAR(100),
    from_user_country VARCHAR(50),
    to_user_name VARCHAR(100),
    to_user_country VARCHAR(50),
    
    amount DECIMAL(15,2),
    currency CHAR(3),
    transaction_date DATE,
    transaction_hour TINYINT,
    
    -- Metrics
    fee_amount DECIMAL(10,2),
    exchange_rate DECIMAL(10,6)
);

-- Query: No JOINs needed, blazing fast!
SELECT 
    from_user_country,
    SUM(amount) as total_volume
FROM fact_transactions
WHERE transaction_date >= '2024-01-01'
GROUP BY from_user_country;
```

### Data Warehouse Design Patterns

#### Star Schema

```
            dim_products
                  │
                  │
dim_customers ──fact_sales── dim_time
                  │
                  │
             dim_stores
```

**Characteristics:**
- One fact table (metrics)
- Multiple dimension tables (denormalized)
- No normalization in dimensions
- Simple JOINs, fast aggregations

#### Snowflake Schema

```
        dim_products
             │
        dim_categories
             │
dim_customers ──fact_sales── dim_time
             │                   │
        dim_regions         dim_quarters
```

**Characteristics:**
- Normalized dimension tables
- More storage efficient
- Slower query performance
- Better data integrity

### Event Sourcing vs Normalized CRUD

**Traditional Normalized:**
```sql
-- Users table (current state only)
UPDATE users SET email = 'new@email.com' WHERE user_id = 123;
-- Old email lost forever
```

**Event Sourcing (Append-only, denormalized log):**
```sql
-- Events table (full history)
INSERT INTO user_events (user_id, event_type, old_value, new_value, timestamp)
VALUES (123, 'email_changed', 'old@email.com', 'new@email.com', NOW());

-- Rebuild current state by replaying events
-- OR maintain materialized view of current state
```

#### CQRS & Event Sourcing Deep Dive (Architect Level)

In highly scalable systems, the same database cannot efficiently serve both complex writes and high-frequency reads.

**CQRS (Command Query Responsibility Segregation)**:
- **Command Side**: Optimized for writes (3NF, strict constraints).
- **Query Side**: Optimized for reads (Denormalized, Flat tables, or even Search Indexes like Elasticsearch).

**Why Architects Choose This?**
- **Independent Scaling**: Scale the read database cluster infinitely without worrying about write locking.
- **Optimized Schemas**: One table per UI view (zero JOINs).

**Event Sourcing**:
- Instead of storing the *state*, you store the *sequence of events*.
- **Auditability**: Perfect for financial/legal systems.
- **Time Travel**: Rebuild state at any point in history.
- **Challenge**: "Snapshotting" is required to avoid replaying millions of events for every read.

### Microservices and Normalization

**Problem:** Can't JOIN across service databases

```
User Service DB          Order Service DB
┌──────────┐            ┌──────────┐
│  users   │     ❌     │  orders  │
└──────────┘  No JOIN   └──────────┘
```

**Solution 1: Denormalize Cross-Service Data**
```sql
-- In Order Service, duplicate user info
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    user_id BIGINT,  -- Reference to User Service
    user_name VARCHAR(100),  -- Denormalized!
    user_email VARCHAR(100), -- Denormalized!
    -- Keep in sync via events
);
```

**Solution 2: Service-to-Service Calls**
```java
// Fetch separately, join in application
Order order = orderRepository.findById(orderId);
User user = userServiceClient.getUserById(order.getUserId());
OrderDTO result = new OrderDTO(order, user);
```

#### Zero-Downtime Schema Evolution (Expand-Contract)

Modifying a table with millions of rows can lock it for hours. Architects use the **Expand-Contract pattern**.

1. **Expand**: Add a new column (nullable) to the table.
2. **Dual-Write**: Update the Java app to write to both the old and new columns.
3. **Backfill**: Run a background job to copy data from the old column to the new one for existing rows.
4. **Transition**: Update the app to read *only* from the new column.
5. **Contract**: Remove the old column (once you are 100% sure the new one is correct).

**Architect Tip**: Use database-specific features like `ONLINE INDEX` (SQL Server) or `ALGORITHM=INPLACE` (MySQL) to reduce lock times during expansion.

### CAP Theorem and Normalization

```
Strong Normalization requires:
- ACID transactions
- Referential integrity
- Immediate consistency

→ Favors CA (Consistency + Availability) in CAP
→ Struggles with partition tolerance (distributed systems)

Denormalization enables:
- Eventual consistency
- Partition tolerance
- Geographic distribution

→ Favors AP (Availability + Partition Tolerance)
```

#### Polyglot Persistence Strategy

Not all data belongs in a Relational Database. An architect chooses the tool for the job.

| Data Type | Best Storage | Why? |
|-----------|--------------|------|
| **Core Business Objects** | PostgreSQL / MySQL | ACID, Transactions, Relational |
| **Search / Catalog** | Elasticsearch / Solr | Full-text search, fuzzy matching |
| **Sessions / Caching** | Redis / Memcached | Sub-millisecond latency (RAM) |
| **Analytics / Time-Series** | InfluxDB / ClickHouse | High-volume aggregation, columnar |
| **Large BLOBs / Assets** | AWS S3 / MinIO | Cost-effective, specialized I/O |

### Cost Analysis

```
Scenario: 100M orders, 10M users

Option 1: Fully Normalized (3NF)
- Storage: 500GB
- Query time (with JOIN): 2-5 seconds
- Write time: 10ms
- Annual cost: $5,000 (smaller DB instance)
- Operational complexity: Low

Option 2: Fully Denormalized
- Storage: 2TB (4x duplication)
- Query time (no JOIN): 50ms
- Write time: 100ms (update multiple tables)
- Annual cost: $15,000 (larger storage + sync jobs)
- Operational complexity: High

Option 3: Hybrid (Recommended)
- Storage: 800GB
- Query time: 100-500ms (caching helps)
- Write time: 20ms
- Annual cost: $8,000
- Operational complexity: Medium
- Denormalize only hot paths
- Keep normalized for data integrity
```

---

## Denormalization Strategies

### Strategy 1: Computed Columns

```sql
-- Store calculated values
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    subtotal DECIMAL(10,2),
    tax_rate DECIMAL(4,3),
    tax_amount DECIMAL(10,2),  -- Denormalized: Could calculate
    total_amount DECIMAL(10,2), -- Denormalized: subtotal + tax
    item_count INT              -- Denormalized: COUNT from order_items
);

-- Maintain with triggers or application logic
```

### Strategy 2: Materialized Views

```sql
-- Expensive query (10 seconds with JOINs)
CREATE VIEW user_analytics AS
SELECT 
    u.user_id,
    u.name,
    COUNT(o.order_id) as total_orders,
    SUM(o.total_amount) as lifetime_value
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY u.user_id;

-- Materialize it (pre-compute and store)
CREATE MATERIALIZED VIEW user_analytics_mat AS
SELECT * FROM user_analytics;

-- Refresh periodically
REFRESH MATERIALIZED VIEW user_analytics_mat;

-- Now queries are instant
SELECT * FROM user_analytics_mat WHERE user_id = 123;
```

### Strategy 3: Cache Tables

```sql
-- Hot data cache table (denormalized)
CREATE TABLE user_dashboard_cache (
    user_id BIGINT PRIMARY KEY,
    last_login TIMESTAMP,
    order_count INT,
    total_spent DECIMAL(10,2),
    favorite_category VARCHAR(50),
    cache_updated_at TIMESTAMP,
    INDEX idx_cache_updated (cache_updated_at)
);

-- Rebuild daily or on-demand
TRUNCATE user_dashboard_cache;
INSERT INTO user_dashboard_cache
SELECT 
    u.user_id,
    u.last_login,
    COUNT(o.order_id),
    SUM(o.total_amount),
    /* complex logic */,
    NOW()
FROM users u
LEFT JOIN orders o ON u.user_id = o.user_id
GROUP BY u.user_id;
```

### Strategy 4: JSON Columns (PostgreSQL, MySQL 5.7+)

```sql
-- Store related data as JSON blob
CREATE TABLE orders (
    order_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    items JSON,  -- Denormalized order items
    shipping_address JSON,  -- Denormalized address
    created_at TIMESTAMP
);

-- Store snapshot at time of order
INSERT INTO orders (user_id, items, shipping_address)
VALUES (
    123,
    '[{"product_id": 5, "name": "Laptop", "qty": 1, "price": 1000}]',
    '{"street": "123 Main St", "city": "NYC", "zip": "10001"}'
);

-- Query JSON
SELECT items->>'$.name' as product_name
FROM orders
WHERE order_id = 1;
```

---

## Performance Trade-offs

### Read vs Write Performance

```sql
-- Benchmark: 1M orders, 100K users

Scenario 1: Normalized (3NF)
-----------------------------------------
SELECT u.name, o.total_amount
FROM users u
JOIN orders o ON u.user_id = o.user_id
WHERE o.order_id = 123;

Execution time: 15ms (with indexes)
Write time (update user): 2ms

Scenario 2: Denormalized
-----------------------------------------
SELECT user_name, total_amount
FROM orders_denormalized
WHERE order_id = 123;

Execution time: 0.5ms
Write time (update user): 50ms (must update all orders!)
```

### Storage vs Speed

```
Normalized: 100GB storage, 500ms queries
+Cache: 120GB storage, 10ms queries (cache hit)
+Denormalize hot paths: 150GB storage, 5ms queries
+Materialize everything: 300GB storage, 1ms queries

Diminishing returns law applies!
```

---

## Common Mistakes

### Mistake 1: Over-Normalization

```sql
-- TOO NORMALIZED (bad)
CREATE TABLE order_street_address (...);
CREATE TABLE order_city (...);
CREATE TABLE order_state (...);
CREATE TABLE order_zip (...);

-- Every order address query requires 4 JOINs!

-- BETTER: Group related attributes
CREATE TABLE order_addresses (
    address_id BIGINT PRIMARY KEY,
    street VARCHAR(200),
    city VARCHAR(100),
    state VARCHAR(50),
    zip VARCHAR(10)
);
```

### Mistake 2: Premature Denormalization

```sql
-- DON'T denormalize without measuring first!

-- Start normalized
CREATE TABLE users (...);
CREATE TABLE orders (...);

-- Measure query performance
EXPLAIN ANALYZE SELECT ...;

-- Only denormalize if:
-- 1. Query is slow (>100ms)
-- 2. Query is frequent (hot path)
-- 3. Data rarely changes
```

### Mistake 3: Inconsistent Denormalized Data

```sql
-- PROBLEM: Denormalized user_name in orders
UPDATE users SET name = 'Alice Cooper' WHERE user_id = 123;
-- Forgot to update orders table!

SELECT user_name FROM orders WHERE user_id = 123;
-- Still shows old name "Alice"

-- SOLUTION: Use triggers or event-driven sync
CREATE TRIGGER sync_user_name
AFTER UPDATE ON users
FOR EACH ROW
UPDATE orders 
SET user_name = NEW.name 
WHERE user_id = NEW.user_id;
```

---

## Production Examples

### Example 1: E-commerce Product Catalog

```sql
-- Normalized design (3NF)
CREATE TABLE categories (
    category_id INT PRIMARY KEY,
    category_name VARCHAR(100),
    parent_category_id INT,
    FOREIGN KEY (parent_category_id) REFERENCES categories(category_id)
);

CREATE TABLE products (
    product_id BIGINT PRIMARY KEY,
    sku VARCHAR(50) UNIQUE,
    name VARCHAR(200),
    description TEXT,
    category_id INT,
    brand_id INT,
    base_price DECIMAL(10,2),
    FOREIGN KEY (category_id) REFERENCES categories(category_id),
    FOREIGN KEY (brand_id) REFERENCES brands(brand_id)
);

CREATE TABLE product_variants (
    variant_id BIGINT PRIMARY KEY,
    product_id BIGINT,
    size VARCHAR(20),
    color VARCHAR(50),
    stock_quantity INT,
    FOREIGN KEY (product_id) REFERENCES products(product_id)
);

-- Denormalized search index (for fast product listing)
CREATE TABLE product_search_index (
    product_id BIGINT PRIMARY KEY,
    full_name VARCHAR(500),  -- "Nike Air Max - Red - Size 10"
    category_path VARCHAR(200),  -- "Shoes > Athletic > Running"
    brand_name VARCHAR(100),
    min_price DECIMAL(10,2),
    max_price DECIMAL(10,2),
    total_stock INT,
    search_keywords TEXT,  -- Pre-computed for full-text search
    last_updated TIMESTAMP
);
```

### Example 2: Social Media Timeline

```sql
-- Write-optimized (normalized)
CREATE TABLE posts (
    post_id BIGINT PRIMARY KEY,
    user_id BIGINT,
    content TEXT,
    created_at TIMESTAMP
);

CREATE TABLE post_likes (
    post_id BIGINT,
    user_id BIGINT,
    created_at TIMESTAMP,
    PRIMARY KEY (post_id, user_id)
);

-- Read-optimized (denormalized timeline cache)
CREATE TABLE user_timelines (
    user_id BIGINT,
    post_id BIGINT,
    author_id BIGINT,
    author_name VARCHAR(100),  -- Denormalized
    author_avatar_url VARCHAR(500),  -- Denormalized
    content TEXT,  -- Denormalized
    like_count INT,  -- Denormalized
    created_at TIMESTAMP,
    PRIMARY KEY (user_id, post_id)
);

-- Fan-out on write: When user posts, write to all followers' timelines
```

---

## Interview Questions

### Beginner Level

**Q1: What is database normalization and why is it important?**

Normalization organizes data to:
- Eliminate redundancy
- Ensure data integrity
- Make updates easier
- Prevent anomalies (insert, update, delete)

**Q2: Explain 1NF, 2NF, and 3NF in simple terms.**

- **1NF:** No repeating groups, atomic values
- **2NF:** 1NF + no partial dependencies on composite keys
- **3NF:** 2NF + no transitive dependencies (non-key columns don't depend on each other)

### Intermediate Level

**Q3: Given this table, normalize to 3NF:**

```
employee_projects
┌──────────┬───────────┬──────────────┬──────────┬──────────┐
│ emp_id   │ emp_name  │ emp_dept     │ proj_id  │ proj_name│
├──────────┼───────────┼──────────────┼──────────┼──────────┤
│ 1        │ Alice     │ Engineering  │ 101      │ Web App  │
│ 1        │ Alice     │ Engineering  │ 102      │ Mobile   │
│ 2        │ Bob       │ Marketing    │ 101      │ Web App  │
└──────────┴───────────┴──────────────┴──────────┴──────────┘
```

**Answer:**
```sql
CREATE TABLE employees (
    emp_id INT PRIMARY KEY,
    emp_name VARCHAR(100),
    dept_id INT,
    FOREIGN KEY (dept_id) REFERENCES departments(dept_id)
);

CREATE TABLE departments (
    dept_id INT PRIMARY KEY,
    dept_name VARCHAR(100)
);

CREATE TABLE projects (
    proj_id INT PRIMARY KEY,
    proj_name VARCHAR(100)
);

CREATE TABLE employee_projects (
    emp_id INT,
    proj_id INT,
    PRIMARY KEY (emp_id, proj_id),
    FOREIGN KEY (emp_id) REFERENCES employees(emp_id),
    FOREIGN KEY (proj_id) REFERENCES projects(proj_id)
);
```

**Q4: When should you denormalize a database?**

Denormalize when:
- Read-to-write ratio is very high (95%+ reads)
- Join performance is unacceptable despite indexing
- Data changes infrequently
- You can handle eventual consistency
- Query response time SLA can't be met otherwise

### Senior Level

**Q5: Design a schema for a multi-tenant SaaS application. Should you normalize tenant data?**

**Options:**

**Option A: Shared schema (normalized)**
```sql
CREATE TABLE tenants (...);
CREATE TABLE users (
    user_id BIGINT,
    tenant_id BIGINT,  -- Every table has tenant_id
    ...
);
-- All tenants share tables
```

**Option B: Schema per tenant (isolated)**
```
tenant_1.users
tenant_2.users
tenant_3.users
...
```

**Recommendation:**
- **< 1000 tenants:** Shared schema with tenant_id filtering
- **Large tenants (millions of rows each):** Schema per tenant
- **Hybrid:** Separate table per large tenant, shared for small tenants

**Q6: You have a slow query with 5 JOINs. Walk through your optimization process.**

1. **EXPLAIN ANALYZE** - See execution plan
2. **Check indexes** - All foreign keys indexed?
3. **Examine cardinality** - How selective are filters?
4. **Rewrite query** - Can subquery become JOIN?
5. **Partial denormalization** - Cache heavily accessed paths
6. **Materialized view** - Pre-compute if refreshable
7. **Application caching** - Redis for truly hot data
8. **Query complexity** - Can we split into multiple simpler queries?

### Architect Level

**Q7: Design a database strategy for a global e-commerce platform with 10M orders/day across 50 countries. Address normalization, sharding, and consistency.**

**Design:**

```
Regional Sharding:
- US Shard (user_id % region)
- EU Shard
- APAC Shard

Within Each Shard:
- Orders: Normalized (3NF) for writes
- Products: Denormalized replica (read-only)
- Users: Normalized master

Read Path:
- Product catalog: Denormalized + CDN cached
- Order history: Normalized with Redis cache
- User dashboard: Materialized view, refreshed nightly

Write Path:
- Orders: Write to local shard (strongly consistent)
- Products: Write to master, async replicate (eventual)
- Users:Write to master, sync replicate to shards

Cross-Shard Queries:
- Aggregate reporting: Separate data warehouse (denormalized)
- User moved regions: Lazy migration on next login

Consistency:
- Orders: Strong (ACID within shard)
- Products: Eventual (async replication, 1-2 sec lag OK)
- Inventory: Eventual with optimistic locking
```

**Q8: Explain the trade-offs between normalization and the Actor model (event sourcing).**

| Aspect | Normalization (CRUD) | Event Sourcing |
|--------|----------------------|----------------|
| **Data Model** | Current state (normalized tables) | Event log (append-only) |
| **Reads** | Fast (if indexed and cached) | Slow (replay events) or cached projections |
| **Writes** | UPDATE in place | Append event (fast) |
| **History** | Lost (unless audited) | Full history preserved |
| **Consistency** | ACID transactions | Eventual (events processed async) |
| **Debugging** | Hard (state at time unknown) | Easy (replay to any point) |
| **Complexity** | Low (well-understood) | High (event handlers, projections) |
| **Use case** | Traditional apps | Event-driven, audit-critical systems |

---

## Cross-References

- **[01-sql-basics.md](./01-sql-basics.md)** - Foundation SQL concepts
- **[02-joins.md](./02-joins.md)** - JOIN performance implications of normalization
- **[04-queries.md](./04-queries.md)** - Optimizing queries on normalized schemas
- **[Spring Data](../springboot/spring-data.md)** - ORM and object-relational mapping
- **[System Design](../system-design/data-modeling.md)** - High-level data architecture
- **[Microservices](../system-design/microservices.md)** - Database-per-service pattern

---

## Summary

### Key Takeaways

```
Beginner:
✓ Understand 1NF, 2NF, 3NF
✓ Know why normalization prevents anomalies
✓ Practice identifying redundant data

Intermediate:
✓ Normalize real-world data to 3NF
✓ Understand when to denormalize
✓ Create proper foreign key relationships

Senior:
✓ Master BCNF and higher normal forms
✓ Design hybrid normalized/denormalized systems
✓ Implement materialized views and cache tables
✓ Optimize JOIN performance

Architect:
✓ Balance normalization for different workloads (OLTP vs OLAP)
✓ Design for distributed systems and microservices
✓ Understand CAP theorem implications
✓ Make cost-effective denormalization decisions
✓ Implement CQRS when appropriate
```

### Decision Tree

```
Is data integrity critical?
├─ Yes → Normalize (3NF)
│   └─ Is read performance poor?
│       ├─ Yes → Add indexes first
│       │   └─ Still slow?
│       │       ├─ Yes → Materialize views or selective denormalization
│       │       └─ No → Stay normalized
│       └─ No → Stay normalized
└─ No → Can tolerate eventual consistency?
    ├─ Yes → Denormalize for performance
    └─ No → Hybrid approach with sync triggers
```

---

**Previous:** [← 02-joins.md](./02-joins.md) - JOIN strategies and optimization  
**Next:** [04-queries.md →](./04-queries.md) - Advanced SQL query patterns

**Author:** Nehru Usare  
**Version:** 1.0 | February 2026
