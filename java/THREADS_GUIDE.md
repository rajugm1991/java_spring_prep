# Java Threads - Complete Advanced Guide 🧵

> Comprehensive guide to Java threads and concurrency for senior engineers - Interview preparation with advanced topics, visual diagrams, and real-world examples

---

## Table of Contents
1. [Thread Fundamentals](#thread-fundamentals)
2. [Thread Lifecycle](#thread-lifecycle)
3. [Thread Creation](#thread-creation)
4. [Synchronization](#synchronization)
5. [Thread Communication](#thread-communication)
6. [Thread Pools and Executors](#thread-pools-and-executors)
7. [Concurrent Collections](#concurrent-collections)
8. [Advanced Topics](#advanced-topics)
9. [CompletableFuture](#completablefuture)
10. [Fork/Join Framework](#forkjoin-framework)
11. [Best Practices](#best-practices)
12. [Performance Optimization](#performance-optimization)
13. [Common Patterns](#common-patterns)
14. [Interview Questions](#interview-questions)

---

## Thread Fundamentals

### What is a Thread?

**Thread** is the smallest unit of execution within a process. A process can have multiple threads running concurrently.

```
Process (JVM)
├── Main Thread
├── Thread-1
├── Thread-2
└── Thread-3
```

### Process vs Thread

| Aspect | Process | Thread |
|--------|---------|--------|
| **Memory** | Separate memory space | Shared memory space |
| **Creation** | Heavyweight (slow) | Lightweight (fast) |
| **Communication** | IPC (Inter-Process Communication) | Shared memory |
| **Isolation** | High isolation | Low isolation |
| **Context Switch** | Expensive | Cheaper |
| **Overhead** | High | Low |

### Why Use Threads?

1. **Performance**: Utilize multiple CPU cores
2. **Responsiveness**: Keep UI responsive while processing
3. **Throughput**: Handle multiple requests simultaneously
4. **Resource Utilization**: Better CPU and I/O utilization

### Thread States

```java
public enum Thread.State {
    NEW,              // Thread created but not started
    RUNNABLE,         // Thread executing in JVM
    BLOCKED,          // Waiting for monitor lock
    WAITING,          // Waiting indefinitely
    TIMED_WAITING,    // Waiting with timeout
    TERMINATED        // Thread completed execution
}
```

---

## Thread Lifecycle

### Visual Lifecycle Diagram

```
    NEW
     │
     │ start()
     ▼
RUNNABLE ◄──────────┐
     │              │
     │ wait()       │ notify()
     │ join()       │ interrupt()
     │ sleep()      │
     ▼              │
BLOCKED/WAITING     │
     │              │
     │              │
     └──────────────┘
     │
     │ run() completes
     ▼
TERMINATED
```

### State Transitions

**1. NEW → RUNNABLE**
```java
Thread thread = new Thread(() -> System.out.println("Running"));
// State: NEW

thread.start();
// State: RUNNABLE
```

**2. RUNNABLE → BLOCKED**
```java
synchronized (lock) {
    // Thread enters BLOCKED state if another thread holds the lock
}
```

**3. RUNNABLE → WAITING**
```java
synchronized (lock) {
    lock.wait(); // Thread enters WAITING state
}
```

**4. RUNNABLE → TIMED_WAITING**
```java
Thread.sleep(1000); // Thread enters TIMED_WAITING state
```

**5. RUNNABLE → TERMINATED**
```java
// Thread completes run() method
// State: TERMINATED
```

### Complete Lifecycle Example

```java
public class ThreadLifecycleDemo {
    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            try {
                System.out.println("Thread state: " + Thread.currentThread().getState()); // RUNNABLE
                Thread.sleep(2000); // TIMED_WAITING
                synchronized (ThreadLifecycleDemo.class) {
                    ThreadLifecycleDemo.class.wait(); // WAITING
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        });
        
        System.out.println("Thread state: " + thread.getState()); // NEW
        thread.start();
        System.out.println("Thread state: " + thread.getState()); // RUNNABLE
        
        Thread.sleep(100);
        System.out.println("Thread state: " + thread.getState()); // TIMED_WAITING
        
        Thread.sleep(2000);
        synchronized (ThreadLifecycleDemo.class) {
            ThreadLifecycleDemo.class.notify();
        }
        
        thread.join();
        System.out.println("Thread state: " + thread.getState()); // TERMINATED
    }
}
```

---

## Thread Creation

### Method 1: Extending Thread Class

```java
public class MyThread extends Thread {
    @Override
    public void run() {
        System.out.println("Thread running: " + Thread.currentThread().getName());
    }
}

// Usage
MyThread thread = new MyThread();
thread.start();
```

**Pros:**
- Simple to understand
- Direct access to Thread methods

**Cons:**
- Cannot extend another class (Java single inheritance)
- Tight coupling

### Method 2: Implementing Runnable Interface

```java
public class MyRunnable implements Runnable {
    @Override
    public void run() {
        System.out.println("Thread running: " + Thread.currentThread().getName());
    }
}

// Usage
Thread thread = new Thread(new MyRunnable());
thread.start();

// Or with lambda
Thread thread = new Thread(() -> {
    System.out.println("Thread running: " + Thread.currentThread().getName());
});
thread.start();
```

**Pros:**
- Can extend another class
- Better separation of concerns
- More flexible

**Cons:**
- No direct access to Thread methods

### Method 3: Implementing Callable Interface

```java
import java.util.concurrent.Callable;
import java.util.concurrent.FutureTask;

public class MyCallable implements Callable<String> {
    @Override
    public String call() throws Exception {
        return "Result from thread: " + Thread.currentThread().getName();
    }
}

// Usage
Callable<String> callable = new MyCallable();
FutureTask<String> futureTask = new FutureTask<>(callable);
Thread thread = new Thread(futureTask);
thread.start();

String result = futureTask.get(); // Blocks until result is available
```

**Pros:**
- Can return a value
- Can throw checked exceptions
- Better for thread pools

### Method 4: Using ExecutorService (Recommended)

```java
ExecutorService executor = Executors.newFixedThreadPool(10);

// Submit Runnable
executor.submit(() -> {
    System.out.println("Task executed");
});

// Submit Callable
Future<String> future = executor.submit(() -> {
    return "Result";
});

String result = future.get();
```

---

## Synchronization

### The Problem: Race Conditions

```java
public class Counter {
    private int count = 0;
    
    public void increment() {
        count++; // NOT thread-safe!
    }
    
    public int getCount() {
        return count;
    }
}
```

**Problem:** Multiple threads can read and write `count` simultaneously, causing lost updates.

### Solution 1: synchronized Keyword

#### Synchronized Method

```java
public class Counter {
    private int count = 0;
    
    public synchronized void increment() {
        count++;
    }
    
    public synchronized int getCount() {
        return count;
    }
}
```

#### Synchronized Block

```java
public class Counter {
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
```

#### Synchronized Static Method

```java
public class Counter {
    private static int count = 0;
    
    public static synchronized void increment() {
        count++;
    }
    
    // Equivalent to:
    public static void increment() {
        synchronized (Counter.class) {
            count++;
        }
    }
}
```

### Solution 2: ReentrantLock

```java
import java.util.concurrent.locks.ReentrantLock;

public class Counter {
    private int count = 0;
    private final ReentrantLock lock = new ReentrantLock();
    
    public void increment() {
        lock.lock();
        try {
            count++;
        } finally {
            lock.unlock(); // Always unlock in finally block
        }
    }
    
    public int getCount() {
        lock.lock();
        try {
            return count;
        } finally {
            lock.unlock();
        }
    }
}
```

**ReentrantLock vs synchronized:**

| Feature | synchronized | ReentrantLock |
|---------|-------------|---------------|
| **Automatic unlock** | Yes | No (must unlock) |
| **Try lock** | No | Yes (tryLock()) |
| **Fairness** | No | Configurable |
| **Interruptible** | No | Yes (lockInterruptibly()) |
| **Multiple conditions** | No | Yes |

### Solution 3: Atomic Classes

```java
import java.util.concurrent.atomic.AtomicInteger;

public class Counter {
    private final AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();
    }
    
    public int getCount() {
        return count.get();
    }
    
    // Advanced operations
    public int addAndGet(int delta) {
        return count.addAndGet(delta);
    }
    
    public boolean compareAndSet(int expect, int update) {
        return count.compareAndSet(expect, update);
    }
}
```

**Available Atomic Classes:**
- `AtomicInteger`
- `AtomicLong`
- `AtomicBoolean`
- `AtomicReference<T>`
- `AtomicIntegerArray`
- `AtomicLongArray`
- `AtomicReferenceArray<T>`

### Solution 4: Volatile Keyword

```java
public class Flag {
    private volatile boolean flag = false;
    
    public void setFlag(boolean value) {
        flag = value; // Write is visible to all threads
    }
    
    public boolean getFlag() {
        return flag; // Read sees latest value
    }
}
```

**What volatile does:**
- Ensures visibility (changes visible to all threads)
- Prevents compiler reordering
- Does NOT provide atomicity

**When to use volatile:**
- Single writer, multiple readers
- Simple flags/status variables
- Variables that don't participate in compound operations

---

## Thread Communication

### wait() and notify()

```java
public class ProducerConsumer {
    private final Object lock = new Object();
    private String message = null;
    private boolean hasMessage = false;
    
    public void produce(String msg) throws InterruptedException {
        synchronized (lock) {
            while (hasMessage) {
                lock.wait(); // Wait until message is consumed
            }
            message = msg;
            hasMessage = true;
            lock.notify(); // Notify consumer
        }
    }
    
    public String consume() throws InterruptedException {
        synchronized (lock) {
            while (!hasMessage) {
                lock.wait(); // Wait until message is available
            }
            String msg = message;
            hasMessage = false;
            lock.notify(); // Notify producer
            return msg;
        }
    }
}
```

**Important Rules:**
1. Must be called inside synchronized block
2. Always use while loop (not if) to check condition
3. Use notifyAll() when multiple threads are waiting

### wait() vs sleep()

| Feature | wait() | sleep() |
|---------|--------|---------|
| **Releases lock** | Yes | No |
| **Called on** | Object | Thread |
| **Can be interrupted** | Yes | Yes |
| **Usage** | Thread communication | Delay execution |

### join()

```java
Thread thread1 = new Thread(() -> {
    System.out.println("Thread 1 running");
});

Thread thread2 = new Thread(() -> {
    try {
        thread1.join(); // Wait for thread1 to complete
        System.out.println("Thread 2 running after thread1");
    } catch (InterruptedException e) {
        Thread.currentThread().interrupt();
    }
});

thread1.start();
thread2.start();
```

### Interrupt Mechanism

```java
Thread thread = new Thread(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        try {
            Thread.sleep(1000);
            System.out.println("Working...");
        } catch (InterruptedException e) {
            // Interrupt flag is cleared when exception is thrown
            Thread.currentThread().interrupt(); // Restore interrupt flag
            break;
        }
    }
});

thread.start();
Thread.sleep(5000);
thread.interrupt(); // Request interruption
```

**Best Practices:**
- Check `isInterrupted()` in loops
- Restore interrupt flag in catch blocks
- Don't swallow InterruptedException

---

## Thread Pools and Executors

### Why Thread Pools?

**Problems with creating threads directly:**
- Thread creation is expensive
- Unbounded thread creation can exhaust resources
- No control over thread lifecycle

**Benefits of thread pools:**
- Reuse threads (reduce overhead)
- Control number of threads
- Manage thread lifecycle
- Better resource management

### ExecutorService Types

#### 1. FixedThreadPool

```java
ExecutorService executor = Executors.newFixedThreadPool(10);

for (int i = 0; i < 100; i++) {
    executor.submit(() -> {
        System.out.println("Task executed by: " + Thread.currentThread().getName());
    });
}

executor.shutdown();
```

**Characteristics:**
- Fixed number of threads
- Unbounded queue
- Threads live until pool is shut down

#### 2. CachedThreadPool

```java
ExecutorService executor = Executors.newCachedThreadPool();
```

**Characteristics:**
- Creates threads on demand
- Reuses idle threads (60 seconds)
- No queue (direct handoff)
- Good for short-lived tasks

#### 3. SingleThreadExecutor

```java
ExecutorService executor = Executors.newSingleThreadExecutor();
```

**Characteristics:**
- Single worker thread
- Tasks executed sequentially
- Unbounded queue
- Guaranteed execution order

#### 4. ScheduledThreadPool

```java
ScheduledExecutorService executor = Executors.newScheduledThreadPool(5);

// Execute after delay
executor.schedule(() -> System.out.println("Delayed task"), 5, TimeUnit.SECONDS);

// Execute periodically with fixed delay
executor.scheduleWithFixedDelay(() -> System.out.println("Periodic task"), 
    0, 1, TimeUnit.SECONDS);

// Execute periodically with fixed rate
executor.scheduleAtFixedRate(() -> System.out.println("Fixed rate task"), 
    0, 1, TimeUnit.SECONDS);
```

### ThreadPoolExecutor (Custom Configuration)

```java
ThreadPoolExecutor executor = new ThreadPoolExecutor(
    5,                          // Core pool size
    10,                         // Maximum pool size
    60L,                        // Keep-alive time
    TimeUnit.SECONDS,           // Time unit
    new LinkedBlockingQueue<>(100), // Work queue
    new ThreadFactory() {
        private int counter = 0;
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r, "worker-" + counter++);
            t.setDaemon(false);
            return t;
        }
    },
    new ThreadPoolExecutor.CallerRunsPolicy() // Rejection policy
);
```

**Parameters Explained:**

1. **Core Pool Size**: Minimum number of threads
2. **Maximum Pool Size**: Maximum number of threads
3. **Keep-Alive Time**: Time idle threads wait before termination
4. **Work Queue**: Queue for holding tasks
5. **Thread Factory**: Creates new threads
6. **Rejection Policy**: Handler for rejected tasks

**Rejection Policies:**

```java
// AbortPolicy (default): Throws RejectedExecutionException
new ThreadPoolExecutor.AbortPolicy()

// CallerRunsPolicy: Executes task in caller thread
new ThreadPoolExecutor.CallerRunsPolicy()

// DiscardPolicy: Silently discards task
new ThreadPoolExecutor.DiscardPolicy()

// DiscardOldestPolicy: Discards oldest task
new ThreadPoolExecutor.DiscardOldestPolicy()
```

### Real-World Example from Codebase

**File:** `product-service/src/main/java/com/ekart/product_service/config/BulkheadConfig.java`

```java
@Configuration
public class BulkheadConfig {

    @Bean("criticalOperationsExecutor")
    public ThreadPoolExecutor criticalOperationsExecutor() {
        return new ThreadPoolExecutor(
            10,  // Core pool size
            10,  // Maximum pool size
            0L, TimeUnit.MILLISECONDS,
            new LinkedBlockingQueue<>(100), // Bounded queue
            new ThreadFactory() {
                private int counter = 0;
                @Override
                public Thread newThread(Runnable r) {
                    Thread t = new Thread(r, "critical-" + counter++);
                    t.setDaemon(false);
                    return t;
                }
            },
            new ThreadPoolExecutor.CallerRunsPolicy() // Reject policy
        );
    }
}
```

**Usage:**

```java
@Service
public class BulkheadProductService {
    
    @Autowired
    @Qualifier("criticalOperationsExecutor")
    private ExecutorService criticalExecutor;
    
    public CompletableFuture<List<ProductDTO>> getCriticalProducts() {
        return CompletableFuture.supplyAsync(() -> {
            // Critical operation
            return fetchProducts();
        }, criticalExecutor);
    }
}
```

---

## Concurrent Collections

### Why Concurrent Collections?

**Problems with synchronized collections:**
- Poor performance (coarse-grained locking)
- Compound operations not atomic
- Iteration requires external synchronization

### ConcurrentHashMap

```java
import java.util.concurrent.ConcurrentHashMap;

ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

// Thread-safe operations
map.put("key1", 1);
map.putIfAbsent("key2", 2);
map.compute("key1", (k, v) -> v == null ? 0 : v + 1);
map.merge("key3", 1, Integer::sum);

// Atomic operations
map.computeIfAbsent("key4", k -> 10);
map.computeIfPresent("key1", (k, v) -> v * 2);
```

**Features:**
- Thread-safe without external synchronization
- Better performance than synchronized HashMap
- Supports atomic operations
- Segmented locking (lock striping)

### BlockingQueue

```java
import java.util.concurrent.BlockingQueue;
import java.util.concurrent.LinkedBlockingQueue;

BlockingQueue<String> queue = new LinkedBlockingQueue<>(10);

// Producer
queue.put("item1"); // Blocks if queue is full
queue.offer("item2", 5, TimeUnit.SECONDS); // Timeout

// Consumer
String item = queue.take(); // Blocks if queue is empty
String item = queue.poll(5, TimeUnit.SECONDS); // Timeout
```

**Types:**
- `LinkedBlockingQueue`: Unbounded or bounded
- `ArrayBlockingQueue`: Bounded, array-based
- `PriorityBlockingQueue`: Priority-based
- `SynchronousQueue`: Direct handoff

### CopyOnWriteArrayList

```java
import java.util.concurrent.CopyOnWriteArrayList;

CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();

list.add("item1");
list.add("item2");

// Iteration doesn't require synchronization
for (String item : list) {
    System.out.println(item);
}
```

**Use Cases:**
- Read-heavy, write-rarely scenarios
- Event listeners
- Snapshot iterators

**Trade-offs:**
- ✅ No locking for reads
- ❌ Expensive writes (copy entire array)
- ❌ Memory overhead

### ConcurrentLinkedQueue

```java
import java.util.concurrent.ConcurrentLinkedQueue;

ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();

queue.offer("item1");
String item = queue.poll();
```

**Features:**
- Lock-free implementation
- Non-blocking operations
- Good for high-concurrency scenarios

---

## Advanced Topics

### Java Memory Model (JMM)

#### Happens-Before Relationship

**Rules:**
1. **Program Order**: Actions in same thread happen in program order
2. **Monitor Lock**: Unlock happens-before subsequent lock
3. **Volatile**: Write happens-before subsequent read
4. **Thread Start**: `start()` happens-before actions in thread
5. **Thread Join**: Actions in thread happen-before `join()` returns
6. **Transitivity**: If A happens-before B and B happens-before C, then A happens-before C

#### Visibility Problem

```java
public class VisibilityProblem {
    private boolean flag = false; // NOT volatile
    
    public void setFlag() {
        flag = true; // May not be visible to other threads
    }
    
    public boolean getFlag() {
        return flag; // May see stale value
    }
}
```

**Solution:**

```java
private volatile boolean flag = false; // Ensures visibility
```

### Volatile Deep Dive

**What volatile guarantees:**
1. **Visibility**: Changes visible to all threads
2. **Ordering**: Prevents reordering
3. **NOT Atomicity**: Compound operations still need synchronization

**Example:**

```java
public class VolatileExample {
    private volatile int count = 0;
    
    public void increment() {
        count++; // NOT atomic! (read-modify-write)
    }
    
    // Correct way:
    private final AtomicInteger count = new AtomicInteger(0);
    
    public void incrementCorrect() {
        count.incrementAndGet(); // Atomic
    }
}
```

### Double-Checked Locking

**Problem Pattern:**

```java
public class Singleton {
    private static Singleton instance;
    
    public static Singleton getInstance() {
        if (instance == null) { // First check
            synchronized (Singleton.class) {
                if (instance == null) { // Second check
                    instance = new Singleton(); // Problem: May see partially constructed object
                }
            }
        }
        return instance;
    }
}
```

**Solution 1: volatile**

```java
public class Singleton {
    private static volatile Singleton instance;
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

**Solution 2: Initialization-on-demand holder**

```java
public class Singleton {
    private Singleton() {}
    
    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }
    
    public static Singleton getInstance() {
        return Holder.INSTANCE; // Thread-safe lazy initialization
    }
}
```

### ThreadLocal

```java
public class ThreadLocalExample {
    private static final ThreadLocal<Integer> threadLocal = new ThreadLocal<>();
    
    public static void main(String[] args) {
        Thread thread1 = new Thread(() -> {
            threadLocal.set(1);
            System.out.println("Thread 1: " + threadLocal.get()); // 1
        });
        
        Thread thread2 = new Thread(() -> {
            threadLocal.set(2);
            System.out.println("Thread 2: " + threadLocal.get()); // 2
        });
        
        thread1.start();
        thread2.start();
    }
}
```

**Use Cases:**
- User context in web applications
- Transaction context
- Request-scoped data

**Memory Leak Warning:**

```java
// Always remove ThreadLocal values
threadLocal.remove(); // Prevents memory leaks
```

### ReadWriteLock

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteLockExample {
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private int value = 0;
    
    public int read() {
        lock.readLock().lock();
        try {
            return value; // Multiple readers can read simultaneously
        } finally {
            lock.readLock().unlock();
        }
    }
    
    public void write(int newValue) {
        lock.writeLock().lock();
        try {
            value = newValue; // Only one writer at a time
        } finally {
            lock.writeLock().unlock();
        }
    }
}
```

**Benefits:**
- Multiple concurrent readers
- Exclusive writer
- Better performance for read-heavy workloads

### StampedLock

```java
import java.util.concurrent.locks.StampedLock;

public class StampedLockExample {
    private final StampedLock lock = new StampedLock();
    private int value = 0;
    
    public int read() {
        long stamp = lock.tryOptimisticRead();
        int currentValue = value;
        
        if (!lock.validate(stamp)) {
            // Fallback to read lock
            stamp = lock.readLock();
            try {
                currentValue = value;
            } finally {
                lock.unlockRead(stamp);
            }
        }
        return currentValue;
    }
    
    public void write(int newValue) {
        long stamp = lock.writeLock();
        try {
            value = newValue;
        } finally {
            lock.unlockWrite(stamp);
        }
    }
}
```

**Features:**
- Optimistic reads (no locking)
- Better performance than ReadWriteLock
- More complex API

---

## CompletableFuture

### Introduction

`CompletableFuture` provides a way to write asynchronous, non-blocking code.

### Basic Usage

```java
// Create CompletableFuture
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    return "Hello";
});

// Get result (blocks)
String result = future.get();

// Non-blocking
future.thenAccept(result -> System.out.println(result));
```

### Chaining Operations

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> "Hello")
    .thenApply(s -> s + " World")
    .thenApply(String::toUpperCase)
    .thenCompose(s -> CompletableFuture.supplyAsync(() -> s + "!"));

String result = future.get(); // "HELLO WORLD!"
```

### Combining Futures

```java
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> "World");

// Combine results
CompletableFuture<String> combined = future1.thenCombine(future2, 
    (s1, s2) -> s1 + " " + s2);

// Wait for all
CompletableFuture<Void> allOf = CompletableFuture.allOf(future1, future2);

// Wait for any
CompletableFuture<Object> anyOf = CompletableFuture.anyOf(future1, future2);
```

### Error Handling

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> {
        if (Math.random() > 0.5) {
            throw new RuntimeException("Error");
        }
        return "Success";
    })
    .exceptionally(ex -> "Error: " + ex.getMessage())
    .handle((result, ex) -> {
        if (ex != null) {
            return "Handled: " + ex.getMessage();
        }
        return result;
    });
```

### Real-World Example

```java
@Service
public class ProductService {
    
    @Autowired
    private ExecutorService executor;
    
    public CompletableFuture<ProductDTO> getProduct(Long id) {
        return CompletableFuture.supplyAsync(() -> {
            return fetchProductFromDatabase(id);
        }, executor);
    }
    
    public CompletableFuture<List<ProductDTO>> getProductsWithDetails(List<Long> ids) {
        List<CompletableFuture<ProductDTO>> futures = ids.stream()
            .map(this::getProduct)
            .collect(Collectors.toList());
        
        return CompletableFuture.allOf(futures.toArray(new CompletableFuture[0]))
            .thenApply(v -> futures.stream()
                .map(CompletableFuture::join)
                .collect(Collectors.toList()));
    }
}
```

---

## Fork/Join Framework

### Introduction

Fork/Join framework is designed for parallel execution of tasks that can be recursively split.

### ForkJoinPool

```java
import java.util.concurrent.ForkJoinPool;
import java.util.concurrent.RecursiveTask;

public class SumTask extends RecursiveTask<Long> {
    private static final int THRESHOLD = 1000;
    private int[] array;
    private int start;
    private int end;
    
    public SumTask(int[] array, int start, int end) {
        this.array = array;
        this.start = start;
        this.end = end;
    }
    
    @Override
    protected Long compute() {
        if (end - start <= THRESHOLD) {
            // Base case: compute directly
            long sum = 0;
            for (int i = start; i < end; i++) {
                sum += array[i];
            }
            return sum;
        } else {
            // Split task
            int mid = (start + end) / 2;
            SumTask left = new SumTask(array, start, mid);
            SumTask right = new SumTask(array, mid, end);
            
            left.fork(); // Execute asynchronously
            long rightResult = right.compute(); // Compute directly
            long leftResult = left.join(); // Wait for result
            
            return leftResult + rightResult;
        }
    }
}

// Usage
ForkJoinPool pool = new ForkJoinPool();
int[] array = new int[10000];
SumTask task = new SumTask(array, 0, array.length);
Long result = pool.invoke(task);
```

### Work Stealing

Fork/Join uses work-stealing algorithm:
- Each thread has its own deque
- When thread's deque is empty, steals work from other threads
- Reduces contention and improves load balancing

---

## Best Practices

### 1. Always Use Thread Pools

**❌ BAD:**
```java
for (int i = 0; i < 1000; i++) {
    new Thread(() -> doWork()).start(); // Creates 1000 threads!
}
```

**✅ GOOD:**
```java
ExecutorService executor = Executors.newFixedThreadPool(10);
for (int i = 0; i < 1000; i++) {
    executor.submit(() -> doWork());
}
```

### 2. Proper Exception Handling

**❌ BAD:**
```java
executor.submit(() -> {
    throw new RuntimeException("Error"); // Exception is swallowed!
});
```

**✅ GOOD:**
```java
CompletableFuture.supplyAsync(() -> {
    throw new RuntimeException("Error");
}).exceptionally(ex -> {
    logger.error("Error occurred", ex);
    return null;
});
```

### 3. Always Shutdown ExecutorService

**❌ BAD:**
```java
ExecutorService executor = Executors.newFixedThreadPool(10);
// Forgot to shutdown - threads never terminate!
```

**✅ GOOD:**
```java
ExecutorService executor = Executors.newFixedThreadPool(10);
try {
    // Use executor
} finally {
    executor.shutdown();
    try {
        if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
            executor.shutdownNow();
        }
    } catch (InterruptedException e) {
        executor.shutdownNow();
        Thread.currentThread().interrupt();
    }
}
```

### 4. Use Concurrent Collections

**❌ BAD:**
```java
Map<String, Integer> map = Collections.synchronizedMap(new HashMap<>());
// Still need external synchronization for compound operations
```

**✅ GOOD:**
```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
// Thread-safe without external synchronization
```

### 5. Avoid Deadlocks

**❌ BAD:**
```java
// Thread 1
synchronized (lock1) {
    synchronized (lock2) {
        // ...
    }
}

// Thread 2
synchronized (lock2) {
    synchronized (lock1) { // Deadlock!
        // ...
    }
}
```

**✅ GOOD:**
```java
// Always acquire locks in same order
synchronized (lock1) {
    synchronized (lock2) {
        // ...
    }
}
```

### 6. Prefer Immutability

```java
// Immutable class is thread-safe
public final class Point {
    private final int x;
    private final int y;
    
    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
    
    // No setters - thread-safe!
}
```

### 7. Use Atomic Classes for Simple Operations

**❌ BAD:**
```java
private int count = 0;

public synchronized void increment() {
    count++; // Overhead of synchronization
}
```

**✅ GOOD:**
```java
private final AtomicInteger count = new AtomicInteger(0);

public void increment() {
    count.incrementAndGet(); // Lock-free, faster
}
```

---

## Performance Optimization

### 1. Reduce Lock Contention

**❌ BAD:**
```java
public synchronized void process() {
    // Long operation holding lock
    Thread.sleep(1000);
}
```

**✅ GOOD:**
```java
public void process() {
    // Do work outside lock
    String data = prepareData();
    
    synchronized (this) {
        // Only lock for critical section
        updateSharedState(data);
    }
}
```

### 2. Use Lock Striping

```java
public class StripedLock {
    private final Lock[] locks;
    private final int mask;
    
    public StripedLock(int stripes) {
        locks = new Lock[stripes];
        for (int i = 0; i < stripes; i++) {
            locks[i] = new ReentrantLock();
        }
        mask = stripes - 1;
    }
    
    public void lock(int key) {
        locks[key & mask].lock();
    }
    
    public void unlock(int key) {
        locks[key & mask].unlock();
    }
}
```

### 3. Avoid False Sharing

```java
// ❌ BAD: Variables on same cache line
public class Counter {
    volatile long count1;
    volatile long count2; // Same cache line - false sharing!
}

// ✅ GOOD: Padding to avoid false sharing
public class Counter {
    volatile long count1;
    long p1, p2, p3, p4, p5, p6, p7; // Padding
    volatile long count2; // Different cache line
}
```

### 4. Use Appropriate Data Structures

- **Read-heavy**: `CopyOnWriteArrayList`
- **Write-heavy**: `ConcurrentLinkedQueue`
- **Mixed**: `ConcurrentHashMap`

---

## Common Patterns

### Producer-Consumer Pattern

```java
public class ProducerConsumer {
    private final BlockingQueue<String> queue = new LinkedBlockingQueue<>(10);
    
    class Producer implements Runnable {
        @Override
        public void run() {
            try {
                for (int i = 0; i < 100; i++) {
                    queue.put("Item " + i);
                    System.out.println("Produced: Item " + i);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
    
    class Consumer implements Runnable {
        @Override
        public void run() {
            try {
                while (true) {
                    String item = queue.take();
                    System.out.println("Consumed: " + item);
                }
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }
        }
    }
}
```

### Worker Thread Pattern

```java
ExecutorService workers = Executors.newFixedThreadPool(10);
BlockingQueue<Task> taskQueue = new LinkedBlockingQueue<>();

// Workers
for (int i = 0; i < 10; i++) {
    workers.submit(() -> {
        while (!Thread.currentThread().isInterrupted()) {
            try {
                Task task = taskQueue.take();
                task.execute();
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
                break;
            }
        }
    });
}
```

### Barrier Pattern

```java
import java.util.concurrent.CyclicBarrier;

CyclicBarrier barrier = new CyclicBarrier(3, () -> {
    System.out.println("All threads reached barrier");
});

for (int i = 0; i < 3; i++) {
    new Thread(() -> {
        try {
            System.out.println("Thread waiting at barrier");
            barrier.await(); // Wait for all threads
            System.out.println("Thread passed barrier");
        } catch (Exception e) {
            e.printStackTrace();
        }
    }).start();
}
```

---

## Interview Questions

### Basic Questions

#### Q1: What is the difference between process and thread?

**Answer:**
"A process is an independent program with its own memory space, while a thread is a lightweight unit of execution within a process that shares memory with other threads. Processes are isolated and communicate via IPC, while threads share memory and can communicate directly. Thread creation is cheaper than process creation, but threads have less isolation."

#### Q2: Explain thread lifecycle states.

**Answer:**
"Threads have six states:
1. **NEW**: Created but not started
2. **RUNNABLE**: Executing in JVM (includes running and ready)
3. **BLOCKED**: Waiting for monitor lock
4. **WAITING**: Waiting indefinitely (wait(), join())
5. **TIMED_WAITING**: Waiting with timeout (sleep(), wait(timeout))
6. **TERMINATED**: Completed execution

Transitions occur via start(), synchronization, wait/notify, sleep, and completion."

#### Q3: What is the difference between wait() and sleep()?

**Answer:**
"wait() is called on an object and releases the lock, allowing other threads to acquire it. sleep() is called on Thread and doesn't release any locks. wait() must be called inside a synchronized block, while sleep() can be called anywhere. wait() is used for thread communication, while sleep() is used for delays."

### Intermediate Questions

#### Q4: Explain synchronized keyword.

**Answer:**
"synchronized provides mutual exclusion and ensures only one thread can execute a synchronized block or method at a time. It uses intrinsic locks (monitor locks) and provides:
1. **Mutual exclusion**: Only one thread can hold the lock
2. **Visibility**: Changes made by one thread are visible to others after unlock
3. **Reentrancy**: A thread can acquire the same lock multiple times

Synchronized methods lock on 'this', static methods lock on the class object, and synchronized blocks lock on the specified object."

#### Q5: What is the difference between synchronized and ReentrantLock?

**Answer:**
"Key differences:
- **Automatic unlock**: synchronized unlocks automatically, ReentrantLock requires explicit unlock
- **Try lock**: ReentrantLock supports tryLock(), synchronized doesn't
- **Fairness**: ReentrantLock can be fair, synchronized is not
- **Interruptible**: ReentrantLock supports lockInterruptibly(), synchronized doesn't
- **Multiple conditions**: ReentrantLock supports multiple Condition objects

Generally, use synchronized for simplicity, ReentrantLock when you need advanced features."

#### Q6: Explain volatile keyword.

**Answer:**
"volatile ensures:
1. **Visibility**: Changes to volatile variables are immediately visible to all threads
2. **Ordering**: Prevents compiler reordering of operations
3. **NOT atomicity**: Compound operations still need synchronization

Use volatile for:
- Single writer, multiple readers
- Simple flags/status variables
- Variables that don't participate in compound operations

Don't use volatile for operations like count++ which require atomicity."

### Advanced Questions

#### Q7: Explain Java Memory Model and happens-before relationship.

**Answer:**
"The Java Memory Model defines how threads interact through memory. The happens-before relationship ensures memory visibility:

1. **Program order**: Actions in same thread happen in program order
2. **Monitor lock**: Unlock happens-before subsequent lock
3. **Volatile**: Write happens-before subsequent read
4. **Thread start**: start() happens-before actions in thread
5. **Thread join**: Actions in thread happen-before join() returns
6. **Transitivity**: If A happens-before B and B happens-before C, then A happens-before C

This ensures that writes by one thread are visible to reads by another thread when there's a happens-before relationship."

#### Q8: What is a deadlock and how do you prevent it?

**Answer:**
"A deadlock occurs when two or more threads are blocked forever, waiting for each other. Conditions for deadlock:
1. Mutual exclusion
2. Hold and wait
3. No preemption
4. Circular wait

Prevention strategies:
1. **Lock ordering**: Always acquire locks in same order
2. **Lock timeout**: Use tryLock() with timeout
3. **Avoid nested locks**: Minimize lock nesting
4. **Lock-free algorithms**: Use atomic classes or lock-free data structures

Detection: Use thread dumps to identify circular dependencies."

#### Q9: Explain ThreadPoolExecutor parameters and how it works.

**Answer:**
"ThreadPoolExecutor has these key parameters:
1. **Core pool size**: Minimum threads kept alive
2. **Maximum pool size**: Maximum threads that can be created
3. **Keep-alive time**: Time idle threads wait before termination
4. **Work queue**: Queue for holding tasks
5. **Thread factory**: Creates new threads
6. **Rejection policy**: Handler for rejected tasks

How it works:
- If threads < core pool size, create new thread
- If threads >= core pool size, add to queue
- If queue is full and threads < max pool size, create new thread
- If queue is full and threads >= max pool size, reject task

This provides efficient thread reuse and resource management."

#### Q10: What is the difference between ConcurrentHashMap and synchronized HashMap?

**Answer:**
"ConcurrentHashMap uses lock striping (multiple locks for different segments), allowing concurrent reads and writes to different segments. synchronized HashMap uses a single lock, blocking all operations.

Benefits of ConcurrentHashMap:
- Better concurrency (multiple threads can read/write simultaneously)
- No external synchronization needed
- Atomic operations (compute, merge, etc.)
- Better performance for high concurrency

synchronized HashMap:
- Simple but poor performance
- Blocks all operations
- Requires external synchronization for compound operations"

#### Q11: Explain CompletableFuture and its advantages.

**Answer:**
"CompletableFuture is a powerful API for asynchronous programming that:
1. **Non-blocking**: Doesn't block calling thread
2. **Composable**: Can chain operations (thenApply, thenCompose)
3. **Combinable**: Can combine multiple futures (allOf, anyOf)
4. **Error handling**: Built-in exception handling (exceptionally, handle)
5. **Callback-based**: Supports callbacks (thenAccept, thenRun)

Advantages over Future:
- Can chain operations without blocking
- Better error handling
- Can combine multiple futures
- More flexible and powerful API

Example:
```java
CompletableFuture.supplyAsync(() -> fetchData())
    .thenApply(data -> process(data))
    .thenCompose(result -> saveAsync(result))
    .exceptionally(ex -> handleError(ex));
```"

#### Q12: What is false sharing and how do you prevent it?

**Answer:**
"False sharing occurs when multiple threads modify variables that happen to be on the same CPU cache line. Even though they're different variables, the cache line invalidation causes performance degradation.

Example:
```java
// Variables on same cache line
volatile long count1;
volatile long count2; // Same cache line - false sharing!
```

Prevention:
1. **Padding**: Add padding between variables
2. **@Contended annotation**: Java 8+ annotation to prevent false sharing
3. **Separate objects**: Put variables in different objects

```java
// With padding
volatile long count1;
long p1, p2, p3, p4, p5, p6, p7; // Padding
volatile long count2; // Different cache line
```"

#### Q13: Explain Fork/Join framework and work-stealing.

**Answer:**
"Fork/Join framework is designed for parallel execution of tasks that can be recursively split:
1. **Fork**: Split task into smaller subtasks
2. **Join**: Combine results from subtasks
3. **Work-stealing**: Idle threads steal work from busy threads

Work-stealing algorithm:
- Each thread has its own deque (double-ended queue)
- Threads push/pop from their own deque
- When deque is empty, thread steals from other threads' deques
- Reduces contention and improves load balancing

Use cases:
- Recursive algorithms (merge sort, quicksort)
- Parallel processing of large datasets
- Divide-and-conquer problems"

#### Q14: What is the difference between Callable and Runnable?

**Answer:**
"Key differences:
1. **Return value**: Callable returns a value, Runnable doesn't
2. **Exceptions**: Callable can throw checked exceptions, Runnable can't
3. **Usage**: Callable used with ExecutorService.submit(), Runnable with execute()
4. **Future**: Callable returns Future<T>, Runnable returns Future<?>

Use Callable when:
- Need return value
- Need to throw checked exceptions
- Working with thread pools

Use Runnable when:
- No return value needed
- Simple fire-and-forget tasks"

#### Q15: How do you handle thread leaks and resource cleanup?

**Answer:**
"Thread leaks occur when threads are created but never terminated. Prevention:
1. **Always shutdown ExecutorService**: Use shutdown() and awaitTermination()
2. **Use daemon threads**: For background tasks that shouldn't prevent JVM shutdown
3. **Clean up ThreadLocal**: Always call remove() to prevent memory leaks
4. **Monitor thread count**: Use JMX or monitoring tools
5. **Use try-with-resources**: For AutoCloseable executors

Example:
```java
ExecutorService executor = Executors.newFixedThreadPool(10);
try {
    // Use executor
} finally {
    executor.shutdown();
    if (!executor.awaitTermination(60, TimeUnit.SECONDS)) {
        executor.shutdownNow();
    }
}
```"

---

## Summary

### Key Takeaways

1. **Thread Fundamentals**: Understand thread lifecycle, creation methods, and states
2. **Synchronization**: Use synchronized, locks, or atomic classes based on requirements
3. **Thread Pools**: Always use ExecutorService instead of creating threads directly
4. **Concurrent Collections**: Use thread-safe collections (ConcurrentHashMap, BlockingQueue)
5. **Advanced Topics**: Understand memory model, volatile, ThreadLocal, and lock types
6. **CompletableFuture**: Use for asynchronous, non-blocking operations
7. **Best Practices**: Proper exception handling, resource cleanup, and avoiding deadlocks

### When to Use What

- **Simple synchronization**: `synchronized`
- **Advanced locking**: `ReentrantLock`
- **Simple counters/flags**: `AtomicInteger`, `volatile`
- **Thread pools**: `ExecutorService`
- **Asynchronous operations**: `CompletableFuture`
- **Parallel processing**: `ForkJoinPool`
- **Thread-safe collections**: `ConcurrentHashMap`, `BlockingQueue`
- **Thread-local data**: `ThreadLocal`

### Common Pitfalls

1. ❌ Creating threads directly instead of using thread pools
2. ❌ Not shutting down ExecutorService
3. ❌ Deadlocks from incorrect lock ordering
4. ❌ Race conditions from missing synchronization
5. ❌ Memory leaks from ThreadLocal
6. ❌ False sharing in high-performance code
7. ❌ Using synchronized collections incorrectly

---

**Next Steps:**
- Practice with real-world scenarios
- Understand JVM internals related to threads
- Learn about lock-free algorithms
- Study concurrent design patterns
- Monitor and profile thread performance

