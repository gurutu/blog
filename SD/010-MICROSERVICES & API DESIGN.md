# **DAY 7 ANSWERS & SELF-ASSESSMENT GUIDE**

## 🧪 **Self-Assessment Answer Key**

**Common Interview Questions for URL Shortener:**

1. **"How would you prevent users from creating inappropriate short URLs?"**
   *Answer: Multiple layers of defense:*
   *1. Content filtering: Check against blacklists (malware, phishing databases)*
   *2. Rate limiting: Per-user/IP limits to prevent spam*
   *3. User verification: Require accounts for high-volume usage*
   *4. Reporting system: Allow users to report inappropriate URLs*
   *5. Manual review: For custom URLs or flagged content*
   *6. Automated scanning: Periodically check existing URLs against updated blacklists*

2. **"What if someone creates millions of short URLs to exhaust your storage?"**
   *Answer: Implement safeguards:*
   *1. Rate limiting: Strict per-user/IP limits (e.g., 1000 URLs/day free tier)*
   *2. Authentication: Require accounts for bulk creation*
   *3. CAPTCHA: For anonymous users*
   *4. Payment tiers: Charge for high-volume usage*
   *5. Monitoring: Alert on unusual patterns (sudden spike from single user)*
   *6. Temporary URLs: Auto-expire unused URLs after certain period*

3. **"How would you handle custom short URL conflicts?"**
   *Answer: Check database before assignment. If conflict:*
   *- For registered users: Suggest alternatives or ask to choose different*
   *- For anonymous users: Auto-generate a similar but available URL*
   *- Reserve namespace: Prevent common words/phrases as custom URLs*
   *- Premium feature: Allow reserved custom URLs for paying customers*

4. **"How would you detect and block spam/malicious URLs?"**
   *Answer: Real-time and batch processing:*
   *1. Real-time: Check against Google Safe Browsing, VirusTotal APIs*
   *2. Batch: Daily scan of all URLs against updated threat databases*
   *3. User feedback: Reporting system with quick takedown*
   *4. Machine learning: Analyze patterns in URL creation and click behavior*
   *5. Partnership: Integrate with security providers*

## 📊 **Design Evaluation Rubric**

**If your design included these, give yourself 5/5:**
- ✅ Load balancing at entry point
- ✅ Caching layer (Redis) for hot URLs
- ✅ Sharded database for URL mappings
- ✅ Separate analytics pipeline
- ✅ Distributed ID generation (Snowflake-like)
- ✅ Estimated scaling numbers
- ✅ Failure handling strategies
- ✅ API specifications

**Common Missing Pieces (Don't worry if you missed some):**
- DNS considerations (short.ly domain routing)
- CDN integration for global redirects
- Data retention policies (delete old URLs?)
- Backup/restore procedures
- Monitoring/alerting infrastructure
- Security measures (DDoS protection, authentication)

---

# **DAY 8: MICROSERVICES & API DESIGN**

## 🎯 **Today's Mission**
Master microservices architecture - how to break monoliths into scalable, independent services like Netflix and Amazon.

## ⏰ **Daily Schedule (2-3 Hours)**

### **BLOCK 1: MICROSERVICES FUNDAMENTALS (60 minutes)**

**0-10 min: Warm-up**
- Monolith: One big codebase (simple but doesn't scale with teams)
- Microservices: Many small services (complex but enables scaling)
- Today: **Decomposition** and **Independence** patterns

**10-40 min: Why Microservices?**

**The Monolith Problem**:
1. **Scaling**: Must scale entire app even if only one feature is busy
2. **Deployment**: Small change requires redeploying everything
3. **Technology Lock-in**: One language/framework for everything
4. **Team Coordination**: Everyone works on same codebase
5. **Failure Isolation**: One bug can bring down entire system

**Microservices Solution**:
```
Principle: Single Responsibility per service
Benefit: Each service can be:
- Developed independently
- Deployed independently  
- Scaled independently
- Written in different technologies
```

**Key Characteristics**:
1. **Loosely Coupled**: Services communicate via APIs, not shared code
2. **Independently Deployable**: Deploy one service without affecting others
3. **Organized Around Business Capabilities**: Not technical layers
4. **Own Their Data**: Each service has its own database

**Draw**: Monolith vs Microservices architecture comparison

**40-60 min: Service Decomposition Patterns**

**How to Split a Monolith**:
1. **By Business Domain** (DDD - Domain Driven Design)
   ```
   E-commerce example:
   - User Service: Authentication, profiles
   - Product Service: Catalog, inventory  
   - Order Service: Orders, payments
   - Shipping Service: Delivery, tracking
   ```

2. **By Data Ownership**
   ```
   Rule: If services frequently query same data together → keep together
   Example: User and UserPreferences should be same service
   ```

3. **By Volatility**
   ```
   Split frequently changing parts from stable parts
   Example: Recommendation engine (changes often) vs Product catalog (stable)
   ```

4. **By Team Structure** (Conway's Law)
   ```
   "Organizations design systems that mirror their communication structure"
   If you have separate frontend/backend teams → split accordingly
   ```

**Decomposition Exercise**: Split our URL shortener:
```
Original monolith:
- URL shortening logic
- Redirect logic  
- Analytics processing
- User management
- Admin dashboard

Microservices:
1. Shortening Service (core business logic)
2. Redirect Service (high traffic, needs low latency)
3. Analytics Service (data processing heavy)
4. User Service (authentication, profiles)
5. Admin Service (dashboard, reporting)
```

### **BLOCK 2: INTER-SERVICE COMMUNICATION (45 minutes)**

**Communication Styles**:

1. **Synchronous (Request-Response)**
   ```
   HTTP/REST: Most common
   gRPC: Faster, binary, supports streaming
   GraphQL: Client specifies needed data
   ```

2. **Asynchronous (Event-Driven)**
   ```
   Message Queues (Day 6): Kafka, RabbitMQ
   Pattern: Publish events, other services react
   ```

**Synchronous Communication Trade-offs**:
```
Pros:
- Simple to understand
- Immediate feedback
- Easy error handling

Cons:
- Creates temporal coupling (both services must be up)
- Latency adds up (chain of calls)
- Can cause cascading failures
```

**REST API Design Principles**:

1. **Resource-Oriented**
   ```http
   GET    /users/{id}        # Retrieve
   POST   /users             # Create  
   PUT    /users/{id}        # Update whole
   PATCH  /users/{id}        # Update partial
   DELETE /users/{id}        # Delete
   ```

2. **Stateless**: Each request contains all needed information
3. **HATEOAS**: Include links to related resources
4. **Versioning**: In URL (`/v1/users`) or headers

**gRPC Advantages**:
- Protocol Buffers: Smaller, faster than JSON
- Bi-directional streaming
- Built-in code generation
- HTTP/2 based (multiplexing)

**Example**: User service API
```protobuf
// Protobuf definition
service UserService {
  rpc GetUser(GetUserRequest) returns (User);
  rpc CreateUser(CreateUserRequest) returns (User);
  rpc UpdateUser(UpdateUserRequest) returns (User);
}

message User {
  string id = 1;
  string email = 2;
  string name = 3;
}
```

**API Gateway Pattern**:
```
Single entry point that:
1. Routes requests to appropriate services
2. Aggregates responses
3. Handles authentication/authorization
4. Rate limiting
5. Request/response transformation
```

**Draw**: Client → API Gateway → [Service1, Service2, Service3]

### **BLOCK 3: DATA MANAGEMENT & CONSISTENCY (45 minutes)**

**The Biggest Challenge**: Data consistency across services

**Problem**: In monolith, one database with ACID transactions
```sql
BEGIN TRANSACTION;
UPDATE inventory SET quantity = quantity - 1;
INSERT INTO orders VALUES (...);
COMMIT;
```

**In Microservices**: Each service has its own database
- Can't use database transactions across services
- Need distributed transaction patterns

**Solutions**:

1. **Saga Pattern** (Most Common)
   ```
   Sequence of local transactions
   Each step publishes event triggering next step
   If any step fails: Execute compensating transactions
   
   Example: Order Saga
   1. Order Service: Create order (pending)
   2. Inventory Service: Reserve items
   3. Payment Service: Charge credit card
   4. If payment fails: Inventory Service: Unreserve items
   ```

2. **Two-Phase Commit (2PC)**
   ```
   Coordinator prepares all services
   If all ready → commit, else → abort
   Problem: Blocking, single point of failure
   ```

3. **Event Sourcing**
   ```
   Store state changes as events
   Services react to events
   Example: OrderCreated, PaymentProcessed, ItemShipped
   ```

**Database Per Service Implications**:
```
Pros:
- Technology freedom per service
- Data encapsulation
- Independent scaling

Cons:
- No JOINs across services
- Data duplication (denormalization)
- Eventual consistency
```

**Data Replication Strategies**:
1. **Event-Driven**: Service publishes events, others maintain read-only copies
2. **ETL Pipeline**: Batch data synchronization
3. **CQRS**: Separate write and read models

**Example**: E-commerce data flow
```
Order Service (writes to Orders DB) → publishes OrderCreated event
↓
Analytics Service (reads events) → updates data warehouse
Reporting Service (reads events) → updates materialized views
Email Service (reads events) → sends confirmation email
```

### **BLOCK 4: INTERVIEW FOCUS (30 minutes)**

**Common Questions & Answers:**

1. **"When would you choose microservices over monolith?"**
   ```
   Choose microservices when:
   1. Multiple teams working on same product
   2. Different scaling requirements per component
   3. Need different technologies for different parts
   4. High availability requirements (can't afford full downtime)
   5. Organization is large (> 50 engineers)
   
   Start with monolith when:
   1. Small team (< 10 engineers)
   2. Unclear product requirements
   3. Need to move fast, prove concept
   4. Simple application with uniform scaling needs
   ```

2. **"How do you handle service discovery in microservices?"**
   ```
   Problem: Services need to find each other (IPs change dynamically)
   
   Solutions:
   1. Client-side discovery: Clients query service registry (Eureka, Consul)
   2. Server-side discovery: Load balancer queries registry, routes requests
   3. DNS-based: Each service gets DNS entry (Kubernetes services)
   
   Example with Kubernetes:
   Service A calls Service B using DNS: http://service-b.namespace.svc.cluster.local
   Kubernetes handles IP resolution and load balancing
   ```

3. **"What are the biggest challenges with microservices?"**
   ```
   1. Distributed complexity: Debugging, monitoring across services
   2. Data consistency: No ACID transactions across services
   3. Network reliability: Services communicating over network
   4. Operational overhead: Deploying, monitoring many services
   5. Testing: End-to-end testing harder
   6. Versioning: API compatibility between services
   
   Mitigations:
   - Comprehensive observability (tracing, logging, metrics)
   - Saga pattern for transactions
   - Circuit breakers for network issues
   - Container orchestration (Kubernetes)
   - Contract testing for APIs
   ```

**Whiteboard Practice**:
Design an e-commerce system as microservices:
```
Services:
1. API Gateway (entry point, auth, rate limiting)
2. User Service (authentication, profiles)
3. Product Service (catalog, inventory)
4. Cart Service (shopping cart operations)
5. Order Service (order processing)
6. Payment Service (payment processing)
7. Shipping Service (delivery tracking)
8. Notification Service (emails, SMS)

Data flow for checkout:
1. User submits cart → Cart Service
2. Reserve inventory → Product Service
3. Create order → Order Service
4. Process payment → Payment Service
5. Schedule shipping → Shipping Service
6. Send confirmation → Notification Service
```

**Failure Scenario**: Payment service is down during checkout
```
Solution: Saga pattern with compensating transactions
1. Create order (pending)
2. Try payment → fails
3. Execute compensation: Release reserved inventory
4. Update order status to "payment failed"
5. Notify user to retry payment
```

---

## 📝 **DAY 8 OUTPUT REQUIRED**

**Complete in your notebook:**

1. **Monolith vs Microservices Decision Matrix**:
```
| Consideration          | Monolith                     | Microservices               |
|------------------------|------------------------------|-----------------------------|
| Team Size              | < 10 engineers               | > 50 engineers              |
| Development Speed      | Fast initially               | Slower initially, faster later|
| Complexity             | Lower operational complexity | Higher operational overhead |
| Scaling                | Vertical scaling             | Horizontal scaling          |
| Technology Flexibility | Single tech stack            | Polyglot (multiple techs)   |
| Failure Isolation      | Single point of failure      | Isolated failures           |
```

2. **Service Decomposition Rules**:
```
When to split:
1. Different scaling requirements (e.g., auth vs video processing)
2. Different availability requirements (critical vs background)
3. Different data access patterns (OLTP vs OLAP)
4. Owned by different teams
5. Different rate of change (volatile vs stable)

When to keep together:
1. Frequent joins between data
2. Need strong consistency
3. Low traffic/importance
4. Tightly coupled functionality
```

3. **Communication Pattern Guide**:
```
Use Synchronous (REST/gRPC) when:
- Need immediate response
- Simple request-response
- Client needs to know result now

Use Asynchronous (Events) when:
- Processing can happen later
- Multiple consumers needed
- Want to decouple services
- Handling background jobs
```

4. **Flashcards Created**:
- Front: Saga Pattern
- Back: Sequence of local transactions with compensating actions on failure. Used for distributed transactions.

- Front: Circuit Breaker Pattern
- Back: Prevents cascading failures. Opens circuit when service fails, fails fast instead of waiting.

---

## 🧪 **Day 8 Self-Test**

**Answer without looking:**
1. What are the main benefits of microservices over monoliths?
2. What is the Saga pattern and when is it used?
3. Why is an API Gateway used in microservices?
4. What's the biggest challenge with databases in microservices?

**Answers:**
1. Independent deployment, scaling, and development; technology flexibility; failure isolation; team autonomy.
2. Saga pattern coordinates distributed transactions through a sequence of local transactions with compensating actions if any step fails. Used when you need transaction-like behavior across multiple services.
3. API Gateway provides single entry point, handles cross-cutting concerns (auth, rate limiting, routing), aggregates responses, and simplifies client interactions.
4. Maintaining data consistency across services since each service has its own database. Can't use ACID transactions across services, leading to eventual consistency challenges.

---

## 🚀 **Day 9 Preview: REAL-TIME SYSTEMS & WEBSOCKETS**

**Preparation Thought:**
- Most APIs are request-response. What about real-time updates?
- How does Facebook show "User is typing..." in chat?
- Think: Live sports scores, stock tickers, collaborative editing

---

## 💪 **Accountability Check**

**Reply with:**
1. ✅ I've completed Day 8
2. 🔀 Real company using microservices you admire: _____
3. ⚡ Most surprising microservices challenge: _____
4. ❓ Question about microservices or API design: _____

**Remember**: Microservices aren't a silver bullet - they trade development complexity for organizational scalability. Choose wisely!

---

**Ready for real-time systems tomorrow?** This is where systems get really exciting! 🚀