# Designing a Distributed Cache (Redis-like)

A distributed cache improves latency and reduces backend load, but it introduces correctness and operational tradeoffs. Staff interviews test whether you can design caching as a **system**, not just “add Redis”.

---

## 1) Requirements (clarify first)

### Functional requirements
- `GET(key)`, `SET(key, value, ttl)`
- Optional: `DEL`, atomic increments, locks, pub/sub, streams (if needed)
- Eviction policy under memory pressure

### Non-functional requirements
- Very low latency (sub-millisecond to a few ms)
- High throughput (100k–1M+ ops/s depending on scale)
- High availability (node failures, AZ failures)
- Predictable behavior on cache miss/stampede

### Key clarifying questions
- Is the cache **best-effort** or a correctness boundary?
- What’s the data volatility? (TTL strategy, invalidation needs)
- Are reads mostly hot keys? Any “celebrity” keys?
- What’s the acceptable stale data window?

---

## 2) Cache patterns (choose explicitly)

### 2.1 Cache-aside (lazy loading) — most common
1. Read from cache.
2. On miss, read DB, write cache, return.

**Pros:** simple, DB remains source of truth  
**Cons:** cache stampede risk; stale data until invalidated/TTL

### 2.2 Read-through
Cache layer fetches from DB on miss.

### 2.3 Write-through
Write goes to cache and DB synchronously.

### 2.4 Write-back (write-behind)
Write to cache first; async flush to DB.

**Pros:** fast writes  
**Cons:** data loss risk; complex recovery; not good for critical data

### 2.5 Refresh-ahead
Preemptively refresh hot keys before TTL expires.

---

## 3) Core architecture: sharding + replication

### 3.1 Sharding (horizontal scale)
Use **consistent hashing** to map keys to nodes:

```
key → hash(key) → ring → node
```

Benefits:
- Rebalancing affects fewer keys when nodes change.
- Supports large clusters.

### 3.2 Replication (availability)
Common model: primary-replica (leader-follower).

**Tradeoff:** replicas improve read scaling, but introduce replication lag.

### 3.3 Redis Cluster reality
- Built-in sharding across hash slots
- Automatic failover (depending on config)
- Operational cost: resharding, hot partitions, client behavior

---

## 4) Eviction policies and TTL strategy

### Eviction policies
- **LRU**: evict least recently used
- **LFU**: evict least frequently used
- **Random**: simple but often poor hit rate

### TTL best practices
- Use TTLs to avoid unbounded growth.
- Add **jitter** to TTLs to reduce synchronized expirations:
  - e.g., `ttl = baseTtl ± random(0..jitter)`

### Negative caching
Cache “not found” results briefly to protect DB from repeated misses (careful with newly-created objects).

---

## 5) Preventing cache stampedes (dogpile effect)

When a hot key expires, many requests can hit DB simultaneously.

Mitigations:
- **Single-flight / request coalescing**: only one request loads the key.
- **Locking** (careful):
  - short TTL lock key: `lock:{key}`
  - others wait/backoff or serve stale data
- **Stale-while-revalidate**:
  - serve slightly stale value; refresh in background

---

## 6) Consistency and invalidation

### The hard truth
**Cache invalidation is the hard part**. You must pick a strategy:

1. **TTL-only**: accept staleness window.
2. **Write-through**: update cache on writes.
3. **Event-driven invalidation**: publish change event → consumers invalidate keys.

In staff interviews, you should discuss:
- Acceptable staleness (business-driven)
- Which endpoints require strong freshness (prices? inventory?)

---

## 7) Failure modes and fallbacks

### Common failures
- Node down / failover
- Network partition
- Hot key overload
- Memory pressure eviction causing miss storms

### Degradation strategies
- Serve stale data for non-critical pages
- Rate limit cache misses (protect DB)
- Circuit breaker to DB for “nice to have” features

---

## 8) Observability (what you must monitor)

### Metrics
- Hit rate (overall + per route)
- P50/P95 latency
- Evictions rate
- Memory usage and fragmentation
- Hot keys (top N keys by ops)
- DB QPS changes (cache efficacy)

### Alerts
- Hit rate drop + DB QPS spike
- Evictions spike (memory pressure)
- Cache latency spike (cluster health)

---

## 9) Interview prompts to practice

1. Cache product catalog reads (high QPS) with strong correctness for price changes.
2. Design session storage (Redis) with high availability and low latency.
3. Prevent stampede on a single hot key during a sale event.
4. Cache invalidation strategy when product attributes update frequently.

---

## 10) Relationship to other guides

- For deep implementation details and examples, see:
  - `docs/imp/REDIS_CACHING_GUIDE.md`


