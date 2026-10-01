# Table

| Interface Name | Abstract Method | Default Methods (Chaining) | Static Methods | Input Parameters | Output / Return Type | Simplified Meaning (Rule of Thumb) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **`Runnable`** | `void run()` | *None* | *None* | **None** | **`void`** | **Just go do something.** (No inputs, no outputs). |
| **`Supplier<T>`** | `T get()` | *None* | *None* | **None** | **`T`** | **Give me something.** (No inputs, just returns a value). |
| **`Consumer<T>`** | `void accept(T t)` | `andThen(Consumer)` | *None* | **One** (`T`) | **`void`** | **Do something with this one thing.** (Takes an input, returns nothing). |
| **`BiConsumer<T, U>`** | `void accept(T t, U u)` | `andThen(BiConsumer)` | *None* | **Two** (`T`, `U`) | **`void`** | **Do something with these two things.** (Takes two inputs, returns nothing). |
| **`Function<T, R>`** | `R apply(T t)` | `andThen(Function)` <br> `compose(Function)` | `identity()` | **One** (`T`) | **`R`** | **Take this thing and change it into that thing.** (Transforms one input into an output). |
| **`BiFunction<T, U, R>`** | `R apply(T t, U u)` | `andThen(Function)` | *None* | **Two** (`T`, `U`) | **`R`** | **Take these two things and combine/change them into a new thing.** (Transforms two inputs into one output). |
| **`Predicate<T>`** | `boolean test(T t)` | `and(Predicate)` <br> `or(Predicate)` <br> `negate()` | `isEqual(Object)` <br> `not(Predicate)`* | **One** (`T`) | **`boolean`** | **Check if this one thing matches a condition.** (Takes an input, returns true or false). |
| **`BiPredicate<T, U>`** | `boolean test(T t, U u)` | `and(BiPredicate)` <br> `or(BiPredicate)` <br> `negate()` | *None* | **Two** (`T`, `U`) | **`boolean`** | **Check if these two things match a relationship condition.** (Takes two inputs, returns true or false). |



---


# Functional Interfaces

A **Functional Interface** in Java is an interface that contains **exactly one abstract method**. They form the backbone of modern Java programming and lambda expressions.

Lambda expressions only work with interfaces that have exactly one abstract method. Because there is only one abstract method option, Java can instantly understand which method the lambda expression is trying to implement.

---

# `Runnable`

```java
package java.lang;

@FunctionalInterface
public interface Runnable {
    public abstract void run();
}
```

## Methods

- **Abstract Method:** `void run()`
- **Default Methods:** None
- **Static Methods:** None
- **Input Parameters:** **None**
- **Output / Return Type:** **`void`**

## Meaning

`Runnable` represents a command or unit of work that can be executed without requiring an input or returning a result.

## When to Use

Use `Runnable` when you need to trigger an operation where no input or return value is required.

### Common Examples

- **Creating Threads:** Spawning a separate, independent worker thread to run code alongside the main program.
- **Background Tasks:** Offloading time-consuming jobs like downloading a file or saving data to a database.
- **Fire-and-Forget Operations:** Executing non-critical tasks such as sending a notification or generating an activity log.
- **Timed Events:** Running a recurring maintenance job, such as auto-saving a user's work-in-progress draft every 60 seconds.

## How to Use `Runnable`

### Method: `run()`

The `run()` method contains the actual task that the `Runnable` represents.

```java
Runnable task = () ->
    System.out.println("Running task");

task.run();
```

Output:

```text
Running task
```

### Important: `run()` vs `Thread.start()`

Calling `run()` directly executes the code in the current thread.

```java
Runnable task = () ->
    System.out.println("Running task");

task.run();
```

To actually start a new thread:

```java
Runnable task = () ->
    System.out.println("Running task");

Thread thread = new Thread(task);
thread.start();
```

Remember:

```text
run()
→ Executes the task directly in the current thread.

start()
→ Starts a new thread and executes the Runnable there.
```

---

## 1. Traditional Anonymous Inner Class

Before Java 8, you had to implement the interface using an anonymous class:

```java
Runnable task = new Runnable() {
    @Override
    public void run() {
        System.out.println(
            "Running task using anonymous class..."
        );
    }
};

Thread thread = new Thread(task);
thread.start();
```

---

## 2. Modern Lambda Expression

Because `Runnable` is a functional interface, the bulky anonymous class can be replaced with a lambda expression:

```java
Runnable task = () ->
    System.out.println(
        "Running task using a Lambda Expression!"
    );

Thread thread = new Thread(task);
thread.start();
```

---

## 3. Inline Direct Execution

You can pass the lambda expression directly into the `Thread` constructor:

```java
new Thread(() -> {
    System.out.println(
        "Thread is running inline code!"
    );
}).start();
```

---

## 4. Production Context

In enterprise applications, `Runnable` can represent an operation that is passed into a centralized wrapper.

```java
public static void redisSave(
        Runnable action,
        String serviceName,
        String methodName) {

    try {
        action.run();

    } catch (Exception e) {

        logger.warn(
            "Redis SAVE failed for method: {}",
            methodName
        );
    }
}
```

Usage:

```java
OperationExecutor.redisSave(
    () -> cacheService.saveToCache(cacheKey, data),
    "BillService",
    "saveBillData"
);
```

Here:

```text
Lambda
   ↓
Runnable
   ↓
action.run()
   ↓
Actual operation
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

## Methods

- **Abstract Method:** `T get()`
- **Default Methods:** None
- **Static Methods:** None
- **Input Parameters:** **None**
- **Output / Return Type:** **`T`**

## Meaning

`Supplier<T>` represents a data provider or factory that produces a value whenever it is called, without requiring any input.

```text
No Input
   ↓
Supplier
   ↓
Value
```

## When to Use

Use `Supplier` when you need to **generate or provide a value without passing any input**.

### Common Examples

- **Generating Random Values:** Creating unique, dynamic runtime data such as random numbers or unique IDs.
- **Lazy Loading:** Waiting to create an expensive object or data value until the exact moment it is needed.
- **Factory Patterns:** Serving as a blueprint factory to instantiate and supply new object instances whenever requested.
- **Default Fallbacks:** Providing a backup or default value when a primary database, file, or cache lookup returns empty.

## How to Use `Supplier`

### Method: `get()`

The `get()` method executes the Supplier and returns the supplied value.

```java
Supplier<String> supplier =
    () -> "Hello Java";

String value = supplier.get();

System.out.println(value);
```

Output:

```text
Hello Java
```

The Supplier does not execute when it is created:

```java
Supplier<String> supplier =
    () -> "Hello Java";
```

The code executes when:

```java
supplier.get();
```

is called.

---

## 1. Traditional Anonymous Inner Class

Before Java 8:

```java
import java.util.function.Supplier;

Supplier<Double> randomSupplier =
    new Supplier<Double>() {

        @Override
        public Double get() {
            return Math.random();
        }
    };

System.out.println(
    randomSupplier.get()
);
```

---

## 2. Modern Lambda Expression

```java
import java.util.function.Supplier;

Supplier<Double> randomSupplier =
    () -> Math.random();

System.out.println(
    randomSupplier.get()
);
```

---

## 3. Inline Direct Execution

A Supplier can be passed directly into methods such as `Optional.orElseGet()`:

```java
import java.util.Optional;

String username =
    Optional.<String>empty()
        .orElseGet(
            () -> "Guest_" +
                  System.currentTimeMillis()
        );

System.out.println(username);
```

Execution:

```text
Optional is empty
       ↓
orElseGet()
       ↓
Supplier.get()
       ↓
Generate fallback value
```

---

## 4. Production Context

A Supplier is useful when a centralized wrapper needs to execute some retrieval operation.

```java
public static <T> T redisGet(
        Supplier<T> action,
        String serviceName,
        String methodName) {

    try {
        return action.get();

    } catch (Exception e) {

        logger.warn(
            "Redis GET failed for method: {}",
            methodName
        );
    }

    return null;
}
```

Usage:

```java
CachedResponse cached =
    OperationExecutor.redisGet(
        () ->
            cacheService.getFromCache(
                cacheKey,
                CachedResponse.class
            ),
        "BillService",
        "getBillById"
    );
```

Here:

```text
Supplier
   ↓
get()
   ↓
Retrieve data
   ↓
Return value
```

---

# `Consumer<T>`

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Consumer<T> {

    void accept(T t);

    default Consumer<T> andThen(
            Consumer<? super T> after) {

        Objects.requireNonNull(after);

        return (T t) -> {
            accept(t);
            after.accept(t);
        };
    }
}
```

## Methods

- **Abstract Method:** `void accept(T t)`
- **Default Method:** `andThen()`
- **Static Methods:** None
- **Input Parameters:** **One**
- **Output / Return Type:** **`void`**

## Meaning

`Consumer<T>` represents an operation that takes one piece of data and performs some action with it without returning a result.

```text
Input
  ↓
Consumer
  ↓
Action
  ↓
void
```

## When to Use

Use `Consumer` when you need to process one input and perform a side effect where no return value is expected.

### Common Examples

- **Data Iteration (`forEach`):** Iterating through a list or stream of elements to process or display each item.
- **Audit Logging:** Passing an event object to a logger system to record user activities or transactional states.
- **Metric & Analytics Tracking:** Sending data payloads or system events to a monitoring system.
- **Data Notifications:** Sending an incoming notification payload to an external communication channel.
- **Post-Processing Pipelines:** Passing success or failure data into a tracking controller after a database operation completes.

## How to Use `Consumer`

### Method 1: `accept()`

`accept()` executes the Consumer using the supplied input.

```java
Consumer<String> printer =
    message ->
        System.out.println(message);

printer.accept("Hello Java");
```

Output:

```text
Hello Java
```

Execution:

```text
Input
  ↓
accept()
  ↓
Consumer logic
  ↓
No return value
```

---

### Method 2: `andThen()`

`andThen()` is used to chain two Consumers.

The first Consumer executes first, followed by the second Consumer.

```java
Consumer<String> first =
    message ->
        System.out.println(
            "First: " + message
        );

Consumer<String> second =
    message ->
        System.out.println(
            "Second: " + message
        );

Consumer<String> combined =
    first.andThen(second);

combined.accept("Java");
```

Output:

```text
First: Java
Second: Java
```

Execution order:

```text
combined.accept("Java")
        ↓
first.accept("Java")
        ↓
second.accept("Java")
```

Remember:

```text
A.andThen(B)

A → B
```

---

## 3. Inline Direct Execution

A Consumer can be passed directly to `forEach()`:

```java
import java.util.Arrays;
import java.util.List;

List<String> logs =
    Arrays.asList(
        "WARN: Low Memory",
        "ERROR: DB Timeout",
        "INFO: Job Completed"
    );

logs.forEach(
    logLine ->
        System.out.println(
            "Log Delivery -> " + logLine
        )
);
```

`forEach()` expects a Consumer.

Conceptually:

```java
Consumer<String> consumer =
    logLine ->
        System.out.println(logLine);
```

Then each element is passed to:

```java
consumer.accept(logLine);
```

---

## 4. Production Context

A Consumer can be used as a post-processing hook.

```java
public static <T> T dbGetWithMetrics(
        Supplier<T> action,
        Consumer<T> metricsHook,
        String serviceName) {

    try {

        T result = action.get();

        if (result != null) {
            metricsHook.accept(result);
        }

        return result;

    } catch (Exception e) {

        logger.error(
            "DB GET FAILED for service: {}",
            serviceName
        );

        throw new DatabaseException();
    }
}
```

Usage:

```java
BillDetailsResponse response =
    OperationExecutor.dbGetWithMetrics(

        () ->
            billRepository.findById(billId),

        billData ->
            metricsCollector.incrementCounter(
                "bill.fetch.success",
                "tier",
                billData.getTierType()
            ),

        "BillService"
    );
```

Execution:

```text
Supplier
   ↓
Fetch data
   ↓
Consumer
   ↓
Metrics / Logging
```

---

# `BiConsumer<T, U>`

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface BiConsumer<T, U> {

    void accept(T t, U u);

    default BiConsumer<T, U> andThen(
            BiConsumer<? super T, ? super U> after) {

        Objects.requireNonNull(after);

        return (l, r) -> {
            accept(l, r);
            after.accept(l, r);
        };
    }
}
```

## Methods

- **Abstract Method:** `void accept(T t, U u)`
- **Default Method:** `andThen()`
- **Static Methods:** None
- **Input Parameters:** **Two**
- **Output / Return Type:** **`void`**

## Meaning

`BiConsumer<T, U>` represents an operation that takes two inputs and performs an action without returning a result.

```text
Input 1 ─┐
         ├──→ BiConsumer → Action
Input 2 ─┘
```

## When to Use

Use `BiConsumer` when you need to process two related input values together.

### Common Examples

- **Map Iteration (`Map.forEach`):** Processing both keys and values of a Map.
- **Contextual Tracking:** Consuming data together with execution context or metadata.
- **Key-Value Configurations:** Processing a property name and its value together.
- **Error Reporting:** Handling a business object together with an exception.
- **Performance Tracking:** Processing a result together with execution time.

## How to Use `BiConsumer`

### Method 1: `accept()`

`accept()` receives two inputs and performs the defined action.

```java
BiConsumer<String, Integer> logger =
    (key, value) ->
        System.out.println(
            key + " = " + value
        );

logger.accept("Age", 25);
```

Output:

```text
Age = 25
```

---

### Method 2: `andThen()`

`andThen()` chains two BiConsumers.

```java
BiConsumer<String, Integer> first =
    (key, value) ->
        System.out.println(
            "First: " + key
        );

BiConsumer<String, Integer> second =
    (key, value) ->
        System.out.println(
            "Second: " + value
        );

BiConsumer<String, Integer> combined =
    first.andThen(second);

combined.accept("Age", 25);
```

Output:

```text
First: Age
Second: 25
```

Execution:

```text
combined.accept()
       ↓
first.accept()
       ↓
second.accept()
```

---

## 3. Inline Direct Execution

`Map.forEach()` provides both a key and a value, so it naturally works with `BiConsumer`.

```java
import java.util.HashMap;
import java.util.Map;

Map<String, String> statusMap =
    new HashMap<>();

statusMap.put("bill_01", "PAID");
statusMap.put("bill_02", "PENDING");

statusMap.forEach(
    (billId, status) ->
        System.out.println(
            "Alert: Bill " +
            billId +
            " is currently " +
            status
        )
);
```

Here:

```text
billId → First input
status → Second input
```

---

## 4. Production Context

A BiConsumer can receive a result together with execution metadata.

```java
public static <T> T dbGetWithTiming(
        Supplier<T> action,
        BiConsumer<T, Long> telemetryHook,
        String serviceName) {

    long startTime =
        System.currentTimeMillis();

    try {

        T result = action.get();

        long duration =
            System.currentTimeMillis()
            - startTime;

        telemetryHook.accept(
            result,
            duration
        );

        return result;

    } catch (Exception e) {

        logger.error(
            "DB GET FAILED for service: {}",
            serviceName
        );

        throw new DatabaseException();
    }
}
```

Usage:

```java
BillDetailsResponse response =
    OperationExecutor.dbGetWithTiming(

        () ->
            billRepository.findById(billId),

        (billData, timeTaken) ->
            logger.info(
                "Bill ID: {} processed in {} ms",
                billData.getId(),
                timeTaken
            ),

        "BillService"
    );
```

Here:

```text
Result + Execution Time
          ↓
      BiConsumer
          ↓
       Logging
```

---

# `Function<T, R>`

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Function<T, R> {

    R apply(T t);

    default <V> Function<V, R> compose(
            Function<? super V, ? extends T> before) {

        Objects.requireNonNull(before);

        return (V v) ->
            apply(before.apply(v));
    }

    default <V> Function<T, V> andThen(
            Function<? super R, ? extends V> after) {

        Objects.requireNonNull(after);

        return (T t) ->
            after.apply(apply(t));
    }

    static <T> Function<T, T> identity() {
        return t -> t;
    }
}
```

## Methods

- **Abstract Method:** `R apply(T t)`
- **Default Methods:** `compose()`, `andThen()`
- **Static Method:** `identity()`
- **Input Parameters:** **One**
- **Output / Return Type:** **`R`**

## Meaning

`Function<T, R>` represents a transformer that takes one input and converts it into an output.

```text
Input
  ↓
Function
  ↓
Output
```

## When to Use

Use `Function` when you need to transform, convert, map, or process one value into another value.

### Common Examples

- **Data Transformation (`Stream.map`):** Converting data models into another representation.
- **Data Parsing:** Converting a String into an Integer.
- **DTO Mapping:** Transforming database entities into API DTOs.
- **Encryption / Hashing:** Transforming plain text into a hashed value.
- **Centralized Processing Pipelines:** Passing data conversion logic into a centralized execution wrapper.

## How to Use `Function`

`Function` has four important methods:

```text
apply()
andThen()
compose()
identity()
```

---

## Method 1: `apply()`

`apply()` directly executes the Function.

```java
Function<String, Integer> stringLength =
    text -> text.length();

int result =
    stringLength.apply("Java");

System.out.println(result);
```

Output:

```text
4
```

Think:

```text
Input
  ↓
apply()
  ↓
Output
```

---

## Method 2: `andThen()`

`andThen()` executes the current Function first and then executes the next Function.

Example:

```java
Function<String, Integer> length =
    text -> text.length();

Function<Integer, Integer> doubleValue =
    number -> number * 2;

Function<String, Integer> pipeline =
    length.andThen(doubleValue);

System.out.println(
    pipeline.apply("Java")
);
```

Execution:

```text
"Java"
  ↓
length()
  ↓
4
  ↓
doubleValue()
  ↓
8
```

Rule:

```text
A.andThen(B)

A → B
```

---

## Method 3: `compose()`

`compose()` executes the `before` Function first and then executes the current Function.

Example:

```java
Function<String, Integer> length =
    text -> text.length();

Function<String, String> uppercase =
    text -> text.toUpperCase();

Function<String, Integer> pipeline =
    length.compose(uppercase);

System.out.println(
    pipeline.apply("java")
);
```

Execution:

```text
"java"
  ↓
uppercase()
  ↓
"JAVA"
  ↓
length()
  ↓
4
```

Rule:

```text
A.compose(B)

B → A
```

### Easy Rule to Remember

```text
andThen:
A.andThen(B)
A → B

compose:
A.compose(B)
B → A
```

---

## Method 4: `identity()`

`identity()` creates a Function that returns the input unchanged.

```java
Function<String, String> same =
    Function.identity();

System.out.println(
    same.apply("Java")
);
```

Output:

```text
Java
```

It is essentially:

```java
x -> x
```

### When to Use `identity()`

Use it when an API requires a Function but you want to keep the input unchanged.

Example:

```java
List<String> names =
    Arrays.asList(
        "Alex",
        "Brian",
        "Charles"
    );

List<String> result =
    names.stream()
         .map(Function.identity())
         .collect(Collectors.toList());
```

---

## 5. Inline Direct Execution

`Stream.map()` accepts a Function.

```java
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

List<String> names =
    Arrays.asList(
        "alex",
        "brian",
        "charles"
    );

List<String> uppercaseNames =
    names.stream()
         .map(name -> name.toUpperCase())
         .collect(Collectors.toList());
```

Execution:

```text
alex
 ↓
Function
 ↓
ALEX

brian
 ↓
Function
 ↓
BRIAN
```

---

## 6. Production Context

```java
public static <T, R> R map(
        Function<T, R> transformer,
        T sourceData,
        String serviceName,
        String methodName) {

    try {

        return transformer.apply(sourceData);

    } catch (Exception e) {

        logger.error(
            "DTO Mapping FAILED: {} | Service: {} | Method: {}",
            e.getMessage(),
            serviceName,
            methodName
        );

        throw new DataProcessingException();
    }
}
```

Usage:

```java
BillDetailsResponse response =
    OperationExecutor.map(

        entity ->
            new BillDetailsResponse(
                entity.getId(),
                entity.getAmount()
            ),

        billEntity,
        "BillService",
        "getBillById"
    );
```

Here:

```text
BillEntity
    ↓
Function.apply()
    ↓
BillDetailsResponse
```

---

# `BiFunction<T, U, R>`

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface BiFunction<T, U, R> {

    R apply(T t, U u);

    default <V> BiFunction<T, U, V> andThen(
            Function<? super R, ? extends V> after) {

        Objects.requireNonNull(after);

        return (T t, U u) ->
            after.apply(apply(t, u));
    }
}
```

## Methods

- **Abstract Method:** `R apply(T t, U u)`
- **Default Method:** `andThen()`
- **Static Methods:** None
- **Input Parameters:** **Two**
- **Output / Return Type:** **`R`**

## Meaning

`BiFunction<T, U, R>` takes two inputs, performs an operation using both, and produces one output.

```text
Input 1 ─┐
         ├──→ BiFunction → Output
Input 2 ─┘
```

## When to Use

Use `BiFunction` when you need to calculate, combine, or transform two input values into one result.

### Common Examples

- **Data Combination & Aggregation:** Combining two different values to calculate a final result.
- **Contextual Object Mapping:** Converting an entity into a DTO using additional context.
- **Mathematical Computations:** Performing calculations using two values.
- **Collection Merging:** Combining or replacing values in map operations such as `Map.merge()` or `Map.replaceAll()`.

## How to Use `BiFunction`

`BiFunction` has two important methods:

```text
apply()
andThen()
```

---

## Method 1: `apply()`

`apply()` directly executes the BiFunction using two inputs.

```java
BiFunction<Integer, Integer, Integer> add =
    (a, b) -> a + b;

int result =
    add.apply(10, 20);

System.out.println(result);
```

Output:

```text
30
```

Think:

```text
Input 1 ─┐
         ├──→ apply() → Output
Input 2 ─┘
```

---

## Method 2: `andThen()`

`andThen()` takes the result produced by the BiFunction and passes it to another regular Function.

```java
BiFunction<Integer, Integer, Integer> add =
    (a, b) -> a + b;

Function<Integer, Integer> doubleValue =
    value -> value * 2;

BiFunction<Integer, Integer, Integer> pipeline =
    add.andThen(doubleValue);

System.out.println(
    pipeline.apply(10, 20)
);
```

Execution:

```text
10 + 20
  ↓
30
  ↓
doubleValue()
  ↓
60
```

Rule:

```text
BiFunction.andThen(Function)

BiFunction
     ↓
Function
```

---

## 3. Inline Direct Execution

`Map.replaceAll()` accepts a BiFunction.

```java
Map<String, Integer> cart =
    new HashMap<>();

cart.put("Laptop", 1200);
cart.put("Mouse", 50);

cart.replaceAll(
    (item, price) ->
        price + 15
);
```

Here:

```text
item  → first input
price → second input
```

The returned value becomes the new map value.

---

## 4. Production Context

```java
public static <T, U, R> R mapWithContext(
        BiFunction<T, U, R> transformer,
        T sourceData,
        U context,
        String serviceName) {

    try {

        return transformer.apply(
            sourceData,
            context
        );

    } catch (Exception e) {

        logger.error(
            "Contextual DTO Mapping FAILED for service: {}",
            serviceName
        );

        throw new DataProcessingException();
    }
}
```

Usage:

```java
BillDetailsResponse response =
    OperationExecutor.mapWithContext(

        (billEntity, userRole) ->
            new BillDetailsResponse(
                billEntity.getId(),
                userRole.isAdmin()
                    ? billEntity.getSecretRoutingCode()
                    : "MASKED"
            ),

        billEntity,
        currentUserRole,
        "BillService"
    );
```

Here:

```text
BillEntity + UserRole
        ↓
    BiFunction
        ↓
BillDetailsResponse
```

---

# `Predicate<T>`

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);

    default Predicate<T> and(
            Predicate<? super T> other) {

        Objects.requireNonNull(other);

        return t ->
            test(t) && other.test(t);
    }

    default Predicate<T> negate() {

        return t -> !test(t);
    }

    default Predicate<T> or(
            Predicate<? super T> other) {

        Objects.requireNonNull(other);

        return t ->
            test(t) || other.test(t);
    }

    static <T> Predicate<T> isEqual(
            Object targetRef) {

        return (null == targetRef)
            ? Objects::isNull
            : object -> targetRef.equals(object);
    }
}
```

## Methods

- **Abstract Method:** `boolean test(T t)`
- **Default Methods:** `and()`, `or()`, `negate()`
- **Static Method:** `isEqual()`
- **Input Parameters:** **One**
- **Output / Return Type:** **`boolean`**

## Meaning

`Predicate<T>` represents a condition that takes one input and returns either `true` or `false`.

```text
Input
  ↓
Predicate
  ↓
true / false
```

## When to Use

Use `Predicate` when you need to **filter, validate, check, or test one input against a condition**.

### Common Examples

- **Data Filtering (`Stream.filter`):** Keeping only elements that satisfy a condition.
- **Data Validation:** Checking whether an object's state is valid.
- **Access & Authorization Checks:** Checking whether a user satisfies certain criteria.
- **Conditional Processing:** Checking a system state before executing an operation.

## How to Use `Predicate`

`Predicate` has five important methods:

```text
test()
and()
or()
negate()
isEqual()
```

---

## Method 1: `test()`

`test()` directly evaluates the Predicate.

```java
Predicate<Integer> isAdult =
    age -> age >= 18;

boolean result =
    isAdult.test(25);

System.out.println(result);
```

Output:

```text
true
```

---

## Method 2: `and()`

`and()` combines two Predicates.

Both conditions must be true.

```java
Predicate<Integer> isAdult =
    age -> age >= 18;

Predicate<Integer> isSenior =
    age -> age >= 60;

Predicate<Integer> isAdultAndSenior =
    isAdult.and(isSenior);

System.out.println(
    isAdultAndSenior.test(65)
);
```

Execution:

```text
65 >= 18 → true
65 >= 60 → true
             ↓
           true
```

Rule:

```text
A.and(B)

A AND B
```

---

## Method 3: `or()`

`or()` combines two Predicates where at least one condition must be true.

```java
Predicate<Integer> isAdult =
    age -> age >= 18;

Predicate<Integer> isChild =
    age -> age < 13;

Predicate<Integer> validGroup =
    isAdult.or(isChild);

System.out.println(
    validGroup.test(10)
);
```

Execution:

```text
10 >= 18 → false
10 < 13  → true
              ↓
            true
```

Rule:

```text
A.or(B)

A OR B
```

---

## Method 4: `negate()`

`negate()` reverses the result of the Predicate.

```java
Predicate<Integer> isAdult =
    age -> age >= 18;

Predicate<Integer> isNotAdult =
    isAdult.negate();

System.out.println(
    isNotAdult.test(15)
);
```

Original:

```text
15 >= 18
    ↓
false
```

After `negate()`:

```text
false
  ↓
true
```

Rule:

```text
A.negate()

NOT A
```

---

## Method 5: `isEqual()`

`isEqual()` is a static method.

It creates a Predicate that checks whether the input equals the specified target.

```java
Predicate<String> isJava =
    Predicate.isEqual("Java");

System.out.println(
    isJava.test("Java")
);
```

Output:

```text
true
```

Another example:

```java
Predicate<String> isPaid =
    Predicate.isEqual("PAID");

System.out.println(
    isPaid.test("PENDING")
);
```

Output:

```text
false
```

Remember:

```text
Predicate.isEqual(value)
        ↓
Creates a Predicate
        ↓
predicate.test(input)
        ↓
true / false
```

---

## 6. Inline Direct Execution

`Stream.filter()` accepts a Predicate.

```java
List<Integer> amounts =
    Arrays.asList(
        150,
        45,
        300,
        12,
        90
    );

List<Integer> highValues =
    amounts.stream()
           .filter(
               value -> value >= 100
           )
           .collect(Collectors.toList());
```

Execution:

```text
150 → true  → keep
45  → false → remove
300 → true  → keep
12  → false → remove
90  → false → remove
```

---

## 7. Combining Predicates

Predicates can be combined before passing them to `filter()`.

```java
Predicate<Integer> greaterThan100 =
    value -> value > 100;

Predicate<Integer> lessThan500 =
    value -> value < 500;

Predicate<Integer> validAmount =
    greaterThan100.and(lessThan500);

List<Integer> result =
    amounts.stream()
           .filter(validAmount)
           .collect(Collectors.toList());
```

The condition is:

```text
value > 100
    AND
value < 500
```

---

## 8. Production Context

```java
public static <T> void validate(
        Predicate<T> businessRule,
        T payload,
        String serviceName,
        String fieldName) {

    try {

        if (!businessRule.test(payload)) {

            logger.warn(
                "Validation failure on field: {} inside service: {}",
                fieldName,
                serviceName
            );

            throw new InvalidDataException();
        }

    } catch (InvalidDataException e) {

        throw e;

    } catch (Exception e) {

        logger.error(
            "System crash during validation evaluation: {}",
            e.getMessage()
        );

        throw new DataProcessingException();
    }
}
```

Usage:

```java
OperationExecutor.validate(

    bill ->
        bill.getAmount() > 0 &&
        bill.getCurrency() != null,

    incomingBillPayload,

    "BillService",
    "billingPricingDetails"
);
```

Here:

```text
Bill
 ↓
Predicate.test()
 ↓
true / false
 ↓
Validation result
```

---

# `BiPredicate<T, U>`

```java
package java.util.function;

import java.util.Objects;

@FunctionalInterface
public interface BiPredicate<T, U> {

    boolean test(T t, U u);

    default BiPredicate<T, U> and(
            BiPredicate<? super T, ? super U> other) {

        Objects.requireNonNull(other);

        return (t, u) ->
            test(t, u) &&
            other.test(t, u);
    }

    default BiPredicate<T, U> negate() {

        return (t, u) ->
            !test(t, u);
    }

    default BiPredicate<T, U> or(
            BiPredicate<? super T, ? super U> other) {

        Objects.requireNonNull(other);

        return (t, u) ->
            test(t, u) ||
            other.test(t, u);
    }
}
```

## Methods

- **Abstract Method:** `boolean test(T t, U u)`
- **Default Methods:** `and()`, `or()`, `negate()`
- **Static Methods:** None
- **Input Parameters:** **Two**
- **Output / Return Type:** **`boolean`**

## Meaning

`BiPredicate<T, U>` represents a condition that evaluates **two input values together** and returns `true` or `false`.

```text
Input 1 ─┐
         ├──→ BiPredicate → true / false
Input 2 ─┘
```

## When to Use

Use `BiPredicate` when the condition requires **two separate input values**.

### Common Examples

- **Credential Verification:** Comparing a provided username or value with another stored value.
- **Contextual Validation:** Validating a request against a user session or context.
- **Data Comparison:** Comparing a transaction value with an account limit.
- **Relationship Filtering:** Checking whether two pieces of data satisfy a relationship.

## How to Use `BiPredicate`

`BiPredicate` has four important methods:

```text
test()
and()
or()
negate()
```

---

## Method 1: `test()`

`test()` directly evaluates two input values.

```java
BiPredicate<String, Integer> checkLength =
    (text, length) ->
        text.length() == length;

boolean result =
    checkLength.test("Decode", 6);

System.out.println(result);
```

Output:

```text
true
```

---

## Method 2: `and()`

`and()` combines two BiPredicates.

Both conditions must be true.

```java
BiPredicate<Integer, Integer> greater =
    (a, b) -> a > b;

BiPredicate<Integer, Integer> difference =
    (a, b) -> a - b > 10;

BiPredicate<Integer, Integer> combined =
    greater.and(difference);

System.out.println(
    combined.test(30, 10)
);
```

Execution:

```text
30 > 10       → true
30 - 10 > 10  → true
                  ↓
                true
```

Rule:

```text
A.and(B)

A AND B
```

---

## Method 3: `or()`

`or()` combines two BiPredicates where at least one condition must be true.

```java
BiPredicate<Integer, Integer> greater =
    (a, b) -> a > b;

BiPredicate<Integer, Integer> equal =
    (a, b) -> a.equals(b);

BiPredicate<Integer, Integer> combined =
    greater.or(equal);

System.out.println(
    combined.test(10, 10)
);
```

Execution:

```text
10 > 10  → false
10 == 10 → true
             ↓
           true
```

Rule:

```text
A.or(B)

A OR B
```

---

## Method 4: `negate()`

`negate()` reverses the result.

```java
BiPredicate<Integer, Integer> greater =
    (a, b) -> a > b;

BiPredicate<Integer, Integer> notGreater =
    greater.negate();

System.out.println(
    notGreater.test(10, 20)
);
```

Original:

```text
10 > 20
  ↓
false
```

After `negate()`:

```text
false
  ↓
true
```

Rule:

```text
A.negate()

NOT A
```

---

## 5. Inline Direct Execution

```java
BiPredicate<Integer, Integer> isOverdraft =
    (balance, withdrawal) ->
        withdrawal > balance;

boolean alertUser =
    isOverdraft.test(500, 650);

System.out.println(alertUser);
```

Output:

```text
true
```

Execution:

```text
Balance + Withdrawal
        ↓
BiPredicate.test()
        ↓
true / false
```

---

## 6. Production Context

A BiPredicate is useful when a business rule requires two different objects.

```java
public static <T, U> void authorize(
        BiPredicate<T, U> safetyRule,
        T payload,
        U context,
        String serviceName) {

    try {

        if (!safetyRule.test(
                payload,
                context)) {

            logger.warn(
                "Unauthorized operation blocked in {}",
                serviceName
            );

            throw new UnauthorizedAccessException();
        }

    } catch (UnauthorizedAccessException e) {

        throw e;

    } catch (Exception e) {

        logger.error(
            "System crash during authorization check: {}",
            e.getMessage()
        );

        throw new DataProcessingException();
    }
}
```

Usage:

```java
OperationExecutor.authorize(

    (billPayload, accountProfile) ->
        accountProfile.getBalance()
            >= billPayload.getAmount(),

    incomingBillPayload,
    activeUserAccount,
    "BillService"
);
```

Here:

```text
Bill Payload ─────┐
                  ├──→ BiPredicate.test()
Account Profile ──┘
                  ↓
             true / false
```

---


# Final Mental Model

The simplest rule is:

```text
Runnable
→ Do something

Supplier
→ Give me something

Consumer
→ Do something with something

BiConsumer
→ Do something with two things

Function
→ Transform something

BiFunction
→ Transform two things

Predicate
→ Check something

BiPredicate
→ Check two things
```

And when you see multiple methods, remember:

```text
Function
→ apply()      = execute
→ andThen()    = current → next
→ compose()    = before → current
→ identity()   = same input

Predicate
→ test()       = check
→ and()        = both
→ or()         = either
→ negate()     = reverse
→ isEqual()    = equality check

Consumer
→ accept()     = execute action
→ andThen()    = first action → second action

BiConsumer
→ accept()     = execute action with two inputs
→ andThen()    = first action → second action

BiFunction
→ apply()      = execute with two inputs
→ andThen()    = BiFunction → Function

BiPredicate
→ test()       = check two inputs
→ and()        = both conditions
→ or()         = either condition
→ negate()     = reverse
```


















