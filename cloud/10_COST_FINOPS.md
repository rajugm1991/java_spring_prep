# Cost & FinOps on AWS — Staff Engineer Topics

## 1) How staff engineers talk about cost

- Cost is a **first-class requirement**: “What are the top 3 cost drivers?”
- Use **unit economics**: cost per order, per 1k requests, per GB processed.
- Apply **guardrails**: budgets/alerts, tagging, quotas, and architectural defaults.

## 2) Common AWS cost drivers (learn these cold)

- **Data transfer**: inter-AZ, inter-region, NAT gateway processing, internet egress.
- **Compute**: over-provisioned instances, idle clusters, always-on dev.
- **Storage**: unbounded logs, snapshots, old objects without lifecycle.
- **Databases**: oversized instances, unused replicas, high IOPS storage classes.
- **Observability**: log ingestion/retention, high-cardinality metrics.

## 3) Quick optimization playbook

1. **Measure**: cost explorer + tag coverage + top services.
2. **Stop waste**: idle resources, non-prod schedules, orphan volumes/snapshots.
3. **Right-size**: compute and DB sizing based on metrics.
4. **Commit**: Savings Plans / RIs for steady-state workloads.
5. **Architect**: reduce NAT usage with VPC endpoints; cache; async.

## 4) Tagging and allocation (org-scale necessity)

- Enforce tags: `owner`, `service`, `env`, `cost_center`, `data_classification`.
- Use ABAC tags to align identity + cost ownership (bonus staff signal).

## 5) Capacity vs cost tradeoffs (say explicitly)

- Multi-AZ adds cost; justify with availability needs.
- Active-active multi-region can be very expensive; only for strict requirements.
- Serverless reduces ops, but can increase unit cost at scale—measure.


