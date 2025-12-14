# AWS System Design Prompts (Staff-Level) + Rubric

Use these prompts to practice end-to-end framing, not just service selection.

## Rubric (how you’re evaluated)

- **Problem framing**: requirements, constraints, assumptions.
- **Architecture**: components and data flow make sense and are implementable.
- **Tradeoffs**: alternatives + why rejected.
- **Reliability**: SLOs, failure modes, mitigation, DR posture.
- **Security**: identity model, data protection, auditability.
- **Operations**: observability, on-call, runbooks, rollout, incident response.
- **Cost**: major cost drivers + levers.

## Prompts

### 1) Global login + session management
- Multi-tenant SaaS auth system with 99.95% availability.
- Requirements: low latency globally, secure session management, revocation, auditability.

### 2) Checkout + payments (idempotency stress test)
- Design checkout that never double-charges and can survive retries/timeouts.
- Include fraud signals, audit logs, and rollback/compensation strategy.

### 3) Inventory reservation at scale
- 10k orders/min bursts; must prevent oversell.
- Discuss consistency model and tradeoffs.

### 4) Seller catalog ingestion pipeline
- Sellers upload CSV/JSON + images; system validates, enriches, and publishes.
- Include DLQs, retries, quarantine, and backpressure.

### 5) Event-driven microservices with guaranteed processing
- At-least-once message delivery; how do you design business-level exactly-once?

### 6) Multi-account platform for 50 teams
- Provide networking, CI/CD, logging, guardrails, and self-service.
- Show SCP strategy and baseline controls.

### 7) DR plan for the whole platform
- Define RTO/RPO per subsystem; propose a DR strategy and test plan.

## Follow-up questions you should practice answering

- “What happens during an AZ outage? What breaks first?”
- “How do you roll out schema changes with zero downtime?”
- “How do you debug a latency regression?”
- “How do you reduce NAT costs without weakening security?”
- “How do you prevent an internal service from data exfiltration?”


