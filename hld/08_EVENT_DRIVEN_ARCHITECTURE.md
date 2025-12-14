# Event-Driven Architecture (EDA) — System Design Chapter

Event-driven architecture decouples services by communicating via **events** rather than synchronous request/response. At staff level, the key is designing for **reliability, idempotency, ordering, schema evolution, and operability**.

---

## 1) What is an event?

- **Event**: “Something that happened” (immutable fact), e.g., `OrderPlaced`, `PaymentAuthorized`.
- **Command**: “Do something” (intent), e.g., `AuthorizePayment`.
- **Query**: “Tell me something” (read), e.g., `GetOrderStatus`.

Staff signal: keep these concepts clean; they drive correct architectures.

---

## 2) Why EDA? (benefits + costs)

### Benefits
- Loose coupling (producers don’t need to know consumers)
- Resilience via buffering (queues/streams)
- Scalability (fan-out, independent scaling of consumers)
- Auditability and replay (with durable logs)

### Costs / risks
- Debuggability is harder (distributed, async)
- Eventual consistency (state converges over time)
- Duplicate delivery is common → **idempotency required**
- Ordering constraints can be tricky

---

## 3) Core building blocks

### Producers and consumers
- Producers emit events.
- Consumers subscribe and process events.
- Expect **at-least-once delivery** end-to-end unless proven otherwise.

### Broker options (conceptually)
- **Queue** (e.g., RabbitMQ/SQS):
  - great for work distribution and buffering
  - consumers compete; each message processed by one consumer (typically)
- **Pub/Sub** (e.g., SNS/EventBridge):
  - fan-out to multiple consumers
- **Stream log** (e.g., Kafka/Kinesis):
  - ordered log within partitions/shards
  - replay and consumer offsets are first-class

---

## 4) Common EDA patterns (know these cold)

### 4.1 Pub-Sub
When `OrderPlaced` occurs, multiple systems react:
- inventory reservation
- payment authorization
- email notification
- analytics

```
Order Service → Event Bus → {Inventory, Payments, Notifications, Analytics}
```

### 4.2 Outbox pattern (transaction + publish safety)

Problem: you update DB and publish an event; one can succeed while the other fails.

Solution:
- Write business update + **outbox row** in the same DB transaction.
- A publisher reads outbox and publishes events.
- Mark outbox row as published (idempotent).

This is a staff-level “must know” because it avoids ghost states.

### 4.3 Saga (distributed transaction)

When a workflow spans services (order → payment → inventory → shipping), use:
- **Orchestration**: centralized coordinator (clear flow, easier debugging)
- **Choreography**: services react to events (looser coupling, can get messy)

### 4.4 CQRS
- Separate write model (commands) from read model (queries).
- Often paired with read-optimized projections.

### 4.5 Event sourcing (use carefully)
- Store a log of events as the source of truth.
- Rebuild state by replaying events.
- Powerful, but operationally complex (schema evolution, replays, projections).

---

## 5) Delivery semantics: idempotency and exactly-once business effects

### Assume at-least-once
Duplicate events are normal under retries, timeouts, consumer restarts.

### Idempotency toolbox
- Event ID + dedupe table (with TTL)
- Conditional writes (DynamoDB conditions / SQL unique constraints)
- Idempotency key per command/request

Goal: **exactly-once business effect**, even with at-least-once delivery.

---

## 6) Ordering and partitioning

Ordering is usually only guaranteed within a partition/shard.

Design rule:
- Choose partition key to preserve the ordering you need:
  - e.g., partition by `orderId` to keep order events in sequence

Tradeoff:
- Partitioning by a single “hot” key can create hotspots.

---

## 7) Schema evolution and compatibility

Staff interviews often ask: “How do you evolve events without breaking consumers?”

Principles:
- Prefer **backward compatible** changes:
  - add optional fields
  - avoid removing/renaming fields abruptly
- Version events carefully (semantic versioning or schema registry strategy)
- Consumers should tolerate unknown fields; producers should avoid breaking changes.

---

## 8) Error handling: retries, DLQs, poison pills

- Retry with backoff + jitter (bounded).
- Use **DLQ** for messages that repeatedly fail.
- Alert on DLQ depth and message age.
- Provide replay tooling and operational runbooks.

---

## 9) Observability and debugging EDA

Minimum expectations:
- Correlation IDs propagate from command → events → downstream actions.
- Metrics:
  - consumer lag / queue age
  - processing success/fail rate
  - retry count and DLQ volume
- Tracing:
  - at least trace spans around publish and consume

Staff signal: you can describe how you’d debug “orders stuck in PROCESSING”.

---

## 10) Example: E-commerce order flow (EDA)

### Events (sample)
- `OrderPlaced`
- `InventoryReserved` / `InventoryRejected`
- `PaymentAuthorized` / `PaymentFailed`
- `OrderConfirmed` / `OrderCancelled`

### Flow (simplified)
```
Client → Order Service (command)
            ↓ (DB tx + outbox)
         OrderPlaced event
            ↓
  Inventory + Payment consumers
            ↓
        OrderConfirmed (or Cancelled)
```

Tradeoff discussion:
- eventual consistency (order status changes over time)
- compensations instead of rollbacks
- idempotency to prevent double-reservation/double-charge

---

## 11) Interview prompts to practice

1. Implement order processing with **exactly-once business effect**.
2. Handle “duplicate events” and “out-of-order events”.
3. Evolve event schema while 20 consumer services run different versions.
4. Debug consumer lag during peak load.

---

## 12) Relationship to other guides

- Deeper implementation notes:
  - `docs/imp/EVENT_DRIVEN_ARCHITECTURE_GUIDE.md`
  - `docs/KAFKA_QUICK_REFERENCE.md`


