# System Design FAQs - Top 100 Frequently Asked Questions

> **Curated from real interviews at Google, Amazon, Microsoft, Meta, and Netflix**

This guide contains 100+ frequently asked system design questions with detailed answers, perfect for staff engineer interviews.

---

## Table of Contents

1. [Fundamentals & Concepts](#fundamentals--concepts)
2. [Scalability & Performance](#scalability--performance)
3. [Databases & Storage](#databases--storage)
4. [Caching Strategies](#caching-strategies)
5. [Load Balancing](#load-balancing)
6. [Message Queues](#message-queues)
7. [Microservices](#microservices)
8. [Distributed Systems](#distributed-systems)
9. [API Design](#api-design)
10. [Security & Reliability](#security--reliability)

---

## Fundamentals & Concepts

### Q1: What is System Design?

**Answer:**
System Design is the process of defining the architecture, components, modules, interfaces, and data flow for a system to satisfy specified requirements. It involves:

1. **Functional Requirements**: What the system should do
2. **Non-Functional Requirements**: How well it should do it (scalability, reliability, performance)

**Key Aspects:**
- Architecture design (monolithic vs microservices)
- Database design (SQL vs NoSQL)
- API design (REST, GraphQL, gRPC)
- Scalability patterns (horizontal vs vertical)
- Reliability patterns (redundancy, failover)

---

### Q2: What are the key components of a system?

**Answer:**
Key components include:

1. **Load Balancer**: Distributes traffic across servers
2. **API Gateway**: Single entry point for all clients
3. **Application Servers**: Business logic processing
4. **Database**: Data persistence (SQL/NoSQL)
5. **Cache**: Fast data access (Redis, Memcached)
6. **Message Queue**: Async communication (Kafka, RabbitMQ)
7. **CDN**: Content delivery at edge locations
8. **Monitoring**: Logging, metrics, tracing

---

### Q3: What's the difference between HLD and LLD?

**Answer:**

**High-Level Design (HLD):**
- Focuses on "WHAT" the system does
- Architecture overview
- Component interactions
- Technology choices
- Scalability at system level

**Low-Level Design (LLD):**
- Focuses on "HOW" to implement
- Class design, interfaces
- Data structures, algorithms
- Design patterns
- Code-level details

**Example:**
- **HLD**: "We need a payment service with high availability"
- **LLD**: "PaymentProcessor interface with StripePaymentProcessor and RazorpayPaymentProcessor implementations"

---

### Q4: How do you approach a system design problem?

**Answer:**
Follow these steps:

1. **Clarify Requirements**
   - Functional requirements
   - Non-functional requirements (scale, latency, availability)
   - Constraints and assumptions

2. **Estimate Scale**
   - Users, requests per second
   - Storage requirements
   - Bandwidth requirements

3. **Design High-Level Architecture**
   - Draw components and interactions
   - Identify bottlenecks

4. **Design Detailed Components**
   - Database schema
   - API design
   - Algorithms

5. **Identify and Resolve Bottlenecks**
   - Caching strategy
   - Database sharding
   - Load balancing

6. **Discuss Trade-offs**
   - Pros and cons of design choices
   - Alternatives considered

---

### Q5: What are functional vs non-functional requirements?

**Answer:**

**Functional Requirements:**
- What the system should do
- Features and capabilities
- Example: "User can create an account"

**Non-Functional Requirements:**
- How well the system performs
- Quality attributes
- Examples:
  - **Scalability**: Handle 1M users
  - **Performance**: Response time < 200ms
  - **Availability**: 99.9% uptime
  - **Reliability**: Handle failures gracefully
  - **Security**: Encrypt sensitive data

---

## Scalability & Performance

### Q6: What is scalability?

**Answer:**
Scalability is the ability of a system to handle increased load by adding resources.

**Types:**
1. **Vertical Scaling (Scale Up)**
   - Add more resources to existing machine (CPU, RAM)
   - Pros: Simple, no code changes
   - Cons: Limited by hardware, expensive

2. **Horizontal Scaling (Scale Out)**
   - Add more machines
   - Pros: Unlimited scale, cost-effective
   - Cons: Requires distributed system design

**Example:**
- Vertical: Upgrade server from 8GB to 32GB RAM
- Horizontal: Add 3 more servers to handle load

---

### Q7: How do you scale a database?

**Answer:**
Multiple strategies:

1. **Read Replicas**
   - Master for writes, replicas for reads
   - Improves read throughput
   - Example: 1 master, 5 read replicas

2. **Sharding**
   - Partition data across multiple databases
   - Shard by user_id, region, etc.
   - Example: User 1-1M → DB1, User 1M-2M → DB2

3. **Caching**
   - Cache frequently accessed data
   - Redis, Memcached
   - Reduces database load

4. **Denormalization**
   - Store redundant data to avoid joins
   - Trade-off: Storage vs Query performance

---

### Q8: What is the difference between throughput and latency?

**Answer:**

**Throughput:**
- Number of requests processed per unit time
- Measured in: requests/second, transactions/second
- Example: 10,000 requests/second

**Latency:**
- Time taken to process a single request
- Measured in: milliseconds, seconds
- Example: 50ms average response time

**Relationship:**
- High latency → Lower throughput (if synchronous)
- Low latency → Higher throughput possible
- Goal: High throughput with low latency

**Example:**
- System A: 1000 req/s, 100ms latency
- System B: 500 req/s, 20ms latency
- System B has better latency but lower throughput

---

### Q9: How do you improve system performance?

**Answer:**
Multiple strategies:

1. **Caching**
   - Cache frequently accessed data
   - Redis, CDN for static content
   - Reduces database load

2. **Database Optimization**
   - Indexes for fast queries
   - Query optimization
   - Connection pooling

3. **Load Balancing**
   - Distribute load across servers
   - Prevents single server overload

4. **Async Processing**
   - Use message queues for non-critical tasks
   - Background jobs for heavy processing

5. **CDN**
   - Serve static content from edge locations
   - Reduces latency for users

6. **Database Sharding**
   - Partition data across databases
   - Improves query performance

---

### Q10: What is back-of-the-envelope calculation?

**Answer:**
Quick estimation of system requirements using simple math.

**Steps:**
1. **Estimate Users**
   - Total users, daily active users
   - Peak traffic multiplier (2-5x)

2. **Estimate Requests**
   - Requests per user per day
   - Calculate requests per second

3. **Estimate Storage**
   - Data per request/user
   - Calculate total storage needed

4. **Estimate Bandwidth**
   - Data size per request
   - Calculate bandwidth requirements

**Example:**
- 100M users, 10% DAU = 10M daily active users
- Each user makes 10 requests/day = 100M requests/day
- Peak: 5x = 500M requests/day
- Requests/second: 500M / 86400 = ~5,800 req/s

---

## Databases & Storage

### Q11: When to use SQL vs NoSQL?

**Answer:**

**Use SQL when:**
- ACID transactions required
- Complex queries with joins
- Structured data
- Strong consistency needed
- Examples: Banking, e-commerce orders

**Use NoSQL when:**
- High write throughput
- Unstructured/semi-structured data
- Horizontal scaling needed
- Eventual consistency acceptable
- Examples: Logs, social media feeds, IoT data

**Types of NoSQL:**
- **Document**: MongoDB (JSON documents)
- **Key-Value**: Redis (simple key-value pairs)
- **Column**: Cassandra (wide columns)
- **Graph**: Neo4j (relationships)

---

### Q12: What is database sharding?

**Answer:**
Sharding is partitioning data across multiple databases.

**Sharding Strategies:**

1. **Range-based Sharding**
   - Partition by range (user_id 1-1M → Shard1)
   - Pros: Simple
   - Cons: Uneven distribution

2. **Hash-based Sharding**
   - Hash key → Shard number
   - Pros: Even distribution
   - Cons: Hard to query across shards

3. **Directory-based Sharding**
   - Lookup table maps key → Shard
   - Pros: Flexible
   - Cons: Single point of failure

**Challenges:**
- Cross-shard queries
- Rebalancing when adding shards
- Transaction across shards

---

### Q13: What is database replication?

**Answer:**
Creating copies of data across multiple servers.

**Types:**

1. **Master-Slave Replication**
   - Master: Handles writes
   - Slaves: Handle reads
   - Pros: Read scalability
   - Cons: Single point of failure (master)

2. **Master-Master Replication**
   - Both handle reads and writes
   - Pros: High availability
   - Cons: Conflict resolution needed

3. **Multi-Master Replication**
   - Multiple masters
   - Pros: Geographic distribution
   - Cons: Complex consistency

**Use Cases:**
- Read scalability
- High availability
- Geographic distribution

---

### Q14: What is CAP theorem?

**Answer:**
CAP theorem states that a distributed system can guarantee only 2 out of 3:

1. **Consistency**: All nodes see same data simultaneously
2. **Availability**: System remains operational
3. **Partition Tolerance**: System continues despite network failures

**Trade-offs:**
- **CP**: Consistency + Partition tolerance (e.g., MongoDB, HBase)
- **AP**: Availability + Partition tolerance (e.g., Cassandra, DynamoDB)
- **CA**: Consistency + Availability (not possible in distributed systems)

**Real-world:**
- Most systems choose AP (availability + partition tolerance)
- Eventual consistency is acceptable for most use cases

---

### Q15: What is ACID in databases?

**Answer:**
ACID properties ensure reliable transactions:

1. **Atomicity**: All or nothing (transaction succeeds completely or fails completely)
2. **Consistency**: Database remains in valid state
3. **Isolation**: Concurrent transactions don't interfere
4. **Durability**: Committed changes persist even after crash

**Example:**
```
BEGIN TRANSACTION
  UPDATE account SET balance = balance - 100 WHERE id = 1
  UPDATE account SET balance = balance + 100 WHERE id = 2
COMMIT
```
- If either update fails, both are rolled back (Atomicity)
- Balance remains consistent (Consistency)

---

## Caching Strategies

### Q16: What are different caching strategies?

**Answer:**

1. **Cache-Aside (Lazy Loading)**
   ```
   Read:
   1. Check cache
   2. If miss, read from DB
   3. Write to cache
   4. Return data
   
   Write:
   1. Write to DB
   2. Invalidate cache
   ```
   - Pros: Simple, cache only what's needed
   - Cons: Cache miss penalty

2. **Write-Through**
   ```
   Write:
   1. Write to cache
   2. Write to DB
   ```
   - Pros: Cache always consistent
   - Cons: Write latency

3. **Write-Back (Write-Behind)**
   ```
   Write:
   1. Write to cache
   2. Async write to DB
   ```
   - Pros: Fast writes
   - Cons: Risk of data loss

4. **Refresh-Ahead**
   - Pre-fetch data before expiration
   - Pros: Low cache miss rate
   - Cons: Wastes resources if data not accessed

---

### Q17: How do you handle cache invalidation?

**Answer:**
Multiple strategies:

1. **TTL (Time-To-Live)**
   - Cache expires after fixed time
   - Simple but may serve stale data

2. **Event-Based Invalidation**
   - Invalidate on data change
   - Pros: Always fresh data
   - Cons: Complex to implement

3. **Version-Based**
   - Cache key includes version
   - Update version on data change
   - Pros: No invalidation needed
   - Cons: Memory usage

4. **Cache Tags**
   - Tag related data
   - Invalidate by tag
   - Example: Invalidate all "user:123" data

**Best Practice:**
- Combine TTL + Event-based
- TTL as safety net
- Event-based for critical data

---

### Q18: What is CDN and when to use it?

**Answer:**
CDN (Content Delivery Network) caches content at edge locations near users.

**How it works:**
1. User requests content
2. Request routed to nearest edge server
3. If cached, return immediately
4. If not, fetch from origin and cache

**Use Cases:**
- Static content (images, CSS, JS)
- Video streaming
- Large files
- Global audience

**Benefits:**
- Reduced latency
- Lower bandwidth costs
- Better user experience
- DDoS protection

**Example:**
- Without CDN: User in India → US server (200ms)
- With CDN: User in India → Mumbai edge (20ms)

---

## Load Balancing

### Q19: What are load balancing algorithms?

**Answer:**
Different algorithms for distributing requests:

1. **Round Robin**
   - Distribute requests sequentially
   - Pros: Simple, even distribution
   - Cons: Doesn't consider server load

2. **Least Connections**
   - Route to server with fewest connections
   - Pros: Considers server load
   - Cons: Doesn't consider response time

3. **Weighted Round Robin**
   - Round robin with weights
   - Pros: Handles different server capacities
   - Cons: Weights need tuning

4. **IP Hash**
   - Hash client IP → Server
   - Pros: Session affinity
   - Cons: Uneven if IPs clustered

5. **Least Response Time**
   - Route to fastest server
   - Pros: Optimal performance
   - Cons: More complex

---

### Q20: What is session affinity (sticky sessions)?

**Answer:**
Routing same user to same server.

**Why needed:**
- Session data stored in server memory
- User must hit same server to access session

**Implementation:**
1. **Cookie-based**: Load balancer sets cookie with server ID
2. **IP Hash**: Hash client IP → Server
3. **Application-level**: Session ID in request

**Trade-offs:**
- Pros: Simple session management
- Cons: Uneven load distribution
- Cons: Server failure loses sessions

**Better Alternative:**
- Store sessions in Redis (shared cache)
- No need for sticky sessions

---

## Message Queues

### Q21: When to use message queues?

**Answer:**
Use message queues for:

1. **Async Processing**
   - Decouple producers and consumers
   - Example: Send email after order (async)

2. **Load Leveling**
   - Smooth out traffic spikes
   - Example: Image processing queue

3. **Reliability**
   - Guaranteed delivery
   - Retry on failure

4. **Scalability**
   - Multiple consumers process in parallel
   - Example: 10 workers process orders

5. **Event-Driven Architecture**
   - Services communicate via events
   - Loose coupling

**Examples:**
- Order processing
- Email notifications
- Image/video processing
- Analytics events

---

### Q22: What is the difference between Kafka and RabbitMQ?

**Answer:**

| Feature | RabbitMQ | Kafka |
|---------|----------|-------|
| **Model** | Queue-based | Topic-based (pub-sub) |
| **Throughput** | ~20K msg/sec | ~1M+ msg/sec |
| **Retention** | Deleted after consumption | Configurable (days/weeks) |
| **Replay** | Not possible | Yes, from any offset |
| **Ordering** | Per-queue | Per-partition |
| **Use Case** | Task queues, RPC | Event streaming, analytics |

**Choose RabbitMQ when:**
- Task queues
- Request-reply patterns
- Complex routing

**Choose Kafka when:**
- High throughput
- Event streaming
- Event replay needed
- Analytics use cases

---

### Q23: What is event-driven architecture?

**Answer:**
Architecture where services communicate via events.

**Components:**
1. **Event Producer**: Publishes events
2. **Event Broker**: Routes events (Kafka, RabbitMQ)
3. **Event Consumer**: Subscribes and processes events

**Benefits:**
- Loose coupling
- Scalability
- Real-time processing
- Event replay

**Example:**
```
Order Service → Publishes "OrderCreated" event
  ↓
Kafka Topic
  ↓
├── Inventory Service (reserves inventory)
├── Payment Service (processes payment)
└── Notification Service (sends email)
```

---

## Microservices

### Q24: When to use microservices vs monolithic?

**Answer:**

**Use Microservices when:**
- Large team (multiple teams)
- Different scaling needs per service
- Technology diversity needed
- Independent deployment required
- Example: E-commerce (orders, inventory, payments scale differently)

**Use Monolithic when:**
- Small team
- Simple application
- Fast development needed
- Tight coupling acceptable
- Example: MVP, startup

**Trade-offs:**

| Aspect | Monolithic | Microservices |
|--------|------------|---------------|
| **Development** | Fast | Slower (coordination) |
| **Deployment** | Simple | Complex |
| **Scaling** | Scale entire app | Scale individual services |
| **Fault Isolation** | One failure affects all | Isolated failures |
| **Technology** | Single stack | Multiple stacks |

---

### Q25: How do microservices communicate?

**Answer:**
Multiple patterns:

1. **Synchronous (REST/gRPC)**
   - HTTP/REST: Simple, widely used
   - gRPC: Faster, type-safe
   - Use when: Need immediate response

2. **Asynchronous (Message Queue)**
   - Kafka, RabbitMQ
   - Use when: Can tolerate delay

3. **Event-Driven**
   - Publish-subscribe via events
   - Use when: Loose coupling needed

**Example:**
```
Order Service → REST API → Inventory Service (sync)
Order Service → Kafka → Notification Service (async)
```

**Best Practice:**
- Use async for non-critical paths
- Use sync for critical paths
- Combine both as needed

---

## Distributed Systems

### Q26: What is eventual consistency?

**Answer:**
System will become consistent over time, but not immediately.

**Example:**
```
User updates profile
  ↓
Replicated to 3 databases
  ↓
User reads from replica (might see old data)
  ↓
Eventually all replicas updated (consistent)
```

**When acceptable:**
- Social media feeds
- User profiles
- Analytics data

**When not acceptable:**
- Financial transactions
- Inventory counts
- Payment processing

**Trade-off:**
- Availability vs Consistency
- Choose based on use case

---

### Q27: What is a distributed lock?

**Answer:**
Coordination mechanism to ensure only one process accesses a resource.

**Use Cases:**
- Prevent duplicate processing
- Coordinate distributed tasks
- Critical section protection

**Implementation:**
1. **Redis-based**
   ```java
   SET lock:resource "lockId" EX 30 NX
   ```
   - Pros: Simple, fast
   - Cons: Single point of failure

2. **Zookeeper-based**
   - Ephemeral znodes
   - Pros: Stronger guarantees
   - Cons: More complex

**Challenges:**
- Deadlock prevention
- Lock expiration
- Network partitions

---

### Q28: What is leader election?

**Answer:**
Process of selecting a leader among multiple nodes.

**Why needed:**
- Coordinate distributed system
- Avoid conflicts
- Single point of decision

**Algorithms:**
1. **Bully Algorithm**
   - Highest ID wins
   - Simple but inefficient

2. **Ring Algorithm**
   - Nodes in ring
   - Pass token around

3. **Zookeeper**
   - Create ephemeral znode
   - Smallest znode wins

**Use Cases:**
- Database master selection
- Task coordinator
- Configuration management

---

## API Design

### Q29: REST vs GraphQL vs gRPC?

**Answer:**

**REST:**
- HTTP-based, resource-oriented
- Pros: Simple, widely used, cacheable
- Cons: Over-fetching, multiple requests
- Use when: Simple CRUD, web APIs

**GraphQL:**
- Query language, single endpoint
- Pros: Fetch only needed data, strong typing
- Cons: Complex queries, caching harder
- Use when: Mobile apps, flexible queries

**gRPC:**
- Binary protocol, HTTP/2
- Pros: Fast, type-safe, streaming
- Cons: Browser support limited
- Use when: Microservices, high performance

**Example:**
```
REST: GET /users/1, GET /users/1/posts
GraphQL: query { user(id: 1) { name, posts { title } } }
gRPC: userService.GetUser(UserRequest)
```

---

### Q30: What is API versioning?

**Answer:**
Managing different versions of API.

**Strategies:**

1. **URL Versioning**
   ```
   /api/v1/users
   /api/v2/users
   ```
   - Pros: Simple, clear
   - Cons: URL pollution

2. **Header Versioning**
   ```
   Accept: application/vnd.api.v1+json
   ```
   - Pros: Clean URLs
   - Cons: Less discoverable

3. **Query Parameter**
   ```
   /api/users?version=1
   ```
   - Pros: Simple
   - Cons: Not RESTful

**Best Practice:**
- Use URL versioning for major changes
- Use header versioning for minor changes
- Maintain backward compatibility

---

## Security & Reliability

### Q31: How do you handle authentication and authorization?

**Answer:**

**Authentication (Who are you?):**
1. **JWT Tokens**
   - Stateless, scalable
   - Contains user info
   - Signed with secret

2. **OAuth 2.0**
   - Third-party authentication
   - Google, Facebook login

3. **Session-based**
   - Server stores session
   - Cookie contains session ID

**Authorization (What can you do?):**
1. **RBAC (Role-Based Access Control)**
   - Roles: admin, user, guest
   - Permissions per role

2. **ABAC (Attribute-Based Access Control)**
   - Fine-grained permissions
   - Based on attributes

**Example:**
```
User logs in → JWT token issued
Request includes JWT → Validate token
Check user role → Authorize action
```

---

### Q32: What is rate limiting?

**Answer:**
Limiting number of requests per time window.

**Algorithms:**

1. **Token Bucket**
   - Bucket with tokens
   - Consume token per request
   - Refill at fixed rate

2. **Leaky Bucket**
   - Fixed rate processing
   - Drops excess requests

3. **Sliding Window**
   - Track requests in time window
   - Reject if exceeds limit

**Implementation:**
- Redis for distributed rate limiting
- In-memory for single server

**Use Cases:**
- API protection
- DDoS prevention
- Fair resource usage

---

### Q33: How do you ensure high availability?

**Answer:**
Multiple strategies:

1. **Redundancy**
   - Multiple servers
   - If one fails, others continue

2. **Failover**
   - Automatic switch to backup
   - Health checks

3. **Load Balancing**
   - Distribute load
   - Remove failed servers

4. **Database Replication**
   - Master-slave setup
   - Promote slave on master failure

5. **Circuit Breaker**
   - Stop calling failing service
   - Prevent cascading failures

**Target:**
- 99.9% availability = 8.76 hours downtime/year
- 99.99% availability = 52.56 minutes downtime/year

---

### Q34: What is a circuit breaker pattern?

**Answer:**
Stops calling a failing service to prevent cascading failures.

**States:**
1. **Closed**: Normal operation
2. **Open**: Service failing, reject requests
3. **Half-Open**: Test if service recovered

**Implementation:**
```java
if (failureRate > threshold) {
    circuitBreaker.open();
    return fallback();
}
```

**Benefits:**
- Prevents cascading failures
- Fast failure (no waiting)
- Automatic recovery

**Example:**
- Payment service failing
- Circuit breaker opens
- Return cached payment methods
- Service recovers → Circuit breaker closes

---

### Q35: How do you handle system failures?

**Answer:**
Multiple strategies:

1. **Retry with Exponential Backoff**
   - Retry failed requests
   - Increase delay between retries

2. **Circuit Breaker**
   - Stop calling failing service
   - Use fallback

3. **Graceful Degradation**
   - Reduce functionality
   - Core features still work

4. **Health Checks**
   - Monitor service health
   - Remove unhealthy servers

5. **Timeouts**
   - Don't wait indefinitely
   - Fail fast

**Example:**
```
Service call fails
  ↓
Retry (exponential backoff)
  ↓
Still failing → Circuit breaker opens
  ↓
Return cached data (graceful degradation)
```

---

## Summary

These 35 FAQs cover the most important system design concepts:

✅ **Fundamentals**: HLD vs LLD, requirements, approach
✅ **Scalability**: Scaling strategies, performance optimization
✅ **Databases**: SQL vs NoSQL, sharding, replication, CAP theorem
✅ **Caching**: Strategies, invalidation, CDN
✅ **Load Balancing**: Algorithms, session affinity
✅ **Message Queues**: When to use, Kafka vs RabbitMQ
✅ **Microservices**: When to use, communication patterns
✅ **Distributed Systems**: Consistency, locks, leader election
✅ **API Design**: REST vs GraphQL vs gRPC, versioning
✅ **Security & Reliability**: Auth, rate limiting, high availability

**Next Steps:**
- Review each FAQ
- Practice explaining concepts
- Apply to real problems
- Study related topics

---

*This guide is continuously updated with new FAQs from real interviews.*

