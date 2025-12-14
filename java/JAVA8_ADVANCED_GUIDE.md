# Java 8 - Complete Advanced Guide for Staff Engineers 🚀

> Comprehensive guide to Java 8 features with advanced topics, generics, lambda expressions, streams, and staff engineer interview preparation

---

## Table of Contents
1. [Lambda Expressions](#lambda-expressions)
2. [Functional Interfaces](#functional-interfaces)
3. [Method References](#method-references)
4. [Streams API](#streams-api)
5. [Optional](#optional)
6. [Default and Static Methods in Interfaces](#default-and-static-methods-in-interfaces)
7. [Advanced Generics](#advanced-generics)
8. [Date/Time API](#datetime-api)
9. [CompletableFuture](#completablefuture)
10. [Parallel Streams](#parallel-streams)
11. [Advanced Patterns](#advanced-patterns)
12. [Performance Considerations](#performance-considerations)
13. [Best Practices](#best-practices)
14. [Interview Questions](#interview-questions)

---

## Lambda Expressions

### What are Lambda Expressions?

**Lambda expressions** provide a concise way to represent anonymous functions (functions without a name).

### Syntax

```java
// Basic syntax
(parameters) -> expression

// With body
(parameters) -> {
    statements;
    return value;
}
```

### Evolution: Before and After Java 8

**Before Java 8 (Anonymous Inner Class):**

```java
// Sorting with Comparator
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return s1.length() - s2.length();
    }
});
```

**After Java 8 (Lambda Expression):**

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// Lambda expression
Collections.sort(names, (s1, s2) -> s1.length() - s2.length());

// Or even simpler with method reference
names.sort(Comparator.comparing(String::length));
```

### Lambda Expression Examples

#### Example 1: Runnable

```java
// Before
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        System.out.println("Running");
    }
});

// After
Thread thread = new Thread(() -> System.out.println("Running"));
```

#### Example 2: Event Handler

```java
// Before
button.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
        System.out.println("Button clicked");
    }
});

// After
button.addActionListener(e -> System.out.println("Button clicked"));
```

#### Example 3: Filtering Collections

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Before
List<Integer> evenNumbers = new ArrayList<>();
for (Integer number : numbers) {
    if (number % 2 == 0) {
        evenNumbers.add(number);
    }
}

// After
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
```

### Lambda Expression Types

#### 1. Zero Parameters

```java
() -> System.out.println("Hello");
() -> 42;
() -> {
    System.out.println("Multiple statements");
    return "result";
}
```

#### 2. Single Parameter

```java
x -> x * 2;
s -> s.toUpperCase();
s -> {
    System.out.println(s);
    return s.length();
}
```

#### 3. Multiple Parameters

```java
(x, y) -> x + y;
(s1, s2) -> s1.compareTo(s2);
(a, b, c) -> a + b + c;
```

#### 4. Explicit Type Declaration

```java
// Type inference (preferred)
(x, y) -> x + y;

// Explicit types (when needed)
(int x, int y) -> x + y;
(String s1, String s2) -> s1.length() - s2.length();
```

### Variable Capture

#### Local Variables

```java
int multiplier = 10;

// Lambda can access effectively final local variables
Function<Integer, Integer> multiply = x -> x * multiplier;

// ❌ Cannot modify local variable
// multiplier = 20; // Compilation error
```

#### Instance and Static Variables

```java
public class LambdaExample {
    private int instanceVar = 10;
    private static int staticVar = 20;
    
    public void test() {
        // Can access and modify instance variables
        Runnable r1 = () -> {
            instanceVar = 30; // OK
            System.out.println(instanceVar);
        };
        
        // Can access and modify static variables
        Runnable r2 = () -> {
            staticVar = 40; // OK
            System.out.println(staticVar);
        };
    }
}
```

### Real-World Example from E-Kart

**Location:** `product-service/src/main/java/com/ekart/product_service/service/ProductService.java`

```java
@Service
public class ProductService {
    
    /**
     * Calculate total amount using lambda expressions
     */
    public BigDecimal calculateTotal(List<CartItem> cartItems) {
        return cartItems.stream()
            .map(item -> {
                BigDecimal unitPrice = getPrice(item.getSkuId());
                return unitPrice.multiply(BigDecimal.valueOf(item.getQuantity()));
            })
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
    
    /**
     * Filter products by category using lambda
     */
    public List<ProductDTO> getProductsByCategory(List<ProductDTO> products, String category) {
        return products.stream()
            .filter(product -> category.equals(product.getCategory()))
            .collect(Collectors.toList());
    }
    
    /**
     * Group products by seller using lambda
     */
    public Map<Long, List<ProductDTO>> groupProductsBySeller(List<ProductDTO> products) {
        return products.stream()
            .collect(Collectors.groupingBy(ProductDTO::getSellerId));
    }
}
```

---

## Functional Interfaces

### What are Functional Interfaces?

A **functional interface** is an interface with exactly one abstract method. They enable lambda expressions.

### @FunctionalInterface Annotation

```java
@FunctionalInterface
public interface MyFunctionalInterface {
    void execute();
    
    // Can have default methods
    default void defaultMethod() {
        System.out.println("Default method");
    }
    
    // Can have static methods
    static void staticMethod() {
        System.out.println("Static method");
    }
}
```

### Built-in Functional Interfaces

#### 1. Function<T, R>

**Represents a function that takes one argument and produces a result.**

```java
@FunctionalInterface
public interface Function<T, R> {
    R apply(T t);
    
    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after);
    default <V> Function<V, R> compose(Function<? super V, ? extends T> before);
    static <T> Function<T, T> identity();
}
```

**Examples:**

```java
// Convert String to Integer
Function<String, Integer> stringToInt = s -> Integer.parseInt(s);
Integer result = stringToInt.apply("123"); // 123

// Convert Integer to String
Function<Integer, String> intToString = i -> String.valueOf(i);
String result = intToString.apply(123); // "123"

// Chain functions
Function<String, Integer> parseAndDouble = stringToInt.andThen(i -> i * 2);
Integer result = parseAndDouble.apply("10"); // 20

// Compose functions
Function<Integer, String> doubleAndString = intToString.compose(i -> i * 2);
String result = doubleAndString.apply(5); // "10"
```

**Real-World Example:**

```java
// Transform product list
List<ProductDTO> products = getProducts();
List<String> productNames = products.stream()
    .map(ProductDTO::getProductName)  // Function<ProductDTO, String>
    .collect(Collectors.toList());
```

#### 2. Predicate<T>

**Represents a boolean-valued function of one argument.**

```java
@FunctionalInterface
public interface Predicate<T> {
    boolean test(T t);
    
    default Predicate<T> and(Predicate<? super T> other);
    default Predicate<T> or(Predicate<? super T> other);
    default Predicate<T> negate();
    static <T> Predicate<T> isEqual(Object targetRef);
}
```

**Examples:**

```java
// Simple predicate
Predicate<Integer> isEven = n -> n % 2 == 0;
boolean result = isEven.test(4); // true

// Combine predicates
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEvenAndPositive = isEven.and(isPositive);
boolean result = isEvenAndPositive.test(4); // true

// Negate predicate
Predicate<Integer> isOdd = isEven.negate();
boolean result = isOdd.test(3); // true

// Chain predicates
Predicate<String> isNotEmpty = s -> !s.isEmpty();
Predicate<String> startsWithA = s -> s.startsWith("A");
Predicate<String> complex = isNotEmpty.and(startsWithA);
```

**Real-World Example:**

```java
// Filter products
List<ProductDTO> products = getProducts();
List<ProductDTO> availableProducts = products.stream()
    .filter(product -> product.getAvailableQuantity() > 0)  // Predicate<ProductDTO>
    .filter(product -> product.getIsActive())
    .collect(Collectors.toList());
```

#### 3. Consumer<T>

**Represents an operation that accepts a single input argument and returns no result.**

```java
@FunctionalInterface
public interface Consumer<T> {
    void accept(T t);
    
    default Consumer<T> andThen(Consumer<? super T> after);
}
```

**Examples:**

```java
// Simple consumer
Consumer<String> printer = s -> System.out.println(s);
printer.accept("Hello"); // Prints "Hello"

// Chain consumers
Consumer<String> upperCase = s -> System.out.println(s.toUpperCase());
Consumer<String> lowerCase = s -> System.out.println(s.toLowerCase());
Consumer<String> both = upperCase.andThen(lowerCase);
both.accept("Hello"); // Prints "HELLO" then "hello"
```

**Real-World Example:**

```java
// Process each product
List<ProductDTO> products = getProducts();
products.forEach(product -> {
    logger.info("Processing product: {}", product.getProductName());
    updateCache(product);
});
```

#### 4. Supplier<T>

**Represents a supplier of results (no arguments, returns a value).**

```java
@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

**Examples:**

```java
// Simple supplier
Supplier<String> stringSupplier = () -> "Hello";
String result = stringSupplier.get(); // "Hello"

// Random number supplier
Supplier<Integer> randomSupplier = () -> new Random().nextInt(100);
Integer random = randomSupplier.get();

// Date supplier
Supplier<LocalDateTime> dateSupplier = LocalDateTime::now;
LocalDateTime now = dateSupplier.get();
```

**Real-World Example:**

```java
// Lazy initialization
Supplier<List<ProductDTO>> productSupplier = () -> {
    logger.info("Loading products from database");
    return productRepository.findAll();
};

// Only loads when needed
if (needsProducts) {
    List<ProductDTO> products = productSupplier.get();
}
```

#### 5. BiFunction<T, U, R>

**Represents a function that accepts two arguments and produces a result.**

```java
@FunctionalInterface
public interface BiFunction<T, U, R> {
    R apply(T t, U u);
    
    default <V> BiFunction<T, U, V> andThen(Function<? super R, ? extends V> after);
}
```

**Examples:**

```java
// Add two numbers
BiFunction<Integer, Integer, Integer> add = (a, b) -> a + b;
Integer result = add.apply(5, 3); // 8

// Concatenate strings
BiFunction<String, String, String> concat = (s1, s2) -> s1 + s2;
String result = concat.apply("Hello", "World"); // "HelloWorld"

// Calculate total price
BiFunction<BigDecimal, Integer, BigDecimal> calculateTotal = 
    (price, quantity) -> price.multiply(BigDecimal.valueOf(quantity));
BigDecimal total = calculateTotal.apply(new BigDecimal("10.50"), 3); // 31.50
```

#### 6. UnaryOperator<T>

**Represents an operation on a single operand that produces a result of the same type.**

```java
@FunctionalInterface
public interface UnaryOperator<T> extends Function<T, T> {
    static <T> UnaryOperator<T> identity();
}
```

**Examples:**

```java
// Double a number
UnaryOperator<Integer> doubleIt = x -> x * 2;
Integer result = doubleIt.apply(5); // 10

// Uppercase a string
UnaryOperator<String> upperCase = String::toUpperCase;
String result = upperCase.apply("hello"); // "HELLO"

// Identity function
UnaryOperator<String> identity = UnaryOperator.identity();
String result = identity.apply("test"); // "test"
```

#### 7. BinaryOperator<T>

**Represents an operation upon two operands of the same type, producing a result of the same type.**

```java
@FunctionalInterface
public interface BinaryOperator<T> extends BiFunction<T, T, T> {
    static <T> BinaryOperator<T> minBy(Comparator<? super T> comparator);
    static <T> BinaryOperator<T> maxBy(Comparator<? super T> comparator);
}
```

**Examples:**

```java
// Add two numbers
BinaryOperator<Integer> add = (a, b) -> a + b;
Integer result = add.apply(5, 3); // 8

// Find maximum
BinaryOperator<Integer> max = BinaryOperator.maxBy(Integer::compareTo);
Integer result = max.apply(5, 3); // 5

// Find minimum
BinaryOperator<String> min = BinaryOperator.minBy(String::compareTo);
String result = min.apply("apple", "banana"); // "apple"
```

### Custom Functional Interfaces

```java
@FunctionalInterface
public interface TriFunction<T, U, V, R> {
    R apply(T t, U u, V v);
}

// Usage
TriFunction<Integer, Integer, Integer, Integer> addThree = (a, b, c) -> a + b + c;
Integer result = addThree.apply(1, 2, 3); // 6

@FunctionalInterface
public interface ThrowableFunction<T, R, E extends Exception> {
    R apply(T t) throws E;
}

// Usage
ThrowableFunction<String, Integer, NumberFormatException> parseInt = Integer::parseInt;
```

---

## Method References

### What are Method References?

**Method references** are shorthand syntax for lambda expressions that call existing methods.

### Types of Method References

#### 1. Static Method Reference

**Syntax:** `ClassName::staticMethodName`

```java
// Lambda
Function<String, Integer> parser = s -> Integer.parseInt(s);

// Method reference
Function<String, Integer> parser = Integer::parseInt;

// Example
List<String> numbers = Arrays.asList("1", "2", "3");
List<Integer> integers = numbers.stream()
    .map(Integer::parseInt)  // Static method reference
    .collect(Collectors.toList());
```

#### 2. Instance Method Reference (of an object)

**Syntax:** `object::instanceMethodName`

```java
String prefix = "Hello ";

// Lambda
Function<String, String> addPrefix = s -> prefix.concat(s);

// Method reference
Function<String, String> addPrefix = prefix::concat;

// Example
List<String> names = Arrays.asList("Alice", "Bob");
List<String> greetings = names.stream()
    .map(prefix::concat)
    .collect(Collectors.toList());
```

#### 3. Instance Method Reference (of an arbitrary object)

**Syntax:** `ClassName::instanceMethodName`

```java
// Lambda
Function<String, Integer> length = s -> s.length();

// Method reference
Function<String, Integer> length = String::length;

// Example
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
List<Integer> lengths = names.stream()
    .map(String::length)  // Instance method reference
    .collect(Collectors.toList());
```

#### 4. Constructor Reference

**Syntax:** `ClassName::new`

```java
// Lambda
Supplier<List<String>> listSupplier = () -> new ArrayList<>();

// Constructor reference
Supplier<List<String>> listSupplier = ArrayList::new;

// Example
List<String> names = Arrays.asList("Alice", "Bob");
List<String> upperCase = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toCollection(ArrayList::new));  // Constructor reference
```

### Real-World Examples

```java
// Static method reference
List<String> numbers = Arrays.asList("1", "2", "3");
List<Integer> integers = numbers.stream()
    .map(Integer::parseInt)
    .collect(Collectors.toList());

// Instance method reference
List<ProductDTO> products = getProducts();
List<String> names = products.stream()
    .map(ProductDTO::getProductName)  // getProductName() method
    .collect(Collectors.toList());

// Constructor reference
List<String> names = Arrays.asList("Alice", "Bob");
Set<String> nameSet = names.stream()
    .collect(Collectors.toCollection(HashSet::new));

// Two-argument constructor
BiFunction<String, Integer, ProductDTO> productCreator = ProductDTO::new;
ProductDTO product = productCreator.apply("Laptop", 1000);
```

---

## Streams API

### What are Streams?

**Streams** represent a sequence of elements supporting sequential and parallel aggregate operations.

### Key Characteristics

1. **Not a data structure** - doesn't store elements
2. **Functional in nature** - operations produce results but don't modify source
3. **Lazy evaluation** - operations are executed only when terminal operation is called
4. **Potentially unbounded** - can work with infinite streams
5. **Consumable** - elements are visited only once

### Stream Creation

#### 1. From Collections

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
Stream<String> stream = names.stream();
```

#### 2. From Arrays

```java
String[] names = {"Alice", "Bob", "Charlie"};
Stream<String> stream = Arrays.stream(names);
```

#### 3. Using Stream.of()

```java
Stream<String> stream = Stream.of("Alice", "Bob", "Charlie");
```

#### 4. Using Stream.builder()

```java
Stream<String> stream = Stream.<String>builder()
    .add("Alice")
    .add("Bob")
    .add("Charlie")
    .build();
```

#### 5. Infinite Streams

```java
// Generate infinite stream
Stream<Integer> infinite = Stream.iterate(0, n -> n + 2);

// Generate with supplier
Stream<Double> random = Stream.generate(Math::random);
```

### Intermediate Operations

**Return a new stream, allowing chaining.**

#### 1. filter()

**Filters elements based on a predicate.**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList()); // [2, 4, 6, 8, 10]
```

**Real-World Example:**

```java
List<ProductDTO> products = getProducts();
List<ProductDTO> availableProducts = products.stream()
    .filter(product -> product.getAvailableQuantity() > 0)
    .filter(product -> product.getIsActive())
    .collect(Collectors.toList());
```

#### 2. map()

**Transforms each element to another value.**

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

List<Integer> lengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList()); // [5, 3, 7]
```

**Real-World Example:**

```java
List<ProductDTO> products = getProducts();
List<String> productNames = products.stream()
    .map(ProductDTO::getProductName)
    .collect(Collectors.toList());
```

#### 3. flatMap()

**Flattens nested structures.**

```java
List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2, 3),
    Arrays.asList(4, 5, 6),
    Arrays.asList(7, 8, 9)
);

List<Integer> flattened = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList()); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

**Real-World Example:**

```java
List<Order> orders = getOrders();
List<OrderItem> allItems = orders.stream()
    .flatMap(order -> order.getItems().stream())
    .collect(Collectors.toList());
```

#### 4. distinct()

**Removes duplicates.**

```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4);

List<Integer> unique = numbers.stream()
    .distinct()
    .collect(Collectors.toList()); // [1, 2, 3, 4]
```

#### 5. sorted()

**Sorts elements.**

```java
List<String> names = Arrays.asList("Charlie", "Alice", "Bob");

List<String> sorted = names.stream()
    .sorted()
    .collect(Collectors.toList()); // [Alice, Bob, Charlie]

// Custom comparator
List<String> sortedByLength = names.stream()
    .sorted(Comparator.comparing(String::length))
    .collect(Collectors.toList());
```

#### 6. peek()

**Performs action on each element (mainly for debugging).**

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

List<String> result = names.stream()
    .peek(System.out::println)  // Debug: print each element
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

#### 7. limit()

**Limits the number of elements.**

```java
Stream<Integer> infinite = Stream.iterate(0, n -> n + 1);

List<Integer> firstTen = infinite
    .limit(10)
    .collect(Collectors.toList()); // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

#### 8. skip()

**Skips the first n elements.**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> skipped = numbers.stream()
    .skip(5)
    .collect(Collectors.toList()); // [6, 7, 8, 9, 10]
```

### Terminal Operations

**Produce a result or side effect, ending the stream.**

#### 1. collect()

**Collects elements into a collection.**

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// To List
List<String> list = names.stream().collect(Collectors.toList());

// To Set
Set<String> set = names.stream().collect(Collectors.toSet());

// To Map
Map<String, Integer> map = names.stream()
    .collect(Collectors.toMap(Function.identity(), String::length));

// To Custom Collection
LinkedList<String> linkedList = names.stream()
    .collect(Collectors.toCollection(LinkedList::new));
```

#### 2. forEach()

**Performs action on each element.**

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

names.stream()
    .forEach(System.out::println);
```

#### 3. reduce()

**Reduces stream to a single value.**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// Sum
Integer sum = numbers.stream()
    .reduce(0, Integer::sum); // 15

// Product
Integer product = numbers.stream()
    .reduce(1, (a, b) -> a * b); // 120

// Maximum
Optional<Integer> max = numbers.stream()
    .reduce(Integer::max); // Optional[5]
```

**Real-World Example:**

```java
List<CartItem> cartItems = getCartItems();
BigDecimal total = cartItems.stream()
    .map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())))
    .reduce(BigDecimal.ZERO, BigDecimal::add);
```

#### 4. count()

**Counts elements.**

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

long count = names.stream()
    .filter(s -> s.length() > 3)
    .count(); // 2
```

#### 5. anyMatch(), allMatch(), noneMatch()

**Checks if elements match a predicate.**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

boolean anyEven = numbers.stream().anyMatch(n -> n % 2 == 0); // true
boolean allEven = numbers.stream().allMatch(n -> n % 2 == 0); // false
boolean noneNegative = numbers.stream().noneMatch(n -> n < 0); // true
```

#### 6. findFirst(), findAny()

**Finds an element.**

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

Optional<String> first = names.stream()
    .filter(s -> s.startsWith("A"))
    .findFirst(); // Optional["Alice"]

Optional<String> any = names.stream()
    .filter(s -> s.length() > 3)
    .findAny(); // Optional["Alice"] or Optional["Charlie"]
```

#### 7. min(), max()

**Finds minimum or maximum element.**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

Optional<Integer> min = numbers.stream().min(Integer::compareTo); // Optional[1]
Optional<Integer> max = numbers.stream().max(Integer::compareTo); // Optional[5]
```

### Collectors

**Advanced collection operations.**

#### 1. groupingBy()

**Groups elements by a classifier.**

```java
List<ProductDTO> products = getProducts();

// Group by category
Map<String, List<ProductDTO>> byCategory = products.stream()
    .collect(Collectors.groupingBy(ProductDTO::getCategory));

// Group by category with counting
Map<String, Long> countByCategory = products.stream()
    .collect(Collectors.groupingBy(ProductDTO::getCategory, Collectors.counting()));

// Group by category with sum of prices
Map<String, BigDecimal> sumByCategory = products.stream()
    .collect(Collectors.groupingBy(
        ProductDTO::getCategory,
        Collectors.reducing(BigDecimal.ZERO, ProductDTO::getSellingPrice, BigDecimal::add)
    ));
```

#### 2. partitioningBy()

**Partitions elements into two groups.**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

Map<Boolean, List<Integer>> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));
// {false=[1, 3, 5, 7, 9], true=[2, 4, 6, 8, 10]}
```

#### 3. joining()

**Joins elements into a string.**

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

String joined = names.stream()
    .collect(Collectors.joining()); // "AliceBobCharlie"

String joinedWithComma = names.stream()
    .collect(Collectors.joining(", ")); // "Alice, Bob, Charlie"

String joinedWithPrefixSuffix = names.stream()
    .collect(Collectors.joining(", ", "[", "]")); // "[Alice, Bob, Charlie]"
```

#### 4. summarizingInt/Long/Double()

**Provides statistics.**

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

IntSummaryStatistics stats = numbers.stream()
    .collect(Collectors.summarizingInt(Integer::intValue));

System.out.println(stats.getCount());    // 5
System.out.println(stats.getSum());      // 15
System.out.println(stats.getMin());      // 1
System.out.println(stats.getMax());      // 5
System.out.println(stats.getAverage());  // 3.0
```

### Real-World Stream Examples from E-Kart

**Location:** `product-service/src/main/java/com/ekart/product_service/service/ProductService.java`

```java
@Service
public class ProductService {
    
    /**
     * Calculate total amount using streams
     */
    public BigDecimal calculateTotal(List<CartItem> cartItems) {
        return cartItems.stream()
            .map(item -> {
                BigDecimal unitPrice = getPrice(item.getSkuId());
                return unitPrice.multiply(BigDecimal.valueOf(item.getQuantity()));
            })
            .reduce(BigDecimal.ZERO, BigDecimal::add);
    }
    
    /**
     * Group products by seller
     */
    public Map<Long, List<ProductDTO>> groupProductsBySeller(List<ProductDTO> products) {
        return products.stream()
            .collect(Collectors.groupingBy(ProductDTO::getSellerId));
    }
    
    /**
     * Filter and transform products
     */
    public List<String> getActiveProductNames(List<ProductDTO> products) {
        return products.stream()
            .filter(ProductDTO::getIsActive)
            .filter(p -> p.getAvailableQuantity() > 0)
            .map(ProductDTO::getProductName)
            .sorted()
            .collect(Collectors.toList());
    }
    
    /**
     * Find product by ID
     */
    public Optional<ProductDTO> findProduct(Long skuId) {
        return products.stream()
            .filter(p -> p.getSkuId().equals(skuId))
            .findFirst();
    }
    
    /**
     * Calculate statistics
     */
    public ProductStatistics calculateStatistics(List<ProductDTO> products) {
        DoubleSummaryStatistics priceStats = products.stream()
            .mapToDouble(p -> p.getSellingPrice().doubleValue())
            .summaryStatistics();
        
        return new ProductStatistics(
            priceStats.getCount(),
            priceStats.getMin(),
            priceStats.getMax(),
            priceStats.getAverage()
        );
    }
}
```

---

## Optional

### What is Optional?

**Optional** is a container object that may or may not contain a non-null value. It helps avoid NullPointerException.

### Creating Optional

```java
// Empty Optional
Optional<String> empty = Optional.empty();

// Optional with value
Optional<String> present = Optional.of("Hello");

// Optional that may be null
String name = getName(); // may return null
Optional<String> optional = Optional.ofNullable(name);
```

### Checking Value Presence

```java
Optional<String> optional = Optional.of("Hello");

if (optional.isPresent()) {
    String value = optional.get();
}

// Java 11+
optional.ifPresent(value -> System.out.println(value));
```

### Getting Values

```java
Optional<String> optional = Optional.of("Hello");

// Get value (throws NoSuchElementException if empty)
String value = optional.get();

// Get value or default
String value = optional.orElse("Default");

// Get value or compute default
String value = optional.orElseGet(() -> "Computed Default");

// Get value or throw exception
String value = optional.orElseThrow(() -> new IllegalArgumentException("No value"));
```

### Transforming Values

```java
Optional<String> optional = Optional.of("Hello");

// Map to another type
Optional<Integer> length = optional.map(String::length);

// FlatMap (when function returns Optional)
Optional<String> upper = optional.flatMap(s -> Optional.of(s.toUpperCase()));

// Filter
Optional<String> filtered = optional.filter(s -> s.length() > 3);
```

### Real-World Examples

```java
@Service
public class ProductService {
    
    /**
     * Find product safely
     */
    public Optional<ProductDTO> findProduct(Long skuId) {
        return productRepository.findById(skuId)
            .map(this::convertToDTO);
    }
    
    /**
     * Get product name or default
     */
    public String getProductName(Long skuId) {
        return findProduct(skuId)
            .map(ProductDTO::getProductName)
            .orElse("Unknown Product");
    }
    
    /**
     * Process product if present
     */
    public void processProduct(Long skuId) {
        findProduct(skuId)
            .ifPresent(product -> {
                logger.info("Processing product: {}", product.getProductName());
                updateCache(product);
            });
    }
    
    /**
     * Chain Optional operations
     */
    public Optional<BigDecimal> getProductPrice(Long skuId) {
        return findProduct(skuId)
            .map(ProductDTO::getSellingPrice)
            .filter(price -> price.compareTo(BigDecimal.ZERO) > 0);
    }
}
```

### Best Practices with Optional

**✅ DO:**
```java
// Use Optional for return types
public Optional<ProductDTO> findProduct(Long id) {
    return Optional.ofNullable(productRepository.findById(id));
}

// Use orElse for simple defaults
String name = optional.orElse("Default");

// Use orElseGet for expensive computations
String name = optional.orElseGet(() -> expensiveComputation());

// Use ifPresent for side effects
optional.ifPresent(value -> process(value));
```

**❌ DON'T:**
```java
// Don't use Optional as field
private Optional<String> name; // BAD

// Don't use Optional in collections
List<Optional<String>> list; // BAD

// Don't use Optional.get() without checking
String value = optional.get(); // BAD - may throw exception

// Don't use Optional for parameters
public void process(Optional<String> value) { // BAD
```

---

## Default and Static Methods in Interfaces

### Default Methods

**Allow interfaces to have method implementations.**

```java
public interface Calculator {
    int add(int a, int b);
    
    // Default method
    default int subtract(int a, int b) {
        return a - b;
    }
    
    // Default method can call other methods
    default int multiply(int a, int b) {
        return add(a, b) * 2; // Calls abstract method
    }
}

public class BasicCalculator implements Calculator {
    @Override
    public int add(int a, int b) {
        return a + b;
    }
    // subtract() and multiply() are inherited
}
```

### Static Methods in Interfaces

**Allow interfaces to have static utility methods.**

```java
public interface MathUtils {
    static int max(int a, int b) {
        return a > b ? a : b;
    }
    
    static int min(int a, int b) {
        return a < b ? a : b;
    }
}

// Usage
int max = MathUtils.max(5, 3); // 5
```

### Real-World Example

```java
public interface ProductRepository {
    Product findById(Long id);
    
    // Default method
    default Optional<Product> findByIdOptional(Long id) {
        Product product = findById(id);
        return Optional.ofNullable(product);
    }
    
    // Static factory method
    static ProductRepository inMemory() {
        return new InMemoryProductRepository();
    }
}
```

---

## Advanced Generics

### Generic Classes

```java
public class Box<T> {
    private T value;
    
    public void setValue(T value) {
        this.value = value;
    }
    
    public T getValue() {
        return value;
    }
}

// Usage
Box<String> stringBox = new Box<>();
stringBox.setValue("Hello");
String value = stringBox.getValue();

Box<Integer> intBox = new Box<>();
intBox.setValue(42);
Integer number = intBox.getValue();
```

### Generic Methods

```java
public class Utils {
    // Generic method
    public static <T> T getFirst(List<T> list) {
        return list.isEmpty() ? null : list.get(0);
    }
    
    // Multiple type parameters
    public static <T, U> U convert(T input, Function<T, U> converter) {
        return converter.apply(input);
    }
}

// Usage
String first = Utils.getFirst(Arrays.asList("Alice", "Bob"));
Integer converted = Utils.convert("123", Integer::parseInt);
```

### Bounded Type Parameters

```java
// Upper bound
public class NumberBox<T extends Number> {
    private T value;
    
    public double getDoubleValue() {
        return value.doubleValue(); // Can call Number methods
    }
}

// Multiple bounds
public class Processor<T extends Number & Comparable<T>> {
    public void process(T value) {
        // T is both Number and Comparable
    }
}

// Lower bound (in wildcards)
public void addNumbers(List<? super Integer> list) {
    list.add(1); // Can add Integer
    list.add(2);
}
```

### Wildcards

```java
// Unbounded wildcard
public void processList(List<?> list) {
    // Can read but not write (except null)
}

// Upper bounded wildcard
public void processNumbers(List<? extends Number> list) {
    // Can read as Number
    Number n = list.get(0);
    // Cannot write (except null)
}

// Lower bounded wildcard
public void addNumbers(List<? super Integer> list) {
    // Can write Integer
    list.add(1);
    // Can read as Object
    Object o = list.get(0);
}
```

### PECS Principle

**Producer Extends, Consumer Super**

```java
// Producer - use extends
public void copyNumbers(List<? extends Number> source, List<? super Number> dest) {
    for (Number n : source) {
        dest.add(n);
    }
}

// Consumer - use super
public void addIntegers(List<? super Integer> list) {
    list.add(1);
    list.add(2);
}
```

### Type Erasure

**Generics are erased at runtime.**

```java
List<String> stringList = new ArrayList<>();
List<Integer> intList = new ArrayList<>();

// At runtime, both are just List
System.out.println(stringList.getClass() == intList.getClass()); // true

// Cannot use instanceof with generics
// if (list instanceof List<String>) { } // Compilation error
```

### Advanced Generic Patterns

#### 1. Generic Builder

```java
public class ProductBuilder<T extends ProductBuilder<T>> {
    protected String name;
    protected BigDecimal price;
    
    public T name(String name) {
        this.name = name;
        return self();
    }
    
    public T price(BigDecimal price) {
        this.price = price;
        return self();
    }
    
    @SuppressWarnings("unchecked")
    protected T self() {
        return (T) this;
    }
    
    public Product build() {
        return new Product(name, price);
    }
}
```

#### 2. Generic Repository

```java
public interface Repository<T, ID> {
    T findById(ID id);
    List<T> findAll();
    T save(T entity);
    void deleteById(ID id);
}

public class ProductRepository implements Repository<Product, Long> {
    @Override
    public Product findById(Long id) {
        // Implementation
    }
    
    // Other methods...
}
```

#### 3. Generic Service

```java
public abstract class BaseService<T, ID> {
    protected Repository<T, ID> repository;
    
    public T findById(ID id) {
        return repository.findById(id);
    }
    
    public List<T> findAll() {
        return repository.findAll();
    }
    
    public T save(T entity) {
        return repository.save(entity);
    }
}

public class ProductService extends BaseService<Product, Long> {
    // Product-specific methods
}
```

---

## Date/Time API

### Problems with Old Date/Time API

```java
// Old API problems
Date date = new Date(); // Mutable, not thread-safe
Calendar calendar = Calendar.getInstance(); // Complex, error-prone
```

### New Date/Time API (java.time)

#### 1. LocalDate

```java
// Current date
LocalDate today = LocalDate.now();

// Specific date
LocalDate date = LocalDate.of(2024, 1, 15);

// Parse from string
LocalDate parsed = LocalDate.parse("2024-01-15");

// Operations
LocalDate tomorrow = today.plusDays(1);
LocalDate nextWeek = today.plusWeeks(1);
LocalDate nextMonth = today.plusMonths(1);
LocalDate nextYear = today.plusYears(1);

// Comparison
boolean isAfter = today.isAfter(date);
boolean isBefore = today.isBefore(date);
```

#### 2. LocalTime

```java
// Current time
LocalTime now = LocalTime.now();

// Specific time
LocalTime time = LocalTime.of(14, 30, 45);

// Parse from string
LocalTime parsed = LocalTime.parse("14:30:45");

// Operations
LocalTime later = now.plusHours(2);
LocalTime earlier = now.minusMinutes(30);
```

#### 3. LocalDateTime

```java
// Current date and time
LocalDateTime now = LocalDateTime.now();

// Specific date and time
LocalDateTime dateTime = LocalDateTime.of(2024, 1, 15, 14, 30, 45);

// From LocalDate and LocalTime
LocalDateTime dateTime = LocalDate.now().atTime(14, 30);

// Operations
LocalDateTime later = now.plusHours(2);
LocalDateTime earlier = now.minusDays(1);
```

#### 4. ZonedDateTime

```java
// Current date and time with timezone
ZonedDateTime now = ZonedDateTime.now();

// Specific timezone
ZonedDateTime tokyo = ZonedDateTime.now(ZoneId.of("Asia/Tokyo"));

// Convert timezone
ZonedDateTime newYork = tokyo.withZoneSameInstant(ZoneId.of("America/New_York"));
```

#### 5. Duration and Period

```java
// Duration (time-based)
Duration duration = Duration.ofHours(2);
Duration between = Duration.between(startTime, endTime);

// Period (date-based)
Period period = Period.ofDays(7);
Period between = Period.between(startDate, endDate);
```

### Real-World Example

```java
@Service
public class OrderService {
    
    public void processOrder(Order order) {
        LocalDateTime orderTime = LocalDateTime.now();
        order.setOrderTime(orderTime);
        
        // Calculate delivery date (3 business days)
        LocalDate deliveryDate = orderTime.toLocalDate()
            .plusDays(3);
        order.setDeliveryDate(deliveryDate);
        
        // Check if order is within business hours
        LocalTime orderTimeOnly = orderTime.toLocalTime();
        if (orderTimeOnly.isBefore(LocalTime.of(9, 0)) || 
            orderTimeOnly.isAfter(LocalTime.of(17, 0))) {
            // Handle after-hours order
        }
    }
}
```

---

## CompletableFuture

### What is CompletableFuture?

**CompletableFuture** is a class that represents a future result of an asynchronous computation.

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

---

## Parallel Streams

### Creating Parallel Streams

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Parallel stream
List<Integer> doubled = numbers.parallelStream()
    .map(n -> n * 2)
    .collect(Collectors.toList());

// Convert sequential to parallel
List<Integer> doubled = numbers.stream()
    .parallel()
    .map(n -> n * 2)
    .collect(Collectors.toList());
```

### When to Use Parallel Streams

**✅ Use when:**
- Large datasets
- CPU-intensive operations
- Independent operations
- No shared mutable state

**❌ Don't use when:**
- Small datasets
- I/O operations
- Operations with shared state
- Order matters

---

## Advanced Patterns

### Strategy Pattern with Lambdas

```java
// Before
public interface PaymentStrategy {
    void pay(BigDecimal amount);
}

public class CreditCardPayment implements PaymentStrategy {
    @Override
    public void pay(BigDecimal amount) {
        // Credit card payment logic
    }
}

// After
public void processPayment(BigDecimal amount, PaymentStrategy strategy) {
    strategy.pay(amount);
}

// Usage
processPayment(amount, a -> creditCardService.charge(a));
processPayment(amount, a -> payPalService.pay(a));
```

### Chain of Responsibility with Lambdas

```java
Function<String, String> chain = 
    ((Function<String, String>) String::trim)
    .andThen(String::toUpperCase)
    .andThen(s -> s.replace(" ", "_"));

String result = chain.apply("  hello world  "); // "HELLO_WORLD"
```

---

## Performance Considerations

### Stream Performance

```java
// ❌ BAD: Multiple iterations
List<String> names = getNames();
List<String> filtered = names.stream().filter(s -> s.length() > 5).collect(toList());
List<String> upper = names.stream().map(String::toUpperCase).collect(toList());

// ✅ GOOD: Single iteration
List<String> result = names.stream()
    .filter(s -> s.length() > 5)
    .map(String::toUpperCase)
    .collect(toList());
```

### Lazy Evaluation

```java
// Stream operations are lazy
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// No terminal operation - nothing happens
Stream<Integer> stream = numbers.stream()
    .filter(n -> {
        System.out.println("Filtering: " + n);
        return n % 2 == 0;
    })
    .map(n -> {
        System.out.println("Mapping: " + n);
        return n * 2;
    });

// Terminal operation triggers evaluation
List<Integer> result = stream.collect(toList());
```

---

## Best Practices

### 1. Prefer Method References

```java
// ❌ BAD
list.stream().map(s -> s.toUpperCase())

// ✅ GOOD
list.stream().map(String::toUpperCase)
```

### 2. Use Appropriate Collectors

```java
// ❌ BAD
list.stream().collect(Collectors.toList())

// ✅ GOOD (when order doesn't matter)
list.stream().collect(Collectors.toSet())
```

### 3. Avoid Side Effects in Streams

```java
// ❌ BAD
list.stream().forEach(item -> {
    process(item);
    updateDatabase(item);
});

// ✅ GOOD
list.stream()
    .map(this::process)
    .forEach(this::updateDatabase);
```

### 4. Use Optional Properly

```java
// ❌ BAD
if (optional.isPresent()) {
    String value = optional.get();
}

// ✅ GOOD
optional.ifPresent(value -> process(value));
```

---

## Interview Questions

### Q1: Explain Lambda Expressions

**Answer:**
"Lambda expressions provide a concise way to represent anonymous functions. They enable functional programming in Java and are used with functional interfaces. The syntax is (parameters) -> expression or (parameters) -> { statements; }. Lambdas enable passing behavior as data, making code more readable and maintainable."

### Q2: What are Functional Interfaces?

**Answer:**
"Functional interfaces are interfaces with exactly one abstract method. They enable lambda expressions. Java 8 provides built-in functional interfaces like Function, Predicate, Consumer, and Supplier. The @FunctionalInterface annotation ensures the interface has only one abstract method."

### Q3: Explain Streams API

**Answer:**
"Streams represent a sequence of elements supporting sequential and parallel aggregate operations. They're not data structures but abstractions for processing collections. Streams are lazy (operations execute only when terminal operation is called), functional (don't modify source), and can be infinite. Operations are divided into intermediate (return stream) and terminal (produce result)."

### Q4: What is Optional and when to use it?

**Answer:**
"Optional is a container that may or may not contain a non-null value. It helps avoid NullPointerException. Use Optional for return types when a method might not return a value. Don't use it for fields, parameters, or in collections. Use methods like orElse(), orElseGet(), ifPresent(), and map() to work with Optional safely."

### Q5: Explain Method References

**Answer:**
"Method references are shorthand for lambda expressions that call existing methods. Types include: static (ClassName::staticMethod), instance of object (object::method), instance of arbitrary object (ClassName::instanceMethod), and constructor (ClassName::new). They make code more readable when lambda just calls a method."

### Q6: What are Default Methods in Interfaces?

**Answer:**
"Default methods allow interfaces to have method implementations. They enable adding new methods to interfaces without breaking existing implementations. Default methods can call other interface methods. They're useful for providing common functionality and evolving interfaces."

### Q7: Explain Generics and Type Erasure

**Answer:**
"Generics provide type safety at compile time. Type erasure means generic type information is removed at runtime. At runtime, List<String> and List<Integer> are both just List. This enables backward compatibility but limits runtime type checking. Use bounded type parameters and wildcards for flexibility."

### Q8: When to use Parallel Streams?

**Answer:**
"Use parallel streams for large datasets with CPU-intensive, independent operations. Don't use for small datasets, I/O operations, or operations with shared mutable state. Parallel streams use ForkJoinPool and can improve performance but have overhead. Measure performance before using."

### Q9: Explain CompletableFuture

**Answer:**
"CompletableFuture represents a future result of asynchronous computation. It provides methods for chaining (thenApply, thenCompose), combining (thenCombine, allOf, anyOf), and error handling (exceptionally, handle). It's more powerful than Future and enables non-blocking asynchronous programming."

### Q10: What is the PECS Principle?

**Answer:**
"PECS stands for Producer Extends, Consumer Super. Use ? extends T for producers (read-only), and ? super T for consumers (write-only). This ensures type safety while maintaining flexibility. For example, copyNumbers(List<? extends Number> source, List<? super Number> dest)."

---

## Summary

### Key Java 8 Features

1. **Lambda Expressions**: Concise anonymous functions
2. **Functional Interfaces**: Single abstract method interfaces
3. **Method References**: Shorthand for method calls
4. **Streams API**: Functional processing of collections
5. **Optional**: Null-safe container
6. **Default Methods**: Interface method implementations
7. **Date/Time API**: Modern date and time handling
8. **CompletableFuture**: Asynchronous programming

### Advanced Topics

1. **Generics**: Type safety and flexibility
2. **Parallel Streams**: Concurrent processing
3. **Advanced Patterns**: Strategy, Chain of Responsibility
4. **Performance**: Lazy evaluation, optimization

### Best Practices

- Use method references when possible
- Prefer functional style over imperative
- Use Optional for return types
- Avoid side effects in streams
- Measure before using parallel streams

---

**Happy Coding with Java 8! 🚀**

