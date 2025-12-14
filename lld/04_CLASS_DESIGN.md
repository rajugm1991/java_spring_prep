# Class Design

## Table of Contents
1. [Introduction](#introduction)
2. [Classes and Objects](#classes-and-objects)
3. [Attributes (Fields)](#attributes-fields)
4. [Methods](#methods)
5. [Responsibilities](#responsibilities)
6. [Class Design Principles](#class-design-principles)
7. [Real-World Examples](#real-world-examples)

---

## Introduction

**Class Design** is the foundation of Low-Level Design. A well-designed class has clear responsibilities, appropriate attributes, and well-defined methods.

### What Makes a Good Class?

1. **Single Responsibility:** Does one thing well
2. **Clear Interface:** Easy to understand and use
3. **Encapsulation:** Hides implementation details
4. **Cohesion:** All parts work together
5. **Low Coupling:** Minimal dependencies

---

## Classes and Objects

### Class vs Object

**Class:** Blueprint/template
**Object:** Instance of a class

**Example:**
```java
// Class (Blueprint)
public class Car {
    private String brand;
    private String model;
    
    public void start() { ... }
}

// Object (Instance)
Car myCar = new Car();  // myCar is an object
```

---

### Identifying Classes

**Ask yourself:**
- What are the main entities?
- What are their responsibilities?
- What data do they hold?
- What operations do they perform?

**Example: E-Commerce System**

```
Main Entities:
- User (customer)
- Product
- Order
- Payment
- Cart
```

---

## Attributes (Fields)

**Attributes** store the state of an object.

### Types of Attributes

#### 1. **Instance Variables**

**Belong to each object:**
```java
public class User {
    private String name;      // Each user has own name
    private String email;     // Each user has own email
    private Long userId;      // Each user has own ID
}
```

---

#### 2. **Class Variables (Static)**

**Shared by all objects:**
```java
public class User {
    private static int totalUsers = 0;  // Shared by all users
    
    public User() {
        totalUsers++;  // Increment for all users
    }
}
```

---

#### 3. **Constants**

**Immutable values:**
```java
public class Order {
    private static final double TAX_RATE = 0.10;  // Constant
    private static final int MAX_ITEMS = 100;      // Constant
}
```

---

### Attribute Design Principles

1. **Encapsulation:** Make private, provide getters/setters
2. **Initialization:** Initialize in constructor
3. **Validation:** Validate in setters
4. **Immutability:** Use `final` when possible

**Example:**
```java
public class User {
    private final Long userId;  // Immutable
    private String name;
    private String email;
    
    public User(Long userId, String name, String email) {
        this.userId = userId;
        this.setName(name);  // Validation in setter
        this.setEmail(email);
    }
    
    public void setEmail(String email) {
        if (email == null || !email.contains("@")) {
            throw new IllegalArgumentException("Invalid email");
        }
        this.email = email;
    }
}
```

---

## Methods

**Methods** define the behavior of a class.

### Types of Methods

#### 1. **Instance Methods**

**Operate on object state:**
```java
public class Order {
    private double total;
    
    public void addItem(Item item) {
        this.total += item.getPrice();
    }
}
```

---

#### 2. **Class Methods (Static)**

**Don't need object instance:**
```java
public class MathUtils {
    public static double calculateTax(double amount) {
        return amount * 0.10;
    }
}

// Usage: MathUtils.calculateTax(100)
```

---

#### 3. **Getter/Setter Methods**

**Access/modify attributes:**
```java
public class User {
    private String name;
    
    public String getName() {  // Getter
        return name;
    }
    
    public void setName(String name) {  // Setter
        this.name = name;
    }
}
```

---

### Method Design Principles

1. **Single Responsibility:** One method, one purpose
2. **Clear Naming:** Method name describes what it does
3. **Appropriate Parameters:** Not too many, not too few
4. **Return Type:** Return meaningful values
5. **Exception Handling:** Handle or declare exceptions

**Example:**
```java
// Bad: Too many responsibilities
public void processOrder(Order order) {
    validateOrder(order);
    calculateTotal(order);
    applyDiscount(order);
    processPayment(order);
    sendConfirmation(order);
    updateInventory(order);
}

// Good: Single responsibility
public void processOrder(Order order) {
    orderValidator.validate(order);
    orderCalculator.calculateTotal(order);
    discountService.applyDiscount(order);
    paymentService.processPayment(order);
    notificationService.sendConfirmation(order);
    inventoryService.updateInventory(order);
}
```

---

## Responsibilities

**Each class should have a single, well-defined responsibility.**

### Single Responsibility Principle (SRP)

**A class should have only one reason to change.**

**Example:**

**Bad (Multiple Responsibilities):**
```java
public class User {
    private String name;
    private String email;
    
    // User management
    public void createUser() { ... }
    public void updateUser() { ... }
    
    // Email sending (wrong responsibility)
    public void sendEmail() { ... }
    
    // Database operations (wrong responsibility)
    public void saveToDatabase() { ... }
}
```

**Good (Single Responsibility):**
```java
// User class - only user data
public class User {
    private String name;
    private String email;
    
    // Getters and setters only
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
}

// Email service - handles emails
public class EmailService {
    public void sendEmail(User user, String message) { ... }
}

// User repository - handles persistence
public class UserRepository {
    public void save(User user) { ... }
}
```

---

## Class Design Principles

### 1. **Cohesion**

**All parts of a class work together toward a single goal.**

**High Cohesion (Good):**
```java
public class ShoppingCart {
    private List<Item> items;
    
    public void addItem(Item item) { ... }
    public void removeItem(Item item) { ... }
    public double calculateTotal() { ... }
    public void clear() { ... }
}
// All methods related to shopping cart
```

**Low Cohesion (Bad):**
```java
public class Utility {
    public void sendEmail() { ... }
    public void calculateTax() { ... }
    public void formatDate() { ... }
    public void validatePassword() { ... }
}
// Methods have nothing in common
```

---

### 2. **Coupling**

**Minimize dependencies between classes.**

**Low Coupling (Good):**
```java
public class OrderService {
    private PaymentProcessor paymentProcessor;  // Interface, not concrete class
    
    public OrderService(PaymentProcessor processor) {
        this.paymentProcessor = processor;  // Dependency injection
    }
}
```

**High Coupling (Bad):**
```java
public class OrderService {
    private StripePaymentProcessor processor;  // Concrete class
    
    public OrderService() {
        this.processor = new StripePaymentProcessor();  // Tight coupling
    }
}
```

---

### 3. **Encapsulation**

**Hide implementation details, expose only what's needed.**

**Example:**
```java
public class BankAccount {
    private double balance;  // Hidden
    
    public double getBalance() {  // Exposed
        return balance;
    }
    
    public void deposit(double amount) {  // Controlled access
        if (amount > 0) {
            balance += amount;
        }
    }
}
```

---

## Real-World Examples

### Example 1: E-Commerce Order System

```java
// Order class - represents an order
public class Order {
    private final String orderId;
    private final Long userId;
    private List<OrderItem> items;
    private OrderStatus status;
    private LocalDateTime createdAt;
    
    public Order(String orderId, Long userId) {
        this.orderId = orderId;
        this.userId = userId;
        this.items = new ArrayList<>();
        this.status = OrderStatus.PENDING;
        this.createdAt = LocalDateTime.now();
    }
    
    public void addItem(OrderItem item) {
        items.add(item);
    }
    
    public double calculateTotal() {
        return items.stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
    }
    
    public void confirm() {
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException("Order already processed");
        }
        this.status = OrderStatus.CONFIRMED;
    }
}
```

---

### Example 2: Library Management System

```java
// Book class
public class Book {
    private final String isbn;
    private String title;
    private String author;
    private boolean isAvailable;
    
    public Book(String isbn, String title, String author) {
        this.isbn = isbn;
        this.title = title;
        this.author = author;
        this.isAvailable = true;
    }
    
    public void borrow() {
        if (!isAvailable) {
            throw new IllegalStateException("Book not available");
        }
        this.isAvailable = false;
    }
    
    public void returnBook() {
        this.isAvailable = true;
    }
    
    public boolean isAvailable() {
        return isAvailable;
    }
}
```

---

### Example 3: Payment System

```java
// Payment class
public class Payment {
    private final String paymentId;
    private final double amount;
    private PaymentStatus status;
    private PaymentMethod method;
    
    public Payment(String paymentId, double amount, PaymentMethod method) {
        this.paymentId = paymentId;
        this.amount = amount;
        this.method = method;
        this.status = PaymentStatus.PENDING;
    }
    
    public void process() {
        if (status != PaymentStatus.PENDING) {
            throw new IllegalStateException("Payment already processed");
        }
        // Process payment logic
        this.status = PaymentStatus.COMPLETED;
    }
    
    public void fail() {
        this.status = PaymentStatus.FAILED;
    }
}
```

---

## Class Design Checklist

### Before Creating a Class

- [ ] What is the single responsibility?
- [ ] What attributes does it need?
- [ ] What methods should it expose?
- [ ] What are its dependencies?
- [ ] Is it cohesive?
- [ ] Is it loosely coupled?

### After Creating a Class

- [ ] Does it follow SRP?
- [ ] Are attributes properly encapsulated?
- [ ] Are method names clear?
- [ ] Are exceptions handled?
- [ ] Is it testable?

---

## Summary

**Key Principles:**
1. ✅ **Single Responsibility:** One class, one purpose
2. ✅ **Encapsulation:** Hide implementation, expose interface
3. ✅ **Cohesion:** All parts work together
4. ✅ **Low Coupling:** Minimal dependencies
5. ✅ **Clear Naming:** Self-documenting code

**Next Steps:**
- Learn [Class Relationships](./05_CLASS_RELATIONSHIPS.md)
- Understand [Interfaces and Abstractions](./06_INTERFACES_AND_ABSTRACTIONS.md)
- Practice [Real-World Problems](./12_PARKING_LOT_SYSTEM.md)

---

**References:**
- [OOP Principles](./02_OOP_PRINCIPLES_FOR_LLD.md)
- [SOLID Principles](./03_SOLID_PRINCIPLES.md)

