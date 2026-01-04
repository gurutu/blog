# **DAY 1 ANSWERS CHECK**

Let's verify your understanding:

## ✅ **Self-Test Answers**
1. **What happens when a server behind LB dies?**
   *Answer: Health checks fail (TCP/HTTP checks every 30s), LB removes it from pool, traffic redistributed to remaining servers*

2. **How does session stickiness work?**
   *Answer: IP Hash or cookie-based routing ensures same client goes to same server for session duration*

3. **Why is consistent hashing better than modulo hashing for scaling?**
   *Answer: Modulo hashing (server = hash(key) % N) redistributes ALL keys when servers added/removed. Consistent hashing redistributes only K/N keys (where K=keys, N=servers)*

4. **What's the difference between active-active and active-passive?**
   *Answer: Active-active = all servers handle traffic simultaneously (better utilization). Active-passive = backup servers only take over when primary fails (simpler, cheaper)*

## 📊 **Expected Diagrams**
You should have drawn:
1. Simple 3-tier with LB
2. Two-tier LB architecture
3. Global load balancing with GeoDNS

## 🎯 **Day 1 Success Checklist**
If you can answer these, you're ready for Day 2:
- [ ] Can explain load balancing to a non-technical person
- [ ] Can name 3+ algorithms with use cases
- [ ] Understands health checks and failover
- [ ] Knows L4 vs L7 differences

---

# **DAY 2: CACHING - Making Systems Blazing Fast**

## 🎯 **Today's Mission**
Master caching strategies - the #1 performance optimization in system design.

## ⏰ **Daily Schedule (2-3 Hours)**

### **BLOCK 1: CORE CONCEPTS (60 minutes)**

**0-10 min: Warm-up**
- Recall: Why was load balancing important? (Scalability, availability)
- Today's focus: **Performance & Cost Reduction**
- Real-world example: Think of Amazon product pages - same product viewed millions of times

**10-40 min: Foundation Learning**
**Essential Concepts:**

1. **What is caching?**
    - Temporary storage of frequently accessed data
    - Faster access than primary data source (DB, API)
    - Location: Memory (RAM) vs Disk (SSD)

2. **Cache Hit vs Miss**
   ```
   Cache Hit: Data found in cache → Fast (1-10ms)
   Cache Miss: Data not in cache → Slow (50-1000ms)
   Hit Rate = Hits / (Hits + Misses)
   Good systems: 80-95% hit rate
   ```

3. **Where to cache?**
    - **Client-side**: Browser cache, mobile app cache
    - **Server-side**: Redis, Memcached, in-memory cache
    - **CDN**: Edge locations globally
    - **Database**: Query cache, buffer pool

4. **Read-through vs Write-through**
    - Draw these flows in your notebook!

**40-60 min: Cache Algorithms**
Study these eviction policies:

1. **LRU (Least Recently Used)** 🏆 Most Common
    - Remove oldest accessed items
    - Implementation: Doubly linked list + hash map
    - Draw: Items with access timestamps

2. **LFU (Least Frequently Used)**
    - Remove least frequently accessed
    - Good for stable access patterns
    - Draw: Items with access counters

3. **FIFO (First In First Out)**
    - Simple queue
    - Pro: Simple, predictable
    - Con: May evict hot items

4. **Random Replacement**
    - Randomly evict items
    - Pro: No metadata needed
    - Con: May evict hot items

**Memory Exercise:** Which to use when?
- Social media feed: LRU (recent posts matter)
- Product catalog: LFU (popular products stay)
- Log buffer: FIFO (old logs irrelevant)

### **BLOCK 2: CACHING PATTERNS (45 minutes)**

**Pattern 1: Cache-Aside (Lazy Loading) - MOST COMMON**
```
1. App checks cache
2. If found (cache hit): return
3. If not (cache miss): 
   a. Read from DB
   b. Write to cache
   c. Return data
```
**Draw this flow!**
- Pro: Simple, only caches accessed data
- Con: Cache miss penalty, stale data possible

**Pattern 2: Write-Through**
```
1. Write to cache AND DB simultaneously
2. Cache always consistent with DB
```
- Pro: Always consistent
- Con: Write latency (wait for both)

**Pattern 3: Write-Back (Write-Behind)**
```
1. Write to cache only
2. Cache asynchronously writes to DB later
```
- Pro: Fast writes
- Con: Risk of data loss if cache fails

**Pattern 4: Refresh-Ahead**
- Proactively refresh cache before expiration
- Used in: Stock prices, weather data

**Practice Question:**
"Design cache for Twitter timeline"
- Which pattern? Cache-Aside (tweets don't change often)
- Eviction policy? LRU (users want recent tweets)
- Cache location? Edge (CDN) for media, Redis for timelines

### **BLOCK 3: REAL-WORLD SYSTEMS (45 minutes)**

**Case Study 1: Facebook's Memcached**
- 28+ TB cache clusters
- Multi-level: Local → Regional → Global
- Invalidation: Web servers invalidate cache on write
- **Key insight**: They cache *everything* possible

**Case Study 2: CDN Caching (Cloudflare, Akamai)**
- Static assets: Images, CSS, JS
- Dynamic content: API responses (harder)
- TTL (Time To Live): How long to cache
- Cache Key: URL + headers + cookies

**Case Study 3: Database Query Cache**
- MySQL query cache (deprecated)
- Redis as cache layer
- Materialized views (pre-computed queries)

**Hands-on Exercise:**
Design cache for:
1. **E-commerce product page**
    - What to cache? Product info, reviews, images
    - Where? CDN for images, Redis for product data
    - TTL? Product data: 5 min, Reviews: 1 hour

2. **News website homepage**
    - Cache entire HTML? Yes, for anonymous users
    - Personalization? Cache fragments, assemble dynamically
    - Invalidation? On new article publish

### **BLOCK 4: INTERVIEW FOCUS (30 minutes)**

**Common Questions & Answers:**

1. **"When would you NOT use caching?"**
   ```
   Answer: When data...
   1. Changes very frequently (stock trades)
   2. Is unique per user (personal dashboard)
   3. Must be 100% real-time (bank balance)
   4. Size exceeds available cache memory
   ```

2. **"Cache invalidation strategies?"**
   ```
   1. TTL (Time To Live): Simple, eventual consistency
   2. Write-through: Always consistent
   3. Invalidation on write: Delete cache on DB update
   4. Versioning: Cache key includes version number
   ```

3. **"Cache penetration, breakdown, avalanche?"**
    - **Penetration**: Frequent misses for non-existent data
      *Solution: Cache null values*
    - **Breakdown**: Single cache failure → DB overload
      *Solution: Replication, fallback*
    - **Avalanche**: Cache mass expiration → DB storm
      *Solution: Random TTL, warm-up cache*

**Whiteboard Practice:**
Draw multi-level cache architecture:
```
[Client] → [CDN] → [Load Balancer] → [App Server with Local Cache] 
                                     → [Distributed Cache (Redis Cluster)]
                                     → [Database]
```

**Estimation Practice:**
- Twitter: 500M tweets/day, 20% cached → Cache size?
- 140 chars = ~200 bytes × 100M tweets = 20GB cache
- Cost: Redis Cloud ~$250/month for 25GB

---

## 📝 **DAY 2 OUTPUT REQUIRED**

**Complete in your notebook:**

1. **Cache Pattern Cheat Sheet**:
```
1. Cache-Aside (Lazy): App manages cache
   Use: Read-heavy, data doesn't change often

2. Write-Through: Write to cache+DB
   Use: Consistency critical (banking)

3. Write-Back: Write to cache, async to DB  
   Use: Write-heavy, can tolerate some loss

4. Write-Around: Write to DB, not cache
   Use: Write-once, read-later data
```

2. **Eviction Policy Matrix**:
```
| Policy | Pros | Cons | Best For |
|--------|------|------|----------|
| LRU    | Simple, tracks recency | Needs metadata | General purpose |
| LFU    | Keeps frequent items | Expensive counters | Stable patterns |
| FIFO   | No metadata | Evicts hot items | Logs, streams |
```

3. **Design Problem Solution**:
   Problem: "Cache for Instagram feed"
   Your solution should include:
    - Cache location: Edge + application layer
    - Pattern: Cache-Aside with TTL
    - Eviction: LRU (users want recent posts)
    - Size estimate: 1M users × 50 posts × 10KB = 500GB

4. **Flashcards Created**:
    - Front: Cache Hit vs Miss
    - Back: Hit = cache has data (fast), Miss = fetch from DB (slow)

    - Front: Cache Penetration vs Avalanche
    - Back: Penetration = frequent misses for non-existent data, Avalanche = mass cache expiration

---

## 🧪 **Day 2 Self-Test**

**Answer without looking:**
1. What's the difference between write-through and write-back?
2. When would you choose LFU over LRU?
3. How do you prevent cache avalanche?
4. What's a good cache hit rate for web applications?

**Answers:**
1. Write-through: sync write to cache+DB. Write-back: async write to DB
2. LFU when access patterns stable (product catalog). LRU when recency matters (social feed)
3. Random TTL values, warm cache before expiration, circuit breakers
4. 80-95% hit rate is good

---

## 🚀 **Day 3 Preview: DATABASES - SQL**

**Preparation Thought:**
- Why do some systems use SQL (MySQL) while others use NoSQL (MongoDB)?
- Think about: Amazon's product catalog vs WhatsApp's messages

---

## 💪 **Accountability Check**

**Reply with:**
1. ✅ I've completed Day 2
2. 🔥 Cache pattern I found most interesting: _____
3. 📈 Real system using caching I analyzed today: _____
4. ❓ Question about caching: _____

**Remember**: Caching is often the difference between a slow system and a fast one. Master this!

---
