# Concurrency in LLD

## Table of Contents
1. [Introduction](#introduction)
2. [Thread Safety](#thread-safety)
3. [Synchronization](#synchronization)
4. [Common Concurrency Patterns](#common-concurrency-patterns)
5. [Real-World Examples](#real-world-examples)

---

## Introduction

**Concurrency** is crucial for designing systems that handle multiple operations simultaneously. Understanding concurrency is essential for LLD.

---

## Thread Safety

### What is Thread Safety?

**Thread-safe code** can be safely executed by multiple threads simultaneously without causing data corruption.

### Thread-Safe Counter

```java
// Not thread-safe
public class Counter {
    private int count = 0;
    
    public void increment() {
        count++;  // Not atomic
    }
    
    public int getCount() {
        return count;
    }
}

// Thread-safe with synchronization
public class ThreadSafeCounter {
    private int count = 0;
    private final Object lock = new Object();
    
    public void increment() {
        synchronized (lock) {
            count++;
        }
    }
    
    public int getCount() {
        synchronized (lock) {
            return count;
        }
    }
}

// Thread-safe with AtomicInteger
public class AtomicCounter {
    private AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();
    }
    
    public int getCount() {
        return count.get();
    }
}
```

---

## Synchronization

### Synchronized Methods

```java
public class BankAccount {
    private double balance;
    
    public synchronized void deposit(double amount) {
        balance += amount;
    }
    
    public synchronized void withdraw(double amount) {
        if (balance >= amount) {
            balance -= amount;
        }
    }
    
    public synchronized double getBalance() {
        return balance;
    }
}
```

---

### Synchronized Blocks

```java
public class BankAccount {
    private double balance;
    private final Object lock = new Object();
    
    public void deposit(double amount) {
        synchronized (lock) {
            balance += amount;
        }
    }
    
    public void transfer(BankAccount to, double amount) {
        synchronized (this) {
            synchronized (to) {
                if (balance >= amount) {
                    this.withdraw(amount);
                    to.deposit(amount);
                }
            }
        }
    }
}
```

---

### ReentrantLock

```java
import java.util.concurrent.locks.ReentrantLock;

public class BankAccount {
    private double balance;
    private final ReentrantLock lock = new ReentrantLock();
    
    public void deposit(double amount) {
        lock.lock();
        try {
            balance += amount;
        } finally {
            lock.unlock();
        }
    }
    
    public void withdraw(double amount) {
        lock.lock();
        try {
            if (balance >= amount) {
                balance -= amount;
            }
        } finally {
            lock.unlock();
        }
    }
}
```

---

## Common Concurrency Patterns

### 1. Producer-Consumer Pattern

```java
import java.util.concurrent.BlockingQueue;

public class ProducerConsumer {
    private BlockingQueue<String> queue = new LinkedBlockingQueue<>(10);
    
    // Producer
    public class Producer implements Runnable {
        public void run() {
            try {
                while (true) {
                    String item = produceItem();
                    queue.put(item);  // Blocks if queue is full
                    System.out.println("Produced: " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    // Consumer
    public class Consumer implements Runnable {
        public void run() {
            try {
                while (true) {
                    String item = queue.take();  // Blocks if queue is empty
                    processItem(item);
                    System.out.println("Consumed: " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

---

### 2. Thread Pool Pattern

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public class TaskProcessor {
    private ExecutorService executor = Executors.newFixedThreadPool(10);
    
    public void processTask(Task task) {
        executor.submit(() -> {
            try {
                task.execute();
            } catch (Exception e) {
                logger.error("Task execution failed", e);
            }
        });
    }
    
    public void shutdown() {
        executor.shutdown();
    }
}
```

---

### 3. Read-Write Lock Pattern

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ThreadSafeCache {
    private Map<String, String> cache = new HashMap<>();
    private ReadWriteLock lock = new ReentrantReadWriteLock();
    
    public String get(String key) {
        lock.readLock().lock();
        try {
            return cache.get(key);
        } finally {
            lock.readLock().unlock();
        }
    }
    
    public void put(String key, String value) {
        lock.writeLock().lock();
        try {
            cache.put(key, value);
        } finally {
            lock.writeLock().unlock();
        }
    }
}
```

---

## Real-World Examples

### Example 1: Thread-Safe Singleton

```java
public class DatabaseConnection {
    private static volatile DatabaseConnection instance;
    private static final Object lock = new Object();
    
    private DatabaseConnection() {
        // Private constructor
    }
    
    public static DatabaseConnection getInstance() {
        if (instance == null) {
            synchronized (lock) {
                if (instance == null) {
                    instance = new DatabaseConnection();
                }
            }
        }
        return instance;
    }
}
```

---

### Example 2: Thread-Safe Order Processing

```java
public class OrderProcessor {
    private final Object lock = new Object();
    private Set<String> processingOrders = new HashSet<>();
    
    public void processOrder(Order order) {
        synchronized (lock) {
            if (processingOrders.contains(order.getId())) {
                throw new IllegalStateException("Order already processing");
            }
            processingOrders.add(order.getId());
        }
        
        try {
            // Process order
            validateOrder(order);
            processPayment(order);
            updateInventory(order);
        } finally {
            synchronized (lock) {
                processingOrders.remove(order.getId());
            }
        }
    }
}
```

---

### Example 3: Concurrent Shopping Cart

```java
import java.util.concurrent.ConcurrentHashMap;

public class ShoppingCart {
    private ConcurrentHashMap<Long, Integer> items = new ConcurrentHashMap<>();
    
    public void addItem(Long productId, int quantity) {
        items.compute(productId, (key, value) -> 
            value == null ? quantity : value + quantity
        );
    }
    
    public void removeItem(Long productId) {
        items.remove(productId);
    }
    
    public int getQuantity(Long productId) {
        return items.getOrDefault(productId, 0);
    }
}
```

---

## Summary

**Key Concepts:**
1. ✅ **Thread Safety:** Safe concurrent access
2. ✅ **Synchronization:** synchronized, locks
3. ✅ **Concurrent Collections:** Thread-safe data structures
4. ✅ **Thread Pools:** Manage threads efficiently
5. ✅ **Common Patterns:** Producer-Consumer, Read-Write Lock

**Next Steps:**
- Learn [Testing](./22_TESTING_IN_LLD.md)
- Review [Best Practices](./23_LLD_BEST_PRACTICES.md)
- Practice [Real-World Problems](./12_PARKING_LOT_SYSTEM.md)

---

**References:**
- [Error Handling](./20_ERROR_HANDLING.md)
- [Design Patterns](./08_CREATIONAL_PATTERNS.md)

