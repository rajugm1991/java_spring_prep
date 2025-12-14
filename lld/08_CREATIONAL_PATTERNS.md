# Creational Design Patterns

## Overview

Creational patterns provide object creation mechanisms that increase flexibility and reuse of existing code. They abstract the instantiation process.

---

## Types of Creational Patterns

1. **Singleton** - One instance only
2. **Factory Method** - Delegate object creation
3. **Abstract Factory** - Families of related objects
4. **Builder** - Complex object construction
5. **Prototype** - Clone existing objects

---

## 1. Singleton Pattern

**Intent:** Ensure a class has only one instance and provide a global point of access to it.

### When to Use
- Need exactly one instance (database connection, logger, config)
- Global access point needed
- Lazy initialization required

### Implementation

#### Eager Initialization
```java
public class DatabaseConnection {
    private static final DatabaseConnection instance = new DatabaseConnection();
    
    private DatabaseConnection() {
        // Private constructor
    }
    
    public static DatabaseConnection getInstance() {
        return instance;
    }
}
```

#### Lazy Initialization (Thread-Safe)
```java
public class DatabaseConnection {
    private static DatabaseConnection instance;
    
    private DatabaseConnection() {
        // Private constructor
    }
    
    public static synchronized DatabaseConnection getInstance() {
        if (instance == null) {
            instance = new DatabaseConnection();
        }
        return instance;
    }
}
```

#### Double-Checked Locking
```java
public class DatabaseConnection {
    private static volatile DatabaseConnection instance;
    
    private DatabaseConnection() {
        // Private constructor
    }
    
    public static DatabaseConnection getInstance() {
        if (instance == null) {
            synchronized (DatabaseConnection.class) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
}
```

#### Bill Pugh Solution (Best for Java)
```java
public class DatabaseConnection {
    private DatabaseConnection() {
        // Private constructor
    }
    
    private static class SingletonHelper {
        private static final DatabaseConnection instance = new DatabaseConnection();
    }
    
    public static DatabaseConnection getInstance() {
        return SingletonHelper.instance;
    }
}
```

### Real-World Example: Logger
```java
public class Logger {
    private static Logger instance;
    private List<String> logs;
    
    private Logger() {
        logs = new ArrayList<>();
    }
    
    public static synchronized Logger getInstance() {
        if (instance == null) {
            instance = new Logger();
        }
        return instance;
    }
    
    public void log(String message) {
        logs.add(LocalDateTime.now() + ": " + message);
    }
    
    public void printLogs() {
        logs.forEach(System.out::println);
    }
}
```

---

## 2. Factory Method Pattern

**Intent:** Define an interface for creating an object, but let subclasses decide which class to instantiate.

### When to Use
- Don't know exact types at compile time
- Want to delegate object creation
- Need to add new types without modifying existing code

### Structure
```
Creator (abstract)
  ├── ConcreteCreator1
  └── ConcreteCreator2

Product (interface)
  ├── ConcreteProduct1
  └── ConcreteProduct2
```

### Implementation

```java
// Product interface
public interface PaymentProcessor {
    PaymentResult processPayment(PaymentRequest request);
}

// Concrete products
public class StripeProcessor implements PaymentProcessor {
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        return new PaymentResult("stripe_123", PaymentStatus.SUCCESS);
    }
}

public class RazorpayProcessor implements PaymentProcessor {
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        return new PaymentResult("razorpay_456", PaymentStatus.SUCCESS);
    }
}

// Creator (abstract)
public abstract class PaymentProcessorFactory {
    public PaymentResult process(PaymentRequest request) {
        PaymentProcessor processor = createProcessor();
        return processor.processPayment(request);
    }
    
    protected abstract PaymentProcessor createProcessor();
}

// Concrete creators
public class StripeFactory extends PaymentProcessorFactory {
    @Override
    protected PaymentProcessor createProcessor() {
        return new StripeProcessor();
    }
}

public class RazorpayFactory extends PaymentProcessorFactory {
    @Override
    protected PaymentProcessor createProcessor() {
        return new RazorpayProcessor();
    }
}
```

### Simple Factory (Static Factory Method)
```java
public class PaymentProcessorFactory {
    public static PaymentProcessor create(String type) {
        switch (type.toLowerCase()) {
            case "stripe":
                return new StripeProcessor();
            case "razorpay":
                return new RazorpayProcessor();
            case "paypal":
                return new PayPalProcessor();
            default:
                throw new IllegalArgumentException("Unknown payment processor: " + type);
        }
    }
}

// Usage
PaymentProcessor processor = PaymentProcessorFactory.create("stripe");
```

### Real-World Example: Vehicle Factory
```java
public interface Vehicle {
    void start();
    void stop();
}

public class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car started");
    }
    
    @Override
    public void stop() {
        System.out.println("Car stopped");
    }
}

public class Motorcycle implements Vehicle {
    @Override
    public void start() {
        System.out.println("Motorcycle started");
    }
    
    @Override
    public void stop() {
        System.out.println("Motorcycle stopped");
    }
}

public class VehicleFactory {
    public static Vehicle create(String type) {
        switch (type.toLowerCase()) {
            case "car":
                return new Car();
            case "motorcycle":
                return new Motorcycle();
            default:
                throw new IllegalArgumentException("Unknown vehicle type");
        }
    }
}
```

---

## 3. Abstract Factory Pattern

**Intent:** Provide an interface for creating families of related objects without specifying their concrete classes.

### When to Use
- Need to create families of related objects
- Want to provide a library of products
- Need to enforce consistency among products

### Structure
```
AbstractFactory
  ├── ConcreteFactory1
  └── ConcreteFactory2

AbstractProductA    AbstractProductB
  ├── ProductA1      ├── ProductB1
  └── ProductA2      └── ProductB2
```

### Implementation

```java
// Abstract products
public interface Button {
    void render();
}

public interface Checkbox {
    void render();
}

// Concrete products - Windows
public class WindowsButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Windows button");
    }
}

public class WindowsCheckbox implements Checkbox {
    @Override
    public void render() {
        System.out.println("Rendering Windows checkbox");
    }
}

// Concrete products - Mac
public class MacButton implements Button {
    @Override
    public void render() {
        System.out.println("Rendering Mac button");
    }
}

public class MacCheckbox implements Checkbox {
    @Override
    public void render() {
        System.out.println("Rendering Mac checkbox");
    }
}

// Abstract factory
public interface GUIFactory {
    Button createButton();
    Checkbox createCheckbox();
}

// Concrete factories
public class WindowsFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new WindowsButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new WindowsCheckbox();
    }
}

public class MacFactory implements GUIFactory {
    @Override
    public Button createButton() {
        return new MacButton();
    }
    
    @Override
    public Checkbox createCheckbox() {
        return new MacCheckbox();
    }
}

// Client
public class Application {
    private Button button;
    private Checkbox checkbox;
    
    public Application(GUIFactory factory) {
        button = factory.createButton();
        checkbox = factory.createCheckbox();
    }
    
    public void render() {
        button.render();
        checkbox.render();
    }
}

// Usage
GUIFactory factory = new WindowsFactory();
Application app = new Application(factory);
app.render();
```

---

## 4. Builder Pattern

**Intent:** Construct complex objects step by step. Allows different representations of the same construction process.

### When to Use
- Complex object construction
- Many optional parameters
- Need immutable objects
- Want to avoid telescoping constructor

### Implementation

```java
public class Pizza {
    private String dough;
    private String sauce;
    private List<String> toppings;
    private String size;
    
    private Pizza(Builder builder) {
        this.dough = builder.dough;
        this.sauce = builder.sauce;
        this.toppings = builder.toppings;
        this.size = builder.size;
    }
    
    // Builder class
    public static class Builder {
        private String dough;
        private String sauce;
        private List<String> toppings = new ArrayList<>();
        private String size;
        
        public Builder dough(String dough) {
            this.dough = dough;
            return this;
        }
        
        public Builder sauce(String sauce) {
            this.sauce = sauce;
            return this;
        }
        
        public Builder addTopping(String topping) {
            this.toppings.add(topping);
            return this;
        }
        
        public Builder size(String size) {
            this.size = size;
            return this;
        }
        
        public Pizza build() {
            if (dough == null || sauce == null) {
                throw new IllegalStateException("Dough and sauce are required");
            }
            return new Pizza(this);
        }
    }
}

// Usage
Pizza pizza = new Pizza.Builder()
    .dough("thin")
    .sauce("tomato")
    .addTopping("cheese")
    .addTopping("pepperoni")
    .size("large")
    .build();
```

### Real-World Example: Order Builder
```java
public class Order {
    private String orderId;
    private Customer customer;
    private List<OrderItem> items;
    private ShippingAddress shippingAddress;
    private PaymentMethod paymentMethod;
    private boolean isGift;
    private String giftMessage;
    
    private Order(Builder builder) {
        this.orderId = builder.orderId;
        this.customer = builder.customer;
        this.items = builder.items;
        this.shippingAddress = builder.shippingAddress;
        this.paymentMethod = builder.paymentMethod;
        this.isGift = builder.isGift;
        this.giftMessage = builder.giftMessage;
    }
    
    public static class Builder {
        private String orderId;
        private Customer customer;
        private List<OrderItem> items = new ArrayList<>();
        private ShippingAddress shippingAddress;
        private PaymentMethod paymentMethod;
        private boolean isGift = false;
        private String giftMessage;
        
        public Builder orderId(String orderId) {
            this.orderId = orderId;
            return this;
        }
        
        public Builder customer(Customer customer) {
            this.customer = customer;
            return this;
        }
        
        public Builder addItem(OrderItem item) {
            this.items.add(item);
            return this;
        }
        
        public Builder shippingAddress(ShippingAddress address) {
            this.shippingAddress = address;
            return this;
        }
        
        public Builder paymentMethod(PaymentMethod method) {
            this.paymentMethod = method;
            return this;
        }
        
        public Builder asGift(String message) {
            this.isGift = true;
            this.giftMessage = message;
            return this;
        }
        
        public Order build() {
            if (orderId == null || customer == null || items.isEmpty()) {
                throw new IllegalStateException("Order ID, customer, and items are required");
            }
            return new Order(this);
        }
    }
}

// Usage
Order order = new Order.Builder()
    .orderId("ORD-123")
    .customer(customer)
    .addItem(item1)
    .addItem(item2)
    .shippingAddress(address)
    .paymentMethod(paymentMethod)
    .asGift("Happy Birthday!")
    .build();
```

---

## 5. Prototype Pattern

**Intent:** Specify the kinds of objects to create using a prototypical instance, and create new objects by copying this prototype.

### When to Use
- Object creation is expensive
- Want to avoid subclassing
- Need to create objects at runtime
- Want to reduce initialization cost

### Implementation

```java
// Prototype interface
public interface Cloneable {
    Cloneable clone();
}

// Concrete prototype
public class Document implements Cloneable {
    private String title;
    private String content;
    private List<String> images;
    
    public Document(String title, String content) {
        this.title = title;
        this.content = content;
        this.images = new ArrayList<>();
    }
    
    public void addImage(String image) {
        images.add(image);
    }
    
    @Override
    public Document clone() {
        Document cloned = new Document(this.title, this.content);
        cloned.images = new ArrayList<>(this.images);
        return cloned;
    }
    
    // Getters
    public String getTitle() { return title; }
    public String getContent() { return content; }
    public List<String> getImages() { return images; }
}

// Usage
Document original = new Document("Report", "Content here");
original.addImage("image1.jpg");
original.addImage("image2.jpg");

Document copy = original.clone();
copy.addImage("image3.jpg"); // Doesn't affect original
```

### Shallow vs Deep Copy

```java
// Shallow copy (default clone)
public class ShallowCopyExample implements Cloneable {
    private List<String> items;
    
    @Override
    public ShallowCopyExample clone() {
        try {
            return (ShallowCopyExample) super.clone(); // Shallow copy
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
}

// Deep copy
public class DeepCopyExample implements Cloneable {
    private List<String> items;
    
    @Override
    public DeepCopyExample clone() {
        DeepCopyExample cloned = new DeepCopyExample();
        cloned.items = new ArrayList<>(this.items); // Deep copy
        return cloned;
    }
}
```

---

## Pattern Comparison

| Pattern | Use Case | Complexity | Flexibility |
|---------|----------|------------|-------------|
| **Singleton** | One instance needed | Low | Low |
| **Factory Method** | Delegate creation | Medium | High |
| **Abstract Factory** | Families of objects | High | High |
| **Builder** | Complex construction | Medium | High |
| **Prototype** | Clone existing objects | Medium | Medium |

---

## Key Takeaways

1. **Singleton**: One instance, global access
2. **Factory Method**: Delegate object creation
3. **Abstract Factory**: Create families of objects
4. **Builder**: Step-by-step construction
5. **Prototype**: Clone existing objects

**Remember:** Choose the pattern that best fits your use case. Don't over-engineer!

---

## Next Steps

1. Study [Structural Patterns](./09_STRUCTURAL_PATTERNS.md)
2. Learn [Behavioral Patterns](./10_BEHAVIORAL_PATTERNS.md)
3. Practice [Real-World Problems](./12_PARKING_LOT_SYSTEM.md)

