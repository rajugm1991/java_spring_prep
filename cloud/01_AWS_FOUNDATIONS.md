# AWS Foundations for Staff Interviews

## Mental models interviewers expect

- **Shared Responsibility Model**: AWS secures the cloud; you secure what you run *in* the cloud (identity, network boundaries, patching responsibilities vary by service).
- **Blast radius**: isolate failures via **multi-AZ**, **multi-account**, and **cell-based** designs.
- **Design for change**: assume requirements will evolve; optimize for reversibility and safe migrations.
- **Well-Architected**: Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimization, Sustainability.

## Regions, Availability Zones, and failure domains

- **Region**: geographic cluster of AZs. Regional failures are rare but real.
- **AZ**: independent-ish datacenter groups with low-latency links. AZ failures happen more often.
- **Multi-AZ default** for production: load balancers, databases (RDS Multi-AZ/Aurora), and stateless compute.
- **Multi-region** is for explicit needs: geo latency, regulatory, business continuity, extreme availability.

### Interview-ready framing
- Ask: “What is the **availability requirement** and what **failure domain** must we survive (instance, AZ, region)?”
- Convert to: **SLO + RTO/RPO** (see `09_RESILIENCE_DR.md`).

## Accounts, Organizations, and environments

### Recommended baseline for real orgs
- **AWS Organizations** with multiple accounts:
  - **Security** (GuardDuty/SecurityHub, detective controls)
  - **Log archive** (immutable centralized logs)
  - **Shared services / networking** (transit, DNS if needed)
  - **Workload accounts** (prod/stage/dev separated)
- **Why it matters**: permission boundaries, billing, quotas, and incident isolation.

### Good defaults to state in interviews
- Separate **prod from non-prod** (hard boundary).
- Prefer **many small accounts** to “one big account with tags” for strong blast-radius control.

## IAM and identity at a glance (preview)

- Prefer **roles** over long-lived users/keys.
- Use **STS** and short-lived credentials.
- Centralize identity using **IAM Identity Center (SSO)**.
- Use **ABAC** (tags) when scaling permissions.

See `02_IDENTITY_SECURITY.md` for full detail.

## Core AWS building blocks (interview vocabulary)

- **Network edge**: Route 53, CloudFront, WAF, Shield, ALB/NLB
- **Compute**: EC2/ASG, ECS/EKS, Lambda
- **Data**: S3, EBS/EFS; RDS/Aurora, DynamoDB, ElastiCache, OpenSearch
- **Integration**: SQS, SNS, EventBridge, Kinesis, Step Functions
- **Observability/audit**: CloudWatch, X-Ray, CloudTrail, Config
- **Security**: IAM, KMS, Secrets Manager, SSM Parameter Store, GuardDuty, Security Hub
- **IaC**: CloudFormation, CDK, Terraform (org-specific)

## Quick decision heuristics (fast recall)

- **Start with managed**: RDS/Aurora over self-managed DB on EC2; ALB over Nginx fleet; MSK/Kinesis over self-managed Kafka (unless strong reasons).
- **Prefer stateless** compute: session state in Redis/DynamoDB, files in S3.
- **Use async** when it improves resilience: queue between spikes and workers.
- **Design for idempotency** when using at-least-once messaging.

## Common staff-level pitfalls (what to avoid saying)

- “We’ll do multi-region active-active by default” (expensive and complex; justify it).
- “We’ll just use Kubernetes” (state why, what you gain, what you pay).
- “Security later” (at staff, security is part of the design).
- “We’ll monitor it” without specifying **SLIs**, alarms, dashboards, and incident response.

## Minimal AWS reference: request path (typical web app)

1. Client → **Route 53** (DNS)
2. Optional → **CloudFront** + **WAF**
3. → **ALB**
4. → **ECS/EKS/EC2/Lambda**
5. → **RDS/Aurora/DynamoDB** + **ElastiCache**
6. Logs/metrics/traces → **CloudWatch/X-Ray**
7. Audit → **CloudTrail**, config drift → **AWS Config**


