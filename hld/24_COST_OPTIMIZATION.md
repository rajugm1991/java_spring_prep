# Cost Optimization

## Table of Contents
1. [Introduction](#introduction)
2. [Resource Right-Sizing](#resource-right-sizing)
3. [Auto-Scaling Strategies](#auto-scaling-strategies)
4. [Cost Analysis](#cost-analysis)
5. [Reserved Instances vs On-Demand](#reserved-instances-vs-on-demand)
6. [Spot Instances](#spot-instances)
7. [Cost Monitoring](#cost-monitoring)
8. [Real-World Examples](#real-world-examples)

---

## Introduction

**Cost Optimization** balances performance with infrastructure costs.

### Cost Factors

1. **Compute:** Servers, containers
2. **Storage:** Databases, object storage
3. **Network:** Bandwidth, CDN
4. **Services:** Managed services

---

## Resource Right-Sizing

### Problem: Over-Provisioning

```
100 servers × $100/month = $10,000/month
Only using 20% capacity ❌
```

### Solution: Right-Sizing

```
20 servers × $100/month = $2,000/month
Using 80% capacity ✅
```

### How to Right-Size

1. **Monitor Usage:**
   - CPU: Average 40-60%
   - Memory: Average 50-70%
   - Network: Peak usage

2. **Downsize:**
   - If consistently < 30% → Smaller instance

3. **Upsize:**
   - If consistently > 80% → Larger instance

---

## Auto-Scaling Strategies

### Reactive Scaling

**Scale based on current metrics:**
```
CPU > 80% → Add server
CPU < 30% → Remove server
```

**Cost:**
```
Normal: 5 servers × $100 = $500/month
Peak: 10 servers × $100 = $1,000/month
Average: $750/month
```

---

### Predictive Scaling

**Scale based on predicted load:**
```
Black Friday coming → Pre-scale servers
```

**Cost:**
```
Pre-scale: 10 servers × $100 = $1,000/month
Only for peak days: $1,000 × 5 days = $5,000/year
```

---

### Scheduled Scaling

**Scale based on time:**
```
9 AM - 5 PM: 10 servers
5 PM - 9 AM: 5 servers
```

**Cost:**
```
Business hours: 10 × 8 hours = 80 server-hours/day
Off hours: 5 × 16 hours = 80 server-hours/day
Total: 160 server-hours/day
Monthly: 160 × 30 = 4,800 server-hours
Cost: 4,800 × $0.10 = $480/month
```

---

## Cost Analysis

### Cost Breakdown

**Example: E-Commerce Platform**

```
Compute (Servers):
- 20 servers × $100/month = $2,000

Database:
- 1 master + 3 replicas × $200 = $800

Storage:
- 10 TB × $0.10/GB = $1,000

CDN:
- 100 TB/month × $0.05/GB = $5,000

Total: $8,800/month
```

---

### Cost Optimization Opportunities

1. **Reserved Instances:** 30-40% savings
2. **Spot Instances:** 70-90% savings
3. **Right-Sizing:** 20-30% savings
4. **Auto-Scaling:** 30-50% savings

---

## Reserved Instances vs On-Demand

### On-Demand Instances

**Pay as you go:**
```
$100/month per server
Flexible, no commitment
```

**Use Case:**
- Variable workloads
- Short-term projects
- Testing

---

### Reserved Instances

**Commit for 1-3 years:**
```
On-Demand: $100/month
Reserved (1 year): $60/month (40% savings)
Reserved (3 years): $40/month (60% savings)
```

**Use Case:**
- Steady workloads
- Long-term projects
- Predictable usage

---

## Spot Instances

**Bid on unused capacity:**
```
On-Demand: $100/month
Spot: $20-30/month (70-80% savings)
```

**Trade-off:**
- ✅ Very cheap
- ❌ Can be terminated with 2-minute notice

**Use Case:**
- Batch processing
- Non-critical workloads
- Fault-tolerant applications

---

## Cost Monitoring

### Key Metrics

1. **Cost per Request:**
   ```
   Total cost / Total requests
   ```

2. **Cost per User:**
   ```
   Total cost / Active users
   ```

3. **Cost Trends:**
   ```
   Compare month-over-month
   ```

### Cost Alerts

```
If cost > $10,000/month → Alert
If cost increase > 20% → Alert
```

---

## Real-World Examples

### Example 1: Startup

**Initial Setup:**
```
On-Demand: $1,000/month
```

**After Optimization:**
```
Reserved Instances: $600/month (40% savings)
Right-Sizing: $480/month (52% savings)
Auto-Scaling: $350/month (65% savings)
```

---

### Example 2: Enterprise

**Before Optimization:**
```
100 servers × $100 = $10,000/month
Over-provisioned: Using 20% capacity
```

**After Optimization:**
```
20 servers × $100 = $2,000/month
Reserved: $1,200/month (40% savings)
Right-sized: Using 80% capacity
Total savings: 88%
```

---

### Example 3: E-Commerce (Seasonal)

**Normal:**
```
10 servers × $100 = $1,000/month
```

**Black Friday:**
```
Auto-scale to 50 servers
50 × $100 × 1 day = $167
Total: $1,167/month
```

---

## Cost Optimization Checklist

- [ ] Right-size instances
- [ ] Use reserved instances for steady workloads
- [ ] Use spot instances for fault-tolerant workloads
- [ ] Implement auto-scaling
- [ ] Monitor and alert on costs
- [ ] Review and optimize regularly
- [ ] Use managed services (reduce operational cost)
- [ ] Optimize storage (delete old data, compress)

---

## Summary

**Key Strategies:**
1. ✅ **Right-Sizing:** Match capacity to usage
2. ✅ **Reserved Instances:** 30-60% savings
3. ✅ **Spot Instances:** 70-90% savings
4. ✅ **Auto-Scaling:** Pay only for what you use
5. ✅ **Monitor:** Track and optimize costs

**Next Steps:**
- Review [System Monitoring](./22_SYSTEM_MONITORING.md)
- Understand [Performance Optimization](./06_PERFORMANCE_OPTIMIZATION.md)
- Practice [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [AWS Cost Optimization](https://aws.amazon.com/pricing/)
- [Cost Management](https://algomaster.io/learn/system-design)
