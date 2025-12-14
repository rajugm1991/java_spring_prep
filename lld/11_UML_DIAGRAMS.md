# UML Diagrams

## Table of Contents
1. [Introduction](#introduction)
2. [Class Diagrams](#class-diagrams)
3. [Sequence Diagrams](#sequence-diagrams)
4. [Use Case Diagrams](#use-case-diagrams)
5. [Activity Diagrams](#activity-diagrams)
6. [State Diagrams](#state-diagrams)
7. [Real-World Examples](#real-world-examples)

---

## Introduction

**UML (Unified Modeling Language)** is a visual language for modeling software systems. It helps communicate design ideas clearly.

### Types of UML Diagrams

1. **Structural:** Class, Component, Deployment
2. **Behavioral:** Sequence, Use Case, Activity, State

---

## Class Diagrams

**Class Diagram** shows classes, their attributes, methods, and relationships.

### Basic Elements

```
┌─────────────────────┐
│      ClassName      │
├─────────────────────┤
│   - attribute1      │  ← Private attribute
│   + attribute2      │  ← Public attribute
│   # attribute3      │  ← Protected attribute
├─────────────────────┤
│   + method1()       │  ← Public method
│   - method2()       │  ← Private method
│   # method3()        │  ← Protected method
└─────────────────────┘
```

### Access Modifiers

- `+` Public
- `-` Private
- `#` Protected
- `~` Package

---

### Example: E-Commerce System

```
┌─────────────────────┐
│       User          │
├─────────────────────┤
│   - userId: Long    │
│   - name: String    │
│   - email: String   │
├─────────────────────┤
│   + getName()       │
│   + getEmail()      │
│   + updateProfile() │
└─────────────────────┘
         │
         │ 1
         │
         ▼
┌─────────────────────┐
│       Order         │
├─────────────────────┤
│   - orderId: String │
│   - userId: Long    │
│   - total: double   │
│   - status: String  │
├─────────────────────┤
│   + createOrder()   │
│   + cancelOrder()   │
│   + calculateTotal()│
└─────────────────────┘
         │
         │ *
         │
         ▼
┌─────────────────────┐
│     OrderItem       │
├─────────────────────┤
│   - productId: Long │
│   - quantity: int   │
│   - price: double   │
└─────────────────────┘
```

---

### Relationships

#### 1. **Association** (Uses-a)

```
User ────────> Order
     (places)
```

**Code:**
```java
public class User {
    private List<Order> orders;  // Association
}
```

---

#### 2. **Aggregation** (Has-a, weak)

```
Department ◇─────── Professor
```

**Code:**
```java
public class Department {
    private List<Professor> professors;  // Aggregation
    // Professors can exist without department
}
```

---

#### 3. **Composition** (Has-a, strong)

```
House ◆─────── Room
```

**Code:**
```java
public class House {
    private List<Room> rooms;  // Composition
    // Rooms destroyed when house is destroyed
}
```

---

#### 4. **Inheritance** (Is-a)

```
Vehicle
   ▲
   │
   ├──────────┬──────────┐
   │          │          │
  Car       Truck      Bike
```

**Code:**
```java
public class Car extends Vehicle { ... }
```

---

## Sequence Diagrams

**Sequence Diagram** shows interactions between objects over time.

### Example: Order Processing

```
User          OrderService    PaymentService    InventoryService
 │                  │                │                  │
 │  createOrder()   │                │                  │
 ├─────────────────>│                │                  │
 │                  │                │                  │
 │                  │  processPayment()                │
 │                  ├────────────────>│                  │
 │                  │                │                  │
 │                  │                │  PaymentResult  │
 │                  │<───────────────┤                  │
 │                  │                │                  │
 │                  │  updateInventory()               │
 │                  ├───────────────────────────────────>│
 │                  │                │                  │
 │                  │                │                  │
 │  OrderResult     │                │                  │
 │<─────────────────┤                │                  │
 │                  │                │                  │
```

---

### Elements

- **Lifeline:** Vertical line representing object
- **Activation:** Box on lifeline (object is active)
- **Message:** Arrow between lifelines
- **Return:** Dashed arrow (optional)

---

## Use Case Diagrams

**Use Case Diagram** shows system functionality from user's perspective.

### Example: E-Commerce System

```
                    ┌─────────────────────┐
                    │  E-Commerce System  │
                    └─────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
        │                   │                   │
   ┌────▼────┐        ┌─────▼─────┐      ┌─────▼─────┐
   │ Browse  │        │ Add to    │      │ Checkout  │
   │Products │        │  Cart     │      │           │
   └─────────┘        └───────────┘      └───────────┘
        │                   │                   │
        └───────────────────┼───────────────────┘
                            │
                    ┌───────▼────────┐
                    │     User        │
                    └────────────────┘
```

---

## Activity Diagrams

**Activity Diagram** shows workflow or business process.

### Example: Order Processing Flow

```
[Start] → [Validate Order] → {Valid?}
                              │
                    ┌─────────┴─────────┐
                    │                   │
                   Yes                  No
                    │                   │
                    ▼                   ▼
        [Process Payment]      [Return Error]
                    │
            {Success?}
                    │
        ┌───────────┴───────────┐
        │                       │
       Yes                      No
        │                       │
        ▼                       ▼
[Update Inventory]      [Refund Payment]
        │
        ▼
[Send Confirmation]
        │
        ▼
      [End]
```

---

## State Diagrams

**State Diagram** shows state transitions of an object.

### Example: Order State Machine

```
     [Start]
        │
        ▼
   ┌─────────┐
   │ PENDING │
   └────┬────┘
        │ process()
        ▼
  ┌──────────┐
  │PROCESSING│
  └────┬─────┘
       │ ship()
       ▼
   ┌────────┐
   │ SHIPPED│
   └────┬───┘
        │ deliver()
        ▼
   ┌──────────┐
   │ DELIVERED│
   └──────────┘

   [PENDING] ──cancel()──> [CANCELLED]
```

---

## Real-World Examples

### Example 1: Parking Lot System Class Diagram

```
┌─────────────────────┐
│   ParkingLot        │
├─────────────────────┤
│   - floors: List    │
├─────────────────────┤
│   + parkVehicle()   │
│   + unparkVehicle() │
└──────────┬──────────┘
           │ 1
           │
           ▼
┌─────────────────────┐
│   ParkingFloor      │
├─────────────────────┤
│   - spots: List     │
└──────────┬──────────┘
           │ 1
           │
           ▼
┌─────────────────────┐
│   ParkingSpot       │
├─────────────────────┤
│   - spotId: int     │
│   - type: SpotType  │
│   - isOccupied: bool│
└─────────────────────┘
```

---

### Example 2: Payment Processing Sequence

```
Client    PaymentService    PaymentGateway    Bank
  │             │                 │            │
  │ process()   │                 │            │
  ├────────────>│                 │            │
  │             │                 │            │
  │             │ charge()        │            │
  │             ├────────────────>│            │
  │             │                 │            │
  │             │                 │ authorize()│
  │             │                 ├───────────>│
  │             │                 │            │
  │             │                 │  Response │
  │             │                 │<───────────┤
  │             │                 │            │
  │             │  Result         │            │
  │             │<────────────────┤            │
  │             │                 │            │
  │  Result     │                 │            │
  │<────────────┤                 │            │
```

---

## Summary

**Key Diagrams:**
1. ✅ **Class Diagram:** Structure and relationships
2. ✅ **Sequence Diagram:** Interactions over time
3. ✅ **Use Case Diagram:** System functionality
4. ✅ **Activity Diagram:** Workflow
5. ✅ **State Diagram:** State transitions

**Next Steps:**
- Practice [Real-World Problems](./12_PARKING_LOT_SYSTEM.md)
- Learn [Design Patterns](./08_CREATIONAL_PATTERNS.md)
- Review [Class Relationships](./05_CLASS_RELATIONSHIPS.md)

---

**References:**
- [UML Documentation](https://www.uml.org/)
- [Class Design](./04_CLASS_DESIGN.md)

