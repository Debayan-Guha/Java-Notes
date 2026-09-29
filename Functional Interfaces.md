# Functional Interface

A **Functional Interface** in Java is an interface that contains **exactly one abstract method**. They form the backbone of modern Java programming and lambda expressions. 

Lambda expressions only work with interfaces that have exactly one abstract method. Because there is only one method option, Java can instantly guess which method your lambda expression is trying to implement.

---

# `Runnable`

```java
package java.lang;

@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```
- **Methods:** It contains exactly one abstract method: `void run()`.
- **Input Parameters:** **None** (Accepts no arguments).
- **Output / Return Type:** **`void`** (Returns no value, meaning it cannot return result data or throw checked exceptions directly).
- **Meaning:** It defines a command or a unit of work designed to be executed asynchronously or concurrently by an independent execution thread.
- **When to Use:** Use it when you need to trigger fire-and-forget background operations, spawn parallel worker threads, or offload tasks where a return status or value is not required.
  - *Creating Threads:* Spawning a separate, independent worker thread to run code alongside your main program.
  - *Background Tasks (BG Tasks):* Offloading time-consuming jobs like downloading a file or saving data to a database so the main application stays fast and responsive.
  - *Fire-and-Forget Operations:* Executing non-critical tasks that you do not need to wait for, such as sending a notification or generating an activity log.
  - *Timed Events:* Running a recurring maintenance job, such as auto-saving a user's work-in-progress draft every 60 seconds.

**How to Use Runnable**

1. **Traditional Anonymous Inner Class (Legacy Approach)**

Before Java 8, you had to implement the interface using an anonymous class:

```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println("Running task using anonymous class...");
    }
};

Thread thread = new Thread(task);
thread.start();
```

2. **Modern Lambda Expression (Java 8+ Approach)**

Because Runnable is a functional interface, you can replace the bulky anonymous class with a clean, concise lambda expression:

```java
Runnable task = () -> System.out.println("Running task using a Lambda Expression!");

Thread thread = new Thread(task);
thread.start();
```

**3. Inline Direct Execution**

You can even pass the lambda expression directly into the Thread constructor without assigning it to a variable first:

```java
new Thread(() -> {
    System.out.println("Thread is running inline code!");
}).start();
```

4. **Production Context (Centralized Executor Wrapper)**
In enterprise systems, `Runnable` represents a raw, standalone command package passed as a parameter into a global protection framework.
```java
// Centralized Wrapper Method
public static void redisSave(Runnable action, String serviceName, String methodName) {
    try {
        action.run(); // Triggers the void command block inside a safety net
    } catch (Exception RedisEx) {
        logger.warn("Redis SAVE failed for method: {}", methodName);
    }
}

// Inline Execution Call
OperationExecutor.redisSave(
    () -> cacheService.saveToCache(cacheKey, data), 
    "BillService", "saveBillData"
);
```

---

# `Supplier<T>`

```java
package java.util.function;

@FunctionalInterface
public interface Supplier<T> {
    T get();
}
```

- **Methods:** It contains exactly one abstract method: `T get()`.
- **Input Parameters:** **None** (Accepts no arguments).
- **Output / Return Type:** **`T`** (Returns an object of generic type `T`).
- **Meaning:** It represents a data provider or factory that produces a value out of thin air whenever it is called, without needing any input information.
- **When to Use:** Use it when you need to generate or provide values, objects, or data without passing any inputs.
  - *Generating Random Values:* Creating unique, dynamic runtime data on demand, such as random numbers or unique IDs (UUIDs).
  - *Lazy Loading (Delayed Execution):* Waiting to create an expensive object or data value until the exact moment it is actually needed, saving memory and CPU time.
  - *Factory Patterns:* Serving as a blueprint factory to instantiate and supply new object instances whenever requested.
  - *Default Fallbacks:* Providing a backup or default value when a primary database, file, or cache lookup returns empty.

**How to Use Supplier**

1. **Traditional Anonymous Inner Class (Legacy Approach)**

Before Java 8, you had to implement the interface using an anonymous class:

```java
import java.util.function.Supplier;

Supplier<Double> randomSupplier = new Supplier<Double>() {
    @Override
    public Double get() {
        return Math.random();
    }
};

System.out.println("Random Number: " + randomSupplier.get());
```

2. **Modern Lambda Expression (Java 8+ Approach)**

Because Supplier is a functional interface, you can replace the bulky anonymous class with a clean, concise lambda expression:

```java
import java.util.function.Supplier;

Supplier<Double> randomSupplier = () -> Math.random();

System.out.println("Random Number: " + randomSupplier.get());
```

**3. Inline Direct Execution**

You can pass the lambda expression directly into methods like `Optional.orElseGet()` to supply a fallback value dynamically:

```java
import java.util.Optional;

String username = Optional.<String>empty()
    .orElseGet(() -> "Guest_" + System.currentTimeMillis());

System.out.println("Active User: " + username);
```

4. **Production Context (Circuit Breaker Fallback Logic)**
Enterprise platforms wrap retrieval logic using a `Supplier` so that if the main server goes down, the handler traps the crash and safely unwraps the supplier to fetch stale local data instead.
```java
// Centralized Wrapper Method 
public static <T> T redisGet(Supplier<T> action, String serviceName, String methodName) {
    try {
        return action.get(); // Triggers the data retrieval lambda inside a try-catch block
    } catch (Exception RedisEx) {
        logger.warn("Redis GET failed for method: {}", methodName);
    }
    return null; // Graceful degradation fallback
}

// Circuit Breaker Fallback Consumption
CachedResponse cached = OperationExecutor.redisGet(
    () -> cacheService.getFromCache(cacheKey, CachedResponse.class), 
    "BillService", "getBillById"
);
```

---


# `Consumer<T>`

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);

    default Consumer<T> andThen(Consumer<T> after) {
        Objects.requireNonNull(after);
        return (T t) -> { accept(t); after.accept(t); };
    }
}
```

- **Methods:** It contains exactly one abstract method: `void accept(T t)`. It also contains a default method `andThen(Consumer<? super T> after)` used for chaining multiple consumer operations sequentially.
- **Input Parameters:** **One** (Accepts a single argument of generic type `T`).
- **Output / Return Type:** **`void`** (Processes the data but returns no result).
- **Meaning:** It represents an operation that takes a piece of data, consumes it, and performs some action or side-effect with it (like printing, saving, or logging) without modifying the original reference or returning any data back.
- **When to Use:** Use it when you need to iterate through data collections, process incoming arguments, or perform side-effect actions where no return value is expected.
  - *Data Iteration (`forEach`):* Iterating through a list or stream of elements to process or display each item individually.
  - *Audit Logging:* Passing an event object to a logger system to record user activities or transactional states without interrupting the main application logic.
  - *Metric & Analytics Tracking:* Shipping data payloads or system events off to monitoring systems (like Prometheus or New Relic) for live telemetry tracking.
  - *Data Notifications:* Sending an incoming notification payload directly out to an external communication channel (such as a Slack webhook or an SMS gateway).
  - *Enterprise Architecture (Post-Processing Pipelines):* Passing success or failure data objects into a tracking controller *after* a database operation completes, allowing the system to run hooks like metrics tracking or sending confirmation notifications dynamically.

**How to Use Consumer**

1. **Traditional Anonymous Inner Class (Legacy Approach)**
Before Java 8, you had to implement the interface using an anonymous class:
```java
import java.util.function.Consumer;

Consumer<String> printConsumer = new Consumer<String>() {
    @Override
    public void accept(String message) {
        System.out.println("Processing notification: " + message);
    }
};

printConsumer.accept("Server started successfully.");
```

2. **Modern Lambda Expression (Java 8+ Approach)**
Because Consumer is a functional interface, you can replace the bulky anonymous class with a clean, concise lambda expression:
```java
import java.util.function.Consumer;

Consumer<String> printConsumer = message -> System.out.println("Processing notification: " + message);

printConsumer.accept("Server started successfully.");
```

3. **Inline Direct Execution**
You can pass the lambda expression directly into collection methods like `Iterable.forEach()` to process stream items inline:
```java
import java.util.Arrays;
import java.util.List;

List<String> logs = Arrays.asList("WARN: Low Memory", "ERROR: DB Timeout", "INFO: Job Completed");

// Inline processing using Consumer lambda inside forEach
logs.forEach(logLine -> System.out.println("Log Delivery -> " + logLine));
```

4. **Production Context (Centralized Metrics & Audit Logging Hook)**
Enterprise architectures often use a `Consumer` inside execution wrappers. After a database or cache action successfully returns data, the wrapper hands that data to a consumer to track metrics or execute auditing asynchronously.
```java
// Centralized Wrapper Method incorporating a Consumer for Post-Processing Metrics
public static <T> T dbGetWithMetrics(Supplier<T> action, Consumer<T> metricsHook, String serviceName) {
    try {
        T result = action.get(); // 1. Fetch data via Supplier
        if (result != null) {
            metricsHook.accept(result); // 2. Pass data to Consumer for side-effect metrics tracking
        }
        return result;
    } catch (Exception e) {
        logger.error("DB GET FAILED for service: {}", serviceName);
        throw new DatabaseException();
    }
}

// Enterprise Call: Fetches billing data and passes a Consumer lambda to publish telemetry
BillDetailsResponse response = OperationExecutor.dbGetWithMetrics(
    () -> billRepository.findById(billId),
    billData -> metricsCollector.incrementCounter("bill.fetch.success", "tier", billData.getTierType()),
    "BillService"
);
```

---


# `BiConsumer<T, U>`

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface BiConsumer<T, U> {

    void accept(T t, U u);

    default BiConsumer<T, U> andThen(BiConsumer<? super T, ? super U> after) {
        Objects.requireNonNull(after);
        return (l, r) -> { accept(l, r); after.accept(l, r); };
    }
}
```

---

- **Methods:** It contains exactly one abstract method: `void accept(T t, U u)`. It also contains a default method `andThen(BiConsumer<? super T, ? super U> after)` used for chaining multiple bi-consumer operations sequentially.
- **Input Parameters:** **Two** (Accepts two arguments: the first of generic type `T` and the second of generic type `U`).
- **Output / Return Type:** **`void`** (Processes both pieces of data but returns no result).
- **Meaning:** It represents an operation that takes two distinct input values, consumes them together, and performs a combined action or side-effect (like logging a key-value pair, processing coordinates, or updating a map tracking context) without returning any data back.
- **When to Use:** Use it when you need to process pairs of associated data, map elements, or contextualized events where no return value is expected.
  - *Map Iteration (`Map.forEach`):* Processing both the keys and values of a `Map` simultaneously inside a loop.
  - *Contextual Tracking:* Consuming a transactional payload alongside its execution context metadata (like a request ID or timestamp) to print comprehensive audit logs.
  - *Key-Value Configurations:* Passing configuration settings where a property name and its value must be processed or bound into a service together.
  - *Error Reporting:* Handling a dynamic business object alongside a thrown exception to build complex error alerts.
  - *Enterprise Architecture (Flexible Multi-Context Wrappers):* Using execution wrappers where you need to pass an operation's response data alongside processing telemetry (such as execution duration or method names) to specialized reporting frameworks.

**How to Use BiConsumer**

1. **Traditional Anonymous Inner Class (Legacy Approach)**
Before Java 8, you had to implement the interface using an anonymous class:
```java
import java.util.function.BiConsumer;

BiConsumer<String, Integer> logMapEntry = new BiConsumer<String, Integer>() {
    @Override
    public void accept(String key, Integer value) {
        System.out.println("Processing -> Key: " + key + " | Value: " + value);
    }
};

logMapEntry.accept("billId_101", 500);
```

2. **Modern Lambda Expression (Java 8+ Approach)**
Because BiConsumer is a functional interface, you can replace the bulky anonymous class with a clean, concise lambda expression:
```java
import java.util.function.BiConsumer;

BiConsumer<String, Integer> logMapEntry = (key, value) -> 
    System.out.println("Processing -> Key: " + key + " | Value: " + value);

logMapEntry.accept("billId_101", 500);
```

3. **Inline Direct Execution**
You can pass the lambda expression directly into map operations like `Map.forEach()` to process keys and values inline:
```java
import java.util.HashMap;
import java.util.Map;

Map<String, String> statusMap = new HashMap<>();
statusMap.put("bill_01", "PAID");
statusMap.put("bill_02", "PENDING");

// Inline processing using BiConsumer lambda inside forEach
statusMap.forEach((billId, status) -> 
    System.out.println("Alert: Bill " + billId + " is currently " + status)
);
```

4. **Production Context (Centralized Performance & Telemetry Tracking Hook)**
Enterprise architectures often use a `BiConsumer` inside wrapper methods to cleanly pass back a primary data result paired with runtime metadata (like execution timing) to separate logging or tracking frameworks.
```java
// Centralized Wrapper Method using a BiConsumer to log execution details after completion
public static <T> T dbGetWithTiming(Supplier<T> action, BiConsumer<T, Long> telemetryHook, String serviceName) {
    long startTime = System.currentTimeMillis();
    try {
        T result = action.get(); // 1. Fetch data via Supplier
        long duration = System.currentTimeMillis() - startTime;
        
        // 2. Pass both the result data AND the duration context to the BiConsumer
        telemetryHook.accept(result, duration); 
        
        return result;
    } catch (Exception e) {
        logger.error("DB GET FAILED for service: {}", serviceName);
        throw new DatabaseException();
    }
}

// Enterprise Call: Fetches database entries and logs the response payload paired with execution speed
BillDetailsResponse response = OperationExecutor.dbGetWithTiming(
    () -> billRepository.findById(billId),
    (billData, timeTaken) -> logger.info("Bill ID: {} processed in {} ms", billData.getId(), timeTaken),
    "BillService"
);
```

---


## `Function<T, R>`

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Function<T, R> {

    R apply(T t);

    default <V> Function<V, R> compose(Function<? super V, ? extends T> before) {
        Objects.requireNonNull(before);
        return (V v) -> apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(Function<? super R, ? extends V> after) {
        Objects.requireNonNull(after);
        return (T t) -> after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

- **Methods:** It contains exactly one abstract method: `R apply(T t)`. It also contains default methods `compose` and `andThen` for pipeline chaining, alongside a static `identity()` method.
- **Input Parameters:** **One** (Accepts a single argument of generic type `T`).
- **Output / Return Type:** **`R`** (Returns a transformed result object of generic type `R`).
- **Meaning:** It represents a transformer that takes an input value, processes it, and converts it into a completely different type or value as an output.
- **When to Use:** Use it when you need to transform, convert, map, or map-reduce objects from one state/class to another.
  - *Data Transformation (`Stream.map`):* Converting data models, such as turning an input collection of user entities into a collection of primitive user names.
  - *Data Parsing:* Converting incoming text configurations or payloads into primitive structures (e.g., converting a numerical String input into an `Integer`).
  - *DTO Mapping:* Transforming database domain entities directly into secure outbound API data transfer objects (DTOs).
  - *Encryption / Hashing:* Taking a plain text password argument and running it through a transformation algorithm to yield a hashed string out.
  - *Enterprise Architecture (Centralised Processing Pipelines):* Passing data conversion actions (like mapping entities to DTOs) down into an execution block that wraps the processing with standardized exception logging or metrics collection.

**How to Use Function**

1. **Traditional Anonymous Inner Class (Legacy Approach)**
Before Java 8, you had to implement the interface using an anonymous class:
```java
import java.util.function.Function;

Function<String, Integer> stringLength = new Function<String, Integer>() {
    @Override
    public Integer apply(String text) {
        return text.length();
    }
};

System.out.println("Length: " + stringLength.apply("Decode")); // Output: 6
```

2. **Modern Lambda Expression (Java 8+ Approach)**
Because Function is a functional interface, you can replace the bulky anonymous class with a clean, concise lambda expression:
```java
import java.util.function.Function;

Function<String, Integer> stringLength = text -> text.length();

System.out.println("Length: " + stringLength.apply("Decode")); // Output: 6
```

3. **Inline Direct Execution**
You can pass the lambda expression directly into collection streaming frameworks like `Stream.map()` to process transformations inline:
```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

List<String> names = Arrays.asList("alex", "brian", "charles");

// Inline data mapping transformation using a Function lambda
List<String> uppercaseNames = names.stream()
    .map(name -> name.toUpperCase())
    .collect(Collectors.toList());
```

4. **Production Context (Centralised DTO Mapping Wrapper)**
Enterprise platforms pass transformation blocks as a `Function` parameter inside centralized wrappers. If a mapping tool crashes due to null pointers, the handler traps the exception globally and yields a standard business exception instead of leaking system stacks.
```java
// Centralised Execution Wrapper inside OperationExecutor
public static <T, R> R map(Function<T, R> transformer, T sourceData, String serviceName, String methodName) {
    try {
        return transformer.apply(sourceData); // Triggers the transformation mapping block
    } catch (Exception MapEx) {
        logger.error("DTO Mapping FAILED: {} | Service: {} | Method: {}", 
            MapEx.getMessage(), serviceName, methodName);
        throw new DataProcessingException(); // Standard corporate exception fallback
    }
}

// Enterprise Call: Maps a raw database entity into a clean response DTO safely
BillDetailsResponse response = OperationExecutor.map(
    entity -> new BillDetailsResponse(entity.getId(), entity.getAmount()),
    billEntity,
    "BillService", "getBillById"
);
```













