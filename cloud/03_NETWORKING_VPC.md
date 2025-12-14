# Networking (VPC) — Staff Interview Guide

## 1) VPC fundamentals (the minimum to be fluent)

- **VPC**: an isolated network (CIDR block).
- **Subnets**: per-AZ slices of the VPC CIDR.
  - **Public subnet**: route table has a route to **Internet Gateway (IGW)**.
  - **Private subnet**: no direct IGW route.
- **Route tables**: control routing for subnets.
- **Security Groups (SGs)**: stateful firewall for ENIs (instance/task/LB).
- **NACLs**: stateless subnet-level firewall (coarse-grained).
- **NAT Gateway**: outbound internet for private subnets.
- **VPC endpoints**: private connectivity to AWS services.

## 2) Default production VPC layout (good “staff default”)

- **3 AZs** if the region supports it and your availability needs justify it (2 AZs minimum).
- Per AZ:
  - **Public subnet**: ALB/NLB, NAT gateway (if used)
  - **Private app subnet**: ECS/EKS/EC2 workloads
  - **Private data subnet**: RDS/Aurora/ElastiCache
- **Egress controls**:
  - Prefer **VPC endpoints** (S3, DynamoDB, ECR, CloudWatch Logs) to reduce NAT cost + exposure.
  - Centralize egress if your org requires it (egress VPC + inspection).

## 3) Routing patterns you should be able to explain

### Internet-bound traffic from private subnets
- Private subnet default route → **NAT Gateway** in public subnet → **IGW**.

### Private access to AWS services
- **Gateway endpoints**: S3, DynamoDB (route-table based).
- **Interface endpoints (PrivateLink)**: most AWS services (ENI-based).

### Cross-VPC connectivity
- **VPC peering**: simple, non-transitive, scaling limits.
- **Transit Gateway (TGW)**: hub-and-spoke, scalable, supports centralized routing/inspection.
- **PrivateLink**: expose a service in one VPC to consumers privately without full network peering.

### Hybrid connectivity
- **Site-to-site VPN**: quicker, internet-based.
- **Direct Connect (DX)**: dedicated, more stable, supports higher bandwidth/latency needs.

## 4) Load balancing and L7/L4 tradeoffs

- **ALB (L7)**:
  - HTTP/HTTPS, path/host routing, WebSockets, integrates with WAF
  - Great for microservices routing and modern web APIs
- **NLB (L4)**:
  - TCP/UDP, very high throughput/low latency, static IPs
  - Good for non-HTTP protocols or when you need static addresses

## 5) DNS and traffic management

- **Route 53**:
  - Weighted routing, latency-based, geolocation, failover
  - Health checks (with caveats; don’t rely on DNS alone for fast failover)
- **CloudFront**:
  - edge caching, origin shielding, WAF integration, TLS termination

## 6) Subnet sizing & IP management (often overlooked)

Failure mode: IP exhaustion (especially with EKS/ECS + ENIs).

- Pick CIDR blocks with growth in mind.
- If you run EKS, understand **pod-to-IP** model and VPC CNI implications (IP demand).
- Use multiple subnets per AZ if you need scaling/segmentation.

## 7) Security groups: common staff patterns

- **Inbound**: only from known SGs (SG-to-SG references) where possible.
- **Outbound**: restrict egress for sensitive workloads (at least to required ports/destinations).
- “Default allow all egress” is common but not always acceptable in regulated environments.

## 8) Interview prompts to practice

1. **Design a VPC** for a 3-tier app with private DB and controlled internet egress.
2. Add **private connectivity** to S3/DynamoDB without NAT.
3. Connect 20 VPCs (multiple teams) with shared services and egress inspection.
4. Expose one internal service to many consumer VPCs without full peering.

## 9) Common pitfalls (call these out explicitly)

- Single NAT Gateway for multiple AZs (AZ failure can break egress; also cross-AZ data charges).
- No plan for IP growth (EKS ENIs, Lambda in VPC, interface endpoints).
- Overusing peering for many VPCs (hard to manage, not transitive).
- Treating NACLs as primary security boundary (SGs are usually the main control).


