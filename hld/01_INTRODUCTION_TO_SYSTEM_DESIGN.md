# Introduction to System Design

## Table of Contents
1. [What is System Design?](#what-is-system-design)
2. [Why System Design Matters](#why-system-design-matters)
3. [The 10 Big Questions of System Design](#the-10-big-questions-of-system-design)
4. [System Design Process](#system-design-process)
5. [Real-World Analogy](#real-world-analogy)
6. [Key Components Overview](#key-components-overview)
7. [System Design vs Low-Level Design](#system-design-vs-low-level-design)

---

## What is System Design?

**System Design** is the process of defining how different parts of a software system interact to meet both **functional** (what it should do) and **non-functional** (how well it should do it) requirements.

It's not about writing code, at least not yet. It's about making **high-level architectural decisions** that balance:
- **Scalability** - Can it handle growth?
- **Reliability** - Will it work consistently?
- **Performance** - Is it fast enough?
- **Cost** - Is it affordable?

### Real-World Example: Instagram

When you scroll through Instagram, you're interacting with a complex system:

```
User Action: Scroll feed
    │
    ▼
┌─────────────────────────────────────┐
│ 1. Load Balancer                    │
│    Routes request to available     │
│    server                           │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 2. API Gateway                    │
│    Handles authentication,         │
│    rate limiting                   │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 3. Feed Service                     │
│    Generates personalized feed      │
└──────────────┬──────────────────────┘
               │
        ┌───────┴───────┐
        │               │
        ▼               ▼
┌───────────┐   ┌───────────┐
│   Cache   │   │  Database  │
│  (Redis)  │   │ (Postgres) │
└───────────┘   └───────────┘
```

**Behind the scenes:**
- Load balancer distributes 1 billion+ requests/day
- CDN serves images from edge locations worldwide
- Database is sharded across multiple servers
- Cache reduces database load by 90%
- Message queue handles async tasks (likes, comments)

---

## Why System Design Matters

### 1. **Scalability**

**Problem:** Your app works great with 100 users, but crashes with 10,000 users.

**Solution:** Design for scale from the beginning.

**Example: Twitter's Evolution**

```
2006: Single Server
┌─────────────────┐
│  Twitter Server  │
│  (All features)  │
└─────────────────┘
Capacity: 1,000 tweets/day

2010: Distributed System
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Tweet   │  │ Timeline │  │ User    │
│ Service │  │ Service  │  │ Service │
└─────────┘  └─────────┘  └─────────┘
Capacity: 500M tweets/day
```

### 2. **Reliability**

**Problem:** Your system goes down, losing revenue and user trust.

**Solution:** Design for fault tolerance.

**Example: Netflix's Resilience**

```
Single Point of Failure:
┌──────────┐
│  Server  │  ❌ If this fails, entire system down
└──────────┘

Fault Tolerant:
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Server 1  │  │ Server 2  │  │ Server 3 │
└──────────┘  └──────────┘  └──────────┘
✅ If one fails, others continue
```

### 3. **Performance**

**Problem:** Your app is slow, users leave.

**Solution:** Optimize at every layer.

**Example: Amazon's Performance**

```
Without Optimization:
User Request → Database → Response
Time: 2 seconds ❌

With Optimization:
User Request → CDN → Cache → Database → Response
Time: 50ms ✅
```

### 4. **Cost Efficiency**

**Problem:** Infrastructure costs are too high.

**Solution:** Right-size resources, use auto-scaling.

**Example: Auto-Scaling**

```
Fixed Capacity (24/7):
10 servers × $100/month = $1,000/month
Even during low traffic ❌

Auto-Scaling:
2 servers (normal) + 8 servers (peak)
Average: $400/month ✅
```

---

## The 10 Big Questions of System Design

Every system design interview and real-world project revolves around these 10 questions:

### 1. **Scalability**
**Question:** How will the system handle a large number of users or requests simultaneously?

**Example: E-Commerce Black Friday**

```
Normal Day: 1,000 orders/hour
Black Friday: 100,000 orders/hour

Solution:
- Horizontal scaling (add more servers)
- Database sharding
- Caching
- Load balancing
```

**Architecture:**
```
┌──────────┐
│  Client  │
└────┬─────┘
     │
     ▼
┌──────────────┐
│ Load         │
│ Balancer    │
└──────┬───────┘
       │
   ┌───┴───┬───┬───┐
   │       │   │   │
   ▼       ▼   ▼   ▼
┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
│App  │ │App  │ │App  │ │App  │
│Srv 1│ │Srv 2│ │Srv 3│ │Srv N│
└─────┘ └─────┘ └─────┘ └─────┘
```

---

### 2. **Latency and Performance**
**Question:** How can we reduce response time and ensure low-latency performance under load?

**Example: Google Search**

```
User types: "best restaurants"
Response time: < 100ms

How?
- CDN for static content
- In-memory caching
- Database indexing
- Parallel processing
```

**Performance Optimization Stack:**
```
┌─────────────────┐
│   CDN Layer     │  ← Static content (10ms)
├─────────────────┤
│  Cache Layer    │  ← Hot data (5ms)
├─────────────────┤
│ Application     │  ← Business logic (20ms)
├─────────────────┤
│  Database       │  ← Persistent data (50ms)
└─────────────────┘
Total: ~85ms
```

---

### 3. **Communication**
**Question:** How do different components of the system interact with each other?

**Example: Microservices Communication**

```
Synchronous (REST/gRPC):
Order Service → Payment Service → Response
Fast but tight coupling

Asynchronous (Kafka/RabbitMQ):
Order Service → Event → Payment Service
Slower but loose coupling
```

**Communication Patterns:**
```
┌──────────┐         ┌──────────┐
│ Service A│────────▶│ Service B│  REST/gRPC
└──────────┘         └──────────┘  (Synchronous)

┌──────────┐         ┌──────────┐
│ Service A│──Event─▶│  Kafka   │──Event─▶│ Service B│
└──────────┘         └──────────┘         └──────────┘
                      (Asynchronous)
```

---

### 4. **Data Management**
**Question:** How should we store, retrieve, and manage data efficiently?

**Example: Choosing the Right Database**

```
Relational (PostgreSQL):
- User accounts, orders, transactions
- ACID compliance needed

NoSQL (MongoDB):
- Product catalogs, user profiles
- Flexible schema

Cache (Redis):
- Session data, hot data
- Fast access needed

Time-Series (InfluxDB):
- Metrics, logs
- Time-based queries
```

**Data Architecture:**
```
┌─────────────────────────────────┐
│      Application Layer         │
└──────────────┬──────────────────┘
               │
    ┌──────────┼──────────┐
    │          │          │
    ▼          ▼          ▼
┌────────┐ ┌────────┐ ┌────────┐
│  Cache  │ │  SQL   │ │ NoSQL  │
│ (Redis) │ │(Postgres│ │(Mongo) │
└────────┘ └────────┘ └────────┘
```

---

### 5. **Fault Tolerance and Reliability**
**Question:** What happens if a part of the system crashes or becomes unreachable?

**Example: Netflix's Chaos Engineering**

```
Single Server:
┌──────────┐
│  Server  │  ❌ If fails, 100% downtime
└──────────┘

Redundant System:
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Server 1 │  │ Server 2 │  │ Server 3 │
└──────────┘  └──────────┘  └──────────┘
✅ If one fails, 0% downtime (others handle load)
```

**Fault Tolerance Patterns:**
```
┌─────────────────────────────────┐
│   Active-Active (All servers    │
│   handle traffic)               │
│                                 │
│  ┌─────┐  ┌─────┐  ┌─────┐    │
│  │ S1  │  │ S2  │  │ S3  │    │
│  └─────┘  └─────┘  └─────┘    │
└─────────────────────────────────┘

┌─────────────────────────────────┐
│   Active-Passive (Backup ready) │
│                                 │
│  ┌─────┐ (Active)               │
│  │ S1  │                        │
│  └─────┘                        │
│  ┌─────┐ (Standby)              │
│  │ S2  │                        │
│  └─────┘                        │
└─────────────────────────────────┘
```

---

### 6. **Security**
**Question:** How do we protect the system against threats such as unauthorized access, data breaches, or denial-of-service attacks?

**Example: Multi-Layer Security**

```
Layer 1: Network Security
- Firewall
- DDoS protection
- VPN

Layer 2: Application Security
- Authentication (JWT, OAuth)
- Authorization (RBAC)
- Input validation

Layer 3: Data Security
- Encryption at rest
- Encryption in transit (TLS)
- Data masking
```

**Security Architecture:**
```
┌─────────────────────────────────┐
│      DDoS Protection            │  ← Block malicious traffic
├─────────────────────────────────┤
│      API Gateway                │  ← Rate limiting, auth
├─────────────────────────────────┤
│      Application                │  ← Input validation
├─────────────────────────────────┤
│      Database                   │  ← Encryption, access control
└─────────────────────────────────┘
```

---

### 7. **Maintainability and Extensibility**
**Question:** How easy is it to maintain, monitor, debug, and evolve the system over time?

**Example: Microservices vs Monolith**

```
Monolith:
┌─────────────────────────┐
│  All Code in One Place  │  ❌ Hard to maintain
│  - User Service         │  ❌ Hard to scale
│  - Order Service        │  ❌ Hard to deploy
│  - Payment Service      │
└─────────────────────────┘

Microservices:
┌──────────┐  ┌──────────┐  ┌──────────┐
│   User   │  │  Order   │  │ Payment  │  ✅ Easy to maintain
│ Service  │  │ Service  │  │ Service  │  ✅ Easy to scale
└──────────┘  └──────────┘  └──────────┘  ✅ Easy to deploy
```

---

### 8. **Cost Efficiency**
**Question:** How can we balance performance with infrastructure cost?

**Example: Cost Optimization**

```
Over-Provisioned:
100 servers × $100/month = $10,000/month
Only using 20% capacity ❌

Right-Sized:
20 servers × $100/month = $2,000/month
Using 80% capacity ✅

Auto-Scaling:
10 servers (normal) + 10 servers (peak)
Average: $1,500/month ✅✅
```

---

### 9. **Observability and Monitoring**
**Question:** How do we monitor system health and diagnose issues in production?

**Example: Three Pillars of Observability**

```
1. Logs (What happened?)
   - Application logs
   - Error logs
   - Access logs

2. Metrics (How is it performing?)
   - CPU usage
   - Memory usage
   - Request rate
   - Error rate

3. Traces (Where is the bottleneck?)
   - Request flow
   - Service dependencies
   - Latency breakdown
```

**Monitoring Stack:**
```
┌─────────────────────────────────┐
│      Application                │
└──────────┬──────────────────────┘
           │
    ┌──────┼──────┐
    │      │      │
    ▼      ▼      ▼
┌──────┐ ┌──────┐ ┌──────┐
│ Logs  │ │Metrics│ │Traces│
│(ELK)  │ │(Prom) │ │(Jaeger)│
└──────┘ └──────┘ └──────┘
```

---

### 10. **Compliance and Privacy**
**Question:** Are we complying with relevant laws and regulations (e.g., GDPR, HIPAA)?

**Example: GDPR Compliance**

```
Requirements:
- User data encryption
- Right to be forgotten
- Data portability
- Consent management

Implementation:
┌─────────────────────────────────┐
│   Data Encryption (at rest)     │
├─────────────────────────────────┤
│   Data Anonymization           │
├─────────────────────────────────┤
│   Audit Logging                │
├─────────────────────────────────┤
│   Consent Management System    │
└─────────────────────────────────┘
```

---

## System Design Process

The system design process follows these steps:

### Step 1: **Requirements Gathering**

**Functional Requirements:**
- What should the system do?
- What features are needed?

**Non-Functional Requirements:**
- How many users?
- What's the expected load?
- What's the acceptable latency?
- What's the availability target?

**Example: Design a URL Shortener**

```
Functional Requirements:
- Shorten long URLs
- Redirect to original URL
- Track click analytics

Non-Functional Requirements:
- 100M URLs/day
- 200ms redirect latency
- 99.9% availability
```

---

### Step 2: **Back-of-the-Envelope Estimation**

Estimate the scale of your system:

```
URL Shortener Example:

Write Operations:
- 100M URLs/day = 1,160 URLs/second
- Each URL: 500 bytes
- Write: 580 KB/second

Read Operations:
- 100:1 read/write ratio
- 100M × 100 = 10B reads/day
- 115,740 reads/second

Storage:
- 100M URLs/day × 365 days = 36.5B URLs/year
- 36.5B × 500 bytes = 18.25 TB/year

Bandwidth:
- 115,740 reads/second × 500 bytes = 58 MB/second
```

---

### Step 3: **High-Level Design (HLD)**

Sketch the core components and how they interact:

```
URL Shortener HLD:

┌──────────┐
│  Client  │
└────┬─────┘
     │
     ▼
┌──────────────┐
│ Load         │
│ Balancer    │
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│ API │ │ API │
│Srv 1│ │Srv 2│
└──┬──┘ └──┬──┘
   │       │
   └───┬───┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│Cache│ │ DB  │
│Redis│ │MySQL│
└─────┘ └─────┘
```

---

### Step 4: **Data Model / API Design**

Define schemas and APIs:

```
URL Shortener:

Database Schema:
- short_url (PK): varchar(7)
- long_url: varchar(2048)
- created_at: timestamp
- expires_at: timestamp

API Endpoints:
POST /api/v1/shorten
  Request: { "long_url": "https://..." }
  Response: { "short_url": "abc1234" }

GET /{short_url}
  Response: 302 Redirect to long_url
```

---

### Step 5: **Detailed Design / Deep Dive**

Zoom into each component:

```
URL Shortener Deep Dive:

1. Short URL Generation:
   - Base62 encoding (a-z, A-Z, 0-9)
   - 7 characters = 62^7 = 3.5 trillion URLs
   - Use hash of long URL + counter for uniqueness

2. Database Sharding:
   - Shard by short_url hash
   - 10 shards for 100M URLs/day

3. Caching Strategy:
   - Cache 20% most popular URLs (80/20 rule)
   - Redis with 24-hour TTL
```

---

### Step 6: **Identify Bottlenecks and Trade-offs**

Every design has trade-offs:

```
URL Shortener Trade-offs:

1. Database vs Cache:
   - Cache: Fast but limited memory
   - Database: Slower but persistent
   → Use both: Cache for hot data, DB for all

2. Consistency vs Availability:
   - Strong consistency: Slower writes
   - Eventual consistency: Faster but may have stale data
   → Accept eventual consistency for reads

3. Short URL Length:
   - 6 chars: More collisions
   - 7 chars: Better uniqueness
   → Use 7 chars with collision handling
```

---

### Step 7: **Review, Explain, and Iterate**

Step back and evaluate:

```
Review Checklist:
✅ Does it meet functional requirements?
✅ Does it meet non-functional requirements?
✅ Are bottlenecks addressed?
✅ Are trade-offs justified?
✅ Is it scalable?
✅ Is it reliable?
```

---

## Real-World Analogy: Designing a Skyscraper

System design is like designing a skyscraper:

### 1. **Requirements**
- How many floors? (Scale)
- How many people? (Users)
- What's the budget? (Cost)

### 2. **Architecture**
- Foundation (Database)
- Structural supports (Load Balancers)
- Elevators (APIs)
- Plumbing (Data Flow)
- Electrical (Caching)

### 3. **Considerations**
- Future expansion (Scalability)
- Earthquake resistance (Fault Tolerance)
- Fire safety (Security)
- Maintenance access (Monitoring)

---

## Key Components Overview

A typical software system has these components:

```
┌─────────────────────────────────────────┐
│           Client/Frontend               │
│    (Web, Mobile, Desktop Apps)          │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│         Load Balancer                   │
│    (Distributes traffic)                │
└──────────────────┬──────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────┐
│         API Gateway                    │
│    (Auth, Rate Limiting, Routing)       │
└──────────────────┬──────────────────────┘
                   │
        ┌──────────┼──────────┐
        │          │          │
        ▼          ▼          ▼
┌──────────┐ ┌──────────┐ ┌──────────┐
│Application│ │  Cache   │ │ Message  │
│ Services  │ │ (Redis)   │ │  Queue   │
└─────┬────┘ └──────────┘ └──────────┘
      │
      ▼
┌─────────────────────────────────────────┐
│         Database Layer                 │
│    (SQL, NoSQL, Time-Series)           │
└─────────────────────────────────────────┘
```

---

## System Design vs Low-Level Design

| Aspect | System Design (HLD) | Low-Level Design (LLD) |
|--------|---------------------|------------------------|
| **Focus** | "What" and "How Well" | "How" |
| **Level** | High-level architecture | Detailed implementation |
| **Scope** | Entire system | Individual components |
| **Questions** | How to scale? Which database? | Which class? Which method? |
| **Example** | "We need a Notification Service" | "Use NotificationSender interface with EmailSender, SmsSender" |

**Example:**

**System Design (HLD):**
```
"We need a notification service that can send 
100M notifications/day with 99.9% availability.
It should support email, SMS, and push notifications."
```

**Low-Level Design (LLD):**
```java
interface NotificationSender {
    void send(Notification notification);
}

class EmailSender implements NotificationSender { ... }
class SmsSender implements NotificationSender { ... }
class PushNotificationSender implements NotificationSender { ... }
```

---

## Summary

System Design is about:
1. ✅ **Understanding requirements** (functional and non-functional)
2. ✅ **Making architectural decisions** (scalability, reliability, performance)
3. ✅ **Balancing trade-offs** (cost vs performance, consistency vs availability)
4. ✅ **Designing for scale** (from day one)
5. ✅ **Communicating clearly** (explain your choices)

**Next Steps:**
- Learn [Core System Design Concepts](./02_CORE_SYSTEM_DESIGN_CONCEPTS.md)
- Understand [Key Components](./03_KEY_COMPONENTS.md)
- Practice with [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [AlgoMaster System Design Guide](https://algomaster.io/learn/system-design/what-is-system-design)
- [System Design Interview Guide](https://www.geeksforgeeks.org/system-design/)

