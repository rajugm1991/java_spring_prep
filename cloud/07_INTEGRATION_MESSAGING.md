# Integration & Messaging (AWS) — Staff Patterns

## 1) The big four and when to use them

- **SQS (queue)**: decouple producers/consumers, buffer spikes, at-least-once delivery.
- **SNS (pub/sub)**: fan-out notifications to many subscribers.
- **EventBridge (event bus)**: event routing, SaaS integrations, schema registry-like ergonomics.
- **Kinesis (stream)**: high-throughput ordered shards, streaming processing, replay.

Also:
- **Step Functions**: orchestration of workflows with retries/timeouts/compensation.

## 2) Delivery semantics (what to say out loud)

- Assume **at-least-once** unless you’ve proven otherwise end-to-end.
- Therefore: **idempotency is mandatory** for side effects (payments, inventory).

### Idempotency toolbox
- **Idempotency key** from client/request
- **Conditional write** (DynamoDB condition expressions)
- **Unique constraint** in relational DB
- **Idempotency table** with TTL
- **Exactly-once business effect** even if transport is at-least-once

## 3) Retries, DLQs, and poison pills

- Retries should be bounded with backoff + jitter.
- Send poison messages to **DLQ** and alert on DLQ depth.
- Add observability: message age, retry count, success/fail rates.

## 4) Ordering and consistency

- SQS standard: best-effort ordering.
- SQS FIFO: ordering per message group, lower throughput; requires dedupe semantics.
- Kinesis: ordering per shard; requires careful partitioning.

## 5) Outbox pattern (staff favorite)

Problem: you need to update DB and publish event atomically.

Solution:
- Write business transaction + **outbox record** in same DB transaction.
- A publisher reads outbox and publishes to EventBridge/SNS/Kinesis.
- Mark outbox sent (idempotent).

## 6) Saga patterns (orchestration vs choreography)

- **Orchestration** (Step Functions): clearer control flow, centralized state.
- **Choreography** (events): loosely coupled but harder to reason about.

Staff guidance: pick based on **complexity**, **ownership boundaries**, and **debuggability**.

## 7) Backpressure and overload protection

- Queues provide buffering but can hide slowness; monitor **queue age**.
- Use **concurrency limits** (Lambda reserved concurrency, worker pools).
- Shed load early (API throttling, WAF rules, rate limiting).


