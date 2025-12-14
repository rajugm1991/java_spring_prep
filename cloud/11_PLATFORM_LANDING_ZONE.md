# Platform + Landing Zone (AWS) — Staff Engineer Guide

This is the “how do you scale engineering safely” chapter: multi-account, guardrails, standardization, and self-service.

## 1) The goal of a landing zone

- Enable teams to ship fast with **safe defaults**.
- Reduce blast radius and security risk with **strong boundaries**.
- Provide **central visibility** (logs, security findings, inventory).

## 2) Multi-account structure (reference)

- **Management** account (minimal usage)
- **Log archive** account (centralized CloudTrail/Config logs)
- **Security tooling** account (GuardDuty/Security Hub aggregation)
- **Shared networking** account (optional: TGW, centralized egress, shared DNS)
- **Workload accounts** per environment/team/product

## 3) Guardrails and governance

- **SCPs** (Service Control Policies) to prevent unsafe actions:
  - block disabling CloudTrail/Config
  - restrict regions
  - deny public S3 buckets
- **Baseline IAM**: SSO, break-glass, separation of duties.
- **Standard logging**: CloudTrail org trail + centralized storage.
- **Config conformance packs** for compliance baselines.

## 4) “Paved roads” (what staff platform engineers build)

- Golden path templates:
  - VPC + subnets + endpoints
  - Service template (ECS/EKS/Lambda) with alarms, dashboards, CI/CD, IAM roles
  - Data stores with backup/retention defaults
- Self-service via IaC + pipelines + docs:
  - CDK/Terraform modules
  - policy-as-code checks (lint + security scanning)

## 5) Operating model

- Define ownership boundaries: platform team vs product teams.
- Define SLOs for platform components (CI, cluster, shared services).
- Provide runbooks and on-call for shared components.


