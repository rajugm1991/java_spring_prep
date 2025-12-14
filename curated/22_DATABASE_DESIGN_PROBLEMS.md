# Database Design Problems - Real-World Problems

> **Curated from top tech companies and real production scenarios**

---

## Problem 1: Design Database for E-Commerce Platform

**Problem Statement:**
Design database schema for e-commerce platform with 10M users and 1M orders per day.

**Requirements:**
- Users, products, orders, payments
- High read/write throughput
- Scalability
- Data consistency

**Solution:**

```sql
-- Normalized Schema
CREATE TABLE users (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    email VARCHAR(255) UNIQUE NOT NULL,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_email (email)
);

CREATE TABLE products (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    price DECIMAL(10,2) NOT NULL,
    category_id BIGINT,
    INDEX idx_category (category_id),
    FULLTEXT INDEX idx_search (name)
);

CREATE TABLE orders (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id BIGINT NOT NULL,
    total_amount DECIMAL(10,2) NOT NULL,
    status VARCHAR(50) NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_user (user_id),
    INDEX idx_status (status)
);

-- Sharding Strategy
-- Shard by user_id: user_id % 4
```

**Key Considerations:**
- Normalization vs Denormalization
- Indexing strategy
- Sharding approach
- Read replicas

---

*This guide contains database design problems covering schema design, sharding, replication, and optimization.*

---

*This guide is continuously updated with new problems.*

