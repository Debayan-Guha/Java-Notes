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















