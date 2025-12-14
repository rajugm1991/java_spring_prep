# Code Review Checklist

## Table of Contents
1. [Introduction](#introduction)
2. [Design & Architecture](#design--architecture)
3. [Code Quality](#code-quality)
4. [Performance](#performance)
5. [Security](#security)
6. [Testing](#testing)

---

## Introduction

**Code Review Checklist** helps ensure code quality and maintainability. Use this checklist when reviewing LLD code.

---

## Design & Architecture

### ✅ SOLID Principles

- [ ] **Single Responsibility:** Each class has one reason to change
- [ ] **Open/Closed:** Open for extension, closed for modification
- [ ] **Liskov Substitution:** Subtypes are substitutable
- [ ] **Interface Segregation:** Clients don't depend on unused interfaces
- [ ] **Dependency Inversion:** Depend on abstractions, not concretions

### ✅ Design Patterns

- [ ] Appropriate design patterns used
- [ ] Patterns applied correctly
- [ ] Not over-engineered

### ✅ Class Design

- [ ] Clear class responsibilities
- [ ] Appropriate use of inheritance vs composition
- [ ] Proper encapsulation
- [ ] Good cohesion, low coupling

---

## Code Quality

### ✅ Naming

- [ ] Clear, descriptive names
- [ ] Consistent naming conventions
- [ ] No abbreviations (unless standard)
- [ ] Methods are verbs, classes are nouns

### ✅ Methods

- [ ] Single responsibility per method
- [ ] Methods are short (< 30 lines)
- [ ] Appropriate parameters (max 3-4)
- [ ] Meaningful return types
- [ ] No side effects (unless intentional)

### ✅ Code Structure

- [ ] No code duplication (DRY)
- [ ] Proper abstraction levels
- [ ] Clear control flow
- [ ] No magic numbers/strings

### ✅ Comments

- [ ] Code is self-documenting
- [ ] Complex logic is explained
- [ ] JavaDoc for public APIs
- [ ] No redundant comments

---

## Performance

### ✅ Efficiency

- [ ] No unnecessary object creation
- [ ] Efficient algorithms used
- [ ] Proper use of collections
- [ ] No memory leaks

### ✅ Concurrency

- [ ] Thread-safe if needed
- [ ] Proper synchronization
- [ ] No deadlocks
- [ ] Efficient concurrent collections

---

## Security

### ✅ Input Validation

- [ ] All inputs validated
- [ ] Null checks where needed
- [ ] Range checks for numbers
- [ ] Sanitization for strings

### ✅ Error Handling

- [ ] Proper exception handling
- [ ] No sensitive info in errors
- [ ] Appropriate exception types
- [ ] Stack traces preserved

---

## Testing

### ✅ Test Coverage

- [ ] Unit tests present
- [ ] Edge cases covered
- [ ] Error cases tested
- [ ] Mocks used appropriately

### ✅ Testability

- [ ] Code is testable
- [ ] Dependencies injected
- [ ] No static dependencies
- [ ] Clear test structure

---

## Example Review

### Code to Review

```java
public class OrderService {
    private PaymentService ps;
    private OrderRepo or;
    
    public void proc(Order o) {
        if (o == null) return;
        PaymentResult pr = ps.pay(o.total);
        if (pr.success) {
            or.save(o);
            o.status = "CONFIRMED";
        }
    }
}
```

### Review Comments

**Design:**
- ❌ Poor naming: `proc`, `ps`, `or`
- ❌ No dependency injection
- ❌ Direct field access: `o.status = "CONFIRMED"`

**Code Quality:**
- ❌ Silent failure: `if (o == null) return;`
- ❌ No error handling
- ❌ Magic string: `"CONFIRMED"`

**Improved Version:**

```java
public class OrderService {
    private final PaymentService paymentService;
    private final OrderRepository orderRepository;
    private static final Logger logger = LoggerFactory.getLogger(OrderService.class);
    
    public OrderService(PaymentService paymentService, 
                       OrderRepository orderRepository) {
        this.paymentService = paymentService;
        this.orderRepository = orderRepository;
    }
    
    public Order processOrder(Order order) {
        if (order == null) {
            throw new IllegalArgumentException("Order cannot be null");
        }
        
        try {
            PaymentResult result = paymentService.processPayment(order.getTotal());
            if (result.isSuccess()) {
                order.confirm();
                orderRepository.save(order);
                logger.info("Order processed: {}", order.getId());
            } else {
                logger.warn("Payment failed for order: {}", order.getId());
            }
            return order;
        } catch (PaymentException e) {
            logger.error("Payment error", e);
            throw new OrderProcessingException("Failed to process order", e);
        }
    }
}
```

---

## Summary

**Review Focus Areas:**
1. ✅ **Design:** SOLID, patterns, architecture
2. ✅ **Quality:** Naming, structure, clarity
3. ✅ **Performance:** Efficiency, concurrency
4. ✅ **Security:** Validation, error handling
5. ✅ **Testing:** Coverage, testability

**Next Steps:**
- Review [Best Practices](./23_LLD_BEST_PRACTICES.md)
- Practice [Interview Preparation](./24_INTERVIEW_PREPARATION.md)
- Learn [Design Patterns](./08_CREATIONAL_PATTERNS.md)

---

**References:**
- [Best Practices](./23_LLD_BEST_PRACTICES.md)
- [SOLID Principles](./03_SOLID_PRINCIPLES.md)
- [Testing](./22_TESTING_IN_LLD.md)

