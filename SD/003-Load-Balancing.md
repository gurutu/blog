# **DAY 1: LOAD BALANCING - Your System Design Journey Begins**

## 🎯 **Today's Mission**
Master load balancing concepts - the foundation of scalable systems.

---

## ⏰ **Daily Schedule (2-3 Hours)**

### **Morning/Evening Block (Choose what works for you)**

#### **BLOCK 1: LEARNING CORE CONCEPTS (60 minutes)**

**0-15 min: Warm-up & Mindset**
- Watch this 5-min motivation: [System Design Basics](https://youtu.be/i7twT3x5yv8)
- Set intention: "Today I'm learning to distribute traffic efficiently"
- Open your notebook/digital notes

**15-45 min: Core Learning**
**READ/WATCH (Choose one path):**

*Option A (Visual Learners)*:
- Watch: ["Load Balancing Explained in 10 Minutes"](https://youtu.be/7OZbVQIr1eM) (10 min)
- Watch: ["Consistent Hashing Simplified"](https://youtu.be/UF9Iqmg94tk) (15 min)

*Option B (Reading Learners)*:
- Read: [Load Balancing 101](https://github.com/donnemartin/system-design-primer#load-balancer) (20 min)
- Read: [Consistent Hashing Guide](https://www.toptal.com/big-data/consistent-hashing) (15 min)

**45-60 min: Key Takeaways**
Write these in your own words:
1. **What is load balancing?** "Distributing network traffic across multiple servers..."
2. **Why is it needed?** Prevent single server overload, increase availability...
3. **Types of LBs**: Hardware vs Software (HAProxy, Nginx, AWS ELB)
4. **Placement**: Between user and web servers, between web and app servers, between app and DB

#### **BLOCK 2: ALGORITHMS DEEP DIVE (45 minutes)**

**Study these 5 algorithms (9 minutes each):**

1. **Round Robin** ✅ Simplest
    - Each request goes to next server in list
    - Pro: Simple, evenly distributed
    - Con: Doesn't consider server load
    - Draw: A circle with servers and arrows rotating

2. **Least Connections** 🏆 Most practical
    - Sends to server with fewest active connections
    - Pro: Accounts for current load
    - Con: Need to track connections
    - Draw: Servers with connection counts

3. **IP Hash** 🔗 Session persistence
    - Hash(client IP) % number of servers
    - Pro: Same client → same server (session sticky)
    - Con: Poor distribution if IPs similar
    - Draw: IP → Hash function → Server mapping

4. **Weighted Round Robin** ⚖️ For uneven servers
    - Servers get weights (CPU/memory based)
    - Stronger servers get more requests
    - Draw: Servers with different sizes

5. **Consistent Hashing** 🎯 For horizontal scaling
    - Servers on hash ring
    - Request → nearest server clockwise
    - Adding/removing servers affects minimal keys
    - **CRITICAL**: Draw this 3 times until memorized

**Quick Quiz (Test yourself):**
- Which algorithm maintains session persistence? (IP Hash)
- Which is best for removing/adding servers frequently? (Consistent Hashing)
- Real web apps mostly use? (Least Connections)

#### **BLOCK 3: HANDS-ON PRACTICE (45 minutes)**

**Exercise 1: Diagram Practice (15 min)**
Draw these scenarios:
1. **Simple setup**: User → Load Balancer → 3 Web Servers
2. **Two-tier LB**: User → L7 LB → Web Servers → L4 LB → App Servers
3. **Global LB**: Users in US, Europe, Asia → GeoDNS → Regional LBs

**Exercise 2: Design Problems (15 min)**
Solve:
1. "How would you handle 10K concurrent users hitting your application?"
    - Your answer should include: Load balancer + auto-scaling group

2. "A server goes down. How does the load balancer handle it?"
    - Health checks (every 30 sec), remove from pool, notify ops

**Exercise 3: Real-world Mapping (15 min)**
Match companies to their LB approach:
- Netflix: Zuul API Gateway (dynamic routing)
- Amazon: AWS Elastic Load Balancer (multiple types)
- Google: Global Load Balancer (anycast IP)

#### **BLOCK 4: INTERVIEW FOCUS (30 minutes)**

**Common Interview Questions:**
1. "How does a load balancer work?"
    - **Your answer framework**:
      ```
      A load balancer distributes incoming network traffic across multiple servers
      to ensure no single server becomes overwhelmed. It increases availability
      and reliability of applications. For example, if we have 3 web servers,
      the LB can use algorithms like round robin or least connections to
      distribute requests. It also performs health checks and removes
      unhealthy servers from the pool.
      ```

2. "What's the difference between L4 and L7 load balancing?"
    - **L4 (Transport)**: Operates on TCP/UDP, faster, sees only IP/port
    - **L7 (Application)**: Operates on HTTP/HTTPS, can route based on URL, headers, cookies
    - **Draw**: L4 = "Traffic cop", L7 = "Smart router"

**Whiteboard Practice:**
Draw this from memory:
```
[Users]
    |
[Load Balancer] → Health Checks → Server Pool
    |               |
[Web Tier]    [Remove if unhealthy]
[App Tier]
[DB Tier]
```

**Estimation Practice:**
- If 1 server handles 1,000 req/sec, how many servers for 100,000 req/sec?
- Answer: 100 servers + 20% buffer = 120 servers
- Load balancer cost? ~$20-100/month per LB

---

## 📝 **DAY 1 OUTPUT REQUIRED**

**Complete these in your notebook:**

1. **Load Balancer Cheat Sheet**:
```
Types:
- Hardware: F5, Citrix
- Software: HAProxy, Nginx, AWS ELB

Algorithms:
1. Round Robin: Simple rotation
2. Least Connections: Most practical  
3. IP Hash: Session stickiness
4. Consistent Hashing: Minimal redistribution

Key Concepts:
- Health Checks: TCP/HTTP, every 30s
- Session Persistence: Sticky sessions
- Auto-scaling: Add/remove servers
```

2. **Diagram Repository**:
    - Save your 3 diagrams from today
    - Label: "Day 1 - Load Balancing"

3. **Interview Ready Answers**:
    - Write your polished answer to "How does a load balancer work?"
    - Time yourself: Can you explain in 2 minutes?

4. **Flashcards Created** (digital or physical):
    - Front: What is Consistent Hashing?
    - Back: Servers on hash ring, minimal redistribution on changes
    - Front: L4 vs L7 Load Balancing
    - Back: L4 = TCP/IP, L7 = HTTP aware

---

## 🔍 **Common Pitfalls to Avoid Today**

1. ❌ **Don't just read** - Draw diagrams for each concept
2. ❌ **Don't memorize** - Understand WHY each algorithm exists
3. ❌ **Don't skip practice** - Actually draw the systems
4. ✅ **Do speak aloud** - Explain concepts to yourself
5. ✅ **Do make connections** - How does this relate to websites you use?

---

## 🎯 **Success Metrics for Day 1**

**By end of today, you should be able to:**
✅ Explain load balancing to a 10-year-old
✅ Draw 3 types of load balancer setups
✅ List 3 algorithms with pros/cons
✅ Answer "Why use load balancing?" in 30 seconds
✅ Identify load balancing in real websites (Amazon, Netflix, etc.)

**Quick Self-Test:**
1. What happens when a server behind LB dies?
2. How does session stickiness work?
3. Why is consistent hashing better than modulo hashing for scaling?
4. What's the difference between active-active and active-passive?

*(Answers: 1. Health checks fail, LB stops sending traffic 2. IP hash or cookies 3. Minimal key redistribution 4. Active-active = all servers handle traffic, active-passive = backup servers)*

---

## 📚 **Optional Deep Dive (If you have extra time)**

1. **Set up actual load balancer**:
    - Install nginx: `sudo apt-get install nginx`
    - Configure simple LB: 2 backend servers on localhost:8080, 8081

2. **Code Consistent Hashing**:
    - Simple Python implementation (20 lines)
    - Add/remove servers and see key distribution

3. **Read**: AWS ELB documentation - Classic vs Application vs Network LBs

---

## 🌟 **Pro Tip for Day 1**

**Connect to Real World**:
Every time you visit a website today, think:
"Which load balancer algorithm are they using?"
"How many servers are behind it?"
"What happens if one fails?"

This builds the "systems thinking" mindset.

---

## 🚀 **Tomorrow Preview: Day 2 - CACHING**

**Quick prep for tomorrow:**
- Think about: Why is Facebook/Instagram so fast?
- Notice: When you revisit a page, it loads faster
- Question: Where does "cached" data live?

---

## 💪 **Accountability Check**

**Reply to this message with:**
1. ✅ I've completed Day 1
2. 📊 Number of diagrams drawn: _____
3. 💡 One "aha!" moment from today: _____
4. ❓ One question I still have: _____

**Remember**: Consistency beats intensity. You've started the journey - that's the hardest part! 🎉

---