# AWS Cheatsheets (Quick Recall)

## IAM quick recall

- Prefer **roles** + **STS** (short-lived).
- Human access via **SSO** (Identity Center).
- Scale with **ABAC tags** + permission boundaries.
- Separate duties; keep break-glass isolated and audited.

## VPC quick recall

- Public subnet = route to **IGW**.
- Private subnet internet egress = **NAT Gateway** (or avoid with endpoints).
- Prefer **VPC endpoints** to reduce NAT cost and exposure.
- Peering: simple, non-transitive. TGW: scalable hub/spoke. PrivateLink: service exposure without full peering.

## Load balancers

- **ALB**: L7 HTTP routing + WAF.
- **NLB**: L4 TCP/UDP, static IPs, high throughput.

## Messaging quick recall

- Assume **at-least-once** → design **idempotency**.
- Retries with backoff + jitter, bounded.
- DLQs + alarms on DLQ depth and message age.

## DR quick recall

- Patterns: backup/restore → pilot light → warm standby → active-active.
- Tie to **RTO/RPO**, not vibes.
- Test restores and run game days.

## Cost quick recall

- Big hidden costs: **NAT**, inter-AZ, log retention, oversized DBs.
- Use endpoints, lifecycle rules, right-sizing, Savings Plans.


