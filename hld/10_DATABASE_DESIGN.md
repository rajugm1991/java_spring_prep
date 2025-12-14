# Database Design

## Table of Contents
1. [Introduction](#introduction)
2. [SQL vs NoSQL](#sql-vs-nosql)
3. [ACID Properties](#acid-properties)
4. [CAP Theorem Deep Dive](#cap-theorem-deep-dive)
5. [Database Sharding Strategies](#database-sharding-strategies)
6. [Schema Design Best Practices](#schema-design-best-practices)
7. [Indexing Strategies](#indexing-strategies)
8. [Real-World Examples](#real-world-examples)

---

## Introduction

**Database Design** is crucial for system performance, scalability, and data integrity. Choosing the right database and designing schemas properly can make or break a system.

---

## SQL vs NoSQL

### SQL Databases (Relational)

**Characteristics:**
- Structured data with relationships
- ACID compliance
- SQL query language
- Fixed schema

**Examples:** PostgreSQL, MySQL, Oracle, SQL Server

**Use Cases:**
- Transactional systems (banking, e-commerce)
- Complex queries with joins
- Data integrity critical
- Structured data

**Example Schema:**
```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100) UNIQUE
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    total DECIMAL(10,2),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

---

### NoSQL Databases

#### 1. **Document Store (MongoDB)**

**Structure:** JSON-like documents

**Example:**
```json
{
  "user_id": 123,
  "name": "John",
  "orders": [
    {"order_id": 1, "total": 100},
    {"order_id": 2, "total": 200}
  ]
}
```

**Use Cases:**
- Content management
- User profiles
- Flexible schema needed

---

#### 2. **Key-Value Store (Redis)**

**Structure:** Key-value pairs

**Example:**
```
key: "user:123"
value: {"name": "John", "email": "john@example.com"}
```

**Use Cases:**
- Caching
- Session storage
- Real-time data

---

#### 3. **Column Store (Cassandra)**

**Structure:** Wide columns

**Use Cases:**
- Time-series data
- High write throughput
- Analytics

---

#### 4. **Graph Database (Neo4j)**

**Structure:** Nodes and relationships

**Use Cases:**
- Social networks
- Recommendation engines
- Fraud detection

---

### SQL vs NoSQL Comparison

| Aspect | SQL | NoSQL |
|--------|-----|-------|
| **Schema** | Fixed | Flexible |
| **ACID** | Yes | Usually No |
| **Scaling** | Vertical | Horizontal |
| **Queries** | Complex joins | Simple lookups |
| **Consistency** | Strong | Eventual |

---

## ACID Properties

**ACID** ensures reliable database transactions.

### A - Atomicity

**All or nothing:** Transaction either completes fully or not at all.

**Example:**
```sql
BEGIN TRANSACTION;
  UPDATE accounts SET balance = balance - 100 WHERE id = 'A';
  UPDATE accounts SET balance = balance + 100 WHERE id = 'B';
COMMIT;

If any step fails → Entire transaction rolls back
```

---

### C - Consistency

**Valid state transitions:** Database remains in valid state.

**Example:**
```sql
-- Constraint: balance >= 0
UPDATE accounts SET balance = -50 WHERE id = 'A';
-- ❌ Transaction fails (violates constraint)
```

---

### I - Isolation

**Concurrent transactions don't interfere.**

**Example:**
```
Transaction 1: Read balance = 100
Transaction 2: Read balance = 100
Transaction 1: Update balance = 90
Transaction 2: Update balance = 80

Isolation ensures transactions don't see each other's uncommitted changes
```

---

### D - Durability

**Committed data persists:** Even if system crashes.

**Example:**
```
Transaction commits → Data written to disk
System crashes → Data still available after restart
```

---

## CAP Theorem Deep Dive

**CAP Theorem:** Distributed system can guarantee only 2 out of 3:

- **C**onsistency: All nodes see same data
- **A**vailability: System responds to requests
- **P**artition Tolerance: Works despite network failures

### CAP Trade-offs

```
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

## Database Sharding Strategies

### 1. **Range-Based Sharding**

Partition by value ranges.

```
Shard 1: User IDs 1-1,000,000
Shard 2: User IDs 1,000,001-2,000,000
Shard 3: User IDs 2,000,001-3,000,000
```

**Pros:** Simple, easy range queries
**Cons:** Uneven distribution, hot shards

---

### 2. **Hash-Based Sharding**

Partition by hash of key.

```
hash(user_id) % number_of_shards = shard_number

User ID 123 → hash(123) % 3 = 0 → Shard 1
User ID 456 → hash(456) % 3 = 1 → Shard 2
```

**Pros:** Even distribution
**Cons:** Cross-shard queries difficult

---

### 3. **Directory-Based Sharding**

Use lookup table.

```
Lookup Table:
User ID 123 → Shard 1
User ID 456 → Shard 2
```

**Pros:** Flexible, easy rebalancing
**Cons:** Single point of failure

---

## Schema Design Best Practices

### 1. **Normalization**

Reduce data redundancy.

**Example:**
```
Before (Denormalized):
Orders Table:
- order_id
- user_name
- user_email
- product_name
- product_price
(Data duplicated)

After (Normalized):
Orders Table:
- order_id
- user_id
- product_id

Users Table:
- user_id
- user_name
- user_email

Products Table:
- product_id
- product_name
- product_price
```

---

### 2. **Denormalization (When Needed)**

Add redundancy for performance.

**Example:**
```
User Orders (Denormalized):
- order_id
- user_id
- user_name (denormalized for faster queries)
- total
```

**Trade-off:** Faster reads vs more storage

---

### 3. **Partitioning**

Split large tables.

```sql
-- Partition by date
CREATE TABLE orders_2024 PARTITION OF orders
FOR VALUES FROM ('2024-01-01') TO ('2025-01-01');
```

---

## Indexing Strategies

### Index Types

#### 1. **B-Tree Index**

**Use:** General purpose, range queries

```sql
CREATE INDEX idx_email ON users(email);
```

---

#### 2. **Hash Index**

**Use:** Equality lookups

```sql
CREATE INDEX idx_user_id ON orders USING HASH(user_id);
```

---

#### 3. **Composite Index**

**Use:** Multiple columns

```sql
CREATE INDEX idx_user_date ON orders(user_id, created_at);
```

---

### Index Best Practices

1. **Index frequently queried columns**
2. **Don't over-index** (slows writes)
3. **Use composite indexes** for multi-column queries
4. **Monitor index usage**

---

## Real-World Examples

### Example 1: E-Commerce Platform

**Database Choice:**
- **PostgreSQL:** Orders, users, products (ACID needed)
- **Redis:** Shopping cart, session (fast access)
- **MongoDB:** Product catalog (flexible schema)

---

### Example 2: Social Media Platform

**Database Choice:**
- **PostgreSQL:** User accounts, posts (relationships)
- **Cassandra:** User feeds (high write, eventual consistency)
- **Redis:** Trending topics (real-time)

---

### Example 3: Financial System

**Database Choice:**
- **PostgreSQL:** Transactions (ACID critical)
- **Redis:** Rate limiting, caching
- **Time-series DB:** Analytics

---

## Summary

**Key Takeaways:**
1. ✅ **SQL:** ACID, relationships, complex queries
2. ✅ **NoSQL:** Flexible, scalable, fast
3. ✅ **ACID:** Reliability for transactions
4. ✅ **CAP:** Choose 2 out of 3
5. ✅ **Sharding:** Scale horizontally
6. ✅ **Indexing:** Speed up queries

**Next Steps:**
- Learn [Caching Strategies](./11_CACHING_STRATEGIES.md)
- Understand [Data Replication](./12_DATA_REPLICATION_AND_CONSISTENCY.md)
- Practice [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [Database Design](https://algomaster.io/learn/system-design)
- [CAP Theorem](https://en.wikipedia.org/wiki/CAP_theorem)
