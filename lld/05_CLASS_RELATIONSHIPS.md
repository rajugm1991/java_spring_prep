# Class Relationships in LLD

## Overview

Classes don't exist in isolation. Understanding and correctly modeling relationships between classes is crucial for good Low-Level Design.

---

## Types of Relationships

```
┌─────────────────────────────────────────┐
│         CLASS RELATIONSHIPS             │
└─────────────────────────────────────────┘

1. ASSOCIATION      ── Uses-A (general relationship)
2. AGGREGATION      ── Weak Has-A (independent lifecycle)
3. COMPOSITION      ── Strong Has-A (dependent lifecycle)
4. INHERITANCE      ── Is-A (parent-child)
5. DEPENDENCY       ── Uses temporarily
6. REALIZATION      ── Implements interface
```

---

## 1. Association (Uses-A)

**Definition:** A general relationship where one class uses another class. Objects can exist independently.

### Characteristics
- Weak relationship
- Objects have independent lifecycles
- Can be one-to-one, one-to-many, or many-to-many

### Example 1: Doctor and Stethoscope

```java
public class Doctor {
    private String name;
    
    public void useStethoscope(Stethoscope stethoscope) {
        // Doctor uses stethoscope
        stethoscope.examine();
    }
}

public class Stethoscope {
    private String brand;
    
    public void examine() {
        System.out.println("Examining with stethoscope...");
    }
}

// Usage
Doctor doctor = new Doctor("Dr. Smith");
Stethoscope stethoscope = new Stethoscope("3M");
doctor.useStethoscope(stethoscope);
// Both can exist independently
```

### Example 2: Student and Course (Many-to-Many)

```java
public class Student {
    private String studentId;
    private String name;
    private List<Course> courses; // Association
    
    public Student(String studentId, String name) {
        this.studentId = studentId;
        this.name = name;
        this.courses = new ArrayList<>();
    }
    
    public void enroll(Course course) {
        courses.add(course);
        course.addStudent(this);
    }
}

public class Course {
    private String courseId;
    private String name;
    private List<Student> students; // Association
    
    public Course(String courseId, String name) {
        this.courseId = courseId;
        this.name = name;
        this.students = new ArrayList<>();
    }
    
    public void addStudent(Student student) {
        students.add(student);
    }
}

// Usage
Student student1 = new Student("S1", "Alice");
Student student2 = new Student("S2", "Bob");
Course course1 = new Course("C1", "Java");
Course course2 = new Course("C2", "Python");

student1.enroll(course1);
student1.enroll(course2);
student2.enroll(course1);
// Many-to-many relationship
```

### UML Notation
```
Doctor ───────> Stethoscope
Student ───────> Course (many-to-many)
```

---

## 2. Aggregation (Weak Has-A)

**Definition:** A "has-a" relationship where the child can exist independently of the parent. It's a weak form of composition.

### Characteristics
- Weak ownership
- Child can exist without parent
- Parent and child have independent lifecycles
- Represented by empty diamond in UML

### Example 1: Department and Professor

```java
public class Department {
    private String name;
    private List<Professor> professors; // Aggregation
    
    public Department(String name) {
        this.name = name;
        this.professors = new ArrayList<>();
    }
    
    public void addProfessor(Professor professor) {
        professors.add(professor);
    }
    
    public void removeProfessor(Professor professor) {
        professors.remove(professor);
    }
}

public class Professor {
    private String name;
    private String specialization;
    
    public Professor(String name, String specialization) {
        this.name = name;
        this.specialization = specialization;
    }
}

// Usage
Department csDept = new Department("Computer Science");
Professor prof1 = new Professor("Dr. Smith", "AI");
Professor prof2 = new Professor("Dr. Jones", "DB");

csDept.addProfessor(prof1);
csDept.addProfessor(prof2);

// If department closes, professors still exist
csDept = null;
// prof1 and prof2 still exist independently
```

### Example 2: Team and Player

```java
public class Team {
    private String name;
    private List<Player> players; // Aggregation
    
    public Team(String name) {
        this.name = name;
        this.players = new ArrayList<>();
    }
    
    public void addPlayer(Player player) {
        players.add(player);
    }
}

public class Player {
    private String name;
    private String position;
    
    public Player(String name, String position) {
        this.name = name;
        this.position = position;
    }
}

// Usage
Team team = new Team("Lakers");
Player player1 = new Player("LeBron", "Forward");
Player player2 = new Player("Davis", "Center");

team.addPlayer(player1);
team.addPlayer(player2);

// Player can leave team and still exist
team.removePlayer(player1);
// player1 still exists
```

### UML Notation
```
Department ◇────> Professor
Team ◇────> Player
```

---

## 3. Composition (Strong Has-A)

**Definition:** A "has-a" relationship where the child cannot exist without the parent. Strong ownership.

### Characteristics
- Strong ownership
- Child cannot exist without parent
- Parent and child have dependent lifecycles
- When parent is destroyed, child is destroyed
- Represented by filled diamond in UML

### Example 1: House and Room

```java
public class House {
    private String address;
    private List<Room> rooms; // Composition
    
    public House(String address) {
        this.address = address;
        this.rooms = new ArrayList<>();
        // Rooms are created with house
        rooms.add(new Room("Living Room"));
        rooms.add(new Room("Bedroom"));
        rooms.add(new Room("Kitchen"));
    }
    
    public void addRoom(Room room) {
        rooms.add(room);
    }
    
    // When house is destroyed, rooms are destroyed too
    public void demolish() {
        rooms.clear();
    }
}

public class Room {
    private String name;
    
    public Room(String name) {
        this.name = name;
    }
}

// Usage
House house = new House("123 Main St");
// Rooms are part of house
// If house is demolished, rooms don't exist independently
```

### Example 2: Order and OrderItem

```java
public class Order {
    private String orderId;
    private List<OrderItem> items; // Composition
    
    public Order(String orderId) {
        this.orderId = orderId;
        this.items = new ArrayList<>();
    }
    
    public void addItem(Product product, int quantity) {
        OrderItem item = new OrderItem(product, quantity);
        items.add(item);
    }
    
    // Order items don't exist without order
    public void cancel() {
        items.clear(); // Items are destroyed with order
    }
}

public class OrderItem {
    private Product product;
    private int quantity;
    private BigDecimal price;
    
    public OrderItem(Product product, int quantity) {
        this.product = product;
        this.quantity = quantity;
        this.price = product.getPrice().multiply(new BigDecimal(quantity));
    }
}

// Usage
Order order = new Order("ORD-123");
order.addItem(product1, 2);
order.addItem(product2, 1);

// Order items are part of order
// If order is cancelled, items are removed
```

### UML Notation
```
House ◆────> Room
Order ◆────> OrderItem
```

---

## 4. Inheritance (Is-A)

**Definition:** A relationship where a child class inherits properties and behaviors from a parent class.

### Characteristics
- "Is-a" relationship
- Code reuse
- Method overriding
- Polymorphism

### Example: Vehicle Hierarchy

```java
// Parent class
public class Vehicle {
    protected String brand;
    protected String model;
    
    public Vehicle(String brand, String model) {
        this.brand = brand;
        this.model = model;
    }
    
    public void start() {
        System.out.println("Vehicle starting...");
    }
    
    public void stop() {
        System.out.println("Vehicle stopping...");
    }
}

// Child class
public class Car extends Vehicle {
    private int numberOfDoors;
    
    public Car(String brand, String model, int numberOfDoors) {
        super(brand, model);
        this.numberOfDoors = numberOfDoors;
    }
    
    @Override
    public void start() {
        System.out.println("Car engine starting...");
    }
    
    public void honk() {
        System.out.println("Beep beep!");
    }
}

// Another child class
public class Motorcycle extends Vehicle {
    private boolean hasSidecar;
    
    public Motorcycle(String brand, String model, boolean hasSidecar) {
        super(brand, model);
        this.hasSidecar = hasSidecar;
    }
    
    @Override
    public void start() {
        System.out.println("Motorcycle engine revving...");
    }
}
```

### UML Notation
```
Vehicle
  ▲
  │
  ├─── Car
  └─── Motorcycle
```

---

## 5. Dependency (Uses Temporarily)

**Definition:** A relationship where one class uses another class temporarily, but doesn't own it.

### Characteristics
- Temporary relationship
- Usually method parameters or local variables
- Weakest relationship

### Example: OrderService and PaymentProcessor

```java
public class OrderService {
    // OrderService depends on PaymentProcessor
    public void processOrder(Order order, PaymentProcessor processor) {
        // Uses processor temporarily
        PaymentResult result = processor.processPayment(order.getAmount());
        // processor is not stored as instance variable
    }
}

public interface PaymentProcessor {
    PaymentResult processPayment(BigDecimal amount);
}
```

### UML Notation
```
OrderService ──> PaymentProcessor (dashed arrow)
```

---

## 6. Realization (Implements Interface)

**Definition:** A class implements an interface, providing concrete implementation for abstract methods.

### Example: Payment Processors

```java
// Interface
public interface PaymentProcessor {
    PaymentResult processPayment(PaymentRequest request);
}

// Implementation
public class StripeProcessor implements PaymentProcessor {
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        // Stripe implementation
        return new PaymentResult("stripe_123", PaymentStatus.SUCCESS);
    }
}

public class RazorpayProcessor implements PaymentProcessor {
    @Override
    public PaymentResult processPayment(PaymentRequest request) {
        // Razorpay implementation
        return new PaymentResult("razorpay_456", PaymentStatus.SUCCESS);
    }
}
```

### UML Notation
```
PaymentProcessor (interface)
  ▲
  │ (implements)
  │
  ├─── StripeProcessor
  └─── RazorpayProcessor
```

---

## Relationship Comparison

| Relationship | Strength | Lifecycle | UML Symbol | Example |
|--------------|----------|-----------|------------|---------|
| **Association** | Weak | Independent | `──>` | Doctor uses Stethoscope |
| **Aggregation** | Medium | Independent | `◇──>` | Department has Professors |
| **Composition** | Strong | Dependent | `◆──>` | House has Rooms |
| **Inheritance** | Strong | Dependent | `▲` | Car is a Vehicle |
| **Dependency** | Weakest | Temporary | `──>` (dashed) | Method parameter |
| **Realization** | Medium | Independent | `──>` (dashed) | Implements interface |

---

## Real-World Example: E-Commerce System

```java
// COMPOSITION: Order has OrderItems
public class Order {
    private String orderId;
    private List<OrderItem> items; // Composition - items don't exist without order
    
    public Order(String orderId) {
        this.orderId = orderId;
        this.items = new ArrayList<>();
    }
}

// AGGREGATION: Order has Customer (customer exists independently)
public class Order {
    private Customer customer; // Aggregation - customer exists without order
}

// ASSOCIATION: Order uses PaymentProcessor
public class Order {
    public void processPayment(PaymentProcessor processor) {
        // Association - uses processor temporarily
        processor.processPayment(this.getTotal());
    }
}

// INHERITANCE: PremiumOrder is an Order
public class PremiumOrder extends Order {
    // PremiumOrder IS-A Order
}

// REALIZATION: StripeProcessor implements PaymentProcessor
public class StripeProcessor implements PaymentProcessor {
    // Realization - implements interface
}
```

---

## Choosing the Right Relationship

### When to Use Association?
- Objects use each other but are independent
- Example: Student enrolls in Course

### When to Use Aggregation?
- Parent "has" child, but child can exist alone
- Example: Team has Players (players can leave team)

### When to Use Composition?
- Child cannot exist without parent
- Example: Order has OrderItems (items don't exist without order)

### When to Use Inheritance?
- Clear "is-a" relationship
- Need code reuse
- Example: Car is a Vehicle

### When to Use Dependency?
- Temporary use
- Method parameters
- Example: Service uses Repository in method

### When to Use Realization?
- Need to implement contract
- Multiple implementations
- Example: PaymentProcessor interface

---

## Key Takeaways

1. **Association**: General "uses" relationship
2. **Aggregation**: Weak "has-a" (independent lifecycle)
3. **Composition**: Strong "has-a" (dependent lifecycle)
4. **Inheritance**: "Is-a" relationship
5. **Dependency**: Temporary use
6. **Realization**: Implements interface

**Remember:** Choose the relationship that best represents the real-world relationship between objects.

---

## Next Steps

1. Practice [Class Design](./04_CLASS_DESIGN.md)
2. Study [Interfaces and Abstractions](./06_INTERFACES_AND_ABSTRACTIONS.md)
3. Learn [Design Patterns](./08_CREATIONAL_PATTERNS.md)

