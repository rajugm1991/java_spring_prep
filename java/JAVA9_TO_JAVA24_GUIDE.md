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


