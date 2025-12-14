# Core System Design Concepts

## Table of Contents
1. [Scalability](#scalability)
2. [Reliability](#reliability)
3. [Availability](#availability)
4. [Performance](#performance)
5. [Consistency](#consistency)
6. [CAP Theorem](#cap-theorem)
7. [ACID vs BASE](#acid-vs-base)
8. [Load Balancing](#load-balancing)
9. [Caching](#caching)
10. [Database Sharding](#database-sharding)

---

## Scalability

**Scalability** is the ability of a system to handle increased load by adding resources.

### Types of Scalability

#### 1. **Vertical Scaling (Scale Up)**

Add more power to existing machines:
- More CPU
- More RAM
- More storage

**Example:**
```
Before: 4 CPU, 8GB RAM
After:  16 CPU, 64GB RAM
```

**Pros:**
- Simple (no code changes)
- No distributed system complexity

**Cons:**
- Limited by hardware
- Expensive
- Single point of failure

**When to Use:**
- Small to medium applications
- When simplicity is priority

---

#### 2. **Horizontal Scaling (Scale Out)**

Add more machines to handle load:
- More servers
- More database replicas
- More cache nodes

**Example:**
```
Before: 1 server
After:  10 servers
```

**Pros:**
- Unlimited scaling potential
- Cost-effective (commodity hardware)
- Fault tolerant

**Cons:**
- Complex (distributed system)
- Requires load balancing
- Data consistency challenges

**When to Use:**
- Large-scale applications
- High availability requirements

---

### Real-World Example: Instagram's Scaling Journey

```
2010: Vertical Scaling
┌─────────────────┐
│  Single Server  │
│  4 CPU, 16GB    │
└─────────────────┘
Users: 1M

2012: Horizontal Scaling
┌─────┐ ┌─────┐ ┌─────┐
│ S1  │ │ S2  │ │ S3  │
└─────┘ └─────┘ └─────┘
Users: 30M

2016: Distributed System
┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
│ S1  │ │ S2  │ │ S3  │ │ S4  │
└─────┘ └─────┘ └─────┘ └─────┘
     └─────┬─────┘
           │
    ┌──────┴──────┐
    │             │
┌─────┐      ┌─────┐
│ DB1 │      │ DB2 │
└─────┘      └─────┘
Users: 500M
```

---

### Scaling Strategies

#### 1. **Stateless Services**

Make services stateless so any server can handle any request:

```
Stateful (Hard to Scale):
┌──────────┐
│ Server 1 │  ← User session stored here
└──────────┘
User must always go to Server 1 ❌

Stateless (Easy to Scale):
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Server 1 │  │ Server 2 │  │ Server 3 │
└──────────┘  └──────────┘  └──────────┘
         └──────────┬──────────┘
                    │
              ┌─────┴─────┐
              │  Session  │
              │   Store   │
              │  (Redis)  │
              └───────────┘
User can go to any server ✅
```

**Example: E-Commerce Platform**

```java
// Stateful (Bad)
class ShoppingCart {
    private Map<String, Item> items; // Stored in server memory
}

// Stateless (Good)
class ShoppingCart {
    // Store in Redis/Database
    void addItem(String userId, Item item) {
        redis.set("cart:" + userId, item);
    }
}
```

---

#### 2. **Database Scaling**

**Read Replicas:**
```
┌──────────┐
│  Master  │  ← Writes
│  (Write) │
└────┬─────┘
     │
┌────┼────┬────┐
│    │    │    │
▼    ▼    ▼    ▼
┌──┐ ┌──┐ ┌──┐ ┌──┐
│R1│ │R2│ │R3│ │R4│  ← Reads
└──┘ └──┘ └──┘ └──┘
```

**Sharding:**
```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Shard 1  │  │ Shard 2  │  │ Shard 3  │
│ (User    │  │ (User    │  │ (User    │
│  1-33M)  │  │  34-66M) │  │  67-100M)│
└──────────┘  └──────────┘  └──────────┘
```

---

## Reliability

**Reliability** is the probability that a system will work correctly over time.

### Reliability Metrics

**MTBF (Mean Time Between Failures):**
- Average time between system failures
- Higher = More reliable

**MTTR (Mean Time To Repair):**
- Average time to fix a failure
- Lower = Better

**Availability = MTBF / (MTBF + MTTR)**

**Example:**
```
MTBF = 720 hours (30 days)
MTTR = 1 hour
Availability = 720 / (720 + 1) = 99.86%
```

---

### Fault Tolerance Patterns

#### 1. **Redundancy**

Have backup components ready:

```
Single Component:
┌──────────┐
│  Server  │  ❌ Single point of failure
└──────────┘

Redundant:
┌──────────┐  ┌──────────┐
│ Server 1 │  │ Server 2 │  ✅ Backup ready
└──────────┘  └──────────┘
```

**Example: Database Replication**

```
┌──────────┐
│  Master  │  ← Primary database
└────┬─────┘
     │ Replication
     ▼
┌──────────┐
│  Slave   │  ← Backup database
└──────────┘
If master fails, slave takes over
```

---

#### 2. **Circuit Breaker Pattern**

Prevent cascading failures:

```
Normal Flow:
Service A → Service B → Response ✅

Service B Fails:
Service A → Service B → Timeout ❌
Service A → Service B → Timeout ❌
Service A → Service B → Timeout ❌
(Keeps trying, wasting resources)

With Circuit Breaker:
Service A → [Circuit Open] → Fallback Response ✅
(Stops calling Service B, uses cached/default response)
```

**Implementation:**
```java
@CircuitBreaker(name = "paymentService", fallbackMethod = "fallback")
public PaymentResult processPayment(PaymentRequest request) {
    return paymentService.process(request);
}

public PaymentResult fallback(PaymentRequest request, Exception e) {
    // Return cached result or default
    return PaymentResult.pending();
}
```

---

#### 3. **Health Checks**

Monitor system health:

```
Health Check Endpoint:
GET /health

Response:
{
  "status": "healthy",
  "database": "connected",
  "cache": "connected",
  "disk": "85% used"
}
```

**Example: Kubernetes Health Checks**

```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

---

## Availability

**Availability** is the percentage of time a system is operational.

### Availability Levels

| Availability | Downtime/Year | Example |
|--------------|---------------|---------|
| 90% (1 nine) | 36.5 days | Basic systems |
| 99% (2 nines) | 3.65 days | Standard systems |
| 99.9% (3 nines) | 8.76 hours | Good systems |
| 99.99% (4 nines) | 52.56 minutes | High availability |
| 99.999% (5 nines) | 5.26 minutes | Mission critical |

---

### Achieving High Availability

#### 1. **Redundancy**

```
Single Server:
┌──────────┐
│  Server  │  ❌ 50% availability (if fails)
└──────────┘

Two Servers:
┌──────────┐  ┌──────────┐
│ Server 1 │  │ Server 2 │  ✅ 99% availability
└──────────┘  └──────────┘
(If one fails, other continues)
```

---

#### 2. **Failover**

Automatic switching to backup:

```
Active-Passive:
┌──────────┐ (Active)
│ Server 1 │  ← Handling requests
└──────────┘
┌──────────┐ (Standby)
│ Server 2 │  ← Monitoring, ready to take over
└──────────┘

If Server 1 fails:
┌──────────┐ (Down)
│ Server 1 │
└──────────┘
┌──────────┐ (Active)
│ Server 2 │  ← Automatically takes over
└──────────┘
```

---

#### 3. **Load Distribution**

```
Round Robin:
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
...

Least Connections:
Request → Server with fewest active connections

Geographic:
Request from US → US Server
Request from EU → EU Server
```

---

## Performance

**Performance** is how fast a system responds to requests.

### Performance Metrics

1. **Latency:** Time to process a request
2. **Throughput:** Number of requests per second
3. **Response Time:** End-to-end time (latency + network)

---

### Performance Optimization Techniques

#### 1. **Caching**

Store frequently accessed data in fast storage:

```
Without Cache:
Request → Database → Response (100ms) ❌

With Cache:
Request → Cache → Response (5ms) ✅
(Cache miss → Database → Update Cache)
```

**Example: E-Commerce Product Page**

```
User requests: /products/123

Without Cache:
1. Query database (50ms)
2. Render page (30ms)
Total: 80ms

With Cache:
1. Check cache (1ms) → Hit!
2. Render page (30ms)
Total: 31ms (60% faster)
```

---

#### 2. **Database Indexing**

Speed up database queries:

```
Without Index:
SELECT * FROM users WHERE email = 'user@example.com'
→ Full table scan (1M rows = 500ms) ❌

With Index:
CREATE INDEX idx_email ON users(email)
→ Index lookup (1M rows = 5ms) ✅
```

---

#### 3. **CDN (Content Delivery Network)**

Serve static content from edge locations:

```
Without CDN:
User (India) → Server (US) → Response (500ms) ❌

With CDN:
User (India) → CDN Edge (India) → Response (50ms) ✅
```

**Example: Image Delivery**

```
┌─────────────────────────────────┐
│         Origin Server           │
│      (US - Single Location)     │
└─────────────────────────────────┘
              │
    ┌─────────┼─────────┐
    │         │         │
    ▼         ▼         ▼
┌──────┐ ┌──────┐ ┌──────┐
│India │ │Europe│ │Asia  │
│Edge  │ │Edge  │ │Edge  │
└──────┘ └──────┘ └──────┘
```

---

#### 4. **Asynchronous Processing**

Don't wait for slow operations:

```
Synchronous:
Request → Process Payment (2s) → Send Email (1s) → Response
Total: 3s ❌

Asynchronous:
Request → Process Payment (2s) → Response
         → Send Email (1s) [Background]
Total: 2s ✅
```

**Example: Order Processing**

```java
// Synchronous (Bad)
public Order createOrder(OrderRequest request) {
    Order order = orderRepository.save(request);
    paymentService.processPayment(order);  // Blocks
    emailService.sendConfirmation(order);  // Blocks
    return order;  // Returns after 3 seconds
}

// Asynchronous (Good)
public Order createOrder(OrderRequest request) {
    Order order = orderRepository.save(request);
    kafkaTemplate.send("payment-request", order);  // Non-blocking
    kafkaTemplate.send("email-request", order);   // Non-blocking
    return order;  // Returns immediately
}
```

---

## Consistency

**Consistency** ensures all nodes see the same data at the same time.

### Consistency Models

#### 1. **Strong Consistency**

All nodes see updates immediately:

```
Write to Node 1:
Node 1: Value = A ✅
Node 2: Value = A ✅ (immediately)
Node 3: Value = A ✅ (immediately)
```

**Example: Financial Transactions**

```
Transfer $100 from Account A to Account B

Strong Consistency:
1. Deduct $100 from A
2. Add $100 to B
3. Both updates visible immediately
→ No possibility of inconsistency
```

---

#### 2. **Eventual Consistency**

All nodes will see updates eventually:

```
Write to Node 1:
Node 1: Value = A ✅
Node 2: Value = A (after replication)
Node 3: Value = A (after replication)
```

**Example: Social Media Likes**

```
User likes a post:
Node 1: Like count = 1001 ✅
Node 2: Like count = 1000 (will update soon)
Node 3: Like count = 1000 (will update soon)

Eventually all nodes show 1001
→ Acceptable for non-critical data
```

---

### Consistency Patterns

#### 1. **Master-Slave Replication**

```
┌──────────┐
│  Master  │  ← All writes
└────┬─────┘
     │ Replication
     ▼
┌──────────┐
│  Slave   │  ← All reads
└──────────┘

Consistency: Strong (slave syncs with master)
```

---

#### 2. **Master-Master Replication**

```
┌──────────┐      ┌──────────┐
│ Master 1 │◄────►│ Master 2 │
└──────────┘      └──────────┘
     │                  │
     └────────┬─────────┘
              │
         ┌────┴────┐
         │  Sync   │
         └─────────┘

Consistency: Eventual (conflicts possible)
```

---

## CAP Theorem

**CAP Theorem** states that a distributed system can only guarantee 2 out of 3:

- **C**onsistency: All nodes see same data
- **A**vailability: System responds to requests
- **P**artition Tolerance: System works despite network failures

```
You can only choose 2:
┌─────────────┐
│   CA        │  ← Strong consistency + Availability
│   (No       │     (No network partitions)
│   Partitions)│
├─────────────┤
│   CP        │  ← Strong consistency + Partition tolerance
│   (Not      │     (Sacrifice availability)
│   Available)│
├─────────────┤
│   AP        │  ← Availability + Partition tolerance
│   (Eventual │     (Sacrifice consistency)
│   Consistency)│
└─────────────┘
```

### Real-World Examples

**CP System: MongoDB (with strong consistency)**
```
Network partition occurs:
- Stops accepting writes
- Maintains consistency
- Sacrifices availability
```

**AP System: Cassandra**
```
Network partition occurs:
- Continues accepting writes
- Maintains availability
- Sacrifices consistency (eventual)
```

**CA System: Traditional RDBMS (single node)**
```
No network partitions:
- Strong consistency
- High availability
- Single point of failure
```

---

## ACID vs BASE

### ACID (Traditional Databases)

**A**tomicity: All or nothing
**C**onsistency: Valid state transitions
**I**solation: Concurrent transactions don't interfere
**D**urability: Committed data persists

**Example: Bank Transfer**

```sql
BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE id = 'A';
  UPDATE accounts SET balance = balance + 100 WHERE id = 'B';
COMMIT;
```

If any step fails, entire transaction rolls back.

---

### BASE (NoSQL Databases)

**B**asically **A**vailable: System is available most of the time
**S**oft state: State may change over time
**E**ventual consistency: Will be consistent eventually

**Example: Social Media Feed**

```
User posts update:
- Immediately visible to some users (available)
- May take time to appear for all users (eventual)
- Acceptable for social media
```

---

## Load Balancing

**Load Balancing** distributes incoming requests across multiple servers.

### Load Balancing Algorithms

#### 1. **Round Robin**

```
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
...
```

---

#### 2. **Least Connections**

```
Request → Server with fewest active connections
```

---

#### 3. **Weighted Round Robin**

```
Server 1 (Weight: 3) → Gets 3 requests
Server 2 (Weight: 2) → Gets 2 requests
Server 3 (Weight: 1) → Gets 1 request
```

---

### Load Balancer Types

#### 1. **Layer 4 (Transport Layer)**

Routes based on IP and port:

```
┌──────────┐
│   Client │
└────┬─────┘
     │
     ▼
┌──────────────┐
│ Layer 4 LB  │  ← Routes by IP:Port
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│ S1  │ │ S2  │
└─────┘ └─────┘
```

---

#### 2. **Layer 7 (Application Layer)**

Routes based on URL, headers, cookies:

```
┌──────────┐
│   Client │
└────┬─────┘
     │
     ▼
┌──────────────┐
│ Layer 7 LB  │  ← Routes by URL/Headers
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│ API │ │ Web │
└─────┘ └─────┘
```

---

## Caching

**Caching** stores frequently accessed data in fast storage.

### Cache Strategies

#### 1. **Cache-Aside (Lazy Loading)**

```
Read:
1. Check cache
2. If miss → Read from DB
3. Update cache
4. Return data

Write:
1. Write to DB
2. Invalidate cache
```

**Example:**
```java
public Product getProduct(String id) {
    // 1. Check cache
    Product product = cache.get("product:" + id);
    if (product != null) {
        return product;  // Cache hit
    }
    
    // 2. Cache miss - read from DB
    product = database.getProduct(id);
    
    // 3. Update cache
    cache.set("product:" + id, product, 3600);  // 1 hour TTL
    
    return product;
}
```

---

#### 2. **Write-Through**

```
Write:
1. Write to cache
2. Write to DB
3. Return

Read:
1. Read from cache (always available)
```

---

#### 3. **Write-Back (Write-Behind)**

```
Write:
1. Write to cache
2. Return immediately
3. Async write to DB (later)
```

---

## Database Sharding

**Sharding** splits a database into smaller pieces (shards).

### Sharding Strategies

#### 1. **Range-Based Sharding**

```
Shard 1: User IDs 1-1M
Shard 2: User IDs 1M-2M
Shard 3: User IDs 2M-3M
```

---

#### 2. **Hash-Based Sharding**

```
hash(userId) % 3 = shard number

User ID 123 → hash(123) % 3 = 0 → Shard 1
User ID 456 → hash(456) % 3 = 1 → Shard 2
User ID 789 → hash(789) % 3 = 2 → Shard 3
```

---

#### 3. **Directory-Based Sharding**

```
Lookup Table:
User ID → Shard Number

User 123 → Shard 1
User 456 → Shard 2
User 789 → Shard 3
```

---

### Sharding Example: E-Commerce Platform

```
┌─────────────────────────────────┐
│      Application Layer          │
└──────────────┬──────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
┌──────────┐   ┌──────────┐
│ Shard 1  │   │ Shard 2  │
│ (US)     │   │ (EU)     │
│ Users:   │   │ Users:   │
│ 1-50M    │   │ 51-100M  │
└──────────┘   └──────────┘
```

**Benefits:**
- Distributes load
- Improves performance
- Enables horizontal scaling

**Challenges:**
- Cross-shard queries
- Rebalancing
- Transaction management

---

## Summary

**Core Concepts:**
1. ✅ **Scalability:** Vertical vs Horizontal
2. ✅ **Reliability:** Fault tolerance, redundancy
3. ✅ **Availability:** High availability patterns
4. ✅ **Performance:** Caching, indexing, CDN
5. ✅ **Consistency:** Strong vs Eventual
6. ✅ **CAP Theorem:** Choose 2 out of 3
7. ✅ **ACID vs BASE:** Consistency models
8. ✅ **Load Balancing:** Distribute traffic
9. ✅ **Caching:** Speed up access
10. ✅ **Sharding:** Scale databases

**Next Steps:**
- Learn [Key Components](./03_KEY_COMPONENTS.md)
- Understand [Scalability Patterns](./04_SCALABILITY_PATTERNS.md)
- Practice [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [AlgoMaster System Design](https://algomaster.io/learn/system-design/what-is-system-design)
- [System Design Concepts](https://www.geeksforgeeks.org/system-design/)

