# Interfaces and Abstractions

## Table of Contents
1. [Introduction](#introduction)
2. [What are Interfaces?](#what-are-interfaces)
3. [Why Use Interfaces?](#why-use-interfaces)
4. [Abstract Classes](#abstract-classes)
5. [Interface vs Abstract Class](#interface-vs-abstract-class)
6. [Dependency Inversion Principle](#dependency-inversion-principle)
7. [Real-World Examples](#real-world-examples)

---

## Introduction

**Interfaces** define contracts between components. They are critical for **loose coupling** and **flexibility**.

### Key Concept

**Program to interfaces, not implementations.**

```
Bad: Depend on concrete classes
Good: Depend on interfaces
```

---

## What are Interfaces?

**Interface** is a contract that defines what a class must do, not how.

### Interface Definition

```java
// Interface (Contract)
public interface PaymentProcessor {
    PaymentResult processPayment(PaymentRequest request);
    void refund(String paymentId);
}

// Implementation
public class StripePaymentProcessor implements PaymentProcessor {
    public PaymentResult processPayment(PaymentRequest request) {
        // Stripe-specific implementation
    }
    
    public void refund(String paymentId) {
        // Stripe-specific refund
    }
}
```

---

### Interface Characteristics

1. **No Implementation:** Only method signatures
2. **Multiple Inheritance:** Class can implement multiple interfaces
3. **Contract:** Must implement all methods
4. **Polymorphism:** Can use interface type

---

## Why Use Interfaces?

### 1. **Loose Coupling**

**Without Interface (Tight Coupling):**
```java
public class OrderService {
    private StripePaymentProcessor processor;  // Tight coupling
    
    public void processOrder(Order order) {
        processor.processPayment(order);  // Can't switch providers
    }
}
```

**With Interface (Loose Coupling):**
```java
public class OrderService {
    private PaymentProcessor processor;  // Loose coupling
    
    public OrderService(PaymentProcessor processor) {
        this.processor = processor;  // Can inject any implementation
    }
    
    public void processOrder(Order order) {
        processor.processPayment(order);  // Works with any provider
    }
}
```

---

### 2. **Flexibility**

**Easy to switch implementations:**
```java
// Switch from Stripe to Razorpay
PaymentProcessor processor = new RazorpayPaymentProcessor();
OrderService service = new OrderService(processor);
// No code changes needed!
```

---

### 3. **Testability**

**Easy to mock for testing:**
```java
// In tests
PaymentProcessor mockProcessor = mock(PaymentProcessor.class);
OrderService service = new OrderService(mockProcessor);
// Test without real payment processing
```

---

### 4. **Multiple Implementations**

**One interface, many implementations:**
```java
public interface NotificationSender {
    void send(Notification notification);
}

public class EmailSender implements NotificationSender { ... }
public class SmsSender implements NotificationSender { ... }
public class PushNotificationSender implements NotificationSender { ... }

// Use any implementation
List<NotificationSender> senders = Arrays.asList(
    new EmailSender(),
    new SmsSender(),
    new PushNotificationSender()
);
```

---

## Abstract Classes

**Abstract Class** is a class that cannot be instantiated and may contain abstract methods.

### Abstract Class Example

```java
public abstract class Animal {
    protected String name;
    
    // Concrete method
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    // Abstract method (must be implemented by subclasses)
    public abstract void makeSound();
}

public class Dog extends Animal {
    public Dog(String name) {
        this.name = name;
    }
    
    @Override
    public void makeSound() {
        System.out.println("Woof!");
    }
}
```

---

## Interface vs Abstract Class

### Comparison

| Aspect | Interface | Abstract Class |
|--------|-----------|----------------|
| **Methods** | Only abstract (Java 8+: default methods) | Abstract + concrete |
| **Variables** | Only constants (public static final) | Any variables |
| **Inheritance** | Multiple interfaces | Single class |
| **Constructor** | No | Yes |
| **Access Modifiers** | Public only | Any |

---

### When to Use Interface

**Use Interface when:**
- Multiple unrelated classes need same behavior
- You want to define a contract
- You need multiple inheritance
- Implementation varies significantly

**Example:**
```java
// Different classes, same behavior
public class Car implements Drivable { ... }
public class Boat implements Drivable { ... }
public class Plane implements Drivable { ... }
```

---

### When to Use Abstract Class

**Use Abstract Class when:**
- Related classes share common code
- You want to provide default implementation
- You need to share state
- Single inheritance is sufficient

**Example:**
```java
// Related classes with shared code
public abstract class Vehicle {
    protected String brand;  // Shared state
    
    public void start() {  // Shared implementation
        System.out.println("Starting " + brand);
    }
    
    public abstract void drive();  // Different implementation
}

public class Car extends Vehicle { ... }
public class Truck extends Vehicle { ... }
```

---

## Dependency Inversion Principle

**Depend on abstractions, not concretions.**

### Violation

```java
// High-level module depends on low-level module
public class OrderService {
    private StripePaymentProcessor processor;  // Depends on concrete class
    
    public OrderService() {
        this.processor = new StripePaymentProcessor();  // Tight coupling
    }
}
```

---

### Following DIP

```java
// High-level module depends on abstraction
public class OrderService {
    private PaymentProcessor processor;  // Depends on interface
    
    public OrderService(PaymentProcessor processor) {
        this.processor = processor;  // Dependency injection
    }
}

// Low-level module implements abstraction
public class StripePaymentProcessor implements PaymentProcessor { ... }
```

---

## Real-World Examples

### Example 1: Payment System

```java
// Interface
public interface PaymentProcessor {
    PaymentResult processPayment(PaymentRequest request);
    void refund(String paymentId);
}

// Implementations
public class StripePaymentProcessor implements PaymentProcessor {
    public PaymentResult processPayment(PaymentRequest request) {
        // Stripe API call
    }
}

public class RazorpayPaymentProcessor implements PaymentProcessor {
    public PaymentResult processPayment(PaymentRequest request) {
        // Razorpay API call
    }
}

// Usage
public class PaymentService {
    private PaymentProcessor processor;
    
    public PaymentService(PaymentProcessor processor) {
        this.processor = processor;  // Any implementation works
    }
}
```

---

### Example 2: Notification System

```java
// Interface
public interface NotificationSender {
    void send(Notification notification);
}

// Implementations
public class EmailSender implements NotificationSender {
    public void send(Notification notification) {
        // Send email
    }
}

public class SmsSender implements NotificationSender {
    public void send(Notification notification) {
        // Send SMS
    }
}

// Manager
public class NotificationManager {
    private List<NotificationSender> senders;
    
    public void sendNotification(Notification notification) {
        for (NotificationSender sender : senders) {
            sender.send(notification);
        }
    }
}
```

---

### Example 3: Data Access Layer

```java
// Interface
public interface UserRepository {
    User findById(Long id);
    void save(User user);
    void delete(Long id);
}

// Implementations
public class DatabaseUserRepository implements UserRepository {
    public User findById(Long id) {
        // Database query
    }
}

public class CacheUserRepository implements UserRepository {
    public User findById(Long id) {
        // Cache lookup
    }
}

// Service uses interface
public class UserService {
    private UserRepository repository;
    
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
}
```

---

## Best Practices

### 1. **Keep Interfaces Small**

**Bad:**
```java
public interface UserOperations {
    void createUser();
    void updateUser();
    void deleteUser();
    void sendEmail();
    void generateReport();
    void exportData();
}
```

**Good:**
```java
public interface UserRepository {
    void createUser();
    void updateUser();
    void deleteUser();
}

public interface EmailService {
    void sendEmail();
}

public interface ReportService {
    void generateReport();
    void exportData();
}
```

---

### 2. **Use Meaningful Names**

```java
// Bad
public interface IUser { ... }

// Good
public interface UserRepository { ... }
```

---

### 3. **Prefer Composition over Inheritance**

```java
// Bad: Inheritance
public class EmailNotification extends Notification { ... }

// Good: Composition with interface
public class EmailNotification implements NotificationSender { ... }
```

---

## Summary

**Key Takeaways:**
1. ✅ **Interfaces define contracts:** What, not how
2. ✅ **Loose coupling:** Depend on interfaces
3. ✅ **Flexibility:** Easy to switch implementations
4. ✅ **Testability:** Easy to mock
5. ✅ **Dependency Inversion:** Depend on abstractions

**Next Steps:**
- Learn [Method Design](./07_METHOD_DESIGN.md)
- Understand [Class Relationships](./05_CLASS_RELATIONSHIPS.md)
- Practice [Design Patterns](./08_CREATIONAL_PATTERNS.md)

---

**References:**
- [SOLID Principles](./03_SOLID_PRINCIPLES.md)
- [OOP Principles](./02_OOP_PRINCIPLES_FOR_LLD.md)

