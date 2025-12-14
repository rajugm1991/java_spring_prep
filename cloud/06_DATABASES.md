# Databases on AWS — Staff Tradeoffs

## 1) Start with access patterns

Staff interviews reward this ordering:
1. Entities + relationships
2. Query patterns (read/write mix, secondary indexes, joins, range scans)
3. Consistency needs (strong vs eventual)
4. Availability + latency targets
5. Growth (data size, throughput), multi-region needs

## 2) RDS / Aurora (relational)

### When it’s a good default
- You need joins, transactions, complex queries, constraints, strong consistency.
- You want managed backups, patching, Multi-AZ.

### Staff points to mention
- **Multi-AZ**: HA, not read scaling.
- **Read replicas**: read scaling, eventual consistency.
- **Connection management**: pooling (RDS Proxy) for spiky workloads/serverless.
- Migrations: expand/contract, online schema changes, rollback constraints.

### Aurora specific
- Good performance and read scaling; managed storage layer.
- Consider cost and operational complexity vs vanilla RDS.

## 3) DynamoDB (key-value / document)

### Great when
- Predictable access by key/partition key, massive scale, low-latency.
- You can model around partition keys and query constraints.

### Staff-grade topics
- Partition key selection and hot partitions.
- Secondary indexes (GSIs/LSIs) and their tradeoffs.
- Capacity modes: on-demand vs provisioned + autoscaling.
- **PITR**, streams, TTL.
- Transaction support exists but is not “Postgres”.

## 4) Caching (ElastiCache: Redis/Memcached)

- Use for read-heavy workloads, session state, rate limiting, short-lived data.
- Staff callouts:
  - Cache-aside vs write-through
  - Stampede protection (locking, jitter)
  - TTL strategy + invalidation
  - Redis durability choices and failure behavior

## 5) Search (OpenSearch)

- Use for full-text search, faceting, relevancy, analytics-ish queries.
- Don’t use as source of truth; sync via events/CDC.

## 6) Data consistency and idempotency

- If you use queues/events, assume at-least-once; design **idempotent writes**.
- Use dedupe keys, conditional writes, unique constraints, or idempotency tables.

## 7) Multi-region data strategy (only when required)

- Relational: often active-passive with read replicas; active-active is complex.
- NoSQL: DynamoDB global tables can help but require careful conflict handling.
- Always tie multi-region to explicit requirement (latency or continuity).


