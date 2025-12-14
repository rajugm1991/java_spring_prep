# Staff Engineer (AWS Cloud) — 4-Week Study Plan + Interview Expectations

This plan assumes **11 years experience** and aims at **Staff/Principal signals**: clear problem framing, crisp tradeoffs, operational excellence, and a strong security/cost posture.

## What “good” looks like at Staff

- **Architecture is a means, not a goal**: you lead with requirements, constraints, and risks—not services.
- **Tradeoffs are explicit**: latency vs cost, consistency vs availability, build vs buy, dev velocity vs blast radius.
- **Operations are designed-in**: SLOs, alerting, runbooks, rollouts, incident response, DR, and capacity planning.
- **Security is systematic**: identity boundaries, least privilege, secret management, encryption, auditability, threat modeling.
- **Multi-account + governance**: you know how to keep many teams safe without blocking delivery.

## Core deliverable you should practice

For any system design prompt, be able to produce:

1. **Requirements**: functional + non-functional (latency, throughput, data retention, compliance).
2. **Constraints**: team size, skills, time, existing tech, regions, budget.
3. **Failure modes**: what breaks, how you detect, how you recover.
4. **Architecture**: components, data flow, network boundaries, dependencies.
5. **Tradeoffs**: alternatives considered and why rejected.
6. **Operations**: dashboards/alarms, on-call, playbooks, chaos/game days.
7. **Security**: IAM model, KMS, secrets, logging/auditing, threat model.
8. **Cost**: major cost drivers, estimates, guardrails, optimization levers.
9. **Rollout plan**: milestones, migration plan, backward compatibility, rollback.

## 4-week plan (90 minutes/day + weekend deep dive)

### Week 1 — Foundations + Networking baseline
- **Read**: `01_AWS_FOUNDATIONS.md`, `03_NETWORKING_VPC.md`
- **Outcomes**
  - Explain regions/AZs, multi-AZ design, multi-account strategy, shared responsibility.
  - Whiteboard a VPC: subnets, routing, NAT vs endpoints, ALB/NLB, DNS.
  - Know when to use private connectivity (VPC endpoints, PrivateLink, DX/VPN).
- **Drills**
  - “Design a private service-to-service network with internet egress controls.”
  - “Explain what happens on a request from client → Route53 → ALB → ECS → RDS.”

### Week 2 — Security + identity + data protection
- **Read**: `02_IDENTITY_SECURITY.md`, `05_STORAGE_DATA_MGMT.md`, `06_DATABASES.md`
- **Outcomes**
  - IAM: roles, trust policies, permission boundaries, ABAC, STS, session management.
  - Data: encryption at rest/in transit, key management, backups, retention, auditability.
  - Database tradeoffs: Aurora vs DynamoDB vs cache.
- **Drills**
  - “Design least-privilege IAM for 30 microservices across multiple accounts.”
  - “How do you rotate secrets without downtime?”

### Week 3 — Compute, integration, observability
- **Read**: `04_COMPUTE_CONTAINERS_SERVERLESS.md`, `07_INTEGRATION_MESSAGING.md`, `08_OBSERVABILITY_OPERATIONS.md`
- **Outcomes**
  - Pick between EC2/ECS/EKS/Lambda with reasoned tradeoffs.
  - Design idempotent event processing with retries, DLQs, ordering constraints.
  - Build an observability plan: golden signals, tracing, log strategy, alert hygiene.
- **Drills**
  - “At-least-once delivery: how do you avoid double-charging?”
  - “What would you alert on, and what would you page for?”

### Week 4 — Resilience/DR + Cost/FinOps + platform thinking
- **Read**: `09_RESILIENCE_DR.md`, `10_COST_FINOPS.md`, `11_PLATFORM_LANDING_ZONE.md`
- **Outcomes**
  - Turn requirements into SLOs, error budgets, and recovery objectives (RTO/RPO).
  - Build a DR plan: multi-AZ vs multi-region; warm standby vs active-active.
  - Explain cost drivers and guardrails (tagging, budgets, unit cost).
  - Design “paved roads” for teams (golden paths, guardrails, templates).
- **Drills**
  - “Walk through a regional outage and your recovery steps.”
  - “Cut cost by 30% without changing product behavior—what’s your approach?”

## Weekly checkpoint rubric (score yourself)

- **Clarity**: can someone implement from your explanation?
- **Depth**: can you discuss failure modes and mitigation?
- **Pragmatism**: incremental rollout, migrations, and how to measure success.
- **Ownership**: metrics, on-call, incident response, and postmortems.

## Recommended artifacts to prepare (bring to interviews mentally)

- A “**reference architecture**” you can adapt quickly (see `12_REFERENCE_ARCHITECTURES.md`)
- A handful of “**war stories**”: incident, migration, cost reduction, security hardening
- Your “**default operating model**”: SLOs, on-call, dashboards, runbooks, DR testing cadence


