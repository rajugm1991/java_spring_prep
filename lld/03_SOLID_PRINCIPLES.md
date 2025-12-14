# SOLID Principles - Complete Guide

## Overview

SOLID is an acronym for five design principles that make software designs more understandable, flexible, and maintainable. These principles are fundamental to Low-Level Design.

```
S - Single Responsibility Principle
O - Open/Closed Principle
L - Liskov Substitution Principle
I - Interface Segregation Principle
D - Dependency Inversion Principle
```

---

## S: Single Responsibility Principle (SRP)

**Definition:** A class should have only one reason to change. It should have only one job or responsibility.

### Violation Example

```java
// BAD: This class has multiple responsibilities
public class User {
    private String name;
    private String email;
    
    // Responsibility 1: User data management
    public void setName(String name) { this.name = name; }
    public String getName() { return name; }
    
    // Responsibility 2: Database operations
    public void saveToDatabase() {
        // Database save logic
    }
    
    // Responsibility 3: Email sending
    public void sendEmail(String message) {
        // Email sending logic
    }
    
    // Responsibility 4: Validation
    public boolean validateEmail() {
        // Email validation logic
    }
}
```

**Problems:**
- If database schema changes, User class changes
- If email service changes, User class changes
- If validation rules change, User class changes
- Hard to test (need to mock database and email service)
- Hard to reuse (tightly coupled to database and email)

### Correct Implementation

```java
// Responsibility 1: User data model
public class User {
    private String name;
    private String email;
    
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
    
    public String getName() { return name; }
    public String getEmail() { return email; }
}

// Responsibility 2: Database operations
public class UserRepository {
    public void save(User user) {
        // Database save logic
    }
    
    public User findById(String id) {
        // Database find logic
    }
}

// Responsibility 3: Email operations
public class EmailService {
    public void sendEmail(String to, String message) {
        // Email sending logic
    }
}

// Responsibility 4: Validation
public class UserValidator {
    public boolean validateEmail(String email) {
        // Email validation logic
        return email.matches("^[A-Za-z0-9+_.-]+@(.+)$");
    }
    
    public boolean validateUser(User user) {
        return user.getName() != null && 
               !user.getName().isEmpty() && 
               validateEmail(user.getEmail());
    }
}
```

### Real-World Example: Order Management

```java
// BAD: Order class doing too much
public class Order {
    private String orderId;
    private List<Item> items;
    
    public void calculateTotal() { ... }
    public void saveToDatabase() { ... }
    public void sendConfirmationEmail() { ... }
    public void generateInvoice() { ... }
    public void updateInventory() { ... }
}

// GOOD: Separated responsibilities
public class Order {
    private String orderId;
    private List<Item> items;
    private BigDecimal totalAmount;
    
    // Only order-related logic
    public void addItem(Item item) { ... }
    public BigDecimal calculateTotal() { ... }
    public OrderStatus getStatus() { ... }
}

public class OrderRepository {
    public void save(Order order) { ... }
    public Order findById(String id) { ... }
}

public class OrderNotificationService {
    public void sendConfirmation(Order order) { ... }
}

public class InvoiceService {
    public Invoice generateInvoice(Order order) { ... }
}

public class InventoryService {
    public void updateInventory(Order order) { ... }
}
```

---

## O: Open/Closed Principle (OCP)

**Definition:** Software entities (classes, modules, functions) should be open for extension but closed for modification.

### Violation Example

```java
// BAD: Need to modify this class for every new shape
public class AreaCalculator {
    public double calculateArea(Object shape) {
        if (shape instanceof Rectangle) {
            Rectangle rect = (Rectangle) shape;
            return rect.getWidth() * rect.getHeight();
        } else if (shape instanceof Circle) {
            Circle circle = (Circle) shape;
            return Math.PI * circle.getRadius() * circle.getRadius();
        } else if (shape instanceof Triangle) {
            // Need to modify this class to add Triangle
            Triangle triangle = (Triangle) shape;
            return 0.5 * triangle.getBase() * triangle.getHeight();
        }
        throw new IllegalArgumentException("Unknown shape");
    }
}
```

**Problems:**
- Must modify existing code to add new shapes
- Violates SRP (handles all shape types)
- Hard to test (need to test all branches)
- Risk of breaking existing functionality

### Correct Implementation

```java
// GOOD: Open for extension, closed for modification
public interface Shape {
    double calculateArea();
}

public class Rectangle implements Shape {
    private double width;
    private double height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return width * height;
    }
}

public class Circle implements Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

// Can add new shapes without modifying AreaCalculator
public class Triangle implements Shape {
    private double base;
    private double height;
    
    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return 0.5 * base * height;
    }
}

// This class never needs to change
public class AreaCalculator {
    public double calculateArea(Shape shape) {
        return shape.calculateArea(); // Works with any Shape
    }
    
    public double calculateTotalArea(List<Shape> shapes) {
        return shapes.stream()
            .mapToDouble(Shape::calculateArea)
            .sum();
    }
}
```

### Real-World Example: Payment Processing

```java
// BAD: Need to modify for each payment method
public class PaymentProcessor {
    public void processPayment(String method, double amount) {
        if (method.equals("credit_card")) {
            // Credit card processing
        } else if (method.equals("paypal")) {
            // PayPal processing
        } else if (method.equals("stripe")) {
            // Need to modify this class
        }
    }
}

// GOOD: Open for extension
public interface PaymentMethod {
    PaymentResult processPayment(double amount);
}

public class CreditCardPayment implements PaymentMethod {
    @Override
    public PaymentResult processPayment(double amount) {
        // Credit card processing
        return new PaymentResult("success", "credit_card_txn_123");
    }
}

public class PayPalPayment implements PaymentMethod {
    @Override
    public PaymentResult processPayment(double amount) {
        // PayPal processing
        return new PaymentResult("success", "paypal_txn_456");
    }
}

// Can add new payment methods without modifying PaymentProcessor
public class StripePayment implements PaymentMethod {
    @Override
    public PaymentResult processPayment(double amount) {
        // Stripe processing
        return new PaymentResult("success", "stripe_txn_789");
    }
}

// This class never needs to change
public class PaymentProcessor {
    public PaymentResult process(PaymentMethod method, double amount) {
        return method.processPayment(amount);
    }
}
```

---

## L: Liskov Substitution Principle (LSP)

**Definition:** Objects of a superclass should be replaceable with objects of a subclass without breaking the application.

### Violation Example

```java
// BAD: Rectangle and Square violate LSP
public class Rectangle {
    protected int width;
    protected int height;
    
    public void setWidth(int width) {
        this.width = width;
    }
    
    public void setHeight(int height) {
        this.height = height;
    }
    
    public int getArea() {
        return width * height;
    }
}

public class Square extends Rectangle {
    @Override
    public void setWidth(int width) {
        this.width = width;
        this.height = width; // Square maintains width == height
    }
    
    @Override
    public void setHeight(int height) {
        this.width = height;
        this.height = height;
    }
}

// This function expects Rectangle behavior
public void testRectangle(Rectangle rect) {
    rect.setWidth(5);
    rect.setHeight(4);
    assert rect.getArea() == 20; // Expects 20
    // But if rect is Square, area will be 16, not 20!
}
```

**Problem:** Square cannot be substituted for Rectangle because it changes the expected behavior.

### Correct Implementation

```java
// GOOD: Don't force inheritance where it doesn't make sense
public interface Shape {
    int getArea();
}

public class Rectangle implements Shape {
    private int width;
    private int height;
    
    public Rectangle(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    public void setWidth(int width) {
        this.width = width;
    }
    
    public void setHeight(int height) {
        this.height = height;
    }
    
    @Override
    public int getArea() {
        return width * height;
    }
}

public class Square implements Shape {
    private int side;
    
    public Square(int side) {
        this.side = side;
    }
    
    public void setSide(int side) {
        this.side = side;
    }
    
    @Override
    public int getArea() {
        return side * side;
    }
}

// Both can be used interchangeably as Shape
public void testShape(Shape shape) {
    int area = shape.getArea(); // Works for both Rectangle and Square
}
```

### Real-World Example: Bird Hierarchy

```java
// BAD: Violates LSP
public class Bird {
    public void fly() {
        System.out.println("Flying...");
    }
}

public class Sparrow extends Bird {
    // Sparrow can fly - OK
}

public class Penguin extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Penguins can't fly!");
    }
}

// Problem: Code expecting Bird to fly will break with Penguin
public void makeBirdFly(Bird bird) {
    bird.fly(); // Crashes if bird is Penguin!
}

// GOOD: Separate interfaces
public interface Bird {
    void eat();
    void sleep();
}

public interface FlyingBird extends Bird {
    void fly();
}

public class Sparrow implements FlyingBird {
    @Override
    public void eat() { ... }
    
    @Override
    public void sleep() { ... }
    
    @Override
    public void fly() { ... }
}

public class Penguin implements Bird {
    @Override
    public void eat() { ... }
    
    @Override
    public void sleep() { ... }
    // No fly() method - penguins can't fly
}

// Now code is safe
public void makeBirdFly(FlyingBird bird) {
    bird.fly(); // Only accepts birds that can fly
}
```

---

## I: Interface Segregation Principle (ISP)

**Definition:** Clients should not be forced to depend on interfaces they don't use. Many client-specific interfaces are better than one general-purpose interface.

### Violation Example

```java
// BAD: Fat interface - forces all clients to implement all methods
public interface Worker {
    void work();
    void eat();
    void sleep();
}

public class HumanWorker implements Worker {
    @Override
    public void work() { ... }
    
    @Override
    public void eat() { ... }
    
    @Override
    public void sleep() { ... }
}

public class RobotWorker implements Worker {
    @Override
    public void work() { ... }
    
    @Override
    public void eat() {
        throw new UnsupportedOperationException("Robots don't eat!");
    }
    
    @Override
    public void sleep() {
        throw new UnsupportedOperationException("Robots don't sleep!");
    }
}
```

**Problems:**
- Robot forced to implement methods it doesn't need
- Violates LSP (Robot can't be substituted for Worker in all contexts)
- Hard to maintain (changes to interface affect all implementations)

### Correct Implementation

```java
// GOOD: Segregated interfaces
public interface Workable {
    void work();
}

public interface Eatable {
    void eat();
}

public interface Sleepable {
    void sleep();
}

public class HumanWorker implements Workable, Eatable, Sleepable {
    @Override
    public void work() { ... }
    
    @Override
    public void eat() { ... }
    
    @Override
    public void sleep() { ... }
}

public class RobotWorker implements Workable {
    @Override
    public void work() { ... }
    // No need to implement eat() or sleep()
}

// Clients depend only on what they need
public class WorkManager {
    public void manageWork(Workable worker) {
        worker.work(); // Only needs work() method
    }
}
```

### Real-World Example: Printer Interface

```java
// BAD: Fat interface
public interface Printer {
    void print();
    void scan();
    void fax();
}

public class SimplePrinter implements Printer {
    @Override
    public void print() { ... }
    
    @Override
    public void scan() {
        throw new UnsupportedOperationException("Can't scan");
    }
    
    @Override
    public void fax() {
        throw new UnsupportedOperationException("Can't fax");
    }
}

// GOOD: Segregated interfaces
public interface Printer {
    void print();
}

public interface Scanner {
    void scan();
}

public interface FaxMachine {
    void fax();
}

public class SimplePrinter implements Printer {
    @Override
    public void print() { ... }
}

public class MultiFunctionPrinter implements Printer, Scanner, FaxMachine {
    @Override
    public void print() { ... }
    
    @Override
    public void scan() { ... }
    
    @Override
    public void fax() { ... }
}

// Clients depend only on what they need
public class DocumentService {
    public void printDocument(Printer printer) {
        printer.print(); // Only needs print() method
    }
}
```

---

## D: Dependency Inversion Principle (DIP)

**Definition:** High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions.

### Violation Example

```java
// BAD: High-level module depends on low-level module
public class OrderService {
    private MySQLDatabase database; // Depends on concrete class
    
    public OrderService() {
        this.database = new MySQLDatabase(); // Tight coupling
    }
    
    public void saveOrder(Order order) {
        database.save(order);
    }
}

public class MySQLDatabase {
    public void save(Order order) {
        // MySQL-specific implementation
    }
}
```

**Problems:**
- Hard to test (can't easily mock database)
- Hard to change (switching to PostgreSQL requires changing OrderService)
- Violates OCP (need to modify OrderService to change database)

### Correct Implementation

```java
// GOOD: Both depend on abstraction
public interface OrderRepository {
    void save(Order order);
    Order findById(String id);
}

public class OrderService {
    private OrderRepository repository; // Depends on abstraction
    
    public OrderService(OrderRepository repository) {
        this.repository = repository; // Dependency Injection
    }
    
    public void saveOrder(Order order) {
        repository.save(order);
    }
}

// Low-level modules implement abstraction
public class MySQLOrderRepository implements OrderRepository {
    @Override
    public void save(Order order) {
        // MySQL-specific implementation
    }
    
    @Override
    public Order findById(String id) {
        // MySQL-specific implementation
    }
}

public class PostgreSQLOrderRepository implements OrderRepository {
    @Override
    public void save(Order order) {
        // PostgreSQL-specific implementation
    }
    
    @Override
    public Order findById(String id) {
        // PostgreSQL-specific implementation
    }
}

// Usage - can easily switch implementations
OrderRepository mysqlRepo = new MySQLOrderRepository();
OrderService service1 = new OrderService(mysqlRepo);

OrderRepository postgresRepo = new PostgreSQLOrderRepository();
OrderService service2 = new OrderService(postgresRepo);

// Easy to test with mock
OrderRepository mockRepo = mock(OrderRepository.class);
OrderService testService = new OrderService(mockRepo);
```

### Real-World Example: Notification System

```java
// BAD: High-level depends on low-level
public class UserService {
    private EmailService emailService;
    private SMSService smsService;
    
    public UserService() {
        this.emailService = new EmailService();
        this.smsService = new SMSService();
    }
    
    public void registerUser(User user) {
        // Save user
        emailService.sendWelcomeEmail(user.getEmail());
        smsService.sendWelcomeSMS(user.getPhone());
    }
}

// GOOD: Both depend on abstraction
public interface NotificationService {
    void send(String recipient, String message);
}

public class UserService {
    private List<NotificationService> notificationServices;
    
    public UserService(List<NotificationService> notificationServices) {
        this.notificationServices = notificationServices;
    }
    
    public void registerUser(User user) {
        // Save user
        String message = "Welcome!";
        for (NotificationService service : notificationServices) {
            service.send(user.getContact(), message);
        }
    }
}

// Low-level implementations
public class EmailService implements NotificationService {
    @Override
    public void send(String recipient, String message) {
        // Email implementation
    }
}

public class SMSService implements NotificationService {
    @Override
    public void send(String recipient, String message) {
        // SMS implementation
    }
}

public class PushNotificationService implements NotificationService {
    @Override
    public void send(String recipient, String message) {
        // Push notification implementation
    }
}
```

---

## SOLID Principles Together: Complete Example

```java
// S: Single Responsibility - Each class has one job
// O: Open/Closed - Open for extension via interfaces
// L: Liskov Substitution - All processors can be substituted
// I: Interface Segregation - Small, focused interfaces
// D: Dependency Inversion - Depend on abstractions

// Interface (I: Interface Segregation, D: Dependency Inversion)
public interface PaymentProcessor {
    PaymentResult process(PaymentRequest request);
}

// Concrete implementations (O: Open/Closed)
public class StripeProcessor implements PaymentProcessor {
    @Override
    public PaymentResult process(PaymentRequest request) {
        // Stripe-specific logic
        return new PaymentResult("stripe_123", PaymentStatus.SUCCESS);
    }
}

public class RazorpayProcessor implements PaymentProcessor {
    @Override
    public PaymentResult process(PaymentRequest request) {
        // Razorpay-specific logic
        return new PaymentResult("razorpay_456", PaymentStatus.SUCCESS);
    }
}

// Validator (S: Single Responsibility)
public class PaymentValidator {
    public void validate(PaymentRequest request) {
        if (request.getAmount().compareTo(BigDecimal.ZERO) <= 0) {
            throw new IllegalArgumentException("Amount must be positive");
        }
        if (request.getCurrency() == null) {
            throw new IllegalArgumentException("Currency is required");
        }
    }
}

// Logger (S: Single Responsibility)
public class PaymentLogger {
    public void log(PaymentRequest request, PaymentResult result) {
        System.out.println("Payment processed: " + result.getTransactionId());
    }
}

// Service (S: Single Responsibility, D: Dependency Inversion)
public class PaymentService {
    private PaymentProcessor processor; // D: Depends on abstraction
    private PaymentValidator validator; // D: Depends on abstraction
    private PaymentLogger logger; // D: Depends on abstraction
    
    public PaymentService(PaymentProcessor processor, 
                         PaymentValidator validator,
                         PaymentLogger logger) {
        this.processor = processor;
        this.validator = validator;
        this.logger = logger;
    }
    
    public PaymentResult processPayment(PaymentRequest request) {
        validator.validate(request); // L: Can substitute with any validator
        PaymentResult result = processor.process(request); // L: Can substitute with any processor
        logger.log(request, result); // L: Can substitute with any logger
        return result;
    }
}

// Usage
PaymentProcessor stripe = new StripeProcessor();
PaymentValidator validator = new PaymentValidator();
PaymentLogger logger = new PaymentLogger();

PaymentService service = new PaymentService(stripe, validator, logger);
PaymentResult result = service.processPayment(new PaymentRequest(...));

// Easy to extend (O: Open/Closed)
PaymentProcessor razorpay = new RazorpayProcessor();
PaymentService service2 = new PaymentService(razorpay, validator, logger);
// No modification to PaymentService needed!
```

---

## Key Takeaways

1. **SRP**: One class, one responsibility
2. **OCP**: Open for extension, closed for modification
3. **LSP**: Subtypes must be substitutable for their base types
4. **ISP**: Many specific interfaces better than one general interface
5. **DIP**: Depend on abstractions, not concretions

**Remember:** SOLID principles work together to create maintainable, testable, and extensible code.

---

## Next Steps

1. Practice [Class Design](./04_CLASS_DESIGN.md)
2. Study [Design Patterns](./08_CREATIONAL_PATTERNS.md)
3. Solve [Real-World Problems](./12_PARKING_LOT_SYSTEM.md)

