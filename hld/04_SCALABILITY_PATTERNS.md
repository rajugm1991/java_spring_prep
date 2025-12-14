# Scalability Patterns

## Table of Contents
1. [Introduction](#introduction)
2. [Vertical Scaling (Scale Up)](#vertical-scaling-scale-up)
3. [Horizontal Scaling (Scale Out)](#horizontal-scaling-scale-out)
4. [Database Sharding](#database-sharding)
5. [Partitioning Strategies](#partitioning-strategies)
6. [Auto-Scaling](#auto-scaling)
7. [Real-World Scaling Examples](#real-world-scaling-examples)

---

## Introduction

**Scalability** is the ability of a system to handle increased load by adding resources without significant performance degradation.

### Scaling Dimensions

1. **Load Scaling:** Handle more requests
2. **Data Scaling:** Handle more data
3. **Geographic Scaling:** Serve users worldwide
4. **Feature Scaling:** Add new features without breaking existing ones

---

## Vertical Scaling (Scale Up)

**Vertical Scaling** means adding more power to existing machines (CPU, RAM, Storage).

### Architecture

```
Before:
┌─────────────────┐
│  Server         │
│  4 CPU, 8GB RAM │
└─────────────────┘

After:
┌─────────────────┐
│  Server         │
│  16 CPU, 64GB   │
│  RAM            │
└─────────────────┘
```

### Pros

- ✅ Simple (no code changes)
- ✅ No distributed system complexity
- ✅ Single point of management
- ✅ No network latency between components

### Cons

- ❌ Limited by hardware maximums
- ❌ Expensive (high-end hardware)
- ❌ Single point of failure
- ❌ Downtime during upgrades

### When to Use

- Small to medium applications
- When simplicity is priority
- When single machine can handle load
- Development/staging environments

### Real-World Example: Small E-Commerce Site

```
Initial Setup:
┌─────────────────┐
│  Application    │
│  4 CPU, 16GB    │
│  Handles 1K     │
│  requests/sec   │
└─────────────────┘

After Scale Up:
┌─────────────────┐
│  Application    │
│  16 CPU, 64GB    │
│  Handles 5K     │
│  requests/sec   │
└─────────────────┘
```

---

## Horizontal Scaling (Scale Out)

**Horizontal Scaling** means adding more machines to handle load.

### Architecture

```
Before:
┌──────────┐
│ Server 1 │
└──────────┘

After:
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Server 1 │  │ Server 2 │  │ Server 3 │
└──────────┘  └──────────┘  └──────────┘
```

### Pros

- ✅ Unlimited scaling potential
- ✅ Cost-effective (commodity hardware)
- ✅ Fault tolerant (if one fails, others continue)
- ✅ No downtime for scaling

### Cons

- ❌ Complex (distributed system)
- ❌ Requires load balancing
- ❌ Data consistency challenges
- ❌ Network latency between services

### When to Use

- Large-scale applications
- High availability requirements
- When load exceeds single machine capacity
- Production environments

### Real-World Example: Instagram's Scaling

```
2010: Single Server
┌─────────────────┐
│  Instagram      │
│  Single Server  │
│  1M users       │
└─────────────────┘

2012: Horizontal Scaling
┌─────┐ ┌─────┐ ┌─────┐
│ S1  │ │ S2  │ │ S3  │
└─────┘ └─────┘ └─────┘
30M users

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
500M users
```

---

## Database Sharding

**Sharding** splits a database into smaller pieces (shards) distributed across multiple servers.

### Sharding Architecture

```
Single Database:
┌─────────────────┐
│   Database      │
│   (All Data)    │
└─────────────────┘

Sharded Database:
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Shard 1  │  │ Shard 2  │  │ Shard 3  │
│ (User    │  │ (User    │  │ (User    │
│  1-33M)  │  │  34-66M) │  │  67-100M)│
└──────────┘  └──────────┘  └──────────┘
```

### Sharding Strategies

#### 1. **Range-Based Sharding**

Partition data by value ranges.

```
Shard 1: User IDs 1-1,000,000
Shard 2: User IDs 1,000,001-2,000,000
Shard 3: User IDs 2,000,001-3,000,000
```

**Pros:**
- Simple to implement
- Easy to query ranges

**Cons:**
- Uneven distribution (hot shards)
- Difficult to rebalance

**Example:**
```sql
-- Shard selection
IF user_id BETWEEN 1 AND 1000000 THEN
    shard = 'shard1'
ELSE IF user_id BETWEEN 1000001 AND 2000000 THEN
    shard = 'shard2'
END IF
```

---

#### 2. **Hash-Based Sharding**

Partition data by hash of key.

```
hash(user_id) % number_of_shards = shard_number

User ID 123 → hash(123) % 3 = 0 → Shard 1
User ID 456 → hash(456) % 3 = 1 → Shard 2
User ID 789 → hash(789) % 3 = 2 → Shard 3
```

**Pros:**
- Even distribution
- Easy to add/remove shards

**Cons:**
- Cross-shard queries difficult
- Rebalancing requires data movement

**Example:**
```java
public String getShard(String userId) {
    int hash = userId.hashCode();
    int shardNumber = Math.abs(hash) % numberOfShards;
    return "shard" + shardNumber;
}
```

---

#### 3. **Directory-Based Sharding**

Use a lookup table to map keys to shards.

```
Lookup Table:
User ID 123 → Shard 1
User ID 456 → Shard 2
User ID 789 → Shard 3
```

**Pros:**
- Flexible (can move data easily)
- No hash collisions

**Cons:**
- Single point of failure (lookup table)
- Additional lookup overhead

**Example:**
```java
public String getShard(String userId) {
    // Query lookup table
    return shardLookupTable.get(userId);
}
```

---

### Sharding Challenges

#### 1. **Cross-Shard Queries**

**Problem:** Querying data across multiple shards is slow.

**Solution:** Denormalize data or use separate aggregation service.

```
Query: "Get all orders for user 123"
User 123 → Shard 1
Orders → Shard 2, 3, 4

Solution: Store user orders in user shard (denormalize)
```

---

#### 2. **Rebalancing**

**Problem:** Adding/removing shards requires data movement.

**Solution:** Use consistent hashing or virtual shards.

```
Consistent Hashing:
- Map shards to hash ring
- Add/remove shards only affects adjacent shards
- Minimal data movement
```

---

#### 3. **Transaction Management**

**Problem:** Transactions across shards are complex.

**Solution:** 
- Avoid cross-shard transactions when possible
- Use saga pattern for distributed transactions
- Accept eventual consistency

---

## Partitioning Strategies

### 1. **Functional Partitioning**

Split by functionality.

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ User     │  │ Order    │  │ Payment  │
│ Service  │  │ Service  │  │ Service  │
└──────────┘  └──────────┘  └──────────┘
```

**Example: Microservices Architecture**

---

### 2. **Data Partitioning**

Split data by access patterns.

```
Hot Data (Frequently accessed):
┌──────────┐
│  Cache   │
│  (Redis) │
└──────────┘

Cold Data (Rarely accessed):
┌──────────┐
│ Archive  │
│  (S3)    │
└──────────┘
```

---

### 3. **Geographic Partitioning**

Split by user location.

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ US       │  │ EU       │  │ Asia     │
│ Region   │  │ Region   │  │ Region   │
└──────────┘  └──────────┘  └──────────┘
```

**Example: CDN Distribution**

---

## Auto-Scaling

**Auto-Scaling** automatically adjusts resources based on load.

### Auto-Scaling Strategies

#### 1. **Reactive Scaling**

Scale based on current metrics.

```
CPU > 80% → Add server
CPU < 30% → Remove server
```

**Pros:**
- Responds to actual load
- Cost-effective

**Cons:**
- Delay in scaling (takes time to add servers)
- May not handle sudden spikes

---

#### 2. **Predictive Scaling**

Scale based on predicted load.

```
Black Friday coming → Pre-scale servers
Expected traffic spike → Add servers before
```

**Pros:**
- Handles known spikes
- Better user experience

**Cons:**
- Requires prediction accuracy
- May over-provision

---

#### 3. **Scheduled Scaling**

Scale based on time schedules.

```
9 AM - 5 PM: 10 servers (business hours)
5 PM - 9 AM: 5 servers (off hours)
```

**Pros:**
- Predictable costs
- Simple to implement

**Cons:**
- Doesn't adapt to actual load
- May under/over provision

---

### Auto-Scaling Example: AWS Auto Scaling

```yaml
AutoScalingGroup:
  MinSize: 2
  MaxSize: 10
  DesiredCapacity: 4
  ScalingPolicies:
    - Metric: CPUUtilization
      Threshold: 70
      Action: AddInstance
    - Metric: CPUUtilization
      Threshold: 30
      Action: RemoveInstance
```

---

## Real-World Scaling Examples

### Example 1: Twitter's Scaling Journey

```
2006: Single Server
┌─────────────────┐
│  Twitter        │
│  Single Server  │
└─────────────────┘
Users: 100K

2008: Vertical Scaling
┌─────────────────┐
│  Twitter        │
│  Bigger Server  │
└─────────────────┘
Users: 1M

2010: Horizontal Scaling
┌─────┐ ┌─────┐ ┌─────┐
│ S1  │ │ S2  │ │ S3  │
└─────┘ └─────┘ └─────┘
Users: 10M

2012: Distributed System
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
Users: 100M

Key Changes:
- Moved from Ruby to Java (performance)
- Implemented caching (Redis)
- Database sharding
- CDN for static content
```

---

### Example 2: Netflix's Scaling

```
Netflix Architecture:

┌─────────────────────────────────┐
│      CDN (Edge Servers)         │
│  ┌────┐ ┌────┐ ┌────┐ ┌────┐  │
│  │US  │ │EU  │ │Asia│ │... │  │
│  └────┘ └────┘ └────┘ └────┘  │
└─────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────┐
│      API Gateway               │
│    (Load Balancer)              │
└──────────────┬──────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
┌──────────┐   ┌──────────┐
│ Content  │   │ User     │
│ Service  │   │ Service  │
└──────────┘   └──────────┘

Scaling Strategies:
- CDN for video delivery (geographic scaling)
- Microservices (functional scaling)
- Auto-scaling based on demand
- Database replication
```

---

### Example 3: Amazon's Scaling

```
Amazon Architecture:

┌─────────────────────────────────┐
│      Load Balancer              │
└──────────────┬──────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
┌──────────┐   ┌──────────┐
│ Product  │   │ Order    │
│ Service  │   │ Service  │
└────┬─────┘   └────┬─────┘
     │              │
     ▼              ▼
┌──────────┐   ┌──────────┐
│ Product  │   │ Order    │
│ Database │   │ Database │
│ (Sharded)│   │ (Sharded)│
└──────────┘   └──────────┘

Scaling Strategies:
- Database sharding (by product category)
- Read replicas (for product catalog)
- Caching (for hot products)
- Auto-scaling (for Black Friday)
```

---

## Summary

**Key Takeaways:**
1. ✅ **Vertical Scaling:** Simple but limited
2. ✅ **Horizontal Scaling:** Complex but unlimited
3. ✅ **Database Sharding:** Essential for large datasets
4. ✅ **Partitioning:** Multiple strategies for different needs
5. ✅ **Auto-Scaling:** Automate resource management
6. ✅ **Real-World:** Learn from Twitter, Netflix, Amazon

**Next Steps:**
- Learn [Reliability & Fault Tolerance](./05_RELIABILITY_AND_FAULT_TOLERANCE.md)
- Understand [Performance Optimization](./06_PERFORMANCE_OPTIMIZATION.md)
- Practice [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [System Design Scalability](https://algomaster.io/learn/system-design)
- [Scaling Patterns](https://www.geeksforgeeks.org/system-design/)
