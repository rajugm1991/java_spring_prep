# ATM System

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Requirements Analysis](#requirements-analysis)
3. [State Machine Design](#state-machine-design)
4. [Class Design](#class-design)
5. [Complete Implementation](#complete-implementation)

---

## Problem Statement

Design an ATM system that can:
- Authenticate users
- Check balance
- Withdraw cash
- Deposit cash
- Transfer money
- Handle different states (idle, authenticated, transaction)

---

## Requirements Analysis

### Functional Requirements

1. **Authentication**
   - Card insertion
   - PIN entry
   - Validation

2. **Transactions**
   - Balance inquiry
   - Cash withdrawal
   - Cash deposit
   - Transfer

3. **Security**
   - PIN validation
   - Transaction limits
   - Session timeout

---

## State Machine Design

### ATM States

```
[No Card] → insertCard() → [Has Card]
                                    │
                                    │ enterPin()
                                    ▼
                            [Has Pin]
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
            selectTransaction()  withdraw()    deposit()
                    │               │               │
                    ▼               ▼               ▼
            [Transaction]    [Withdrawing]   [Depositing]
                    │               │               │
                    └───────────────┼───────────────┘
                                    │
                                    │ complete()
                                    ▼
                            [Has Pin]
                                    │
                                    │ ejectCard()
                                    ▼
                            [No Card]
```

---

## Class Design

### Core Classes

```java
// ATM State interface
public interface ATMState {
    void insertCard();
    void enterPin(String pin);
    void selectTransaction(TransactionType type);
    void withdraw(double amount);
    void deposit(double amount);
    void ejectCard();
}

// No Card State
public class NoCardState implements ATMState {
    private ATM atm;
    
    public NoCardState(ATM atm) {
        this.atm = atm;
    }
    
    public void insertCard() {
        System.out.println("Card inserted");
        atm.setState(new HasCardState(atm));
    }
    
    public void enterPin(String pin) {
        throw new IllegalStateException("No card inserted");
    }
    
    public void selectTransaction(TransactionType type) {
        throw new IllegalStateException("No card inserted");
    }
    
    public void withdraw(double amount) {
        throw new IllegalStateException("No card inserted");
    }
    
    public void deposit(double amount) {
        throw new IllegalStateException("No card inserted");
    }
    
    public void ejectCard() {
        throw new IllegalStateException("No card to eject");
    }
}

// Has Card State
public class HasCardState implements ATMState {
    private ATM atm;
    
    public HasCardState(ATM atm) {
        this.atm = atm;
    }
    
    public void enterPin(String pin) {
        if (validatePin(pin)) {
            System.out.println("PIN correct");
            atm.setState(new HasPinState(atm));
        } else {
            System.out.println("Invalid PIN");
            atm.setState(new NoCardState(atm));
        }
    }
    
    private boolean validatePin(String pin) {
        // Validate PIN logic
        return true;
    }
    
    // Other methods throw IllegalStateException
}

// Has Pin State
public class HasPinState implements ATMState {
    private ATM atm;
    
    public HasPinState(ATM atm) {
        this.atm = atm;
    }
    
    public void selectTransaction(TransactionType type) {
        System.out.println("Transaction selected: " + type);
        atm.setTransactionType(type);
    }
    
    public void withdraw(double amount) {
        if (atm.getBalance() >= amount) {
            System.out.println("Withdrawing: " + amount);
            atm.deductBalance(amount);
            System.out.println("Please take your cash");
        } else {
            System.out.println("Insufficient balance");
        }
    }
    
    public void deposit(double amount) {
        System.out.println("Depositing: " + amount);
        atm.addBalance(amount);
        System.out.println("Deposit successful");
    }
    
    public void ejectCard() {
        System.out.println("Card ejected");
        atm.setState(new NoCardState(atm));
    }
    
    // Other methods
}

// ATM class
public class ATM {
    private ATMState state;
    private String cardNumber;
    private double balance;
    private TransactionType transactionType;
    
    public ATM() {
        this.state = new NoCardState(this);
        this.balance = 1000.0;  // Example balance
    }
    
    public void setState(ATMState state) {
        this.state = state;
    }
    
    public void insertCard() {
        state.insertCard();
    }
    
    public void enterPin(String pin) {
        state.enterPin(pin);
    }
    
    public void withdraw(double amount) {
        state.withdraw(amount);
    }
    
    public void deposit(double amount) {
        state.deposit(amount);
    }
    
    public void ejectCard() {
        state.ejectCard();
    }
    
    public double getBalance() {
        return balance;
    }
    
    public void deductBalance(double amount) {
        balance -= amount;
    }
    
    public void addBalance(double amount) {
        balance += amount;
    }
}
```

---

## Summary

**Key Components:**
1. ✅ **State Pattern:** Different ATM states
2. ✅ **State Transitions:** Card → Pin → Transaction
3. ✅ **Security:** PIN validation, balance checks
4. ✅ **Transactions:** Withdraw, deposit, transfer

**Next Steps:**
- Practice [Vending Machine](./17_VENDING_MACHINE.md)
- Learn [State Pattern](./10_BEHAVIORAL_PATTERNS.md)
- Review [Design Patterns](./08_CREATIONAL_PATTERNS.md)

---

**References:**
- [State Pattern](./10_BEHAVIORAL_PATTERNS.md)
- [Class Design](./04_CLASS_DESIGN.md)

