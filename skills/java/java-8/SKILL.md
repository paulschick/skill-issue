---
name: java-8
description: >
  Java 8 language patterns and version constraints for legacy codebases.
  Trigger: When writing Java 8 code, when targeting Java 8 runtime, when working in a Java 8 codebase.
metadata:
  author: paulschick
  version: "1.0"
---

## When to Use

Load this skill when:

- Working in a Java 8 codebase
- Targeting Java 8 runtime
- Project uses `sourceCompatibility = '1.8'` or `<maven.compiler.source>1.8</maven.compiler.source>`
- Any Java project where Java 9+ features are not available

## Version Constraints

These features are NOT available in Java 8. Do not use them.

| Banned Feature                             | Introduced In     | Java 8 Alternative                                    |
|--------------------------------------------|-------------------|-------------------------------------------------------|
| `var`                                      | Java 10 (JEP 286) | Explicit type declarations                            |
| `var` in lambdas                           | Java 11 (JEP 323) | Explicit types or inferred without `var`              |
| `List.of()`, `Map.of()`, `Set.of()`        | Java 9            | `Arrays.asList()`, `Collections.unmodifiableList()`   |
| Records                                    | Java 16 (JEP 395) | Final classes with manual equals/hashCode/toString    |
| Sealed classes                             | Java 17 (JEP 409) | Abstract classes or interfaces with convention        |
| Text blocks (`"""`)                        | Java 15 (JEP 378) | String concatenation or `String.join()`               |
| Switch expressions                         | Java 14 (JEP 361) | Traditional switch statements                         |
| Pattern matching `instanceof`              | Java 16 (JEP 394) | Explicit cast after instanceof check                  |
| Pattern matching switch                    | Java 21 (JEP 441) | if-else chains or visitor pattern                     |
| Modules (`module-info.java`)               | Java 9 (JSR 376)  | Classpath-based dependencies                          |
| Virtual threads                            | Java 21           | Thread pools via `ExecutorService`                    |
| `Stream.toList()`                          | Java 16           | `Collectors.toList()`                                 |
| `String.isBlank()`, `.strip()`, `.lines()` | Java 11           | `trim()`, manual checks                               |
| `Map.entry()`                              | Java 9            | `new AbstractMap.SimpleEntry<>()`                     |
| `Optional.ifPresentOrElse()`               | Java 9            | `if (opt.isPresent()) ... else ...`                   |
| `CompletableFuture.completeOnTimeout()`    | Java 9            | Manual timeout handling                               |
| `List.copyOf()`, `Map.copyOf()`            | Java 10           | `Collections.unmodifiableList(new ArrayList<>(list))` |

## Critical Patterns

### Pattern 1: Lambdas and functional interfaces

Use lambdas over anonymous inner classes. Use standard functional interfaces from `java.util.function`: `Function`,
`Predicate`, `Supplier`, `Consumer`, `BiFunction`, `UnaryOperator`.

```java
// Good: lambda with functional interface
Predicate<String> isNotEmpty = s -> s != null && !s.trim().isEmpty();
Function<String, String> toUpper = String::toUpperCase;
List<String> filtered = names.stream()
    .filter(isNotEmpty)
    .map(toUpper)
    .collect(Collectors.toList());
```

### Pattern 2: Streams for collection processing

Use the Stream API for filtering, mapping, and collecting. Always terminate with `Collectors.toList()`, never
`.toList()`.

```java
Map<String, List<Employee>> byDepartment = employees.stream()
    .filter(e -> e.getSalary() > 50000)
    .collect(Collectors.groupingBy(Employee::getDepartment));
```

### Pattern 3: Optional for nullable returns

Use `Optional<T>` for method return values that may be absent. Do NOT use Optional for fields, constructor parameters,
or method parameters.

```java
public Optional<User> findByEmail(String email) {
    User user = userMap.get(email);
    return Optional.ofNullable(user);
}

// Consuming Optional
String name = findByEmail("test@example.com")
    .map(User::getName)
    .orElse("Unknown");
```

### Pattern 4: java.time over Date/Calendar

Use `LocalDate`, `LocalDateTime`, `ZonedDateTime`, `Instant`, `Duration`, `Period`. Never use `java.util.Date` or
`java.util.Calendar` for new code.

```java
LocalDate today = LocalDate.now();
LocalDate deadline = today.plusWeeks(2);
Duration timeout = Duration.ofSeconds(30);
Instant timestamp = Instant.now();
```

### Pattern 5: CompletableFuture for async

Use `CompletableFuture` for composable async operations. Chain with `thenApply`, `thenCompose`, `thenAccept`,
`exceptionally`.

```java
CompletableFuture<String> future = CompletableFuture
    .supplyAsync(() -> fetchData(url), executor)
    .thenApply(data -> parse(data))
    .exceptionally(ex -> {
        logger.error("Fetch failed", ex);
        return defaultValue;
    });
```

### Pattern 6: Try-with-resources

Always use try-with-resources for `AutoCloseable` resources.

```java
try (BufferedReader reader = new BufferedReader(new FileReader(path))) {
    return reader.lines()
        .filter(line -> !line.trim().isEmpty())
        .collect(Collectors.toList());
}
```

### Pattern 7: Diamond operator

Use `<>` on the right side of generic instantiations.

```java
Map<String, List<String>> index = new HashMap<>();
List<Employee> employees = new ArrayList<>();
```

### Pattern 8: Unmodifiable collections

Use `Collections.unmodifiableList()`, `Collections.unmodifiableMap()`, `Collections.unmodifiableSet()`. Use
`Arrays.asList()` for fixed-content lists.

```java
// Fixed-content list
List<String> statuses = Arrays.asList("ACTIVE", "INACTIVE", "PENDING");

// Defensive copy as unmodifiable
List<String> safeList = Collections.unmodifiableList(new ArrayList<>(mutableList));
Map<String, Integer> safeMap = Collections.unmodifiableMap(new HashMap<>(mutableMap));
```

## Code Examples

### Example 1: Value object (Java 8 equivalent of a record)

```java
package com.example.model;

import java.util.Objects;

public final class Email {

    private final String value;

    public Email(String value) {
        if (value == null || !value.contains("@")) {
            throw new IllegalArgumentException("Invalid email: " + value);
        }
        this.value = value;
    }

    public String getValue() {
        return value;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Email email = (Email) o;
        return Objects.equals(value, email.value);
    }

    @Override
    public int hashCode() {
        return Objects.hash(value);
    }

    @Override
    public String toString() {
        return "Email{value='" + value + "'}";
    }
}
```

### Example 2: Stream pipeline with collectors

```java
package com.example.reporting;

import java.util.Arrays;
import java.util.Collections;
import java.util.List;
import java.util.Map;
import java.util.stream.Collectors;

public final class SalesReport {

    public Map<String, Double> averageSaleByRegion(List<Sale> sales) {
        return sales.stream()
            .filter(s -> s.getAmount() > 0)
            .collect(Collectors.groupingBy(
                Sale::getRegion,
                Collectors.averagingDouble(Sale::getAmount)
            ));
    }

    public List<String> topRegions(List<Sale> sales, double threshold) {
        return sales.stream()
            .collect(Collectors.groupingBy(
                Sale::getRegion,
                Collectors.summingDouble(Sale::getAmount)
            ))
            .entrySet().stream()
            .filter(e -> e.getValue() > threshold)
            .map(Map.Entry::getKey)
            .collect(Collectors.toList());
    }
}
```

### Example 3: CompletableFuture composition with error handling

```java
package com.example.service;

import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;

public final class OrderService {

    private final ExecutorService executor = Executors.newFixedThreadPool(4);

    public CompletableFuture<OrderResult> processOrder(OrderRequest request) {
        return CompletableFuture
            .supplyAsync(() -> validateOrder(request), executor)
            .thenCompose(validated -> CompletableFuture
                .supplyAsync(() -> chargePayment(validated), executor))
            .thenApply(charged -> fulfillOrder(charged))
            .exceptionally(ex -> OrderResult.failure(ex.getMessage()));
    }

    private OrderRequest validateOrder(OrderRequest request) {
        if (request.getItems().isEmpty()) {
            throw new IllegalArgumentException("Order must have at least one item");
        }
        return request;
    }

    private OrderRequest chargePayment(OrderRequest request) {
        // payment processing logic
        return request;
    }

    private OrderResult fulfillOrder(OrderRequest request) {
        return OrderResult.success(request.getOrderId());
    }
}
```

### Example 4: Unmodifiable collections the Java 8 way

```java
package com.example.config;

import java.util.Arrays;
import java.util.Collections;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public final class AppConfig {

    // Fixed-content list (do NOT use List.of())
    private static final List<String> VALID_ENVS =
        Collections.unmodifiableList(Arrays.asList("dev", "staging", "prod"));

    // Fixed-content map (do NOT use Map.of())
    private static final Map<String, Integer> DEFAULTS;
    static {
        Map<String, Integer> map = new HashMap<>();
        map.put("timeout", 30);
        map.put("retries", 3);
        map.put("poolSize", 10);
        DEFAULTS = Collections.unmodifiableMap(map);
    }

    public static List<String> getValidEnvironments() {
        return VALID_ENVS;
    }

    public static Map<String, Integer> getDefaults() {
        return DEFAULTS;
    }
}
```

## Anti-Patterns

### Don't: Use `var` (Java 10+)

```java
// BAD — not available in Java 8
var name = "Alice";
var users = new ArrayList<String>();
```

```java
// GOOD — explicit types
String name = "Alice";
List<String> users = new ArrayList<>();
```

### Don't: Use `List.of()` / `Map.of()` / `Set.of()` (Java 9+)

```java
// BAD — not available in Java 8
List<String> items = List.of("a", "b", "c");
Map<String, Integer> scores = Map.of("alice", 100, "bob", 85);
```

```java
// GOOD — Java 8 alternatives
List<String> items = Collections.unmodifiableList(Arrays.asList("a", "b", "c"));
Map<String, Integer> scores;
Map<String, Integer> tmp = new HashMap<>();
tmp.put("alice", 100);
tmp.put("bob", 85);
scores = Collections.unmodifiableMap(tmp);
```

### Don't: Use records (Java 16+)

```java
// BAD — not available in Java 8
public record Point(int x, int y) {}
```

```java
// GOOD — final class with manual implementations
public final class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    public int getX() { return x; }
    public int getY() { return y; }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        Point point = (Point) o;
        return x == point.x && y == point.y;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }

    @Override
    public String toString() {
        return "Point{x=" + x + ", y=" + y + "}";
    }
}
```

### Don't: Use text blocks (Java 15+)

```java
// BAD — not available in Java 8
String json = """
    {
        "name": "Alice",
        "age": 30
    }
    """;
```

```java
// GOOD — string concatenation
String json = "{\n"
    + "    \"name\": \"Alice\",\n"
    + "    \"age\": 30\n"
    + "}";
```

### Don't: Use `Stream.toList()` (Java 16+)

```java
// BAD — not available in Java 8
List<String> names = people.stream()
    .map(Person::getName)
    .toList();
```

```java
// GOOD — Collectors.toList()
List<String> names = people.stream()
    .map(Person::getName)
    .collect(Collectors.toList());
```

### Don't: Use `String.isBlank()` (Java 11+)

```java
// BAD — not available in Java 8
if (input.isBlank()) { ... }
```

```java
// GOOD — trim().isEmpty()
if (input == null || input.trim().isEmpty()) { ... }
```

## Quick Reference

| Task                   | Pattern                                                       |
|------------------------|---------------------------------------------------------------|
| Immutable value object | Final class, private final fields, equals/hashCode/toString   |
| Filter and collect     | `stream().filter(...).collect(Collectors.toList())`           |
| Group by key           | `Collectors.groupingBy(...)`                                  |
| Nullable return        | `Optional.ofNullable(value)`                                  |
| Consume optional       | `.map(...).orElse(default)`                                   |
| Fixed list             | `Collections.unmodifiableList(Arrays.asList(...))`            |
| Fixed map              | Static block + `Collections.unmodifiableMap(...)`             |
| Async chain            | `CompletableFuture.supplyAsync(...).thenApply(...)`           |
| Date/time              | `LocalDate.now()`, `Instant.now()`, `Duration.ofSeconds(...)` |
| Resource cleanup       | `try (Resource r = ...) { ... }`                              |
| Check blank string     | `str == null \|\| str.trim().isEmpty()`                       |

## Resources

- Java 8 API: https://docs.oracle.com/javase/8/docs/api/
- Java 8 Language Spec: https://docs.oracle.com/javase/specs/jls/se8/html/index.html
- JEP Index: https://openjdk.org/jeps/0
