# Data Replication & Consistency

## Table of Contents
1. [Introduction](#introduction)
2. [Master-Slave Replication](#master-slave-replication)
3. [Master-Master Replication](#master-master-replication)
4. [Eventual Consistency](#eventual-consistency)
5. [Strong Consistency](#strong-consistency)
6. [Consistency Models](#consistency-models)
7. [Real-World Examples](#real-world-examples)

---

## Introduction

**Replication** copies data across multiple nodes for:
- **Availability:** If one node fails, others continue
- **Performance:** Distribute read load
- **Geographic Distribution:** Serve users worldwide

---

## Master-Slave Replication

**One master handles writes, multiple slaves handle reads.**

### Architecture

```
┌──────────┐
│  Master  │  ← Handles all writes
└────┬─────┘
     │ Replication (async)
     │
┌────┼────┬────┐
│    │    │    │
▼    ▼    ▼    ▼
┌──┐ ┌──┐ ┌──┐ ┌──┐
│S1│ │S2│ │S3│ │S4│  ← Handle reads
└──┘ └──┘ └──┘ └──┘
```

### Flow

```
Write:
1. Write to master
2. Master replicates to slaves (async)
3. Return success

Read:
1. Read from any slave
2. Fast (distributed load)
```

### Pros

- ✅ Scale reads horizontally
- ✅ Reduce master load
- ✅ Geographic distribution
- ✅ Simple to implement

### Cons

- ❌ Single point of failure (master)
- ❌ Replication lag (eventual consistency)
- ❌ Master can become bottleneck

### Failover

```
Master fails:
1. Promote one slave to master
2. Other slaves replicate from new master
3. Update application config
```

---

## Master-Master Replication

**Multiple masters handle writes and reads.**

### Architecture

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
```

### Flow

```
Write:
1. Write to any master
2. Masters sync with each other
3. Return success

Read:
1. Read from any master
2. Fast (no single point)
```

### Pros

- ✅ No single point of failure
- ✅ Write to any master
- ✅ High availability

### Cons

- ❌ Conflict resolution needed
- ❌ Complex to implement
- ❌ Eventual consistency

### Conflict Resolution

```
Master 1: Update user balance = 100
Master 2: Update user balance = 200

Conflict Resolution:
- Last write wins
- Timestamp-based
- Application-level logic
```

---

## Eventual Consistency

**All nodes will see updates eventually.**

### Example

```
Write to Node 1:
Node 1: Value = A ✅
Node 2: Value = A (after replication)
Node 3: Value = A (after replication)

Eventually all nodes show A
```

### Use Cases

- Social media likes
- Product views
- Analytics data
- Non-critical data

### Trade-offs

**Pros:**
- ✅ High availability
- ✅ Better performance
- ✅ Handles network partitions

**Cons:**
- ❌ Stale data possible
- ❌ Complex to reason about

---

## Strong Consistency

**All nodes see updates immediately.**

### Example

```
Write to Node 1:
Node 1: Value = A ✅
Node 2: Value = A ✅ (immediately)
Node 3: Value = A ✅ (immediately)
```

### Use Cases

- Financial transactions
- Inventory management
- Critical operations

### Trade-offs

**Pros:**
- ✅ Always correct data
- ✅ Easy to reason about

**Cons:**
- ❌ Slower (wait for replication)
- ❌ Lower availability

---

## Consistency Models

### 1. **Strong Consistency**

All reads see latest write.

```
Write → All nodes updated → Read sees latest
```

---

### 2. **Eventual Consistency**

All reads will eventually see latest write.

```
Write → Nodes update eventually → Read may see stale
```

---

### 3. **Causal Consistency**

Causally related operations seen in order.

```
If A happens before B, all nodes see A before B
```

---

### 4. **Session Consistency**

Consistency within a session.

```
User session sees consistent data
Different sessions may see different data
```

---

## Real-World Examples

### Example 1: E-Commerce Inventory

**Strategy:** Strong Consistency

```
Reserve inventory:
1. Lock inventory item
2. Update all replicas
3. Confirm reservation

Required for: Preventing overselling
```

---

### Example 2: Social Media Likes

**Strategy:** Eventual Consistency

```
Like a post:
1. Update like count on one node
2. Replicate to other nodes
3. Acceptable delay (few seconds)

Required for: High availability, performance
```

---

### Example 3: User Profile

**Strategy:** Session Consistency

```
Update profile:
1. Update on master
2. Replicate to slaves
3. User sees update immediately (session)
4. Others may see old data briefly

Required for: Good UX, acceptable consistency
```

---

## Summary

**Key Concepts:**
1. ✅ **Master-Slave:** Simple, scale reads
2. ✅ **Master-Master:** High availability
3. ✅ **Eventual Consistency:** Performance, availability
4. ✅ **Strong Consistency:** Correctness, reliability
5. ✅ **Choose based on use case**

**Next Steps:**
- Learn [Database Design](./10_DATABASE_DESIGN.md)
- Understand [Reliability](./05_RELIABILITY_AND_FAULT_TOLERANCE.md)
- Practice [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [Data Replication](https://algomaster.io/learn/system-design)
- [Consistency Models](https://en.wikipedia.org/wiki/Consistency_model)
