# System Monitoring & Observability

## Table of Contents
1. [Introduction](#introduction)
2. [Three Pillars of Observability](#three-pillars-of-observability)
3. [Logging](#logging)
4. [Metrics](#metrics)
5. [Tracing](#tracing)
6. [Alerting](#alerting)
7. [Real-World Examples](#real-world-examples)

---

## Introduction

**Observability** is the ability to understand what's happening inside a system by examining its outputs.

### Why Observability Matters

- **Debug Issues:** Find root cause quickly
- **Performance:** Identify bottlenecks
- **Capacity Planning:** Understand resource needs
- **User Experience:** Monitor user-facing metrics

---

## Three Pillars of Observability

### 1. Logs

**What happened?**
- Application logs
- Error logs
- Access logs

### 2. Metrics

**How is it performing?**
- CPU usage
- Memory usage
- Request rate
- Error rate

### 3. Traces

**Where is the bottleneck?**
- Request flow
- Service dependencies
- Latency breakdown

---

## Logging

### Log Levels

```
DEBUG: Detailed information for debugging
INFO:  General information
WARN:  Warning messages
ERROR: Error messages
FATAL: Critical errors
```

### Structured Logging

**Bad (Unstructured):**
```
"User 123 placed order 456"
```

**Good (Structured):**
```json
{
  "level": "INFO",
  "timestamp": "2024-01-01T10:00:00Z",
  "service": "order-service",
  "userId": 123,
  "orderId": 456,
  "message": "Order placed"
}
```

### Log Aggregation

**ELK Stack:**
```
Application → Logstash → Elasticsearch → Kibana
```

**Example:**
```java
logger.info("Order placed", 
    Map.of("userId", userId, "orderId", orderId));
```

---

## Metrics

### Types of Metrics

#### 1. **Counter**

**Monotonically increasing:**
```
Total requests: 1,000,000
```

**Example:**
```java
meterRegistry.counter("http.requests.total").increment();
```

---

#### 2. **Gauge**

**Current value:**
```
Active connections: 150
```

**Example:**
```java
meterRegistry.gauge("connections.active", connections.size());
```

---

#### 3. **Histogram**

**Distribution of values:**
```
Response time: P50=50ms, P95=200ms, P99=500ms
```

**Example:**
```java
Timer.Sample sample = Timer.start(meterRegistry);
// ... operation ...
sample.stop(meterRegistry.timer("operation.duration"));
```

---

### Key Metrics to Monitor

1. **System Metrics:**
   - CPU usage
   - Memory usage
   - Disk I/O
   - Network I/O

2. **Application Metrics:**
   - Request rate
   - Error rate
   - Response time
   - Throughput

3. **Business Metrics:**
   - Orders per second
   - Revenue
   - User signups

---

## Tracing

### Distributed Tracing

**Track request across services:**
```
Request → Service A → Service B → Service C
Trace ID: abc123 (same for all)
```

### OpenTelemetry

**Standard for tracing:**
```java
Span span = tracer.spanBuilder("processOrder")
    .startSpan();
try {
    // Process order
} finally {
    span.end();
}
```

### Trace Visualization

**Jaeger/Zipkin:**
```
Service A (50ms)
  └─ Service B (30ms)
      └─ Service C (20ms)
```

---

## Alerting

### Alert Levels

1. **Critical:** System down, immediate action needed
2. **Warning:** Degraded performance, investigate
3. **Info:** Important events, monitor

### Alert Rules

**Example:**
```
If error_rate > 5% for 5 minutes → Alert
If response_time > 1s for 10 minutes → Alert
If CPU > 90% for 5 minutes → Alert
```

### Alerting Stack

```
Metrics → Prometheus → Alertmanager → PagerDuty/Slack
```

---

## Real-World Examples

### Example 1: Netflix

**Observability Stack:**
- **Logs:** ELK Stack
- **Metrics:** Atlas (Netflix's metrics system)
- **Traces:** Zipkin
- **Alerts:** PagerDuty

**Key Metrics:**
- Stream starts
- Buffering events
- Playback errors

---

### Example 2: Uber

**Observability Stack:**
- **Logs:** ELK Stack
- **Metrics:** Prometheus
- **Traces:** Jaeger
- **Alerts:** PagerDuty

**Key Metrics:**
- Ride requests
- Driver availability
- ETA accuracy

---

### Example 3: E-Commerce Platform

**Monitoring Setup:**
```yaml
Metrics:
  - Request rate
  - Error rate
  - Response time
  - Database query time
  - Cache hit rate

Alerts:
  - Error rate > 1%
  - Response time > 500ms
  - Database connections > 80%
```

---

## Summary

**Key Concepts:**
1. ✅ **Logs:** What happened (ELK Stack)
2. ✅ **Metrics:** How is it performing (Prometheus)
3. ✅ **Traces:** Where is the bottleneck (Jaeger)
4. ✅ **Alerts:** Notify on issues (Alertmanager)

**Next Steps:**
- Learn [Security](./23_SECURITY_IN_SYSTEM_DESIGN.md)
- Understand [Cost Optimization](./24_COST_OPTIMIZATION.md)
- Practice [Real-World Designs](./13_DESIGNING_URL_SHORTENER.md)

---

**References:**
- [Observability](https://algomaster.io/learn/system-design)
- [OpenTelemetry](https://opentelemetry.io/)
