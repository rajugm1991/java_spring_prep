# Designing a Payment System (Stripe)

## Table of Contents
1. [Problem Statement](#problem-statement)
2. [Requirements Gathering](#requirements-gathering)
3. [Capacity Estimation](#capacity-estimation)
4. [High-Level Design](#high-level-design)
5. [Transaction Processing](#transaction-processing)
6. [Fraud Detection](#fraud-detection)
7. [Idempotency](#idempotency)

---

## Problem Statement

Design a payment processing system like Stripe that:
- Processes payments securely
- Handles multiple payment methods
- Detects fraud
- Ensures idempotency

---

## Requirements Gathering

### Functional Requirements

1. **Payment Processing**
   - Credit/debit cards
   - Bank transfers
   - Digital wallets
   - Refunds

2. **Security**
   - PCI-DSS compliance
   - Encryption
   - Fraud detection

3. **Features**
   - Recurring payments
   - Split payments
   - Multi-currency

### Non-Functional Requirements

1. **Scale**
   - 100M transactions/day
   - 1,200 transactions/second
   - Peak: 10,000 transactions/second

2. **Performance**
   - Payment processing: < 2 seconds
   - 99.99% availability

3. **Security**
   - PCI-DSS compliant
   - Fraud detection
   - Audit trail

---

## Capacity Estimation

### Traffic Estimates

**Transactions:**
```
100M transactions/day
= 100M / 86,400 seconds
= 1,157 transactions/second
Peak: 1,157 × 10 = 11,570 ≈ 12K transactions/second
```

**Storage:**
```
Per transaction: 1 KB
100M × 1 KB = 100 GB/day
Retention: 7 years = 255 TB
```

---

## High-Level Design

### Architecture

```
┌──────────┐
│ Merchant │
└────┬─────┘
     │
     ▼
┌──────────────┐
│ Payment      │
│ Gateway      │
└──────┬───────┘
       │
   ┌───┴───┐
   │       │
   ▼       ▼
┌─────┐ ┌─────┐
│Fraud│ │Payment│
│Detect│ │Processor│
└─────┘ └─────┘
       │
       ▼
┌──────────────┐
│   Bank       │
│   Network    │
└──────────────┘
```

### Components

1. **Payment Gateway:** Entry point
2. **Fraud Detection:** Analyze transactions
3. **Payment Processor:** Route to banks
4. **Database:** Store transactions

---

## Transaction Processing

### Payment Flow

```
┌─────────────────────────────────────┐
│ 1. Merchant sends payment request  │
│    - Amount, card details          │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 2. Validate request                 │
│    - Check card format              │
│    - Verify amount                  │
└──────────────┬──────────────────────┘
               │
               ▼
┌─────────────────────────────────────┐
│ 3. Fraud detection                  │
│    - Check risk score               │
│    - Verify patterns                │
└──────────────┬──────────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
┌──────────┐   ┌──────────┐
│ Low Risk │   │High Risk │
└────┬─────┘   └────┬─────┘
     │              │
     │              ▼
     │      ┌──────────┐
     │      │  Block   │
     │      └──────────┘
     │
     ▼
┌─────────────────────────────────────┐
│ 4. Process payment                  │
│    - Route to payment processor     │
│    - Communicate with bank          │
└──────────────┬──────────────────────┘
               │
        ┌──────┴──────┐
        │             │
        ▼             ▼
┌──────────┐   ┌──────────┐
│ Success  │   │ Failure  │
└────┬─────┘   └────┬─────┘
     │              │
     └──────┬───────┘
            │
            ▼
┌─────────────────────────────────────┐
│ 5. Update transaction status        │
│    - Store in database              │
│    - Notify merchant                │
└─────────────────────────────────────┘
```

---

## Fraud Detection

### Risk Factors

1. **Transaction Amount:** Unusually large
2. **Location:** Different from usual
3. **Velocity:** Too many transactions
4. **Device:** New device
5. **Pattern:** Unusual behavior

### Implementation

```java
public RiskScore calculateRisk(Transaction transaction) {
    double risk = 0;
    
    // Amount check
    if (transaction.getAmount() > user.getAverageAmount() * 3) {
        risk += 0.3;
    }
    
    // Location check
    if (!isKnownLocation(transaction.getLocation())) {
        risk += 0.2;
    }
    
    // Velocity check
    if (getTransactionCount(transaction.getUserId(), Duration.ofMinutes(5)) > 10) {
        risk += 0.3;
    }
    
    // Device check
    if (!isKnownDevice(transaction.getDeviceId())) {
        risk += 0.2;
    }
    
    return new RiskScore(risk);
}
```

---

## Idempotency

### Problem

**Duplicate requests:**
```
User clicks "Pay" twice
→ Two payment requests
→ Charge twice ❌
```

### Solution

**Idempotency Key:**
```
Request 1: idempotency_key = "abc123"
Request 2: idempotency_key = "abc123" (same)
→ Process only once ✅
```

### Implementation

```java
public PaymentResult processPayment(PaymentRequest request) {
    // Check idempotency
    String idempotencyKey = request.getIdempotencyKey();
    PaymentResult existing = paymentRepository.findByKey(idempotencyKey);
    if (existing != null) {
        return existing;  // Return cached result
    }
    
    // Process payment
    PaymentResult result = processPaymentInternal(request);
    
    // Store with idempotency key
    paymentRepository.saveWithKey(idempotencyKey, result);
    
    return result;
}
```

---

## Database Design

### Schema

```sql
CREATE TABLE transactions (
    id BIGINT PRIMARY KEY,
    idempotency_key VARCHAR(255) UNIQUE,
    merchant_id BIGINT,
    amount DECIMAL(10,2),
    currency VARCHAR(3),
    status VARCHAR(20),  -- pending, success, failed
    created_at TIMESTAMP,
    INDEX idx_merchant (merchant_id, created_at),
    INDEX idx_idempotency (idempotency_key)
);

CREATE TABLE payment_methods (
    id BIGINT PRIMARY KEY,
    user_id BIGINT,
    type VARCHAR(20),  -- card, bank, wallet
    last_four VARCHAR(4),
    encrypted_data TEXT,
    INDEX idx_user (user_id)
);
```

---

## Security

### PCI-DSS Compliance

1. **Encryption:** Encrypt card data
2. **Tokenization:** Don't store raw card numbers
3. **Access Control:** Limit who can access data
4. **Audit Logging:** Track all access

### Implementation

```java
// Don't store raw card number
// Store token instead
public String tokenizeCard(String cardNumber) {
    String token = generateToken();
    // Store mapping: token → encrypted card
    secureStorage.store(token, encrypt(cardNumber));
    return token;
}
```

---

## Scaling the System

### 1. Database Sharding

```
Shard by merchant_id:
hash(merchant_id) % number_of_shards = shard_number
```

### 2. Async Processing

```
Payment request → Queue → Process → Notify
(Fast response, process later)
```

### 3. Caching

```
Cache payment methods:
- User's saved cards
- Reduce database load
```

---

## Summary

**Key Design Decisions:**
1. ✅ **Idempotency:** Prevent duplicate charges
2. ✅ **Fraud Detection:** Risk scoring
3. ✅ **Security:** PCI-DSS compliance
4. ✅ **Sharding:** Scale transactions
5. ✅ **Async Processing:** Handle peaks

**Next Steps:**
- Learn [Distributed Systems](./21_DISTRIBUTED_SYSTEMS.md)
- Understand [Security](./23_SECURITY_IN_SYSTEM_DESIGN.md)
- Practice [More System Designs](./README.md)

---

**References:**
- [Stripe Architecture](https://stripe.com)
- [Payment Systems](https://algomaster.io/learn/system-design)
