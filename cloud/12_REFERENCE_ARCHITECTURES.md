# AWS Reference Architectures (Interview-Ready)

Use these as “defaults” you can adapt quickly.

## 1) Standard web app (3-tier) — scalable + secure

**Edge**
- Route 53 → CloudFront (optional) → WAF → ALB

**App**
- ECS (Fargate) or EKS or EC2/ASG in private subnets
- Service-to-service auth via IAM roles (where applicable) + app tokens

**Data**
- Aurora/RDS Multi-AZ (transactions)
- ElastiCache Redis (cache/session/rate limit)
- S3 (uploads, exports)

**Async**
- SQS for buffering background jobs
- EventBridge for domain events (optional)

**Ops**
- CloudWatch dashboards/alarms
- X-Ray/OTel traces
- CloudTrail + Config

## 2) Event-driven order processing (common e-commerce prompt)

- API → write order → outbox → publish `OrderCreated`
- Consumers:
  - inventory reservation
  - payment authorization
  - shipping orchestration
- Use Step Functions for orchestration if flows are complex.
- Idempotency keys and DLQs everywhere.

## 3) Data ingestion + analytics-lite

- Ingest → Kinesis (or SQS) → processing (Lambda/ECS) → S3 data lake
- Query via Athena/Glue (if needed) and materialize aggregates to DynamoDB/ElastiCache.

## 4) Decision tables (quick)

### Compute
- **Need quickest ops + spiky traffic**: Lambda
- **Need containers without K8s**: ECS
- **Need K8s ecosystem**: EKS
- **Need full control/legacy**: EC2/ASG

### Integration
- **Buffer + decouple**: SQS
- **Fan-out**: SNS
- **Event routing**: EventBridge
- **Ordered streaming + replay**: Kinesis

### Data
- **Relational/transactions**: RDS/Aurora
- **Massive scale by key**: DynamoDB
- **Caching/session**: Redis
- **Files/durable blobs**: S3


