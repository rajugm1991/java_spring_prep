# LLD Best Practices

## Table of Contents
1. [Introduction](#introduction)
2. [Code Quality](#code-quality)
3. [Maintainability](#maintainability)
4. [Scalability](#scalability)
5. [Design Principles](#design-principles)
6. [Real-World Examples](#real-world-examples)

---

## Introduction

**Best Practices** guide us to write better, more maintainable code. Following these practices leads to professional-quality software.

---

## Code Quality

### 1. Clear Naming

```java
// Bad: Unclear names
public void proc(Data d) { ... }

// Good: Clear names
public void processOrder(Order order) { ... }
```

---

### 2. Small Methods

```java
// Bad: Long method
public void processOrder(Order order) {
    // 100 lines of code
}

// Good: Small, focused methods
public void processOrder(Order order) {
    validateOrder(order);
    calculateTotal(order);
    processPayment(order);
    updateInventory(order);
}
```

---

### 3. Avoid Magic Numbers

```java
// Bad: Magic numbers
if (age >= 18) { ... }
if (price > 100) { ... }

// Good: Named constants
private static final int MINIMUM_AGE = 18;
private static final double DISCOUNT_THRESHOLD = 100.0;

if (age >= MINIMUM_AGE) { ... }
if (price > DISCOUNT_THRESHOLD) { ... }
```

---

## Maintainability

### 1. Single Responsibility

```java
// Bad: Multiple responsibilities
public class OrderService {
    public void processOrder(Order order) {
        validateOrder(order);
        calculateTotal(order);
        processPayment(order);
        sendEmail(order);
        updateDatabase(order);
        generateReport(order);
    }
}

// Good: Single responsibility
public class OrderService {
    private OrderValidator validator;
    private OrderCalculator calculator;
    private PaymentService paymentService;
    private EmailService emailService;
    private OrderRepository repository;
    
    public void processOrder(Order order) {
        validator.validate(order);
        calculator.calculateTotal(order);
        paymentService.processPayment(order);
        emailService.sendConfirmation(order);
        repository.save(order);
    }
}
```

---

### 2. DRY (Don't Repeat Yourself)

```java
// Bad: Repeated code
public void createUser(String name, String email) {
    if (name == null || name.isEmpty()) {
        throw new IllegalArgumentException("Name required");
    }
    // ...
}

public void updateUser(String name, String email) {
    if (name == null || name.isEmpty()) {
        throw new IllegalArgumentException("Name required");
    }
    // ...
}

// Good: Extract common logic
private void validateName(String name) {
    if (name == null || name.isEmpty()) {
        throw new IllegalArgumentException("Name required");
    }
}
```

---

### 3. Comments and Documentation

```java
/**
 * Processes an order by validating, calculating total, and processing payment.
 * 
 * @param order The order to process
 * @return Processed order with updated status
 * @throws InvalidOrderException if order is invalid
 * @throws PaymentException if payment fails
 */
public Order processOrder(Order order) {
    // Implementation
}
```

---

## Scalability

### 1. Design for Extension

```java
// Good: Open for extension
public interface PaymentProcessor {
    PaymentResult process(PaymentRequest request);
}

public class CreditCardProcessor implements PaymentProcessor { ... }
public class PayPalProcessor implements PaymentProcessor { ... }
// Easy to add new processors
```

---

### 2. Avoid Tight Coupling

```java
// Bad: Tight coupling
public class OrderService {
    private StripePaymentProcessor processor = new StripePaymentProcessor();
}

// Good: Loose coupling
public class OrderService {
    private PaymentProcessor processor;
    
    public OrderService(PaymentProcessor processor) {
        this.processor = processor;
    }
}
```

---

## Design Principles

### 1. SOLID Principles

- **S**ingle Responsibility Principle
- **O**pen/Closed Principle
- **L**iskov Substitution Principle
- **I**nterface Segregation Principle
- **D**ependency Inversion Principle

---

### 2. Design Patterns

Use appropriate design patterns:
- **Strategy:** For interchangeable algorithms
- **Factory:** For object creation
- **Observer:** For event handling
- **Singleton:** For single instance

---

### 3. Error Handling

```java
// Good: Proper error handling
public Order createOrder(OrderRequest request) {
    try {
        validateRequest(request);
        Order order = buildOrder(request);
        return repository.save(order);
    } catch (ValidationException e) {
        logger.warn("Invalid request", e);
        throw e;
    } catch (Exception e) {
        logger.error("Unexpected error", e);
        throw new OrderCreationException("Failed to create order", e);
    }
}
```

---

## Real-World Examples

### Example 1: Well-Designed Service

```java
public class OrderService {
    private final OrderValidator validator;
    private final OrderCalculator calculator;
    private final PaymentService paymentService;
    private final OrderRepository repository;
    private final Logger logger;
    
    public OrderService(OrderValidator validator,
                       OrderCalculator calculator,
                       PaymentService paymentService,
                       OrderRepository repository,
                       Logger logger) {
        this.validator = validator;
        this.calculator = calculator;
        this.paymentService = paymentService;
        this.repository = repository;
        this.logger = logger;
    }
    
    public Order processOrder(OrderRequest request) {
        logger.info("Processing order: {}", request);
        
        try {
            validator.validate(request);
            Order order = calculator.calculateTotal(request);
            PaymentResult payment = paymentService.processPayment(order);
            
            if (payment.isSuccess()) {
                order.confirm();
                repository.save(order);
                logger.info("Order processed successfully: {}", order.getId());
            } else {
                logger.warn("Payment failed for order: {}", order.getId());
            }
            
            return order;
        } catch (ValidationException e) {
            logger.error("Validation failed", e);
            throw e;
        } catch (Exception e) {
            logger.error("Unexpected error processing order", e);
            throw new OrderProcessingException("Failed to process order", e);
        }
    }
}
```

---

### Example 2: Clean Class Design

```java
public class ShoppingCart {
    private final Long userId;
    private final List<CartItem> items;
    private final CartCalculator calculator;
    
    public ShoppingCart(Long userId) {
        this.userId = userId;
        this.items = new ArrayList<>();
        this.calculator = new CartCalculator();
    }
    
    public void addItem(Product product, int quantity) {
        validateProduct(product);
        validateQuantity(quantity);
        
        CartItem existingItem = findItem(product);
        if (existingItem != null) {
            existingItem.increaseQuantity(quantity);
        } else {
            items.add(new CartItem(product, quantity));
        }
    }
    
    public double calculateTotal() {
        return calculator.calculateTotal(items);
    }
    
    private void validateProduct(Product product) {
        if (product == null) {
            throw new IllegalArgumentException("Product cannot be null");
        }
    }
    
    private void validateQuantity(int quantity) {
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be positive");
        }
    }
    
    private CartItem findItem(Product product) {
        return items.stream()
            .filter(item -> item.getProduct().equals(product))
            .findFirst()
            .orElse(null);
    }
}
```

---

## Summary

**Key Practices:**
1. ✅ **Clear Naming:** Self-documenting code
2. ✅ **Small Methods:** Single responsibility
3. ✅ **DRY:** Don't repeat code
4. ✅ **SOLID:** Follow design principles
5. ✅ **Error Handling:** Proper exception handling
6. ✅ **Documentation:** Clear comments

**Next Steps:**
- Review [Code Review Checklist](./25_CODE_REVIEW_CHECKLIST.md)
- Practice [Real-World Problems](./12_PARKING_LOT_SYSTEM.md)
- Learn [Interview Preparation](./24_INTERVIEW_PREPARATION.md)

---

**References:**
- [SOLID Principles](./03_SOLID_PRINCIPLES.md)
- [Design Patterns](./08_CREATIONAL_PATTERNS.md)
- [Testing](./22_TESTING_IN_LLD.md)

