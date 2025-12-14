# Compute on AWS — EC2 vs ECS/EKS vs Lambda (Staff Tradeoffs)

## 1) Start from workload shape (how staff answers)

Ask first:
- **Traffic**: steady vs spiky? batch vs real-time?
- **Latency**: p50/p95 targets? cold start tolerance?
- **Isolation**: multi-tenant risk? noisy neighbors? compliance?
- **Ops model**: who runs the platform? do you have SRE/platform team?
- **Deployment cadence**: many services? frequent releases?

Then pick the simplest compute that satisfies constraints.

## 2) EC2 + Auto Scaling Group (ASG)

### Best when
- You need full control (custom networking, legacy stacks, special agents).
- You want predictable performance and can own AMI/patching.

### Costs/risks you should mention
- Patch management, AMI pipelines, hardening, instance lifecycle.
- Capacity planning and bin-packing is on you (unless you add more tooling).

## 3) Containers: ECS vs EKS

### ECS (with Fargate or EC2)
- **Easiest** managed container runtime on AWS.
- Great default when you want containers without operating Kubernetes.

### EKS (Kubernetes)
- Best when you need K8s ecosystem: operators, portability, multi-cloud constraints, advanced scheduling.
- You must discuss the **tax**: cluster ops, CNI/IP planning, upgrades, security posture, platform ownership.

### Interview-grade decision summary
- **Few teams / fast delivery**: ECS (often Fargate) is a strong default.
- **Strong platform team + ecosystem needs**: EKS can be justified.

## 4) Serverless: Lambda

### Great for
- Event-driven workloads, glue code, bursty traffic, cron, async processing.
- Clear scaling model and less infra management.

### Key constraints
- Cold starts (esp. in VPC), execution time limits, concurrency control.
- Observability and local testing complexity (manageable, but needs discipline).

### Staff patterns
- Use Lambda behind **API Gateway** or **ALB** for simple APIs.
- Use Lambda + **SQS/EventBridge** for async pipelines (with idempotency + DLQs).
- Use **reserved concurrency** to protect downstream systems.

## 5) API front doors

- **API Gateway**:
  - Great for serverless APIs, auth integration, throttling, request validation.
  - Cost model: per request + data transfer; can get expensive at huge scale.
- **ALB**:
  - Great for container/EC2 apps; can also target Lambda.

## 6) Deployment & release strategies (mention these)

- **Blue/Green** and **Canary** (weighted routing) to reduce risk.
- **Feature flags** for safe incremental rollout.
- **Backwards-compatible** DB migrations (expand/contract).
- Strong rollback plan (including schema rollback constraints).

## 7) Caching patterns with compute

- Put **session state** in Redis/DynamoDB (not in-process).
- Cache at edge (CloudFront) for static/content.
- Use application caching responsibly (cache stampede control).

## 8) Interview prompts to practice

1. Pick compute for: “global API, spiky traffic, 99.9% SLO, small team.”
2. Migrate from EC2 monolith → containers without downtime.
3. Design a worker system that can process 1M jobs/day with retries and ordering constraints.
4. Explain how you’d avoid overload when downstream DB is slow.


