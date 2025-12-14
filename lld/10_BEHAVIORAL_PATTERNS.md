# Behavioral Patterns

## Table of Contents
1. [Introduction](#introduction)
2. [Observer Pattern](#observer-pattern)
3. [Strategy Pattern](#strategy-pattern)
4. [Command Pattern](#command-pattern)
5. [State Pattern](#state-pattern)
6. [Template Method Pattern](#template-method-pattern)
7. [Chain of Responsibility](#chain-of-responsibility)
8. [Real-World Examples](#real-world-examples)

---

## Introduction

**Behavioral Patterns** focus on communication between objects and how responsibilities are assigned.

---

## Observer Pattern

**Observer** defines a one-to-many dependency between objects so that when one object changes state, all dependents are notified.

### Problem

**Tight coupling:**
```java
public class OrderService {
    private EmailService emailService;
    private SmsService smsService;
    private AnalyticsService analyticsService;
    
    public void createOrder(Order order) {
        // Create order
        emailService.sendEmail(order);  // Tight coupling
        smsService.sendSms(order);
        analyticsService.track(order);
    }
}
```

### Solution: Observer

```java
// Subject (Observable)
public interface OrderSubject {
    void attach(OrderObserver observer);
    void detach(OrderObserver observer);
    void notifyObservers(Order order);
}

// Observer interface
public interface OrderObserver {
    void update(Order order);
}

// Concrete subject
public class OrderService implements OrderSubject {
    private List<OrderObserver> observers = new ArrayList<>();
    
    public void attach(OrderObserver observer) {
        observers.add(observer);
    }
    
    public void notifyObservers(Order order) {
        for (OrderObserver observer : observers) {
            observer.update(order);
        }
    }
    
    public void createOrder(Order order) {
        // Create order
        notifyObservers(order);  // Notify all observers
    }
}

// Concrete observers
public class EmailObserver implements OrderObserver {
    public void update(Order order) {
        // Send email
    }
}

public class SmsObserver implements OrderObserver {
    public void update(Order order) {
        // Send SMS
    }
}

// Usage
OrderService orderService = new OrderService();
orderService.attach(new EmailObserver());
orderService.attach(new SmsObserver());
orderService.createOrder(order);  // All observers notified
```

### Real-World Example: Event-Driven System

```java
// Using Java's built-in Observer (deprecated, but concept applies)
public class OrderEventPublisher {
    private List<OrderEventListener> listeners = new ArrayList<>();
    
    public void addListener(OrderEventListener listener) {
        listeners.add(listener);
    }
    
    public void publishOrderCreated(Order order) {
        OrderEvent event = new OrderEvent(order, "CREATED");
        for (OrderEventListener listener : listeners) {
            listener.onOrderCreated(event);
        }
    }
}

public interface OrderEventListener {
    void onOrderCreated(OrderEvent event);
}

public class EmailService implements OrderEventListener {
    public void onOrderCreated(OrderEvent event) {
        sendConfirmationEmail(event.getOrder());
    }
}
```

---

## Strategy Pattern

**Strategy** defines a family of algorithms, encapsulates each one, and makes them interchangeable.

### Problem

**Multiple algorithms, hard to switch:**
```java
public class PaymentService {
    public void processPayment(String method, double amount) {
        if (method.equals("CREDIT_CARD")) {
            // Credit card logic
        } else if (method.equals("PAYPAL")) {
            // PayPal logic
        } else if (method.equals("BANK_TRANSFER")) {
            // Bank transfer logic
        }
        // Hard to add new methods, violates OCP
    }
}
```

### Solution: Strategy

```java
// Strategy interface
public interface PaymentStrategy {
    PaymentResult pay(double amount);
}

// Concrete strategies
public class CreditCardStrategy implements PaymentStrategy {
    private String cardNumber;
    
    public PaymentResult pay(double amount) {
        // Credit card payment logic
        return new PaymentResult("SUCCESS", "Card payment processed");
    }
}

public class PayPalStrategy implements PaymentStrategy {
    private String email;
    
    public PaymentResult pay(double amount) {
        // PayPal payment logic
        return new PaymentResult("SUCCESS", "PayPal payment processed");
    }
}

// Context
public class PaymentService {
    private PaymentStrategy strategy;
    
    public PaymentService(PaymentStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void setStrategy(PaymentStrategy strategy) {
        this.strategy = strategy;
    }
    
    public PaymentResult processPayment(double amount) {
        return strategy.pay(amount);
    }
}

// Usage
PaymentService service = new PaymentService(new CreditCardStrategy());
service.processPayment(100.0);

// Switch strategy
service.setStrategy(new PayPalStrategy());
service.processPayment(100.0);
```

### Real-World Example: Sorting Algorithms

```java
public interface SortStrategy {
    void sort(int[] array);
}

public class QuickSortStrategy implements SortStrategy {
    public void sort(int[] array) {
        // Quick sort implementation
    }
}

public class MergeSortStrategy implements SortStrategy {
    public void sort(int[] array) {
        // Merge sort implementation
    }
}

public class Sorter {
    private SortStrategy strategy;
    
    public Sorter(SortStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void sort(int[] array) {
        strategy.sort(array);
    }
}
```

---

## Command Pattern

**Command** encapsulates a request as an object, allowing parameterization and queuing of requests.

### Problem

**Tight coupling between invoker and receiver:**
```java
public class Button {
    private Light light;  // Tight coupling
    
    public void click() {
        light.turnOn();  // Can only turn on light
    }
}
```

### Solution: Command

```java
// Command interface
public interface Command {
    void execute();
    void undo();
}

// Concrete commands
public class LightOnCommand implements Command {
    private Light light;
    
    public LightOnCommand(Light light) {
        this.light = light;
    }
    
    public void execute() {
        light.turnOn();
    }
    
    public void undo() {
        light.turnOff();
    }
}

// Invoker
public class RemoteControl {
    private Command command;
    
    public void setCommand(Command command) {
        this.command = command;
    }
    
    public void pressButton() {
        command.execute();
    }
}

// Usage
Light light = new Light();
Command lightOn = new LightOnCommand(light);
RemoteControl remote = new RemoteControl();
remote.setCommand(lightOn);
remote.pressButton();
```

### Real-World Example: Order Commands

```java
public interface OrderCommand {
    void execute();
    void undo();
}

public class CreateOrderCommand implements OrderCommand {
    private OrderService orderService;
    private Order order;
    private Order createdOrder;
    
    public CreateOrderCommand(OrderService orderService, Order order) {
        this.orderService = orderService;
        this.order = order;
    }
    
    public void execute() {
        createdOrder = orderService.createOrder(order);
    }
    
    public void undo() {
        if (createdOrder != null) {
            orderService.cancelOrder(createdOrder.getId());
        }
    }
}

// Command queue for undo/redo
public class CommandManager {
    private Stack<OrderCommand> history = new Stack<>();
    
    public void execute(OrderCommand command) {
        command.execute();
        history.push(command);
    }
    
    public void undo() {
        if (!history.isEmpty()) {
            OrderCommand command = history.pop();
            command.undo();
        }
    }
}
```

---

## State Pattern

**State** allows an object to alter its behavior when its internal state changes.

### Problem

**State management with if-else:**
```java
public class Order {
    private String state;
    
    public void process() {
        if (state.equals("PENDING")) {
            // Process pending order
            state = "PROCESSING";
        } else if (state.equals("PROCESSING")) {
            // Process processing order
            state = "SHIPPED";
        } else if (state.equals("SHIPPED")) {
            // Can't process shipped order
        }
        // Hard to maintain, violates OCP
    }
}
```

### Solution: State

```java
// State interface
public interface OrderState {
    void process(Order order);
    void cancel(Order order);
    void ship(Order order);
}

// Concrete states
public class PendingState implements OrderState {
    public void process(Order order) {
        System.out.println("Processing order");
        order.setState(new ProcessingState());
    }
    
    public void cancel(Order order) {
        System.out.println("Cancelling order");
        order.setState(new CancelledState());
    }
    
    public void ship(Order order) {
        throw new IllegalStateException("Cannot ship pending order");
    }
}

public class ProcessingState implements OrderState {
    public void process(Order order) {
        throw new IllegalStateException("Order already processing");
    }
    
    public void ship(Order order) {
        System.out.println("Shipping order");
        order.setState(new ShippedState());
    }
}

// Context
public class Order {
    private OrderState state;
    
    public Order() {
        this.state = new PendingState();
    }
    
    public void setState(OrderState state) {
        this.state = state;
    }
    
    public void process() {
        state.process(this);
    }
    
    public void cancel() {
        state.cancel(this);
    }
    
    public void ship() {
        state.ship(this);
    }
}
```

### Real-World Example: ATM State Machine

```java
public interface ATMState {
    void insertCard();
    void enterPin();
    void withdrawCash(double amount);
    void ejectCard();
}

public class NoCardState implements ATMState {
    private ATM atm;
    
    public void insertCard() {
        System.out.println("Card inserted");
        atm.setState(new HasCardState());
    }
    
    public void enterPin() {
        throw new IllegalStateException("No card inserted");
    }
    
    public void withdrawCash(double amount) {
        throw new IllegalStateException("No card inserted");
    }
    
    public void ejectCard() {
        throw new IllegalStateException("No card to eject");
    }
}

public class HasCardState implements ATMState {
    public void enterPin() {
        System.out.println("PIN entered");
        atm.setState(new HasPinState());
    }
    // ... other methods
}
```

---

## Template Method Pattern

**Template Method** defines the skeleton of an algorithm, letting subclasses override specific steps.

### Example

```java
// Abstract class with template method
public abstract class DataProcessor {
    // Template method (defines algorithm structure)
    public final void process() {
        readData();
        processData();
        saveData();
    }
    
    // Steps that subclasses must implement
    protected abstract void readData();
    protected abstract void processData();
    
    // Optional step (hook)
    protected void saveData() {
        // Default implementation
        System.out.println("Saving data");
    }
}

// Concrete implementations
public class CSVProcessor extends DataProcessor {
    protected void readData() {
        System.out.println("Reading CSV");
    }
    
    protected void processData() {
        System.out.println("Processing CSV");
    }
}

public class XMLProcessor extends DataProcessor {
    protected void readData() {
        System.out.println("Reading XML");
    }
    
    protected void processData() {
        System.out.println("Processing XML");
    }
}
```

---

## Chain of Responsibility

**Chain of Responsibility** passes requests along a chain of handlers.

### Example: Request Validation Chain

```java
public abstract class ValidationHandler {
    protected ValidationHandler next;
    
    public void setNext(ValidationHandler next) {
        this.next = next;
    }
    
    public abstract boolean validate(UserRequest request);
    
    protected boolean validateNext(UserRequest request) {
        if (next == null) {
            return true;
        }
        return next.validate(request);
    }
}

public class EmailValidationHandler extends ValidationHandler {
    public boolean validate(UserRequest request) {
        if (!isValidEmail(request.getEmail())) {
            return false;
        }
        return validateNext(request);
    }
}

public class PasswordValidationHandler extends ValidationHandler {
    public boolean validate(UserRequest request) {
        if (request.getPassword().length() < 8) {
            return false;
        }
        return validateNext(request);
    }
}

// Usage
ValidationHandler chain = new EmailValidationHandler();
chain.setNext(new PasswordValidationHandler());
chain.setNext(new AgeValidationHandler());

boolean isValid = chain.validate(request);
```

---

## Real-World Examples

### Example 1: Notification Strategy

```java
public interface NotificationStrategy {
    void send(Notification notification);
}

public class EmailStrategy implements NotificationStrategy {
    public void send(Notification notification) {
        // Send email
    }
}

public class SmsStrategy implements NotificationStrategy {
    public void send(Notification notification) {
        // Send SMS
    }
}

public class NotificationService {
    private NotificationStrategy strategy;
    
    public void setStrategy(NotificationStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void sendNotification(Notification notification) {
        strategy.send(notification);
    }
}
```

---

### Example 2: Order State Machine

```java
public class Order {
    private OrderState state;
    
    public Order() {
        this.state = new PendingState();
    }
    
    public void process() {
        state.process(this);
    }
    
    public void cancel() {
        state.cancel(this);
    }
}
```

---

## Summary

**Key Patterns:**
1. ✅ **Observer:** One-to-many dependency
2. ✅ **Strategy:** Interchangeable algorithms
3. ✅ **Command:** Encapsulate requests
4. ✅ **State:** Behavior changes with state
5. ✅ **Template Method:** Algorithm skeleton
6. ✅ **Chain of Responsibility:** Request chain

**Next Steps:**
- Learn [UML Diagrams](./11_UML_DIAGRAMS.md)
- Understand [Creational Patterns](./08_CREATIONAL_PATTERNS.md)
- Practice [Real-World Problems](./12_PARKING_LOT_SYSTEM.md)

---

**References:**
- [Design Patterns](https://refactoring.guru/design-patterns)
- [SOLID Principles](./03_SOLID_PRINCIPLES.md)

