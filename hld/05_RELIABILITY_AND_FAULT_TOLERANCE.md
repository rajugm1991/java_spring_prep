# Reliability & Fault Tolerance

## Table of Contents
1. [Introduction](#introduction)
2. [Redundancy Patterns](#redundancy-patterns)
3. [Failover Strategies](#failover-strategies)
4. [Circuit Breaker Pattern](#circuit-breaker-pattern)
5. [Health Checks](#health-checks)
6. [Retry Mechanisms](#retry-mechanisms)
7. [Real-World Examples](#real-world-examples)

---

## Introduction

**Reliability** is the probability that a system will work correctly over time. **Fault Tolerance** is the ability of a system to continue operating despite failures.

### Key Metrics

- **MTBF (Mean Time Between Failures):** Average time between system failures
- **MTTR (Mean Time To Repair):** Average time to fix a failure
- **Availability = MTBF / (MTBF + MTTR)**

**Example:**
```
MTBF = 720 hours (30 days)
MTTR = 1 hour
Availability = 720 / (720 + 1) = 99.86%
```

---

## Redundancy Patterns

### 1. **Active-Active (Load Sharing)**

All servers handle traffic simultaneously.

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│ Server 1  │  │ Server 2  │  │ Server 3 │
│ (Active)  │  │ (Active)  │  │ (Active) │
└──────────┘  └──────────┘  └──────────┘
     │             │             │
     └─────────────┼─────────────┘
                   │
              ┌────┴────┐
              │  Load   │
              │Balancer │
              └─────────┘
```

**Pros:**
- Maximum resource utilization
- No idle resources
- Automatic load distribution

**Cons:**
- More complex to manage
- Requires load balancing

**Use Case:** Web servers, API servers

---

### 2. **Active-Passive (Hot Standby)**

One server active, others on standby.

```
┌──────────┐ (Active)
│ Server 1 │  ← Handling all traffic
└──────────┘
┌──────────┐ (Standby)
│ Server 2 │  ← Monitoring, ready to take over
└──────────┘
```

**Pros:**
- Simple failover
- Predictable performance

**Cons:**
- Wasted resources (standby servers idle)
- Failover time (seconds to minutes)

**Use Case:** Database master-slave, critical systems

---

### 3. **N+1 Redundancy**

N servers handle load, 1 extra for failover.

```
Normal: 3 servers handle load
Backup: 1 server ready
If one fails: Backup takes over
```

**Example:**
```
4 servers total
3 active (handle 100% load)
1 standby (ready for failover)
If any active fails, standby activates
```

---

## Failover Strategies

### 1. **Automatic Failover**

System automatically switches to backup.

```
Primary fails:
1. Health check detects failure
2. Load balancer stops routing to primary
3. Traffic routed to secondary
4. Secondary becomes primary
```

**Example: Database Failover**

```
┌──────────┐
│  Master  │  ← Primary (handling writes)
└────┬─────┘
     │ Replication
     ▼
┌──────────┐
│  Slave   │  ← Standby (ready)

Master fails:
┌──────────┐ (Down)
│  Master  │
└──────────┘
┌──────────┐ (Promoted)
│  Slave   │  ← Now primary
└──────────┘
```

---

### 2. **Manual Failover**

Admin manually switches to backup.

**Use Case:** Planned maintenance, testing

---

### 3. **Geographic Failover**

Failover to different data center.

```
Primary: US East
Backup:  US West

If US East fails → Traffic routes to US West
```

---

## Circuit Breaker Pattern

**Circuit Breaker** prevents cascading failures by stopping calls to failing services.

### States

```
CLOSED (Normal):
Service A → Service B → Response ✅
(Circuit allows traffic)

OPEN (Failing):
Service A → [Circuit Open] → Fallback ✅
(Circuit blocks traffic, returns fallback)

HALF-OPEN (Testing):
Service A → Service B (test call)
If success → CLOSED
If failure → OPEN
```

### Implementation

```java
@CircuitBreaker(name = "paymentService", fallbackMethod = "fallback")
public PaymentResult processPayment(PaymentRequest request) {
    return paymentService.process(request);
}

public PaymentResult fallback(PaymentRequest request, Exception e) {
    // Return cached result or default
    logger.warn("Payment service unavailable, using fallback");
    return PaymentResult.pending("Payment will be processed later");
}
```

### Configuration

```yaml
resilience4j:
  circuitbreaker:
    instances:
      paymentService:
        failureRateThreshold: 50  # Open if 50% requests fail
        waitDurationInOpenState: 10s  # Wait 10s before testing
        slidingWindowSize: 10  # Last 10 requests
        minimumNumberOfCalls: 5  # Need 5 calls to evaluate
```

---

## Health Checks

### Types of Health Checks

#### 1. **Liveness Probe**

Is the service alive?

```
GET /health/live

Response:
{
  "status": "alive"
}
```

**Use Case:** Kubernetes restarts container if dead

---

#### 2. **Readiness Probe**

Is the service ready to handle requests?

```
GET /health/ready

Response:
{
  "status": "ready",
  "database": "connected",
  "cache": "connected"
}
```

**Use Case:** Load balancer routes traffic only if ready

---

#### 3. **Startup Probe**

Has the service finished starting?

```
GET /health/startup

Response:
{
  "status": "started"
}
```

**Use Case:** Kubernetes waits before marking ready

---

### Health Check Implementation

```java
@RestController
public class HealthController {
    
    @Autowired
    private DataSource dataSource;
    
    @Autowired
    private RedisTemplate<String, String> redisTemplate;
    
    @GetMapping("/health")
    public ResponseEntity<Map<String, Object>> health() {
        Map<String, Object> health = new HashMap<>();
        
        // Check database
        try {
            dataSource.getConnection().close();
            health.put("database", "connected");
        } catch (Exception e) {
            health.put("database", "disconnected");
            return ResponseEntity.status(503).body(health);
        }
        
        // Check cache
        try {
            redisTemplate.getConnectionFactory().getConnection().ping();
            health.put("cache", "connected");
        } catch (Exception e) {
            health.put("cache", "disconnected");
        }
        
        health.put("status", "healthy");
        return ResponseEntity.ok(health);
    }
}
```

---

## Retry Mechanisms

### Retry Strategies

#### 1. **Simple Retry**

Retry fixed number of times.

```java
@Retry(name = "paymentService")
public PaymentResult processPayment(PaymentRequest request) {
    return paymentService.process(request);
}
```

**Configuration:**
```yaml
resilience4j:
  retry:
    instances:
      paymentService:
        maxAttempts: 3
        waitDuration: 1000  # Wait 1 second between retries
```

---

#### 2. **Exponential Backoff**

Wait time increases exponentially.

```
Attempt 1: Wait 1s
Attempt 2: Wait 2s
Attempt 3: Wait 4s
Attempt 4: Wait 8s
```

**Configuration:**
```yaml
resilience4j:
  retry:
    instances:
      paymentService:
        maxAttempts: 5
        waitDuration: 1000
        enableExponentialBackoff: true
        exponentialBackoffMultiplier: 2
```

---

#### 3. **Jitter**

Add randomness to prevent thundering herd.

```
Without Jitter:
All clients retry at same time → Overload server

With Jitter:
Clients retry at random times → Spread load
```

---

## Real-World Examples

### Example 1: Netflix Chaos Engineering

**Netflix's Approach:**
- Intentionally break things in production
- Test system resilience
- Improve fault tolerance

**Chaos Monkey:**
- Randomly terminates instances
- Tests auto-recovery
- Ensures system can handle failures

**Benefits:**
- Found 1000+ bugs before customers
- Improved system reliability
- Better fault tolerance

---

### Example 2: AWS Multi-AZ Deployment

```
Availability Zone 1 (US East):
┌──────────┐
│  Server  │
└──────────┘

Availability Zone 2 (US East):
┌──────────┐
│  Server  │  ← Backup
└──────────┘

If AZ1 fails → Traffic routes to AZ2
```

---

### Example 3: Database Replication

```
Master-Slave Replication:

┌──────────┐
│  Master  │  ← Handles writes
└────┬─────┘
     │ Replication (async)
     │
┌────┼────┬────┐
│    │    │    │
▼    ▼    ▼    ▼
┌──┐ ┌──┐ ┌──┐ ┌──┐
│S1│ │S2│ │S3│ │S4│  ← Handle reads
└──┘ └──┘ └──┘ └──┘

If master fails:
- One slave promoted to master
- Other slaves replicate from new master
- Minimal downtime
```

---

## Summary

**Key Patterns:**
1. ✅ **Redundancy:** Active-Active, Active-Passive, N+1
2. ✅ **Failover:** Automatic, Manual, Geographic
3. ✅ **Circuit Breaker:** Prevent cascading failures
4. ✅ **Health Checks:** Liveness, Readiness, Startup
5. ✅ **Retry:** Simple, Exponential Backoff, Jitter

**Next Steps:**
- Learn [Performance Optimization](./06_PERFORMANCE_OPTIMIZATION.md)
- Understand [Scalability Patterns](./04_SCALABILITY_PATTERNS.md)
- Practice [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [Netflix Chaos Engineering](https://netflix.github.io/chaosmonkey/)
- [Resilience4j Documentation](https://resilience4j.readme.io/)
