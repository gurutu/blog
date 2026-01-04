# **DAY 6 ANSWERS CHECK**

## ✅ **Self-Test Answers**

1. **What's the difference between message queues and pub-sub?**
   *Message Queues: Point-to-point communication where each message is consumed by exactly one consumer. Messages are removed from queue after consumption. Used for task distribution and load balancing.*
   *Pub-Sub: Broadcast pattern where each message is delivered to all interested subscribers. Messages persist independent of consumption. Used for event broadcasting and notifications.*

2. **Why would you choose Kafka over RabbitMQ?**
   *Choose Kafka when:*
   *- You need very high throughput (millions of messages/sec)*
   *- You need message replayability (log-based architecture)*
   *- You're doing event sourcing or stream processing*
   *- You need long-term retention (days/weeks)*
   *Choose RabbitMQ when:*
   *- You need complex routing rules*
   *- You need lower latency (sub-millisecond)*
   *- You're doing task queues/work distribution*
   *- You need priority queues or delayed messages*

3. **What is a dead letter queue and why is it important?**
   *A DLQ is a special queue that receives messages that repeatedly fail processing. Importance:*
   *1. Prevents poison pills from blocking the main queue*
   *2. Allows debugging of problematic messages*
   *3. Enables manual intervention and retry mechanisms*
   *4. Maintains visibility into system failures*
   *Without DLQ: Failed messages either get retried forever or are lost.*

4. **How does Kafka preserve message ordering?**
   *Kafka preserves ordering **within partitions only**. Messages with the same key always go to the same partition (via hash(key) % partitions). Within that partition, messages are strictly ordered by offset. If you need global ordering, use a single partition (but lose parallelism).*

## 📊 **Real-World Queue Implementations**
- **LinkedIn**: Built Kafka internally, now open-source
- **Netflix**: Uses Kafka for 1 trillion+ messages/day
- **Uber**: Kafka for driver location updates (billions/day)
- **Twitter**: Early adopter of message queues for tweets
- **Airbnb**: Uses RabbitMQ for async task processing

---

# **DAY 7: PRACTICE DAY - DESIGN A URL SHORTENER**

## 🎯 **Today's Mission**
Apply everything you've learned to design a complete system - the classic URL shortener (like Bitly, TinyURL).

## ⏰ **Daily Schedule (3 Hours - Practice Intensive)**

### **PART 1: REQUIREMENTS GATHERING (30 minutes)**

**The Problem**: Design a service that takes long URLs and returns short URLs.
Example: `https://www.amazon.com/very-long-product-url...` → `https://short.ly/abc123`

**Step 1: Clarify Requirements (Interview Style)**

**Ask these questions** (imagine you're in an interview):

1. **"What's the scale we're designing for?"**
    - Daily active users? (Assume: 100M DAU)
    - URL shortening rate? (Assume: 100M URLs/day)
    - Read:shorten ratio? (Assume: 100:1 - 10B redirects/day)

2. **"What features should it support?"**
    - Shorten long URLs
    - Redirect short→long URLs
    - Custom short URLs? (Optional)
    - Analytics? (Click counts, referrers)
    - URL expiration? (Optional)
    - User accounts? (Optional for basic)

3. **"Non-functional requirements?"**
    - Availability: 99.99% (redirects must work)
    - Latency: < 100ms for redirects
    - Durability: Never lose a shortened URL
    - Unpredictable: Short URLs should be hard to guess

**Step 2: Write Down Assumptions**
Based on the answers, document:
```
Scale:
- 100M URLs/day = ~1,200 URLs/sec (peaks 5x = 6K/sec)
- 10B redirects/day = ~115K redirects/sec (peaks 5x = 575K/sec)
- Storage: 100M/day × 365 × 5 years = ~182B URLs
- Each URL: 500 bytes = 91TB total

Features (Priority):
1. Shorten URL (required)
2. Redirect (required)  
3. Analytics (important)
4. Custom URLs (nice-to-have)
5. Expiration (optional)
6. User accounts (optional)
```

### **PART 2: HIGH-LEVEL DESIGN (60 minutes)**

**Step 1: API Design**
```python
# 1. Create short URL
POST /api/v1/shorten
{
  "long_url": "https://example.com/very-long-path",
  "custom_alias": "optional_custom",  # Optional
  "user_id": "optional_user_id",      # Optional
  "expires_at": "2025-12-31"          # Optional
}

Response:
{
  "short_url": "https://short.ly/abc123",
  "created_at": "2024-01-01T10:00:00Z"
}

# 2. Redirect (not an API, but web endpoint)
GET /abc123  # Returns 301 redirect to long URL

# 3. Get analytics
GET /api/v1/analytics/abc123
Response: {"clicks": 12345, "referrers": [...], "timeline": [...]}
```

**Step 2: System Components Diagram**
Draw this in your notebook:
```
[Users] 
     |
[Load Balancer] (Day 1: Round Robin/Least Connections)
     |
[Web Servers] (Stateless, auto-scaling)
     |           |
[Shorten Service] [Redirect Service]
     |                    |
[ID Generator]     [Cache (Day 2: Redis)]
     |                    |
[Database]          [Database]
     |
[Analytics Pipeline] → [Data Warehouse]
```

**Step 3: Data Flow**
1. **Shorten Flow**:
   ```
   Client → LB → Web Server → Check custom alias → Generate unique ID → 
   Store (ID, long_url, metadata) in DB → Return short URL
   ```

2. **Redirect Flow**:
   ```
   Client → LB → Web Server → Check cache → If miss, read from DB → 
   Store in cache → Return 301 redirect
   ```

3. **Analytics Flow**:
   ```
   Redirect → Log event → Message Queue (Day 6: Kafka) → 
   Analytics Processor → Update counters → Data Warehouse
   ```

### **PART 3: DEEP DIVE - KEY DECISIONS (60 minutes)**

**Problem 1: How to generate short URLs?**
Options:
1. **Base62 Encoding of Auto-increment ID**
    - Simple: ID = 1 → "a", ID = 2 → "b", ...
    - Problem: Predictable (users can guess other URLs)
    - Solution: Use distributed ID generator (Snowflake)

2. **Hash-based (MD5/SHA)**
    - Hash(long_url) → take first 6 chars
    - Problem: Collisions (same short for different longs)
    - Solution: Check DB, if exists, add salt and retry

3. **Random Strings**
    - Generate random 6-char string
    - Check DB for uniqueness
    - Probability of collision low with 62^6 possibilities

**Your Decision**:
```
I choose: Base62 encoding of distributed ID (Twitter Snowflake)
Why:
1. Guaranteed uniqueness
2. No collisions to handle
3. Sequential IDs helpful for database sharding
4. Add randomness by encoding in different base or shuffling
```

**Problem 2: Database Schema Design**
Design tables:
```sql
-- URLs table (Day 3: SQL or NoSQL?)
CREATE TABLE urls (
    id BIGINT PRIMARY KEY,           -- Snowflake ID
    short_code VARCHAR(10) UNIQUE,   -- "abc123"
    long_url TEXT NOT NULL,          -- Original URL
    user_id BIGINT,                  -- Optional user
    created_at TIMESTAMP,
    expires_at TIMESTAMP,            -- Optional
    click_count BIGINT DEFAULT 0
);

-- Analytics table (separate for scaling)
CREATE TABLE analytics (
    id BIGINT PRIMARY KEY,
    short_code VARCHAR(10),
    timestamp TIMESTAMP,
    ip_address VARCHAR(45),
    user_agent TEXT,
    referrer TEXT
);

-- Indexes:
CREATE INDEX idx_short_code ON urls(short_code);
CREATE INDEX idx_user_id ON urls(user_id);
CREATE INDEX idx_analytics_short_time ON analytics(short_code, timestamp);
```

**Problem 3: SQL vs NoSQL Choice**
```
Choice: Use both!
- SQL (PostgreSQL) for URL mappings (need strong consistency)
- NoSQL (Cassandra) for analytics (write-heavy, time-series)
Why: Right tool for each job
```

**Problem 4: Caching Strategy (Day 2)**
```
Cache: Redis with LRU eviction
Pattern: Cache-Aside (lazy loading)
TTL: 24 hours for popular URLs
Hot URLs: Always in cache (celebrity problem from Day 5)
Cache size: Estimate 1M URLs × 1KB = 1GB RAM
```

**Problem 5: Scaling the Database (Day 5)**
```
Sharding strategy: By short_code range
Why: Even distribution, redirects lookup by short_code
Replication: Master-slave, 1 master + 2 replicas per shard
Number of shards: Start with 10, each handling ~10K writes/sec
```

### **PART 4: COMPLETE ARCHITECTURE (45 minutes)**

**Final Architecture Diagram**:
Draw this comprehensive diagram:
```
┌─────────────────────────────────────────────────────────────┐
│                     CLIENT REQUESTS                          │
└────────────────┬────────────────────────────────────────────┘
                 │
         ┌───────▼────────┐
         │  Load Balancer │ (Multiple AZs)
         └───────┬────────┘
                 │
    ┌────────────┼───────────────────┐
    │            │                   │
┌───▼───┐   ┌───▼───┐         ┌─────▼─────┐
│Web Tier│   │Web Tier│   ...  │  Web Tier │ (Auto-scaled)
│(Nginx) │   │(Nginx) │         │ (Nginx)   │
└───┬───┘   └───┬───┘         └─────┬─────┘
    │           │                   │
┌───▼───┐   ┌───▼───┐         ┌─────▼─────┐
│App     │   │App     │   ...  │  App      │ (Stateless)
│Server  │   │Server  │         │ Server    │
└───┬───┘   └───┬───┘         └─────┬─────┘
    │           │                   │
    └───────────┼───────────────────┘
                │
    ┌───────────▼───────────────────────────┐
    │          Service Layer                │
    ├───────────────────────────────────────┤
    │  • Shorten Service  (writes to DB)    │
    │  • Redirect Service (reads cache/DB)  │
    │  • Analytics Service (writes to Kafka)│
    └───────────┬───────────────────────────┘
                │
    ┌───────────▼───────────────────────────┐
    │          Cache Layer                  │
    │          Redis Cluster                │
    │      (In-memory URL mappings)         │
    └───────────┬───────────────────────────┘
                │
    ┌───────────▼───────────────────────────┐
    │         Database Layer                │
    ├───────────────────────────────────────┤
    │  • PostgreSQL (sharded) - URL mappings│
    │  • Cassandra - Analytics data         │
    │  • Replicated across 3 AZs            │
    └───────────┬───────────────────────────┘
                │
    ┌───────────▼───────────────────────────┐
    │       Analytics Pipeline              │
    ├───────────────────────────────────────┤
    │  Kafka → Spark/Flink → Data Warehouse │
    │  → Dashboard (click analytics)        │
    └───────────────────────────────────────┘
```

**Data Flow Details**:

1. **Shorten Request**:
   ```
   Client → LB → Web → App → Generate ID (Snowflake) → 
   Base62 encode → Check custom alias conflict → 
   Write to PostgreSQL → Return short URL
   ```

2. **Redirect Request**:
   ```
   Client → LB → Web → App → Check Redis cache → 
   If hit: Return 301 redirect
   If miss: Read PostgreSQL → Store in Redis → Return 301
   ```

3. **Analytics Collection**:
   ```
   During redirect: Log event → Kafka topic → 
   Spark Streaming → Update counters in Cassandra → 
   Also store raw events in S3 for batch processing
   ```

### **PART 5: SCALING NUMBERS & COST (30 minutes)**

**Estimation Exercise**:

1. **Traffic Estimates**:
   ```
   Shorten requests: 6K/sec (peak)
   Redirect requests: 575K/sec (peak)
   ```

2. **Web Tier**:
   ```
   Each server: 10K requests/sec
   Needed: 575K / 10K = 58 servers
   Auto-scale: 10-100 servers
   Cost: ~$5K/month
   ```

3. **Cache Tier**:
   ```
   Cache hit rate: 90%
   Cache misses: 57.5K/sec
   Each Redis: 50K ops/sec
   Needed: 2 Redis clusters (primary+replica)
   Memory: 100GB each (1M hot URLs)
   Cost: ~$3K/month
   ```

4. **Database Tier**:
   ```
   PostgreSQL writes: 6K/sec (shorten)
   PostgreSQL reads: 57.5K/sec (redirect misses)
   Shards: 10 shards = 600 writes/shard, 5.75K reads/shard
   Each shard: Master + 2 replicas = 30 instances
   Storage: 100TB total = 3.3TB/shard
   Cost: ~$15K/month
   ```

5. **Analytics Tier**:
   ```
   Kafka: 575K events/sec = 10 brokers
   Spark: 20-node cluster
   Cassandra: 20 nodes for analytics
   Cost: ~$10K/month
   ```

**Total Monthly Cost**: ~$33K/month

**Optimizations**:
1. Use CDN for redirects? (Cloudflare could help)
2. Read replicas for PostgreSQL to handle read load
3. More aggressive caching (95% hit rate possible)

---

## 📝 **DAY 7 OUTPUT REQUIRED**

**Complete in your notebook:**

1. **URL Shortener Design Document**:
```
Requirements:
- Scale: 100M URLs/day, 10B redirects/day
- Availability: 99.99%
- Latency: < 100ms redirects

Architecture:
- Web Tier: Auto-scaling load balancers + app servers
- Cache: Redis with 90% hit rate
- DB: PostgreSQL sharded by short_code
- Analytics: Kafka + Spark + Cassandra
- ID Generation: Twitter Snowflake algorithm

APIs:
1. POST /shorten → returns short URL
2. GET /:code → 301 redirect
3. GET /analytics/:code → click stats
```

2. **Scaling Strategy Table**:
```
| Component     | Peak Load    | Solution                    | Instances |
|---------------|--------------|-----------------------------|-----------|
| Web Servers   | 575K req/sec | Auto-scale (10-100)         | 58        |
| Redis Cache   | 57.5K miss/s | Redis Cluster               | 6 nodes   |
| PostgreSQL    | 6K write/s   | 10 shards × 3 replicas      | 30        |
| Kafka         | 575K msg/s   | 10 brokers                  | 10        |
| Analytics DB  | 575K write/s | Cassandra 20 nodes          | 20        |
```

3. **Trade-offs Documented**:
```
1. ID Generation: Snowflake vs Hash-based
   - Chose Snowflake: Guaranteed uniqueness, predictable scaling
   - Trade-off: Slightly more complex than hashing

2. Database: SQL vs NoSQL for mappings
   - Chose SQL: Need strong consistency for URL mappings
   - Trade-off: Harder to scale than NoSQL

3. Caching: Write-through vs Cache-aside
   - Chose Cache-aside: Simpler, only cache popular URLs
   - Trade-off: Cache miss penalty on first access
```

4. **Failure Scenarios & Solutions**:
```
1. Cache failure: Continue with DB (slower but works)
2. DB shard failure: Read replicas take over
3. ID generator failure: Multiple generators with different datacenter IDs
4. Analytics pipeline failure: Store raw logs, reprocess later
```

---

## 🧪 **Day 7 Self-Assessment**

**Rate your design (1-5 scale)**:
1. **Completeness**: Did you consider all components? ____
2. **Scalability**: Will it handle the required load? ____
3. **Practicality**: Could this be built and maintained? ____
4. **Cost Efficiency**: Is the cost reasonable? ____
5. **Interview Readiness**: Could you explain this in 45 min? ____

**Common Interview Questions for URL Shortener**:
1. "How would you prevent users from creating inappropriate short URLs?"
2. "What if someone creates millions of short URLs to exhaust your storage?"
3. "How would you handle custom short URL conflicts?"
4. "How would you detect and block spam/malicious URLs?"

**Think about your answers to these!**

---

## 🚀 **Day 8 Preview: MICROSERVICES & API DESIGN**

**Preparation Thought:**
- Our URL shortener was a monolithic design. What if we need to scale teams?
- How do companies like Amazon, Netflix break systems into independent services?
- Think: Why does Amazon's checkout work even if recommendations are down?

---

## 💪 **Accountability Check**

**Reply with:**
1. ✅ I've completed Day 7 design challenge
2. 🎯 Most challenging part of the design: _____
3. 💡 One optimization you'd add: _____
4. ❓ Question about the URL shortener design: _____

**Congratulations!** You've designed your first complete system from scratch. This is exactly what FAANG interviews test!

---

**Ready to dive into microservices tomorrow?** This is how modern systems are built at scale! 🚀