# Java 9 → Java 24 — Complete Guide with Examples (Staff Engineer Focus) 🚀

This guide summarizes **Java 9 through Java 24** with **practical examples** and interview-relevant guidance.

## How to use this guide

- If you’re preparing for interviews, prioritize **LTS releases**: **11**, **17**, **21** (and know the “headline” features around them).
- Use per-version sections for quick recall, and the **“Must-know topics”** section for depth.

---

## Table of Contents

1. [Release roadmap (LTS vs non-LTS)](#release-roadmap-lts-vs-non-lts)
2. [Must-know topics (9→24)](#must-know-topics-924)
3. [Java 9 Highlights](#java-9-highlights)
4. [Java 10 Highlights](#java-10-highlights)
5. [Java 11 (LTS) Highlights](#java-11-lts-highlights)
6. [Java 12 Highlights](#java-12-highlights)
7. [Java 13 Highlights](#java-13-highlights)
8. [Java 14 Highlights](#java-14-highlights)
9. [Java 15 Highlights](#java-15-highlights)
10. [Java 16 Highlights](#java-16-highlights)
11. [Java 17 (LTS) Highlights](#java-17-lts-highlights)
12. [Java 18 Highlights](#java-18-highlights)
13. [Java 19 Highlights](#java-19-highlights)
14. [Java 20 Highlights](#java-20-highlights)
15. [Java 21 (LTS) Highlights](#java-21-lts-highlights)
16. [Java 22 Highlights](#java-22-highlights)
17. [Java 23 Highlights](#java-23-highlights)
18. [Java 24 Highlights](#java-24-highlights)
19. [Interview Questions](#interview-questions)

---

## Release roadmap (LTS vs non-LTS)

- **LTS releases** (commonly adopted in production): **11**, **17**, **21**
- **Non-LTS releases**: 9,10,12–16,18–20,22–24  

Staff-level note: interviewers care less about “every JEP number” and more about whether you can apply the features safely (compatibility, performance, operability).

---

## Must-know topics (9→24)

### 1) Java Platform Module System (JPMS) (Java 9)

Why you care:
- Stronger encapsulation than classpath.
- Cleaner dependency boundaries for large codebases.

**Example: `module-info.java`**

```java
module com.acme.orders {
    requires java.net.http;     // module dependency
    exports com.acme.orders.api; // exported package

    // if needed:
    // opens com.acme.orders.internal to com.fasterxml.jackson.databind;
}
```
### Java 9 Module System — Quick Summary

* **Not mandatory** ❌
  You can run Java 11/17/21 apps **without JPMS** using the classpath.

* **Mandatory only if you opt in**
  JPMS is enforced **only when you add `module-info.java` and use module-path**.

* **Spring Boot & microservices**
  ❌ Usually **avoid JPMS** due to heavy reflection (`opens`, `--add-opens` needed).

* **Where JPMS is useful**
  ✅ Library/framework development
  ✅ Platform/SDKs
  ✅ Large systems needing strict encapsulation
  ✅ Security-sensitive code

* **Industry reality**
  90%+ real-world projects **don’t use JPMS**.

### Interview One-Liner

> *“Java 9 modules are optional; most applications stay on the classpath. JPMS is mainly valuable for library and platform development.”*

module service.module {
    requires spring.context;
    exports com.example.service;
    opens com.example.service to spring.core, spring.beans;
}


### 2) Local variable type inference `var` (Java 10)

Use for readability when the RHS makes the type obvious.

```java
var names = java.util.List.of("Alice", "Bob");
var map = java.util.Map.of("a", 1, "b", 2);
```

Pitfall: don’t use `var` when it hides important types at API boundaries.

Java 10 = productivity (var) + JVM optimizations + container support

Java 10 introduced local variable type inference using var, improved JVM performance, and added container awareness.
It followed a short-term release model, so most enterprises adopted these features later via Java 11 or 17.

### 3) New HTTP Client (standard since Java 11)

```java
import java.net.URI;
import java.net.http.HttpClient;
import java.net.http.HttpRequest;
import java.net.http.HttpResponse;
import java.time.Duration;

public class HttpExample {
    public static void main(String[] args) throws Exception {
        HttpClient client = HttpClient.newBuilder()
                .connectTimeout(Duration.ofSeconds(3))
                .build();

        HttpRequest request = HttpRequest.newBuilder()
                .uri(new URI("https://example.com"))
                .timeout(Duration.ofSeconds(5))
                .GET()
                .build();

        HttpResponse<String> resp = client.send(request, HttpResponse.BodyHandlers.ofString());
        System.out.println(resp.statusCode());
        System.out.println(resp.body().substring(0, Math.min(resp.body().length(), 120)));
    }
}
```

### 4) Switch expressions + pattern matching direction

Switch expressions (finalized before 17) are now a normal idiom:

```java
static String category(int httpStatus) {
    return switch (httpStatus) {
        case 200, 201 -> "OK";
        case 400, 404 -> "CLIENT_ERROR";
        case 500, 503 -> "SERVER_ERROR";
        default -> "UNKNOWN";
    };
}
```

Pattern matching for `instanceof` (final since Java 16):

```java
static int lengthOf(Object x) {
    if (x instanceof String s) return s.length();
    return -1;
}
```

### 5) Text blocks (final since Java 15)

```java
String json = """
{
  "id": 123,
  "status": "PAID"
}
""";
```

### 6) Records (final since Java 16)

Great for DTOs/value types.

```java
public record Money(String currency, long cents) {
    public Money {
        if (currency == null || currency.isBlank()) throw new IllegalArgumentException("currency");
    }

    public String formatted() {
        return currency + " " + (cents / 100.0);
    }
}
```

### 7) Sealed classes (final since Java 17)

Useful for closed hierarchies and exhaustive switches.

```java
sealed interface PaymentResult permits PaymentResult.Approved, PaymentResult.Declined {
    record Approved(String authId) implements PaymentResult {}
    record Declined(String reason) implements PaymentResult {}
}
```

### 8) Virtual threads (final since Java 21)

Use for high-concurrency I/O-bound workloads (typical microservices).

```java
import java.util.concurrent.Executors;

public class VirtualThreadsExample {
    public static void main(String[] args) throws Exception {
        try (var exec = Executors.newVirtualThreadPerTaskExecutor()) {
            for (int i = 0; i < 10_000; i++) {
                int id = i;
                exec.submit(() -> {
                    // Simulate IO
                    Thread.sleep(10);
                    return "ok-" + id;
                });
            }
        }
        System.out.println("done");
    }
}
```

Staff notes:
- Still keep timeouts, bulkheads, rate limits. Virtual threads help concurrency, not correctness.
- Be aware of blocking calls and thread-local usage (context propagation strategies).

### 9) Structured concurrency (incubator/preview in recent releases)

Idea: treat concurrent tasks as a single unit with clear lifecycle and failure handling.

Example sketch (API may differ by release because it has been incubating):

```java
// Pseudocode-style example of the structured concurrency idea:
//
// try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
//     var user = scope.fork(() -> userService.fetchUser(userId));
//     var orders = scope.fork(() -> orderService.fetchOrders(userId));
//     scope.join().throwIfFailed();
//     return new Profile(user.resultNow(), orders.resultNow());
// }
```

### 10) Foreign Function & Memory (FFM) API (final in recent releases)

Use case: call native libraries safely without JNI ceremony. API details evolve by release; treat this as “know it exists + where it fits”.

---

## Java 9 Highlights

- **JPMS (modules)**
- **JShell**
- **Collection factory methods** (`List.of`, `Set.of`, `Map.of`)
- **Stream API additions** (`takeWhile`, `dropWhile`, `iterate` overload)
- **Flow API** (reactive streams interfaces)

Examples:

```java
var list = java.util.List.of("a", "b", "c"); // immutable
```

```java
java.util.stream.Stream.of(1, 2, 3, 4, 5)
        .takeWhile(n -> n < 4)
        .forEach(System.out::println); // 1,2,3
```

---

## Java 10 Highlights

- **`var`** local variable type inference
- Performance and GC improvements (practical takeaway: upgrade is usually beneficial)

Example:

```java
var userIds = java.util.Set.of(10L, 11L, 12L);
```

---

## Java 11 (LTS) Highlights

- **Standard HTTP Client** (`java.net.http`)
- **New String methods**: `isBlank`, `lines`, `strip`, `repeat`
- **Files convenience methods**: `readString`, `writeString`
- `var` in lambda parameters (readability; use sparingly)

Examples:

```java
System.out.println("  \n".isBlank()); // true
System.out.println("hi".repeat(3));   // hihihi
```

```java
import java.nio.file.Files;
import java.nio.file.Path;

Path p = Path.of("note.txt");
Files.writeString(p, "hello\n");
System.out.println(Files.readString(p));
```

---

## Java 12 Highlights

- Switch expressions (early form; now widely used)
- Useful small API improvements

---

## Java 13 Highlights

- Text blocks (preview at the time; final in 15)

---

## Java 14 Highlights

- Records (preview at the time; final in 16)
- Helpful NPE messages
- Switch improvements continued

---

## Java 15 Highlights

- **Text blocks** (final)
- Sealed classes (preview at the time; final in 17)

---

## Java 16 Highlights

- **Records** (final)
- **Pattern matching for `instanceof`** (final)

---

## Java 17 (LTS) Highlights

- **Sealed classes** (final)
- Strong encapsulation (JPMS-related hardening continues)
- Many JVM/runtime improvements (upgrade-friendly for most services)

Example (sealed + records):

```java
sealed interface Shape permits Circle, Rectangle {}
record Circle(double r) implements Shape {}
record Rectangle(double w, double h) implements Shape {}
```

---

## Java 18 Highlights

- Incremental platform improvements
- Tooling improvements (e.g., a simple HTTP file server was introduced around this era)

---

## Java 19 Highlights

- Early virtual threads preview work (led to Java 21 final)
- Preview/incubator features for concurrency evolution (varies by release)

---

## Java 20 Highlights

- Continued preview/incubator iteration for modern concurrency features

---

## Java 21 (LTS) Highlights

- **Virtual threads** (final)
- **Pattern matching for switch** and **record patterns** became stable around this timeframe (depending on finalization)
- **Sequenced collections** (a big quality-of-life improvement)

Sequenced collections example:

```java
import java.util.ArrayList;
import java.util.List;

List<Integer> xs = new ArrayList<>(List.of(1, 2, 3));
System.out.println(xs.reversed()); // [3, 2, 1]
```

---

## Java 22 Highlights

- Continued modernization: language/runtime + API improvements
- Foreign Function & Memory (FFM) API became production-ready in recent releases (exact stabilization depends on release line)

---

## Java 23 Highlights

- Continued evolution of pattern matching/concurrency-related features and platform APIs
- Typically: fewer “headline” changes, more refinement

---

## Java 24 Highlights

- Continued refinement of the post-21 era (concurrency, language ergonomics, platform APIs)

Staff interview guidance for 23/24:
- Be able to explain why teams adopt **21 LTS** widely and then selectively adopt newer non-LTS features when it’s worth it.

---

## Interview Questions

### Modules
1. Why did JPMS exist? What problems does it solve vs classpath?
2. What is the difference between `exports` and `opens`?

### `var`
1. When does `var` improve readability vs hurt it?

### Records and sealed classes
1. When would you choose records over Lombok?
2. How do sealed hierarchies help with correctness?

### Concurrency (post Java 21)
1. What problems do virtual threads solve? When would you not use them?
2. How do you keep systems safe even with “easy concurrency” (timeouts, bulkheads, limits)?

### API evolution / compatibility
1. How do you roll a Java upgrade across many services safely?
2. What’s your approach to preview/incubator features in production?

Version	Core Feature
Java 9	Module System, Jshell, Stream API enhancements, String memory improvements
Java 10	var, G1 GC, Application class- data sharing (APPcds) improves JVM starup
Java 11	HTTP Client, String APIs
Java 12	Switch Expressions (preview)
Java 13	Text Blocks
Java 14	Records (preview), NPE Improvements
Java 15	Sealed Classes (preview)
Java 16	Pattern Matching
Java 17	LTS – Sealed Classes, ZGC
Java 18	Simple Web Server
Java 19	Virtual Threads (preview)
Java 20	Structured Concurrency
Java 21	LTS – Virtual Threads (final), Pattern Matching, Sequenced Collections




✅ Java 9 (Sep 2017)

1. Project Jigsaw (Module System)

This module system addresses problems related to the scaling of software systems, encapsulation, dependency management, and application packaging.

Key Goals of Project Jigsaw
	1. Make the JDK more modular – Break the JDK into smaller, interoperable modules.
	2. Improve encapsulation – Internal APIs (like sun.*) can be properly hidden.
	3. Support reliable configuration – Detect missing dependencies and conflicts at compile time or startup.
	4. Enable scalable applications – Developers can create applications composed of smaller, well-defined modules.
	5. Enable smaller runtime images – Applications can include only the modules they need, using tools like jlink.

📦 Basic Concepts
	• Module: A self-describing collection of code and data. Each module has:
		○ A module-info.java file declaring the module’s name, dependencies, and exported packages.
	• Exports: Only explicitly exported packages are accessible to other modules.
	• Requires: Used to declare module dependencies.
	• Transitive dependencies: Using requires transitive lets dependent modules inherit dependencies.

module com.example.myapp {
    requires java.sql;
    requires com.example.utils;

    exports com.example.myapp.api;
}

📊 JDK Modularization Example
Before Jigsaw, the entire JDK was a monolith. Now it's broken into modules like:
	• java.base: Core Java APIs (always required, implicitly).
	• java.sql: JDBC and database access.
	• java.xml: XML processing.
Run this to list JDK modules:

java --list-modules

🛠️ Tools Introduced or Enhanced
	• jlink: Create custom runtime images with only the necessary modules.
	• jdeps: Analyze dependencies between classes and modules.
	• jshell: REPL that also understands modules.

📉 Benefits of Using Modules
	• Better encapsulation and security.
	• Reduced startup time and footprint with custom runtimes.
	• Clear dependency management.
	• Prevention of "classpath hell".



	• Breaks JDK into modules: better encapsulation.
	• module-info.java to declare dependencies.
	• ⛔ Common Pitfall: Classpath conflicts in large apps.
	• 🔥 Used in large microservices or platforms with plugin architecture.


2. JShell (REPL)
	• Interactive shell for quick testing.
	• Useful for quick prototyping or DSA practice.

  To start JShell, open a terminal and type:

📘 Basic Usage Examples

jshell> int x = 10;
x ==> 10

jshell> x + 5
$2 ==> 15

jshell> String message = "Hello, JShell!";
message ==> "Hello, JShell!"

jshell> System.out.println(message);
Hello, JShell!


✅ Java 10 (Mar 2018)
1. var Keyword (Local Variable Type Inference)
	• Cleaner syntax, improves readability.

var list = new ArrayList<String>();

When to Use
Use var when the type is obvious from the right-hand side, or when the type is long/complex and clutters the code.

🚫 Limitations
	• Cannot be used for method parameters or return types.
	• Cannot be used for class fields (instance or static variables).
	• Overuse can reduce code readability, especially with complex types.


2. G1 GC Improvements
The G1 (Garbage First) Garbage Collector in Java has seen significant improvements in recent Java versions (especially Java 11 through Java 17 and beyond). These enhancements aim to improve performance, predictability, and memory efficiency. Below are some key G1 GC improvements:

🆕 Key G1 GC Improvements

1. Improved Pause-Time Control
	• Goal: Better adhere to user-specified pause time goals (-XX:MaxGCPauseMillis).
	• How: More accurate prediction and selection of regions to collect, allowing more consistent pause durations.
	• Benefit: More predictable GC behavior for latency-sensitive applications.

2. Garbage-First Heap Region Allocation Enhancements
	• G1's region allocation strategy is smarter now:
		○ Reduces fragmentation.
		○ More efficient young generation sizing.
		○ Better balancing between young and old generation collections.
3. Concurrent Refinement Enhancements
	• Parallel and concurrent refinement threads have been tuned to:
		○ Reduce contention and overhead.
		○ Improve scalability with modern CPUs.
	• Leads to more efficient remembered set (RSet) processing.
4. String Deduplication Improvements
	• G1 GC supports string deduplication (-XX:+UseStringDeduplication) to reduce memory usage.
	• Improved in newer versions:
		○ Smarter heuristics.
		○ Reduced CPU overhead during deduplication.
5. Heap Memory Efficiency
	• G1 now better reclaims Humongous Objects (objects > 50% of a region size).
	• Introduced early reclamation and reduced memory waste from large object allocation.

6. Improved Mixed GC Phases
	• Mixed GCs (young + old collection) are now more efficient:
		○ Old regions are collected in a more adaptive way.
		○ Avoids unnecessary collection work if goals are already met.
7. Better Adaptive Policies
	• Heuristics have been refined to:
		○ Predict pause times more accurately.
		○ Automatically tune parameters for optimal behaviour.

✅ Java 11 (Sep 2018) – LTS

1. HTTP Client API (Standardized)

HttpClient client = HttpClient.newHttpClient();

Replaces HttpURLConnection, supports async, HTTP/2.
 Key Features:
	• Standard API in java.net.http package
	• Supports HTTP/1.1 and HTTP/2
	• Asynchronous (non-blocking) and synchronous calls
	• Uses CompletableFuture and WebSocket support
	• Replaces legacy HttpURLConnection

2. String Enhancements

isBlank, lines(), strip()-remove leading/trailing spaces, stripeLeading() or stripeTrailing(), repeat

3. Removed APIs

Java EE modules like JAXB, JAX-WS removed from JDK.
Soap, xml binding 

✅ Java 12 (Mar 2019)

1. Switch Expression (Preview)

int result = switch (day) {
    case MONDAY -> 1;
    default -> 0;
};

int day = 3;

String result = switch (day) {
    case 1, 2, 3 -> "Weekday";
    case 4, 5    -> "Weekend";
    default      -> "Unknown";
};

Java 12 = Cleaner switch + better Strings + powerful collectors + JVM tuning

Feature	Description
Arrow (->) syntax	No need for break; avoids fall-through bugs
Returns a value	Can assign directly to a variable
Multiple labels	Group cases using commas
Exhaustiveness check	Useful in enum and sealed types

2. Shenandoah GC (Experimental)

✅ Java 13 (Sep 2019)
1. Text Blocks (Preview)

String html = """
    <html>
      <body>Hello</body>
    </html>
    """;
Why Use Text Blocks?
They solve problems with traditional string syntax like:
	• Escaping newlines (\n) and quotes (\")
	• Poor readability of multi-line strings (e.g., JSON, SQL, HTML)

Makes multi-line strings readable.

✅ Java 14 – Modern Syntax (Mar 2020)
Records (Preview)
Java 14 introduced records as a preview feature, providing a compact syntax for declaring classes that are transparent holders for shallowly immutable data.

🔍 What Are Records?
A record is a special kind of class in Java that is:
	• Final (cannot be extended)
	• Automatically provides:
		○ equals()
		○ hashCode()
		○ toString()
		○ Getters for each field
		○ Canonical constructor

✅ Syntax
record Person(String name, int age) {}

This is equivalent to writing:

final class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String name() { return name; }
    public int age() { return age; }

    @Override
    public boolean equals(Object o) { /* ... */ }

    @Override
    public int hashCode() { /* ... */ }

    @Override
    public String toString() { /* ... */ }
}
record Point(int x, int y) {}
 

📦 Key Features
	• Concise syntax for data carriers.
	• Immutable by default (fields are final).
	• Built-in implementations for equals, hashCode, and toString.
	• Can include methods, static fields, and compact constructors.


🧠 Use Cases
DTOs (Data Transfer Objects)
Value classes
Configuration models
Immutable domain models

🔹 Helpful NullPointerExceptions

Java 14 introduced Helpful NullPointerExceptions as an opt-in JVM feature, designed to make debugging easier by showing exactly what was null when a NullPointerException (NPE) occurred.

✅ The Solution in Java 14
Enable Helpful NullPointerExceptions, and the JVM will tell you exactly what part of the expression was null.

🛠 How to Enable
You must enable it explicitly with a JVM flag:

java -XX:+ShowCodeDetailsInExceptionMessages MyApp


🧱 Java 15 (Released: Sep 2020)
🔹 Key Features:
	1. Text Blocks (Standard).
	2. Sealed Classes (Preview):
		○ Restrict which classes can extend a class.
Sealed classes

Java 15 introduced sealed classes as a preview feature, which allows you to control which classes can extend or implement a superclass or interface. This gives you more control over class hierarchies, improves maintainability, and allows the compiler and tools to reason more effectively about the code.

🔒 What are Sealed Classes?

A sealed class or interface restricts which other classes or interfaces may extend or implement it.

public sealed class Shape
    permits Circle, Rectangle, Square {
    // class body
}

public final class Circle extends Shape {
    // class body
}

public non-sealed class Rectangle extends Shape {
    // class body
}

public sealed class Square extends Shape
    permits SmallSquare, BigSquare {
    // class body
}

final class SmallSquare extends Square { }
final class BigSquare extends Square { }

| Keyword      | Description                                                              |
| ------------ | ------------------------------------------------------------------------ |
| `sealed`     | Declares a class/interface with restricted inheritance.                  |
| `permits`    | Lists the allowed subclasses (required unless all permitted are nested). |
| `final`      | Declares that a subclass cannot be further extended.                     |
| `non-sealed` | Allows a subclass to be extended freely (removes sealing from subclass). |

🧠 Why Use Sealed Classes?
	• Restrict extensibility – Know all subclasses at compile time.
	• Improve pattern matching – Switch expressions can become exhaustive.
	• Enhance security and API design – Prevent unintended subclassing.
	• Encourage clear architecture – Only explicitly allowed subclasses.

🔧 Requirements
	• All subclasses must be in the same module (or the same package if modules aren’t used).
	• All permitted subclasses must directly extend or implement the sealed class or interface.

public sealed interface Shape
    permits Circle, Rectangle {
    double area();
}

public final class Circle implements Shape {
    double radius;
    public double area() { return Math.PI * radius * radius; }
}

public final class Rectangle implements Shape {
    double width, height;
    public double area() { return width * height; }
}

With this structure, the compiler knows Shape can only be a Circle or Rectangle, which enables exhaustive checks in pattern matching (especially useful in Java 17+).


🧱 Java 16 (Released: Mar 2021)
🔹 Key Features:
	1. Records (Standard).
	2. Pattern Matching for instanceof (Standard).
	3. JEP 376 – ZGC on macOS and Windows.



1. Records (Standardized)
	Introduced in Java 14 as a preview, finalized in Java 16.

What it is:
A concise way to create data-carrying classes (POJOs) with automatic constructors, accessors, equals/hashCode, toString, etc.
📌 Immutable by default, great for DTOs and functional programming.

2. Pattern Matching for instanceof (Preview)
Makes type checks and casting cleaner.

if (obj instanceof String s) {
    System.out.println(s.toUpperCase());  // no explicit cast
}

✅ No need to cast after checking with instanceof.

3. Sealed Classes (Second Preview)
Improved from Java 15; allows better modeling of restricted class hierarchies.


✅ Java 17: Key Features & Enhancements

1. 🧩 Sealed Classes (Standard – JEP 409)


Purpose: Control which classes or interfaces can extend or implement a class/interface.

2. 🪓 Pattern Matching for switch (Preview – JEP 406)

Adds more flexibility and safety to switch expressions.

static String formatShape(Shape s) {
    return switch (s) {
        case Circle c -> "Circle with radius " + c.radius();
        case Square sq -> "Square with side " + sq.side();
    };
}


5. 🔒 Strong Encapsulation of JDK Internals
Most internal JDK APIs are now strongly encapsulated by default. This breaks access to sun.* classes unless explicitly allowed.
✔ Improves security
❗ Might break older code that relied on internal APIs

6. 🧹 Deprecations & Removals
• Removed: Applets (finally deprecated)
• Removed: RMI Activation System (JEP 407)
• Removed: Experimental AOT and JIT compiler (JEP 410)
• Removed: Security Manager marked for removal (JEP 411)

7. 🚀 Performance & JVM Improvements

• New macOS/AArch64 port (Apple M1 support) — JEP 391
• Better garbage collection (G1, ZGC optimizations)
• Updated Flight Recorder and monitoring tools

⚙️ LTS Benefit
Being an LTS release, Java 17 is the most stable choice for enterprise development until at least Java 21 or 25. It replaces Java 11 as the go-to production version


✅ Java 18 Key Features & Details
1. 🌐 Simple Web Server (JEP 408)
jwebserver --port 8080 --directory .

• Supports HTTP/1.1
• Serves static files (HTML, CSS, JS, etc.)
• No need for external server setup

✅ Java 21 — Key Features & Details

🆕 1. Record Patterns (JEP 440 – Final)
Allows pattern matching directly on record components:

record Point(int x, int y) {}

static void print(Point p) {
    if (p instanceof Point(int x, int y)) {
        System.out.println("x = " + x + ", y = " + y);
    }
}
✔ Combines pattern matching and records
✔ Simplifies destructuring

🆕 2. Pattern Matching for switch (JEP 441 – Final)
Enables type-safe, expressive switch with types and guards:

static String handle(Object obj) {
    return switch (obj) {
        case String s -> "A string: " + s;
        case Integer i when i > 0 -> "Positive int: " + i;
        default -> "Something else";
    };
}

✔ Cleaner and safer switch logic
✔ Great for sealed classes and ADTs
🧩 3. Sealed Classes (JEP 409 – Standard since Java 17)
Fully supported in Java 21 — great with pattern matching.

sealed interface Shape permits Circle, Square {}
final class Circle implements Shape {}
final class Square implements Shape {}

🌐 4. Virtual Threads (JEP 444 – Final, formerly preview)
A massive change to Java’s concurrency model!

Runnable task = () -> System.out.println("Running in thread: " + Thread.currentThread());
Thread.startVirtualThread(task);

✔ Lightweight threads
✔ Enables millions of concurrent tasks
✔ Game-changing for server-side apps (replaces complex thread pools)

Feature	Platform Thread (Thread)	Virtual Thread (Thread.ofVirtual())
Backed by OS thread?	✅ Yes	❌ No (user-mode, managed by JVM)
Memory cost	🧵 High (1MB+ per thread)	🪶 Low (~few KB)
Suitable for blocking?	🚫 No	✅ Yes
Concurrency scale	❌ Limited (10k+)	✅ Huge (1M+)







![Uploading image.png…]()



