# Caching Strategies

## Table of Contents
1. [Introduction](#introduction)
2. [Cache-Aside Pattern](#cache-aside-pattern)
3. [Write-Through Pattern](#write-through-pattern)
4. [Write-Back Pattern](#write-back-pattern)
5. [Cache Invalidation](#cache-invalidation)
6. [Cache Coherence](#cache-coherence)
7. [Real-World Examples](#real-world-examples)

---

## Introduction

**Caching** stores frequently accessed data in fast storage to improve performance.

### Cache Levels

```
Level 1: Application Cache (In-Memory) - Fastest
Level 2: Distributed Cache (Redis) - Fast
Level 3: CDN Cache - Medium
Level 4: Database - Slowest
```

---

## Cache-Aside Pattern (Lazy Loading)

**Most common pattern:** Application manages cache.

### Flow

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

### Example

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

public void updateProduct(String id, Product product) {
    // 1. Write to DB
    database.updateProduct(id, product);
    
    // 2. Invalidate cache
    cache.delete("product:" + id);
}
```

### Pros

- ✅ Simple to implement
- ✅ Cache only what's accessed
- ✅ Handles cache failures gracefully

### Cons

- ❌ Cache miss penalty (2 round trips)
- ❌ Stale data possible (between write and invalidation)

---

## Write-Through Pattern

**Cache and database updated together.**

### Flow

```
Write:
1. Write to cache
2. Write to DB
3. Return

Read:
1. Read from cache (always available)
```

### Example

```java
public void updateProduct(String id, Product product) {
    // 1. Write to cache
    cache.set("product:" + id, product, 3600);
    
    // 2. Write to DB
    database.updateProduct(id, product);
}

public Product getProduct(String id) {
    // Always read from cache
    return cache.get("product:" + id);
}
```

### Pros

- ✅ Cache always consistent
- ✅ Fast reads (always in cache)

### Cons

- ❌ Slower writes (2 writes)
- ❌ Wasted cache if data rarely read

---

## Write-Back Pattern (Write-Behind)

**Write to cache first, DB later.**

### Flow

```
Write:
1. Write to cache
2. Return immediately
3. Async write to DB (later)

Read:
1. Read from cache
```

### Example

```java
public void updateProduct(String id, Product product) {
    // 1. Write to cache
    cache.set("product:" + id, product, 3600);
    
    // 2. Return immediately
    // 3. Async write to DB
    asyncExecutor.execute(() -> {
        database.updateProduct(id, product);
    });
}
```

### Pros

- ✅ Very fast writes
- ✅ Can batch DB writes

### Cons

- ❌ Risk of data loss (cache failure)
- ❌ Complex to implement
- ❌ Eventual consistency

---

## Cache Invalidation

### Strategies

#### 1. **TTL (Time To Live)**

Cache expires after time.

```java
cache.set("product:" + id, product, 3600);  // 1 hour
```

**Pros:** Simple
**Cons:** May serve stale data

---

#### 2. **Explicit Invalidation**

Delete cache on update.

```java
public void updateProduct(String id, Product product) {
    database.updateProduct(id, product);
    cache.delete("product:" + id);  // Invalidate
}
```

**Pros:** Always fresh
**Cons:** Requires invalidation logic

---

#### 3. **Event-Based Invalidation**

Invalidate on events.

```java
@EventListener
public void onProductUpdated(ProductUpdatedEvent event) {
    cache.delete("product:" + event.getProductId());
}
```

---

## Cache Coherence

**Ensuring all cache nodes have consistent data.**

### Problem

```
Cache Node 1: product:123 = Product A
Cache Node 2: product:123 = Product B (stale)
```

### Solutions

#### 1. **Cache Invalidation Messages**

```
Update → Invalidate all cache nodes
```

#### 2. **Cache Versioning**

```
Cache stores version number
Compare versions to detect stale data
```

---

## Real-World Examples

### Example 1: E-Commerce Product Page

**Strategy:** Cache-Aside with TTL

```java
// Cache hot products for 1 hour
public Product getProduct(String id) {
    Product product = cache.get("product:" + id);
    if (product == null) {
        product = database.getProduct(id);
        cache.set("product:" + id, product, 3600);
    }
    return product;
}
```

**Result:** 80% cache hit rate, 10x faster

---

### Example 2: User Session

**Strategy:** Write-Through

```java
// Session always in cache
public void updateSession(String sessionId, Session session) {
    cache.set("session:" + sessionId, session, 1800);  // 30 min
    database.saveSession(sessionId, session);
}
```

---

### Example 3: Analytics Data

**Strategy:** Write-Back

```java
// Batch analytics writes
public void trackEvent(Event event) {
    cache.append("analytics:queue", event);
    // Batch write to DB every 5 minutes
}
```

---

## Summary

**Key Patterns:**
1. ✅ **Cache-Aside:** Most common, simple
2. ✅ **Write-Through:** Always consistent
3. ✅ **Write-Back:** Fastest writes
4. ✅ **Invalidation:** TTL, Explicit, Event-based
5. ✅ **Coherence:** Versioning, invalidation messages

**Next Steps:**
- Learn [Data Replication](./12_DATA_REPLICATION_AND_CONSISTENCY.md)
- Understand [Performance Optimization](./06_PERFORMANCE_OPTIMIZATION.md)
- Practice [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [Caching Patterns](https://algomaster.io/learn/system-design)
- [Redis Caching Guide](../imp/REDIS_CACHING_GUIDE.md)
