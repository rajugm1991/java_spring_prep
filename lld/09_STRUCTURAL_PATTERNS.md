# Structural Patterns

## Table of Contents
1. [Introduction](#introduction)
2. [Adapter Pattern](#adapter-pattern)
3. [Facade Pattern](#facade-pattern)
4. [Decorator Pattern](#decorator-pattern)
5. [Proxy Pattern](#proxy-pattern)
6. [Bridge Pattern](#bridge-pattern)
7. [Composite Pattern](#composite-pattern)
8. [Real-World Examples](#real-world-examples)

---

## Introduction

**Structural Patterns** deal with object composition and relationships between objects. They help assemble objects into larger structures while keeping them flexible and efficient.

---

## Adapter Pattern

**Adapter** allows incompatible interfaces to work together.

### Problem

**Incompatible interfaces:**
```java
// Existing interface
public interface PaymentProcessor {
    void pay(double amount);
}

// New third-party library with different interface
public class StripeAPI {
    public void chargePayment(String currency, double amount) { ... }
}
```

### Solution: Adapter

```java
// Adapter implements target interface, wraps adaptee
public class StripeAdapter implements PaymentProcessor {
    private StripeAPI stripeAPI;
    
    public StripeAdapter(StripeAPI stripeAPI) {
        this.stripeAPI = stripeAPI;
    }
    
    @Override
    public void pay(double amount) {
        // Adapt StripeAPI to PaymentProcessor interface
        stripeAPI.chargePayment("USD", amount);
    }
}

// Usage
PaymentProcessor processor = new StripeAdapter(new StripeAPI());
processor.pay(100.0);  // Works with PaymentProcessor interface
```

### Real-World Example: Payment Gateway Adapter

```java
// Target interface
public interface PaymentGateway {
    PaymentResult processPayment(PaymentRequest request);
}

// Adapters for different providers
public class StripeAdapter implements PaymentGateway {
    private StripeClient stripeClient;
    
    public PaymentResult processPayment(PaymentRequest request) {
        // Convert PaymentRequest to Stripe format
        StripeCharge charge = stripeClient.charge(
            request.getAmount(),
            request.getCurrency()
        );
        // Convert Stripe response to PaymentResult
        return convertToPaymentResult(charge);
    }
}

public class RazorpayAdapter implements PaymentGateway {
    private RazorpayClient razorpayClient;
    
    public PaymentResult processPayment(PaymentRequest request) {
        // Similar adaptation for Razorpay
    }
}
```

---

## Facade Pattern

**Facade** provides a simplified interface to a complex subsystem.

### Problem

**Complex subsystem:**
```java
// Multiple classes to interact with
OrderService orderService = new OrderService();
PaymentService paymentService = new PaymentService();
InventoryService inventoryService = new InventoryService();
NotificationService notificationService = new NotificationService();

// Complex interaction
orderService.createOrder(order);
paymentService.processPayment(order);
inventoryService.updateInventory(order);
notificationService.sendConfirmation(order);
```

### Solution: Facade

```java
// Facade simplifies the interface
public class OrderFacade {
    private OrderService orderService;
    private PaymentService paymentService;
    private InventoryService inventoryService;
    private NotificationService notificationService;
    
    public OrderResult placeOrder(OrderRequest request) {
        // Hide complexity behind simple interface
        Order order = orderService.createOrder(request);
        paymentService.processPayment(order);
        inventoryService.updateInventory(order);
        notificationService.sendConfirmation(order);
        return new OrderResult(order);
    }
}

// Usage: Simple
OrderFacade facade = new OrderFacade();
OrderResult result = facade.placeOrder(request);
```

### Real-World Example: E-Commerce Facade

```java
public class ShoppingFacade {
    private ProductService productService;
    private CartService cartService;
    private OrderService orderService;
    private PaymentService paymentService;
    
    public OrderResult checkout(Long userId, String paymentMethod) {
        // Simplified interface for complex checkout process
        Cart cart = cartService.getCart(userId);
        Order order = orderService.createOrder(cart);
        PaymentResult payment = paymentService.processPayment(order, paymentMethod);
        if (payment.isSuccess()) {
            orderService.confirmOrder(order);
            cartService.clearCart(userId);
            return new OrderResult(order, OrderStatus.CONFIRMED);
        }
        return new OrderResult(order, OrderStatus.PAYMENT_FAILED);
    }
}
```

---

## Decorator Pattern

**Decorator** adds behavior to objects dynamically without altering their structure.

### Problem

**Need to add features without modifying base class:**
```java
public class Coffee {
    public double getCost() {
        return 5.0;
    }
}

// Want to add milk, sugar, etc. without modifying Coffee class
```

### Solution: Decorator

```java
// Component interface
public interface Beverage {
    double getCost();
    String getDescription();
}

// Concrete component
public class Coffee implements Beverage {
    public double getCost() {
        return 5.0;
    }
    
    public String getDescription() {
        return "Coffee";
    }
}

// Decorator base class
public abstract class BeverageDecorator implements Beverage {
    protected Beverage beverage;
    
    public BeverageDecorator(Beverage beverage) {
        this.beverage = beverage;
    }
    
    public String getDescription() {
        return beverage.getDescription();
    }
}

// Concrete decorators
public class MilkDecorator extends BeverageDecorator {
    public MilkDecorator(Beverage beverage) {
        super(beverage);
    }
    
    public double getCost() {
        return beverage.getCost() + 1.0;
    }
    
    public String getDescription() {
        return beverage.getDescription() + ", Milk";
    }
}

public class SugarDecorator extends BeverageDecorator {
    public SugarDecorator(Beverage beverage) {
        super(beverage);
    }
    
    public double getCost() {
        return beverage.getCost() + 0.5;
    }
    
    public String getDescription() {
        return beverage.getDescription() + ", Sugar";
    }
}

// Usage
Beverage coffee = new Coffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);
// Cost: 5.0 + 1.0 + 0.5 = 6.5
```

### Real-World Example: Notification Decorator

```java
public interface Notification {
    void send();
}

public class EmailNotification implements Notification {
    public void send() {
        // Send email
    }
}

public abstract class NotificationDecorator implements Notification {
    protected Notification notification;
    
    public NotificationDecorator(Notification notification) {
        this.notification = notification;
    }
}

public class RetryDecorator extends NotificationDecorator {
    public RetryDecorator(Notification notification) {
        super(notification);
    }
    
    public void send() {
        int maxRetries = 3;
        for (int i = 0; i < maxRetries; i++) {
            try {
                notification.send();
                return;
            } catch (Exception e) {
                if (i == maxRetries - 1) throw e;
            }
        }
    }
}

public class LoggingDecorator extends NotificationDecorator {
    public LoggingDecorator(Notification notification) {
        super(notification);
    }
    
    public void send() {
        logger.info("Sending notification");
        notification.send();
        logger.info("Notification sent");
    }
}

// Usage
Notification notification = new EmailNotification();
notification = new RetryDecorator(notification);
notification = new LoggingDecorator(notification);
notification.send();
```

---

## Proxy Pattern

**Proxy** provides a placeholder or surrogate for another object to control access.

### Types of Proxies

#### 1. **Virtual Proxy**

**Lazy loading:**
```java
public interface Image {
    void display();
}

public class RealImage implements Image {
    private String filename;
    
    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();  // Expensive operation
    }
    
    public void display() {
        System.out.println("Displaying " + filename);
    }
    
    private void loadFromDisk() {
        // Load image from disk (slow)
    }
}

public class ProxyImage implements Image {
    private String filename;
    private RealImage realImage;
    
    public ProxyImage(String filename) {
        this.filename = filename;
    }
    
    public void display() {
        if (realImage == null) {
            realImage = new RealImage(filename);  // Lazy loading
        }
        realImage.display();
    }
}
```

---

#### 2. **Protection Proxy**

**Access control:**
```java
public interface BankAccount {
    void deposit(double amount);
    void withdraw(double amount);
    double getBalance();
}

public class RealBankAccount implements BankAccount {
    private double balance;
    
    public void deposit(double amount) {
        balance += amount;
    }
    
    public void withdraw(double amount) {
        balance -= amount;
    }
    
    public double getBalance() {
        return balance;
    }
}

public class ProtectionProxy implements BankAccount {
    private RealBankAccount account;
    private String userRole;
    
    public ProtectionProxy(RealBankAccount account, String userRole) {
        this.account = account;
        this.userRole = userRole;
    }
    
    public void deposit(double amount) {
        if (userRole.equals("ADMIN") || userRole.equals("USER")) {
            account.deposit(amount);
        } else {
            throw new UnauthorizedException("Cannot deposit");
        }
    }
    
    public void withdraw(double amount) {
        if (userRole.equals("ADMIN")) {
            account.withdraw(amount);
        } else {
            throw new UnauthorizedException("Only admin can withdraw");
        }
    }
    
    public double getBalance() {
        return account.getBalance();
    }
}
```

---

#### 3. **Remote Proxy**

**Network communication:**
```java
// Client-side proxy
public class RemoteOrderService implements OrderService {
    private String serverUrl;
    
    public Order createOrder(OrderRequest request) {
        // Make network call to remote server
        return httpClient.post(serverUrl + "/orders", request);
    }
}
```

---

## Bridge Pattern

**Bridge** decouples abstraction from implementation.

### Problem

**Tight coupling between abstraction and implementation:**
```java
// If we add new shape or color, need to create many classes
public class RedCircle extends Circle { ... }
public class BlueCircle extends Circle { ... }
public class RedSquare extends Square { ... }
public class BlueSquare extends Square { ... }
```

### Solution: Bridge

```java
// Abstraction
public abstract class Shape {
    protected Color color;  // Bridge to implementation
    
    public Shape(Color color) {
        this.color = color;
    }
    
    public abstract void draw();
}

// Refined abstraction
public class Circle extends Shape {
    public Circle(Color color) {
        super(color);
    }
    
    public void draw() {
        System.out.println("Drawing circle with " + color.fill());
    }
}

// Implementation interface
public interface Color {
    String fill();
}

// Concrete implementations
public class Red implements Color {
    public String fill() {
        return "red color";
    }
}

public class Blue implements Color {
    public String fill() {
        return "blue color";
    }
}

// Usage
Shape circle = new Circle(new Red());
circle.draw();  // Drawing circle with red color
```

---

## Composite Pattern

**Composite** composes objects into tree structures to represent part-whole hierarchies.

### Example: File System

```java
// Component interface
public interface FileSystemNode {
    void display(String indent);
    long getSize();
}

// Leaf
public class File implements FileSystemNode {
    private String name;
    private long size;
    
    public File(String name, long size) {
        this.name = name;
        this.size = size;
    }
    
    public void display(String indent) {
        System.out.println(indent + "File: " + name);
    }
    
    public long getSize() {
        return size;
    }
}

// Composite
public class Directory implements FileSystemNode {
    private String name;
    private List<FileSystemNode> children;
    
    public Directory(String name) {
        this.name = name;
        this.children = new ArrayList<>();
    }
    
    public void add(FileSystemNode node) {
        children.add(node);
    }
    
    public void display(String indent) {
        System.out.println(indent + "Directory: " + name);
        for (FileSystemNode child : children) {
            child.display(indent + "  ");
        }
    }
    
    public long getSize() {
        return children.stream()
            .mapToLong(FileSystemNode::getSize)
            .sum();
    }
}

// Usage
Directory root = new Directory("root");
root.add(new File("file1.txt", 100));
Directory subdir = new Directory("subdir");
subdir.add(new File("file2.txt", 200));
root.add(subdir);
root.display("");
```

---

## Real-World Examples

### Example 1: Payment Gateway Adapter

```java
// Unified interface
public interface PaymentGateway {
    PaymentResult charge(double amount, String currency);
}

// Adapters for different providers
public class StripeAdapter implements PaymentGateway {
    private StripeClient client;
    
    public PaymentResult charge(double amount, String currency) {
        StripeCharge charge = client.createCharge(amount, currency);
        return new PaymentResult(charge.getId(), charge.getStatus());
    }
}

public class PayPalAdapter implements PaymentGateway {
    private PayPalClient client;
    
    public PaymentResult charge(double amount, String currency) {
        PayPalPayment payment = client.pay(amount, currency);
        return new PaymentResult(payment.getId(), payment.getStatus());
    }
}
```

---

### Example 2: API Gateway Facade

```java
public class ApiGatewayFacade {
    private UserService userService;
    private OrderService orderService;
    private PaymentService paymentService;
    private NotificationService notificationService;
    
    public OrderResponse placeOrder(OrderRequest request) {
        // Simplified interface for complex operation
        User user = userService.getUser(request.getUserId());
        Order order = orderService.createOrder(request);
        PaymentResult payment = paymentService.processPayment(order);
        if (payment.isSuccess()) {
            notificationService.sendConfirmation(user, order);
            return new OrderResponse(order, "SUCCESS");
        }
        return new OrderResponse(order, "PAYMENT_FAILED");
    }
}
```

---

## Summary

**Key Patterns:**
1. ✅ **Adapter:** Make incompatible interfaces work together
2. ✅ **Facade:** Simplify complex subsystem
3. ✅ **Decorator:** Add behavior dynamically
4. ✅ **Proxy:** Control access to object
5. ✅ **Bridge:** Decouple abstraction from implementation
6. ✅ **Composite:** Represent part-whole hierarchies

**Next Steps:**
- Learn [Behavioral Patterns](./10_BEHAVIORAL_PATTERNS.md)
- Understand [Creational Patterns](./08_CREATIONAL_PATTERNS.md)
- Practice [Real-World Problems](./12_PARKING_LOT_SYSTEM.md)

---

**References:**
- [Design Patterns](https://refactoring.guru/design-patterns)
- [SOLID Principles](./03_SOLID_PRINCIPLES.md)

