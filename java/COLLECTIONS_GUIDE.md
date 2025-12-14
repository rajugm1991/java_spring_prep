# Java Collections Framework - Complete Guide 🗂️

> Comprehensive guide covering Java Collections from basics to advanced topics, design patterns, thread safety, and interview preparation

---

## Table of Contents
1. [Overview of Collections Framework](#overview-of-collections-framework)
2. [Core Interfaces and Abstract Classes](#core-interfaces-and-abstract-classes)
3. [When to Use Abstract Class vs Interface](#when-to-use-abstract-class-vs-interface)
4. [List Implementations](#list-implementations)
5. [Set Implementations](#set-implementations)
6. [Map Implementations](#map-implementations)
7. [Queue Implementations](#queue-implementations)
8. [Design Patterns with Collections](#design-patterns-with-collections)
9. [Thread Safety and Concurrent Collections](#thread-safety-and-concurrent-collections)
10. [Advanced Topics](#advanced-topics)
11. [Best Practices](#best-practices)
12. [Performance Considerations](#performance-considerations)
13. [Interview Questions](#interview-questions)

---

## Overview of Collections Framework

### What is Collections Framework?

The **Java Collections Framework** provides a unified architecture for representing and manipulating collections of objects.

### Collection Hierarchy

```
┌─────────────────────────────────────────────────────────┐
│              COLLECTIONS FRAMEWORK                      │
└─────────────────────────────────────────────────────────┘

                    Collection
                    (Interface)
                         │
        ┌────────────────┼────────────────┐
        │                │                │
       List            Set              Queue
   (Interface)      (Interface)       (Interface)
        │                │                │
    ┌───┴───┐        ┌───┴───┐        ┌───┴───┐
    │       │        │       │        │       │
ArrayList LinkedList HashSet TreeSet PriorityQueue Deque
```

### Key Interfaces

1. **Collection** - Root interface
2. **List** - Ordered, allows duplicates
3. **Set** - No duplicates
4. **Map** - Key-value pairs (not part of Collection)
5. **Queue** - FIFO/LIFO operations
6. **Deque** - Double-ended queue

---

## Core Interfaces and Abstract Classes

### Collection Interface

**Root interface** for all collection types (except Map)

```java
public interface Collection<E> extends Iterable<E> {
    // Basic operations
    int size();
    boolean isEmpty();
    boolean contains(Object o);
    boolean add(E e);
    boolean remove(Object o);
    Iterator<E> iterator();
    
    // Bulk operations
    boolean containsAll(Collection<?> c);
    boolean addAll(Collection<? extends E> c);
    boolean removeAll(Collection<?> c);
    boolean retainAll(Collection<?> c);
    void clear();
    
    // Array operations
    Object[] toArray();
    <T> T[] toArray(T[] a);
}
```

### AbstractCollection Class

**Abstract implementation** of Collection interface

**Purpose:**
- Provides skeletal implementation
- Reduces effort to implement Collection
- Implements most methods, leaves few for subclasses

```java
public abstract class AbstractCollection<E> 
    implements Collection<E> {
    
    // Concrete implementations
    public boolean isEmpty() {
        return size() == 0;
    }
    
    public boolean contains(Object o) {
        Iterator<E> it = iterator();
        if (o == null) {
            while (it.hasNext()) {
                if (it.next() == null) return true;
            }
        } else {
            while (it.hasNext()) {
                if (o.equals(it.next())) return true;
            }
        }
        return false;
    }
    
    // Abstract methods (must implement)
    public abstract Iterator<E> iterator();
    public abstract int size();
}
```

**When to Extend:**
- Creating custom collection types
- Need default implementations
- Want to minimize code

**Example:**
```java
public class CustomCollection<E> extends AbstractCollection<E> {
    private List<E> elements = new ArrayList<>();
    
    @Override
    public Iterator<E> iterator() {
        return elements.iterator();
    }
    
    @Override
    public int size() {
        return elements.size();
    }
    
    @Override
    public boolean add(E e) {
        return elements.add(e);
    }
}
```

---

## When to Use Abstract Class vs Interface

### Abstract Class

**Use Abstract Class When:**

1. **Shared Implementation**
   - Multiple classes share common code
   - Want to provide default implementations
   - Need to reduce code duplication

```java
// Abstract class with shared implementation
public abstract class AbstractList<E> implements List<E> {
    protected int modCount = 0;  // Shared state
    
    // Shared implementation
    public boolean isEmpty() {
        return size() == 0;
    }
    
    // Abstract methods
    public abstract E get(int index);
    public abstract int size();
}

// Concrete implementations
public class ArrayList<E> extends AbstractList<E> {
    // Uses isEmpty() from AbstractList
    // Implements get() and size()
}

public class Vector<E> extends AbstractList<E> {
    // Uses isEmpty() from AbstractList
    // Implements get() and size()
}
```

2. **State Management**
   - Need instance variables
   - Need to maintain state across methods
   - Need constructors

```java
public abstract class AbstractMap<K, V> implements Map<K, V> {
    // Shared state
    transient Set<K> keySet;
    transient Collection<V> values;
    
    // Shared implementation
    public boolean isEmpty() {
        return size() == 0;
    }
    
    // Abstract methods
    public abstract Set<Entry<K, V>> entrySet();
}
```

3. **Template Method Pattern**
   - Define algorithm skeleton
   - Let subclasses implement specific steps

```java
public abstract class AbstractList<E> {
    // Template method
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);
        boolean modified = false;
        for (E e : c) {
            add(index++, e);  // Calls abstract add()
            modified = true;
        }
        return modified;
    }
    
    // Abstract method (subclass implements)
    public abstract void add(int index, E element);
}
```

4. **Access Modifiers**
   - Need protected methods
   - Need package-private methods
   - Need to control inheritance

```java
public abstract class AbstractCollection<E> {
    // Protected method for subclasses
    protected AbstractCollection() {
    }
    
    // Package-private helper
    String toString(Collection<?> c) {
        // Implementation
    }
}
```

### Interface

**Use Interface When:**

1. **Multiple Inheritance**
   - Class needs to implement multiple interfaces
   - Want to define contracts without implementation

```java
// Multiple interfaces
public class ArrayList<E> extends AbstractList<E> 
    implements List<E>, RandomAccess, Cloneable, Serializable {
    // Can implement multiple interfaces
}
```

2. **API Contracts**
   - Define behavior contract
   - Don't need shared implementation
   - Want loose coupling

```java
// Interface defines contract
public interface List<E> extends Collection<E> {
    // Contract methods
    E get(int index);
    E set(int index, E element);
    void add(int index, E element);
    E remove(int index);
}

// Multiple implementations
public class ArrayList<E> implements List<E> { }
public class LinkedList<E> implements List<E> { }
public class Vector<E> implements List<E> { }
```

3. **Default Methods (Java 8+)**
   - Want to add methods to existing interfaces
   - Provide default implementation
   - Backward compatibility

```java
public interface List<E> extends Collection<E> {
    // Default implementation
    default void sort(Comparator<? super E> c) {
        Collections.sort(this, c);
    }
    
    // Default stream support
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
}
```

4. **Functional Interfaces**
   - Single abstract method
   - Lambda expressions
   - Method references

```java
@FunctionalInterface
public interface Comparator<T> {
    int compare(T o1, T o2);
    
    // Default methods
    default Comparator<T> reversed() {
        return Collections.reverseOrder(this);
    }
}
```

### Decision Matrix

| Scenario | Use Abstract Class | Use Interface |
|----------|-------------------|---------------|
| **Shared Implementation** | ✅ Yes | ❌ No |
| **State/Fields** | ✅ Yes | ❌ No (Java 8+ can have constants) |
| **Multiple Inheritance** | ❌ No | ✅ Yes |
| **Template Method** | ✅ Yes | ❌ No |
| **API Contract** | ❌ No | ✅ Yes |
| **Default Methods** | ❌ No | ✅ Yes (Java 8+) |
| **Constructors** | ✅ Yes | ❌ No |
| **Access Modifiers** | ✅ Yes (protected) | ❌ No (all public) |

### Real-World Example: Collections Framework

```java
// Abstract class for shared implementation
public abstract class AbstractList<E> implements List<E> {
    protected int modCount = 0;  // Shared state
    
    // Shared implementations
    public boolean isEmpty() { return size() == 0; }
    public boolean add(E e) { add(size(), e); return true; }
    
    // Abstract methods
    public abstract E get(int index);
    public abstract int size();
}

// Interface for contract
public interface List<E> extends Collection<E> {
    // Contract methods
    E get(int index);
    E set(int index, E element);
    void add(int index, E element);
}

// Concrete implementation
public class ArrayList<E> extends AbstractList<E> 
    implements List<E>, RandomAccess {
    // Gets isEmpty(), add() from AbstractList
    // Implements get(), size() from AbstractList
    // Implements List contract
}
```

---

## List Implementations

### ArrayList

**Characteristics:**
- Dynamic array
- Random access (O(1))
- Insertion/Deletion (O(n))
- Not thread-safe

```java
List<String> list = new ArrayList<>();
list.add("Apple");
list.add("Banana");
list.get(0);  // O(1) random access
```

**When to Use:**
- Frequent random access
- Iteration
- Small to medium size
- Single-threaded

**Internal Structure:**
```java
public class ArrayList<E> {
    private static final int DEFAULT_CAPACITY = 10;
    private Object[] elementData;
    private int size;
    
    // Grows by 50% when full
    private void grow(int minCapacity) {
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);  // 1.5x
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
}
```

### LinkedList

**Characteristics:**
- Doubly linked list
- Sequential access
- Insertion/Deletion (O(1) at known position)
- Not thread-safe

```java
List<String> list = new LinkedList<>();
list.add("Apple");
list.addFirst("Banana");  // O(1)
list.removeLast();        // O(1)
```

**When to Use:**
- Frequent insertion/deletion
- Queue/Deque operations
- No random access needed

**Internal Structure:**
```java
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
}
```

### Vector

**Characteristics:**
- Similar to ArrayList
- Thread-safe (synchronized)
- Slower than ArrayList
- Legacy class

```java
Vector<String> vector = new Vector<>();
vector.add("Apple");  // Synchronized
```

**When to Use:**
- Legacy code
- Need thread-safety (better use CopyOnWriteArrayList)

### Comparison

| Feature | ArrayList | LinkedList | Vector |
|---------|-----------|------------|--------|
| **Random Access** | O(1) | O(n) | O(1) |
| **Insert at End** | O(1) amortized | O(1) | O(1) |
| **Insert at Middle** | O(n) | O(1) if position known | O(n) |
| **Delete** | O(n) | O(1) if position known | O(n) |
| **Thread-Safe** | No | No | Yes |
| **Memory** | Less (contiguous) | More (pointers) | Less |

---

## Set Implementations

### HashSet

**Characteristics:**
- Hash table implementation
- No order
- O(1) average operations
- Not thread-safe

```java
Set<String> set = new HashSet<>();
set.add("Apple");
set.add("Banana");
set.contains("Apple");  // O(1) average
```

**When to Use:**
- Fast lookups
- No duplicates
- Order doesn't matter

### LinkedHashSet

**Characteristics:**
- Hash table + Linked list
- Maintains insertion order
- O(1) average operations
- Not thread-safe

```java
Set<String> set = new LinkedHashSet<>();
set.add("Apple");
set.add("Banana");
// Iteration order: Apple, Banana
```

**When to Use:**
- Need insertion order
- Fast lookups
- No duplicates

### TreeSet

**Characteristics:**
- Red-Black tree
- Sorted order
- O(log n) operations
- Not thread-safe

```java
Set<String> set = new TreeSet<>();
set.add("Banana");
set.add("Apple");
// Sorted: Apple, Banana
```

**When to Use:**
- Need sorted order
- Range queries
- Can accept O(log n)

### Comparison

| Feature | HashSet | LinkedHashSet | TreeSet |
|---------|---------|---------------|---------|
| **Order** | No | Insertion | Sorted |
| **Add** | O(1) avg | O(1) avg | O(log n) |
| **Contains** | O(1) avg | O(1) avg | O(log n) |
| **Memory** | Less | More | More |
| **Null** | One null | One null | No null |

---

## Map Implementations

### HashMap

**Characteristics:**
- Hash table
- No order
- O(1) average operations
- Not thread-safe
- Allows one null key

```java
Map<String, Integer> map = new HashMap<>();
map.put("Apple", 1);
map.get("Apple");  // O(1) average
```

**Internal Structure:**
```java
// Java 8+: Array of buckets (bins)
// Each bucket: Tree (if > 8 nodes) or LinkedList
static class Node<K, V> implements Map.Entry<K, V> {
    final int hash;
    final K key;
    V value;
    Node<K, V> next;
}
```

### LinkedHashMap

**Characteristics:**
- Hash table + Linked list
- Maintains insertion/access order
- O(1) average operations
- Not thread-safe

```java
Map<String, Integer> map = new LinkedHashMap<>();
map.put("Apple", 1);
map.put("Banana", 2);
// Iteration order: Apple, Banana
```

**When to Use:**
- Need insertion order
- LRU cache implementation

### TreeMap

**Characteristics:**
- Red-Black tree
- Sorted by keys
- O(log n) operations
- Not thread-safe

```java
Map<String, Integer> map = new TreeMap<>();
map.put("Banana", 2);
map.put("Apple", 1);
// Sorted: Apple=1, Banana=2
```

**When to Use:**
- Need sorted keys
- Range queries
- Navigable operations

### Hashtable

**Characteristics:**
- Legacy class
- Thread-safe (synchronized)
- No null keys/values
- Slower than HashMap

```java
Hashtable<String, Integer> table = new Hashtable<>();
table.put("Apple", 1);  // Synchronized
```

**When to Use:**
- Legacy code
- Need thread-safety (better use ConcurrentHashMap)

### Comparison

| Feature | HashMap | LinkedHashMap | TreeMap | Hashtable |
|---------|---------|---------------|---------|-----------|
| **Order** | No | Insertion | Sorted | No |
| **Get/Put** | O(1) avg | O(1) avg | O(log n) | O(1) avg |
| **Null Key** | One | One | No | No |
| **Thread-Safe** | No | No | No | Yes |
| **Legacy** | No | No | No | Yes |

---

## Queue Implementations

### PriorityQueue

**Characteristics:**
- Min/Max heap
- Priority-based ordering
- O(log n) insert/remove
- Not thread-safe

```java
Queue<Integer> pq = new PriorityQueue<>();
pq.offer(5);
pq.offer(1);
pq.offer(3);
pq.poll();  // Returns 1 (smallest)
```

**When to Use:**
- Priority scheduling
- Top K problems
- Dijkstra's algorithm

### ArrayDeque

**Characteristics:**
- Resizable array
- Double-ended queue
- O(1) operations
- Not thread-safe
- Faster than Stack/LinkedList

```java
Deque<String> deque = new ArrayDeque<>();
deque.offerFirst("First");
deque.offerLast("Last");
deque.pollFirst();  // O(1)
```

**When to Use:**
- Stack operations
- Queue operations
- Sliding window

### BlockingQueue Implementations

**ArrayBlockingQueue:**
- Bounded queue
- Thread-safe
- Array-backed

**LinkedBlockingQueue:**
- Unbounded (or bounded)
- Thread-safe
- Linked list-backed

---

## Design Patterns with Collections

### 1. Iterator Pattern

**Purpose:** Provide a way to access elements sequentially without exposing internal structure

```java
// Iterator interface
public interface Iterator<E> {
    boolean hasNext();
    E next();
    void remove();
}

// Usage
List<String> list = Arrays.asList("A", "B", "C");
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}

// Enhanced for loop (uses iterator internally)
for (String item : list) {
    System.out.println(item);
}
```

**Benefits:**
- Uniform interface for all collections
- Hides implementation details
- Supports multiple traversals

### 2. Factory Pattern

**Purpose:** Create collections without specifying exact implementation

```java
// Factory methods (Java 9+)
List<String> list = List.of("A", "B", "C");
Set<Integer> set = Set.of(1, 2, 3);
Map<String, Integer> map = Map.of("A", 1, "B", 2);

// Collections utility
List<String> synchronizedList = Collections.synchronizedList(new ArrayList<>());
List<String> unmodifiableList = Collections.unmodifiableList(list);
```

**Benefits:**
- Encapsulate creation logic
- Return appropriate implementation
- Easy to change implementation

### 3. Adapter Pattern

**Purpose:** Adapt one interface to another

```java
// Array to List adapter
String[] array = {"A", "B", "C"};
List<String> list = Arrays.asList(array);  // Fixed-size list

// List to Queue adapter
Queue<String> queue = new LinkedList<>(list);
```

### 4. Decorator Pattern

**Purpose:** Add functionality to collections dynamically

```java
// Unmodifiable decorator
List<String> original = new ArrayList<>();
List<String> unmodifiable = Collections.unmodifiableList(original);

// Synchronized decorator
List<String> syncList = Collections.synchronizedList(original);

// Custom decorator
public class LoggingList<E> extends AbstractList<E> {
    private final List<E> delegate;
    
    public LoggingList(List<E> delegate) {
        this.delegate = delegate;
    }
    
    @Override
    public boolean add(E e) {
        System.out.println("Adding: " + e);
        return delegate.add(e);
    }
    
    @Override
    public E get(int index) {
        return delegate.get(index);
    }
    
    @Override
    public int size() {
        return delegate.size();
    }
}
```

### 5. Strategy Pattern

**Purpose:** Define family of algorithms (comparators)

```java
// Different sorting strategies
List<Person> people = new ArrayList<>();

// Sort by name
people.sort(Comparator.comparing(Person::getName));

// Sort by age
people.sort(Comparator.comparingInt(Person::getAge));

// Sort by multiple fields
people.sort(Comparator
    .comparing(Person::getAge)
    .thenComparing(Person::getName));
```

### 6. Template Method Pattern

**Purpose:** Define algorithm skeleton in abstract class

```java
public abstract class AbstractList<E> {
    // Template method
    public boolean addAll(Collection<? extends E> c) {
        boolean modified = false;
        for (E e : c) {
            if (add(e)) {  // Calls abstract add()
                modified = true;
            }
        }
        return modified;
    }
    
    // Abstract method (subclass implements)
    public abstract boolean add(E e);
}
```

---

## Thread Safety and Concurrent Collections

### Thread Safety Issues

**Problem with Non-Thread-Safe Collections:**

```java
// ❌ NOT THREAD-SAFE
List<String> list = new ArrayList<>();

// Thread 1
list.add("A");

// Thread 2
list.add("B");

// Result: May throw ConcurrentModificationException
// or lose data
```

### Solutions

#### 1. Synchronized Collections

**Collections.synchronizedXXX()**

```java
// Synchronized wrapper
List<String> syncList = Collections.synchronizedList(new ArrayList<>());

// All operations are synchronized
syncList.add("A");  // Thread-safe
syncList.get(0);    // Thread-safe

// But iteration still needs synchronization
synchronized(syncList) {
    for (String item : syncList) {
        // Safe iteration
    }
}
```

**Limitations:**
- Coarse-grained locking (entire collection)
- Poor performance under contention
- Iteration needs external synchronization

#### 2. Concurrent Collections (Java 5+)

**Better performance with fine-grained locking**

##### ConcurrentHashMap

**Characteristics:**
- Thread-safe HashMap
- Fine-grained locking (segment-based or CAS)
- Better performance than synchronized HashMap
- No locking for reads

```java
Map<String, Integer> map = new ConcurrentHashMap<>();
map.put("Apple", 1);  // Thread-safe, no locking
map.get("Apple");     // Thread-safe, no locking

// Atomic operations
map.compute("Apple", (k, v) -> v == null ? 1 : v + 1);
map.merge("Apple", 1, Integer::sum);
```

**Internal Structure (Java 8+):**
```java
// Array of buckets
// Each bucket: Tree or LinkedList
// Lock-free reads
// Fine-grained locking for writes
```

##### CopyOnWriteArrayList

**Characteristics:**
- Thread-safe ArrayList
- Copy-on-write strategy
- Fast reads, slow writes
- No locking for reads

```java
List<String> list = new CopyOnWriteArrayList<>();
list.add("A");  // Creates copy, then writes

// Iteration uses snapshot (no ConcurrentModificationException)
for (String item : list) {
    // Safe iteration, even if list modified
}
```

**When to Use:**
- Read-heavy workloads
- Infrequent writes
- Iteration over modification

##### ConcurrentLinkedQueue

**Characteristics:**
- Lock-free queue
- CAS-based operations
- High concurrency
- No size() method (expensive)

```java
Queue<String> queue = new ConcurrentLinkedQueue<>();
queue.offer("A");  // Lock-free
queue.poll();      // Lock-free
```

##### BlockingQueue Implementations

**ArrayBlockingQueue:**
```java
BlockingQueue<String> queue = new ArrayBlockingQueue<>(10);
queue.put("A");     // Blocks if full
queue.take();        // Blocks if empty
```

**LinkedBlockingQueue:**
```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>();
queue.put("A");
queue.take();
```

**PriorityBlockingQueue:**
```java
BlockingQueue<Integer> pq = new PriorityBlockingQueue<>();
pq.put(5);
pq.take();  // Returns smallest
```

### Thread Safety Comparison

| Collection | Thread-Safe | Locking | Performance |
|------------|-------------|---------|-------------|
| **ArrayList** | ❌ No | N/A | Fast |
| **Vector** | ✅ Yes | Coarse | Slow |
| **Collections.synchronizedList** | ✅ Yes | Coarse | Slow |
| **CopyOnWriteArrayList** | ✅ Yes | Copy-on-write | Fast reads |
| **HashMap** | ❌ No | N/A | Fast |
| **Hashtable** | ✅ Yes | Coarse | Slow |
| **ConcurrentHashMap** | ✅ Yes | Fine-grained | Fast |
| **ConcurrentLinkedQueue** | ✅ Yes | Lock-free | Very fast |

### Best Practices for Thread Safety

**1. Use Concurrent Collections:**
```java
// ✅ GOOD
Map<String, Integer> map = new ConcurrentHashMap<>();
List<String> list = new CopyOnWriteArrayList<>();

// ❌ BAD
Map<String, Integer> map = Collections.synchronizedMap(new HashMap<>());
```

**2. Atomic Operations:**
```java
// ✅ GOOD - Atomic
map.compute("key", (k, v) -> v == null ? 1 : v + 1);

// ❌ BAD - Not atomic
if (map.containsKey("key")) {
    map.put("key", map.get("key") + 1);  // Race condition!
}
```

**3. Immutable Collections:**
```java
// ✅ GOOD - Thread-safe by design
List<String> immutable = List.of("A", "B", "C");
Set<Integer> immutableSet = Set.of(1, 2, 3);
```

**4. Proper Iteration:**
```java
// ✅ GOOD - CopyOnWriteArrayList
for (String item : copyOnWriteList) {
    // Safe
}

// ✅ GOOD - Synchronized with lock
synchronized(syncList) {
    for (String item : syncList) {
        // Safe
    }
}

// ❌ BAD - No synchronization
for (String item : syncList) {
    // May throw ConcurrentModificationException
}
```

---

## Advanced Topics

### 1. Custom Collections

**Creating Custom Collection:**

```java
public class CircularQueue<E> extends AbstractQueue<E> {
    private final Object[] elements;
    private int head = 0;
    private int tail = 0;
    private int size = 0;
    
    public CircularQueue(int capacity) {
        this.elements = new Object[capacity];
    }
    
    @Override
    public boolean offer(E e) {
        if (size == elements.length) return false;
        elements[tail] = e;
        tail = (tail + 1) % elements.length;
        size++;
        return true;
    }
    
    @Override
    public E poll() {
        if (size == 0) return null;
        @SuppressWarnings("unchecked")
        E element = (E) elements[head];
        elements[head] = null;
        head = (head + 1) % elements.length;
        size--;
        return element;
    }
    
    @Override
    public E peek() {
        return size == 0 ? null : (E) elements[head];
    }
    
    @Override
    public Iterator<E> iterator() {
        return new Iterator<E>() {
            private int index = 0;
            
            @Override
            public boolean hasNext() {
                return index < size;
            }
            
            @Override
            public E next() {
                if (!hasNext()) throw new NoSuchElementException();
                @SuppressWarnings("unchecked")
                E element = (E) elements[(head + index) % elements.length];
                index++;
                return element;
            }
        };
    }
    
    @Override
    public int size() {
        return size;
    }
}
```

### 2. Stream API Integration

**Collections with Streams:**

```java
List<String> list = Arrays.asList("apple", "banana", "cherry");

// Filter and collect
List<String> filtered = list.stream()
    .filter(s -> s.startsWith("a"))
    .collect(Collectors.toList());

// Map and collect
List<Integer> lengths = list.stream()
    .map(String::length)
    .collect(Collectors.toList());

// Group by
Map<Integer, List<String>> grouped = list.stream()
    .collect(Collectors.groupingBy(String::length));

// Partition
Map<Boolean, List<String>> partitioned = list.stream()
    .collect(Collectors.partitioningBy(s -> s.length() > 5));
```

### 3. Comparators and Sorting

**Advanced Sorting:**

```java
List<Person> people = new ArrayList<>();

// Simple comparator
people.sort(Comparator.comparing(Person::getName));

// Reversed
people.sort(Comparator.comparing(Person::getName).reversed());

// Multiple fields
people.sort(Comparator
    .comparing(Person::getAge)
    .thenComparing(Person::getName)
    .thenComparing(Person::getCity));

// Custom comparator
people.sort((p1, p2) -> {
    int ageCompare = Integer.compare(p1.getAge(), p2.getAge());
    if (ageCompare != 0) return ageCompare;
    return p1.getName().compareTo(p2.getName());
});

// Null handling
people.sort(Comparator
    .comparing(Person::getName, Comparator.nullsLast(String::compareTo)));
```

### 4. Fail-Fast vs Fail-Safe Iterators

**Fail-Fast (ArrayList, HashMap):**
```java
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
Iterator<String> it = list.iterator();

list.add("D");  // Modifies list
it.next();      // Throws ConcurrentModificationException
```

**Fail-Safe (CopyOnWriteArrayList, ConcurrentHashMap):**
```java
List<String> list = new CopyOnWriteArrayList<>(Arrays.asList("A", "B", "C"));
Iterator<String> it = list.iterator();

list.add("D");  // Creates new copy
it.next();      // Uses old snapshot, no exception
```

### 5. Performance Optimization

**Initial Capacity:**
```java
// ✅ GOOD - Prevents resizing
List<String> list = new ArrayList<>(1000);
Map<String, Integer> map = new HashMap<>(16, 0.75f);

// ❌ BAD - Multiple resizes
List<String> list = new ArrayList<>();  // Default 10, resizes multiple times
```

**Load Factor (HashMap):**
```java
// Default: 0.75 (75% full before resize)
// Lower: More memory, fewer collisions
// Higher: Less memory, more collisions

Map<String, Integer> map = new HashMap<>(16, 0.5f);  // Resize at 50%
```

---

## Best Practices

### 1. Choose Right Collection

```java
// ✅ Need random access
List<String> list = new ArrayList<>();

// ✅ Need frequent insertions/deletions
List<String> list = new LinkedList<>();

// ✅ Need no duplicates, fast lookup
Set<String> set = new HashSet<>();

// ✅ Need sorted order
Set<String> set = new TreeSet<>();

// ✅ Need key-value, fast lookup
Map<String, Integer> map = new HashMap<>();

// ✅ Need thread-safety
Map<String, Integer> map = new ConcurrentHashMap<>();
```

### 2. Use Interface Types

```java
// ✅ GOOD - Program to interface
List<String> list = new ArrayList<>();
Set<Integer> set = new HashSet<>();
Map<String, Integer> map = new HashMap<>();

// ❌ BAD - Concrete type
ArrayList<String> list = new ArrayList<>();
```

### 3. Initialize with Capacity

```java
// ✅ GOOD - Prevents resizing
List<String> list = new ArrayList<>(expectedSize);
Map<String, Integer> map = new HashMap<>(expectedSize * 2);

// ❌ BAD - Multiple resizes
List<String> list = new ArrayList<>();
```

### 4. Use Immutable Collections

```java
// ✅ GOOD - Thread-safe, clear intent
List<String> immutable = List.of("A", "B", "C");
Set<Integer> immutableSet = Set.of(1, 2, 3);
Map<String, Integer> immutableMap = Map.of("A", 1, "B", 2);

// ❌ BAD - Mutable, not thread-safe
List<String> mutable = new ArrayList<>();
```

### 5. Avoid Premature Optimization

```java
// ✅ GOOD - Start simple
List<String> list = new ArrayList<>();

// Only optimize if profiling shows need
// ❌ BAD - Over-engineering
CustomOptimizedList<String> list = new CustomOptimizedList<>();
```

### 6. Handle Nulls Properly

```java
// HashMap allows one null key
Map<String, Integer> map = new HashMap<>();
map.put(null, 1);  // OK

// TreeMap doesn't allow null
Map<String, Integer> treeMap = new TreeMap<>();
treeMap.put(null, 1);  // Throws NullPointerException

// HashSet allows one null
Set<String> set = new HashSet<>();
set.add(null);  // OK
```

### 7. Use Streams for Complex Operations

```java
// ✅ GOOD - Readable, functional
List<String> result = list.stream()
    .filter(s -> s.length() > 5)
    .map(String::toUpperCase)
    .collect(Collectors.toList());

// ❌ BAD - Imperative, verbose
List<String> result = new ArrayList<>();
for (String s : list) {
    if (s.length() > 5) {
        result.add(s.toUpperCase());
    }
}
```

---

## Performance Considerations

### Time Complexity

| Operation | ArrayList | LinkedList | HashSet | HashMap | TreeSet | TreeMap |
|-----------|-----------|------------|---------|---------|---------|---------|
| **Add** | O(1) amortized | O(1) | O(1) avg | O(1) avg | O(log n) | O(log n) |
| **Remove** | O(n) | O(1) if known | O(1) avg | O(1) avg | O(log n) | O(log n) |
| **Get/Contains** | O(1) | O(n) | O(1) avg | O(1) avg | O(log n) | O(log n) |
| **Iteration** | O(n) | O(n) | O(n) | O(n) | O(n) | O(n) |

### Space Complexity

- **ArrayList**: O(n) - Contiguous array
- **LinkedList**: O(n) - Extra pointers
- **HashSet/HashMap**: O(n) - Hash table overhead
- **TreeSet/TreeMap**: O(n) - Tree structure

### Memory Considerations

```java
// ArrayList: Grows by 50%
// Initial: 10, then 15, 22, 33, 49, 73...

// HashMap: Resizes when load factor reached
// Default: 16 buckets, resize at 12 elements (0.75 * 16)
```

---

## Interview Questions

### Q1: What is the difference between ArrayList and LinkedList?

**Answer:**
"ArrayList is implemented as a dynamic array, providing O(1) random access but O(n) for insertions/deletions in the middle. LinkedList is implemented as a doubly linked list, providing O(1) insertions/deletions at known positions but O(n) for random access. I choose ArrayList for frequent random access and iteration, and LinkedList for frequent insertions/deletions or when implementing queues/deques."

**Code Example:**
```java
// ArrayList - Fast random access
List<String> arrayList = new ArrayList<>();
arrayList.get(100);  // O(1)

// LinkedList - Fast insertion
List<String> linkedList = new LinkedList<>();
linkedList.add(0, "First");  // O(1)
```

---

### Q2: When would you use Abstract Class vs Interface for Collections?

**Answer:**
"I use Abstract Class when I need to provide shared implementation and reduce code duplication. For example, AbstractList provides common implementations for isEmpty(), contains(), etc., while leaving abstract methods like get() and size() for subclasses. I use Interface when I need to define contracts and allow multiple inheritance. The Collections Framework uses both: interfaces like List define contracts, while abstract classes like AbstractList provide shared implementations."

**Example:**
```java
// Abstract class for shared implementation
public abstract class AbstractList<E> {
    public boolean isEmpty() { return size() == 0; }  // Shared
    public abstract E get(int index);  // Subclass implements
}

// Interface for contract
public interface List<E> {
    E get(int index);  // Contract
}
```

---

### Q3: How does HashMap work internally?

**Answer:**
"HashMap uses a hash table with an array of buckets. When we put a key-value pair, it calculates the hash code of the key, uses it to find the bucket index, and stores the entry there. In Java 8+, each bucket can be a linked list (for small number of entries) or a red-black tree (when entries exceed 8) to handle collisions. The load factor (default 0.75) determines when to resize. When the number of entries exceeds capacity * load factor, the map doubles in size and rehashes all entries."

**Internal Structure:**
```java
// Array of buckets
Node<K, V>[] table;

// Each bucket: LinkedList or Tree
static class Node<K, V> {
    final int hash;
    final K key;
    V value;
    Node<K, V> next;
}
```

---

### Q4: What is the difference between HashMap and ConcurrentHashMap?

**Answer:**
"HashMap is not thread-safe and can cause data corruption or ConcurrentModificationException in multi-threaded environments. ConcurrentHashMap is thread-safe and uses fine-grained locking (segment-based in Java 7, CAS and synchronized blocks in Java 8+) for better performance. ConcurrentHashMap allows concurrent reads without locking and uses lock striping for writes, making it much faster than synchronized HashMap under high concurrency."

**Code:**
```java
// ❌ Not thread-safe
Map<String, Integer> map = new HashMap<>();

// ✅ Thread-safe, better performance
Map<String, Integer> concurrentMap = new ConcurrentHashMap<>();
```

---

### Q5: How do you handle thread safety in collections?

**Answer:**
"I use concurrent collections like ConcurrentHashMap and CopyOnWriteArrayList for thread safety. ConcurrentHashMap uses fine-grained locking for better performance, while CopyOnWriteArrayList uses copy-on-write strategy for read-heavy workloads. For legacy code, I might use Collections.synchronizedXXX(), but I always synchronize iteration externally. I avoid Vector and Hashtable as they use coarse-grained locking and have poor performance."

**Examples:**
```java
// ✅ ConcurrentHashMap - Fine-grained locking
Map<String, Integer> map = new ConcurrentHashMap<>();

// ✅ CopyOnWriteArrayList - Copy-on-write
List<String> list = new CopyOnWriteArrayList<>();

// ⚠️ Synchronized - Needs external sync for iteration
List<String> syncList = Collections.synchronizedList(new ArrayList<>());
synchronized(syncList) {
    for (String item : syncList) { }
}
```

---

### Q6: What is fail-fast and fail-safe iterator?

**Answer:**
"Fail-fast iterators throw ConcurrentModificationException when the collection is modified during iteration. ArrayList, HashMap use fail-fast iterators. Fail-safe iterators work on a snapshot of the collection, so modifications don't affect iteration. CopyOnWriteArrayList and ConcurrentHashMap use fail-safe iterators. Fail-fast helps detect bugs early, while fail-safe allows safe iteration in concurrent environments."

**Example:**
```java
// Fail-fast
List<String> list = new ArrayList<>(Arrays.asList("A", "B"));
for (String item : list) {
    list.add("C");  // Throws ConcurrentModificationException
}

// Fail-safe
List<String> list = new CopyOnWriteArrayList<>(Arrays.asList("A", "B"));
for (String item : list) {
    list.add("C");  // No exception, uses snapshot
}
```

---

### Q7: How does TreeSet maintain sorted order?

**Answer:**
"TreeSet uses a Red-Black tree (self-balancing binary search tree) internally. When we add elements, they're inserted in sorted order based on their natural ordering or a provided Comparator. The tree maintains balance through rotations, ensuring O(log n) operations. TreeSet doesn't allow null values and requires elements to be Comparable or provide a Comparator."

**Code:**
```java
Set<Integer> set = new TreeSet<>();
set.add(3);
set.add(1);
set.add(2);
// Iteration: 1, 2, 3 (sorted)
```

---

### Q8: What is the load factor in HashMap?

**Answer:**
"The load factor (default 0.75) determines when HashMap resizes. When the number of entries exceeds capacity * load factor, HashMap doubles its capacity and rehashes all entries. A lower load factor (e.g., 0.5) means more memory but fewer collisions. A higher load factor (e.g., 0.9) means less memory but more collisions. The default 0.75 provides a good balance between memory usage and performance."

**Example:**
```java
// Default: resize at 75% full
Map<String, Integer> map = new HashMap<>();  // 16 buckets, resize at 12

// Custom: resize at 50% full
Map<String, Integer> map = new HashMap<>(16, 0.5f);  // Resize at 8
```

---

### Q9: How do you implement a custom collection?

**Answer:**
"I extend AbstractCollection or AbstractList/AbstractSet/AbstractMap to minimize code. I implement abstract methods like iterator() and size(), and override methods as needed. I ensure proper equals(), hashCode(), and toString() implementations. For thread safety, I use appropriate synchronization or concurrent collections."

**Example:**
```java
public class CustomSet<E> extends AbstractSet<E> {
    private final Map<E, Object> map = new HashMap<>();
    private static final Object PRESENT = new Object();
    
    @Override
    public Iterator<E> iterator() {
        return map.keySet().iterator();
    }
    
    @Override
    public int size() {
        return map.size();
    }
    
    @Override
    public boolean add(E e) {
        return map.put(e, PRESENT) == null;
    }
}
```

---

### Q10: What design patterns are used in Collections Framework?

**Answer:**
"The Collections Framework uses several design patterns: Iterator Pattern for uniform traversal, Factory Pattern for creating collections, Adapter Pattern for adapting arrays to lists, Decorator Pattern for adding functionality (synchronized, unmodifiable), Strategy Pattern for comparators, and Template Method Pattern in abstract classes like AbstractList. These patterns provide flexibility, maintainability, and extensibility."

**Examples:**
```java
// Iterator Pattern
for (String item : list) { }

// Factory Pattern
List<String> list = List.of("A", "B");

// Decorator Pattern
List<String> sync = Collections.synchronizedList(list);

// Strategy Pattern
list.sort(Comparator.comparing(String::length));
```

---

## Quick Reference Table

| Collection | Order | Duplicates | Null | Thread-Safe | Use Case |
|------------|-------|------------|------|-------------|----------|
| **ArrayList** | Yes | Yes | Yes | No | Random access |
| **LinkedList** | Yes | Yes | Yes | No | Frequent insertions |
| **HashSet** | No | No | One | No | Fast lookup |
| **LinkedHashSet** | Insertion | No | One | No | Ordered + fast lookup |
| **TreeSet** | Sorted | No | No | No | Sorted order |
| **HashMap** | No | No keys | One key | No | Key-value pairs |
| **LinkedHashMap** | Insertion | No keys | One key | No | Ordered + key-value |
| **TreeMap** | Sorted | No keys | No | No | Sorted keys |
| **ConcurrentHashMap** | No | No keys | No | Yes | Thread-safe map |
| **CopyOnWriteArrayList** | Yes | Yes | Yes | Yes | Read-heavy, thread-safe |

---

## Memory Tips 🧠

### Collection Selection
- **A**rray**L**ist = **A**rray + **L**ist (random access)
- **L**inked**L**ist = **L**inks (pointers)
- **H**ash**S**et = **H**ash + **S**et (no duplicates)
- **T**ree**S**et = **T**ree + **S**orted

### Thread Safety
- **C**oncurrent = **C**oncurrent (thread-safe)
- **C**opy**O**n**W**rite = **C**opy **O**n **W**rite (snapshot)

### Performance
- **H**ash = **H**ash (O(1) average)
- **T**ree = **T**ree (O(log n))

---

## Daily Revision Checklist ✅

- [ ] Understand Collection hierarchy
- [ ] Know when to use Abstract Class vs Interface
- [ ] Choose right List implementation
- [ ] Choose right Set implementation
- [ ] Choose right Map implementation
- [ ] Understand thread safety options
- [ ] Know design patterns used
- [ ] Understand internal implementations
- [ ] Know time/space complexities
- [ ] Practice interview questions

---

**Happy Coding! 🗂️**

