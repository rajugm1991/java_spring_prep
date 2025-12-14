# Error Handling & Exceptions

## Table of Contents
1. [Introduction](#introduction)
2. [Exception Hierarchy](#exception-hierarchy)
3. [Checked vs Unchecked Exceptions](#checked-vs-unchecked-exceptions)
4. [Custom Exceptions](#custom-exceptions)
5. [Exception Handling Best Practices](#exception-handling-best-practices)
6. [Real-World Examples](#real-world-examples)

---

## Introduction

**Error Handling** is crucial for robust software. Proper exception handling makes code more reliable and maintainable.

---

## Exception Hierarchy

### Java Exception Hierarchy

```
Throwable
├── Error (unchecked)
│   ├── OutOfMemoryError
│   └── StackOverflowError
└── Exception
    ├── RuntimeException (unchecked)
    │   ├── IllegalArgumentException
    │   ├── NullPointerException
    │   └── IllegalStateException
    └── Checked Exceptions
        ├── IOException
        ├── SQLException
        └── Custom Exceptions
```

---

## Checked vs Unchecked Exceptions

### Checked Exceptions

**Must be handled or declared:**
```java
public void readFile(String path) throws IOException {
    // Must handle IOException
    FileReader reader = new FileReader(path);
}
```

**Usage:**
- Recoverable conditions
- External dependencies (I/O, network)
- Must be handled by caller

---

### Unchecked Exceptions

**Don't need to be declared:**
```java
public void divide(int a, int b) {
    if (b == 0) {
        throw new IllegalArgumentException("Division by zero");
    }
    // IllegalArgumentException is unchecked
}
```

**Usage:**
- Programming errors
- Invalid arguments
- Illegal state

---

## Custom Exceptions

### Creating Custom Exceptions

```java
// Custom checked exception
public class PaymentException extends Exception {
    public PaymentException(String message) {
        super(message);
    }
    
    public PaymentException(String message, Throwable cause) {
        super(message, cause);
    }
}

// Custom unchecked exception
public class InvalidOrderException extends RuntimeException {
    public InvalidOrderException(String message) {
        super(message);
    }
    
    public InvalidOrderException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

---

### Exception Hierarchy Example

```java
// Base exception
public class OrderException extends Exception {
    public OrderException(String message) {
        super(message);
    }
}

// Specific exceptions
public class InvalidOrderException extends OrderException {
    public InvalidOrderException(String message) {
        super(message);
    }
}

public class OrderNotFoundException extends OrderException {
    public OrderNotFoundException(String orderId) {
        super("Order not found: " + orderId);
    }
}

public class InsufficientStockException extends OrderException {
    public InsufficientStockException(String productId) {
        super("Insufficient stock for product: " + productId);
    }
}
```

---

## Exception Handling Best Practices

### 1. Use Specific Exceptions

```java
// Bad: Too generic
public void process() throws Exception { ... }

// Good: Specific
public void process() throws PaymentException, ValidationException { ... }
```

---

### 2. Don't Swallow Exceptions

```java
// Bad: Swallowed exception
try {
    processPayment();
} catch (Exception e) {
    // Silent failure - no logging, no handling
}

// Good: Proper handling
try {
    processPayment();
} catch (PaymentException e) {
    logger.error("Payment failed", e);
    throw new OrderProcessingException("Failed to process payment", e);
}
```

---

### 3. Fail Fast

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

### 4. Preserve Stack Trace

```java
// Bad: Loses stack trace
try {
    processPayment();
} catch (PaymentException e) {
    throw new OrderException("Payment failed");  // Stack trace lost
}

// Good: Preserves stack trace
try {
    processPayment();
} catch (PaymentException e) {
    throw new OrderException("Payment failed", e);  // Stack trace preserved
}
```

---

### 5. Use Finally for Cleanup

```java
public void processFile(String path) {
    FileReader reader = null;
    try {
        reader = new FileReader(path);
        // Process file
    } catch (IOException e) {
        logger.error("Error reading file", e);
    } finally {
        if (reader != null) {
            try {
                reader.close();
            } catch (IOException e) {
                logger.error("Error closing file", e);
            }
        }
    }
}

// Better: Try-with-resources
public void processFile(String path) {
    try (FileReader reader = new FileReader(path)) {
        // Process file
    } catch (IOException e) {
        logger.error("Error reading file", e);
    }
    // Automatically closes
}
```

---

## Real-World Examples

### Example 1: Payment Processing

```java
public class PaymentService {
    public PaymentResult processPayment(PaymentRequest request) {
        try {
            validatePaymentRequest(request);
            PaymentResult result = paymentGateway.charge(request);
            return result;
        } catch (InvalidPaymentException e) {
            logger.error("Invalid payment request", e);
            return PaymentResult.failure("Invalid payment request");
        } catch (PaymentGatewayException e) {
            logger.error("Payment gateway error", e);
            throw new PaymentProcessingException("Payment processing failed", e);
        }
    }
    
    private void validatePaymentRequest(PaymentRequest request) {
        if (request == null) {
            throw new InvalidPaymentException("Payment request cannot be null");
        }
        if (request.getAmount() <= 0) {
            throw new InvalidPaymentException("Amount must be positive");
        }
    }
}
```

---

### Example 2: Order Processing

```java
public class OrderService {
    public Order createOrder(OrderRequest request) {
        try {
            validateOrderRequest(request);
            checkInventory(request);
            Order order = buildOrder(request);
            saveOrder(order);
            return order;
        } catch (ValidationException e) {
            logger.warn("Invalid order request", e);
            throw e;
        } catch (InsufficientStockException e) {
            logger.warn("Insufficient stock", e);
            throw e;
        } catch (DatabaseException e) {
            logger.error("Database error", e);
            throw new OrderCreationException("Failed to create order", e);
        }
    }
}
```

---

### Example 3: Retry Mechanism

```java
public class RetryableService {
    private static final int MAX_RETRIES = 3;
    
    public void executeWithRetry(Runnable task) {
        int attempts = 0;
        while (attempts < MAX_RETRIES) {
            try {
                task.run();
                return;
            } catch (TransientException e) {
                attempts++;
                if (attempts >= MAX_RETRIES) {
                    throw new MaxRetriesExceededException("Max retries exceeded", e);
                }
                waitBeforeRetry(attempts);
            } catch (Exception e) {
                throw e;  // Don't retry non-transient exceptions
            }
        }
    }
    
    private void waitBeforeRetry(int attempt) {
        try {
            Thread.sleep(1000 * attempt);  // Exponential backoff
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

---

## Summary

**Key Principles:**
1. ✅ **Use Specific Exceptions:** Not generic Exception
2. ✅ **Don't Swallow:** Always log or handle
3. ✅ **Fail Fast:** Validate early
4. ✅ **Preserve Stack Trace:** Include cause
5. ✅ **Cleanup Resources:** Use finally or try-with-resources

**Next Steps:**
- Learn [Concurrency](./21_CONCURRENCY_IN_LLD.md)
- Review [Best Practices](./23_LLD_BEST_PRACTICES.md)
- Practice [Testing](./22_TESTING_IN_LLD.md)

---

**References:**
- [Method Design](./07_METHOD_DESIGN.md)
- [Best Practices](./23_LLD_BEST_PRACTICES.md)

