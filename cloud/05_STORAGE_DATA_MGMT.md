# Storage & Data Management (AWS) — Staff Level

## 1) S3 (object storage) — the default for durable data

### When to use
- Static assets, uploads, exports, backups, data lake, logs, event payload archival.

### Staff-grade patterns
- **Bucket per domain boundary** (or per environment) with tight policies.
- **Block Public Access** ON; avoid public ACLs.
- **SSE-KMS** for sensitive data; enforce via bucket policy.
- **Lifecycle rules** (IA/Glacier) + retention policies.
- **Versioning** for safety; pair with lifecycle to control cost.
- **Eventing**: S3 → EventBridge/SNS/SQS/Lambda (be mindful of retries/duplicates).

### Common pitfalls
- Storing “hot query data” in S3 and then trying to query it like a DB.
- No plan for large-object upload (use multipart, pre-signed URLs).

## 2) EBS (block storage)

- Attached to EC2; good for databases on EC2, stateful workloads, low-latency needs.
- Staff considerations: performance class (gp3/io2), snapshots, encryption, AZ coupling.

## 3) EFS (shared filesystem)

- Network filesystem, multi-AZ, POSIX-like.
- Useful for shared content, legacy apps, simple shared storage.
- Consider latency and cost; don’t use as a high-QPS database.

## 4) Backup and recovery strategy (say this explicitly)

- Define **RPO/RTO** per data class (see `09_RESILIENCE_DR.md`).
- Use managed backups where possible:
  - RDS/Aurora automated backups + snapshots
  - DynamoDB PITR
  - EBS snapshots
- Cross-account + cross-region backup copies for critical systems.
- Test restores (backup without restore testing is theatre).

## 5) Data retention and compliance

- Classify data: PII, PCI, secrets, logs.
- Enforce retention with lifecycle + legal hold when required.
- Centralize audit logs (CloudTrail) into a log archive account.

## 6) Data transfer and cost gotchas

- Cross-AZ traffic can cost money (depends on path). Avoid needless cross-AZ chatty designs.
- NAT Gateway data processing cost is often huge; prefer VPC endpoints.


