# LLD Interview Preparation Guide

## Overview

This guide helps you prepare for Low-Level Design (LLD) interviews at staff engineer level. It covers common questions, approach, tips, and what interviewers look for.

---

## What Interviewers Evaluate

### 1. Problem-Solving Ability
- Can you break down complex problems?
- Do you ask clarifying questions?
- Can you identify edge cases?

### 2. OOP Understanding
- Do you apply OOP principles correctly?
- Can you design proper class hierarchies?
- Do you understand relationships (inheritance, composition, etc.)?

### 3. Design Principles
- SOLID principles
- DRY (Don't Repeat Yourself)
- KISS (Keep It Simple, Stupid)
- YAGNI (You Aren't Gonna Need It)

### 4. Design Patterns
- When to use which pattern
- Can you recognize patterns in problems?
- Do you apply patterns appropriately?

### 5. Code Quality
- Clean, readable code
- Proper naming conventions
- Good method signatures
- Error handling

### 6. Communication
- Can you explain your design choices?
- Can you justify trade-offs?
- Do you communicate clearly?

---

## Interview Process

### Step 1: Understand Requirements (5-10 minutes)

**Ask Clarifying Questions:**

1. **Scope Questions:**
   - What are the main features?
   - What's out of scope?
   - Any constraints?

2. **Scale Questions:**
   - How many users?
   - Expected load?
   - Data volume?

3. **Functional Questions:**
   - What are the use cases?
   - Any special requirements?
   - Edge cases to consider?

**Example: Parking Lot System**

❌ **Bad:** Start coding immediately

✅ **Good:**
- "What vehicle types should we support?"
- "How many floors and spots?"
- "Do we need to support reservations?"
- "What's the pricing model?"
- "Do we need to handle concurrent access?"

### Step 2: Identify Core Entities (5 minutes)

**List Main Classes:**

1. Identify nouns in requirements → potential classes
2. Identify verbs → potential methods
3. Group related functionality

**Example: Library Management System**

**Nouns:** Library, Book, Member, Librarian, Transaction
**Verbs:** Issue, Return, Search, Reserve

**Classes:**
- `Library`
- `Book`
- `Member`
- `Librarian`
- `Transaction`

### Step 3: Define Relationships (5 minutes)

**Map Relationships:**

- Association
- Aggregation
- Composition
- Inheritance

**Example:**
```
Library ──has──> List<Book>
Library ──has──> List<Member>
Member ──has──> List<Transaction>
Transaction ──uses──> Book
Transaction ──uses──> Member
```

### Step 4: Design Classes (10-15 minutes)

**For Each Class:**
- Attributes (fields)
- Methods
- Relationships
- Responsibilities

**Example: Book Class**
```java
public class Book {
    // Attributes
    private String isbn;
    private String title;
    private String author;
    private BookStatus status;
    
    // Methods
    public void issue(Member member) { ... }
    public void returnBook() { ... }
    public boolean isAvailable() { ... }
}
```

### Step 5: Apply Design Patterns (5-10 minutes)

**Identify Where Patterns Fit:**

- Need single instance? → Singleton
- Need to create objects? → Factory
- Need to add behavior? → Decorator
- Need to notify observers? → Observer

### Step 6: Code Implementation (20-30 minutes)

**Write Clean Code:**
- Follow naming conventions
- Add proper access modifiers
- Handle edge cases
- Add validation

### Step 7: Discuss Trade-offs (5 minutes)

**Be Ready to Explain:**
- Why you chose this design
- Alternatives considered
- Trade-offs made
- How to extend the design

---

## Common Interview Problems

### 1. Parking Lot System
**Key Points:**
- Vehicle types (Car, Motorcycle, Truck)
- Spot types (Small, Medium, Large)
- Multiple floors
- Pricing strategy
- Thread safety

**Classes:**
- `ParkingLot`, `ParkingFloor`, `ParkingSpot`
- `Vehicle`, `Car`, `Motorcycle`, `Truck`
- `Ticket`, `Receipt`
- `PricingStrategy`

### 2. Library Management System
**Key Points:**
- Books, Members, Librarians
- Issue/Return books
- Search functionality
- Fine calculation
- Reservation system

**Classes:**
- `Library`, `Book`, `Member`, `Librarian`
- `Transaction`, `Fine`
- `SearchService`, `ReservationService`

### 3. Elevator System
**Key Points:**
- Multiple elevators
- Multiple floors
- Request handling
- Direction (Up/Down)
- Thread safety

**Classes:**
- `Elevator`, `ElevatorController`
- `Floor`, `Request`
- `ElevatorState` (enum)

### 4. ATM System
**Key Points:**
- Account management
- Transaction types (Withdraw, Deposit, Transfer)
- State machine (Idle, Processing, etc.)
- Security (PIN validation)

**Classes:**
- `ATM`, `Account`, `Card`
- `Transaction`, `WithdrawTransaction`, `DepositTransaction`
- `ATMState`

### 5. Online Shopping System
**Key Points:**
- Products, Cart, Order
- Payment processing
- Inventory management
- Shipping

**Classes:**
- `Product`, `Cart`, `Order`
- `PaymentProcessor`, `InventoryService`
- `ShippingService`

### 6. Vending Machine
**Key Points:**
- Products and inventory
- Payment (Cash, Card)
- State machine
- Change calculation

**Classes:**
- `VendingMachine`, `Product`
- `PaymentProcessor`, `ChangeCalculator`
- `VendingMachineState`

### 7. Tic-Tac-Toe
**Key Points:**
- Board representation
- Player turns
- Win condition checking
- AI player (optional)

**Classes:**
- `Board`, `Player`, `Game`
- `Move`, `GameResult`

### 8. Snake and Ladder
**Key Points:**
- Board with snakes and ladders
- Players
- Dice rolling
- Game logic

**Classes:**
- `Board`, `Player`, `Game`
- `Snake`, `Ladder`, `Dice`

### 9. Chess Game
**Key Points:**
- Board and pieces
- Move validation
- Turn management
- Check/Checkmate detection

**Classes:**
- `Board`, `Piece`, `King`, `Queen`, etc.
- `Move`, `Game`, `Player`

### 10. Splitwise
**Key Points:**
- Users and groups
- Expenses and splits
- Balance calculation
- Settlement

**Classes:**
- `User`, `Group`, `Expense`
- `Split`, `Balance`, `Settlement`

---

## Common Interview Questions

### Q1: Design a Parking Lot System

**Approach:**
1. Ask about vehicle types, floors, spots
2. Identify classes: Vehicle, ParkingSpot, ParkingFloor, ParkingLot
3. Design relationships
4. Add pricing strategy
5. Handle concurrency

**Key Classes:**
```java
public class ParkingLot {
    private List<ParkingFloor> floors;
    public Ticket parkVehicle(Vehicle vehicle) { ... }
    public Receipt unparkVehicle(Ticket ticket) { ... }
}

public class ParkingSpot {
    private SpotType type;
    private Vehicle vehicle;
    public boolean canFit(VehicleType vehicleType) { ... }
}
```

### Q2: Design a Library Management System

**Approach:**
1. Identify entities: Book, Member, Librarian
2. Operations: Issue, Return, Search, Reserve
3. Fine calculation
4. Book availability

**Key Classes:**
```java
public class Library {
    private List<Book> books;
    private List<Member> members;
    public void issueBook(Book book, Member member) { ... }
    public void returnBook(Book book, Member member) { ... }
}
```

### Q3: Design an Elevator System

**Approach:**
1. Multiple elevators, multiple floors
2. Request handling (internal/external)
3. Direction management
4. Thread safety

**Key Classes:**
```java
public class Elevator {
    private int currentFloor;
    private Direction direction;
    private Queue<Request> requests;
    public void move() { ... }
}

public class ElevatorController {
    private List<Elevator> elevators;
    public void requestElevator(int floor, Direction direction) { ... }
}
```

---

## Design Patterns in Interviews

### When to Use Which Pattern

**Singleton:**
- "Only one instance needed"
- Example: Database connection, Logger

**Factory:**
- "Create objects without specifying exact class"
- Example: Payment processor factory

**Builder:**
- "Complex object construction"
- Example: Order builder with many optional fields

**Strategy:**
- "Interchangeable algorithms"
- Example: Pricing strategies, Sorting algorithms

**Observer:**
- "Notify multiple objects of changes"
- Example: Stock price updates, Event notifications

**Decorator:**
- "Add behavior dynamically"
- Example: Coffee with milk, sugar, etc.

**Adapter:**
- "Make incompatible interfaces work together"
- Example: Third-party payment integration

**State:**
- "Object behavior changes with state"
- Example: ATM states, Order states

---

## Code Quality Checklist

### ✅ Good Code

```java
// Clear naming
public class OrderService {
    private OrderRepository repository;
    
    public Order createOrder(OrderRequest request) {
        validateRequest(request);
        Order order = buildOrder(request);
        return repository.save(order);
    }
    
    private void validateRequest(OrderRequest request) {
        if (request == null) {
            throw new IllegalArgumentException("Request cannot be null");
        }
    }
}
```

### ❌ Bad Code

```java
// Unclear naming, no validation
public class OS {
    private OR r;
    
    public Order o(OR req) {
        Order o = new Order();
        return r.s(o);
    }
}
```

### Best Practices

1. **Naming:**
   - Classes: Noun (Order, User, Payment)
   - Methods: Verb (createOrder, processPayment)
   - Variables: Descriptive (orderId, not o)

2. **Access Modifiers:**
   - Private for internal state
   - Public for API
   - Protected for inheritance

3. **Method Design:**
   - Single responsibility
   - Clear parameters
   - Meaningful return types
   - Handle exceptions

4. **Validation:**
   - Validate inputs
   - Check nulls
   - Validate state transitions

---

## Common Mistakes to Avoid

### 1. Starting to Code Too Early
❌ Start coding without understanding requirements
✅ Ask questions, clarify requirements first

### 2. Over-Engineering
❌ Use complex patterns everywhere
✅ Keep it simple, use patterns when needed

### 3. Ignoring Edge Cases
❌ Only handle happy path
✅ Consider edge cases, null checks, validation

### 4. Poor Naming
❌ Short, unclear names
✅ Descriptive, self-documenting names

### 5. Tight Coupling
❌ Direct dependencies on concrete classes
✅ Depend on abstractions (interfaces)

### 6. No Error Handling
❌ Assume everything works
✅ Handle exceptions, validate inputs

### 7. Not Explaining Design
❌ Just write code silently
✅ Explain your choices, discuss trade-offs

---

## Sample Interview Flow

### Interviewer: "Design a Parking Lot System"

**You:**
1. "Let me clarify requirements first..."
   - "What vehicle types? (Car, Motorcycle, Truck)"
   - "How many floors and spots?"
   - "Do we need reservations?"
   - "What's the pricing model?"

2. "Let me identify the main entities..."
   - ParkingLot, ParkingFloor, ParkingSpot
   - Vehicle (Car, Motorcycle, Truck)
   - Ticket, Receipt

3. "Let me design the classes..."
   ```java
   public class ParkingLot {
       private List<ParkingFloor> floors;
       public Ticket parkVehicle(Vehicle vehicle) { ... }
   }
   ```

4. "I'll use Strategy pattern for pricing..."
   ```java
   public interface PricingStrategy {
       BigDecimal calculateFee(Ticket ticket);
   }
   ```

5. "For thread safety, I'll use ConcurrentHashMap..."

6. "Trade-offs: This design is extensible but might be overkill for a simple parking lot..."

---

## Practice Tips

### 1. Practice Common Problems
- Solve 5-10 common LLD problems
- Time yourself (45-60 minutes)
- Code on whiteboard/paper first

### 2. Review Design Patterns
- Know when to use each pattern
- Practice applying patterns
- Understand trade-offs

### 3. Study SOLID Principles
- Understand each principle
- Practice applying them
- Recognize violations

### 4. Code Review
- Review your own code
- Look for improvements
- Refactor for better design

### 5. Mock Interviews
- Practice with friends
- Get feedback
- Improve communication

---

## Key Takeaways

1. **Ask Questions First**: Don't start coding immediately
2. **Think Before Coding**: Design classes and relationships
3. **Apply SOLID**: Use design principles
4. **Use Patterns Appropriately**: Don't over-engineer
5. **Write Clean Code**: Good naming, validation, error handling
6. **Communicate**: Explain your design choices
7. **Discuss Trade-offs**: Show you understand alternatives

---

## Resources

1. [SOLID Principles](./03_SOLID_PRINCIPLES.md)
2. [Design Patterns](./08_CREATIONAL_PATTERNS.md)
3. [Parking Lot System](./12_PARKING_LOT_SYSTEM.md)
4. [OOP Principles](./02_OOP_PRINCIPLES_FOR_LLD.md)

---

**Good luck with your interviews! 🚀**

