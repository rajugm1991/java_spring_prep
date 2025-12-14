# Designing a Rate Limiter (Distributed, Production-Grade)

Rate limiting protects your system from **abuse**, **traffic spikes**, and **cascading failures**. In staff interviews, it’s a classic “deep dive” topic because it involves **distributed systems, correctness, fairness, and operational tradeoffs**.

---

## 1) Requirements (clarify first in interviews)

### Functional requirements
- Limit requests by **identity** (API key / user / tenant / IP / token).
- Support multiple rules (e.g., `read: 1000/min`, `write: 60/min`).
- Return clear responses and limit state (headers).
- Allow bursts (optional), and different tiers/plans (free/pro/enterprise).

### Non-functional requirements
- Low latency (ideally single-digit ms overhead).
- High availability (rate limiter should not be a global single point of failure).
- Consistent behavior under load (no “retry storms”).
- Observability: metrics, dashboards, alerts, logging.

### Key clarifying questions
- What is the limiter scope? **Edge** (API gateway) vs **service-level** vs both.
- Do we need **per-second** precision or “good enough” per minute?
- Is fairness required across instances? (distributed coordination)
- What’s the failure behavior: **fail-open** (allow) or **fail-closed** (block)?

---

## 2) Where to enforce rate limits (common architectures)

### Option A: Edge / API Gateway rate limiting
**Best** for user-facing APIs; reduces load on downstream services.

```
Client → WAF / API Gateway / LB → Services
            ↑
       Rate Limiter
```

### Option B: Service-level rate limiting
Useful for internal APIs (service-to-service) and protecting dependencies.

### Option C: Multi-layer (recommended at scale)
- **Edge** protects the system boundary.
- **Per-service** protects internal dependencies and prevents noisy neighbors.

---

## 3) Core algorithms (know these + tradeoffs)

### 3.1 Fixed window counter
- Count requests in a fixed time bucket (e.g., per minute).
- **Pros**: simplest.
- **Cons**: boundary spikes (burst at window edges).

### 3.2 Sliding window log
- Track timestamps of requests, count those in last N seconds.
- **Pros**: accurate.
- **Cons**: memory heavy at high QPS; expensive.

### 3.3 Sliding window counter (rolling)
- Approximate sliding window using two buckets with weights.
- **Pros**: good accuracy, cheaper than log.
- **Cons**: approximation; careful math.

### 3.4 Token bucket (common production default)
- Bucket refills at rate `r`; each request consumes one token.
- **Pros**: allows bursts, smooths traffic.
- **Cons**: distributed coordination is tricky; needs atomic updates.

### 3.5 Leaky bucket
- Requests enter a queue drained at constant rate.
- **Pros**: smooth output rate.
- **Cons**: adds queueing latency; can drop on overflow.

**Interview guidance:** if you’re unsure, choose **token bucket** or **sliding window counter**, then explain the tradeoff.

---

## 4) Distributed rate limiting with Redis (practical approach)

### Why Redis?
- Central atomic updates with low latency.
- Supports Lua scripts for **atomic read-modify-write**.
- Can be clustered for HA/scale.

### Key design decisions
- **Key format**: `rl:{tenantId}:{userId}:{route}:{windowStart}`
- **Shard key**: include stable prefix to avoid hot shard.
- **TTL**: set TTL slightly above window duration to auto-clean.
- **Fail behavior**:
  - **Fail-open**: allow if Redis is down (availability bias).
  - **Fail-closed**: block if Redis is down (security/abuse bias).

### Token bucket with Redis (high-level pseudocode)

```
Inputs: key, capacity, refillRatePerSec, now
Stored: tokens, lastRefillTs

newTokens = min(capacity, tokens + (now-lastRefillTs)*refillRatePerSec)
if newTokens < 1:
  reject
else:
  tokens = newTokens - 1
  lastRefillTs = now
  allow
```

### Production concerns
- Hot keys for shared identities (e.g., a big tenant): mitigate with **per-tenant limits** + **per-user limits** + burst controls.
- Redis cluster sizing, replication, and monitoring are part of the design.

---

## 5) API behavior: responses and headers (good client UX)

If rate limited:
- Status: `429 Too Many Requests`
- Headers:
  - `Retry-After: <seconds>`
  - `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` (or a documented scheme)

Return a stable error body:
```json
{
  "code": "RATE_LIMITED",
  "message": "Too many requests",
  "retryable": true
}
```

---

## 6) Advanced topics interviewers love

### 6.1 Hierarchical limits (multi-tenant)
- Per-tenant limit (protect global capacity)
- Per-user limit (fairness)
- Per-route limits (protect expensive endpoints)

### 6.2 Different costs per request (weighted rate limiting)
- e.g., `POST /checkout` costs 10 tokens, `GET /products` costs 1.

### 6.3 Burst handling + fairness
- Token bucket naturally allows bursts; tune capacity separately from refill rate.

### 6.4 Global vs local limiting
- Local (per-instance) is fast but inconsistent.
- Global (shared store) is consistent but adds dependency.
- Hybrid: local “soft limit” + global “hard limit” for better resilience.

---

## 7) Reliability & operational excellence

### Metrics (minimum)
- Allowed vs throttled count (by route, tenant, user)
- Redis latency / errors
- P95 limiter overhead
- Key cardinality and hot-key detection

### Alarms (examples)
- Throttles spike for a critical route
- Redis error rate > threshold
- Limiter latency > threshold

### Runbook essentials
- “If throttling spikes, how to identify top offenders?”
- “How to temporarily increase limits for a tenant?”
- “How to fail-open safely during Redis incident?”

---

## 8) Common pitfalls

- Using fixed windows without addressing edge bursts.
- Making the limiter a single global SPOF (no redundancy/cluster plan).
- No documented limit headers → poor client behavior.
- No idempotency strategy on clients (rate limiting + retries can cause duplicates elsewhere).

---

## 9) Relationship to other guides

- For deeper implementation examples and patterns, see:
  - `docs/imp/RATE_LIMITING_COMPLETE_GUIDE.md`


