# OOPS (Object-Oriented Programming) - Complete Guide for Senior Engineers 🎯

> Comprehensive guide covering fundamental to advanced OOP concepts with visual diagrams, real-world examples, and interview preparation

---

## Table of Contents
1. [Core OOP Principles](#core-oop-principles)
2. [Classes and Objects](#classes-and-objects)
3. [Inheritance](#inheritance)
4. [Polymorphism](#polymorphism)
5. [Encapsulation](#encapsulation)
6. [Abstraction](#abstraction)
7. [Advanced Topics](#advanced-topics)
8. [Design Patterns Integration](#design-patterns-integration)
9. [Best Practices](#best-practices)
10. [Interview Questions](#interview-questions)

---

## Core OOP Principles

### The Four Pillars of OOP

```
┌─────────────────────────────────────────────────────────┐
│              FOUR PILLARS OF OOP                        │
└─────────────────────────────────────────────────────────┘

1. ENCAPSULATION
   └─ Data hiding + Methods
   └─ Private fields, Public methods
   └─ Example: BankAccount with private balance

2. INHERITANCE
   └─ IS-A relationship
   └─ Code reuse, hierarchy
   └─ Example: Animal → Dog, Cat

3. POLYMORPHISM
   └─ Many forms
   └─ Runtime (method overriding)
   └─ Compile-time (method overloading)
   └─ Example: Shape.draw() → Circle.draw(), Rectangle.draw()

4. ABSTRACTION
   └─ Hide complexity, show essentials
   └─ Abstract classes, Interfaces
   └─ Example: Vehicle interface, Car implements Vehicle
```

---

## Classes and Objects

### What is a Class?

**Class:** Blueprint or template for creating objects

**Object:** Instance of a class (real-world entity)

### Class Structure

```java
// Class Definition
public class BankAccount {
    // 1. Fields (State/Attributes)
    private String accountNumber;
    private double balance;
    private String ownerName;
    
    // 2. Constructors (Object Creation)
    public BankAccount(String accountNumber, String ownerName) {
        this.accountNumber = accountNumber;
        this.ownerName = ownerName;
        this.balance = 0.0;
    }
    
    // 3. Methods (Behavior)
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
    
    public void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
        }
    }
    
    // 4. Getters and Setters (Accessors)
    public double getBalance() {
        return balance;
    }
    
    public String getAccountNumber() {
        return accountNumber;
    }
}
```

### Object Creation and Memory

```
┌─────────────────────────────────────────────────────────┐
│              OBJECT CREATION IN MEMORY                  │
└─────────────────────────────────────────────────────────┘

Stack Memory                    Heap Memory
┌─────────────┐                ┌──────────────────────┐
│   account1  │ ────────────→  │  BankAccount Object  │
│  (reference)│                │  ┌────────────────┐  │
└─────────────┘                │  │ accountNumber  │  │
                               │  │ "ACC001"       │  │
                               │  ├────────────────┤  │
                               │  │ balance: 1000  │  │
                               │  ├────────────────┤  │
                               │  │ ownerName      │  │
                               │  │ "John Doe"     │  │
                               │  └────────────────┘  │
                               └──────────────────────┘

BankAccount account1 = new BankAccount("ACC001", "John Doe");
```

### Constructor Types

#### 1. Default Constructor
```java
public class BankAccount {
    private double balance;
    
    // Default constructor (provided by compiler if no constructor defined)
    public BankAccount() {
        balance = 0.0;
    }
}
```

#### 2. Parameterized Constructor
```java
public BankAccount(String accountNumber, String ownerName, double initialBalance) {
    this.accountNumber = accountNumber;
    this.ownerName = ownerName;
    this.balance = initialBalance;
}
```

#### 3. Constructor Chaining
```java
public class BankAccount {
    private String accountNumber;
    private String ownerName;
    private double balance;
    
    // Constructor 1: Full parameters
    public BankAccount(String accountNumber, String ownerName, double balance) {
        this.accountNumber = accountNumber;
        this.ownerName = ownerName;
        this.balance = balance;
    }
    
    // Constructor 2: Calls Constructor 1
    public BankAccount(String accountNumber, String ownerName) {
        this(accountNumber, ownerName, 0.0);  // Constructor chaining
    }
    
    // Constructor 3: Calls Constructor 2
    public BankAccount(String accountNumber) {
        this(accountNumber, "Unknown", 0.0);
    }
}
```

#### 4. Copy Constructor
```java
public BankAccount(BankAccount other) {
    this.accountNumber = other.accountNumber;
    this.ownerName = other.ownerName;
    this.balance = other.balance;
}
```

### Static vs Instance Members

```
┌─────────────────────────────────────────────────────────┐
│         STATIC vs INSTANCE MEMBERS                      │
└─────────────────────────────────────────────────────────┘

STATIC (Class-level)              INSTANCE (Object-level)
┌─────────────────────┐          ┌─────────────────────┐
│ Shared by all       │          │ Unique per object   │
│ objects             │          │                     │
│                     │          │                     │
│ ClassName.method()  │          │ object.method()     │
│                     │          │                     │
│ Memory: 1 copy      │          │ Memory: N copies    │
└─────────────────────┘          └─────────────────────┘
```

**Example:**
```java
public class Counter {
    // Instance variable (each object has its own)
    private int instanceCount = 0;
    
    // Static variable (shared by all objects)
    private static int staticCount = 0;
    
    public void increment() {
        instanceCount++;   // Each object has its own
        staticCount++;     // Shared across all objects
    }
    
    public int getInstanceCount() {
        return instanceCount;
    }
    
    public static int getStaticCount() {
        return staticCount;  // Can access without object
    }
}

// Usage
Counter c1 = new Counter();
Counter c2 = new Counter();

c1.increment();  // instanceCount=1, staticCount=1
c2.increment();  // instanceCount=1, staticCount=2

c1.getInstanceCount();  // 1
c2.getInstanceCount();  // 1
Counter.getStaticCount(); // 2 (shared)
```

---

## Inheritance

### What is Inheritance?

**Inheritance:** Mechanism where a child class acquires properties and behaviors from a parent class

### Inheritance Hierarchy

```
┌─────────────────────────────────────────────────────────┐
│              INHERITANCE HIERARCHY                       │
└─────────────────────────────────────────────────────────┘

                    Animal (Parent/Base/Super class)
                    ├─ name: String
                    ├─ age: int
                    └─ makeSound(): void
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼
      Dog              Cat              Bird (Child/Derived/Sub classes)
      ├─ breed         ├─ color          ├─ canFly
      └─ bark()        └─ meow()         └─ fly()
```

### Types of Inheritance

#### 1. Single Inheritance
```java
class Animal {
    protected String name;
    
    public void eat() {
        System.out.println(name + " is eating");
    }
}

class Dog extends Animal {
    private String breed;
    
    public void bark() {
        System.out.println(name + " is barking");
    }
}
```

#### 2. Multilevel Inheritance
```java
class Animal {
    protected String name;
}

class Mammal extends Animal {
    protected boolean hasFur;
}

class Dog extends Mammal {
    private String breed;
    
    // Dog has: name (from Animal), hasFur (from Mammal), breed (own)
}
```

#### 3. Hierarchical Inheritance
```java
class Animal {
    protected String name;
}

class Dog extends Animal {
    // Dog-specific
}

class Cat extends Animal {
    // Cat-specific
}

class Bird extends Animal {
    // Bird-specific
}
```

#### 4. Multiple Inheritance (via Interfaces)
```java
// Java doesn't support multiple inheritance for classes
// But supports it via interfaces

interface Flyable {
    void fly();
}

interface Swimmable {
    void swim();
}

class Duck implements Flyable, Swimmable {
    public void fly() {
        System.out.println("Duck is flying");
    }
    
    public void swim() {
        System.out.println("Duck is swimming");
    }
}
```

### Method Overriding

**Overriding:** Child class provides specific implementation of parent's method

```java
class Animal {
    public void makeSound() {
        System.out.println("Animal makes a sound");
    }
}

class Dog extends Animal {
    @Override  // Annotation ensures we're overriding
    public void makeSound() {
        System.out.println("Dog barks: Woof!");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Cat meows: Meow!");
    }
}
```

**Rules for Method Overriding:**
1. Method name and signature must match
2. Return type must be same or covariant (subtype)
3. Access modifier can't be more restrictive
4. Can't override static, final, or private methods
5. Must use `@Override` annotation

### super Keyword

```java
class Animal {
    protected String name;
    
    public Animal(String name) {
        this.name = name;
        System.out.println("Animal constructor");
    }
    
    public void eat() {
        System.out.println(name + " is eating");
    }
}

class Dog extends Animal {
    private String breed;
    
    public Dog(String name, String breed) {
        super(name);  // Call parent constructor
        this.breed = breed;
        System.out.println("Dog constructor");
    }
    
    @Override
    public void eat() {
        super.eat();  // Call parent method
        System.out.println(name + " is eating dog food");
    }
}
```

### final Keyword

```java
// final class: Cannot be inherited
final class MathUtils {
    // Utility class - no inheritance needed
}

// final method: Cannot be overridden
class Animal {
    public final void breathe() {
        System.out.println("Breathing...");
    }
}

class Dog extends Animal {
    // Cannot override breathe()
    // public void breathe() { }  // ❌ Compile error
}

// final variable: Cannot be reassigned
class Constants {
    public static final double PI = 3.14159;
    // PI = 3.14;  // ❌ Compile error
}
```

---

## Polymorphism

### What is Polymorphism?

**Polymorphism:** Ability of objects to take multiple forms

**Types:**
1. **Compile-time Polymorphism** (Method Overloading)
2. **Runtime Polymorphism** (Method Overriding)

### Compile-time Polymorphism (Method Overloading)

**Overloading:** Multiple methods with same name but different parameters

```java
class Calculator {
    // Method 1: Add two integers
    public int add(int a, int b) {
        return a + b;
    }
    
    // Method 2: Add three integers
    public int add(int a, int b, int c) {
        return a + b + c;
    }
    
    // Method 3: Add two doubles
    public double add(double a, double b) {
        return a + b;
    }
    
    // Method 4: Add array of integers
    public int add(int[] numbers) {
        int sum = 0;
        for (int num : numbers) {
            sum += num;
        }
        return sum;
    }
}

// Usage
Calculator calc = new Calculator();
calc.add(5, 3);           // Calls Method 1
calc.add(5, 3, 2);        // Calls Method 2
calc.add(5.5, 3.2);       // Calls Method 3
calc.add(new int[]{1,2,3}); // Calls Method 4
```

**Rules for Method Overloading:**
1. Methods must have same name
2. Must differ in:
   - Number of parameters, OR
   - Type of parameters, OR
   - Order of parameters
3. Return type doesn't matter (can't overload based on return type alone)
4. Access modifier doesn't matter

### Runtime Polymorphism (Method Overriding)

**Visual Representation:**

```
┌─────────────────────────────────────────────────────────┐
│         RUNTIME POLYMORPHISM (Method Overriding)       │
└─────────────────────────────────────────────────────────┘

Reference Type: Animal
Actual Object: Dog

Animal animal = new Dog();
animal.makeSound();  // Calls Dog's makeSound() at runtime

┌──────────────┐
│ Animal ref   │ ────┐
└──────────────┘     │
                     │ Points to
                     ▼
              ┌──────────────┐
              │  Dog object │
              │  makeSound() │ ──→ "Woof!" (Dog's implementation)
              └──────────────┘
```

**Example:**
```java
class Shape {
    public void draw() {
        System.out.println("Drawing a shape");
    }
    
    public double calculateArea() {
        return 0.0;
    }
}

class Circle extends Shape {
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

class Rectangle extends Shape {
    private double width, height;
    
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

// Runtime Polymorphism in action
Shape[] shapes = {
    new Circle(5.0),
    new Rectangle(4.0, 6.0),
    new Circle(3.0)
};

for (Shape shape : shapes) {
    shape.draw();              // Calls appropriate draw() at runtime
    System.out.println("Area: " + shape.calculateArea());
}
```

**Output:**
```
Drawing a circle with radius 5.0
Area: 78.53981633974483
Drawing a rectangle 4.0x6.0
Area: 24.0
Drawing a circle with radius 3.0
Area: 28.274333882308138
```

### Virtual Method Invocation

**How it works:**

```
┌─────────────────────────────────────────────────────────┐
│         VIRTUAL METHOD INVOCATION                        │
└─────────────────────────────────────────────────────────┘

1. Compile Time:
   - Compiler checks if method exists in reference type
   - Animal.makeSound() exists? ✅

2. Runtime:
   - JVM looks at actual object type
   - Dog object? → Call Dog.makeSound()
   - Cat object? → Call Cat.makeSound()

Animal animal = new Dog();
animal.makeSound();  // Virtual method call
```

---

## Encapsulation

### What is Encapsulation?

**Encapsulation:** Bundling data and methods together, hiding internal details

### Access Modifiers

```
┌─────────────────────────────────────────────────────────┐
│              ACCESS MODIFIERS IN JAVA                    │
└─────────────────────────────────────────────────────────┘

Modifier    Same Class  Same Package  Subclass  Other Package
────────────────────────────────────────────────────────────
private     ✅          ❌            ❌        ❌
(default)   ✅          ✅            ❌        ❌
protected   ✅          ✅            ✅        ❌
public      ✅          ✅            ✅        ✅
```

### Encapsulation Example

```java
public class BankAccount {
    // Private fields (data hiding)
    private String accountNumber;
    private double balance;
    private String ownerName;
    
    // Public constructor
    public BankAccount(String accountNumber, String ownerName) {
        this.accountNumber = accountNumber;
        this.ownerName = ownerName;
        this.balance = 0.0;
    }
    
    // Public methods (controlled access)
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
            logTransaction("Deposit", amount);
        } else {
            throw new IllegalArgumentException("Amount must be positive");
        }
    }
    
    public void withdraw(double amount) {
        if (amount > 0 && amount <= balance) {
            balance -= amount;
            logTransaction("Withdraw", amount);
        } else {
            throw new IllegalArgumentException("Invalid withdrawal amount");
        }
    }
    
    // Public getter (read-only access)
    public double getBalance() {
        return balance;
    }
    
    public String getAccountNumber() {
        return accountNumber;
    }
    
    // Private helper method (internal implementation)
    private void logTransaction(String type, double amount) {
        System.out.println(type + ": " + amount + " | Balance: " + balance);
    }
}
```

### Benefits of Encapsulation

1. **Data Protection:** Prevents unauthorized access
2. **Flexibility:** Can change implementation without affecting clients
3. **Validation:** Can add validation in setters
4. **Maintainability:** Easier to maintain and debug

---

## Abstraction

### What is Abstraction?

**Abstraction:** Hiding implementation details, showing only essential features

### Abstract Classes

```java
// Abstract class: Cannot be instantiated
abstract class Animal {
    protected String name;
    
    // Concrete method (has implementation)
    public void eat() {
        System.out.println(name + " is eating");
    }
    
    // Abstract method (no implementation, must be overridden)
    public abstract void makeSound();
    
    // Can have constructor
    public Animal(String name) {
        this.name = name;
    }
}

// Concrete class: Must implement abstract methods
class Dog extends Animal {
    public Dog(String name) {
        super(name);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " barks: Woof!");
    }
}

class Cat extends Animal {
    public Cat(String name) {
        super(name);
    }
    
    @Override
    public void makeSound() {
        System.out.println(name + " meows: Meow!");
    }
}
```

### Interfaces

```java
// Interface: Contract that classes must follow
interface Flyable {
    // All methods are public abstract by default
    void fly();
    
    // Default method (Java 8+)
    default void takeOff() {
        System.out.println("Taking off...");
    }
    
    // Static method (Java 8+)
    static void showInfo() {
        System.out.println("This is a flyable object");
    }
}

interface Swimmable {
    void swim();
}

// Class implements interface
class Duck implements Flyable, Swimmable {
    @Override
    public void fly() {
        System.out.println("Duck is flying");
    }
    
    @Override
    public void swim() {
        System.out.println("Duck is swimming");
    }
}
```

### Abstract Class vs Interface

#### Detailed Comparison

```
┌─────────────────────────────────────────────────────────┐
│      ABSTRACT CLASS vs INTERFACE - DETAILED             │
└─────────────────────────────────────────────────────────┘

FEATURE                    ABSTRACT CLASS          INTERFACE
──────────────────────────────────────────────────────────────────
Instance Variables        ✅ Yes                  ❌ No (only constants)
Constructors              ✅ Yes                  ❌ No
Concrete Methods          ✅ Yes                  ✅ Yes (default methods Java 8+)
Abstract Methods          ✅ Yes                  ✅ Yes (implicitly abstract)
Static Methods            ✅ Yes                  ✅ Yes (Java 8+)
Default Methods           ❌ No                   ✅ Yes (Java 8+)
Private Methods           ✅ Yes                  ✅ Yes (Java 9+)
Multiple Inheritance      ❌ No (single)         ✅ Yes (multiple)
Access Modifiers          Any modifier           Public (implicitly)
Keyword                   "extends"              "implements"
Instantiation             ❌ Cannot instantiate   ❌ Cannot instantiate
```

#### When to Use Abstract Class

**Use Abstract Class When:**

1. **IS-A Relationship with Shared Code**
   ```java
   // Abstract class: Animals share common behavior
   abstract class Animal {
       protected String name;  // Shared state
       
       // Shared implementation
       public void breathe() {
           System.out.println(name + " is breathing");
       }
       
       // Must be implemented by subclasses
       public abstract void makeSound();
   }
   
   class Dog extends Animal {
       @Override
       public void makeSound() {
           System.out.println("Woof!");
       }
   }
   ```

2. **Need Constructor for Initialization**
   ```java
   abstract class Vehicle {
       protected String brand;
       protected int year;
       
       // Constructor to initialize common fields
       public Vehicle(String brand, int year) {
           this.brand = brand;
           this.year = year;
       }
       
       public abstract void start();
   }
   ```

3. **Need to Control Access with Access Modifiers**
   ```java
   abstract class BaseProcessor {
       private String config;  // Private field
       protected String state; // Protected field
       
       protected abstract void process();
   }
   ```

4. **Want to Provide Default Implementation**
   ```java
   abstract class DataProcessor {
       // Default implementation
       public void process() {
           validate();
           transform();
           save();
       }
       
       protected abstract void transform();
       
       private void validate() { /* common validation */ }
       private void save() { /* common save logic */ }
   }
   ```

5. **Single Inheritance is Sufficient**
   ```java
   // Animal hierarchy - single inheritance makes sense
   abstract class Animal { }
   abstract class Mammal extends Animal { }
   class Dog extends Mammal { }
   ```

**Real-World Example:**
```java
// E-commerce: Payment processing
abstract class PaymentProcessor {
    protected String merchantId;
    protected double fee;
    
    public PaymentProcessor(String merchantId, double fee) {
        this.merchantId = merchantId;
        this.fee = fee;
    }
    
    // Shared implementation
    public final PaymentResult processPayment(PaymentRequest request) {
        validateRequest(request);
        PaymentResult result = executePayment(request);
        logTransaction(result);
        return result;
    }
    
    // Must be implemented by subclasses
    protected abstract PaymentResult executePayment(PaymentRequest request);
    
    // Shared helper methods
    protected void validateRequest(PaymentRequest request) {
        if (request.getAmount() <= 0) {
            throw new IllegalArgumentException("Invalid amount");
        }
    }
    
    private void logTransaction(PaymentResult result) {
        // Common logging logic
    }
}

class CreditCardProcessor extends PaymentProcessor {
    public CreditCardProcessor(String merchantId) {
        super(merchantId, 0.02); // 2% fee
    }
    
    @Override
    protected PaymentResult executePayment(PaymentRequest request) {
        // Credit card specific implementation
        return new PaymentResult("SUCCESS", "Transaction ID: " + UUID.randomUUID());
    }
}
```

#### When to Use Interface

**Use Interface When:**

1. **CAN-DO Relationship (Capabilities)**
   ```java
   // Interface: Defines what an object CAN DO
   interface Flyable {
       void fly();
   }
   
   interface Swimmable {
       void swim();
   }
   
   // Duck CAN fly AND swim
   class Duck implements Flyable, Swimmable {
       public void fly() { System.out.println("Flying"); }
       public void swim() { System.out.println("Swimming"); }
   }
   ```

2. **Multiple Behaviors Needed**
   ```java
   // Class can implement multiple interfaces
   interface Readable {
       String read();
   }
   
   interface Writable {
       void write(String data);
   }
   
   interface Closable {
       void close();
   }
   
   class FileHandler implements Readable, Writable, Closable {
       // Implements all three behaviors
   }
   ```

3. **Defining Contract/API**
   ```java
   // Interface defines contract for service
   interface UserService {
       User createUser(UserRequest request);
       User getUserById(Long id);
       void updateUser(Long id, UserRequest request);
       void deleteUser(Long id);
   }
   
   // Multiple implementations possible
   class DatabaseUserService implements UserService { }
   class InMemoryUserService implements UserService { }
   ```

4. **Callback/Event Handling**
   ```java
   interface EventListener {
       void onEvent(Event event);
   }
   
   class Button {
       private List<EventListener> listeners = new ArrayList<>();
       
       public void addListener(EventListener listener) {
           listeners.add(listener);
       }
       
       public void click() {
           Event event = new Event("click");
           listeners.forEach(l -> l.onEvent(event));
       }
   }
   ```

5. **Strategy Pattern Implementation**
   ```java
   interface SortingStrategy {
       void sort(int[] array);
   }
   
   class QuickSort implements SortingStrategy { }
   class MergeSort implements SortingStrategy { }
   class BubbleSort implements SortingStrategy { }
   ```

**Real-World Example:**
```java
// E-commerce: Payment methods
interface PaymentMethod {
    PaymentResult pay(double amount);
    boolean isAvailable();
    String getProviderName();
}

class CreditCardPayment implements PaymentMethod {
    private String cardNumber;
    
    public PaymentResult pay(double amount) {
        // Credit card payment logic
        return new PaymentResult("SUCCESS");
    }
    
    public boolean isAvailable() {
        return cardNumber != null && !cardNumber.isEmpty();
    }
    
    public String getProviderName() {
        return "Credit Card";
    }
}

class PayPalPayment implements PaymentMethod {
    private String email;
    
    public PaymentResult pay(double amount) {
        // PayPal payment logic
        return new PaymentResult("SUCCESS");
    }
    
    public boolean isAvailable() {
        return email != null && email.contains("@");
    }
    
    public String getProviderName() {
        return "PayPal";
    }
}

// Usage: Can easily add new payment methods
class PaymentService {
    private List<PaymentMethod> paymentMethods;
    
    public void processPayment(double amount, String methodType) {
        PaymentMethod method = paymentMethods.stream()
            .filter(m -> m.getProviderName().equals(methodType))
            .findFirst()
            .orElseThrow();
        
        if (method.isAvailable()) {
            method.pay(amount);
        }
    }
}
```

#### Decision Matrix: Abstract Class vs Interface

```
┌─────────────────────────────────────────────────────────┐
│         DECISION MATRIX                                 │
└─────────────────────────────────────────────────────────┘

QUESTION                          ANSWER
──────────────────────────────────────────────────────────
Need to share code?              → Abstract Class
Need multiple inheritance?       → Interface
IS-A relationship?               → Abstract Class
CAN-DO relationship?             → Interface
Need constructors?                → Abstract Class
Need instance variables?         → Abstract Class
Defining contract/API?           → Interface
Need default implementation?      → Abstract Class
Want loose coupling?             → Interface
Need access modifiers?            → Abstract Class
```

#### Java 8+ Interface Enhancements

**Default Methods:**
```java
interface Vehicle {
    void start();
    
    // Default method - provides implementation
    default void stop() {
        System.out.println("Vehicle stopped");
    }
    
    // Can be overridden
    default void honk() {
        System.out.println("Beep beep!");
    }
}

class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car started");
    }
    
    // Can override default method
    @Override
    public void honk() {
        System.out.println("Car horn: Honk honk!");
    }
    
    // stop() uses default implementation
}
```

**Static Methods:**
```java
interface MathUtils {
    static int add(int a, int b) {
        return a + b;
    }
    
    static double calculateArea(double radius) {
        return Math.PI * radius * radius;
    }
}

// Usage: Called on interface, not implementation
int sum = MathUtils.add(5, 3);
double area = MathUtils.calculateArea(5.0);
```

**Private Methods (Java 9+):**
```java
interface DataProcessor {
    default void process() {
        validate();
        transform();
        save();
    }
    
    // Private helper method - can't be accessed outside
    private void validate() {
        System.out.println("Validating...");
    }
    
    private void transform() {
        System.out.println("Transforming...");
    }
    
    private void save() {
        System.out.println("Saving...");
    }
}
```

#### Best Practices: Abstract Class vs Interface

**✅ DO:**
- Use abstract class for IS-A relationships with shared code
- Use interface for CAN-DO relationships and contracts
- Use interface for multiple behaviors
- Use default methods in interfaces for backward compatibility
- Use abstract class when you need constructors or instance variables

**❌ DON'T:**
- Don't use abstract class just to prevent instantiation (use private constructor)
- Don't use interface when you need to share implementation code
- Don't create "fat" interfaces (follow Interface Segregation Principle)
- Don't use abstract class for multiple inheritance scenarios
- Don't mix abstract class and interface unnecessarily

#### Hybrid Approach: Abstract Class Implementing Interface

```java
// Interface defines contract
interface Drawable {
    void draw();
    void resize(double factor);
}

// Abstract class provides common implementation
abstract class Shape implements Drawable {
    protected String color;
    protected double area;
    
    public Shape(String color) {
        this.color = color;
    }
    
    // Common implementation
    @Override
    public void resize(double factor) {
        this.area *= factor;
        System.out.println("Resized by factor: " + factor);
    }
    
    // Must be implemented by subclasses
    @Override
    public abstract void draw();
    
    public String getColor() {
        return color;
    }
}

// Concrete implementation
class Circle extends Shape {
    private double radius;
    
    public Circle(String color, double radius) {
        super(color);
        this.radius = radius;
        this.area = Math.PI * radius * radius;
    }
    
    @Override
    public void draw() {
        System.out.println("Drawing circle with radius " + radius);
    }
}
```

---

## Advanced Topics

### Composition vs Inheritance

#### Composition (HAS-A Relationship)

**Composition:** Object contains another object as a member

```java
// Composition Example
class Engine {
    public void start() {
        System.out.println("Engine started");
    }
}

class Car {
    private Engine engine;  // HAS-A relationship
    
    public Car() {
        this.engine = new Engine();  // Composition
    }
    
    public void start() {
        engine.start();
        System.out.println("Car is ready to drive");
    }
}
```

#### Inheritance (IS-A Relationship)

**Inheritance:** Child class is a type of parent class

```java
// Inheritance Example
class Vehicle {
    public void start() {
        System.out.println("Vehicle started");
    }
}

class Car extends Vehicle {  // IS-A relationship
    @Override
    public void start() {
        super.start();
        System.out.println("Car is ready to drive");
    }
}
```

#### When to Use What?

```
┌─────────────────────────────────────────────────────────┐
│         COMPOSITION vs INHERITANCE                      │
└─────────────────────────────────────────────────────────┘

Use INHERITANCE when:              Use COMPOSITION when:
──────────────────────────────────────────────────────────
- IS-A relationship                - HAS-A relationship
- Need polymorphism                - Need flexibility
- Tight coupling is OK             - Loose coupling needed
- Code reuse is primary goal        - Behavior reuse needed
- Example: Dog IS-A Animal         - Example: Car HAS-A Engine
```

**Favor Composition Over Inheritance:**
- More flexible
- Easier to test
- Avoids inheritance hierarchy issues
- Follows "Program to interface, not implementation"

### Method Resolution Order (MRO)

**How Java resolves method calls:**

```
┌─────────────────────────────────────────────────────────┐
│         METHOD RESOLUTION ORDER                          │
└─────────────────────────────────────────────────────────┘

1. Check actual object type (runtime)
2. Look for method in that class
3. If not found, check parent class
4. Continue up inheritance chain
5. If not found, compile error
```

**Example:**
```java
class A {
    public void method() {
        System.out.println("A.method()");
    }
}

class B extends A {
    @Override
    public void method() {
        System.out.println("B.method()");
    }
}

class C extends B {
    // No method() override
}

A obj = new C();
obj.method();  // Calls B.method() (nearest override in hierarchy)
```

### Covariant Return Types

**Covariant Return Type:** Overriding method can return subtype of parent's return type

```java
class Animal {
    public Animal getAnimal() {
        return new Animal();
    }
}

class Dog extends Animal {
    @Override
    public Dog getAnimal() {  // Covariant return type
        return new Dog();     // Returns Dog (subtype of Animal)
    }
}
```

### Method Hiding (Static Methods)

**Static methods are hidden, not overridden:**

```java
class Parent {
    public static void staticMethod() {
        System.out.println("Parent.staticMethod()");
    }
    
    public void instanceMethod() {
        System.out.println("Parent.instanceMethod()");
    }
}

class Child extends Parent {
    // Method hiding (not overriding)
    public static void staticMethod() {
        System.out.println("Child.staticMethod()");
    }
    
    // Method overriding
    @Override
    public void instanceMethod() {
        System.out.println("Child.instanceMethod()");
    }
}

Parent p = new Child();
p.staticMethod();      // Calls Parent.staticMethod() (compile-time)
p.instanceMethod();    // Calls Child.instanceMethod() (runtime)
```

### Inner Classes

#### Types of Inner Classes

**1. Non-static Inner Class (Member Inner Class)**
```java
class Outer {
    private int outerVar = 10;
    
    class Inner {
        private int innerVar = 20;
        
        public void display() {
            System.out.println("Outer: " + outerVar);  // Can access outer members
            System.out.println("Inner: " + innerVar);
        }
    }
}

// Usage
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();
inner.display();
```

**2. Static Nested Class**
```java
class Outer {
    private static int outerVar = 10;
    
    static class Nested {
        public void display() {
            System.out.println("Outer: " + outerVar);  // Can access static members
        }
    }
}

// Usage
Outer.Nested nested = new Outer.Nested();
nested.display();
```

**3. Local Inner Class**
```java
class Outer {
    public void method() {
        class Local {
            public void display() {
                System.out.println("Local inner class");
            }
        }
        
        Local local = new Local();
        local.display();
    }
}
```

**4. Anonymous Inner Class**
```java
interface Greeting {
    void greet();
}

class Outer {
    public void method() {
        Greeting greeting = new Greeting() {  // Anonymous inner class
            @Override
            public void greet() {
                System.out.println("Hello from anonymous class");
            }
        };
        
        greeting.greet();
    }
}
```

### Lambda Expressions and Functional Interfaces

**Functional Interface:** Interface with exactly one abstract method

```java
@FunctionalInterface
interface Calculator {
    int calculate(int a, int b);
}

// Lambda expression
Calculator add = (a, b) -> a + b;
Calculator multiply = (a, b) -> a * b;

System.out.println(add.calculate(5, 3));        // 8
System.out.println(multiply.calculate(5, 3));  // 15
```

**Built-in Functional Interfaces:**
```java
// Predicate: Takes one argument, returns boolean
Predicate<Integer> isEven = n -> n % 2 == 0;

// Function: Takes one argument, returns result
Function<String, Integer> length = s -> s.length();

// Consumer: Takes one argument, returns void
Consumer<String> print = s -> System.out.println(s);

// Supplier: Takes no argument, returns result
Supplier<Double> random = () -> Math.random();
```

### Generics

**Generics:** Type parameters for classes, interfaces, and methods

```java
// Generic class
class Box<T> {
    private T item;
    
    public void setItem(T item) {
        this.item = item;
    }
    
    public T getItem() {
        return item;
    }
}

// Usage
Box<String> stringBox = new Box<>();
stringBox.setItem("Hello");
String value = stringBox.getItem();

Box<Integer> intBox = new Box<>();
intBox.setItem(42);
Integer number = intBox.getItem();
```

**Bounded Type Parameters:**
```java
// Upper bound: T must be Number or its subclass
class NumberBox<T extends Number> {
    private T number;
    
    public double getDoubleValue() {
        return number.doubleValue();
    }
}

// Multiple bounds
interface A { }
interface B { }
class C<T extends Number & A & B> { }
```

**Wildcards:**
```java
// Unbounded wildcard
List<?> list;

// Upper bounded wildcard
List<? extends Number> numbers;  // Number or subclass

// Lower bounded wildcard
List<? super Integer> integers;  // Integer or superclass
```

### Enums

**Enum:** Special class representing a group of constants

```java
enum Day {
    MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY, SUNDAY
}

// Enum with methods and fields
enum Status {
    PENDING("Waiting"),
    IN_PROGRESS("Processing"),
    COMPLETED("Done"),
    FAILED("Error");
    
    private final String description;
    
    Status(String description) {
        this.description = description;
    }
    
    public String getDescription() {
        return description;
    }
}

// Usage
Status status = Status.PENDING;
System.out.println(status.getDescription());  // "Waiting"
```

### Records (Java 14+)

**Record:** Immutable data carrier class

```java
// Traditional class
class Person {
    private final String name;
    private final int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() { return name; }
    public int getAge() { return age; }
    
    @Override
    public boolean equals(Object o) { ... }
    @Override
    public int hashCode() { ... }
    @Override
    public String toString() { ... }
}

// Record (equivalent, but concise)
record Person(String name, int age) {
    // Automatically generates:
    // - Constructor
    // - Getters (name(), age())
    // - equals(), hashCode(), toString()
}

// Usage
Person person = new Person("John", 30);
System.out.println(person.name());  // "John"
```

### Sealed Classes (Java 17+)

**Sealed Class:** Restricts which classes can extend it

```java
// Sealed class: Only specified classes can extend
public sealed class Shape 
    permits Circle, Rectangle, Triangle {
    
    public abstract double area();
}

// Permitted subclasses
public final class Circle extends Shape {
    private double radius;
    
    @Override
    public double area() {
        return Math.PI * radius * radius;
    }
}

public final class Rectangle extends Shape {
    private double width, height;
    
    @Override
    public double area() {
        return width * height;
    }
}
```

### Reflection

**Reflection:** Ability to inspect and modify classes, methods, fields at runtime

**Use Cases:**
- Frameworks (Spring, Hibernate)
- Testing frameworks (JUnit)
- Serialization/Deserialization
- Dynamic proxy creation

**Example:**
```java
import java.lang.reflect.*;

class Person {
    private String name;
    private int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    public String getName() {
        return name;
    }
    
    private void setAge(int age) {
        this.age = age;
    }
}

// Reflection usage
public class ReflectionExample {
    public static void main(String[] args) throws Exception {
        // Get class
        Class<?> personClass = Person.class;
        
        // Get constructors
        Constructor<?> constructor = personClass.getConstructor(String.class, int.class);
        Person person = (Person) constructor.newInstance("John", 30);
        
        // Get methods
        Method getNameMethod = personClass.getMethod("getName");
        String name = (String) getNameMethod.invoke(person);
        System.out.println("Name: " + name);
        
        // Get private method
        Method setAgeMethod = personClass.getDeclaredMethod("setAge", int.class);
        setAgeMethod.setAccessible(true);  // Access private method
        setAgeMethod.invoke(person, 35);
        
        // Get fields
        Field nameField = personClass.getDeclaredField("name");
        nameField.setAccessible(true);
        nameField.set(person, "Jane");
        
        // Get annotations
        Annotation[] annotations = personClass.getAnnotations();
    }
}
```

**When to Use:**
- Framework development
- Testing
- Debugging tools
- Code analysis tools

**When NOT to Use:**
- Regular application code (performance overhead)
- Security-sensitive code (can bypass access controls)

### Annotations

**Annotations:** Metadata about code elements

**Built-in Annotations:**
```java
@Override      // Indicates method overrides parent method
@Deprecated    // Marks element as deprecated
@SuppressWarnings("unchecked")  // Suppresses compiler warnings
@FunctionalInterface  // Marks interface as functional (one abstract method)
```

**Custom Annotations:**
```java
// Define annotation
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@interface LogExecution {
    String value() default "";
    boolean logParameters() default true;
}

// Use annotation
class Service {
    @LogExecution(value = "UserService", logParameters = true)
    public void processUser(User user) {
        // Method implementation
    }
}

// Process annotation
class AnnotationProcessor {
    public static void processAnnotations(Object obj) {
        Class<?> clazz = obj.getClass();
        Method[] methods = clazz.getDeclaredMethods();
        
        for (Method method : methods) {
            if (method.isAnnotationPresent(LogExecution.class)) {
                LogExecution annotation = method.getAnnotation(LogExecution.class);
                System.out.println("Logging: " + annotation.value());
            }
        }
    }
}
```

### Proxy Pattern

**Proxy:** Provides a placeholder for another object

**Static Proxy:**
```java
interface Image {
    void display();
}

class RealImage implements Image {
    private String filename;
    
    public RealImage(String filename) {
        this.filename = filename;
        loadFromDisk();
    }
    
    private void loadFromDisk() {
        System.out.println("Loading " + filename);
    }
    
    public void display() {
        System.out.println("Displaying " + filename);
    }
}

class ImageProxy implements Image {
    private RealImage realImage;
    private String filename;
    
    public ImageProxy(String filename) {
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

**Dynamic Proxy (Java):**
```java
import java.lang.reflect.*;

interface UserService {
    void saveUser(User user);
    void deleteUser(Long id);
}

class UserServiceImpl implements UserService {
    public void saveUser(User user) {
        System.out.println("Saving user: " + user.getName());
    }
    
    public void deleteUser(Long id) {
        System.out.println("Deleting user: " + id);
    }
}

class LoggingInvocationHandler implements InvocationHandler {
    private Object target;
    
    public LoggingInvocationHandler(Object target) {
        this.target = target;
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before method: " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("After method: " + method.getName());
        return result;
    }
}

// Usage
UserService realService = new UserServiceImpl();
UserService proxy = (UserService) Proxy.newProxyInstance(
    UserService.class.getClassLoader(),
    new Class[]{UserService.class},
    new LoggingInvocationHandler(realService)
);

proxy.saveUser(new User("John"));  // Logs before and after
```

### Method References

**Method References:** Shorthand for lambda expressions

**Types:**
```java
// 1. Static method reference
Function<String, Integer> parseInt = Integer::parseInt;

// 2. Instance method reference
String str = "Hello";
Supplier<Integer> length = str::length;

// 3. Instance method of arbitrary object
Function<String, String> toUpperCase = String::toUpperCase;

// 4. Constructor reference
Supplier<List<String>> listSupplier = ArrayList::new;
Function<Integer, List<String>> listWithSize = ArrayList::new;
```

**Example:**
```java
List<String> names = Arrays.asList("John", "Jane", "Bob");

// Lambda
names.forEach(name -> System.out.println(name));

// Method reference
names.forEach(System.out::println);

// Lambda
names.stream().map(name -> name.toUpperCase());

// Method reference
names.stream().map(String::toUpperCase);
```

### Type Erasure and Generics

**Type Erasure:** Generic type information is removed at compile time

```java
// At compile time
List<String> stringList = new ArrayList<>();
List<Integer> intList = new ArrayList<>();

// After type erasure (runtime)
List stringList = new ArrayList();  // Raw type
List intList = new ArrayList();     // Raw type

// Both are same type at runtime!
System.out.println(stringList.getClass() == intList.getClass());  // true
```

**Implications:**
- Can't use `instanceof` with generic types
- Can't create arrays of generic types
- Can't catch generic exceptions

**Workarounds:**
```java
// Use Class object for type checking
public <T> void process(List<T> list, Class<T> clazz) {
    if (clazz == String.class) {
        // Handle String
    }
}

// Use wildcards for flexibility
public void process(List<? extends Number> numbers) {
    // Can read as Number
}
```

---

## Design Patterns Integration

### Creational Patterns

#### 1. Singleton Pattern

**Purpose:** Ensure only one instance of a class exists

**Use Cases:**
- Database connections
- Logger instances
- Configuration managers
- Cache managers

**Implementation 1: Eager Initialization**
```java
class Singleton {
    // Eager initialization - created at class loading
    private static final Singleton instance = new Singleton();
    
    // Private constructor prevents instantiation
    private Singleton() {
        // Initialization code
    }
    
    public static Singleton getInstance() {
        return instance;
    }
}
```

**Implementation 2: Lazy Initialization (Thread-Safe)**
```java
class Singleton {
    private static volatile Singleton instance;
    
    private Singleton() { }
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {  // Double-check locking
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Implementation 3: Bill Pugh Solution (Recommended)**
```java
class Singleton {
    private Singleton() { }
    
    // Inner static class - loaded only when getInstance() is called
    private static class SingletonHelper {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}
```

**Implementation 4: Enum Singleton (Best for Java)**
```java
enum Singleton {
    INSTANCE;
    
    public void doSomething() {
        System.out.println("Doing something");
    }
}

// Usage
Singleton.INSTANCE.doSomething();
```

**Real-World Example:**
```java
// Database Connection Manager
public class DatabaseConnection {
    private static volatile DatabaseConnection instance;
    private Connection connection;
    
    private DatabaseConnection() {
        try {
            this.connection = DriverManager.getConnection(
                "jdbc:mysql://localhost:3306/mydb", "user", "password");
        } catch (SQLException e) {
            throw new RuntimeException("Failed to create database connection", e);
        }
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
    
    public Connection getConnection() {
        return connection;
    }
}
```

#### 2. Builder Pattern

**Purpose:** Construct complex objects step by step

**Use Cases:**
- Objects with many optional parameters
- Immutable objects
- Complex object construction

**Basic Implementation:**
```java
class User {
    private final String name;      // Required
    private final String email;     // Required
    private final int age;          // Optional
    private final String address;   // Optional
    private final String phone;     // Optional
    
    private User(Builder builder) {
        this.name = builder.name;
        this.email = builder.email;
        this.age = builder.age;
        this.address = builder.address;
        this.phone = builder.phone;
    }
    
    public static class Builder {
        private final String name;    // Required
        private final String email;   // Required
        private int age = 0;          // Optional with default
        private String address;       // Optional
        private String phone;         // Optional
        
        public Builder(String name, String email) {
            this.name = name;
            this.email = email;
        }
        
        public Builder age(int age) {
            this.age = age;
            return this;
        }
        
        public Builder address(String address) {
            this.address = address;
            return this;
        }
        
        public Builder phone(String phone) {
            this.phone = phone;
            return this;
        }
        
        public User build() {
            // Validation
            if (name == null || email == null) {
                throw new IllegalArgumentException("Name and email are required");
            }
            return new User(this);
        }
    }
    
    // Getters
    public String getName() { return name; }
    public String getEmail() { return email; }
    public int getAge() { return age; }
    public String getAddress() { return address; }
    public String getPhone() { return phone; }
}

// Usage
User user = new User.Builder("John Doe", "john@example.com")
    .age(30)
    .address("123 Main St")
    .phone("123-456-7890")
    .build();
```

**Real-World Example:**
```java
// HTTP Request Builder
public class HttpRequest {
    private final String method;
    private final String url;
    private final Map<String, String> headers;
    private final String body;
    
    private HttpRequest(Builder builder) {
        this.method = builder.method;
        this.url = builder.url;
        this.headers = builder.headers;
        this.body = builder.body;
    }
    
    public static class Builder {
        private String method = "GET";
        private String url;
        private Map<String, String> headers = new HashMap<>();
        private String body;
        
        public Builder url(String url) {
            this.url = url;
            return this;
        }
        
        public Builder method(String method) {
            this.method = method;
            return this;
        }
        
        public Builder header(String key, String value) {
            this.headers.put(key, value);
            return this;
        }
        
        public Builder body(String body) {
            this.body = body;
            return this;
        }
        
        public HttpRequest build() {
            if (url == null) {
                throw new IllegalArgumentException("URL is required");
            }
            return new HttpRequest(this);
        }
    }
}

// Usage
HttpRequest request = new HttpRequest.Builder()
    .url("https://api.example.com/users")
    .method("POST")
    .header("Content-Type", "application/json")
    .header("Authorization", "Bearer token123")
    .body("{\"name\":\"John\"}")
    .build();
```

#### 3. Factory Pattern

**Purpose:** Create objects without specifying exact class

**Simple Factory:**
```java
interface Animal {
    void makeSound();
}

class Dog implements Animal {
    public void makeSound() {
        System.out.println("Woof!");
    }
}

class Cat implements Animal {
    public void makeSound() {
        System.out.println("Meow!");
    }
}

class AnimalFactory {
    public static Animal createAnimal(String type) {
        switch (type.toLowerCase()) {
            case "dog":
                return new Dog();
            case "cat":
                return new Cat();
            default:
                throw new IllegalArgumentException("Unknown animal type: " + type);
        }
    }
}

// Usage
Animal dog = AnimalFactory.createAnimal("dog");
dog.makeSound();  // "Woof!"
```

**Factory Method Pattern:**
```java
abstract class AnimalFactory {
    public abstract Animal createAnimal();
    
    public void displayAnimal() {
        Animal animal = createAnimal();
        animal.makeSound();
    }
}

class DogFactory extends AnimalFactory {
    @Override
    public Animal createAnimal() {
        return new Dog();
    }
}

class CatFactory extends AnimalFactory {
    @Override
    public Animal createAnimal() {
        return new Cat();
    }
}

// Usage
AnimalFactory factory = new DogFactory();
factory.displayAnimal();  // Creates and displays Dog
```

**Abstract Factory Pattern:**
```java
// Abstract Factory
interface UIFactory {
    Button createButton();
    Dialog createDialog();
}

// Concrete Factories
class WindowsUIFactory implements UIFactory {
    public Button createButton() {
        return new WindowsButton();
    }
    
    public Dialog createDialog() {
        return new WindowsDialog();
    }
}

class MacUIFactory implements UIFactory {
    public Button createButton() {
        return new MacButton();
    }
    
    public Dialog createDialog() {
        return new MacDialog();
    }
}

// Usage
UIFactory factory = new WindowsUIFactory();
Button button = factory.createButton();
Dialog dialog = factory.createDialog();
```

### Structural Patterns

#### 4. Adapter Pattern

**Purpose:** Allow incompatible interfaces to work together

**Example:**
```java
// Target interface
interface MediaPlayer {
    void play(String audioType, String fileName);
}

// Adaptee (incompatible interface)
class AdvancedMediaPlayer {
    public void playVlc(String fileName) {
        System.out.println("Playing VLC file: " + fileName);
    }
    
    public void playMp4(String fileName) {
        System.out.println("Playing MP4 file: " + fileName);
    }
}

// Adapter
class MediaAdapter implements MediaPlayer {
    private AdvancedMediaPlayer advancedPlayer;
    
    public MediaAdapter(String audioType) {
        if (audioType.equalsIgnoreCase("vlc")) {
            advancedPlayer = new AdvancedMediaPlayer();
        } else if (audioType.equalsIgnoreCase("mp4")) {
            advancedPlayer = new AdvancedMediaPlayer();
        }
    }
    
    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("vlc")) {
            advancedPlayer.playVlc(fileName);
        } else if (audioType.equalsIgnoreCase("mp4")) {
            advancedPlayer.playMp4(fileName);
        }
    }
}

// Client
class AudioPlayer implements MediaPlayer {
    private MediaAdapter adapter;
    
    @Override
    public void play(String audioType, String fileName) {
        if (audioType.equalsIgnoreCase("mp3")) {
            System.out.println("Playing MP3 file: " + fileName);
        } else if (audioType.equalsIgnoreCase("vlc") || 
                   audioType.equalsIgnoreCase("mp4")) {
            adapter = new MediaAdapter(audioType);
            adapter.play(audioType, fileName);
        } else {
            System.out.println("Invalid media type");
        }
    }
}
```

#### 5. Decorator Pattern

**Purpose:** Add behavior to objects dynamically

**Example:**
```java
// Component
interface Coffee {
    double getCost();
    String getDescription();
}

// Concrete Component
class SimpleCoffee implements Coffee {
    public double getCost() {
        return 2.0;
    }
    
    public String getDescription() {
        return "Simple coffee";
    }
}

// Decorator
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
    
    public double getCost() {
        return coffee.getCost();
    }
    
    public String getDescription() {
        return coffee.getDescription();
    }
}

// Concrete Decorators
class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }
    
    public double getCost() {
        return coffee.getCost() + 0.5;
    }
    
    public String getDescription() {
        return coffee.getDescription() + ", Milk";
    }
}

class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }
    
    public double getCost() {
        return coffee.getCost() + 0.2;
    }
    
    public String getDescription() {
        return coffee.getDescription() + ", Sugar";
    }
}

// Usage
Coffee coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);
coffee = new SugarDecorator(coffee);

System.out.println(coffee.getDescription());  // "Simple coffee, Milk, Sugar"
System.out.println(coffee.getCost());          // 2.7
```

### Behavioral Patterns

#### 6. Strategy Pattern with Interfaces

**Strategy Pattern:** Define family of algorithms, make them interchangeable

```java
// Strategy interface
interface PaymentStrategy {
    void pay(double amount);
}

// Concrete strategies
class CreditCardPayment implements PaymentStrategy {
    private String cardNumber;
    
    public CreditCardPayment(String cardNumber) {
        this.cardNumber = cardNumber;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid " + amount + " using Credit Card: " + cardNumber);
    }
}

class PayPalPayment implements PaymentStrategy {
    private String email;
    
    public PayPalPayment(String email) {
        this.email = email;
    }
    
    @Override
    public void pay(double amount) {
        System.out.println("Paid " + amount + " using PayPal: " + email);
    }
}

// Context class
class ShoppingCart {
    private PaymentStrategy paymentStrategy;
    
    public void setPaymentStrategy(PaymentStrategy strategy) {
        this.paymentStrategy = strategy;
    }
    
    public void checkout(double amount) {
        paymentStrategy.pay(amount);
    }
}

// Usage
ShoppingCart cart = new ShoppingCart();
cart.setPaymentStrategy(new CreditCardPayment("1234-5678"));
cart.checkout(100.0);
```

### Factory Pattern with Abstract Classes

**Factory Pattern:** Create objects without specifying exact class

```java
// Abstract product
abstract class Animal {
    public abstract void makeSound();
}

// Concrete products
class Dog extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Woof!");
    }
}

class Cat extends Animal {
    @Override
    public void makeSound() {
        System.out.println("Meow!");
    }
}

// Factory
class AnimalFactory {
    public static Animal createAnimal(String type) {
        if (type.equalsIgnoreCase("dog")) {
            return new Dog();
        } else if (type.equalsIgnoreCase("cat")) {
            return new Cat();
        }
        throw new IllegalArgumentException("Unknown animal type");
    }
}

// Usage
Animal dog = AnimalFactory.createAnimal("dog");
dog.makeSound();  // "Woof!"
```

### Observer Pattern with Interfaces

**Observer Pattern:** One-to-many dependency between objects

```java
// Observer interface
interface Observer {
    void update(String message);
}

// Subject interface
interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

// Concrete subject
class NewsAgency implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String news;
    
    @Override
    public void attach(Observer observer) {
        observers.add(observer);
    }
    
    @Override
    public void detach(Observer observer) {
        observers.remove(observer);
    }
    
    @Override
    public void notifyObservers() {
        for (Observer observer : observers) {
            observer.update(news);
        }
    }
    
    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }
}

// Concrete observer
class NewsChannel implements Observer {
    private String name;
    
    public NewsChannel(String name) {
        this.name = name;
    }
    
    @Override
    public void update(String message) {
        System.out.println(name + " received: " + message);
    }
}
```

### Template Method Pattern with Abstract Classes

**Template Method:** Define skeleton of algorithm, let subclasses override steps

```java
abstract class DataProcessor {
    // Template method
    public final void process() {
        readData();
        processData();
        saveData();
    }
    
    protected abstract void readData();
    protected abstract void processData();
    
    protected void saveData() {
        System.out.println("Saving data...");
    }
}

class CSVProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading CSV data...");
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing CSV data...");
    }
}

class XMLProcessor extends DataProcessor {
    @Override
    protected void readData() {
        System.out.println("Reading XML data...");
    }
    
    @Override
    protected void processData() {
        System.out.println("Processing XML data...");
    }
}
```

---

## Best Practices

### SOLID Principles

#### 1. Single Responsibility Principle (SRP)

**A class should have only one reason to change**

```java
// ❌ BAD: Multiple responsibilities
class User {
    private String name;
    private String email;
    
    public void saveToDatabase() { ... }
    public void sendEmail() { ... }
    public void generateReport() { ... }
}

// ✅ GOOD: Single responsibility
class User {
    private String name;
    private String email;
    // Only user data
}

class UserRepository {
    public void save(User user) { ... }
}

class EmailService {
    public void sendEmail(User user) { ... }
}
```

#### 2. Open/Closed Principle (OCP)

**Open for extension, closed for modification**

```java
// ❌ BAD: Need to modify for new shapes
class AreaCalculator {
    public double calculateArea(Object shape) {
        if (shape instanceof Circle) {
            // Calculate circle area
        } else if (shape instanceof Rectangle) {
            // Calculate rectangle area
        }
        // Need to modify for new shapes
    }
}

// ✅ GOOD: Open for extension
interface Shape {
    double calculateArea();
}

class Circle implements Shape {
    @Override
    public double calculateArea() { ... }
}

class Rectangle implements Shape {
    @Override
    public double calculateArea() { ... }
}

// Can add new shapes without modifying existing code
```

#### 3. Liskov Substitution Principle (LSP)

**Subtypes must be substitutable for their base types**

```java
// ❌ BAD: Violates LSP
class Rectangle {
    protected int width, height;
    
    public void setWidth(int w) { width = w; }
    public void setHeight(int h) { height = h; }
}

class Square extends Rectangle {
    @Override
    public void setWidth(int w) {
        width = height = w;  // Breaks rectangle behavior
    }
}

// ✅ GOOD: Follows LSP
interface Shape {
    int getArea();
}

class Rectangle implements Shape {
    private int width, height;
    public int getArea() { return width * height; }
}

class Square implements Shape {
    private int side;
    public int getArea() { return side * side; }
}
```

#### 4. Interface Segregation Principle (ISP)

**Clients shouldn't depend on interfaces they don't use**

```java
// ❌ BAD: Fat interface
interface Worker {
    void work();
    void eat();
    void sleep();
}

class Human implements Worker {
    public void work() { ... }
    public void eat() { ... }
    public void sleep() { ... }
}

class Robot implements Worker {
    public void work() { ... }
    public void eat() { ... }  // Robot doesn't eat!
    public void sleep() { ... }  // Robot doesn't sleep!
}

// ✅ GOOD: Segregated interfaces
interface Workable {
    void work();
}

interface Eatable {
    void eat();
}

interface Sleepable {
    void sleep();
}

class Human implements Workable, Eatable, Sleepable {
    // Implements all
}

class Robot implements Workable {
    // Only implements work
}
```

#### 5. Dependency Inversion Principle (DIP)

**Depend on abstractions, not concretions**

```java
// ❌ BAD: Depends on concrete class
class EmailService {
    public void sendEmail() {
        // Direct dependency
    }
}

class NotificationService {
    private EmailService emailService;  // Concrete dependency
    
    public void notify() {
        emailService.sendEmail();
    }
}

// ✅ GOOD: Depends on abstraction
interface MessageService {
    void send(String message);
}

class EmailService implements MessageService {
    @Override
    public void send(String message) { ... }
}

class SMSService implements MessageService {
    @Override
    public void send(String message) { ... }
}

class NotificationService {
    private MessageService messageService;  // Abstract dependency
    
    public NotificationService(MessageService service) {
        this.messageService = service;  // Dependency injection
    }
    
    public void notify(String message) {
        messageService.send(message);
    }
}
```

### Code Quality Guidelines

#### 1. Use Composition Over Inheritance

**Why Composition is Better:**
- More flexible
- Easier to test
- Avoids fragile base class problem
- Follows "Program to interface" principle

**Example:**
```java
// ❌ BAD: Inheritance
class Stack extends ArrayList {
    public void push(Object item) {
        add(item);
    }
    
    public Object pop() {
        return remove(size() - 1);
    }
}
// Problem: Stack inherits all ArrayList methods (clear, add at index, etc.)

// ✅ GOOD: Composition
class Stack {
    private List<Object> list = new ArrayList<>();
    
    public void push(Object item) {
        list.add(item);
    }
    
    public Object pop() {
        return list.remove(list.size() - 1);
    }
    
    public boolean isEmpty() {
        return list.isEmpty();
    }
}
// Only exposes Stack-specific methods
```

```java
// Prefer composition
class Car {
    private Engine engine;  // Composition
    private Wheel[] wheels; // Composition
    
    public Car(Engine engine, Wheel[] wheels) {
        this.engine = engine;
        this.wheels = wheels;
    }
}
```

#### 2. Program to Interface, Not Implementation

```java
// ✅ GOOD
List<String> list = new ArrayList<>();  // Interface reference

// ❌ BAD
ArrayList<String> list = new ArrayList<>();  // Concrete class reference
```

#### 3. Favor Immutability

```java
// Immutable class
public final class Person {
    private final String name;
    private final int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
    
    // No setters, only getters
    public String getName() { return name; }
    public int getAge() { return age; }
}
```

#### 4. Use Builder Pattern for Complex Objects

```java
class User {
    private String name;
    private String email;
    private int age;
    private String address;
    
    private User(Builder builder) {
        this.name = builder.name;
        this.email = builder.email;
        this.age = builder.age;
        this.address = builder.address;
    }
    
    public static class Builder {
        private String name;
        private String email;
        private int age;
        private String address;
        
        public Builder name(String name) {
            this.name = name;
            return this;
        }
        
        public Builder email(String email) {
            this.email = email;
            return this;
        }
        
        public Builder age(int age) {
            this.age = age;
            return this;
        }
        
        public Builder address(String address) {
            this.address = address;
            return this;
        }
        
        public User build() {
            return new User(this);
        }
    }
}

// Usage
User user = new User.Builder()
    .name("John")
    .email("john@example.com")
    .age(30)
    .address("123 Main St")
    .build();
```

#### 5. Use Immutable Objects

**Benefits:**
- Thread-safe
- No side effects
- Easier to reason about
- Can be safely shared

**Creating Immutable Classes:**
```java
// ✅ GOOD: Immutable class
public final class ImmutablePerson {
    private final String name;
    private final int age;
    private final List<String> hobbies;
    
    public ImmutablePerson(String name, int age, List<String> hobbies) {
        this.name = name;
        this.age = age;
        // Defensive copy
        this.hobbies = new ArrayList<>(hobbies);
    }
    
    public String getName() {
        return name;
    }
    
    public int getAge() {
        return age;
    }
    
    public List<String> getHobbies() {
        // Return defensive copy
        return new ArrayList<>(hobbies);
    }
    
    // No setters!
}
```

#### 6. Prefer Dependency Injection

**Benefits:**
- Loose coupling
- Easier testing
- Better flexibility

```java
// ❌ BAD: Tight coupling
class OrderService {
    private EmailService emailService = new EmailService();
    
    public void processOrder(Order order) {
        // Process order
        emailService.sendEmail(order.getCustomerEmail());
    }
}

// ✅ GOOD: Dependency Injection
class OrderService {
    private final EmailService emailService;
    
    public OrderService(EmailService emailService) {
        this.emailService = emailService;  // Injected dependency
    }
    
    public void processOrder(Order order) {
        // Process order
        emailService.sendEmail(order.getCustomerEmail());
    }
}

// Usage
EmailService emailService = new EmailService();
OrderService orderService = new OrderService(emailService);
```

#### 7. Use Enums for Constants

```java
// ❌ BAD: Magic numbers/strings
if (status == 1) { }
if (status.equals("ACTIVE")) { }

// ✅ GOOD: Enums
enum OrderStatus {
    PENDING,
    PROCESSING,
    SHIPPED,
    DELIVERED,
    CANCELLED
}

if (status == OrderStatus.PENDING) { }
```

#### 8. Avoid Deep Inheritance Hierarchies

**Problem:** Deep hierarchies are hard to maintain

```java
// ❌ BAD: Too deep
class Animal { }
class Mammal extends Animal { }
class Dog extends Mammal { }
class Labrador extends Dog { }
class BlackLabrador extends Labrador { }  // Too deep!

// ✅ GOOD: Prefer composition
class Animal { }
class Dog extends Animal {
    private Breed breed;  // Composition
}

class Breed {
    private String name;
    private String color;
}
```

#### 9. Use Method Chaining for Fluent APIs

```java
class QueryBuilder {
    private String select;
    private String from;
    private String where;
    
    public QueryBuilder select(String columns) {
        this.select = columns;
        return this;
    }
    
    public QueryBuilder from(String table) {
        this.from = table;
        return this;
    }
    
    public QueryBuilder where(String condition) {
        this.where = condition;
        return this;
    }
    
    public String build() {
        return "SELECT " + select + " FROM " + from + " WHERE " + where;
    }
}

// Usage: Fluent API
String query = new QueryBuilder()
    .select("name, email")
    .from("users")
    .where("age > 18")
    .build();
```

#### 10. Use Optional for Null Safety

```java
// ❌ BAD: Null checks everywhere
public String getUserName(Long userId) {
    User user = userRepository.findById(userId);
    if (user != null) {
        return user.getName();
    }
    return null;
}

// ✅ GOOD: Optional
public Optional<String> getUserName(Long userId) {
    return userRepository.findById(userId)
        .map(User::getName);
}

// Usage
Optional<String> name = getUserName(123L);
name.ifPresent(System.out::println);
```

---

## Interview Questions

### Q1: Explain the four pillars of OOP with examples.

**Answer:**
"The four pillars of OOP are:

1. **Encapsulation**: Bundling data and methods together, hiding internal details. Example: A BankAccount class with private balance field and public deposit/withdraw methods.

2. **Inheritance**: Child class acquires properties from parent class. Example: Dog extends Animal, inheriting name and age fields.

3. **Polymorphism**: Objects can take multiple forms. Example: Shape reference can point to Circle or Rectangle, calling appropriate draw() method at runtime.

4. **Abstraction**: Hiding complexity, showing only essentials. Example: Vehicle interface with start() method, implemented by Car and Bike classes.

In our e-commerce system, I use encapsulation for Order class (private fields with public methods), inheritance for different payment types, polymorphism for payment processing, and abstraction through service interfaces."

### Q2: What is the difference between method overloading and overriding?

**Answer:**
"Method overloading is compile-time polymorphism where multiple methods have the same name but different parameters. The compiler decides which method to call based on arguments.

Method overriding is runtime polymorphism where a child class provides a specific implementation of a parent's method. The JVM decides which method to call based on the actual object type.

**Key differences:**
- Overloading: Same class, different parameters, compile-time resolution
- Overriding: Different classes (parent-child), same signature, runtime resolution
- Overloading: Return type can differ
- Overriding: Return type must be same or covariant
- Overloading: Access modifier can differ
- Overriding: Can't be more restrictive"

### Q3: Explain the difference between abstract class and interface.

**Answer:**
"Abstract classes and interfaces both provide abstraction, but serve different purposes:

**Abstract Class:**
- Can have instance variables and constructors
- Can have both abstract and concrete methods
- Supports single inheritance
- Use 'extends' keyword
- Use when you have IS-A relationship and want to share code

**Interface:**
- Only constants (public static final)
- All methods are abstract (before Java 8)
- Supports multiple inheritance
- Use 'implements' keyword
- Use when you have CAN-DO relationship and want to define contract

**Java 8+ changes:**
- Interfaces can have default and static methods
- Still can't have instance variables or constructors

**Best practice:** Use interfaces for contracts, abstract classes for shared implementation."

### Q4: What is the difference between composition and inheritance?

**Answer:**
"Composition and inheritance are two ways to reuse code:

**Inheritance (IS-A):**
- Child class IS-A type of parent class
- Tight coupling
- Example: Dog IS-A Animal

**Composition (HAS-A):**
- Class HAS-A member of another class
- Loose coupling
- Example: Car HAS-A Engine

**When to use:**
- Use inheritance when there's a clear IS-A relationship and you need polymorphism
- Use composition when you need flexibility and want to avoid inheritance hierarchy issues

**Favor composition over inheritance** because it provides:
- Better flexibility
- Easier testing
- Avoids fragile base class problem
- Follows 'Program to interface' principle"

### Q5: Explain polymorphism with a real-world example.

**Answer:**
"Polymorphism allows objects of different types to be treated as objects of a common base type.

**Example from our e-commerce system:**

```java
interface PaymentProcessor {
    void processPayment(double amount);
}

class CreditCardProcessor implements PaymentProcessor {
    public void processPayment(double amount) {
        // Credit card specific logic
    }
}

class PayPalProcessor implements PaymentProcessor {
    public void processPayment(double amount) {
        // PayPal specific logic
    }
}

// Polymorphism in action
PaymentProcessor processor = getPaymentProcessor(type);
processor.processPayment(100.0);  // Calls appropriate implementation
```

At runtime, the JVM determines which implementation to call based on the actual object type. This allows us to add new payment methods without modifying existing code."

### Q6: What is method hiding in Java?

**Answer:**
"Method hiding occurs with static methods. Unlike instance methods, static methods are not overridden but hidden.

**Key points:**
- Static methods are resolved at compile-time based on reference type
- Instance methods are resolved at runtime based on actual object type
- Use class name to call static methods, not instance reference

**Example:**
```java
class Parent {
    static void method() { System.out.println("Parent"); }
    void instanceMethod() { System.out.println("Parent instance"); }
}

class Child extends Parent {
    static void method() { System.out.println("Child"); }  // Hiding
    void instanceMethod() { System.out.println("Child instance"); }  // Overriding
}

Parent p = new Child();
p.method();  // Prints "Parent" (compile-time, based on reference)
p.instanceMethod();  // Prints "Child instance" (runtime, based on object)
```"

### Q7: Explain the 'super' keyword and its usage.

**Answer:**
"The 'super' keyword refers to the immediate parent class instance.

**Uses:**
1. **Call parent constructor**: `super(parameters)` - Must be first statement
2. **Call parent method**: `super.methodName()` - When overriding
3. **Access parent field**: `super.fieldName` - When shadowed

**Example:**
```java
class Animal {
    protected String name;
    
    public Animal(String name) {
        this.name = name;
    }
    
    public void eat() {
        System.out.println(name + " is eating");
    }
}

class Dog extends Animal {
    public Dog(String name) {
        super(name);  // Call parent constructor
    }
    
    @Override
    public void eat() {
        super.eat();  // Call parent method
        System.out.println(name + " is eating dog food");
    }
}
```"

### Q8: What are sealed classes and when to use them?

**Answer:**
"Sealed classes (Java 17+) restrict which classes can extend them, providing controlled inheritance.

**Benefits:**
- Prevents unauthorized inheritance
- Makes class hierarchy explicit
- Enables exhaustive pattern matching

**Example:**
```java
public sealed class Shape 
    permits Circle, Rectangle, Triangle {
    // Only Circle, Rectangle, Triangle can extend
}

public final class Circle extends Shape { }
public final class Rectangle extends Shape { }
public final class Triangle extends Shape { }
```

**Use when:**
- You want to control class hierarchy
- You need exhaustive pattern matching
- You want to prevent third-party extensions"

### Q9: Explain covariant return types.

**Answer:**
"Covariant return types allow an overriding method to return a subtype of the parent method's return type.

**Example:**
```java
class Animal {
    public Animal getAnimal() {
        return new Animal();
    }
}

class Dog extends Animal {
    @Override
    public Dog getAnimal() {  // Returns Dog (subtype of Animal)
        return new Dog();
    }
}
```

**Benefits:**
- More specific return types
- Better type safety
- More intuitive API

**Rules:**
- Return type must be a subtype
- Can't use primitive types (must be reference types)
- Must maintain method signature compatibility"

### Q10: What is the difference between 'this' and 'super'?

**Answer:**
"'this' refers to the current object instance, while 'super' refers to the parent class instance.

**'this' usage:**
- Access current class members
- Call current class constructor: `this(parameters)`
- Pass current object as parameter

**'super' usage:**
- Access parent class members
- Call parent class constructor: `super(parameters)`
- Call overridden parent method: `super.methodName()`

**Example:**
```java
class Parent {
    protected int value = 10;
}

class Child extends Parent {
    private int value = 20;
    
    public void display() {
        System.out.println(this.value);   // 20 (current class)
        System.out.println(super.value);  // 10 (parent class)
    }
}
```"

### Q11: Explain the difference between static and instance methods.

**Answer:**
"Static methods belong to the class, instance methods belong to objects.

**Static methods:**
- Called using class name: `ClassName.method()`
- Can't access instance variables
- Can't use 'this' or 'super'
- Shared by all instances
- Resolved at compile-time

**Instance methods:**
- Called using object: `object.method()`
- Can access both static and instance variables
- Can use 'this' and 'super'
- Each object has its own copy
- Resolved at runtime (for overridden methods)

**Example:**
```java
class Counter {
    static int count = 0;  // Class variable
    int instanceCount = 0;  // Instance variable
    
    static void increment() {
        count++;  // Can access static
        // instanceCount++;  // ❌ Can't access instance
    }
    
    void incrementInstance() {
        count++;  // Can access static
        instanceCount++;  // Can access instance
    }
}
```"

### Q12: What are inner classes and their types?

**Answer:**
"Inner classes are classes defined inside another class.

**Types:**
1. **Member Inner Class**: Non-static, has access to outer class members
2. **Static Nested Class**: Static, can access only static members
3. **Local Inner Class**: Defined inside a method
4. **Anonymous Inner Class**: Class without a name, often for interfaces

**Use cases:**
- Logical grouping of classes
- Encapsulation
- Callback implementations
- Event handling

**Example:**
```java
class Outer {
    private int outerVar = 10;
    
    class Inner {  // Member inner class
        void display() {
            System.out.println(outerVar);  // Can access outer members
        }
    }
    
    static class Nested {  // Static nested class
        void display() {
            // Can't access outerVar (not static)
        }
    }
}
```"

### Q13: Explain SOLID principles with examples.

**Answer:**
"SOLID principles are five design principles for object-oriented programming:

1. **S - Single Responsibility**: A class should have one reason to change
   - Example: Separate User, UserRepository, EmailService classes

2. **O - Open/Closed**: Open for extension, closed for modification
   - Example: Shape interface, add new shapes without modifying existing code

3. **L - Liskov Substitution**: Subtypes must be substitutable for base types
   - Example: Any Animal subclass should work wherever Animal is expected

4. **I - Interface Segregation**: Clients shouldn't depend on unused interfaces
   - Example: Separate Workable, Eatable interfaces instead of one Worker interface

5. **D - Dependency Inversion**: Depend on abstractions, not concretions
   - Example: Depend on MessageService interface, not EmailService class

These principles make code more maintainable, testable, and flexible."

### Q14: What is the difference between 'final', 'finally', and 'finalize'?

**Answer:**
"These are three different concepts in Java:

**'final' keyword:**
- Final variable: Can't be reassigned
- Final method: Can't be overridden
- Final class: Can't be inherited

**'finally' block:**
- Used in try-catch-finally
- Always executes (except System.exit())
- Used for cleanup code

**'finalize' method:**
- Called by garbage collector before object destruction
- Deprecated in Java 9, removed in Java 18
- Not reliable, use try-with-resources instead

**Example:**
```java
class Example {
    final int value = 10;  // Final variable
    
    public final void method() { }  // Final method
    
    public void cleanup() {
        try {
            // Code
        } finally {
            // Always executes
        }
    }
}
```"

### Q15: Explain method resolution in Java.

**Answer:**
"Method resolution determines which method to call.

**For instance methods (runtime polymorphism):**
1. JVM looks at actual object type
2. Searches method in that class
3. If not found, searches parent class
4. Continues up inheritance chain
5. If not found, compile error

**For static methods (compile-time):**
1. Compiler looks at reference type
2. Resolves at compile-time
3. No runtime lookup

**Example:**
```java
class A {
    void method() { System.out.println("A"); }
}

class B extends A {
    void method() { System.out.println("B"); }
}

A obj = new B();
obj.method();  // Prints "B" (runtime resolution based on actual object)
```"

### Q16: When should you use an abstract class vs an interface?

**Answer:**
"I use abstract classes when I have an IS-A relationship and need to share code between related classes. For example, in our payment system, I have an abstract PaymentProcessor class that provides common validation and logging logic, while subclasses implement payment-specific logic.

I use interfaces when I have a CAN-DO relationship and want to define a contract. For example, I use a PaymentMethod interface that different payment providers implement, allowing the system to work with any payment method without knowing the specific implementation.

**Key decision factors:**
- **Abstract class**: Need shared code, constructors, instance variables, single inheritance
- **Interface**: Need multiple inheritance, define contract, loose coupling, multiple behaviors

**Best practice**: Start with interfaces, use abstract classes when you need to share implementation code."

### Q17: Explain the Builder pattern and when to use it.

**Answer:**
"The Builder pattern constructs complex objects step by step. I use it when objects have many optional parameters or when I want to create immutable objects.

**Benefits:**
- Readable code
- Handles optional parameters elegantly
- Can validate before object creation
- Immutable objects

**Example from our system:**
```java
Order order = new Order.Builder()
    .customerId(123L)
    .items(items)
    .shippingAddress(address)
    .paymentMethod(payment)
    .build();
```

This is much clearer than a constructor with 10 parameters. I also use it for creating configuration objects and DTOs with many optional fields."

### Q18: What is the difference between method hiding and method overriding?

**Answer:**
"Method hiding occurs with static methods, while method overriding occurs with instance methods.

**Method Hiding (Static):**
- Resolved at compile-time based on reference type
- Uses class name, not object instance
- Not polymorphic

**Method Overriding (Instance):**
- Resolved at runtime based on actual object type
- Uses object instance
- Polymorphic behavior

**Example:**
```java
class Parent {
    static void staticMethod() { System.out.println("Parent static"); }
    void instanceMethod() { System.out.println("Parent instance"); }
}

class Child extends Parent {
    static void staticMethod() { System.out.println("Child static"); }  // Hiding
    void instanceMethod() { System.out.println("Child instance"); }     // Overriding
}

Parent p = new Child();
p.staticMethod();      // "Parent static" (compile-time, reference type)
p.instanceMethod();   // "Child instance" (runtime, object type)
```

**Best practice**: Always call static methods using class name, not instance reference."

### Q19: Explain the Singleton pattern and its implementations.

**Answer:**
"The Singleton pattern ensures only one instance of a class exists. I use it for database connections, logger instances, and configuration managers.

**Implementation approaches:**

1. **Eager Initialization**: Simple but creates instance even if not used
2. **Lazy Initialization**: Creates on first access, needs synchronization
3. **Double-Check Locking**: Thread-safe lazy initialization
4. **Bill Pugh Solution**: Uses inner static class, recommended
5. **Enum Singleton**: Best for Java, thread-safe and serialization-safe

**Example (Bill Pugh - Recommended):**
```java
class Singleton {
    private Singleton() { }
    
    private static class SingletonHelper {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return SingletonHelper.INSTANCE;
    }
}
```

**When to use:**
- Shared resources (database connections)
- Logging
- Configuration
- Cache managers

**When NOT to use:**
- Regular business objects
- When you need multiple instances
- When testing (hard to mock)"

### Q20: What are default methods in interfaces and why were they introduced?

**Answer:**
"Default methods (Java 8+) allow interfaces to provide method implementations. They were introduced to add new methods to existing interfaces without breaking existing implementations.

**Benefits:**
- Backward compatibility
- Can add functionality to interfaces
- Multiple inheritance of behavior

**Example:**
```java
interface List {
    void add(Object item);
    
    // Default method - provides implementation
    default void addAll(Collection items) {
        for (Object item : items) {
            add(item);
        }
    }
}
```

**Rules:**
- Must use `default` keyword
- Can be overridden by implementing classes
- Can call other interface methods
- Resolves conflicts if multiple interfaces have same default method

**Use cases:**
- Adding methods to existing interfaces (like Collection API)
- Providing common implementation
- Multiple inheritance of behavior"

### Q21: Explain the Adapter pattern with a real-world example.

**Answer:**
"The Adapter pattern allows incompatible interfaces to work together. It acts as a bridge between two incompatible interfaces.

**Real-world example from our system:**
```java
// Third-party payment gateway (incompatible interface)
class PayPalAPI {
    public void makePayment(String email, double amount) {
        // PayPal specific implementation
    }
}

// Our system's interface
interface PaymentProcessor {
    void processPayment(PaymentRequest request);
}

// Adapter
class PayPalAdapter implements PaymentProcessor {
    private PayPalAPI paypalAPI;
    
    public PayPalAdapter(PayPalAPI paypalAPI) {
        this.paypalAPI = paypalAPI;
    }
    
    @Override
    public void processPayment(PaymentRequest request) {
        // Adapt PayPal API to our interface
        paypalAPI.makePayment(
            request.getUserEmail(),
            request.getAmount()
        );
    }
}
```

**Use cases:**
- Integrating third-party libraries
- Legacy code integration
- Making incompatible interfaces work together
- Wrapper classes"

### Q22: What is the Decorator pattern and how does it differ from inheritance?

**Answer:**
"The Decorator pattern adds behavior to objects dynamically at runtime, while inheritance adds behavior at compile-time.

**Key differences:**
- **Decorator**: Runtime behavior addition, composition-based
- **Inheritance**: Compile-time behavior, IS-A relationship

**Example:**
```java
// Component
interface Coffee {
    double getCost();
    String getDescription();
}

// Decorator
abstract class CoffeeDecorator implements Coffee {
    protected Coffee coffee;
    
    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }
}

// Usage: Add behaviors dynamically
Coffee coffee = new SimpleCoffee();
coffee = new MilkDecorator(coffee);      // Add milk
coffee = new SugarDecorator(coffee);     // Add sugar
coffee = new CaramelDecorator(coffee);   // Add caramel
```

**Benefits over inheritance:**
- More flexible (can combine decorators in any order)
- Avoids class explosion (don't need CoffeeWithMilk, CoffeeWithSugar, CoffeeWithMilkAndSugar, etc.)
- Runtime composition

**Use cases:**
- Adding features to objects dynamically
- Wrapping objects with additional behavior
- I/O streams in Java (BufferedReader, FileReader, etc.)"

### Q23: Explain reflection in Java and its use cases.

**Answer:**
"Reflection allows inspection and modification of classes, methods, and fields at runtime. It's powerful but should be used carefully.

**Use cases:**
- **Frameworks**: Spring uses reflection for dependency injection
- **Testing**: JUnit uses reflection to find and run test methods
- **Serialization**: JSON/XML libraries use reflection to serialize objects
- **Dynamic proxies**: Creating proxies at runtime

**Example:**
```java
Class<?> clazz = Person.class;
Method method = clazz.getMethod("getName");
Object result = method.invoke(person);
```

**When to use:**
- Framework development
- Testing frameworks
- Code analysis tools
- Dynamic behavior

**When NOT to use:**
- Regular application code (performance overhead)
- Security-sensitive code (can bypass access controls)
- When compile-time solutions exist

**Performance impact:**
- Reflection is slower than direct calls
- Use caching when possible
- Consider alternatives like code generation"

### Q24: What are sealed classes and what problems do they solve?

**Answer:**
"Sealed classes (Java 17+) restrict which classes can extend them, providing controlled inheritance.

**Problems they solve:**
1. **Uncontrolled inheritance**: Prevents unauthorized classes from extending
2. **Exhaustive pattern matching**: Compiler knows all possible subclasses
3. **API design**: Makes class hierarchy explicit

**Example:**
```java
public sealed class PaymentMethod 
    permits CreditCard, PayPal, BankTransfer {
    // Only these three classes can extend
}

public final class CreditCard extends PaymentMethod { }
public final class PayPal extends PaymentMethod { }
public final class BankTransfer extends PaymentMethod { }
```

**Benefits:**
- **Security**: Prevents third-party extensions
- **Maintainability**: Clear class hierarchy
- **Pattern matching**: Exhaustive switch expressions
- **Documentation**: Makes design intent clear

**Use cases:**
- API design (control extensions)
- Domain modeling (limited set of types)
- State machines
- Algebraic data types"

### Q25: Explain type erasure in Java generics.

**Answer:**
"Type erasure means generic type information is removed at compile time and replaced with raw types or Object.

**What happens:**
```java
// Source code
List<String> list = new ArrayList<>();

// After type erasure (bytecode)
List list = new ArrayList();  // Raw type
```

**Implications:**
1. **Can't use instanceof with generics**
   ```java
   // ❌ Compile error
   if (obj instanceof List<String>) { }
   
   // ✅ Workaround
   if (obj instanceof List) { }
   ```

2. **Can't create arrays of generic types**
   ```java
   // ❌ Compile error
   List<String>[] array = new List<String>[10];
   
   // ✅ Workaround
   List<String>[] array = (List<String>[]) new List[10];
   ```

3. **Can't catch generic exceptions**
   ```java
   // ❌ Compile error
   catch (Exception<String> e) { }
   ```

**Why type erasure?**
- Backward compatibility with pre-generics code
- Simpler JVM implementation
- No runtime overhead

**Workarounds:**
- Use Class objects for type information
- Use wildcards for flexibility
- Use type tokens in some cases"

### Q26: What is the difference between 'extends' and 'super' in generics?

**Answer:**
"'extends' and 'super' are wildcards that provide flexibility in generics.

**'extends' (Upper Bounded Wildcard):**
- `List<? extends Number>`: Can hold Number or its subclasses
- **PECS: Producer Extends** - Use when you're reading from the collection

**'super' (Lower Bounded Wildcard):**
- `List<? super Integer>`: Can hold Integer or its superclasses
- **PECS: Consumer Super** - Use when you're writing to the collection

**Example:**
```java
// Producer - reading from list
public void processNumbers(List<? extends Number> numbers) {
    for (Number num : numbers) {
        System.out.println(num.doubleValue());  // Can read as Number
    }
    // numbers.add(new Integer(1));  // ❌ Can't add (don't know exact type)
}

// Consumer - writing to list
public void addNumbers(List<? super Integer> numbers) {
    numbers.add(new Integer(1));  // ✅ Can add Integer
    numbers.add(new Integer(2));
    // Integer num = numbers.get(0);  // ❌ Can't read (don't know exact type)
}
```

**PECS Principle:**
- **Producer Extends**: When you produce/read from collection
- **Consumer Super**: When you consume/write to collection

**Use cases:**
- Collections.copy()
- Comparator implementations
- Generic algorithms"

### Q27: Explain the Observer pattern and its implementation.

**Answer:**
"The Observer pattern defines a one-to-many dependency between objects. When one object changes state, all dependents are notified.

**Components:**
- **Subject**: Maintains list of observers, notifies them
- **Observer**: Interface for objects that should be notified

**Example:**
```java
interface Observer {
    void update(String event);
}

interface Subject {
    void attach(Observer observer);
    void detach(Observer observer);
    void notifyObservers();
}

class NewsAgency implements Subject {
    private List<Observer> observers = new ArrayList<>();
    private String news;
    
    public void setNews(String news) {
        this.news = news;
        notifyObservers();
    }
    
    public void attach(Observer observer) {
        observers.add(observer);
    }
    
    public void notifyObservers() {
        observers.forEach(o -> o.update(news));
    }
}
```

**Use cases:**
- Event handling systems
- Model-View-Controller (MVC)
- Publish-Subscribe systems
- Java's Observer/Observable (deprecated)

**Benefits:**
- Loose coupling
- Dynamic relationships
- Broadcast communication

**Java 9+**: Observer/Observable are deprecated, use PropertyChangeListener or custom implementation"

### Q28: What are method references and when to use them?

**Answer:**
"Method references are shorthand for lambda expressions that call existing methods.

**Types:**
1. **Static**: `Integer::parseInt`
2. **Instance**: `str::length`
3. **Arbitrary object**: `String::toUpperCase`
4. **Constructor**: `ArrayList::new`

**Example:**
```java
// Lambda
list.forEach(item -> System.out.println(item));

// Method reference
list.forEach(System.out::println);

// Lambda
list.stream().map(item -> item.toUpperCase());

// Method reference
list.stream().map(String::toUpperCase);
```

**When to use:**
- Lambda just calls existing method
- Code is more readable
- Slight performance benefit (no lambda object creation)

**When NOT to use:**
- Method call is complex
- Need to modify parameters
- Readability suffers"

### Q29: Explain the Factory pattern and its variants.

**Answer:**
"The Factory pattern creates objects without specifying exact classes.

**Variants:**

1. **Simple Factory**: Single factory class creates objects
   ```java
   class AnimalFactory {
       public static Animal create(String type) {
           if (type.equals("dog")) return new Dog();
           if (type.equals("cat")) return new Cat();
           throw new IllegalArgumentException();
       }
   }
   ```

2. **Factory Method**: Subclasses decide which class to instantiate
   ```java
   abstract class AnimalFactory {
       public abstract Animal createAnimal();
   }
   
   class DogFactory extends AnimalFactory {
       public Animal createAnimal() {
           return new Dog();
       }
   }
   ```

3. **Abstract Factory**: Creates families of related objects
   ```java
   interface UIFactory {
       Button createButton();
       Dialog createDialog();
   }
   ```

**Use cases:**
- Object creation is complex
- Need to decouple creation from usage
- Need to support multiple product types
- Configuration-based object creation

**Benefits:**
- Loose coupling
- Single Responsibility
- Open/Closed Principle"

### Q30: What is the difference between composition and aggregation?

**Answer:**
"Both are forms of association, but with different lifecycles.

**Composition (Strong HAS-A):**
- Part cannot exist without whole
- Lifecycle is controlled by whole
- Whole is responsible for creation/destruction
- Example: House and Room (room can't exist without house)

**Aggregation (Weak HAS-A):**
- Part can exist independently
- Lifecycle is independent
- Whole doesn't control part's lifecycle
- Example: University and Student (student exists without university)

**Code Example:**
```java
// Composition
class House {
    private Room room;  // Room created/destroyed with House
    
    public House() {
        this.room = new Room();  // Composition
    }
}

// Aggregation
class University {
    private List<Student> students;  // Students exist independently
    
    public University(List<Student> students) {
        this.students = students;  // Aggregation
    }
}
```

**UML:**
- Composition: Filled diamond
- Aggregation: Empty diamond

**When to use:**
- **Composition**: Strong ownership, part doesn't make sense alone
- **Aggregation**: Weak ownership, part can exist independently"

---

## Quick Reference

### Access Modifiers Summary

| Modifier | Same Class | Same Package | Subclass | Other Package |
|----------|------------|--------------|----------|---------------|
| private | ✅ | ❌ | ❌ | ❌ |
| (default) | ✅ | ✅ | ❌ | ❌ |
| protected | ✅ | ✅ | ✅ | ❌ |
| public | ✅ | ✅ | ✅ | ✅ |

### Inheritance Keywords

- `extends`: For class inheritance
- `implements`: For interface implementation
- `super`: Access parent class
- `this`: Access current class
- `final`: Prevent inheritance/override
- `abstract`: Must be extended/implemented

### Method Types

- **Instance Method**: Belongs to object, can access instance variables
- **Static Method**: Belongs to class, can't access instance variables
- **Abstract Method**: No implementation, must be overridden
- **Final Method**: Can't be overridden
- **Default Method**: Interface method with implementation (Java 8+)

---

## Summary

**Core OOP Concepts:**
- **Encapsulation**: Data hiding and controlled access
- **Inheritance**: Code reuse through IS-A relationship
- **Polymorphism**: Multiple forms (overloading and overriding)
- **Abstraction**: Hide complexity, show essentials

**Advanced Topics:**
- Composition vs Inheritance
- Inner classes and Lambda expressions
- Generics and Type safety
- Records and Sealed classes
- SOLID principles

**Best Practices:**
- Favor composition over inheritance
- Program to interface, not implementation
- Follow SOLID principles
- Use design patterns appropriately
- Write clean, maintainable code

---

**Happy Learning! 🎯**

*Document completed. All sections saved successfully.*

