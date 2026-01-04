# **DAY 3 ANSWERS CHECK**

## ✅ **Self-Test Answers**

1. **What are the ACID properties and why are they important?**
   *Atomicity: Transactions are all-or-nothing. Consistency: Database rules are always maintained. Isolation: Concurrent transactions don't interfere. Durability: Committed data survives crashes. They're crucial for financial systems, e-commerce, and anywhere data integrity is non-negotiable.*

2. **When would you choose SQL over NoSQL?**
   *1. When ACID transactions are required (banking, payments)*
   *2. For complex queries with multiple joins*
   *3. When data has a fixed, predictable schema*
   *4. For reporting and analytics requiring SQL*
   *5. When strong consistency is more important than availability*

3. **What's horizontal vs vertical partitioning?**
   *Horizontal partitioning (sharding): Splitting rows across multiple servers (users_1, users_2).*
   *Vertical partitioning: Splitting columns into separate tables (user_profiles, user_settings) based on access patterns.*

4. **How do database indexes work?**
   *Indexes create sorted data structures (usually B-trees) that allow O(log n) lookups instead of O(n) full table scans. They're like a book's index - you find the page number quickly instead of scanning every page.*

## 📊 **SQL Database Quick Facts**
- **Most Common**: MySQL, PostgreSQL, SQL Server, Oracle
- **Scaling Limit**: ~1-10 billion rows comfortably
- **Typical Use Cases**: Banking, ERP, E-commerce, CRM systems
- **Cost**: Open-source (MySQL, PostgreSQL) to expensive (Oracle)

---

# **DAY 4: NOSQL DATABASES - BEYOND RELATIONAL**

## 🎯 **Today's Mission**
Master NoSQL databases - the engines behind massively scalable systems like Facebook, Google, and Amazon.

## ⏰ **Daily Schedule (2-3 Hours)**

### **BLOCK 1: NOSQL FUNDAMENTALS & CAP THEOREM (60 minutes)**

**0-10 min: Warm-up**
- SQL gave us ACID but hit scaling limits
- NoSQL emerged from Google, Amazon, Facebook's scaling challenges
- Today's mantra: **"Scale over everything"** (well, almost)

**10-40 min: The CAP Theorem - Your Guiding Star**

**Brewer's Theorem**: You can only have 2 of 3:
```
C: Consistency - All nodes see same data at same time
A: Availability - Every request gets response (no errors)
P: Partition Tolerance - System works despite network failures
```

**Visualize this**:
- **CP** (Consistency + Partition Tolerance): SQL databases, HBase
    - If network partition, system becomes unavailable
    - Example: Banking systems

- **AP** (Availability + Partition Tolerance): Cassandra, DynamoDB
    - If network partition, keep serving with eventual consistency
    - Example: Social media feeds

- **CA** (Consistency + Availability): Single-node databases
    - Doesn't exist in distributed systems (must handle partitions)

**Draw**: Venn diagram with CP, AP, CA

**PACELC Extension**:
```
If Partition occurs: Choose between Availability and Consistency (A vs C)
Else: Choose between Latency and Consistency (L vs C)
```

**40-60 min: NoSQL Database Types**

**Four Main Categories**:
1. **Key-Value Stores** (Redis, DynamoDB)
    - Simple: key → value
    - Use: Caching, sessions, counters
    - Draw: Hash table visualization

2. **Document Databases** (MongoDB, Couchbase)
    - Store JSON/BSON documents
    - Use: User profiles, product catalogs
    - Example: `{user_id: 1, name: "John", orders: [...]}`

3. **Column-Family Stores** (Cassandra, HBase)
    - Rows with flexible columns per row
    - Use: Time-series, analytics
    - Draw: Wide rows with varying columns

4. **Graph Databases** (Neo4j, Neptune)
    - Nodes + relationships
    - Use: Social networks, recommendations
    - Example: User → Friends With → Other User

**Hands-on**: Which type for each use case?
- Shopping cart: Key-Value (Redis)
- Product catalog: Document (MongoDB)
- Sensor data: Column-Family (Cassandra)
- Friend recommendations: Graph (Neo4j)

### **BLOCK 2: DEEP DIVE: CASSANDRA & DYNAMODB (45 minutes)**

**Cassandra (Facebook's Creation)**
```
Architecture: Masterless ring, all nodes equal
Data Model: Column-family, wide rows
Consistency: Tunable (ONE, QUORUM, ALL)
Write Path: Write to commit log + memtable → SSTable
Read Path: Check memtable → SSTables → Merge
```

**Key Features**:
- Linear scalability: Add nodes, get more capacity
- Multi-DC replication: Built-in
- CQL: SQL-like language (but no joins!)

**DynamoDB (Amazon's Managed DB)**
```
Managed: No servers to manage
Data Model: Key-value + optional documents
Capacity: Provisioned or on-demand
Features: Global tables, streams, DAX cache
```

**Comparison Table**:
```
| Aspect       | Cassandra            | DynamoDB            |
|--------------|----------------------|---------------------|
| Management   | Self-managed         | Fully managed       |
| Cost         | Capex + Ops          | Pay-per-request     |
| Scaling      | Add nodes            | Auto-scaling        |
| Consistency  | Tunable              | Eventually + strong |
```

**Design Pattern**: Time-series in Cassandra
```sql
CREATE TABLE sensor_data (
    sensor_id uuid,
    bucket timestamp,  -- e.g., day bucket
    event_time timestamp,
    value float,
    PRIMARY KEY ((sensor_id, bucket), event_time)
) WITH CLUSTERING ORDER BY (event_time DESC);
```
**Why bucket?** Avoid hotspot partitions!

### **BLOCK 3: REAL-WORLD NOSQL PATTERNS (45 minutes)**

**Case Study 1: Facebook's Messages (Initially Cassandra)**
- Problem: Billions of messages, need infinite scale
- Solution: Cassandra's horizontal scaling
- Data model: `(user_id, thread_id, message_id) → message`
- Challenge: Secondary indexes slow at scale

**Case Study 2: Amazon's Shopping Cart (DynamoDB)**
- Problem: Peak holiday traffic (10x normal)
- Solution: DynamoDB auto-scaling
- Data model: `user_id → {cart_items: [...], last_updated: ...}`
- Why NoSQL? Simple access pattern (get by user_id)

**Case Study 3: Netflix's Viewing History (Cassandra)**
- Problem: Track what billions watch
- Solution: Time-series in Cassandra
- Data model: `(user_id, date) → [viewed_movies]`
- Analytics: Pre-compute recommendations

**Design Exercise**: Choose NoSQL database for:

1. **Real-time Leaderboard** (Redis)
    - Why? Sorted sets, atomic increments
    - Pattern: `ZADD leaderboard score user_id`

2. **Social Media Activity Feed** (Cassandra)
    - Why? Write-heavy, time-ordered
    - Pattern: `(user_id, day) → [activities]`

3. **Product Recommendations** (Neo4j)
    - Why? Graph traversal for "people also bought"
    - Pattern: `Product → BOUGHT_WITH → Other Product`

**Anti-Patterns Alert**:
- ❌ Don't use NoSQL when you need complex transactions
- ❌ Don't use NoSQL when data fits in one machine
- ❌ Don't use NoSQL for heavy analytics (without proper design)
- ✅ Do use NoSQL for massive scale simple operations

### **BLOCK 4: INTERVIEW FOCUS (30 minutes)**

**Common Questions & Answers:**

1. **"When would you choose NoSQL over SQL?"**
   ```
   Answer: When...
   1. Scale > 1B records or > 1M writes/sec
   2. Simple access patterns (key-value, simple queries)
   3. Flexible/evolving schema needed
   4. High write throughput required
   5. Availability more important than strong consistency
   
   Example: Social media feeds, IoT sensor data, product catalogs
   ```

2. **"Explain eventual consistency"**
   ```
   Answer: All nodes will become consistent over time, but 
   may have temporary inconsistencies. Like DNS propagation.
   
   Real example: When you 'like' a post on Facebook, your 
   friend might not see it immediately, but will eventually.
   
   Trade-off: Better availability, worse immediate consistency.
   ```

3. **"How does Cassandra handle writes?"**
   ```
   1. Write to commit log (for durability)
   2. Write to memtable (in-memory)
   3. When memtable full, flush to SSTable (on disk)
   4. Periodically compact SSTables
   
   Why fast? Sequential writes, no random I/O
   ```

**Whiteboard Practice**:
Design a Twitter-like system's database layer:
```
SQL (PostgreSQL):
- Users, relationships, tweets metadata

NoSQL (Cassandra):
- Timeline: (user_id, day) → [tweet_ids]
- Followers: user_id → [follower_ids]
- Likes: tweet_id → [user_ids who liked]

Cache (Redis):
- User sessions
- Trending hashtags
```

**Estimation Practice**:
- Twitter: 500M tweets/day, store for 7 years
- Raw storage: 500M × 280 chars × 7 × 365 = ~320 TB
- Compression: ~80 TB
- Cassandra cluster: 100 nodes × 4TB each
- Cost: ~$500K/year on AWS

---

## 📝 **DAY 4 OUTPUT REQUIRED**

**Complete in your notebook:**

1. **NoSQL Decision Framework**:
```
Ask these questions:
1. Scale needed? (>1B rows → NoSQL)
2. Access pattern? (Simple key lookup → NoSQL)
3. Consistency needed? (Strong → SQL, Eventual → NoSQL)
4. Schema flexibility? (Changing → NoSQL)
5. Team expertise? (Consider learning curve)
```

2. **CAP Theorem Application Guide**:
```
Banking System: CP (Consistency + Partition Tolerance)
  - During network partition: Become unavailable
  - Better to say "system down" than show wrong balance

Social Media: AP (Availability + Partition Tolerance)
  - During network partition: Keep serving, sync later
  - Better to show slightly stale feed than be unavailable
```

3. **Database Selection Matrix**:
```
| Use Case               | Recommended DB      | Reason                          |
|------------------------|---------------------|---------------------------------|
| User sessions          | Redis (Key-Value)   | Fast, ephemeral                 |
| Product catalog        | MongoDB (Document)  | Flexible schema                 |
| Time-series data       | Cassandra (Column)  | Write-optimized, scales linearly|
| Friend recommendations | Neo4j (Graph)       | Natural for relationships       |
| Financial transactions | PostgreSQL (SQL)    | ACID compliance                 |
```

4. **Flashcards Created**:
- Front: CAP Theorem choices
- Back: CP (SQL), AP (NoSQL), CA (doesn't exist distributed)

- Front: Eventual Consistency Example
- Back: DNS propagation, social media likes

---

## 🧪 **Day 4 Self-Test**

**Answer without looking:**
1. What does CAP theorem state about distributed systems?
2. What are the four types of NoSQL databases?
3. When is eventual consistency acceptable?
4. Why is Cassandra good for time-series data?

**Answers:**
1. In distributed systems, you can only guarantee two of: Consistency, Availability, Partition Tolerance.
2. Key-Value, Document, Column-Family, Graph databases.
3. When immediate consistency isn't critical: social feeds, product recommendations, comments/likes.
4. Write-optimized architecture, linear scalability, good time-range queries with proper data modeling.

---

## 🚀 **Day 5 Preview: REPLICATION & PARTITIONING**

**Preparation Thought:**
- Now we have databases, but how do we make them highly available?
- How does Facebook keep serving even if a data center burns down?
- Think: Your WhatsApp messages syncing across phone and web

---

## 💪 **Accountability Check**

**Reply with:**
1. ✅ I've completed Day 4
2. 📱 One app you use daily that uses NoSQL: _____
3. 🔄 Most confusing part of CAP theorem: _____
4. ❓ Question about NoSQL: _____

**Remember**: NoSQL isn't "better" than SQL - it's different. Each has its place in the architecture!

---

**Ready to learn how databases survive disasters tomorrow?** This is critical for FAANG interviews! 🚀