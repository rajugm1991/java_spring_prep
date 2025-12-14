# Introduction to Low-Level Design (LLD)

## What is Low-Level Design?

**Low-Level Design (LLD)** is the process of translating abstract ideas into concrete implementation. It's where you translate **High-Level Design (HLD)** into detailed class diagrams, interfaces, object relationships, and design patterns.

### The "How" vs "What"

```
High-Level Design (HLD)          Low-Level Design (LLD)
──────────────────────          ──────────────────────
"What"                            "How"
System Architecture               Class Structure
Services & Components             Classes & Interfaces
Data Flow                         Method Signatures
Infrastructure                    Object Relationships
```

**Example:**

**HLD says:** "We need a Notification Service that sends emails, SMS, and push notifications."

**LLD says:** 
```java
// Interface defining the contract
public interface NotificationSender {
    void send(Notification notification);
}

// Concrete implementations
public class EmailSender implements NotificationSender { ... }
public class SmsSender implements NotificationSender { ... }
public class PushNotificationSender implements NotificationSender { ... }

// Manager class that orchestrates
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

## Why LLD Matters

### 1. **Bridges Idea and Execution**

LLD turns broad architectural concepts into practical, working software components.

**Without LLD:**
```java
// Everything in one class - hard to maintain
public class OrderService {
    public void processOrder(Order order) {
        // Validate order
        // Check inventory
        // Process payment
        // Send notification
        // Update database
        // Generate invoice
        // ... 500 lines of code
    }
}
```

**With LLD:**
```java
// Separated concerns
public class OrderService {
    private OrderValidator validator;
    private InventoryService inventoryService;
    private PaymentProcessor paymentProcessor;
    private NotificationService notificationService;
    
    public void processOrder(Order order) {
        validator.validate(order);
        inventoryService.reserve(order);
        paymentProcessor.process(order);
        notificationService.sendConfirmation(order);
    }
}
```

### 2. **Maintainability**

Well-designed code is easy to read, debug, and extend.

**Bad Design:**
```java
// Hard to test, hard to change
public class PaymentService {
    public void processPayment(double amount, String cardNumber) {
        // Stripe API call hardcoded
        Stripe.apiKey = "sk_test_...";
        Charge charge = Charge.create(...);
        // What if we want to switch to Razorpay?
    }
}
```

**Good Design:**
```java
// Easy to test, easy to change
public interface PaymentProcessor {
    PaymentResult process(PaymentRequest request);
}

public class PaymentService {
    private PaymentProcessor processor;
    
    public PaymentService(PaymentProcessor processor) {
        this.processor = processor; // Can inject any implementation
    }
    
    public PaymentResult processPayment(PaymentRequest request) {
        return processor.process(request);
    }
}
```

### 3. **Testability**

Clean LLD leads to loosely coupled components, making unit testing straightforward.

```java
// Easy to test with mocks
@Test
public void testProcessOrder() {
    // Mock dependencies
    OrderValidator mockValidator = mock(OrderValidator.class);
    PaymentProcessor mockProcessor = mock(PaymentProcessor.class);
    
    // Create service with mocks
    OrderService service = new OrderService(mockValidator, mockProcessor);
    
    // Test
    service.processOrder(order);
    
    // Verify interactions
    verify(mockValidator).validate(order);
    verify(mockProcessor).process(order);
}
```

### 4. **Scalability**

While HLD focuses on infrastructure-level scalability, LLD ensures individual components can scale gracefully.

**Example: Sorting Module**
```java
// Works efficiently for 100 or 10,000 records
public class Sorter {
    private SortingStrategy strategy;
    
    public List<T> sort(List<T> items) {
        // Choose strategy based on size
        if (items.size() < 100) {
            strategy = new QuickSort();
        } else {
            strategy = new MergeSort();
        }
        return strategy.sort(items);
    }
}
```

### 5. **Collaboration**

LLD provides a clear blueprint for developers, enabling multiple engineers to work on the same module concurrently.

```java
// Clear contracts enable parallel development
public interface PaymentProcessor {
    PaymentResult process(PaymentRequest request);
}

// Developer A works on Stripe implementation
public class StripePaymentProcessor implements PaymentProcessor { ... }

// Developer B works on Razorpay implementation
public class RazorpayPaymentProcessor implements PaymentProcessor { ... }

// Both can work independently!
```

---

## Core Components of LLD

### 1. Classes and Objects

Identify the main entities and their roles:

**Questions to Ask:**
- What are the key classes?
- What are their responsibilities?
- What data do they hold (attributes)?
- What operations do they perform (methods)?

**Example: Food Delivery System**

```java
// Key classes
public class Customer {
    private String customerId;
    private String name;
    private Address address;
    
    public Order placeOrder(List<Item> items) { ... }
}

public class Restaurant {
    private String restaurantId;
    private String name;
    private Menu menu;
    
    public void prepareOrder(Order order) { ... }
}

public class Order {
    private String orderId;
    private String customerId;
    private List<Item> items;
    private BigDecimal totalAmount;
    private OrderStatus status;
    
    public void calculateTotal() { ... }
    public void addItem(Item item) { ... }
    public void updateStatus(OrderStatus status) { ... }
}

public class DeliveryAgent {
    private String agentId;
    private String name;
    private Location currentLocation;
    
    public void deliverOrder(Order order) { ... }
}
```

### 2. Interfaces and Abstractions

Interfaces define contracts between components, ensuring loose coupling.

**Example: Payment Processing**

```java
// Interface defines contract
public interface PaymentProcessor {
    PaymentResult processPayment(PaymentRequest request);
    boolean supports(PaymentMethod method);
}

// Multiple implementations
public class StripePaymentProcessor implements PaymentProcessor {
    public PaymentResult processPayment(PaymentRequest request) {
        // Stripe-specific implementation
    }
}

public class RazorpayPaymentProcessor implements PaymentProcessor {
    public PaymentResult processPayment(PaymentRequest request) {
        // Razorpay-specific implementation
    }
}

// Core logic depends on interface, not implementation
public class PaymentService {
    private PaymentProcessor processor;
    
    public PaymentService(PaymentProcessor processor) {
        this.processor = processor; // Dependency Injection
    }
    
    public PaymentResult process(PaymentRequest request) {
        return processor.processPayment(request);
    }
}
```

**Benefits:**
- Switch between Stripe and Razorpay without changing business logic
- Add new payment providers easily
- Test with mock implementations

### 3. Relationships Between Classes

Classes don't exist in isolation. LLD defines relationships clearly:

#### Association (Uses-A)
```java
public class Doctor {
    public void useStethoscope(Stethoscope stethoscope) {
        // Doctor uses stethoscope
    }
}
```

#### Aggregation (Weak "Has-A")
```java
public class Department {
    private List<Professor> professors; // Department has professors
    
    // Professors can exist without department
    // If department closes, professors still exist
}
```

#### Composition (Strong "Has-A")
```java
public class House {
    private List<Room> rooms; // House is composed of rooms
    
    // Rooms cannot exist without house
    // If house is demolished, rooms are destroyed
}
```

#### Inheritance (Is-A)
```java
public class Vehicle {
    public void start() { ... }
}

public class Car extends Vehicle {
    // Car is a Vehicle
    @Override
    public void start() {
        super.start();
        // Car-specific start logic
    }
}
```

### 4. Method Signatures

Well-designed method signatures are self-documenting:

**Bad:**
```java
void sendMsg(String str);
void proc(String id, int amt, boolean flag);
```

**Good:**
```java
void sendNotification(Notification notification);
void processPayment(String orderId, BigDecimal amount, boolean isRecurring);
```

**Consider:**
- Method name should describe what it does
- Parameters should be clear and typed
- Return type should be meaningful
- Should it throw exceptions?
- Is it synchronous or asynchronous?

### 5. Design Patterns

Design patterns provide proven solutions to common problems:

**Singleton:** One instance across system
```java
public class DatabaseConnection {
    private static DatabaseConnection instance;
    
    private DatabaseConnection() {}
    
    public static DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();
        }
        return instance;
    }
}
```

**Factory:** Delegate object creation
```java
public class PaymentProcessorFactory {
    public static PaymentProcessor create(String type) {
        switch (type) {
            case "stripe": return new StripePaymentProcessor();
            case "razorpay": return new RazorpayPaymentProcessor();
            default: throw new IllegalArgumentException();
        }
    }
}
```

**Strategy:** Interchangeable algorithms
```java
public interface SortingStrategy {
    List<Integer> sort(List<Integer> list);
}

public class QuickSort implements SortingStrategy { ... }
public class MergeSort implements SortingStrategy { ... }

public class Sorter {
    private SortingStrategy strategy;
    
    public void setStrategy(SortingStrategy strategy) {
        this.strategy = strategy;
    }
    
    public List<Integer> sort(List<Integer> list) {
        return strategy.sort(list);
    }
}
```

---

## LLD vs HLD

| Aspect | High-Level Design (HLD) | Low-Level Design (LLD) |
|--------|------------------------|------------------------|
| **Focus** | System architecture | Class structure |
| **Level** | Macro (big picture) | Micro (details) |
| **Questions** | What services? | What classes? |
| **Output** | Architecture diagrams | Class diagrams |
| **Concerns** | Scalability, Availability | Maintainability, Testability |
| **Example** | "We need a Payment Service" | "PaymentService uses PaymentProcessor interface" |

**Example: E-Commerce System**

**HLD:**
```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │
┌──────▼──────────┐
│  API Gateway    │
└──────┬──────────┘
       │
   ┌───┴───┬─────────┬──────────┐
   │       │         │          │
┌──▼──┐ ┌─▼──┐  ┌───▼──┐  ┌───▼──┐
│Order│ │Pay │  │Invent│  │Notif │
│Serv │ │Serv│  │Serv  │  │Serv  │
└─────┘ └────┘  └──────┘  └──────┘
```

**LLD:**
```java
// Classes and relationships
public class OrderService {
    private PaymentService paymentService;
    private InventoryService inventoryService;
    private NotificationService notificationService;
    
    public Order createOrder(OrderRequest request) {
        // Implementation
    }
}

public interface PaymentService {
    PaymentResult processPayment(PaymentRequest request);
}
```

---

## Importance in Interviews

### What Interviewers Evaluate

1. **Problem-Solving**: Can you break down complex problems?
2. **OOP Understanding**: Do you apply OOP principles correctly?
3. **Design Principles**: SOLID, DRY, KISS
4. **Design Patterns**: When and how to use patterns
5. **Code Quality**: Clean, readable, maintainable code
6. **Communication**: Can you explain your design choices?

### Interview Process

1. **Understand Requirements**: Ask clarifying questions
2. **Identify Entities**: What are the main classes?
3. **Define Relationships**: How do classes interact?
4. **Apply Patterns**: Which patterns fit?
5. **Code Implementation**: Write clean, working code
6. **Explain Trade-offs**: Why did you choose this design?

---

## Key Takeaways

1. **LLD is about "HOW"**: It translates abstract ideas into concrete implementation
2. **Focus on Design**: Not just working code, but well-designed code
3. **Apply Principles**: SOLID, OOP, Design Patterns
4. **Think Scalable**: Design for growth and change
5. **Communicate**: Explain your design choices clearly

---

## Next Steps

1. Master [OOP Principles](./02_OOP_PRINCIPLES_FOR_LLD.md)
2. Learn [SOLID Principles](./03_SOLID_PRINCIPLES.md)
3. Practice [Class Design](./04_CLASS_DESIGN.md)
4. Study [Design Patterns](./08_CREATIONAL_PATTERNS.md)

---

**Remember:** Good LLD is not about writing code that works—it's about writing code that works well over time. 🚀

