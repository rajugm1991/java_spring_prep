# Method Design

## Table of Contents
1. [Introduction](#introduction)
2. [Method Signatures](#method-signatures)
3. [Parameters](#parameters)
4. [Return Types](#return-types)
5. [Exception Handling](#exception-handling)
6. [Method Naming](#method-naming)
7. [Method Length](#method-length)
8. [Real-World Examples](#real-world-examples)

---

## Introduction

**Method Design** is crucial for readable, maintainable, and testable code. Well-designed methods are the building blocks of good software.

### Principles of Good Method Design

1. **Single Responsibility:** One method, one purpose
2. **Clear Naming:** Method name describes what it does
3. **Appropriate Parameters:** Not too many, not too few
4. **Meaningful Return:** Return useful information
5. **Error Handling:** Handle or declare exceptions

---

## Method Signatures

**Method Signature** includes:
- Method name
- Parameters
- Return type
- Access modifier
- Exceptions

### Example

```java
public PaymentResult processPayment(PaymentRequest request) throws PaymentException
│        │            │              │                    │
│        │            │              │                    └─ Exception
│        │            │              └─ Parameter
│        │            └─ Method name
│        └─ Return type
└─ Access modifier
```

---

## Parameters

### Parameter Count

**Rule of Thumb:** Maximum 3-4 parameters

**Bad (Too Many Parameters):**
```java
public void createUser(String firstName, String lastName, String email, 
                       String phone, String address, String city, 
                       String state, String zipCode) {
    // Hard to remember order
    // Easy to make mistakes
}
```

**Good (Use Object):**
```java
public void createUser(UserRequest request) {
    // Clear, easy to use
    // Can add fields without breaking signature
}

// UserRequest class
public class UserRequest {
    private String firstName;
    private String lastName;
    private String email;
    // ... other fields
}
```

---

### Parameter Types

**Prefer specific types:**
```java
// Bad: Too generic
public void process(Object data) { ... }

// Good: Specific type
public void process(PaymentRequest request) { ... }
```

---

### Optional Parameters

**Use Builder Pattern or Optional:**
```java
// Bad: Multiple overloaded methods
public void sendEmail(String to) { ... }
public void sendEmail(String to, String subject) { ... }
public void sendEmail(String to, String subject, String body) { ... }

// Good: Builder pattern
Email.builder()
    .to("user@example.com")
    .subject("Hello")
    .body("Message")
    .send();
```

---

## Return Types

### Return Meaningful Values

**Bad:**
```java
public void processOrder(Order order) {
    // Process order
    // No return value - can't check if successful
}
```

**Good:**
```java
public boolean processOrder(Order order) {
    // Process order
    return success;  // Indicates success/failure
}

// Or better
public ProcessResult processOrder(Order order) {
    // Process order
    return new ProcessResult(success, message, orderId);
}
```

---

### Return Types to Avoid

**1. Returning null:**
```java
// Bad
public User findUser(Long id) {
    if (exists(id)) {
        return user;
    }
    return null;  // Null pointer risk
}

// Good
public Optional<User> findUser(Long id) {
    if (exists(id)) {
        return Optional.of(user);
    }
    return Optional.empty();
}
```

---

**2. Returning void when value needed:**
```java
// Bad
public void calculateTotal(Order order) {
    order.setTotal(calculate(order));  // Side effect
}

// Good
public double calculateTotal(Order order) {
    return calculate(order);  // Return value
}
```

---

## Exception Handling

### Checked vs Unchecked Exceptions

**Checked Exceptions:** Must be handled or declared
```java
public void readFile(String path) throws IOException {
    // Must handle IOException
}
```

**Unchecked Exceptions:** Don't need to be declared
```java
public void divide(int a, int b) {
    if (b == 0) {
        throw new IllegalArgumentException("Division by zero");
    }
    // IllegalArgumentException is unchecked
}
```

---

### Exception Best Practices

**1. Use Specific Exceptions:**
```java
// Bad: Too generic
public void process() throws Exception { ... }

// Good: Specific
public void process() throws PaymentException, ValidationException { ... }
```

---

**2. Don't Swallow Exceptions:**
```java
// Bad
try {
    processPayment();
} catch (Exception e) {
    // Swallowed - no logging, no handling
}

// Good
try {
    processPayment();
} catch (PaymentException e) {
    logger.error("Payment failed", e);
    throw new OrderProcessingException("Failed to process payment", e);
}
```

---

**3. Fail Fast:**
```java
public void processOrder(Order order) {
    if (order == null) {
        throw new IllegalArgumentException("Order cannot be null");
    }
    if (order.getItems().isEmpty()) {
        throw new IllegalArgumentException("Order must have items");
    }
    // Process order
}
```

---

## Method Naming

### Naming Conventions

**Use verbs:**
```java
// Good
public void sendEmail() { ... }
public User findUser() { ... }
public boolean isValid() { ... }
public double calculateTotal() { ... }
```

**Avoid:**
```java
// Bad
public void email() { ... }  // Not clear
public void user() { ... }   // Not a verb
public void data() { ... }   // Not descriptive
```

---

### Common Naming Patterns

| Pattern | Example | Use Case |
|---------|---------|----------|
| `get*` | `getUser()` | Retrieve value |
| `set*` | `setName()` | Set value |
| `is*` | `isValid()` | Boolean check |
| `has*` | `hasPermission()` | Boolean check |
| `create*` | `createOrder()` | Create new |
| `update*` | `updateUser()` | Update existing |
| `delete*` | `deleteUser()` | Remove |
| `find*` | `findUser()` | Search |
| `calculate*` | `calculateTotal()` | Compute value |
| `process*` | `processPayment()` | Process operation |

---

## Method Length

### Keep Methods Short

**Rule of Thumb:** 20-30 lines maximum

**Bad (Too Long):**
```java
public void processOrder(Order order) {
    // 100 lines of code
    // Hard to understand
    // Hard to test
    // Hard to maintain
}
```

**Good (Short and Focused):**
```java
public void processOrder(Order order) {
    validateOrder(order);
    calculateTotal(order);
    applyDiscount(order);
    processPayment(order);
    updateInventory(order);
    sendConfirmation(order);
}
```

---

### Extract Methods

**Break long methods into smaller ones:**
```java
// Before: Long method
public void processOrder(Order order) {
    // Validation logic (20 lines)
    // Calculation logic (30 lines)
    // Payment logic (40 lines)
    // Inventory logic (25 lines)
    // Notification logic (15 lines)
}

// After: Extracted methods
public void processOrder(Order order) {
    validateOrder(order);
    calculateTotal(order);
    processPayment(order);
    updateInventory(order);
    sendConfirmation(order);
}

private void validateOrder(Order order) {
    // Validation logic
}

private void calculateTotal(Order order) {
    // Calculation logic
}
// ... etc
```

---

## Real-World Examples

### Example 1: E-Commerce Order Processing

```java
public class OrderService {
    
    // Good: Clear name, specific parameters, meaningful return
    public OrderResult processOrder(OrderRequest request) {
        Order order = createOrder(request);
        validateOrder(order);
        calculateTotal(order);
        processPayment(order);
        updateInventory(order);
        sendConfirmation(order);
        return new OrderResult(order.getId(), OrderStatus.CONFIRMED);
    }
    
    // Good: Single responsibility, clear name
    private void validateOrder(Order order) {
        if (order.getItems().isEmpty()) {
            throw new InvalidOrderException("Order must have items");
        }
        if (order.getTotal() <= 0) {
            throw new InvalidOrderException("Order total must be positive");
        }
    }
    
    // Good: Returns value, no side effects
    private double calculateTotal(Order order) {
        double subtotal = order.getItems().stream()
            .mapToDouble(item -> item.getPrice() * item.getQuantity())
            .sum();
        double tax = subtotal * TAX_RATE;
        return subtotal + tax;
    }
}
```

---

### Example 2: User Management

```java
public class UserService {
    
    // Good: Optional return type, handles null
    public Optional<User> findUserById(Long id) {
        if (id == null || id <= 0) {
            return Optional.empty();
        }
        return userRepository.findById(id);
    }
    
    // Good: Clear name, validation, exception handling
    public User createUser(UserRequest request) {
        validateUserRequest(request);
        if (userRepository.existsByEmail(request.getEmail())) {
            throw new DuplicateUserException("Email already exists");
        }
        User user = new User(request);
        return userRepository.save(user);
    }
    
    // Good: Single responsibility
    private void validateUserRequest(UserRequest request) {
        if (request.getEmail() == null || !isValidEmail(request.getEmail())) {
            throw new ValidationException("Invalid email");
        }
        if (request.getPassword() == null || request.getPassword().length() < 8) {
            throw new ValidationException("Password must be at least 8 characters");
        }
    }
}
```

---

### Example 3: Payment Processing

```java
public class PaymentService {
    
    // Good: Returns result object with details
    public PaymentResult processPayment(PaymentRequest request) {
        validatePaymentRequest(request);
        Payment payment = createPayment(request);
        PaymentResult result = paymentProcessor.process(payment);
        if (result.isSuccess()) {
            updateOrderStatus(request.getOrderId(), OrderStatus.PAID);
        }
        return result;
    }
    
    // Good: Specific exception, clear validation
    private void validatePaymentRequest(PaymentRequest request) {
        if (request.getAmount() <= 0) {
            throw new InvalidPaymentException("Amount must be positive");
        }
        if (request.getPaymentMethod() == null) {
            throw new InvalidPaymentException("Payment method required");
        }
    }
}
```

---

## Method Design Checklist

### Before Writing a Method

- [ ] What is the single purpose?
- [ ] Is the name clear and descriptive?
- [ ] Are parameters appropriate (not too many)?
- [ ] What should it return?
- [ ] What exceptions might it throw?

### After Writing a Method

- [ ] Is it short enough (< 30 lines)?
- [ ] Does it do one thing?
- [ ] Is it testable?
- [ ] Are exceptions handled properly?
- [ ] Is the name self-documenting?

---

## Summary

**Key Principles:**
1. ✅ **Single Responsibility:** One method, one purpose
2. ✅ **Clear Naming:** Verb-based, descriptive
3. ✅ **Appropriate Parameters:** 3-4 max, use objects if more
4. ✅ **Meaningful Returns:** Return useful information
5. ✅ **Exception Handling:** Handle or declare, be specific
6. ✅ **Keep Short:** 20-30 lines maximum

**Next Steps:**
- Learn [Design Patterns](./08_CREATIONAL_PATTERNS.md)
- Understand [Class Relationships](./05_CLASS_RELATIONSHIPS.md)
- Practice [Real-World Problems](./12_PARKING_LOT_SYSTEM.md)

---

**References:**
- [Class Design](./04_CLASS_DESIGN.md)
- [SOLID Principles](./03_SOLID_PRINCIPLES.md)

