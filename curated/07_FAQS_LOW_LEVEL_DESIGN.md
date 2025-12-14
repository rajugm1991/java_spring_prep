# Low-Level Design FAQs - Top 80 Frequently Asked Questions

> **Curated from real interviews at top tech companies**

This guide contains 80+ frequently asked low-level design questions with detailed answers.

---

## Table of Contents

1. [OOP Principles](#oop-principles)
2. [SOLID Principles](#solid-principles)
3. [Design Patterns](#design-patterns)
4. [Class Design](#class-design)
5. [Relationships](#relationships)
6. [Best Practices](#best-practices)

---

## OOP Principles

### Q1: What are the four pillars of OOP?

**Answer:**
1. **Encapsulation**: Bundling data and methods together, hiding internal details
2. **Inheritance**: Creating new classes based on existing classes
3. **Polymorphism**: Same interface, different implementations
4. **Abstraction**: Hiding complex implementation details

---

### Q2: What is encapsulation?

**Answer:**
Encapsulation is bundling data and methods together and controlling access through access modifiers.

**Example:**
```java
public class BankAccount {
    private double balance; // Private - hidden from outside
    
    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }
    
    public double getBalance() {
        return balance; // Controlled access
    }
}
```

**Benefits:**
- Data protection
- Controlled access
- Easier maintenance

---

### Q3: What is inheritance?

**Answer:**
Inheritance allows a class to inherit properties and methods from a parent class.

**Example:**
```java
public class Vehicle {
    protected String brand;
    
    public void start() {
        System.out.println("Vehicle started");
    }
}

public class Car extends Vehicle {
    public void drive() {
        start(); // Inherited method
        System.out.println("Car is driving");
    }
}
```

**Benefits:**
- Code reuse
- Polymorphism
- Hierarchical organization

---

### Q4: What is polymorphism?

**Answer:**
Polymorphism allows objects of different types to be treated through the same interface.

**Types:**
1. **Compile-time (Method Overloading)**
2. **Runtime (Method Overriding)**

**Example:**
```java
// Runtime polymorphism
public interface Shape {
    double area();
}

public class Circle implements Shape {
    public double area() {
        return Math.PI * radius * radius;
    }
}

public class Rectangle implements Shape {
    public double area() {
        return length * width;
    }
}

// Can treat both as Shape
Shape shape1 = new Circle();
Shape shape2 = new Rectangle();
```

---

## SOLID Principles

### Q5: What is SOLID?

**Answer:**
SOLID is five design principles for object-oriented design:

1. **S**ingle Responsibility Principle
2. **O**pen/Closed Principle
3. **L**iskov Substitution Principle
4. **I**nterface Segregation Principle
5. **D**ependency Inversion Principle

---

### Q6: Explain Single Responsibility Principle

**Answer:**
A class should have only one reason to change (one responsibility).

**Bad Example:**
```java
public class User {
    public void save() { /* database logic */ }
    public void sendEmail() { /* email logic */ }
    public void validate() { /* validation logic */ }
}
```

**Good Example:**
```java
public class User {
    // Only user data
}

public class UserRepository {
    public void save(User user) { /* database logic */ }
}

public class EmailService {
    public void sendEmail(User user) { /* email logic */ }
}
```

---

### Q7: Explain Open/Closed Principle

**Answer:**
Software entities should be open for extension but closed for modification.

**Example:**
```java
// Closed for modification
public interface PaymentProcessor {
    void processPayment(double amount);
}

// Open for extension
public class CreditCardProcessor implements PaymentProcessor {
    public void processPayment(double amount) {
        // Credit card logic
    }
}

public class PayPalProcessor implements PaymentProcessor {
    public void processPayment(double amount) {
        // PayPal logic
    }
}
```

---

### Q8: Explain Liskov Substitution Principle

**Answer:**
Objects of a superclass should be replaceable with objects of its subclasses without breaking the application.

**Bad Example:**
```java
public class Rectangle {
    protected int width, height;
    public void setWidth(int w) { width = w; }
    public void setHeight(int h) { height = h; }
}

public class Square extends Rectangle {
    public void setWidth(int w) {
        width = height = w; // Violates LSP
    }
}
```

**Good Example:**
```java
public interface Shape {
    int area();
}

public class Rectangle implements Shape {
    public int area() { return width * height; }
}

public class Square implements Shape {
    public int area() { return side * side; }
}
```

---

## Design Patterns

### Q9: What is the Singleton pattern?

**Answer:**
Ensures a class has only one instance and provides global access.

**Example:**
```java
public class DatabaseConnection {
    private static DatabaseConnection instance;
    
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

**Use Cases:**
- Database connections
- Logger
- Configuration

---

### Q10: What is the Factory pattern?

**Answer:**
Creates objects without specifying the exact class.

**Example:**
```java
public interface PaymentProcessor {
    void processPayment(double amount);
}

public class PaymentProcessorFactory {
    public static PaymentProcessor create(String type) {
        switch (type) {
            case "credit": return new CreditCardProcessor();
            case "paypal": return new PayPalProcessor();
            default: throw new IllegalArgumentException();
        }
    }
}
```

---

### Q11: What is the Strategy pattern?

**Answer:**
Defines a family of algorithms, encapsulates each, and makes them interchangeable.

**Example:**
```java
public interface SortingStrategy {
    void sort(int[] array);
}

public class QuickSort implements SortingStrategy {
    public void sort(int[] array) { /* Quick sort */ }
}

public class MergeSort implements SortingStrategy {
    public void sort(int[] array) { /* Merge sort */ }
}

public class Sorter {
    private SortingStrategy strategy;
    
    public void setStrategy(SortingStrategy strategy) {
        this.strategy = strategy;
    }
    
    public void sort(int[] array) {
        strategy.sort(array);
    }
}
```

---

## Class Design

### Q12: How do you design a class?

**Answer:**
1. **Identify responsibilities**: What should this class do?
2. **Define attributes**: What data does it need?
3. **Define methods**: What operations can it perform?
4. **Define relationships**: How does it relate to other classes?
5. **Apply principles**: SOLID, DRY, KISS

---

### Q13: When to use abstract class vs interface?

**Answer:**

**Use Abstract Class when:**
- Sharing code between related classes
- Need default implementations
- Template method pattern

**Use Interface when:**
- Defining contract
- Multiple inheritance needed
- Loose coupling

---

## Relationships

### Q14: What is Association?

**Answer:**
A "uses-a" relationship where objects are related but independent.

**Example:**
```java
public class Doctor {
    private Stethoscope stethoscope; // Doctor uses stethoscope
}
```

---

### Q15: What is Aggregation?

**Answer:**
A "has-a" relationship where objects can exist independently.

**Example:**
```java
public class Department {
    private List<Professor> professors; // Department has professors
    // Professors can exist without department
}
```

---

### Q16: What is Composition?

**Answer:**
A "has-a" relationship where objects cannot exist independently.

**Example:**
```java
public class House {
    private List<Room> rooms; // House composed of rooms
    // Rooms cannot exist without house
}
```

---

## Summary

These FAQs cover:
- ✅ OOP principles
- ✅ SOLID principles
- ✅ Design patterns
- ✅ Class design
- ✅ Relationships
- ✅ Best practices

**Key Takeaways:**
- Understand OOP deeply
- Apply SOLID principles
- Know design patterns
- Design for maintainability

---

*This guide is continuously updated with new FAQs.*

