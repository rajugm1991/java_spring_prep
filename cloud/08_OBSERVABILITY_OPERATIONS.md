# Observability & Operations (AWS) — Staff Level

## 1) What staff interviews test here

- You design **how you’ll know it’s broken** and **how you’ll restore service**.
- You can separate **symptoms** (latency) from **causes** (DB saturation, thread pool exhaustion).
- You know what you page for vs what you ticket for.

## 2) The “three pillars” and AWS mapping

- **Metrics**: CloudWatch Metrics (service metrics + custom)
- **Logs**: CloudWatch Logs (or centralized logging)
- **Traces**: X-Ray / OpenTelemetry (vendor choice varies)

Plus:
- **Audit**: CloudTrail
- **Config drift**: AWS Config

## 3) Golden signals (baseline dashboard)

For each API/service:
- **Latency** (p50/p95/p99)
- **Traffic** (RPS, QPS)
- **Errors** (rate, types)
- **Saturation** (CPU/memory, DB connections, queue depth/age)

## 4) Alarming philosophy (staff-grade)

- Alert on **user-visible symptoms** aligned to SLOs.
- Use multi-window multi-burn alerts for error budgets (fast burn vs slow burn).
- Avoid alert storms: group, dedupe, add runbook links, tune thresholds.

## 5) Incident response essentials

- **Ownership**: on-call rotations, escalation policy, incident commander.
- **Runbooks**: “what to check first” with links to dashboards and known mitigations.
- **Postmortems**: blameless, with follow-ups and verification.
- **Game days**: practice failures before they happen.

## 6) Operational patterns worth mentioning

- **Circuit breakers / timeouts / retries**: protect dependencies.
- **Bulkheads**: isolate resource pools to prevent cascading failures.
- **Rate limiting**: WAF/API Gateway/LB/app-level; protect critical systems.
- **Feature flags**: fast mitigation for risky features.

## 7) AWS-specific tools to name (don’t over-index, just be fluent)

- CloudWatch dashboards + alarms
- CloudWatch Logs Insights
- X-Ray service map / traces
- CloudTrail for audit events
- AWS Config rules and conformance packs
- (Optional) Managed APM tooling or OpenTelemetry collector


