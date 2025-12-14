# Event-Driven Architecture Problems - Real-World Problems

> **Curated from top tech companies and real production scenarios**

---

## Problem 1: Design Event-Driven Order Processing

**Problem Statement:**
Design an event-driven system for processing orders across multiple services.

**Requirements:**
- Order creation event
- Inventory reservation
- Payment processing
- Shipping notification
- Event replay capability

**Solution:**

```java
// Event Publisher
@Service
public class OrderEventPublisher {
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;
    
    public void publishOrderCreated(Order order) {
        OrderCreatedEvent event = new OrderCreatedEvent(order);
        kafkaTemplate.send("order-created", order.getId(), event);
    }
}

// Event Consumers
@KafkaListener(topics = "order-created", groupId = "inventory-service")
public void handleOrderCreated(OrderCreatedEvent event) {
    inventoryService.reserve(event.getOrderId(), event.getItems());
}

@KafkaListener(topics = "order-created", groupId = "payment-service")
public void handleOrderCreated(OrderCreatedEvent event) {
    paymentService.process(event.getOrderId(), event.getAmount());
}
```

**Key Patterns:**
- Event Sourcing
- CQRS
- Saga Pattern
- Event Replay

---

*This guide contains event-driven architecture problems with Kafka, event sourcing, and CQRS patterns.*

---

*This guide is continuously updated with new problems.*

