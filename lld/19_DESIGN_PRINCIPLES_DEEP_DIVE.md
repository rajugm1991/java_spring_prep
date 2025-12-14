# Design Principles Deep Dive

## Table of Contents
1. [Introduction](#introduction)
2. [DRY (Don't Repeat Yourself)](#dry-dont-repeat-yourself)
3. [KISS (Keep It Simple, Stupid)](#kiss-keep-it-simple-stupid)
4. [YAGNI (You Aren't Gonna Need It)](#yagni-you-arent-gonna-need-it)
5. [Composition over Inheritance](#composition-over-inheritance)
6. [Separation of Concerns](#separation-of-concerns)
7. [Real-World Examples](#real-world-examples)

---

## Introduction

Beyond SOLID principles, there are additional design principles that guide good software design.

---

## DRY (Don't Repeat Yourself)

**Principle:** Every piece of knowledge must have a single, unambiguous representation.

### Violation Example

```java
// Bad: Repeated code
public class UserService {
    public void createUser(String name, String email) {
        if (name == null || name.isEmpty()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Invalid email");
        }
        // Create user
    }
    
    public void updateUser(String name, String email) {
        if (name == null || name.isEmpty()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Invalid email");
        }
        // Update user
    }
}
```

### Following DRY

```java
// Good: Extract validation
public class UserValidator {
    public void validateName(String name) {
        if (name == null || name.isEmpty()) {
            throw new IllegalArgumentException("Name cannot be empty");
        }
    }
    
    public void validateEmail(String email) {
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Invalid email");
        }
    }
}

public class UserService {
    private UserValidator validator;
    
    public void createUser(String name, String email) {
        validator.validateName(name);
        validator.validateEmail(email);
        // Create user
    }
    
    public void updateUser(String name, String email) {
        validator.validateName(name);
        validator.validateEmail(email);
        // Update user
    }
}
```

---

## KISS (Keep It Simple, Stupid)

**Principle:** Simplicity should be a key goal in design.

### Violation Example

```java
// Bad: Overcomplicated
public class OrderProcessor {
    public void process(Order order) {
        Optional<Order> orderOptional = Optional.ofNullable(order);
        orderOptional.ifPresent(o -> {
            CompletableFuture.supplyAsync(() -> {
                return processOrderAsync(o);
            }).thenApply(result -> {
                return handleResult(result);
            }).exceptionally(throwable -> {
                return handleError(throwable);
            });
        });
    }
}
```

### Following KISS

```java
// Good: Simple and clear
public class OrderProcessor {
    public void process(Order order) {
        if (order == null) {
            throw new IllegalArgumentException("Order cannot be null");
        }
        
        try {
            processOrder(order);
        } catch (Exception e) {
            handleError(e);
        }
    }
    
    private void processOrder(Order order) {
        // Simple processing logic
    }
}
```

---

## YAGNI (You Aren't Gonna Need It)

**Principle:** Don't add functionality until it's necessary.

### Violation Example

```java
// Bad: Adding features "just in case"
public class PaymentProcessor {
    // Current requirement: Credit card only
    // But adding PayPal, Stripe, etc. "for future"
    
    public void processCreditCard() { ... }
    public void processPayPal() { ... }  // Not needed yet
    public void processStripe() { ... }  // Not needed yet
    public void processBitcoin() { ... } // Not needed yet
}
```

### Following YAGNI

```java
// Good: Only what's needed now
public class PaymentProcessor {
    // Current requirement: Credit card only
    public void processCreditCard() { ... }
    
    // Add other methods when actually needed
}
```

---

## Composition over Inheritance

**Principle:** Favor object composition over class inheritance.

### Inheritance Problem

```java
// Bad: Inheritance hierarchy
public class Vehicle {
    public void start() { ... }
    public void stop() { ... }
}

public class Car extends Vehicle {
    public void drive() { ... }
}

public class Truck extends Vehicle {
    public void drive() { ... }
}

public class Motorcycle extends Vehicle {
    public void drive() { ... }
}

// Problem: What if we need a flying car?
public class FlyingCar extends Car {
    // Can't extend both Car and Plane
}
```

### Composition Solution

```java
// Good: Composition
public interface Engine {
    void start();
    void stop();
}

public interface Drivable {
    void drive();
}

public interface Flyable {
    void fly();
}

public class Car {
    private Engine engine;
    private Drivable drivable;
    
    public Car(Engine engine, Drivable drivable) {
        this.engine = engine;
        this.drivable = drivable;
    }
    
    public void start() {
        engine.start();
    }
    
    public void drive() {
        drivable.drive();
    }
}

public class FlyingCar {
    private Engine engine;
    private Drivable drivable;
    private Flyable flyable;
    
    public FlyingCar(Engine engine, Drivable drivable, Flyable flyable) {
        this.engine = engine;
        this.drivable = drivable;
        this.flyable = flyable;
    }
    
    public void fly() {
        flyable.fly();
    }
}
```

---

## Separation of Concerns

**Principle:** Each module should focus on a single concern.

### Violation Example

```java
// Bad: Multiple concerns mixed
public class OrderService {
    public void processOrder(Order order) {
        // Validation
        if (order == null) { ... }
        
        // Business logic
        calculateTotal(order);
        
        // Database operations
        saveToDatabase(order);
        
        // Email sending
        sendEmail(order);
        
        // Logging
        logOrder(order);
    }
}
```

### Following Separation of Concerns

```java
// Good: Separate concerns
public class OrderService {
    private OrderValidator validator;
    private OrderCalculator calculator;
    private OrderRepository repository;
    private EmailService emailService;
    private Logger logger;
    
    public void processOrder(Order order) {
        validator.validate(order);
        calculator.calculateTotal(order);
        repository.save(order);
        emailService.sendConfirmation(order);
        logger.logOrder(order);
    }
}
```

---

## Real-World Examples

### Example 1: Payment System

```java
// Following DRY, KISS, YAGNI
public class PaymentService {
    private PaymentValidator validator;
    private PaymentProcessor processor;
    
    public PaymentResult processPayment(PaymentRequest request) {
        validator.validate(request);  // DRY: Validation in one place
        return processor.process(request);  // KISS: Simple flow
        // YAGNI: Only current payment methods
    }
}
```

---

### Example 2: Notification System

```java
// Composition over Inheritance
public interface NotificationSender {
    void send(Notification notification);
}

public class NotificationService {
    private List<NotificationSender> senders;  // Composition
    
    public void send(Notification notification) {
        for (NotificationSender sender : senders) {
            sender.send(notification);
        }
    }
}
```

---

## Summary

**Key Principles:**
1. ✅ **DRY:** Don't repeat code
2. ✅ **KISS:** Keep it simple
3. ✅ **YAGNI:** Don't add until needed
4. ✅ **Composition over Inheritance:** Favor composition
5. ✅ **Separation of Concerns:** One concern per module

**Next Steps:**
- Learn [Error Handling](./20_ERROR_HANDLING.md)
- Review [SOLID Principles](./03_SOLID_PRINCIPLES.md)
- Practice [Best Practices](./23_LLD_BEST_PRACTICES.md)

---

**References:**
- [SOLID Principles](./03_SOLID_PRINCIPLES.md)
- [Class Design](./04_CLASS_DESIGN.md)

