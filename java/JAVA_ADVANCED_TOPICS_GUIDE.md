# Java Advanced Topics - Staff Engineer Interview Guide 🚀

> Comprehensive guide to advanced Java topics for staff engineer interviews - Deep dive into JVM internals, performance optimization, advanced language features, and system design integration

---

## Table of Contents
1. [JVM Internals Deep Dive](#jvm-internals-deep-dive)
2. [Memory Management Advanced](#memory-management-advanced)
3. [Performance Optimization](#performance-optimization)
4. [Advanced Concurrency](#advanced-concurrency)
5. [Reflection and Annotations](#reflection-and-annotations)
6. [Bytecode Manipulation](#bytecode-manipulation)
7. [Advanced Generics](#advanced-generics)
8. [Lambda and Streams Advanced](#lambda-and-streams-advanced)
9. [Java Module System](#java-module-system)
10. [Class Loading and ClassLoaders](#class-loading-and-classloaders)
11. [Instrumentation and Profiling](#instrumentation-and-profiling)
12. [Garbage Collection Tuning](#garbage-collection-tuning)
13. [JIT Compilation](#jit-compilation)
14. [Memory Models and Visibility](#memory-models-and-visibility)
15. [Advanced Design Patterns](#advanced-design-patterns)
16. [System Design Integration](#system-design-integration)
17. [Interview Questions](#interview-questions)

---

## JVM Internals Deep Dive

### JVM Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    JVM ARCHITECTURE                      │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    Class Loader Subsystem                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐              │
│  │Bootstrap │  │Extension │  │Application│             │
│  │ClassLoader│ │ClassLoader│ │ClassLoader│             │
│  └──────────┘  └──────────┘  └──────────┘              │
└─────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│                    Runtime Data Areas                    │
│  ┌──────────────┐  ┌──────────────┐                    │
│  │Method Area   │  │Heap          │                    │
│  │(Class Info)  │  │(Objects)     │                    │
│  └──────────────┘  └──────────────┘                    │
│  ┌──────────────┐  ┌──────────────┐                    │
│  │Stack Area    │  │PC Registers  │                    │
│  │(Frames)      │  │(Instructions)│                   │
│  └──────────────┘  └──────────────┘                    │
│  ┌──────────────┐                                       │
│  │Native Method │                                       │
│  │Stack         │                                       │
│  └──────────────┘                                       │
└─────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│              Execution Engine                           │
│  ┌──────────────┐  ┌──────────────┐                    │
│  │Interpreter   │  │JIT Compiler  │                    │
│  └──────────────┘  └──────────────┘                    │
│  ┌──────────────┐                                       │
│  │Garbage       │                                       │
│  │Collector     │                                       │
│  └──────────────┘                                       │
└─────────────────────────────────────────────────────────┘
```

### Runtime Data Areas

#### 1. Method Area (Metaspace in Java 8+)

**Stores:**
- Class metadata
- Static variables
- Method bytecode
- Constant pool
- Runtime constant pool

**Configuration:**
```bash
# Java 7 and earlier (PermGen)
-XX:PermSize=256m
-XX:MaxPermSize=512m

# Java 8+ (Metaspace)
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m
```

**Example:**
```java
public class MethodAreaExample {
    // Static variables stored in Method Area
    private static final String CONSTANT = "Hello";
    private static int counter = 0;
    
    // Class metadata stored in Method Area
    public void method() {
        // Method bytecode stored in Method Area
    }
}
```

#### 2. Heap Memory

**Structure:**
```
┌─────────────────────────────────────────┐
│              HEAP MEMORY                │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────────────────────────────┐   │
│  │      Young Generation            │   │
│  │  ┌──────────┐  ┌──────────┐    │   │
│  │  │  Eden    │  │  Survivor│    │   │
│  │  │          │  │  (S0/S1) │    │   │
│  │  └──────────┘  └──────────┘    │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │      Old Generation             │   │
│  │  (Tenured Space)                │   │
│  └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

**Heap Configuration:**
```bash
# Initial and maximum heap size
-Xms2g          # Initial heap size
-Xmx4g          # Maximum heap size

# Young generation size
-Xmn1g          # Young generation size
-XX:NewRatio=2  # Ratio of old to young (2:1)

# Survivor space ratio
-XX:SurvivorRatio=8  # Eden:Survivor (8:1:1)
```

#### 3. Stack Memory

**Stack Frame Structure:**
```
┌─────────────────────────────────────────┐
│           STACK FRAME                   │
├─────────────────────────────────────────┤
│  Local Variables                        │
│  ┌─────────────────────────────────┐   │
│  │  int a = 10;                    │   │
│  │  String b = "hello";            │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Operand Stack                          │
│  ┌─────────────────────────────────┐   │
│  │  [operand1] [operand2] [result] │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Frame Data                             │
│  ┌─────────────────────────────────┐   │
│  │  Return Address                 │   │
│  │  Reference to Constant Pool      │   │
│  │  Exception Table                 │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

**Stack Configuration:**
```bash
# Thread stack size
-Xss1m          # 1MB per thread stack
-Xss512k        # 512KB per thread stack
```

**Example:**
```java
public class StackExample {
    public void method1() {
        int localVar = 10;  // Stored in stack frame
        method2(localVar);
    }
    
    public void method2(int param) {
        String str = "test";  // New stack frame
        // Stack frame destroyed when method returns
    }
}
```

### Object Memory Layout

**Object Header Structure:**
```
┌─────────────────────────────────────────┐
│         OBJECT MEMORY LAYOUT            │
├─────────────────────────────────────────┤
│  Object Header (12-16 bytes)           │
│  ┌─────────────────────────────────┐   │
│  │  Mark Word (8 bytes)            │   │
│  │  - Hash code                    │   │
│  │  - GC age                       │   │
│  │  - Lock information             │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │  Class Pointer (4-8 bytes)      │   │
│  │  - Points to class metadata      │   │
│  └─────────────────────────────────┘   │
│  ┌─────────────────────────────────┐   │
│  │  Array Length (if array)        │   │
│  └─────────────────────────────────┘   │
├─────────────────────────────────────────┤
│  Instance Data                          │
│  ┌─────────────────────────────────┐   │
│  │  Fields (aligned to 8 bytes)    │   │
│  └─────────────────────────────────┘   │
├─────────────────────────────────────────┤
│  Padding (for alignment)                │
└─────────────────────────────────────────┘
```

**Example:**
```java
public class ObjectLayoutExample {
    // Object header: 12-16 bytes
    private int field1;      // 4 bytes
    private long field2;     // 8 bytes
    private boolean field3;  // 1 byte (but takes 4 bytes due to alignment)
    // Total: ~28-32 bytes (with padding)
}
```

### Mark Word Structure

**Mark Word (64-bit JVM):**
```
┌─────────────────────────────────────────┐
│           MARK WORD (64 bits)           │
├─────────────────────────────────────────┤
│  Normal Object:                         │
│  ┌─────────────────────────────────┐   │
│  │  Hash:25 | Age:4 | Biased:1     │   │
│  │  Lock:2  |      Unused:32       │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Locked Object:                         │
│  ┌─────────────────────────────────┐   │
│  │  Lock Record Pointer:62         │   │
│  │  Lock:2                          │   │
│  └─────────────────────────────────┘   │
│                                         │
│  Biased Lock:                           │
│  ┌─────────────────────────────────┐   │
│  │  Thread ID:54 | Epoch:2         │   │
│  │  Age:4 | Biased:1 | Lock:2      │   │
│  └─────────────────────────────────┘   │
└─────────────────────────────────────────┘
```

---

## Memory Management Advanced

### Memory Allocation Strategies

#### 1. TLAB (Thread-Local Allocation Buffer)

**Purpose:** Reduce contention in heap allocation

**How it works:**
- Each thread gets its own allocation buffer
- Objects allocated in TLAB don't need synchronization
- When TLAB is full, allocate new TLAB

**Configuration:**
```bash
-XX:+UseTLAB              # Enable TLAB (default)
-XX:TLABSize=256k         # TLAB size
-XX:TLABRefillWasteFraction=64  # Refill threshold
```

**Example:**
```java
public class TLABExample {
    public void allocateObjects() {
        // Objects allocated in thread's TLAB
        for (int i = 0; i < 1000; i++) {
            new Object();  // Fast allocation in TLAB
        }
    }
}
```

#### 2. Bump-the-Pointer Allocation

**Eden Space Allocation:**
```
┌─────────────────────────────────────────┐
│              EDEN SPACE                 │
├─────────────────────────────────────────┤
│  [allocated] [allocated] [free]         │
│                    ↑                    │
│                 pointer                 │
└─────────────────────────────────────────┘
```

**Advantages:**
- Very fast allocation (just increment pointer)
- No fragmentation in Eden
- Works well with TLAB

#### 3. Free List Allocation (Old Generation)

**How it works:**
- Maintains list of free memory chunks
- Searches for appropriate chunk
- Splits chunk if larger than needed

**Trade-offs:**
- Slower than bump-the-pointer
- Can cause fragmentation
- Used for old generation

### Memory Barriers and Ordering

#### Happens-Before Relationships

**Java Memory Model Rules:**

1. **Program Order Rule**
   ```java
   int x = 1;  // Happens-before
   int y = 2;  // This line
   ```

2. **Monitor Lock Rule**
   ```java
   synchronized (lock) {
       x = 1;  // Unlock happens-before
   }
   synchronized (lock) {
       int y = x;  // Subsequent lock
   }
   ```

3. **Volatile Rule**
   ```java
   volatile int x = 1;  // Write happens-before
   int y = x;           // Subsequent read
   ```

4. **Thread Start Rule**
   ```java
   x = 1;
   thread.start();  // Happens-before
   // All actions in thread
   ```

5. **Thread Join Rule**
   ```java
   // All actions in thread
   thread.join();  // Happens-before
   x = 1;
   ```

#### Memory Barriers

**Types:**
- **LoadLoad**: Ensures loads before barrier complete before loads after
- **StoreStore**: Ensures stores before barrier visible before stores after
- **LoadStore**: Ensures loads complete before stores
- **StoreLoad**: Ensures stores visible before loads (most expensive)

**Example:**
```java
public class MemoryBarrierExample {
    private volatile int flag = 0;
    private int data = 0;
    
    public void writer() {
        data = 42;           // Store
        flag = 1;            // Volatile store (StoreStore barrier)
    }
    
    public void reader() {
        if (flag == 1) {     // Volatile load (LoadLoad barrier)
            int value = data; // Load (guaranteed to see 42)
        }
    }
}
```

### Memory Leaks Detection

#### Common Memory Leak Patterns

**1. Static Collections**
```java
// ❌ BAD: Memory leak
public class Cache {
    private static final Map<String, Object> cache = new HashMap<>();
    
    public void add(String key, Object value) {
        cache.put(key, value);  // Never removed!
    }
}

// ✅ GOOD: Use WeakHashMap or size limits
public class Cache {
    private static final Map<String, Object> cache = new ConcurrentHashMap<>();
    private static final int MAX_SIZE = 1000;
    
    public void add(String key, Object value) {
        if (cache.size() >= MAX_SIZE) {
            // Remove oldest entries
            evictOldest();
        }
        cache.put(key, value);
    }
}
```

**2. ThreadLocal Not Cleaned**
```java
// ❌ BAD: Memory leak
public class ThreadLocalExample {
    private static final ThreadLocal<Map<String, Object>> context = 
        new ThreadLocal<Map<String, Object>>() {
            @Override
            protected Map<String, Object> initialValue() {
                return new HashMap<>();
            }
        };
    
    // Never calls context.remove()!
}

// ✅ GOOD: Always remove
public void cleanup() {
    try {
        // Use ThreadLocal
    } finally {
        context.remove();  // Always remove
    }
}
```

**3. Listeners Not Removed**
```java
// ❌ BAD: Memory leak
public class EventSource {
    private List<EventListener> listeners = new ArrayList<>();
    
    public void addListener(EventListener listener) {
        listeners.add(listener);  // Never removed!
    }
}

// ✅ GOOD: Use WeakReference or remove listeners
public class EventSource {
    private List<WeakReference<EventListener>> listeners = new ArrayList<>();
    
    public void addListener(EventListener listener) {
        listeners.add(new WeakReference<>(listener));
    }
}
```

#### Memory Leak Detection Tools

**1. Heap Dump Analysis**
```bash
# Generate heap dump
jmap -dump:format=b,file=heap.hprof <pid>

# Analyze with jhat or Eclipse MAT
jhat heap.hprof
```

**2. JVisualVM**
```bash
# Monitor memory usage
jvisualvm
```

**3. Memory Profilers**
- Eclipse MAT (Memory Analyzer Tool)
- YourKit
- JProfiler

---

## Performance Optimization

### JIT Compilation Optimization

#### HotSpot Compiler Levels

```
┌─────────────────────────────────────────┐
│      JIT COMPILATION LEVELS              │
├─────────────────────────────────────────┤
│  Level 0: Interpreter                   │
│  Level 1: C1 (Client Compiler)         │
│    - Fast compilation                   │
│    - Basic optimizations                │
│  Level 2: C1 with profiling             │
│  Level 3: C1 with full profiling        │
│  Level 4: C2 (Server Compiler)         │
│    - Slow compilation                   │
│    - Aggressive optimizations           │
└─────────────────────────────────────────┘
```

**Configuration:**
```bash
# Compiler thresholds
-XX:CompileThreshold=10000        # Method invocation threshold
-XX:TieredCompilation             # Enable tiered compilation
-XX:TieredStopAtLevel=1           # Stop at level 1

# Inlining
-XX:MaxInlineSize=35              # Max bytecode for inlining
-XX:FreqInlineSize=325            # Max bytecode for hot inlining
-XX:InlineSmallCode=1000          # Max code size for inlining
```

#### Common Optimizations

**1. Method Inlining**
```java
// Before optimization
public int calculate(int a, int b) {
    return add(a, b);
}

private int add(int a, int b) {
    return a + b;
}

// After inlining
public int calculate(int a, int b) {
    return a + b;  // Inlined
}
```

**2. Loop Unrolling**
```java
// Before
for (int i = 0; i < 4; i++) {
    array[i] = i;
}

// After unrolling
array[0] = 0;
array[1] = 1;
array[2] = 2;
array[3] = 3;
```

**3. Dead Code Elimination**
```java
// Before
if (false) {
    // Dead code - removed by compiler
    System.out.println("Never executed");
}
```

**4. Constant Folding**
```java
// Before
int result = 2 + 3;

// After
int result = 5;  // Computed at compile time
```

### Performance Tuning Strategies

#### 1. Reduce Object Allocation

**❌ BAD:**
```java
public String concatenate(String a, String b) {
    return a + b;  // Creates StringBuilder internally
}
```

**✅ GOOD:**
```java
public String concatenate(String a, String b) {
    return new StringBuilder(a.length() + b.length())
        .append(a)
        .append(b)
        .toString();
}
```

#### 2. Use Primitive Types

**❌ BAD:**
```java
List<Integer> numbers = new ArrayList<>();
for (int i = 0; i < 1000000; i++) {
    numbers.add(i);  // Autoboxing overhead
}
```

**✅ GOOD:**
```java
int[] numbers = new int[1000000];
for (int i = 0; i < 1000000; i++) {
    numbers[i] = i;  // No boxing
}
```

#### 3. Avoid Reflection in Hot Paths

**❌ BAD:**
```java
Method method = obj.getClass().getMethod("process");
method.invoke(obj);  // Slow reflection
```

**✅ GOOD:**
```java
obj.process();  // Direct call
```

#### 4. Use Appropriate Collections

**❌ BAD:**
```java
List<String> list = new LinkedList<>();  // For random access
String item = list.get(1000);  // O(n) operation
```

**✅ GOOD:**
```java
List<String> list = new ArrayList<>();  // For random access
String item = list.get(1000);  // O(1) operation
```

### Benchmarking

**Using JMH (Java Microbenchmark Harness):**
```java
@BenchmarkMode(Mode.AverageTime)
@OutputTimeUnit(TimeUnit.NANOSECONDS)
@State(Scope.Benchmark)
public class PerformanceBenchmark {
    
    private List<Integer> arrayList;
    private List<Integer> linkedList;
    
    @Setup
    public void setup() {
        arrayList = new ArrayList<>();
        linkedList = new LinkedList<>();
        for (int i = 0; i < 1000; i++) {
            arrayList.add(i);
            linkedList.add(i);
        }
    }
    
    @Benchmark
    public int arrayListGet() {
        return arrayList.get(500);
    }
    
    @Benchmark
    public int linkedListGet() {
        return linkedList.get(500);
    }
}
```

---

## Advanced Concurrency

### Lock-Free Algorithms

#### Compare-And-Swap (CAS)

**Atomic Operations:**
```java
import java.util.concurrent.atomic.AtomicInteger;

public class CASExample {
    private final AtomicInteger counter = new AtomicInteger(0);
    
    public void increment() {
        int current;
        int next;
        do {
            current = counter.get();
            next = current + 1;
        } while (!counter.compareAndSet(current, next));
    }
}
```

**ABA Problem:**
```java
// Problem: Value changes from A -> B -> A
// CAS sees A and succeeds, but value changed

// Solution: Use versioned references
public class VersionedReference<T> {
    private final T value;
    private final int version;
    
    public boolean compareAndSet(T expectedValue, int expectedVersion, T newValue) {
        if (this.value == expectedValue && this.version == expectedVersion) {
            this.value = newValue;
            this.version++;
            return true;
        }
        return false;
    }
}
```

#### Lock-Free Data Structures

**Lock-Free Stack:**
```java
import java.util.concurrent.atomic.AtomicReference;

public class LockFreeStack<T> {
    private static class Node<T> {
        final T value;
        final Node<T> next;
        
        Node(T value, Node<T> next) {
            this.value = value;
            this.next = next;
        }
    }
    
    private final AtomicReference<Node<T>> head = new AtomicReference<>();
    
    public void push(T value) {
        Node<T> newHead = new Node<>(value, null);
        Node<T> oldHead;
        do {
            oldHead = head.get();
            newHead.next = oldHead;
        } while (!head.compareAndSet(oldHead, newHead));
    }
    
    public T pop() {
        Node<T> oldHead;
        Node<T> newHead;
        do {
            oldHead = head.get();
            if (oldHead == null) {
                return null;
            }
            newHead = oldHead.next;
        } while (!head.compareAndSet(oldHead, newHead));
        return oldHead.value;
    }
}
```

### Advanced Synchronization

#### StampedLock Advanced Usage

```java
import java.util.concurrent.locks.StampedLock;

public class StampedLockAdvanced {
    private final StampedLock lock = new StampedLock();
    private double x, y;
    
    // Optimistic read
    public double distanceFromOrigin() {
        long stamp = lock.tryOptimisticRead();
        double curX = x, curY = y;
        
        if (!lock.validate(stamp)) {
            // Fallback to read lock
            stamp = lock.readLock();
            try {
                curX = x;
                curY = y;
            } finally {
                lock.unlockRead(stamp);
            }
        }
        return Math.sqrt(curX * curX + curY * curY);
    }
    
    // Write lock
    public void move(double deltaX, double deltaY) {
        long stamp = lock.writeLock();
        try {
            x += deltaX;
            y += deltaY;
        } finally {
            lock.unlockWrite(stamp);
        }
    }
    
    // Try to upgrade read lock to write lock
    public void moveIfAtOrigin(double newX, double newY) {
        long stamp = lock.readLock();
        try {
            while (x == 0.0 && y == 0.0) {
                long writeStamp = lock.tryConvertToWriteLock(stamp);
                if (writeStamp != 0L) {
                    stamp = writeStamp;
                    x = newX;
                    y = newY;
                    break;
                } else {
                    lock.unlockRead(stamp);
                    stamp = lock.writeLock();
                }
            }
        } finally {
            lock.unlock(stamp);
        }
    }
}
```

#### Phaser for Multi-Phase Coordination

```java
import java.util.concurrent.Phaser;

public class PhaserExample {
    private final Phaser phaser = new Phaser(3); // 3 parties
    
    public void phase1() {
        System.out.println("Phase 1: " + Thread.currentThread().getName());
        phaser.arriveAndAwaitAdvance(); // Wait for all parties
    }
    
    public void phase2() {
        System.out.println("Phase 2: " + Thread.currentThread().getName());
        phaser.arriveAndAwaitAdvance(); // Wait for all parties
    }
    
    public void phase3() {
        System.out.println("Phase 3: " + Thread.currentThread().getName());
        phaser.arriveAndDeregister(); // Complete and deregister
    }
}
```

---

## Reflection and Annotations

### Advanced Reflection

#### Dynamic Proxy

```java
import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;

public interface Service {
    void doSomething();
}

public class ServiceImpl implements Service {
    @Override
    public void doSomething() {
        System.out.println("Doing something");
    }
}

public class LoggingInvocationHandler implements InvocationHandler {
    private final Object target;
    
    public LoggingInvocationHandler(Object target) {
        this.target = target;
    }
    
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        System.out.println("Before: " + method.getName());
        Object result = method.invoke(target, args);
        System.out.println("After: " + method.getName());
        return result;
    }
}

// Usage
Service service = (Service) Proxy.newProxyInstance(
    Service.class.getClassLoader(),
    new Class[]{Service.class},
    new LoggingInvocationHandler(new ServiceImpl())
);

service.doSomething();
```

#### Method Handles (Java 7+)

```java
import java.lang.invoke.MethodHandle;
import java.lang.invoke.MethodHandles;
import java.lang.invoke.MethodType;

public class MethodHandleExample {
    public static void main(String[] args) throws Throwable {
        MethodHandles.Lookup lookup = MethodHandles.lookup();
        
        // Get method handle
        MethodHandle mh = lookup.findVirtual(
            String.class,
            "substring",
            MethodType.methodType(String.class, int.class, int.class)
        );
        
        // Invoke
        String result = (String) mh.invoke("Hello World", 0, 5);
        System.out.println(result); // "Hello"
    }
}
```

### Custom Annotations

#### Annotation Processing

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Cacheable {
    String key() default "";
    int ttl() default 3600;
    boolean evictOnUpdate() default true;
}

// Usage
public class ProductService {
    @Cacheable(key = "product:#{skuId}", ttl = 3600)
    public Product getProduct(Long skuId) {
        return productRepository.findById(skuId);
    }
}

// Annotation processor
public class CacheableProcessor {
    public Object process(Method method, Object[] args) {
        Cacheable annotation = method.getAnnotation(Cacheable.class);
        String key = resolveKey(annotation.key(), method, args);
        
        // Check cache
        Object cached = cache.get(key);
        if (cached != null) {
            return cached;
        }
        
        // Execute method
        Object result = invokeMethod(method, args);
        
        // Cache result
        cache.put(key, result, annotation.ttl());
        
        return result;
    }
}
```

---

## Bytecode Manipulation

### ASM Framework

**Example: Adding logging to methods:**
```java
import org.objectweb.asm.*;

public class LoggingClassVisitor extends ClassVisitor {
    public LoggingClassVisitor(ClassVisitor cv) {
        super(Opcodes.ASM9, cv);
    }
    
    @Override
    public MethodVisitor visitMethod(int access, String name, String descriptor,
                                     String signature, String[] exceptions) {
        MethodVisitor mv = cv.visitMethod(access, name, descriptor, signature, exceptions);
        
        if (mv != null && !name.equals("<init>") && !name.equals("<clinit>")) {
            mv = new LoggingMethodVisitor(mv, name);
        }
        
        return mv;
    }
    
    private static class LoggingMethodVisitor extends MethodVisitor {
        private final String methodName;
        
        public LoggingMethodVisitor(MethodVisitor mv, String methodName) {
            super(Opcodes.ASM9, mv);
            this.methodName = methodName;
        }
        
        @Override
        public void visitCode() {
            mv.visitCode();
            mv.visitFieldInsn(Opcodes.GETSTATIC, "java/lang/System", "out",
                "Ljava/io/PrintStream;");
            mv.visitLdcInsn("Entering: " + methodName);
            mv.visitMethodInsn(Opcodes.INVOKEVIRTUAL, "java/io/PrintStream", "println",
                "(Ljava/lang/String;)V", false);
        }
    }
}
```

### Javassist

**Example: Dynamic class modification:**
```java
import javassist.*;

public class JavassistExample {
    public static void main(String[] args) throws Exception {
        ClassPool pool = ClassPool.getDefault();
        CtClass cc = pool.get("com.example.MyClass");
        
        // Add method
        CtMethod method = CtNewMethod.make(
            "public void newMethod() { System.out.println(\"New method\"); }",
            cc
        );
        cc.addMethod(method);
        
        // Modify existing method
        CtMethod existingMethod = cc.getDeclaredMethod("existingMethod");
        existingMethod.insertBefore("System.out.println(\"Before\");");
        existingMethod.insertAfter("System.out.println(\"After\");");
        
        // Write class
        cc.writeFile();
    }
}
```

---

## Advanced Generics

### Type Erasure and Reification

**Type Erasure:**
```java
// At compile time
List<String> stringList = new ArrayList<>();
List<Integer> intList = new ArrayList<>();

// At runtime (after erasure)
List stringList = new ArrayList();  // Raw type
List intList = new ArrayList();     // Raw type
```

**Bypassing Type Erasure:**
```java
// Using Class objects
public <T> T createInstance(Class<T> clazz) throws Exception {
    return clazz.newInstance();
}

// Usage
String str = createInstance(String.class);
```

### Wildcards and Bounds

**PECS Principle (Producer Extends, Consumer Super):**
```java
// Producer - use extends
public void processNumbers(List<? extends Number> numbers) {
    for (Number n : numbers) {
        // Can read, but cannot add (except null)
    }
}

// Consumer - use super
public void addNumbers(List<? super Integer> numbers) {
    numbers.add(1);  // Can add, but cannot read specific type
}
```

**Advanced Bounds:**
```java
// Multiple bounds
public <T extends Number & Comparable<T>> T max(T a, T b) {
    return a.compareTo(b) > 0 ? a : b;
}

// Wildcard with bounds
public void process(List<? extends Comparable<? super Number>> list) {
    // Complex wildcard usage
}
```

---

## Lambda and Streams Advanced

### Custom Collectors

```java
import java.util.stream.Collector;
import java.util.stream.Collectors;

public class CustomCollectors {
    // Custom collector for statistics
    public static Collector<Integer, ?, Statistics> statistics() {
        return Collector.of(
            Statistics::new,
            Statistics::accept,
            Statistics::combine,
            Statistics::finish
        );
    }
    
    static class Statistics {
        private int count = 0;
        private int sum = 0;
        private int min = Integer.MAX_VALUE;
        private int max = Integer.MIN_VALUE;
        
        public void accept(int value) {
            count++;
            sum += value;
            min = Math.min(min, value);
            max = Math.max(max, value);
        }
        
        public Statistics combine(Statistics other) {
            count += other.count;
            sum += other.sum;
            min = Math.min(min, other.min);
            max = Math.max(max, other.max);
            return this;
        }
        
        public Statistics finish() {
            return this;
        }
        
        public double average() {
            return count > 0 ? (double) sum / count : 0;
        }
    }
}
```

### Parallel Streams Optimization

```java
public class ParallelStreamOptimization {
    // ✅ GOOD: Use parallel for CPU-intensive operations
    public long sumLargeArray(int[] array) {
        return Arrays.stream(array)
            .parallel()
            .mapToLong(i -> (long) i * i)  // CPU-intensive
            .sum();
    }
    
    // ❌ BAD: Don't use parallel for I/O operations
    public List<String> readFiles(List<Path> paths) {
        return paths.stream()
            .parallel()  // I/O doesn't benefit from parallel
            .map(this::readFile)
            .collect(Collectors.toList());
    }
    
    // ✅ GOOD: Use custom ForkJoinPool for better control
    public void customParallelProcessing() {
        ForkJoinPool customPool = new ForkJoinPool(4);
        customPool.submit(() -> {
            IntStream.range(0, 1000000)
                .parallel()
                .filter(i -> i % 2 == 0)
                .sum();
        }).join();
    }
}
```

---

## Java Module System (Java 9+)

### Module Definition

**module-info.java:**
```java
module com.ekart.product.service {
    // Exported packages
    exports com.ekart.product.service.api;
    exports com.ekart.product.service.dto;
    
    // Required modules
    requires java.base;
    requires java.sql;
    requires spring.core;
    requires spring.beans;
    
    // Transitive dependencies
    requires transitive com.ekart.common;
    
    // Services
    provides com.ekart.product.service.spi.ProductService
        with com.ekart.product.service.impl.ProductServiceImpl;
    
    uses com.ekart.product.service.spi.ProductService;
    
    // Opens for reflection
    opens com.ekart.product.service.internal
        to spring.core;
}
```

### Module Layers

```java
import java.lang.module.Configuration;
import java.lang.module.ModuleFinder;
import java.lang.module.ModuleLayer;

public class ModuleLayerExample {
    public static void main(String[] args) {
        // Create module layer
        ModuleFinder finder = ModuleFinder.of(Paths.get("modules"));
        Configuration config = ModuleLayer.boot()
            .configuration()
            .resolve(finder, ModuleFinder.of(), Set.of("com.example.module"));
        
        ModuleLayer layer = ModuleLayer.boot()
            .defineModulesWithOneLoader(config, ClassLoader.getSystemClassLoader());
        
        // Use modules in layer
        Class<?> clazz = layer.findLoader("com.example.module")
            .loadClass("com.example.Class");
    }
}
```

---

## Class Loading and ClassLoaders

### Custom ClassLoader

```java
public class CustomClassLoader extends ClassLoader {
    private final String basePath;
    
    public CustomClassLoader(String basePath, ClassLoader parent) {
        super(parent);
        this.basePath = basePath;
    }
    
    @Override
    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = loadClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException(name);
        }
        return defineClass(name, classData, 0, classData.length);
    }
    
    private byte[] loadClassData(String className) {
        String path = basePath + File.separatorChar +
            className.replace('.', File.separatorChar) + ".class";
        try {
            InputStream is = new FileInputStream(path);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 4096;
            byte[] buffer = new byte[bufferSize];
            int bytesNumRead;
            while ((bytesNumRead = is.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesNumRead);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            return null;
        }
    }
}
```

### Class Loading Process

```
1. Loading
   └─ Find and load .class file
   └─ Create Class object

2. Linking
   ├─ Verification
   │  └─ Check bytecode validity
   ├─ Preparation
   │  └─ Allocate memory for static variables
   └─ Resolution
      └─ Resolve symbolic references

3. Initialization
   └─ Execute <clinit> method
   └─ Initialize static variables
```

---

## Instrumentation and Profiling

### Java Agent

```java
import java.lang.instrument.Instrumentation;

public class MyAgent {
    public static void premain(String agentArgs, Instrumentation inst) {
        inst.addTransformer(new MyTransformer());
    }
    
    static class MyTransformer implements ClassFileTransformer {
        @Override
        public byte[] transform(ClassLoader loader, String className,
                               Class<?> classBeingRedefined,
                               ProtectionDomain protectionDomain,
                               byte[] classfileBuffer) {
            // Transform bytecode
            return transformBytecode(classfileBuffer);
        }
    }
}
```

**Manifest:**
```
Manifest-Version: 1.0
Premain-Class: MyAgent
```

**Usage:**
```bash
java -javaagent:myagent.jar MyApplication
```

---

## Garbage Collection Tuning

### GC Algorithms Comparison

| GC Algorithm | Throughput | Latency | Footprint | Use Case |
|-------------|------------|---------|-----------|----------|
| Serial GC | Low | High | Small | Small apps |
| Parallel GC | High | Medium | Medium | Batch processing |
| G1 GC | High | Low | Medium | Large heaps |
| ZGC | High | Very Low | Large | Low latency |
| Shenandoah | High | Very Low | Large | Low latency |

### G1 GC Tuning

```bash
# Enable G1
-XX:+UseG1GC

# Heap size
-Xms4g -Xmx4g

# G1 specific
-XX:MaxGCPauseMillis=200        # Target pause time
-XX:G1HeapRegionSize=16m         # Region size
-XX:InitiatingHeapOccupancyPercent=45  # Start concurrent marking
-XX:ConcGCThreads=4             # Concurrent GC threads
-XX:ParallelGCThreads=8          # Parallel GC threads
```

### GC Logging

```bash
# Java 8
-XX:+PrintGCDetails
-XX:+PrintGCDateStamps
-Xloggc:gc.log

# Java 9+
-Xlog:gc*:file=gc.log:time,level,tags
-Xlog:gc+heap:file=heap.log
```

---

## Interview Questions

### Q1: Explain JVM memory model and how objects are allocated.

**Answer:**
"The JVM memory model consists of several runtime data areas:
1. **Method Area (Metaspace)**: Stores class metadata, static variables, and method bytecode
2. **Heap**: Stores objects, divided into Young (Eden, Survivor) and Old generation
3. **Stack**: Stores local variables and method frames, one per thread
4. **PC Registers**: Store instruction pointers
5. **Native Method Stack**: For native method calls

Object allocation:
- New objects allocated in Eden space using TLAB (Thread-Local Allocation Buffer)
- When Eden is full, minor GC occurs
- Surviving objects moved to Survivor space
- After multiple GC cycles, objects promoted to Old generation
- Old generation uses mark-sweep-compact algorithms

I optimize allocation by:
- Using object pooling for frequently created objects
- Avoiding unnecessary object creation in hot paths
- Using primitive types instead of wrappers
- Pre-sizing collections to avoid resizing"

### Q2: How does JIT compilation work and what optimizations does it perform?

**Answer:**
"JIT compilation converts frequently executed bytecode to native machine code for better performance. HotSpot uses tiered compilation:

1. **Level 0**: Interpreter - executes bytecode directly
2. **Level 1-3**: C1 compiler - fast compilation with basic optimizations
3. **Level 4**: C2 compiler - slow compilation with aggressive optimizations

Key optimizations:
- **Method inlining**: Replaces method calls with method body
- **Loop unrolling**: Reduces loop overhead
- **Dead code elimination**: Removes unreachable code
- **Constant folding**: Computes constants at compile time
- **Escape analysis**: Allocates objects on stack instead of heap
- **Lock coarsening**: Combines adjacent synchronized blocks

I monitor JIT with -XX:+PrintCompilation and optimize hot methods by:
- Keeping methods small for better inlining
- Using final methods/classes for devirtualization
- Avoiding reflection in hot paths"

### Q3: Explain memory barriers and happens-before relationships.

**Answer:**
"Memory barriers ensure memory visibility and ordering guarantees:

**Types:**
- **LoadLoad**: Ensures loads before barrier complete before loads after
- **StoreStore**: Ensures stores before barrier visible before stores after
- **LoadStore**: Ensures loads complete before stores
- **StoreLoad**: Most expensive, ensures stores visible before loads

**Happens-before relationships:**
1. **Program order**: Actions in same thread happen in program order
2. **Monitor lock**: Unlock happens-before subsequent lock
3. **Volatile**: Write happens-before subsequent read
4. **Thread start**: start() happens-before thread actions
5. **Thread join**: Thread actions happen-before join() returns
6. **Transitivity**: If A happens-before B and B happens-before C, then A happens-before C

In practice, I use volatile for simple flags, synchronized for complex synchronization, and atomic classes for lock-free operations."

### Q4: How do you detect and fix memory leaks?

**Answer:**
"Memory leak detection process:

1. **Symptoms**: OutOfMemoryError, increasing heap usage, performance degradation

2. **Tools**:
   - Heap dumps: `jmap -dump:format=b,file=heap.hprof <pid>`
   - Analysis: Eclipse MAT, jhat, JVisualVM
   - Profilers: YourKit, JProfiler

3. **Common patterns**:
   - Static collections growing unbounded
   - ThreadLocal not cleaned
   - Listeners not removed
   - Circular references (though GC handles these)

4. **Fixes**:
   - Use WeakReference for caches
   - Always remove ThreadLocal values
   - Implement size limits on collections
   - Use connection pooling with proper cleanup

Example from our codebase: I ensure ThreadLocal values are removed in finally blocks, and use WeakHashMap for caches that should be garbage collected."

### Q5: Explain lock-free algorithms and when to use them.

**Answer:**
"Lock-free algorithms use atomic operations (CAS) instead of locks:

**Advantages:**
- No blocking - threads never wait
- Better scalability
- No deadlocks

**Disadvantages:**
- More complex to implement
- ABA problem
- Higher CPU usage (spinning)

**CAS (Compare-And-Swap):**
```java
do {
    current = value.get();
    next = current + 1;
} while (!value.compareAndSet(current, next));
```

**When to use:**
- High contention scenarios
- Simple operations (counters, flags)
- Performance-critical paths

**When NOT to use:**
- Complex operations
- Need for blocking semantics
- Low contention (locks are simpler)

I use AtomicInteger for counters, ConcurrentHashMap for maps, and implement lock-free stacks/queues for high-performance scenarios."

---

## Summary

### Key Takeaways

1. **JVM Internals**: Understand memory layout, object allocation, and GC
2. **Performance**: Profile, benchmark, and optimize based on data
3. **Concurrency**: Use appropriate synchronization mechanisms
4. **Memory**: Detect and prevent leaks, understand visibility
5. **Tools**: Master profiling and debugging tools
6. **Advanced Features**: Reflection, bytecode manipulation, modules

### Staff Engineer Skills

- **System Thinking**: Understand how Java fits into larger systems
- **Performance**: Optimize at multiple levels (code, JVM, system)
- **Debugging**: Diagnose complex issues in production
- **Architecture**: Design systems considering JVM characteristics
- **Mentoring**: Guide teams on best practices

---

**Next Steps:**
- Practice with real production scenarios
- Deep dive into specific GC algorithms
- Experiment with bytecode manipulation
- Profile real applications
- Study JVM source code

