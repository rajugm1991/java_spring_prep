# JVM & Garbage Collection - Advanced Interview Guide 🚀

> Comprehensive guide for senior engineers covering JVM internals, memory management, and garbage collection

---

## Table of Contents
1. [JVM Architecture](#jvm-architecture)
2. [Memory Areas](#memory-areas)
3. [Class Loading](#class-loading)
4. [Garbage Collection Fundamentals](#garbage-collection-fundamentals)
5. [GC Algorithms](#gc-algorithms)
6. [GC Tuning & Optimization](#gc-tuning--optimization)
7. [Memory Leaks & Troubleshooting](#memory-leaks--troubleshooting)
8. [JVM Monitoring & Tools](#jvm-monitoring--tools)
9. [Advanced Topics](#advanced-topics)
10. [Interview Questions](#interview-questions)

---

## JVM Architecture

### What is JVM?

**JVM (Java Virtual Machine)** is an abstract computing machine that enables Java programs to run on any platform without modification.

### JVM Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    JVM ARCHITECTURE                     │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│                    Class Loader Subsystem               │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐            │
│  │Bootstrap │  │Extension │  │Application│            │
│  │  Loader │  │  Loader  │  │  Loader   │            │
│  └──────────┘  └──────────┘  └──────────┘            │
└─────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│                    Runtime Data Areas                   │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │              Method Area (Metaspace)             │  │
│  │  - Class metadata, constants, static variables  │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │                    Heap                           │  │
│  │  ┌──────────────┐  ┌──────────────┐            │  │
│  │  │ Young Gen   │  │  Old Gen     │            │  │
│  │  │  - Eden     │  │  - Tenured   │            │  │
│  │  │  - S0       │  │              │            │  │
│  │  │  - S1       │  │              │            │  │
│  │  └──────────────┘  └──────────────┘            │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │              Java Stack (per thread)             │  │
│  │  - Local variables, method calls, return values │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │              PC Register (per thread)            │  │
│  │  - Current instruction pointer                   │  │
│  └──────────────────────────────────────────────────┘  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │              Native Method Stack                 │  │
│  │  - Native method calls                           │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│              Execution Engine                            │
│  ┌──────────────┐  ┌──────────────┐                  │
│  │ Interpreter  │  │   JIT Compiler│                  │
│  │              │  │   (C1, C2)    │                  │
│  └──────────────┘  └──────────────┘                  │
│                                                          │
│  ┌──────────────────────────────────────────────────┐  │
│  │              Garbage Collector                    │  │
│  │  - Mark & Sweep, Mark & Compact, etc.            │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────────────┐
│              Native Method Interface (JNI)              │
│              Native Method Libraries                    │
└─────────────────────────────────────────────────────────┘
```

### Key Components

1. **Class Loader Subsystem**: Loads, links, and initializes classes
2. **Runtime Data Areas**: Memory areas for execution
3. **Execution Engine**: Executes bytecode (Interpreter + JIT Compiler)
4. **Garbage Collector**: Manages memory automatically
5. **Native Interface**: Interface for native code

---

## Memory Areas

### 1. Method Area (Metaspace in Java 8+)

**Purpose:** Stores class metadata, constants, static variables

**Characteristics:**
- Shared across all threads
- Created at JVM startup
- Can be garbage collected (unlike PermGen)
- Size: `-XX:MetaspaceSize` and `-XX:MaxMetaspaceSize`

**What's Stored:**
- Class structure (fields, methods, constructors)
- Runtime constant pool
- Static variables
- Method bytecode

**Java 7 vs Java 8+:**

```
Java 7:
┌─────────────────┐
│   PermGen       │  ← Fixed size, OutOfMemoryError risk
│   (64MB default)│
└─────────────────┘

Java 8+:
┌─────────────────┐
│   Metaspace     │  ← Native memory, auto-grows
│   (unlimited)   │
└─────────────────┘
```

**Configuration:**
```bash
# Java 7 (PermGen)
-XX:PermSize=256m
-XX:MaxPermSize=512m

# Java 8+ (Metaspace)
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m
```

### 2. Heap Memory

**Purpose:** Stores object instances

**Structure:**

```
┌─────────────────────────────────────────────────┐
│                    HEAP                         │
│                                                 │
│  ┌───────────────────────────────────────────┐ │
│  │         Young Generation                  │ │
│  │  ┌──────────┐  ┌──────┐  ┌──────┐       │ │
│  │  │  Eden    │  │  S0  │  │  S1  │       │ │
│  │  │  (80%)   │  │(10%) │  │(10%) │       │ │
│  │  └──────────┘  └──────┘  └──────┘       │ │
│  └───────────────────────────────────────────┘ │
│                                                 │
│  ┌───────────────────────────────────────────┐ │
│  │         Old Generation (Tenured)          │ │
│  │  - Long-lived objects                     │ │
│  └───────────────────────────────────────────┘ │
└─────────────────────────────────────────────────┘
```

**Heap Regions:**

1. **Eden Space**: New objects allocated here
2. **Survivor Space (S0, S1)**: Objects that survive minor GC
3. **Old Generation**: Objects that survive multiple GC cycles

**Heap Size Configuration:**
```bash
# Initial and maximum heap size
-Xms2g          # Initial heap size
-Xmx4g          # Maximum heap size

# Young generation size
-Xmn1g          # Young generation size
-XX:NewRatio=2  # Ratio of old:young (2:1)

# Survivor space ratio
-XX:SurvivorRatio=8  # Eden:Survivor (8:1:1)
```

### 3. Java Stack (Per Thread)

**Purpose:** Stores method calls, local variables, return values

**Structure:**

```
┌─────────────────────────────────────┐
│         Java Stack (Thread 1)       │
│  ┌───────────────────────────────┐  │
│  │   Stack Frame (method3)      │  │  ← Top
│  │   - Local variables          │  │
│  │   - Operand stack            │  │
│  │   - Return value             │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │   Stack Frame (method2)      │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │   Stack Frame (method1)      │  │
│  └───────────────────────────────┘  │
│  ┌───────────────────────────────┐  │
│  │   Stack Frame (main)         │  │  ← Bottom
│  └───────────────────────────────┘  │
└─────────────────────────────────────┘
```

**Stack Frame Components:**

1. **Local Variable Array**: Method parameters and local variables
2. **Operand Stack**: Intermediate calculations
3. **Frame Data**: Return address, exception table

**Configuration:**
```bash
-Xss1m          # Stack size per thread (default 1MB)
-XX:ThreadStackSize=1024
```

**Stack Overflow Example:**
```java
public class StackOverflowExample {
    public static void recursiveCall(int count) {
        System.out.println("Count: " + count);
        recursiveCall(count + 1);  // Infinite recursion
    }
    
    public static void main(String[] args) {
        recursiveCall(0);  // StackOverflowError
    }
}
```

### 4. PC Register (Program Counter)

**Purpose:** Stores address of current instruction being executed

**Characteristics:**
- One per thread
- Small memory area
- Points to next instruction

### 5. Native Method Stack

**Purpose:** Stores native method calls (C/C++ code)

**Characteristics:**
- Separate from Java stack
- Used by JNI (Java Native Interface)

---

## Class Loading

### Class Loading Process

```
┌─────────────────────────────────────────────────┐
│           CLASS LOADING PHASES                  │
└─────────────────────────────────────────────────┘

Phase 1: Loading
    │
    ├─→ Find .class file
    ├─→ Read binary data
    └─→ Create Class object in method area
    │
    ▼
Phase 2: Linking
    │
    ├─→ Verification
    │   ├─→ Check file format
    │   ├─→ Check bytecode validity
    │   └─→ Check symbol references
    │
    ├─→ Preparation
    │   ├─→ Allocate memory for static variables
    │   └─→ Initialize with default values
    │
    └─→ Resolution
        └─→ Replace symbolic references with direct references
    │
    ▼
Phase 3: Initialization
    │
    ├─→ Execute static initializers
    ├─→ Initialize static variables
    └─→ Execute <clinit> method
```

### Class Loader Hierarchy

```
┌─────────────────────────────────────┐
│    Bootstrap Class Loader (null)    │  ← JVM internal
│    - Loads rt.jar, core Java classes│
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│    Extension Class Loader           │
│    - Loads jre/lib/ext/*.jar        │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│    Application/System Class Loader │
│    - Loads classpath classes        │
└─────────────────────────────────────┘
              │
              ▼
┌─────────────────────────────────────┐
│    Custom Class Loader              │
│    - User-defined class loaders     │
└─────────────────────────────────────┘
```

### Delegation Model

**Principle:** Child class loader delegates to parent first

```
Application requests: MyClass.class

1. Application Class Loader
   └─→ Check if already loaded? No
       └─→ Delegate to Extension Class Loader

2. Extension Class Loader
   └─→ Check if already loaded? No
       └─→ Delegate to Bootstrap Class Loader

3. Bootstrap Class Loader
   └─→ Check if already loaded? No
       └─→ Try to load from rt.jar? No
           └─→ Return to Extension

4. Extension Class Loader
   └─→ Try to load from ext/*.jar? No
       └─→ Return to Application

5. Application Class Loader
   └─→ Load from classpath ✅
```

**Code Example:**
```java
public class ClassLoaderExample {
    public static void main(String[] args) {
        // Get class loader
        ClassLoader loader = ClassLoaderExample.class.getClassLoader();
        System.out.println("Application Class Loader: " + loader);
        
        // Parent class loader
        System.out.println("Extension Class Loader: " + loader.getParent());
        
        // Bootstrap class loader (null in Java)
        System.out.println("Bootstrap Class Loader: " + loader.getParent().getParent());
        
        // Load class
        try {
            Class<?> clazz = loader.loadClass("java.lang.String");
            System.out.println("Loaded: " + clazz.getName());
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
    }
}
```

---

## Garbage Collection Fundamentals

### What is Garbage Collection?

**Garbage Collection (GC)** is the automatic memory management process that reclaims memory occupied by objects that are no longer in use.

### Object Lifecycle

```
┌─────────────────────────────────────────┐
│         OBJECT LIFECYCLE                 │
└─────────────────────────────────────────┘

1. Creation (Eden)
   │
   ├─→ Object allocated in Eden space
   │
   ▼
2. Minor GC (Young Generation)
   │
   ├─→ If object is still referenced
   │   └─→ Move to Survivor Space (S0 or S1)
   │
   ├─→ If object survives multiple GCs
   │   └─→ Promote to Old Generation
   │
   └─→ If object is not referenced
       └─→ Marked for deletion
   │
   ▼
3. Major GC (Old Generation)
   │
   ├─→ If object is still referenced
   │   └─→ Keep in Old Generation
   │
   └─→ If object is not referenced
       └─→ Reclaim memory
```

### GC Roots

**GC Roots** are objects that are always reachable and never garbage collected.

**Types of GC Roots:**

1. **Local variables** in active stack frames
2. **Static variables** in loaded classes
3. **Active threads**
4. **JNI references** (native code)
5. **Synchronized objects** (monitors)

```
┌─────────────────────────────────────────┐
│           GC ROOTS                      │
└─────────────────────────────────────────┘

GC Root 1: Local Variable
    │
    └─→ Object A ──→ Object B ──→ Object C
                    │
                    └─→ Object D

GC Root 2: Static Variable
    │
    └─→ Object E ──→ Object F

GC Root 3: Active Thread
    │
    └─→ Object G

All objects reachable from GC Roots are LIVE
All other objects are GARBAGE
```

### Reachability Levels

```
┌─────────────────────────────────────────┐
│      REACHABILITY LEVELS                 │
└─────────────────────────────────────────┘

Strong Reference (Always reachable)
    Object obj = new Object();

Soft Reference (Cleared when memory is low)
    SoftReference<Object> softRef = new SoftReference<>(obj);

Weak Reference (Cleared on next GC)
    WeakReference<Object> weakRef = new WeakReference<>(obj);

Phantom Reference (Finalization queue)
    PhantomReference<Object> phantomRef = new PhantomReference<>(obj, queue);
```

---

## GC Algorithms

### GC Algorithm Types

#### 1. Serial GC

**Characteristics:**
- Single-threaded
- Stop-the-world (STW)
- Suitable for small applications

**Algorithm:** Mark-Sweep-Compact

**Process:**
```
┌─────────────────────────────────────────┐
│         SERIAL GC PROCESS                │
└─────────────────────────────────────────┘

1. Stop all application threads (STW)
   │
   ▼
2. Mark Phase
   ├─→ Start from GC roots
   ├─→ Mark all reachable objects
   └─→ Traverse object graph
   │
   ▼
3. Sweep Phase
   ├─→ Identify unmarked objects
   └─→ Mark them as garbage
   │
   ▼
4. Compact Phase
   ├─→ Move live objects together
   └─→ Update references
   │
   ▼
5. Resume application threads
```

**Configuration:**
```bash
-XX:+UseSerialGC
```

**Use Case:** Small applications, single-core systems

#### 2. Parallel GC (Throughput Collector)

**Characteristics:**
- Multi-threaded
- Stop-the-world
- Optimized for throughput

**Algorithm:** Parallel Mark-Sweep-Compact

**Process:**
```
┌─────────────────────────────────────────┐
│         PARALLEL GC PROCESS              │
└─────────────────────────────────────────┘

Thread 1: Mark region A ──┐
Thread 2: Mark region B ──┤
Thread 3: Mark region C ──┼─→ Merge results
Thread 4: Mark region D ──┘
         │
         ▼
Parallel Sweep & Compact
```

**Configuration:**
```bash
-XX:+UseParallelGC
-XX:ParallelGCThreads=4        # Number of GC threads
-XX:MaxGCPauseMillis=200       # Target pause time
-XX:GCTimeRatio=19             # GC time ratio (1:19)
```

**Use Case:** Batch processing, high throughput applications

#### 3. CMS (Concurrent Mark Sweep)

**Characteristics:**
- Concurrent (runs with application)
- Low pause times
- More CPU intensive

**Algorithm:** Concurrent Mark-Sweep

**Phases:**
```
┌─────────────────────────────────────────┐
│         CMS GC PHASES                    │
└─────────────────────────────────────────┘

Phase 1: Initial Mark (STW)
   ├─→ Mark GC roots
   └─→ Short pause
   │
   ▼
Phase 2: Concurrent Mark
   ├─→ Mark reachable objects
   ├─→ Runs concurrently with application
   └─→ Application continues running
   │
   ▼
Phase 3: Remark (STW)
   ├─→ Mark objects changed during concurrent mark
   └─→ Short pause
   │
   ▼
Phase 4: Concurrent Sweep
   ├─→ Reclaim memory
   └─→ Runs concurrently with application
```

**Configuration:**
```bash
-XX:+UseConcMarkSweepGC
-XX:CMSInitiatingOccupancyFraction=70  # Start at 70% full
-XX:+UseCMSInitiatingOccupancyOnly
-XX:+CMSParallelRemarkEnabled         # Parallel remark
```

**Use Case:** Low-latency applications, interactive systems

**Limitations:**
- Fragmentation issues
- CPU overhead
- Deprecated in Java 9+, removed in Java 14

#### 4. G1 GC (Garbage First)

**Characteristics:**
- Low pause times
- Predictable pause times
- Designed for large heaps (>4GB)

**Algorithm:** Region-based, incremental

**Heap Structure:**
```
┌─────────────────────────────────────────┐
│         G1 HEAP STRUCTURE                │
└─────────────────────────────────────────┘

Heap divided into regions (1MB - 32MB each)

┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
│ Eden│ │Surv │ │ Old │ │ Hum │
│     │ │     │ │     │ │ong  │
└─────┘ └─────┘ └─────┘ └─────┘
┌─────┐ ┌─────┐ ┌─────┐ ┌─────┐
│ Eden│ │Surv │ │ Old │ │ Hum │
│     │ │     │ │     │ │ong  │
└─────┘ └─────┘ └─────┘ └─────┘
```

**Phases:**
```
┌─────────────────────────────────────────┐
│         G1 GC PHASES                     │
└─────────────────────────────────────────┘

Young Generation Collection:
   1. Stop-the-world
   2. Evacuate (copy) live objects
   3. Reclaim entire regions

Mixed Collection:
   1. Collect young generation
   2. Collect some old generation regions
   3. Prioritize regions with most garbage

Concurrent Marking:
   1. Mark live objects concurrently
   2. Identify regions for collection
```

**Configuration:**
```bash
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200        # Target pause time
-XX:G1HeapRegionSize=16m        # Region size
-XX:InitiatingHeapOccupancyPercent=45  # Start concurrent marking
```

**Use Case:** Large heaps, low-latency requirements

#### 5. ZGC (Z Garbage Collector)

**Characteristics:**
- Ultra-low latency (<10ms pauses)
- Scalable to multi-terabyte heaps
- Concurrent (no stop-the-world)

**Algorithm:** Concurrent mark and relocate

**Key Features:**
- Colored pointers (metadata in pointers)
- Load barriers (read-time barriers)
- Concurrent relocation

**Configuration:**
```bash
-XX:+UseZGC
-XX:+UnlockExperimentalVMOptions  # Java 11-14
```

**Use Case:** Very large heaps, ultra-low latency

#### 6. Shenandoah GC

**Characteristics:**
- Concurrent evacuation
- Low pause times
- Similar to ZGC

**Algorithm:** Concurrent mark and evacuate

**Configuration:**
```bash
-XX:+UseShenandoahGC
-XX:+UnlockExperimentalVMOptions
```

**Use Case:** Large heaps, low-latency

### GC Algorithm Comparison

| GC Algorithm | Pause Time | Throughput | Heap Size | Use Case |
|-------------|------------|------------|-----------|----------|
| **Serial** | High | Medium | Small | Small apps |
| **Parallel** | High | High | Medium | Batch processing |
| **CMS** | Low | Medium | Medium | Low latency (deprecated) |
| **G1** | Low | Medium | Large | Large heaps, low latency |
| **ZGC** | Ultra-low | Medium | Very Large | Ultra-low latency |
| **Shenandoah** | Low | Medium | Large | Low latency |

---

## GC Tuning & Optimization

### GC Tuning Strategy

```
┌─────────────────────────────────────────┐
│      GC TUNING STRATEGY                  │
└─────────────────────────────────────────┘

1. Measure First
   ├─→ GC logs
   ├─→ JVM metrics
   └─→ Application metrics
   │
   ▼
2. Identify Issues
   ├─→ High pause times?
   ├─→ Low throughput?
   ├─→ Memory leaks?
   └─→ OutOfMemoryError?
   │
   ▼
3. Tune Parameters
   ├─→ Heap size
   ├─→ GC algorithm
   ├─→ GC parameters
   └─→ Application code
   │
   ▼
4. Validate
   ├─→ Load testing
   ├─→ Monitor metrics
   └─→ Iterate
```

### Common GC Tuning Parameters

#### Heap Size Tuning

```bash
# Initial and maximum heap
-Xms4g                    # Start with 4GB
-Xmx8g                    # Maximum 8GB

# Rule of thumb: Xms = Xmx (avoid heap resizing)
```

#### Young Generation Tuning

```bash
# Young generation size
-Xmn2g                    # 2GB for young gen
-XX:NewRatio=2            # Old:Young = 2:1

# Survivor space ratio
-XX:SurvivorRatio=8        # Eden:Survivor = 8:1:1
```

#### G1 GC Tuning

```bash
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200   # Target pause time
-XX:G1HeapRegionSize=16m   # Region size
-XX:InitiatingHeapOccupancyPercent=45
-XX:ConcGCThreads=4        # Concurrent threads
```

#### GC Logging

```bash
# Java 8 and earlier
-XX:+PrintGC
-XX:+PrintGCDetails
-XX:+PrintGCTimeStamps
-Xloggc:/path/to/gc.log

# Java 9+
-Xlog:gc*:file=/path/to/gc.log:time,level,tags
-Xlog:gc*:file=/path/to/gc.log:time,level,tags:filecount=5,filesize=10M
```

### GC Log Analysis

**Sample GC Log:**
```
[0.123s][info][gc] Using G1
[1.456s][info][gc] GC(0) Pause Young (Normal) 512M->256M (2G) 45.123ms
[2.789s][info][gc] GC(1) Pause Young (Normal) 768M->384M (2G) 52.456ms
[5.234s][info][gc] GC(2) Pause Mixed 1.5G->800M (4G) 120.789ms
```

**Key Metrics:**
- **Pause Time**: How long application was stopped
- **Throughput**: Percentage of time spent in GC
- **Memory**: Before -> After (Total heap)

### GC Optimization Tips

1. **Avoid Premature Optimization**
   - Measure first, tune later
   - Use default GC for most cases

2. **Right-Size Heap**
   - Too small → Frequent GC
   - Too large → Long GC pauses

3. **Minimize Object Creation**
   - Reuse objects where possible
   - Use object pooling for expensive objects

4. **Avoid Memory Leaks**
   - Clear references
   - Use weak references for caches

5. **Tune for Your Use Case**
   - Low latency → G1, ZGC, Shenandoah
   - High throughput → Parallel GC

---

## Memory Leaks & Troubleshooting

### What is a Memory Leak?

**Memory Leak:** Objects that are no longer needed but still referenced, preventing garbage collection.

### Common Memory Leak Patterns

#### 1. Static Collections

**Problem:** Static collections hold references forever

```java
// ❌ BAD: Memory leak
public class Cache {
    private static Map<String, Object> cache = new HashMap<>();
    
    public void add(String key, Object value) {
        cache.put(key, value);  // Never removed!
    }
}

// ✅ GOOD: Use WeakHashMap or clear entries
public class Cache {
    private static Map<String, Object> cache = new WeakHashMap<>();
    // Or implement size limit and eviction policy
}
```

#### 2. Unclosed Resources

**Problem:** Resources not closed properly

```java
// ❌ BAD: Memory leak
public void readFile() {
    FileInputStream fis = new FileInputStream("file.txt");
    // Forgot to close!
}

// ✅ GOOD: Use try-with-resources
public void readFile() {
    try (FileInputStream fis = new FileInputStream("file.txt")) {
        // Auto-closed
    }
}
```

#### 3. Listener Registration

**Problem:** Listeners not unregistered

```java
// ❌ BAD: Memory leak
public class Component {
    public void addListener(Listener listener) {
        listeners.add(listener);  // Never removed!
    }
}

// ✅ GOOD: Unregister when done
public void removeListener(Listener listener) {
    listeners.remove(listener);
}
```

#### 4. Inner Classes Holding Outer Reference

**Problem:** Inner class holds reference to outer class

```java
// ❌ BAD: Memory leak
public class Outer {
    private byte[] largeData = new byte[1000000];
    
    public Inner createInner() {
        return new Inner();  // Holds reference to Outer
    }
    
    class Inner {
        // Implicitly holds reference to Outer
    }
}

// ✅ GOOD: Use static inner class
public class Outer {
    private byte[] largeData = new byte[1000000];
    
    public Inner createInner() {
        return new Inner();
    }
    
    static class Inner {
        // No reference to Outer
    }
}
```

#### 5. ThreadLocal Not Cleared

**Problem:** ThreadLocal values not removed

```java
// ❌ BAD: Memory leak
private static ThreadLocal<Object> threadLocal = new ThreadLocal<>();

public void setValue(Object value) {
    threadLocal.set(value);  // Never removed!
}

// ✅ GOOD: Remove when done
public void cleanup() {
    threadLocal.remove();
}
```

### Memory Leak Detection

#### 1. Heap Dump Analysis

**Generate Heap Dump:**
```bash
# Automatic on OutOfMemoryError
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/path/to/heapdump.hprof

# Manual dump
jmap -dump:format=b,file=heapdump.hprof <pid>
```

**Analyze with Tools:**
- **Eclipse MAT (Memory Analyzer Tool)**
- **VisualVM**
- **jhat** (Java Heap Analysis Tool)

#### 2. Memory Profiling

**VisualVM:**
```bash
# Start VisualVM
jvisualvm

# Connect to running application
# Monitor memory usage, take heap dumps
```

**JProfiler:**
- Commercial tool
- Real-time memory profiling
- Leak detection

#### 3. GC Log Analysis

**Identify Memory Leaks from GC Logs:**
```
# Healthy application
Before GC: 1GB, After GC: 500MB
Before GC: 1GB, After GC: 500MB  # Consistent

# Memory leak
Before GC: 1GB, After GC: 600MB
Before GC: 1.2GB, After GC: 700MB  # Growing
Before GC: 1.5GB, After GC: 900MB  # Still growing
```

### OutOfMemoryError Types

#### 1. Java Heap Space

**Cause:** Heap is full, cannot allocate more objects

**Solution:**
```bash
# Increase heap size
-Xmx4g

# Or fix memory leak
```

#### 2. Metaspace (Java 8+)

**Cause:** Too many classes loaded

**Solution:**
```bash
# Increase Metaspace
-XX:MaxMetaspaceSize=512m

# Or reduce class loading
```

#### 3. Direct Memory

**Cause:** Native memory exhausted (NIO buffers)

**Solution:**
```bash
# Limit direct memory
-XX:MaxDirectMemorySize=1g
```

#### 4. Unable to Create Native Thread

**Cause:** Too many threads created

**Solution:**
- Reduce thread creation
- Use thread pools
- Increase OS limits

---

## JVM Monitoring & Tools

### JVM Monitoring Tools

#### 1. jstat (JVM Statistics)

**Monitor GC:**
```bash
# GC statistics every 1 second
jstat -gc <pid> 1000

# Output:
 S0C    S1C    S0U    S1U      EC       EU        OC         OU       MC     MU    CCSC   CCSU   YGC     YGCT    FGC    FGCT     GCT
512.0  512.0   0.0   256.0   2048.0   1536.0    4096.0     1024.0   256.0  200.0  32.0   24.0      5     0.123     1     0.456   0.579
```

**Columns:**
- **S0C/S1C**: Survivor 0/1 capacity
- **EC/EU**: Eden capacity/used
- **OC/OU**: Old generation capacity/used
- **YGC/YGCT**: Young GC count/time
- **FGC/FGCT**: Full GC count/time

**Monitor Class Loading:**
```bash
jstat -class <pid> 1000
```

**Monitor Compilation:**
```bash
jstat -compiler <pid> 1000
```

#### 2. jmap (Memory Map)

**Heap Dump:**
```bash
# Generate heap dump
jmap -dump:format=b,file=heapdump.hprof <pid>

# Heap summary
jmap -heap <pid>

# Histogram of objects
jmap -histo <pid>
```

#### 3. jstack (Stack Trace)

**Thread Dump:**
```bash
# Generate thread dump
jstack <pid> > threaddump.txt

# Thread dump with locks
jstack -l <pid> > threaddump.txt
```

**Analyze Thread Dump:**
- Look for deadlocks
- Identify blocked threads
- Find CPU-intensive threads

#### 4. jinfo (Configuration Info)

**JVM Configuration:**
```bash
# Print all flags
jinfo -flags <pid>

# Print specific flag
jinfo -flag MaxHeapSize <pid>
```

#### 5. VisualVM

**Features:**
- Real-time monitoring
- Heap dump analysis
- Thread analysis
- CPU profiling
- Memory profiling

**Usage:**
```bash
jvisualvm
# Connect to local/remote JVM
```

#### 6. JConsole

**Features:**
- Memory monitoring
- Thread monitoring
- GC monitoring
- MBean browser

**Usage:**
```bash
jconsole
# Connect to JVM
```

### JVM Flags Reference

#### Memory Flags

```bash
# Heap
-Xms2g                    # Initial heap size
-Xmx4g                    # Maximum heap size
-Xmn1g                    # Young generation size

# Metaspace
-XX:MetaspaceSize=256m
-XX:MaxMetaspaceSize=512m

# Direct memory
-XX:MaxDirectMemorySize=1g
```

#### GC Flags

```bash
# GC Algorithm
-XX:+UseSerialGC
-XX:+UseParallelGC
-XX:+UseG1GC
-XX:+UseZGC

# GC Logging (Java 8)
-XX:+PrintGC
-XX:+PrintGCDetails
-Xloggc:/path/to/gc.log

# GC Logging (Java 9+)
-Xlog:gc*:file=/path/to/gc.log
```

#### Performance Flags

```bash
# JIT Compiler
-XX:+TieredCompilation
-XX:CompileThreshold=10000

# String deduplication
-XX:+UseStringDeduplication

# Compressed OOPs
-XX:+UseCompressedOops
```

---

## Advanced Topics

### JIT Compilation

**Just-In-Time (JIT) Compiler** compiles frequently executed bytecode to native code.

#### Compilation Levels

```
┌─────────────────────────────────────────┐
│      JIT COMPILATION LEVELS              │
└─────────────────────────────────────────┘

Level 0: Interpreter
   ├─→ Execute bytecode directly
   └─→ Slow but no compilation overhead
   │
   ▼
Level 1: C1 Compiler (Client)
   ├─→ Quick compilation
   ├─→ Basic optimizations
   └─→ Good for startup
   │
   ▼
Level 2: C1 Compiler (Optimized)
   ├─→ More optimizations
   └─→ Better performance
   │
   ▼
Level 3: C2 Compiler (Server)
   ├─→ Aggressive optimizations
   ├─→ Inlining, loop unrolling
   └─→ Best performance
```

#### HotSpot Detection

**Method call count threshold:**
- **-XX:CompileThreshold=10000** (default)
- Methods called >10000 times are compiled

**On-Stack Replacement (OSR):**
- Compile hot loops while executing
- Replace interpreted code with compiled code

#### JIT Optimizations

1. **Method Inlining**
   - Replace method calls with method body
   - Reduces call overhead

2. **Loop Unrolling**
   - Duplicate loop body
   - Reduces loop overhead

3. **Dead Code Elimination**
   - Remove unreachable code

4. **Constant Folding**
   - Evaluate constants at compile time

### String Intern Pool

**String intern pool** stores unique string literals.

```java
String s1 = "Hello";           // In pool
String s2 = "Hello";           // Reuses from pool
String s3 = new String("Hello"); // New object, not in pool
String s4 = s3.intern();       // Gets from pool

System.out.println(s1 == s2);  // true (same reference)
System.out.println(s1 == s3);  // false (different objects)
System.out.println(s1 == s4);  // true (interned)
```

**Memory Location:**
- **Java 7-**: PermGen
- **Java 8+**: Heap (part of old generation)

### Compressed OOPs

**Compressed Ordinary Object Pointers (OOPs)** reduce memory usage.

**How it works:**
- 64-bit JVM uses 8 bytes per reference
- Compressed OOPs use 4 bytes (for heaps <32GB)
- Saves significant memory

**Enable/Disable:**
```bash
-XX:+UseCompressedOops      # Enable (default)
-XX:-UseCompressedOops      # Disable
```

### TLAB (Thread-Local Allocation Buffer)

**TLAB** is a region of Eden space allocated per thread.

**Benefits:**
- Reduces contention (no locking needed)
- Faster allocation
- Better cache locality

**Configuration:**
```bash
-XX:+UseTLAB                # Enable (default)
-XX:TLABSize=256k           # TLAB size
```

### Card Table

**Card Table** tracks references from old to young generation.

**Purpose:**
- Optimize GC root scanning
- Only scan dirty cards during minor GC

**How it works:**
```
Old Generation divided into cards (512 bytes each)

When object in old gen references young gen:
  1. Mark corresponding card as dirty
  2. During minor GC, only scan dirty cards
  3. Reduces scanning time
```

---

## Interview Questions

### JVM Architecture Questions

#### Q1: Explain JVM architecture and its components.

**Answer:**
"JVM consists of three main components:

1. **Class Loader Subsystem**: Loads, links, and initializes classes. It follows a delegation model where child class loaders delegate to parent class loaders first.

2. **Runtime Data Areas**:
   - **Method Area (Metaspace)**: Stores class metadata, constants, static variables
   - **Heap**: Stores object instances (Young + Old generation)
   - **Java Stack**: Per-thread stack for method calls and local variables
   - **PC Register**: Current instruction pointer per thread
   - **Native Method Stack**: For native method calls

3. **Execution Engine**: 
   - **Interpreter**: Executes bytecode line by line
   - **JIT Compiler**: Compiles hot methods to native code (C1 and C2 compilers)
   - **Garbage Collector**: Manages memory automatically

The JVM also includes JNI for native code integration and native method libraries."

#### Q2: What is the difference between PermGen and Metaspace?

**Answer:**
"PermGen (Permanent Generation) was used in Java 7 and earlier:
- Fixed size (default 64MB)
- Part of heap memory
- Prone to OutOfMemoryError
- Stores class metadata, interned strings

Metaspace (Java 8+):
- Native memory (not part of heap)
- Auto-grows (no fixed limit by default)
- Less prone to OutOfMemoryError
- Stores only class metadata (strings moved to heap)
- Configurable with -XX:MetaspaceSize and -XX:MaxMetaspaceSize

The change was made to:
- Avoid PermGen OutOfMemoryError
- Allow dynamic class loading
- Better memory management"

#### Q3: Explain the class loading process.

**Answer:**
"Class loading happens in three phases:

1. **Loading**: 
   - Find .class file
   - Read binary data
   - Create Class object in method area

2. **Linking**:
   - **Verification**: Check file format, bytecode validity, symbol references
   - **Preparation**: Allocate memory for static variables, initialize with default values
   - **Resolution**: Replace symbolic references with direct references

3. **Initialization**:
   - Execute static initializers
   - Initialize static variables
   - Execute <clinit> method

The class loader follows delegation model: Application → Extension → Bootstrap. Child delegates to parent first, then tries to load itself."

### Memory Management Questions

#### Q4: Explain heap memory structure.

**Answer:**
"Heap is divided into two main areas:

1. **Young Generation**:
   - **Eden Space**: New objects allocated here (80% of young gen)
   - **Survivor Spaces (S0, S1)**: Objects that survive minor GC (10% each)
   - Objects promoted to old gen after surviving multiple GCs

2. **Old Generation (Tenured)**:
   - Long-lived objects
   - Collected during major/full GC
   - Larger than young generation

Heap size configured with -Xms (initial) and -Xmx (maximum). Young generation size with -Xmn or -XX:NewRatio."

#### Q5: What are GC roots? Give examples.

**Answer:**
"GC roots are objects always reachable and never garbage collected. Types include:

1. **Local variables** in active stack frames
2. **Static variables** in loaded classes
3. **Active threads** and their stack frames
4. **JNI references** (native code references)
5. **Synchronized objects** (monitors)

Any object reachable from GC roots is considered live. Objects not reachable from any GC root are eligible for garbage collection."

#### Q6: Explain object lifecycle in JVM.

**Answer:**
"Object lifecycle:

1. **Creation**: Object allocated in Eden space
2. **Minor GC**: 
   - If still referenced → moved to Survivor space (S0 or S1)
   - If not referenced → marked for deletion
3. **Promotion**: After surviving multiple minor GCs → promoted to Old Generation
4. **Major GC**: 
   - If still referenced → kept in Old Generation
   - If not referenced → memory reclaimed

Objects can also be directly allocated in Old Generation if they're large (via -XX:PretenureSizeThreshold)."

### Garbage Collection Questions

#### Q7: Explain different GC algorithms and when to use each.

**Answer:**
"Main GC algorithms:

1. **Serial GC**: 
   - Single-threaded, stop-the-world
   - Use: Small applications, single-core systems
   - Flag: -XX:+UseSerialGC

2. **Parallel GC**:
   - Multi-threaded, stop-the-world
   - Use: Batch processing, high throughput
   - Flag: -XX:+UseParallelGC

3. **CMS (Concurrent Mark Sweep)**:
   - Concurrent, low pause times
   - Use: Low-latency applications (deprecated in Java 14)
   - Flag: -XX:+UseConcMarkSweepGC

4. **G1 GC**:
   - Region-based, predictable pauses
   - Use: Large heaps (>4GB), low-latency
   - Flag: -XX:+UseG1GC

5. **ZGC**:
   - Ultra-low latency (<10ms)
   - Use: Very large heaps, ultra-low latency
   - Flag: -XX:+UseZGC

6. **Shenandoah**:
   - Concurrent evacuation
   - Use: Large heaps, low-latency
   - Flag: -XX:+UseShenandoahGC"

#### Q8: Explain G1 GC algorithm in detail.

**Answer:**
"G1 (Garbage First) GC:

**Key Features:**
- Divides heap into regions (1MB-32MB)
- Predictable pause times (configurable with -XX:MaxGCPauseMillis)
- Designed for large heaps (>4GB)

**Phases:**
1. **Young Generation Collection**: 
   - Stop-the-world
   - Evacuate (copy) live objects
   - Reclaim entire regions

2. **Concurrent Marking**:
   - Mark live objects concurrently
   - Identify regions for collection
   - Runs with application

3. **Mixed Collection**:
   - Collect young generation
   - Collect some old generation regions
   - Prioritizes regions with most garbage (garbage first)

**Configuration:**
- -XX:MaxGCPauseMillis=200 (target pause time)
- -XX:InitiatingHeapOccupancyPercent=45 (start concurrent marking at 45% full)"

#### Q9: What is stop-the-world (STW) pause?

**Answer:**
"Stop-the-world pause is when JVM stops all application threads to perform garbage collection.

**Why needed:**
- GC needs consistent view of object graph
- Objects can't be modified during GC
- Prevents race conditions

**Impact:**
- Application appears frozen
- Affects response time
- Measured in milliseconds

**Minimizing STW:**
- Use concurrent GC (G1, ZGC, Shenandoah)
- Tune GC parameters
- Right-size heap
- Minimize object creation"

#### Q10: Explain the difference between minor, major, and full GC.

**Answer:**
"**Minor GC (Young Generation Collection)**:
- Collects only young generation (Eden + Survivor)
- Frequent, fast (usually <100ms)
- Surviving objects promoted to old generation

**Major GC (Old Generation Collection)**:
- Collects old generation
- Less frequent, slower (can be seconds)
- Usually triggered when old generation is full

**Full GC**:
- Collects entire heap (young + old + metaspace)
- Least frequent, slowest
- Usually indicates memory pressure or misconfiguration

Note: Terminology varies. Some consider major GC same as full GC."

### Memory Leak Questions

#### Q11: How do you identify and fix memory leaks?

**Answer:**
"**Identification:**
1. Monitor heap usage (growing over time)
2. Analyze GC logs (heap not reducing after GC)
3. Generate heap dumps (jmap, -XX:+HeapDumpOnOutOfMemoryError)
4. Analyze with tools (Eclipse MAT, VisualVM)

**Common Causes:**
1. Static collections holding references
2. Unclosed resources (files, connections)
3. Listener registration without unregistration
4. Inner classes holding outer class references
5. ThreadLocal not cleared

**Fixes:**
1. Use WeakHashMap for caches
2. Use try-with-resources
3. Unregister listeners
4. Use static inner classes
5. Clear ThreadLocal values"

#### Q12: What is the difference between memory leak and memory overflow?

**Answer:**
"**Memory Leak:**
- Objects no longer needed but still referenced
- Heap gradually grows over time
- GC can't reclaim memory
- Eventually leads to OutOfMemoryError

**Memory Overflow:**
- Heap is full, cannot allocate more objects
- Can happen even without leaks (just too many objects)
- Immediate OutOfMemoryError
- Can be fixed by increasing heap size

**Example:**
- Memory leak: Cache growing indefinitely
- Memory overflow: Processing large dataset with small heap"

### Advanced Questions

#### Q13: Explain JIT compilation and optimization techniques.

**Answer:**
"JIT (Just-In-Time) compiler compiles frequently executed bytecode to native code.

**Compilation Levels:**
- Level 0: Interpreter (bytecode execution)
- Level 1-2: C1 compiler (quick compilation, basic optimizations)
- Level 3: C2 compiler (aggressive optimizations)

**HotSpot Detection:**
- Methods called >10000 times (default) are compiled
- Configurable with -XX:CompileThreshold

**Optimizations:**
1. **Method Inlining**: Replace calls with method body
2. **Loop Unrolling**: Duplicate loop body
3. **Dead Code Elimination**: Remove unreachable code
4. **Constant Folding**: Evaluate constants at compile time
5. **Escape Analysis**: Allocate objects on stack if they don't escape

**Benefits:**
- Faster execution than interpreter
- Adapts to runtime behavior
- Better than AOT (Ahead-of-Time) for long-running applications"

#### Q14: What is TLAB and why is it important?

**Answer:**
"TLAB (Thread-Local Allocation Buffer) is a region of Eden space allocated per thread.

**Benefits:**
1. **Reduces Contention**: No locking needed for allocation
2. **Faster Allocation**: Pointer bump allocation
3. **Better Cache Locality**: Thread-local memory access

**How it works:**
- Each thread gets its own TLAB in Eden
- Objects allocated in TLAB without synchronization
- When TLAB full, allocate new one or use shared Eden

**Configuration:**
- -XX:+UseTLAB (enabled by default)
- -XX:TLABSize=256k (TLAB size)

**Impact:**
- Significant performance improvement for multi-threaded applications
- Reduces GC pressure"

#### Q15: Explain compressed OOPs.

**Answer:**
"Compressed OOPs (Ordinary Object Pointers) reduce memory usage for object references.

**Problem:**
- 64-bit JVM uses 8 bytes per reference
- Wastes memory for heaps <32GB

**Solution:**
- Compressed OOPs use 4 bytes per reference
- Automatically enabled for heaps <32GB
- Saves significant memory (can be 20-30%)

**How it works:**
- References are shifted left by 3 bits (multiply by 8)
- Base address added during dereference
- Transparent to application code

**Configuration:**
- -XX:+UseCompressedOops (default, auto-enabled)
- -XX:-UseCompressedOops (disable)

**Limitation:**
- Only works for heaps <32GB
- Larger heaps must use 8-byte references"

#### Q16: How do you tune GC for low-latency applications?

**Answer:**
"**GC Algorithm:**
- Use G1, ZGC, or Shenandoah (concurrent GC)
- Avoid Serial/Parallel GC (high pause times)

**Heap Configuration:**
- Right-size heap (not too large, not too small)
- Set -Xms = -Xmx (avoid resizing)
- Tune young generation size

**G1 GC Tuning:**
```bash
-XX:+UseG1GC
-XX:MaxGCPauseMillis=200        # Target pause time
-XX:InitiatingHeapOccupancyPercent=45
-XX:G1HeapRegionSize=16m
```

**Application Code:**
- Minimize object creation
- Reuse objects where possible
- Avoid large object allocation
- Clear references promptly

**Monitoring:**
- GC logs with pause time analysis
- Real-time monitoring (VisualVM, JProfiler)
- Load testing to validate"

#### Q17: Explain card table and its role in GC.

**Answer:**
"Card Table optimizes GC root scanning.

**Problem:**
- During minor GC, need to scan old generation for references to young generation
- Scanning entire old generation is expensive

**Solution:**
- Divide old generation into cards (512 bytes each)
- When object in old gen references young gen, mark card as dirty
- During minor GC, only scan dirty cards

**How it works:**
1. Write barrier marks card when old→young reference created
2. Minor GC scans only dirty cards
3. Clear dirty marks after scanning

**Benefits:**
- Reduces scanning time significantly
- Makes minor GC faster
- Enables concurrent GC algorithms"

#### Q18: What are the different types of OutOfMemoryError?

**Answer:**
"**1. Java Heap Space:**
- Heap is full
- Fix: Increase -Xmx or fix memory leak

**2. Metaspace (Java 8+):**
- Too many classes loaded
- Fix: Increase -XX:MaxMetaspaceSize or reduce class loading

**3. Direct Memory:**
- Native memory exhausted (NIO buffers)
- Fix: Increase -XX:MaxDirectMemorySize or reduce direct buffer usage

**4. Unable to Create Native Thread:**
- Too many threads
- Fix: Reduce thread creation, use thread pools, increase OS limits

**5. Requested Array Size Exceeds VM Limit:**
- Array too large
- Fix: Reduce array size or increase heap"

#### Q19: How do you analyze GC logs?

**Answer:**
"**GC Log Format (Java 9+):**
```
[0.123s][info][gc] Using G1
[1.456s][info][gc] GC(0) Pause Young (Normal) 512M->256M (2G) 45.123ms
```

**Key Metrics:**
- **Pause Time**: How long application stopped (target: <200ms)
- **Throughput**: % time in GC (target: <5%)
- **Memory**: Before->After (Total) - should reduce after GC
- **Frequency**: How often GC runs

**Red Flags:**
- Growing heap after GC (memory leak)
- High pause times (need tuning)
- Frequent full GC (heap too small)
- Low throughput (GC overhead too high)

**Tools:**
- GCViewer (visualize GC logs)
- GCPlot (online analysis)
- Custom scripts for parsing"

#### Q20: Explain the difference between G1, ZGC, and Shenandoah.

**Answer:**
"**G1 GC:**
- Region-based, incremental
- Pause times: <200ms (configurable)
- Heap size: >4GB recommended
- Mature, production-ready
- Concurrent marking, stop-the-world evacuation

**ZGC:**
- Ultra-low latency
- Pause times: <10ms (sub-millisecond target)
- Heap size: Multi-terabyte
- Concurrent mark and relocate
- Colored pointers, load barriers
- Experimental in Java 11-14, production in Java 15+

**Shenandoah:**
- Low latency
- Pause times: <10ms
- Heap size: Large
- Concurrent evacuation
- Similar to ZGC but different implementation
- Experimental in Java 12-14, production in Java 15+

**When to use:**
- G1: Most applications, large heaps, low-latency needs
- ZGC: Ultra-low latency requirements, very large heaps
- Shenandoah: Low latency, large heaps (alternative to ZGC)"

---

## Quick Reference

### Essential JVM Flags

```bash
# Heap
-Xms2g -Xmx4g                    # Heap size
-Xmn1g                           # Young generation

# GC
-XX:+UseG1GC                     # G1 GC
-XX:MaxGCPauseMillis=200         # Target pause

# Memory
-XX:MetaspaceSize=256m           # Metaspace
-XX:MaxMetaspaceSize=512m

# Logging
-Xlog:gc*:file=gc.log            # GC logs (Java 9+)

# Dumps
-XX:+HeapDumpOnOutOfMemoryError  # Auto heap dump
-XX:HeapDumpPath=/path/to/dump
```

### GC Algorithm Selection

```
Small heap (<4GB) → Parallel GC
Large heap (>4GB) → G1 GC
Ultra-low latency → ZGC or Shenandoah
High throughput → Parallel GC
```

### Memory Leak Checklist

- [ ] Static collections cleared?
- [ ] Resources closed (try-with-resources)?
- [ ] Listeners unregistered?
- [ ] ThreadLocal cleared?
- [ ] Inner classes static?
- [ ] Weak references for caches?

---

## Summary

**Key Takeaways:**

1. **JVM Architecture**: Class Loader → Runtime Data Areas → Execution Engine
2. **Memory Areas**: Method Area, Heap, Stack, PC Register, Native Stack
3. **Heap Structure**: Young Generation (Eden, Survivor) + Old Generation
4. **GC Algorithms**: Serial, Parallel, CMS, G1, ZGC, Shenandoah
5. **GC Tuning**: Right-size heap, choose appropriate GC, monitor and iterate
6. **Memory Leaks**: Identify with heap dumps, fix common patterns
7. **Monitoring**: jstat, jmap, jstack, VisualVM, GC logs
8. **Advanced**: JIT compilation, TLAB, Compressed OOPs, Card Table

**Best Practices:**

- Measure before tuning
- Use appropriate GC for use case
- Right-size heap (not too large, not too small)
- Monitor GC metrics regularly
- Fix memory leaks promptly
- Use modern GC (G1, ZGC) for production

---

**Happy Learning! 🚀**


