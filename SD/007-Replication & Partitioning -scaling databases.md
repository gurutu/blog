# **DAY 4 ANSWERS CHECK**

## ✅ **Self-Test Answers**

1. **What does CAP theorem state about distributed systems?**
   *In any distributed system, you can only guarantee two of these three properties simultaneously: Consistency (all nodes see the same data at the same time), Availability (every request receives a response), and Partition Tolerance (the system continues operating despite network failures). This means in practice: SQL databases typically choose CP (Consistency + Partition Tolerance), NoSQL databases often choose AP (Availability + Partition Tolerance), and CA systems don't exist in distributed environments because partitions are inevitable.*

2. **What are the four types of NoSQL databases?**
   *1. **Key-Value Stores**: Redis, DynamoDB - Simple key→value mapping, great for caching and sessions.*
   *2. **Document Databases**: MongoDB, Couchbase - Store JSON/BSON documents, flexible schemas.*
   *3. **Column-Family Stores**: Cassandra, HBase - Wide-column stores optimized for writes and scale.*
   *4. **Graph Databases**: Neo4j, Amazon Neptune - Optimized for relationship-heavy data (social networks).*

3. **When is eventual consistency acceptable?**
   *Eventual consistency is acceptable when:*
   *- Immediate data synchronization isn't critical (social media likes/comments)*
   *- The system prioritizes availability over consistency (during network partitions)*
   *- Users can tolerate temporary inconsistencies (shopping cart across devices)*
   *- The use case involves append-only or immutable data (activity feeds, logs)*
   *Real-world example: When you 'like' a Facebook post, it's okay if some users see the updated count a few seconds later.*

4. **Why is Cassandra good for time-series data?**
   *Cassandra excels with time-series data because:*
   *1. **Write-optimized architecture**: Sequential writes to commit log, minimal disk seeks*
   *2. **Linear scalability**: Add nodes to handle increased write volume*
   *3. **Time-based partitioning**: Natural fit for time-series bucketing strategies*
   *4. **Efficient range queries**: With proper clustering keys, can query time ranges efficiently*
   *5. **High availability**: Multi-datacenter support for global deployments*

## 📊 **Real-World NoSQL Usage**
- **Facebook**: Cassandra for messages, TAO for social graph
- **Netflix**: Cassandra for viewing history
- **Amazon**: DynamoDB for shopping cart
- **Uber**: Cassandra for trip data
- **Instagram**: Cassandra for direct messages

---

# **DAY 5: REPLICATION & PARTITIONING - SCALING DATABASES**

## 🎯 **Today's Mission**
Master database replication and partitioning - the techniques that make databases survive failures and handle massive scale.

## ⏰ **Daily Schedule (2-3 Hours)**

### **BLOCK 1: REPLICATION FUNDAMENTALS (60 minutes)**

**0-10 min: Warm-up**
- Single database server = single point of failure
- What if it crashes? Data loss, downtime
- Today: **Redundancy** and **Distribution** strategies

**10-40 min: Replication Strategies**

**Why Replicate?**
1. **High Availability**: Survive machine failures
2. **Read Scaling**: Distribute read traffic
3. **Geographic Distribution**: Serve users closer to data
4. **Disaster Recovery**: Survive datacenter outages

**Replication Models:**

**1. Single Leader (Master-Slave) - MOST COMMON**
```
One primary (leader) for writes
Multiple secondaries (followers) for reads
Writes: Leader → Replication → Followers
Reads: Can read from followers
```
**Draw**: Leader with arrows to multiple followers

**How it works**:
- Synchronous: Wait for followers to confirm (strong consistency)
- Asynchronous: Don't wait (better performance, risk of data loss)
- Semi-synchronous: Wait for at least N followers

**Failure Handling**:
- Leader fails: **Failover** to a follower
- Challenges: Split brain, replication lag

**2. Multi-Leader (Master-Master)**
```
Multiple leaders, all accept writes
Writes replicated between leaders
Use: Multiple datacenters, offline-first apps
```
**Problems**:
- Write conflicts (same data modified in two places)
- Example: Google Docs, mobile apps syncing

**3. Leaderless (Dynamo-style)**
```
Any node can accept writes
Quorum: W = write quorum, R = read quorum
Rule: W + R > N for consistency
```
**Example**: DynamoDB, Cassandra

**40-60 min: Consistency Models in Replication**

**Problem**: Replication lag causes consistency issues
- User writes to leader, reads from lagging follower → sees old data

**Solutions**:

1. **Read Your Writes Consistency**
    - After writing, ensure reads see that write
    - Implementation: Remember timestamp of last write, read from replica that caught up

2. **Monotonic Reads**
    - User never sees data moving backward in time
    - Implementation: Always read from same replica

3. **Consistent Prefix Reads**
    - See writes in the order they happened
    - Critical for: Chat messages, financial transactions

**Visualize these problems**:
- Draw timeline showing write→replication lag→stale read
- Show how different consistency models fix it

### **BLOCK 2: PARTITIONING/SHARDING (45 minutes)**

**The Problem**: One database server can't handle:
- Too much data (> storage capacity)
- Too many queries (> CPU/RAM)
- Too many connections (> network limits)

**Solution**: **Sharding** - Split data across multiple machines

**Sharding Strategies**:

**1. Key-Based (Hash) Partitioning**
```
shard = hash(key) % number_of_shards
Example: user_id 123 → hash(123) % 10 → shard 3
```
**Pros**: Even distribution
**Cons**: Adding/removing shards requires rehashing (expensive!)

**2. Range-Based Partitioning**
```
Assign ranges to shards
Shard 1: user_id 1-1M
Shard 2: user_id 1M-2M
```
**Pros**: Range queries efficient
**Cons**: Hotspots (all new users to last shard)

**3. Directory-Based Partitioning**
```
Lookup service tracks key → shard mapping
Can move data between shards easily
```
**Pros**: Flexible, easy rebalancing
**Cons**: Single point of failure (the directory)

**4. Geographic Partitioning**
```
Users in US → US shard
Users in EU → EU shard
```
**Pros**: Data locality, compliance (GDPR)
**Cons**: Cross-region queries slow

**The Rebalancing Problem**:
When adding new shards, need to move data
Strategies:
- **Fixed number of partitions**: Create many small partitions, assign multiple to each shard
- **Dynamic partitioning**: Like B-tree, split partitions when they get too big
- **Consistent hashing**: Minimal data movement (learned in Day 1!)

**Hands-on Exercise**:
Design sharding for Twitter:
- 500M users, 500B tweets
- Strategy: Shard by user_id range
- Challenge: Celebrity users (Katy Perry: 100M followers)
- Solution: Special handling for hot partitions

### **BLOCK 3: REAL-WORLD PATTERNS & TRADEOFFS (45 minutes)**

**Case Study 1: Facebook's TAO System**
- Problem: Social graph too big for any database
- Solution: Geographical sharding + caching layer
- Architecture: Read-through cache with MySQL backing
- Scale: Trillions of associations

**Case Study 2: Uber's Trip Data**
- Problem: Millions of concurrent rides, need real-time location
- Solution: Time-based partitioning + geospatial indexing
- Sharding: By city_id + time range
- Challenge: Cross-city trips

**Case Study 3: YouTube's Video Metadata**
- Problem: Billions of videos, viral videos get millions of views
- Solution: Consistent hashing for distribution + caching layer
- Hot video handling: Multi-level caching (edge → regional → global)

**Design Patterns for Hot Partitions**:

1. **Vertical Partitioning**
   ```
   Hot columns (likes, views) in separate table
   Cold columns (description, tags) in another table
   ```

2. **Write De-amplification**
   ```
   Instead of: increment counter for every view
   Do: Batch increments every N seconds
   ```

3. **Materialized Views**
   ```
   Pre-compute expensive queries
   Update asynchronously
   ```

**Anti-Patterns Alert**:
- ❌ Don't shard too early (premature optimization)
- ❌ Don't use joins across shards (very expensive)
- ❌ Don't forget about distributed transactions
- ✅ Do plan for rebalancing from day 1
- ✅ Do monitor shard distribution regularly

### **BLOCK 4: INTERVIEW FOCUS (30 minutes)**

**Common Questions & Answers:**

1. **"How would you design a database for 1 billion users?"**
   ```
   1. Sharding strategy: By user_id range or hash
   2. Replication: Master-slave for each shard
   3. Caching: Redis for hot user data
   4. Monitoring: Shard health, automatic rebalancing
   5. Special handling: Celebrity users (separate shards/caching)
   
   Example numbers:
   - 1000 shards = 1M users/shard
   - Each shard: Master + 2 replicas
   - Total: 3000 database instances
   ```

2. **"What happens during database failover?"**
   ```
   1. Monitoring detects primary failure
   2. Elect new leader (based on replication lag, priority)
   3. Reconfigure applications to use new leader
   4. Old leader comes back → becomes replica
   
   Challenges:
   - Split brain (two primaries)
   - Data loss (if async replication)
   - Application connection storms
   ```

3. **"How do you handle celebrity users problem?"**
   ```
   Celebrity problem: One user gets disproportionate traffic
   
   Solutions:
   1. Separate shard for celebrities
   2. Multi-level caching (edge → app → database)
   3. Write de-amplification (batch updates)
   4. Read replicas just for that user's data
   5. Materialized views pre-computed
   
   Example: Kim Kardashian's tweets served from Redis, not MySQL
   ```

**Whiteboard Practice**:
Design Instagram's photo storage and metadata:
```
Storage (Photos):
- Object storage (S3) with CDN
- Key: user_id/photo_id.jpg

Metadata (Database):
- Sharded by user_id (hash-based)
- Each shard: MySQL master + 2 replicas
- Hot users: Separate caching layer

Queries:
- User's photos: Single shard query
- Explore page: Separate analytics cluster
```

**Estimation Practice**:
- Instagram: 1B users, 100B photos
- Metadata size: 100B × 1KB = 100TB
- Shards: 1000 shards = 100GB/shard (comfortable for MySQL)
- Replicas: 3× replication = 300TB total storage
- Cost: ~$1M/year for database servers

---

## 📝 **DAY 5 OUTPUT REQUIRED**

**Complete in your notebook:**

1. **Replication vs Partitioning Matrix**:
```
| Aspect          | Replication                    | Partitioning                 |
|-----------------|--------------------------------|------------------------------|
| Primary Goal    | Availability, Read scaling     | Storage, Write scaling       |
| Data            | Same data on all nodes         | Different data on each node  |
| Failure Impact  | Data available from replicas   | Only some data unavailable   |
| Complexity      | Medium (sync issues)           | High (resharding, queries)   |
```

2. **Sharding Strategy Decision Tree**:
```
Start → Will you need range queries? → Yes → Range-based sharding
                  ↓ No
           Even distribution critical? → Yes → Hash-based sharding
                  ↓ No
           Need flexible rebalancing? → Yes → Directory-based
                  ↓ No
           Data has natural geography? → Yes → Geographic sharding
```

3. **Consistency Level Trade-offs**:
```
Strong Consistency:
+ No stale reads
- Higher latency
- Lower availability

Eventual Consistency:
+ High availability  
+ Low latency
- Temporary inconsistencies
- Harder to program against
```

4. **Flashcards Created**:
- Front: Leader vs Leaderless Replication
- Back: Leader: One node handles writes. Leaderless: Any node can write, use quorums.

- Front: What is a "hot partition"?
- Back: A shard receiving disproportionate traffic (celebrity users, popular products).

---

## 🧪 **Day 5 Self-Test**

**Answer without looking:**
1. What are the three main replication models?
2. What is the "celebrity user problem" in sharding?
3. How does consistent hashing help with sharding?
4. What's the difference between synchronous and asynchronous replication?

**Answers:**
1. Single-leader (master-slave), Multi-leader (master-master), Leaderless (Dynamo-style).
2. When one user/entity gets disproportionate traffic, overwhelming their shard (e.g., celebrity tweets, viral products).
3. When adding/removing shards, only K/N keys need to move (where K=total keys, N=shards), instead of all keys with modulo hashing.
4. Synchronous: Wait for replicas to confirm write before returning success (strong consistency). Asynchronous: Return success immediately, replicate in background (better performance, risk of data loss).

---

## 🚀 **Day 6 Preview: MESSAGE QUEUES & STREAM PROCESSING**

**Preparation Thought:**
- Databases handle storage, but how do systems communicate asynchronously?
- How does Uber notify 1000 drivers when you request a ride?
- Think: Order confirmation emails that arrive seconds after you click "buy"

---

## 💪 **Accountability Check**

**Reply with:**
1. ✅ I've completed Day 5
2. 🌍 A global service that uses geographic partitioning: _____
3. ⚖️ Most challenging trade-off in replication: _____
4. ❓ Question about sharding or replication: _____

**Remember**: Every large-scale system uses these techniques. Facebook, Google, Amazon - they all shard and replicate!

---

**Ready to learn about asynchronous communication tomorrow?** Message queues are critical for decoupling systems! 🚀