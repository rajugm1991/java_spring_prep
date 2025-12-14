# Resilience & Disaster Recovery (AWS) — Staff Playbook

## 1) Start with SLOs and recovery objectives

- **SLO**: target reliability (e.g., 99.9% monthly availability).
- **SLIs**: what you measure (request success rate, latency).
- **Error budget**: how much unreliability you can “spend”.
- **RTO**: how long to restore service.
- **RPO**: how much data you can lose.

Staff move: tie architecture choices to these numbers.

## 2) Failure domains and design targets

- **Instance failure**: use stateless compute + auto scaling.
- **AZ failure**: multi-AZ (LB + compute across AZs + Multi-AZ DB).
- **Region failure**: multi-region DR patterns (only if required).

## 3) Patterns for high availability

- **Load balancers across AZs** + health checks.
- **Multi-AZ databases** (RDS Multi-AZ / Aurora).
- **Graceful degradation**: shed non-critical features under duress.
- **Backpressure**: queues and concurrency limits.
- **Timeouts + retries**: but with jitter and caps; avoid retry storms.

## 4) DR strategies (in increasing cost/complexity)

1. **Backup & restore**
   - Cheapest; longer RTO/RPO.
2. **Pilot light**
   - Minimal always-on core; scale up during disaster.
3. **Warm standby**
   - Scaled-down replica running; faster recovery.
4. **Active-active**
   - Highest availability and complexity; needs data strategy and traffic management.

## 5) Multi-region decision rubric (staff-level)

Do multi-region only if you have:
- Regulatory/data residency needs
- Very low latency to multiple geos
- Business continuity requirements beyond what multi-AZ provides

Then pick:
- Active-passive (simpler) vs active-active (complex)
- Data replication approach (read replicas, global tables, async replication)

## 6) Testing reliability (what most teams skip)

- **Load testing**: validate saturation points and scaling behavior.
- **Chaos**: controlled fault injection (kill instances, blackhole dependencies).
- **Game days**: rehearsed incident simulations with runbooks.
- **Restore drills**: validate backups and RTO/RPO in practice.

## 7) What to say in interviews during “outage” follow-ups

- How you detect (dashboards/alarms, customer signals)
- How you mitigate quickly (feature flags, throttling, rollback)
- How you stabilize (capacity, dependency isolation)
- How you prevent recurrence (postmortem actions, tests, guardrails)


