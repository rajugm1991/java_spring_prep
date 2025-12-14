# Identity & Security (AWS) — Staff Level

## 1) Threat-model-first approach (what interviewers want)

Start by naming:
- **Assets**: customer PII, payment tokens, auth sessions, internal IP, logs.
- **Actors**: external attacker, compromised internal principal, supply-chain attack.
- **Entry points**: public APIs, CI/CD, admin consoles, data ingestion.
- **Controls**: identity boundaries, network boundaries, encryption, detection, response.

## 2) IAM: scaling permissions safely

### Core primitives
- **Principal**: user/role/service. Prefer **roles**.
- **Policy**: permissions. Prefer least privilege + explicit denies where needed.
- **Resource policy**: attached to the resource (e.g., S3 bucket policy).
- **Trust policy**: who can assume a role.
- **STS**: issues temporary credentials (short-lived).

### Staff-grade best practices
- **No long-lived access keys** for humans; use **IAM Identity Center (SSO)**.
- **Use roles everywhere** (workloads use instance/task roles / IRSA / Lambda role).
- **Permission boundaries** for teams to self-serve without escalating.
- **ABAC** (tag-based) for large fleets (services, environments, cost centers).
- **Separate duties**: break-glass admin, deployer, operator, auditor.

### “Explain STS” in one minute
- A principal **assumes** a role (via trust policy) and receives **temporary credentials**.
- Temporary creds reduce blast radius (expiration, session policies).
- Cross-account access uses role assumption; no need to share keys.

## 3) Encryption & key management (KMS)

### What you should say at staff level
- **Encrypt in transit**: TLS everywhere; mTLS for internal high-trust boundaries when needed.
- **Encrypt at rest by default**: S3 SSE-KMS, EBS encryption, RDS encryption.
- **Control key access**: KMS key policy + IAM policy + grants.

### Common pitfalls
- Using one KMS key for everything without access boundaries.
- Not planning for **key rotation**, **audit**, and **deletion protection**.
- Logging secrets (tokens, passwords) into CloudWatch logs.

## 4) Secrets management

- **Secrets Manager**: rotation workflows, secret versions.
- **SSM Parameter Store**: good for config; prefer SecureString with KMS for sensitive values.
- Pattern: apps retrieve secrets at startup (or via sidecar/agent), cache with TTL, rotate safely.

## 5) Network security baseline (preview; details in `03_NETWORKING_VPC.md`)

- Default to **private subnets** for workloads; **public only** for LBs / edge.
- Use **security groups** as primary micro-perimeter; **NACLs** for coarse boundaries if needed.
- Use **VPC endpoints** for private access to AWS services (avoid NAT egress where possible).

## 6) Detective controls and auditability

- **CloudTrail**: API audit logs (centralize into log-archive account; make immutable).
- **AWS Config**: configuration history and drift detection.
- **GuardDuty**: threat detection.
- **Security Hub**: aggregate findings.
- **IAM Access Analyzer**: unintended public/cross-account access.

## 7) Edge protection

- **WAF**: OWASP protections, rate limiting at edge, allow/deny lists.
- **Shield**: DDoS protection (Standard vs Advanced depends on exposure/risk).
- **CloudFront**: reduces origin load, improves latency, provides edge controls.

## 8) Secure CI/CD (common staff interview area)

- Treat CI as production: least-privilege deploy roles, short-lived tokens, artifact signing.
- Separate build vs deploy permissions; promote artifacts, don’t rebuild per env.
- Store secrets in managed stores; avoid plaintext in pipelines.
- Use IaC with code review + policy checks (lint + security scanning).

## 9) Answer templates (use these in interviews)

### “How would you secure this system?”
- Identity boundaries (SSO + roles + least privilege + separation of duties)
- Network boundaries (private subnets + SGs + endpoints)
- Data protection (TLS + KMS + backups + retention)
- Detective controls (CloudTrail/Config/GuardDuty)
- Incident response (alerts + runbooks + break-glass)

### “How do you prevent a compromised service from accessing everything?”
- Per-service roles, scoped IAM
- SG boundaries + VPC endpoints + egress restrictions
- Secrets scoped per service
- Audit + anomaly detection


