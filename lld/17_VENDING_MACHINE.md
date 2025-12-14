# Vending Machine

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Requirements Analysis](#requirements-analysis)
3. [State Machine Design](#state-machine-design)
4. [Class Design](#class-design)
5. [Complete Implementation](#complete-implementation)

---

## Problem Statement

Design a vending machine that can:
- Accept coins/bills
- Select products
- Dispense products
- Return change
- Handle different states (idle, has money, dispensing)

---

## Requirements Analysis

### Functional Requirements

1. **Money Handling**
   - Accept coins/bills
   - Calculate total inserted
   - Return change

2. **Product Selection**
   - Display available products
   - Select product
   - Check availability

3. **Dispensing**
   - Dispense product
   - Return change
   - Update inventory

---

## State Machine Design

### Vending Machine States

```
[Idle] → insertMoney() → [Has Money]
                              │
                              │ selectProduct()
                              ▼
                        [Product Selected]
                              │
                    ┌──────────┴──────────┐
                    │                    │
            sufficient?              insufficient?
                    │                    │
                    ▼                    ▼
            [Dispensing]         [Insufficient Money]
                    │                    │
                    │                    │ insertMoreMoney()
                    │                    └─────────────────┐
                    │                                      │
                    └──────────────────────────────────────┘
                                    │
                                    ▼
                            [Dispensing]
                                    │
                                    │ dispense()
                                    ▼
                              [Idle]
```

---

## Class Design

### Core Classes

```java
// Vending Machine State
public interface VendingMachineState {
    void insertMoney(double amount);
    void selectProduct(int productId);
    void dispense();
    void returnChange();
}

// Idle State
public class IdleState implements VendingMachineState {
    private VendingMachine machine;
    
    public IdleState(VendingMachine machine) {
        this.machine = machine;
    }
    
    public void insertMoney(double amount) {
        machine.addMoney(amount);
        machine.setState(new HasMoneyState(machine));
    }
    
    public void selectProduct(int productId) {
        throw new IllegalStateException("Insert money first");
    }
    
    public void dispense() {
        throw new IllegalStateException("No product selected");
    }
    
    public void returnChange() {
        throw new IllegalStateException("No money to return");
    }
}

// Has Money State
public class HasMoneyState implements VendingMachineState {
    private VendingMachine machine;
    
    public HasMoneyState(VendingMachine machine) {
        this.machine = machine;
    }
    
    public void insertMoney(double amount) {
        machine.addMoney(amount);
    }
    
    public void selectProduct(int productId) {
        Product product = machine.getProduct(productId);
        if (product == null || !product.isAvailable()) {
            throw new IllegalArgumentException("Product not available");
        }
        
        machine.setSelectedProduct(product);
        
        if (machine.getInsertedMoney() >= product.getPrice()) {
            machine.setState(new DispensingState(machine));
        } else {
            machine.setState(new InsufficientMoneyState(machine));
        }
    }
    
    public void dispense() {
        throw new IllegalStateException("Select product first");
    }
    
    public void returnChange() {
        double change = machine.getInsertedMoney();
        machine.returnMoney(change);
        machine.setState(new IdleState(machine));
    }
}

// Dispensing State
public class DispensingState implements VendingMachineState {
    private VendingMachine machine;
    
    public DispensingState(VendingMachine machine) {
        this.machine = machine;
    }
    
    public void insertMoney(double amount) {
        throw new IllegalStateException("Product dispensing");
    }
    
    public void selectProduct(int productId) {
        throw new IllegalStateException("Product already selected");
    }
    
    public void dispense() {
        Product product = machine.getSelectedProduct();
        double price = product.getPrice();
        double inserted = machine.getInsertedMoney();
        
        // Dispense product
        machine.dispenseProduct(product);
        
        // Return change
        if (inserted > price) {
            machine.returnMoney(inserted - price);
        }
        
        machine.setState(new IdleState(machine));
    }
    
    public void returnChange() {
        throw new IllegalStateException("Product dispensing");
    }
}

// Vending Machine
public class VendingMachine {
    private VendingMachineState state;
    private double insertedMoney;
    private Product selectedProduct;
    private Map<Integer, Product> products;
    
    public VendingMachine() {
        this.state = new IdleState(this);
        this.products = new HashMap<>();
        initializeProducts();
    }
    
    public void setState(VendingMachineState state) {
        this.state = state;
    }
    
    public void insertMoney(double amount) {
        state.insertMoney(amount);
    }
    
    public void selectProduct(int productId) {
        state.selectProduct(productId);
    }
    
    public void dispense() {
        state.dispense();
    }
    
    public void returnChange() {
        state.returnChange();
    }
    
    public void addMoney(double amount) {
        this.insertedMoney += amount;
    }
    
    public double getInsertedMoney() {
        return insertedMoney;
    }
    
    public Product getProduct(int productId) {
        return products.get(productId);
    }
    
    public void setSelectedProduct(Product product) {
        this.selectedProduct = product;
    }
    
    public Product getSelectedProduct() {
        return selectedProduct;
    }
    
    public void dispenseProduct(Product product) {
        product.reduceStock(1);
        System.out.println("Dispensing: " + product.getName());
    }
    
    public void returnMoney(double amount) {
        this.insertedMoney = 0;
        System.out.println("Returning change: " + amount);
    }
    
    private void initializeProducts() {
        products.put(1, new Product(1, "Coke", 1.50, 10));
        products.put(2, new Product(2, "Pepsi", 1.50, 10));
        products.put(3, new Product(3, "Water", 1.00, 10));
    }
}

// Product
public class Product {
    private int id;
    private String name;
    private double price;
    private int stock;
    
    public Product(int id, String name, double price, int stock) {
        this.id = id;
        this.name = name;
        this.price = price;
        this.stock = stock;
    }
    
    public boolean isAvailable() {
        return stock > 0;
    }
    
    public void reduceStock(int quantity) {
        if (stock < quantity) {
            throw new InsufficientStockException();
        }
        stock -= quantity;
    }
}
```

---

## Summary

**Key Components:**
1. ✅ **State Pattern:** Different vending machine states
2. ✅ **State Transitions:** Idle → Has Money → Dispensing
3. ✅ **Product Management:** Inventory tracking
4. ✅ **Money Handling:** Insert, calculate, return change

**Next Steps:**
- Practice [Tic-Tac-Toe](./18_TIC_TAC_TOE.md)
- Learn [State Pattern](./10_BEHAVIORAL_PATTERNS.md)
- Review [Design Patterns](./08_CREATIONAL_PATTERNS.md)

---

**References:**
- [State Pattern](./10_BEHAVIORAL_PATTERNS.md)
- [Class Design](./04_CLASS_DESIGN.md)

