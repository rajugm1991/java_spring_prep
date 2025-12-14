# OOP Principles for Low-Level Design

## Overview

Object-Oriented Programming (OOP) is the foundation of Low-Level Design. Understanding and applying OOP principles correctly is crucial for designing maintainable, scalable systems.

---

## The Four Pillars of OOP

```
┌─────────────────────────────────────────────────────────┐
│              FOUR PILLARS OF OOP                        │
└─────────────────────────────────────────────────────────┘

1. ENCAPSULATION ── Data hiding + Methods
2. INHERITANCE   ── IS-A relationship
3. POLYMORPHISM  ── Many forms
4. ABSTRACTION   ── Hide complexity, show essentials
```

---

## 1. Encapsulation

**Definition:** Bundling data and methods that operate on the data within one unit (class), and restricting access to internal details.

### Key Concepts

- **Data Hiding**: Private fields prevent direct access
- **Public Methods**: Controlled access through methods
- **Getters/Setters**: Controlled modification of data

### Example: Bank Account

**Without Encapsulation (Bad):**
```java
public class BankAccount {
    public double balance; // Anyone can modify directly!
    
    public static void main(String[] args) {
        BankAccount account = new BankAccount();
        account.balance = 1000;
        account.balance = -500; // Oops! Negative balance allowed
    }
}
```

**With Encapsulation (Good):**
```java
public class BankAccount {
    private double balance; // Private - cannot be accessed directly
    
    public BankAccount(double initialBalance) {
        if (initialBalance < 0) {
            throw new IllegalArgumentException("Balance cannot be negative");
        }
        this.balance = initialBalance;
    }
    
    public double getBalance() {
        return balance;
    }
    
    public void deposit(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        balance += amount;
    }
    
    public void withdraw(double amount) {
        if (amount <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        if (amount > balance) {
            throw new InsufficientFundsException("Insufficient balance");
        }
        balance -= amount;
    }
}
```

### Benefits

1. **Data Integrity**: Prevents invalid states
2. **Controlled Access**: Can add validation logic
3. **Flexibility**: Can change internal implementation without affecting clients
4. **Security**: Sensitive data is protected

### LLD Application

```java
// Encapsulation in Order Management
public class Order {
    private String orderId;
    private OrderStatus status;
    private List<OrderItem> items;
    private BigDecimal totalAmount;
    
    // Private constructor - use factory method
    private Order(String orderId) {
        this.orderId = orderId;
        this.status = OrderStatus.PENDING;
        this.items = new ArrayList<>();
    }
    
    // Factory method with validation
    public static Order create(String orderId) {
        if (orderId == null || orderId.isEmpty()) {
            throw new IllegalArgumentException("Order ID cannot be empty");
        }
        return new Order(orderId);
    }
    
    // Controlled status changes
    public void confirm() {
        if (this.status != OrderStatus.PENDING) {
            throw new IllegalStateException("Only pending orders can be confirmed");
        }
        this.status = OrderStatus.CONFIRMED;
    }
    
    public void cancel() {
        if (this.status == OrderStatus.DELIVERED) {
            throw new IllegalStateException("Delivered orders cannot be cancelled");
        }
        this.status = OrderStatus.CANCELLED;
    }
    
    // Read-only access to items
    public List<OrderItem> getItems() {
        return Collections.unmodifiableList(items);
    }
    
    // Controlled item addition
    public void addItem(OrderItem item) {
        if (this.status != OrderStatus.PENDING) {
            throw new IllegalStateException("Cannot modify confirmed order");
        }
        items.add(item);
        recalculateTotal();
    }
    
    private void recalculateTotal() {
        totalAmount = items.stream()
            .map(OrderItem::getTotalPrice)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}
```

---

## 2. Inheritance

**Definition:** Mechanism where one class can inherit attributes and methods from another class, establishing an "IS-A" relationship.

### Key Concepts

- **Code Reuse**: Avoid duplicating code
- **Hierarchy**: Parent-child relationship
- **Method Overriding**: Child can override parent methods
- **Super Keyword**: Access parent class members

### Example: Vehicle Hierarchy

```java
// Base class
public class Vehicle {
    protected String brand;
    protected String model;
    protected int year;
    
    public Vehicle(String brand, String model, int year) {
        this.brand = brand;
        this.model = model;
        this.year = year;
    }
    
    public void start() {
        System.out.println("Vehicle is starting...");
    }
    
    public void stop() {
        System.out.println("Vehicle is stopping...");
    }
    
    public void displayInfo() {
        System.out.println("Brand: " + brand + ", Model: " + model + ", Year: " + year);
    }
}

// Child class
public class Car extends Vehicle {
    private int numberOfDoors;
    
    public Car(String brand, String model, int year, int numberOfDoors) {
        super(brand, model, year); // Call parent constructor
        this.numberOfDoors = numberOfDoors;
    }
    
    @Override
    public void start() {
        System.out.println("Car engine is starting...");
        super.start(); // Call parent method
    }
    
    public void honk() {
        System.out.println("Beep beep!");
    }
}

// Another child class
public class Motorcycle extends Vehicle {
    private boolean hasSidecar;
    
    public Motorcycle(String brand, String model, int year, boolean hasSidecar) {
        super(brand, model, year);
        this.hasSidecar = hasSidecar;
    }
    
    @Override
    public void start() {
        System.out.println("Motorcycle engine is revving...");
    }
}
```

### Types of Inheritance

#### Single Inheritance
```java
public class Animal { }
public class Dog extends Animal { } // Dog IS-A Animal
```

#### Multilevel Inheritance
```java
public class Animal { }
public class Mammal extends Animal { }
public class Dog extends Mammal { } // Dog IS-A Mammal IS-A Animal
```

#### Hierarchical Inheritance
```java
public class Animal { }
public class Dog extends Animal { }
public class Cat extends Animal { } // Both Dog and Cat are Animals
```

### LLD Application: Payment Processors

```java
// Base class
public abstract class PaymentProcessor {
    protected String processorId;
    protected PaymentConfig config;
    
    public PaymentProcessor(String processorId, PaymentConfig config) {
        this.processorId = processorId;
        this.config = config;
    }
    
    // Template method pattern
    public final PaymentResult processPayment(PaymentRequest request) {
        validateRequest(request);
        PaymentResult result = executePayment(request);
        logTransaction(request, result);
        return result;
    }
    
    protected abstract void validateRequest(PaymentRequest request);
    protected abstract PaymentResult executePayment(PaymentRequest request);
    
    protected void logTransaction(PaymentRequest request, PaymentResult result) {
        // Common logging logic
        System.out.println("Transaction logged: " + result.getTransactionId());
    }
}

// Concrete implementations
public class StripePaymentProcessor extends PaymentProcessor {
    public StripePaymentProcessor(PaymentConfig config) {
        super("stripe", config);
    }
    
    @Override
    protected void validateRequest(PaymentRequest request) {
        // Stripe-specific validation
        if (request.getAmount().compareTo(new BigDecimal("0.50")) < 0) {
            throw new IllegalArgumentException("Stripe minimum amount is $0.50");
        }
    }
    
    @Override
    protected PaymentResult executePayment(PaymentRequest request) {
        // Stripe API call
        return new PaymentResult("stripe_txn_123", PaymentStatus.SUCCESS);
    }
}

public class RazorpayPaymentProcessor extends PaymentProcessor {
    public RazorpayPaymentProcessor(PaymentConfig config) {
        super("razorpay", config);
    }
    
    @Override
    protected void validateRequest(PaymentRequest request) {
        // Razorpay-specific validation
        if (request.getAmount().compareTo(new BigDecimal("1.00")) < 0) {
            throw new IllegalArgumentException("Razorpay minimum amount is ₹1.00");
        }
    }
    
    @Override
    protected PaymentResult executePayment(PaymentRequest request) {
        // Razorpay API call
        return new PaymentResult("razorpay_txn_456", PaymentStatus.SUCCESS);
    }
}
```

### When to Use Inheritance

✅ **Use Inheritance When:**
- There's a clear "IS-A" relationship
- You need code reuse
- Child classes share common behavior
- You want polymorphic behavior

❌ **Avoid Inheritance When:**
- Relationship is "HAS-A" (use composition)
- You need to inherit from multiple classes (Java doesn't support multiple inheritance)
- Child class doesn't truly "IS-A" parent

---

## 3. Polymorphism

**Definition:** Ability to process objects differently based on their data type or class. "One interface, many implementations."

### Types of Polymorphism

#### 1. Compile-Time Polymorphism (Method Overloading)

```java
public class Calculator {
    // Same method name, different parameters
    public int add(int a, int b) {
        return a + b;
    }
    
    public double add(double a, double b) {
        return a + b;
    }
    
    public int add(int a, int b, int c) {
        return a + b + c;
    }
}

// Usage
Calculator calc = new Calculator();
calc.add(5, 10);        // Calls first method
calc.add(5.5, 10.5);   // Calls second method
calc.add(5, 10, 15);   // Calls third method
```

#### 2. Runtime Polymorphism (Method Overriding)

```java
// Base class
public class Shape {
    public void draw() {
        System.out.println("Drawing a shape");
    }
    
    public double calculateArea() {
        return 0;
    }
}

// Child classes
public class Circle extends Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing a circle with radius " + radius);
    }
    
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

public class Rectangle extends Shape {
    private double width;
    private double height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing a rectangle " + width + "x" + height);
    }
    
    @Override
    public double calculateArea() {
        return width * height;
    }
}

// Polymorphic usage
public class ShapeRenderer {
    public void render(List<Shape> shapes) {
        for (Shape shape : shapes) {
            shape.draw(); // Calls appropriate draw() method
            System.out.println("Area: " + shape.calculateArea());
        }
    }
}

// Usage
List<Shape> shapes = Arrays.asList(
    new Circle(5),
    new Rectangle(4, 6),
    new Circle(3)
);

ShapeRenderer renderer = new ShapeRenderer();
renderer.render(shapes);
// Output:
// Drawing a circle with radius 5.0
// Area: 78.53981633974483
// Drawing a rectangle 4.0x6.0
// Area: 24.0
// Drawing a circle with radius 3.0
// Area: 28.274333882308138
```

### LLD Application: Notification System

```java
// Interface (contract)
public interface NotificationSender {
    void send(Notification notification);
    boolean supports(NotificationType type);
}

// Implementations
public class EmailSender implements NotificationSender {
    @Override
    public void send(Notification notification) {
        System.out.println("Sending email to: " + notification.getRecipient());
        // Email sending logic
    }
    
    @Override
    public boolean supports(NotificationType type) {
        return type == NotificationType.EMAIL;
    }
}

public class SmsSender implements NotificationSender {
    @Override
    public void send(Notification notification) {
        System.out.println("Sending SMS to: " + notification.getRecipient());
        // SMS sending logic
    }
    
    @Override
    public boolean supports(NotificationType type) {
        return type == NotificationType.SMS;
    }
}

public class PushNotificationSender implements NotificationSender {
    @Override
    public void send(Notification notification) {
        System.out.println("Sending push notification to: " + notification.getRecipient());
        // Push notification logic
    }
    
    @Override
    public boolean supports(NotificationType type) {
        return type == NotificationType.PUSH;
    }
}

// Polymorphic usage
public class NotificationService {
    private List<NotificationSender> senders;
    
    public NotificationService(List<NotificationSender> senders) {
        this.senders = senders;
    }
    
    public void sendNotification(Notification notification) {
        for (NotificationSender sender : senders) {
            if (sender.supports(notification.getType())) {
                sender.send(notification); // Polymorphic call
                break;
            }
        }
    }
}

// Usage
List<NotificationSender> senders = Arrays.asList(
    new EmailSender(),
    new SmsSender(),
    new PushNotificationSender()
);

NotificationService service = new NotificationService(senders);
service.sendNotification(new Notification(NotificationType.EMAIL, "user@example.com", "Hello!"));
```

### Benefits

1. **Flexibility**: Easy to add new types without changing existing code
2. **Code Reuse**: Common interface, different implementations
3. **Maintainability**: Changes to one implementation don't affect others
4. **Extensibility**: Add new implementations easily

---

## 4. Abstraction

**Definition:** Hiding complex implementation details and showing only the necessary features of an object.

### Types of Abstraction

#### 1. Abstract Classes

```java
// Abstract class - cannot be instantiated
public abstract class Animal {
    protected String name;
    
    public Animal(String name) {
        this.name = name;
    }
    
    // Concrete method
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    // Abstract method - must be implemented by child classes
    public abstract void makeSound();
    
    public abstract void move();
}

// Concrete implementations
public class Dog extends Animal {
    public Dog(String name) {
        super(name);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " barks: Woof! Woof!");
    }
    
    @Override
    public void move() {
        System.out.println(name + " runs on four legs");
    }
}

public class Bird extends Animal {
    public Bird(String name) {
        super(name);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " chirps: Tweet! Tweet!");
    }
    
    @Override
    public void move() {
        System.out.println(name + " flies in the sky");
    }
}
```

#### 2. Interfaces

```java
// Interface - pure abstraction
public interface PaymentProcessor {
    PaymentResult processPayment(PaymentRequest request);
    void refund(String transactionId, BigDecimal amount);
    boolean isAvailable();
}

// Implementation
public class StripePaymentProcessor implements PaymentProcessor {
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        // Implementation details hidden
        return new PaymentResult("stripe_123", PaymentStatus.SUCCESS);
    }
    
    @Override
    public void refund(String transactionId, BigDecimal amount) {
        // Implementation details hidden
    }
    
    @Override
    public boolean isAvailable() {
        // Implementation details hidden
        return true;
    }
}
```

### LLD Application: Database Access

```java
// Abstraction - hide database implementation details
public interface UserRepository {
    User findById(String userId);
    List<User> findAll();
    void save(User user);
    void delete(String userId);
}

// Implementation - details hidden from clients
public class JdbcUserRepository implements UserRepository {
    private DataSource dataSource;
    
    @Override
    public User findById(String userId) {
        // JDBC implementation details hidden
        try (Connection conn = dataSource.getConnection();
             PreparedStatement stmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?")) {
            stmt.setString(1, userId);
            ResultSet rs = stmt.executeQuery();
            // Map result set to User object
            return mapToUser(rs);
        } catch (SQLException e) {
            throw new RepositoryException("Failed to find user", e);
        }
    }
    
    // Other methods...
}

// Client code - doesn't know about JDBC
public class UserService {
    private UserRepository repository; // Depends on abstraction
    
    public UserService(UserRepository repository) {
        this.repository = repository;
    }
    
    public User getUser(String userId) {
        return repository.findById(userId); // Uses abstraction
    }
}
```

### Benefits

1. **Simplification**: Hide complexity from clients
2. **Flexibility**: Can change implementation without affecting clients
3. **Testability**: Easy to mock interfaces for testing
4. **Loose Coupling**: Clients depend on abstractions, not concrete classes

---

## Combining All Four Pillars

### Example: E-Commerce Order System

```java
// ABSTRACTION: Interface defines contract
public interface OrderProcessor {
    OrderResult process(Order order);
}

// ENCAPSULATION: Data and methods bundled together
public class Order {
    private String orderId;
    private OrderStatus status;
    private List<OrderItem> items;
    private BigDecimal totalAmount;
    
    // Private fields, public methods
    public void addItem(OrderItem item) {
        validateStatus();
        items.add(item);
        recalculateTotal();
    }
    
    private void validateStatus() {
        if (status != OrderStatus.PENDING) {
            throw new IllegalStateException("Cannot modify order");
        }
    }
    
    private void recalculateTotal() {
        totalAmount = items.stream()
            .map(OrderItem::getTotalPrice)
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
}

// INHERITANCE: Base class with common behavior
public abstract class BaseOrderProcessor implements OrderProcessor {
    protected OrderValidator validator;
    protected PaymentProcessor paymentProcessor;
    
    public BaseOrderProcessor(OrderValidator validator, PaymentProcessor paymentProcessor) {
        this.validator = validator;
        this.paymentProcessor = paymentProcessor;
    }
    
    @Override
    public final OrderResult process(Order order) {
        validate(order);
        PaymentResult paymentResult = processPayment(order);
        return createResult(order, paymentResult);
    }
    
    protected abstract void validate(Order order);
    protected abstract PaymentResult processPayment(Order order);
    protected abstract OrderResult createResult(Order order, PaymentResult paymentResult);
}

// POLYMORPHISM: Different implementations
public class StandardOrderProcessor extends BaseOrderProcessor {
    public StandardOrderProcessor(OrderValidator validator, PaymentProcessor paymentProcessor) {
        super(validator, paymentProcessor);
    }
    
    @Override
    protected void validate(Order order) {
        validator.validateStandardOrder(order);
    }
    
    @Override
    protected PaymentResult processPayment(Order order) {
        return paymentProcessor.processPayment(order.getTotalAmount());
    }
    
    @Override
    protected OrderResult createResult(Order order, PaymentResult paymentResult) {
        return new OrderResult(order.getOrderId(), paymentResult.getStatus());
    }
}

public class PremiumOrderProcessor extends BaseOrderProcessor {
    public PremiumOrderProcessor(OrderValidator validator, PaymentProcessor paymentProcessor) {
        super(validator, paymentProcessor);
    }
    
    @Override
    protected void validate(Order order) {
        validator.validatePremiumOrder(order);
    }
    
    @Override
    protected PaymentResult processPayment(Order order) {
        // Premium orders get discount
        BigDecimal discountedAmount = order.getTotalAmount().multiply(new BigDecimal("0.9"));
        return paymentProcessor.processPayment(discountedAmount);
    }
    
    @Override
    protected OrderResult createResult(Order order, PaymentResult paymentResult) {
        return new OrderResult(order.getOrderId(), paymentResult.getStatus(), "PREMIUM");
    }
}

// Usage - POLYMORPHISM in action
public class OrderService {
    private List<OrderProcessor> processors;
    
    public void processOrder(Order order) {
        OrderProcessor processor = selectProcessor(order);
        OrderResult result = processor.process(order); // Polymorphic call
        // Handle result
    }
    
    private OrderProcessor selectProcessor(Order order) {
        if (order.isPremium()) {
            return new PremiumOrderProcessor(validator, paymentProcessor);
        }
        return new StandardOrderProcessor(validator, paymentProcessor);
    }
}
```

---

## Key Takeaways

1. **Encapsulation**: Hide data, expose behavior through methods
2. **Inheritance**: Use for "IS-A" relationships, code reuse
3. **Polymorphism**: One interface, many implementations
4. **Abstraction**: Hide complexity, show essentials

**Remember:** These principles work together to create maintainable, scalable, and flexible designs.

---

## Next Steps

1. Learn [SOLID Principles](./03_SOLID_PRINCIPLES.md)
2. Study [Class Design](./04_CLASS_DESIGN.md)
3. Practice [Class Relationships](./05_CLASS_RELATIONSHIPS.md)

