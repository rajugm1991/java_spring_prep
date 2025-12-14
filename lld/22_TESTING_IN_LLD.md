# Testing in LLD

## Table of Contents
1. [Introduction](#introduction)
2. [Unit Testing](#unit-testing)
3. [Mocking](#mocking)
4. [Testability](#testability)
5. [Test-Driven Development (TDD)](#test-driven-development-tdd)
6. [Real-World Examples](#real-world-examples)

---

## Introduction

**Testing** is essential for ensuring code quality and reliability. Well-designed code is testable code.

---

## Unit Testing

### What is Unit Testing?

**Unit tests** test individual components in isolation.

### Example: Calculator

```java
public class Calculator {
    public int add(int a, int b) {
        return a + b;
    }
    
    public int divide(int a, int b) {
        if (b == 0) {
            throw new IllegalArgumentException("Division by zero");
        }
        return a / b;
    }
}

// Unit test
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

public class CalculatorTest {
    private Calculator calculator = new Calculator();
    
    @Test
    public void testAdd() {
        assertEquals(5, calculator.add(2, 3));
        assertEquals(0, calculator.add(-1, 1));
    }
    
    @Test
    public void testDivide() {
        assertEquals(2, calculator.divide(6, 3));
    }
    
    @Test
    public void testDivideByZero() {
        assertThrows(IllegalArgumentException.class, () -> {
            calculator.divide(5, 0);
        });
    }
}
```

---

## Mocking

### What is Mocking?

**Mocking** replaces dependencies with fake objects for testing.

### Example: Order Service

```java
public class OrderService {
    private PaymentService paymentService;
    private InventoryService inventoryService;
    
    public OrderService(PaymentService paymentService, 
                       InventoryService inventoryService) {
        this.paymentService = paymentService;
        this.inventoryService = inventoryService;
    }
    
    public Order processOrder(Order order) {
        PaymentResult payment = paymentService.processPayment(order);
        if (payment.isSuccess()) {
            inventoryService.updateInventory(order);
            order.confirm();
        }
        return order;
    }
}

// Test with mocks
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;
import static org.mockito.Mockito.*;

public class OrderServiceTest {
    @Mock
    private PaymentService paymentService;
    
    @Mock
    private InventoryService inventoryService;
    
    private OrderService orderService;
    
    @BeforeEach
    public void setUp() {
        MockitoAnnotations.openMocks(this);
        orderService = new OrderService(paymentService, inventoryService);
    }
    
    @Test
    public void testProcessOrder_Success() {
        Order order = new Order("123", 100.0);
        PaymentResult successResult = new PaymentResult(true, "Success");
        
        when(paymentService.processPayment(order)).thenReturn(successResult);
        
        Order result = orderService.processOrder(order);
        
        assertEquals(OrderStatus.CONFIRMED, result.getStatus());
        verify(inventoryService).updateInventory(order);
    }
    
    @Test
    public void testProcessOrder_PaymentFailed() {
        Order order = new Order("123", 100.0);
        PaymentResult failedResult = new PaymentResult(false, "Failed");
        
        when(paymentService.processPayment(order)).thenReturn(failedResult);
        
        Order result = orderService.processOrder(order);
        
        assertEquals(OrderStatus.PENDING, result.getStatus());
        verify(inventoryService, never()).updateInventory(order);
    }
}
```

---

## Testability

### Design for Testability

**Testable code:**
- Uses dependency injection
- Has clear responsibilities
- Avoids static dependencies
- Is loosely coupled

### Example: Testable vs Non-Testable

```java
// Bad: Hard to test
public class OrderService {
    private PaymentService paymentService = new PaymentService();
    private DatabaseConnection db = DatabaseConnection.getInstance();
    
    public void processOrder(Order order) {
        // Hard to mock dependencies
    }
}

// Good: Easy to test
public class OrderService {
    private PaymentService paymentService;
    private OrderRepository repository;
    
    public OrderService(PaymentService paymentService, 
                       OrderRepository repository) {
        this.paymentService = paymentService;
        this.repository = repository;
    }
    
    public void processOrder(Order order) {
        // Easy to inject mocks
    }
}
```

---

## Test-Driven Development (TDD)

### TDD Cycle

1. **Red:** Write failing test
2. **Green:** Write code to pass test
3. **Refactor:** Improve code

### Example: TDD for Calculator

```java
// Step 1: Write failing test
@Test
public void testMultiply() {
    assertEquals(6, calculator.multiply(2, 3));
}

// Step 2: Write code to pass
public int multiply(int a, int b) {
    return a * b;
}

// Step 3: Refactor if needed
```

---

## Real-World Examples

### Example 1: User Service Testing

```java
public class UserService {
    private UserRepository repository;
    private EmailService emailService;
    
    public UserService(UserRepository repository, EmailService emailService) {
        this.repository = repository;
        this.emailService = emailService;
    }
    
    public User createUser(UserRequest request) {
        validateRequest(request);
        User user = new User(request);
        repository.save(user);
        emailService.sendWelcomeEmail(user);
        return user;
    }
    
    private void validateRequest(UserRequest request) {
        if (request.getEmail() == null) {
            throw new ValidationException("Email required");
        }
    }
}

// Tests
public class UserServiceTest {
    @Mock
    private UserRepository repository;
    
    @Mock
    private EmailService emailService;
    
    private UserService userService;
    
    @Test
    public void testCreateUser_Success() {
        UserRequest request = new UserRequest("test@example.com", "John");
        User savedUser = new User(1L, "test@example.com", "John");
        
        when(repository.save(any(User.class))).thenReturn(savedUser);
        
        User result = userService.createUser(request);
        
        assertEquals("test@example.com", result.getEmail());
        verify(repository).save(any(User.class));
        verify(emailService).sendWelcomeEmail(savedUser);
    }
    
    @Test
    public void testCreateUser_InvalidEmail() {
        UserRequest request = new UserRequest(null, "John");
        
        assertThrows(ValidationException.class, () -> {
            userService.createUser(request);
        });
        
        verify(repository, never()).save(any());
    }
}
```

---

### Example 2: Payment Service Testing

```java
public class PaymentService {
    private PaymentGateway gateway;
    private TransactionRepository repository;
    
    public PaymentResult processPayment(PaymentRequest request) {
        try {
            PaymentResult result = gateway.charge(request);
            repository.saveTransaction(result);
            return result;
        } catch (PaymentException e) {
            logger.error("Payment failed", e);
            throw new PaymentProcessingException("Payment failed", e);
        }
    }
}

// Tests
public class PaymentServiceTest {
    @Mock
    private PaymentGateway gateway;
    
    @Mock
    private TransactionRepository repository;
    
    private PaymentService paymentService;
    
    @Test
    public void testProcessPayment_Success() {
        PaymentRequest request = new PaymentRequest(100.0);
        PaymentResult result = new PaymentResult(true, "Success");
        
        when(gateway.charge(request)).thenReturn(result);
        
        PaymentResult actual = paymentService.processPayment(request);
        
        assertTrue(actual.isSuccess());
        verify(repository).saveTransaction(result);
    }
    
    @Test
    public void testProcessPayment_Failure() {
        PaymentRequest request = new PaymentRequest(100.0);
        
        when(gateway.charge(request)).thenThrow(new PaymentException("Failed"));
        
        assertThrows(PaymentProcessingException.class, () -> {
            paymentService.processPayment(request);
        });
    }
}
```

---

## Summary

**Key Concepts:**
1. ✅ **Unit Testing:** Test individual components
2. ✅ **Mocking:** Replace dependencies with mocks
3. ✅ **Testability:** Design code to be testable
4. ✅ **TDD:** Test-driven development
5. ✅ **Best Practices:** Clear tests, good coverage

**Next Steps:**
- Learn [Best Practices](./23_LLD_BEST_PRACTICES.md)
- Review [Error Handling](./20_ERROR_HANDLING.md)
- Practice [Real-World Problems](./12_PARKING_LOT_SYSTEM.md)

---

**References:**
- [Method Design](./07_METHOD_DESIGN.md)
- [Best Practices](./23_LLD_BEST_PRACTICES.md)

