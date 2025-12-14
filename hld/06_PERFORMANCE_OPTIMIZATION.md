# Performance Optimization

## Table of Contents
1. [Introduction](#introduction)
2. [Caching Strategies](#caching-strategies)
3. [Database Optimization](#database-optimization)
4. [CDN Implementation](#cdn-implementation)
5. [Load Balancing](#load-balancing)
6. [Performance Monitoring](#performance-monitoring)
7. [Real-World Examples](#real-world-examples)

---

## Introduction

**Performance Optimization** improves system response time, throughput, and resource utilization.

### Performance Metrics

- **Latency:** Time to process a request
- **Throughput:** Requests per second
- **Response Time:** End-to-end time
- **Resource Utilization:** CPU, Memory, Network

---

## Caching Strategies

### Multi-Level Caching

```
Level 1: Application Cache (In-Memory)
┌─────────────────┐
│  Application    │
│  ┌───────────┐  │
│  │  Cache    │  │  ← Fastest (nanoseconds)
│  │  (Local)  │  │
│  └───────────┘  │
└─────────────────┘

Level 2: Distributed Cache (Redis)
┌──────────┐  ┌──────────┐
│ App 1    │  │ App 2    │
└────┬─────┘  └────┬─────┘
     │            │
     └─────┬──────┘
           │
      ┌────┴────┐
      │  Redis  │  ← Fast (milliseconds)
      └─────────┘

Level 3: CDN Cache
┌──────────┐
│   CDN    │  ← Fast (tens of milliseconds)
└──────────┘

Level 4: Database
┌──────────┐
│ Database │  ← Slowest (hundreds of milliseconds)
└──────────┘
```

### Cache Patterns

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
    // Check cache
    Product product = cache.get("product:" + id);
    if (product != null) {
        return product;  // Cache hit
    }
    
    // Cache miss - read from DB
    product = database.getProduct(id);
    
    // Update cache
    cache.set("product:" + id, product, 3600);
    
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

## Database Optimization

### 1. **Indexing**

**Without Index:**
```sql
SELECT * FROM users WHERE email = 'user@example.com'
-- Full table scan (1M rows = 500ms) ❌
```

**With Index:**
```sql
CREATE INDEX idx_email ON users(email)
SELECT * FROM users WHERE email = 'user@example.com'
-- Index lookup (1M rows = 5ms) ✅
```

**Index Types:**
- **B-Tree:** General purpose (PostgreSQL, MySQL)
- **Hash:** Equality lookups (PostgreSQL)
- **Bitmap:** Low cardinality columns

---

### 2. **Query Optimization**

**Bad Query:**
```sql
SELECT * FROM orders o
JOIN users u ON o.user_id = u.id
WHERE u.email LIKE '%@gmail.com'
ORDER BY o.created_at DESC
LIMIT 10
-- Scans all orders, then filters ❌
```

**Optimized Query:**
```sql
SELECT o.* FROM orders o
WHERE o.user_id IN (
    SELECT id FROM users WHERE email LIKE '%@gmail.com'
)
ORDER BY o.created_at DESC
LIMIT 10
-- Filters users first, then joins ✅
```

---

### 3. **Connection Pooling**

```
Without Pooling:
Request → Create Connection → Query → Close Connection
Request → Create Connection → Query → Close Connection
(Slow - creates connection each time) ❌

With Pooling:
Request → Get from Pool → Query → Return to Pool
Request → Get from Pool → Query → Return to Pool
(Fast - reuses connections) ✅
```

---

## CDN Implementation

### How CDN Works

```
Without CDN:
User (India) → Origin Server (US) → Response (500ms) ❌

With CDN:
User (India) → CDN Edge (India) → Response (50ms) ✅
```

### CDN Architecture

```
┌─────────────────────────────────┐
│      Origin Server              │
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

### CDN Use Cases

1. **Static Assets:** Images, CSS, JavaScript
2. **Video Streaming:** Netflix, YouTube
3. **API Responses:** Cached API responses
4. **Software Downloads:** Large files

---

## Load Balancing

### Algorithms

#### 1. **Round Robin**

```
Request 1 → Server 1
Request 2 → Server 2
Request 3 → Server 3
Request 4 → Server 1
```

#### 2. **Least Connections**

```
Server 1: 10 connections
Server 2: 5 connections
Server 3: 8 connections

New Request → Server 2 (least connections)
```

#### 3. **Weighted Round Robin**

```
Server 1 (Weight: 3) → Gets 3 requests
Server 2 (Weight: 2) → Gets 2 requests
Server 3 (Weight: 1) → Gets 1 request
```

---

## Performance Monitoring

### Key Metrics

1. **Response Time:** P50, P95, P99
2. **Throughput:** Requests per second
3. **Error Rate:** Percentage of failed requests
4. **Resource Usage:** CPU, Memory, Network

### Monitoring Stack

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

## Real-World Examples

### Example 1: Google Search

**Optimizations:**
- CDN for static content
- In-memory caching
- Database indexing
- Parallel processing

**Result:** < 100ms response time for billions of searches/day

---

### Example 2: Amazon

**Optimizations:**
- Multi-level caching
- Database read replicas
- CDN for product images
- Connection pooling

**Result:** Handles Black Friday traffic spikes

---

### Example 3: Netflix

**Optimizations:**
- CDN for video delivery
- Predictive caching
- Adaptive bitrate streaming
- Regional data centers

**Result:** Smooth streaming for 200M+ users

---

## Summary

**Key Optimizations:**
1. ✅ **Caching:** Multi-level, appropriate patterns
2. ✅ **Database:** Indexing, query optimization, pooling
3. ✅ **CDN:** Geographic distribution
4. ✅ **Load Balancing:** Appropriate algorithms
5. ✅ **Monitoring:** Track key metrics

**Next Steps:**
- Learn [Caching Strategies](./11_CACHING_STRATEGIES.md)
- Understand [Database Design](./10_DATABASE_DESIGN.md)
- Practice [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [Performance Optimization](https://algomaster.io/learn/system-design)
- [Caching Patterns](https://www.geeksforgeeks.org/system-design/)
