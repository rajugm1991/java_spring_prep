# Cloud (AWS) — Staff Engineer Interview Guide

This folder is a **staff-level AWS cloud guide** focused on **architecture, reliability, security, operations, and cost tradeoffs**—the stuff interview loops probe for when you have 10+ years of experience.

## How to use this guide

- **If you’re short on time**: start with `00_STAFF_STUDY_PLAN.md`, then skim `12_REFERENCE_ARCHITECTURES.md` + `13_SYSTEM_DESIGN_QUESTIONS_AWS.md`.
- **If you want depth**: go in order from `01_...` onward.
- **If you’re interviewing for platform / infra**: prioritize `02`, `03`, `08`, `09`, `11`.

## Contents

1. [`00_STAFF_STUDY_PLAN.md`](./00_STAFF_STUDY_PLAN.md) — 4-week plan, what “good” looks like in staff interviews
2. [`01_AWS_FOUNDATIONS.md`](./01_AWS_FOUNDATIONS.md) — regions/AZs, shared responsibility, Well-Architected, account strategy
3. [`02_IDENTITY_SECURITY.md`](./02_IDENTITY_SECURITY.md) — IAM, STS, KMS, secrets, GuardDuty/SecurityHub, threat modeling
4. [`03_NETWORKING_VPC.md`](./03_NETWORKING_VPC.md) — VPC design, routing, endpoints, NAT, hybrid, LB, DNS
5. [`04_COMPUTE_CONTAINERS_SERVERLESS.md`](./04_COMPUTE_CONTAINERS_SERVERLESS.md) — EC2/ASG, ECS/EKS, Lambda, API Gateway
6. [`05_STORAGE_DATA_MGMT.md`](./05_STORAGE_DATA_MGMT.md) — S3/EBS/EFS, lifecycle, backup/restore, data protection
7. [`06_DATABASES.md`](./06_DATABASES.md) — RDS/Aurora, DynamoDB, caching, consistency, migrations
8. [`07_INTEGRATION_MESSAGING.md`](./07_INTEGRATION_MESSAGING.md) — SQS/SNS/EventBridge/Kinesis/Step Functions, idempotency
9. [`08_OBSERVABILITY_OPERATIONS.md`](./08_OBSERVABILITY_OPERATIONS.md) — CloudWatch/X-Ray, logs/metrics/traces, incident response
10. [`09_RESILIENCE_DR.md`](./09_RESILIENCE_DR.md) — SLOs, multi-AZ/region, DR patterns, chaos, game days
11. [`10_COST_FINOPS.md`](./10_COST_FINOPS.md) — cost models, tagging, budgets, RIs/Savings Plans, unit economics
12. [`11_PLATFORM_LANDING_ZONE.md`](./11_PLATFORM_LANDING_ZONE.md) — orgs/accounts, SCPs, baseline guardrails, IaC, paved roads
13. [`12_REFERENCE_ARCHITECTURES.md`](./12_REFERENCE_ARCHITECTURES.md) — common architectures and decision tables
14. [`13_SYSTEM_DESIGN_QUESTIONS_AWS.md`](./13_SYSTEM_DESIGN_QUESTIONS_AWS.md) — staff-style prompts + evaluation rubric
15. [`14_CHEATSHEETS.md`](./14_CHEATSHEETS.md) — quick recall pages (VPC, IAM, DR, queues, caching, ops)

## Suggested practice loop (interview-calibrated)

1. Pick one prompt from `13_SYSTEM_DESIGN_QUESTIONS_AWS.md`
2. Write: **requirements → constraints → risks → architecture → tradeoffs → rollout plan**
3. Add: **SLOs/SLIs**, **failure modes**, **security controls**, **cost model**
4. End with: “**How I’d verify it works**” (dashboards, alarms, load tests, game day)


