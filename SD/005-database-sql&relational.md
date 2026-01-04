# **DAY 2 ANSWERS CHECK**

## ✅ **Self-Test Answers**

1. **What's the difference between write-through and write-back?**
   *Write-through: Synchronous write to both cache and database. Always consistent but slower writes.*
   *Write-back: Write to cache only, asynchronous write to database later. Faster writes but risk data loss if cache fails before sync.*

2. **When would you choose LFU over LRU?**
   *LFU (Least Frequently Used) when access patterns are stable and predictable (product catalog, popular articles).*
   *LRU (Least Recently Used) when recency matters more than frequency (social media feeds, news headlines).*

3. **How do you prevent cache avalanche?**
   *1. Randomize TTL values (add jitter)*
   *2. Implement cache warm-up before expiration*
   *3. Use circuit breakers to protect database*
   *4. Implement staggered expiration*

4. **What's a good cache hit rate for web applications?**
   *80-95% is excellent. Below 70% indicates poor cache strategy or wrong data being cached.*

## 📊 **Cache Pattern Quick Reference**
- **Cache-Aside**: Most common, app-managed, eventual consistency
- **Write-Through**: Banking systems, absolute consistency required
- **Write-Back**: Write-heavy systems, can tolerate some data loss
- **Refresh-Ahead**: Predictive systems, pre-warm cache

---

# **DAY 3: DATABASES - SQL & RELATIONAL DATABASES**

## 🎯 **Today's Mission**
Master relational database design - the backbone of transactional systems.

## ⏰ **Daily Schedule (2-3 Hours)**

### **BLOCK 1: SQL DATABASE FUNDAMENTALS (60 minutes)**

**0-10 min: Warm-up**
- Why SQL after caching? Because caches serve data FROM databases!
- SQL databases are for **structured**, **transactional** data
- Think: Banking systems, e-commerce orders, user accounts

**10-40 min: Core Concepts**

**1. ACID Properties - The SQL Superpower**
```
A: Atomicity - All or nothing (transactions)
C: Consistency - Data rules always maintained
I: Isolation - Concurrent transactions don't interfere  
D: Durability - Committed data survives crashes
```
**Draw**: Transaction flow with commit/rollback

**2. Normalization (1NF, 2NF, 3NF)**
```
1NF: Each column atomic, no repeating groups
2NF: No partial dependencies (all on primary key)
3NF: No transitive dependencies
```
**Example**: Design user orders database
- Denormalized: Users(name, email, orders[...])
- Normalized: Users(id, name, email), Orders(id, user_id, ...)

**3. Indexing - Making Queries Fast**
```sql
-- Without index: O(n) scan
-- With index: O(log n) search

CREATE INDEX idx_email ON users(email);
CREATE INDEX idx_composite ON orders(user_id, created_at);
```
**Types**:
- B-tree: Default, range queries
- Hash: Exact matches only
- Bitmap: Low cardinality columns

**40-60 min: Hands-on Design**
Design Twitter's database schema:
```sql
-- Users table
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP
);

-- Tweets table  
CREATE TABLE tweets (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT FOREIGN KEY REFERENCES users(id),
    content VARCHAR(280),
    created_at TIMESTAMP,
    INDEX idx_user_created (user_id, created_at)
);

-- Follows table
CREATE TABLE follows (
    follower_id BIGINT FOREIGN KEY REFERENCES users(id),
    followee_id BIGINT FOREIGN KEY REFERENCES users(id),
    created_at TIMESTAMP,
    PRIMARY KEY (follower_id, followee_id)
);
```
**Draw** these tables with relationships!

### **BLOCK 2: ADVANCED SQL CONCEPTS (45 minutes)**

**1. Joins - The Relational Magic**
```
INNER JOIN: Only matching rows
LEFT JOIN: All left + matching right
RIGHT JOIN: All right + matching left  
FULL OUTER JOIN: All rows from both
CROSS JOIN: Cartesian product
```
**Practice**: Write query for "get user with their tweet count"

**2. Transactions & Isolation Levels**
```sql
BEGIN TRANSACTION;
-- Do multiple operations
COMMIT; -- or ROLLBACK;
```

**Isolation Levels**:
- Read Uncommitted: Dirty reads possible
- Read Committed: No dirty reads (default in most)
- Repeatable Read: Same read in transaction
- Serializable: Complete isolation

**3. Partitioning - Scaling SQL**
```
Horizontal: Split rows by range/hash
  Example: users_2023, users_2024
Vertical: Split columns by access pattern
  Example: user_profiles, user_settings
```

**4. Replication - High Availability**
```
Master-Slave: Read from slaves, write to master
Master-Master: Bidirectional replication (conflict resolution needed)
```

### **BLOCK 3: SQL vs NOSQL DECISION MAKING (45 minutes)**

**When to use SQL?**
```
✅ Structured data with clear schema
✅ Complex queries with joins
✅ ACID transactions required  
✅ Data integrity critical (banking)
✅ Medium scale (up to billions of rows)
```

**When NOT to use SQL?**
```
❌ Unstructured or semi-structured data
❌ Massive scale (trillions of rows)
❌ Simple key-value access
❌ Flexible schema needed
❌ Write-heavy with simple reads
```

**CAP Theorem Reminder**:
- SQL: Chooses Consistency + Partition Tolerance (CP)
- NoSQL: Often chooses Availability + Partition Tolerance (AP)

**Case Studies**:

1. **Banking System (SQL)**
    - Why? Transactions, integrity, complex reporting
    - Tables: Accounts, Transactions, Customers
    - Indexes: Account number, transaction dates

2. **Social Media (Mixed)**
    - SQL: User profiles, relationships
    - NoSQL: Activity feeds, messages
    - Cache: Frequently accessed data

3. **E-commerce (SQL)**
    - Orders, inventory, payments need ACID
    - Product catalog might use NoSQL for flexibility

**Design Exercise**: Choose database for:
- Uber ride tracking: NoSQL (time-series, location data)
- Hospital patient records: SQL (critical, structured)
- Product reviews: Either (SQL for analytics, NoSQL for scale)

### **BLOCK 4: INTERVIEW FOCUS (30 minutes)**

**Common Questions & Answers:**

1. **"How would you scale a SQL database?"**
   ```
   1. Vertical scaling: Bigger machine (RAM, CPU)
   2. Read replicas: Offload reads
   3. Sharding: Split data by user_id, region
   4. Caching: Redis/Memcached for hot data
   5. Denormalization: Reduce joins (trading consistency for speed)
   ```

2. **"What's the N+1 query problem?"**
   ```
   Example: Fetch users and their posts
   Bad: 1 query for users, then N queries for each user's posts
   Good: Use JOIN or batch loading
   
   Solution: Eager loading, JOIN queries, database-level optimization
   ```

3. **"How do you handle database migrations?"**
   ```
   1. Version control migrations
   2. Backward compatible changes
   3. Blue-green deployment
   4. Feature flags for schema changes
   5. Never delete columns immediately
   ```

**Whiteboard Practice**:
Design an e-commerce database:
```
Tables needed:
- Users (id, email, created_at)
- Products (id, name, price, inventory)
- Orders (id, user_id, status, total)
- Order_items (order_id, product_id, quantity, price)
- Payments (id, order_id, amount, status)
```

**Estimation Practice**:
- Amazon: 300M users, 10 orders/user average
- Orders table size: 3B rows
- Storage: 3B × 1KB = 3TB
- Indexes: Additional 1-2TB
- Sharding: By user_id range or geographic region

---

## 📝 **DAY 3 OUTPUT REQUIRED**

**Complete in your notebook:**

1. **SQL vs NoSQL Decision Matrix**:
```
| Requirement          | SQL               | NoSQL             |
|----------------------|-------------------|-------------------|
| Schema flexibility   | Rigid             | Flexible          |
| Scaling              | Vertical/Hard     | Horizontal/Easy   |
| Transactions         | Strong ACID       | Eventually consistent |
| Query complexity     | Complex joins     | Simple lookups    |
| Use case example     | Banking           | Social feed       |
```

2. **Normalization Examples**:
```
Denormalized (1 table):
users_orders(id, username, email, order_id, product, price)

Normalized (3 tables):
users(id, username, email)
orders(id, user_id, total)
order_items(id, order_id, product, price)
```

3. **Index Strategy Guide**:
```
When to index:
1. Primary keys (auto-indexed)
2. Foreign keys
3. Columns in WHERE clauses
4. Columns in JOIN conditions
5. Columns in ORDER BY

When NOT to index:
1. Tables with frequent writes
2. Columns with few unique values
3. Very small tables
```

4. **Flashcards Created**:
- Front: ACID properties
- Back: Atomicity, Consistency, Isolation, Durability

- Front: N+1 Query Problem
- Back: 1 query + N individual queries = inefficient. Fix with JOINs

---

## 🧪 **Day 3 Self-Test**

**Answer without looking:**
1. What are the ACID properties and why are they important?
2. When would you choose SQL over NoSQL?
3. What's horizontal vs vertical partitioning?
4. How do database indexes work?

**Answers:**
1. Atomicity (all or nothing), Consistency (rules maintained), Isolation (transactions don't interfere), Durability (data survives crashes). Critical for financial systems.
2. When you need transactions, complex queries, data integrity, or structured schema.
3. Horizontal = split rows (sharding). Vertical = split columns (separate tables by access pattern).
4. Indexes create sorted data structures (B-trees) allowing O(log n) lookups instead of O(n) scans.

---

## 🚀 **Day 4 Preview: NOSQL DATABASES**

**Preparation Thought:**
- SQL gave us ACID but limited scale. What if we need billions of users?
- Think: Facebook's messages, Google's search index, Amazon's product catalog

---

## 💪 **Accountability Check**

**Reply with:**
1. ✅ I've completed Day 3
2. 🏦 One real system that definitely uses SQL: _____
3. 🔍 Most surprising thing about database indexing: _____
4. ❓ Question about SQL databases: _____

**Remember**: SQL databases power the world's financial systems. Master these fundamentals!

---
