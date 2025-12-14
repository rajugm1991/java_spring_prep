# Monitoring & Observability Problems - Real-World Problems

> **Curated from top tech companies and real production scenarios**

---

## Problem 1: Design Observability System

**Problem Statement:**
Design observability system for microservices architecture with 20+ services.

**Requirements:**
- Logging
- Metrics
- Tracing
- Alerting
- Dashboards

**Solution:**

```java
// Distributed Tracing
@RestController
public class OrderController {
    @Autowired
    private Tracer tracer;
    
    @PostMapping("/orders")
    public ResponseEntity<Order> createOrder(@RequestBody OrderRequest request) {
        Span span = tracer.nextSpan().name("create-order").start();
        try (Tracer.SpanInScope ws = tracer.withSpanInScope(span)) {
            // Business logic
            Order order = orderService.createOrder(request);
            span.tag("order.id", order.getId().toString());
            return ResponseEntity.ok(order);
        } finally {
            span.end();
        }
    }
}

// Metrics
@RestController
public class ProductController {
    @Autowired
    private MeterRegistry meterRegistry;
    
    @GetMapping("/products/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable Long id) {
        Timer.Sample sample = Timer.start(meterRegistry);
        try {
            Product product = productService.getProduct(id);
            meterRegistry.counter("product.requests", "status", "success").increment();
            return ResponseEntity.ok(product);
        } catch (Exception e) {
            meterRegistry.counter("product.requests", "status", "error").increment();
            throw e;
        } finally {
            sample.stop(meterRegistry.timer("product.request.duration"));
        }
    }
}
```

**Observability Stack:**
- **Logging**: ELK Stack (Elasticsearch, Logstash, Kibana)
- **Metrics**: Prometheus + Grafana
- **Tracing**: Jaeger/Zipkin
- **APM**: New Relic, Datadog

---

*This guide contains monitoring and observability problems covering logging, metrics, tracing, and alerting.*

---

*This guide is continuously updated with new problems.*

