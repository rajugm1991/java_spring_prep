
# System Scalability – General Design Guide

## 1. What is Scalability?

Scalability is the ability of a system to handle increased load by adding resources while maintaining performance and reliability.

A scalable system should:
- Maintain predictable latency
- Increase throughput proportionally with resources
- Avoid single points of failure

---

## 2. Types of Scalability

### Vertical Scaling (Scale Up)
- Add more CPU, RAM, or storage to a single machine
- Limited by hardware limits
- Simple but not elastic

### Horizontal Scaling (Scale Out)
- Add more machines
- Requires stateless services
- Enables high availability

---

## 3. Core Scalability Metrics

| Metric | Description |
|------|-------------|
| Throughput | Requests per second (RPS) |
| Latency | Response time (P50, P95, P99) |
| Error Rate | Failed requests |
| Resource Utilization | CPU, memory, I/O |
| Saturation | Resource exhaustion level |

---

## 4. General Load Model

Define:
- Peak traffic (RPS)
- Read vs write ratio
- Latency SLO (Service Level Objective)

Example formula:
```
Required Capacity = Peak RPS × Safety Factor
```

---

## 5. Application Layer Scalability

### Key Concepts
- Stateless services
- Horizontal scaling
- Load balancing

### Capacity Calculation

```
Per-instance capacity = CPU available / CPU per request
```

Example:
```
2000 ms/sec ÷ 4 ms = 500 RPS
```

---

## 6. Cache Layer Scalability

### Purpose
- Reduce load on backend systems
- Improve response latency

### Key Metrics
- Cache hit ratio
- Eviction rate
- Memory usage

### Capacity Estimation
```
Cache Size = Number of Keys × Size per Key
```

---

## 7. Database Scalability

### Common Challenges
- Write amplification
- Lock contention
- Connection limits

### Scaling Techniques
- Read replicas
- Sharding
- Index optimization
- Async writes

---

## 8. Bottleneck Identification

| Symptom | Likely Cause |
|------|---------------|
| High latency | CPU or DB contention |
| High error rate | Resource exhaustion |
| Low throughput | Serialization or I/O |
| Spikes | Load imbalance |

---

## 9. Capacity Planning Formula

```
System Capacity =
min(
  App Capacity,
  Cache Capacity,
  Database Capacity
)
```

---

## 10. Autoscaling Strategy

### Triggers
- CPU utilization
- Request rate
- Queue depth

### Policies
- Scale out quickly
- Scale in slowly

---

## 11. Reliability Considerations

- Graceful degradation
- Circuit breakers
- Rate limiting
- Backpressure

---

## 12. Observability

### Metrics
- Latency (P95/P99)
- Error rate
- Saturation

### Tools
- Prometheus
- Grafana
- CloudWatch

---

## 13. Summary

A scalable system:
- Measures before scaling
- Eliminates bottlenecks
- Scales horizontally
- Maintains predictable performance

---

## 14. Key Principle

> Scalability is not about handling more traffic — it is about handling growth gracefully.



---Examples------


# URL Shortener – Scalability Design Guide

## 1. Overview
A URL shortener converts long URLs into short aliases and redirects users efficiently.
The system is read-heavy and must support very high throughput with low latency.
<br>
<b>System Characterstics </b>

| Aspect              | Value          |
| ------------------- | -------------- |
| Read-heavy          | ✅ Very high    |
| Write-heavy         | ❌ Low          |
| Latency sensitivity | Very high      |
| Cache dependency    | Extremely high |

---

## 2. Key Requirements

### Functional
- Create short URL
- Redirect short → long URL

### Non-Functional
- High availability (99.9%+)
- Low latency (<50ms)
- Horizontally scalable

---

## 3. Traffic Assumptions

| Metric | Value |
|------|------|
| Daily reads | 1,000,000,000 |
| Daily writes | 10,000,000 |
| Peak multiplier | 10× |
| Read:Write ratio | 100:1 |

### Convert to RPS

Reads/sec = 1B / 86400 ≈ 11,574  
Peak reads ≈ 115,000 RPS  

Writes/sec = 10M / 86400 ≈ 116  
Peak writes = ~1,160 RPS  

---

## 4. High-Level Architecture

Client  
→ Load Balancer  
→ App Servers (EC2)  
→ Redis Cache  
→ Relational DB  

---

## 5. Cache Layer (Redis)

Cache Hit Ratio: **95%**

Backend RPS:
115,000 × 0.05 = 5,750 RPS

---

## 6. Redis Memory Estimation

Hot URLs: 300M  
Per key ≈ 200 bytes  

Total memory ≈ 60 GB  

Redis Setup:
- 3 Masters + 3 Replicas
- Instance: r6g.2xlarge

---

## 7. Application Scaling

CPU cost per request: ~5ms  

2 vCPU = 2000 ms/sec  

Capacity per instance:
2000 / 5 = 400 RPS  

Required instances:
5750 / 400 ≈ 18

---

## 8. Database Load

Only cache misses hit DB:

~5,750 QPS  

Handled via read replicas and indexing.

---

## 9. Bottlenecks & Mitigation

| Issue | Solution |
|------|---------|
| Hot keys | Local cache / key sharding |
| Cache eviction | TTL + LRU |
| Redis failure | Replication & failover |
| Traffic spikes | Auto scaling |

---

## 10. Summary

- Cache-driven architecture
- Horizontally scalable
- Supports 100k+ RPS

----Example--2

# E-Commerce (Ekart-like) Scalability Design

## 1. Overview
An e-commerce platform supports browsing, cart, and checkout with strong consistency.

<br>
<b>System charterstics
</b>

---

| Aspect       | Value    |
| ------------ | -------- |
| Read-heavy   | Yes      |
| Write-heavy  | Also yes |
| Consistency  | Strong   |
| Transactions | Critical |
| Cache usage  | Partial  |

## 2. Traffic Assumptions

| Metric | Value |
|------|------|
| Daily active users | 20M |
| Peak concurrent users | 2M |
| Avg requests/user | 5/min |
| Peak RPS | ~170,000 |

---

## 3. Request Distribution

| Operation | % | RPS |
|----------|----|-----|
| Browse | 70% | 119,000 |
| Cart | 20% | 34,000 |
| Checkout | 10% | 17,000 |

---

## 4. Cache Strategy

| Area | Hit Rate |
|------|----------|
| Product catalog | 95% |
| Search | 90% |
| Cart | 60% |
| Checkout | 0% |

---

## 5. Backend Load

Browse: 119k × 5% = 5,950  
Cart: 34k × 40% = 13,600  
Checkout: 17k × 100% = 17,000  

Total ≈ 36,500 RPS  

---

## 6. CPU Cost Estimation

| Operation | CPU (ms) |
|-----------|----------|
| Browse | 2 |
| Cart | 6 |
| Checkout | 15 |

Weighted avg ≈ 9 ms  

---

## 7. Application Scaling

2 vCPU = 2000 ms/sec  
2000 / 9 ≈ 222 RPS per instance  

Required instances ≈ 165  
With buffer ≈ 200 instances  

---

## 8. Database Load

| Area | QPS |
|------|-----|
| Orders | 17k |
| Cart | 13.6k |
| Inventory | 17k |

Use partitioning + replicas.

---

## 9. Key Bottlenecks

| Area | Risk |
|------|------|
| Inventory | Overselling |
| Checkout | Lock contention |
| DB | Write saturation |
| Cache | Invalidation |

---

## 10. Summary

E-commerce systems require careful write handling, strong consistency, and controlled scaling.

--------Database scalling techniques--


# Database Scaling Using Partitioning and Replication

## 1. Problem Statement
High-scale systems must handle massive read and write traffic while maintaining low latency and high availability.

### Example Load
| Area | QPS |
|------|------|
| Orders | 17,000 |
| Cart | 13,600 |
| Inventory | 17,000 |
| **Total** | **47,600 QPS** |

---

## 2. Core Scaling Techniques

| Technique | Purpose |
|----------|----------|
| Partitioning (Sharding) | Scale writes |
| Replication | Scale reads and improve availability |

---

## 3. Partitioning (Sharding)

Partitioning splits data horizontally across multiple databases.

### Example:
Instead of one large table:
```
orders
```

Use:
```
orders_0
orders_1
orders_2
orders_3
```

Each shard holds a subset of data.

---

## 4. Shard Key Selection

Good shard keys:
- user_id
- order_id
- product_id

Shard formula:
```
shard_id = hash(key) % N
```

---

## 5. Applying Partitioning

### Orders Service
- QPS: 17,000
- Target per shard: 5,000 QPS

```
17,000 / 5,000 ≈ 4 shards
```

### Inventory Service
- Shard by product_id
- 4 shards

### Cart Service
- Shard by user_id
- 13,600 / 4 ≈ 3,400 QPS per shard

---

## 6. Replication Strategy

Each shard has replicas:

```
Primary (writes)
Replica 1 (reads)
Replica 2 (reads)
```

---

## 7. Read and Write Flow

### Write Flow
```
App → Primary → WAL → Replicas
```

### Read Flow
```
App → Replica
```

---

## 8. Failure Handling

| Failure | Handling |
|--------|----------|
| Replica down | Traffic rerouted |
| Primary down | Replica promoted |
| Network issue | Retry & fallback |

---

## 9. Why This Works

| Problem | Solution |
|--------|----------|
| High writes | Sharding |
| High reads | Replication |
| Large data | Horizontal scaling |
| Fault tolerance | Replica failover |

---

## 10. Summary

This design supports:
- High throughput
- Horizontal scaling
- Fault tolerance
- Predictable performance

---

## 11. Interview Summary

> We shard data to distribute writes and use replicas for read scalability. Each shard operates independently, enabling linear scaling and resilience.
