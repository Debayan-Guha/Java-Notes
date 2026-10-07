# `CompletableFuture`

A **`CompletableFuture`** (introduced in Java 8) is an evolution of the traditional `Future`. While standard `Future` forces you to **block** using `get()` to retrieve results, `CompletableFuture` enables **non-blocking, reactive, and declarative pipeline programming**.

It allows you to chain multiple asynchronous tasks together (e.g., *Task A -> then transform -> then combine with Task B -> handle error*), running them asynchronously without blocking precious threads.


## Key Methods

| Method Signature | Description | Behavior |
| :--- | :--- | :--- |
| `CompletableFuture.runAsync(Runnable)` | Runs a `Runnable` task asynchronously in the background. | Returns `CompletableFuture<Void>` (no result). |
| `CompletableFuture.supplyAsync(Supplier)` | Runs a `Supplier` task asynchronously and returns its result. | Returns `CompletableFuture<T>` with a computed value. |
| `thenApply(Function)` | Transforms the result of the previous stage when it completes. | Non-blocking callback; returns a new `CompletableFuture`. |
| `thenAccept(Consumer)` | Consumes the final result without returning anything new. | Non-blocking terminal step. |
| `thenCombine(CompletableFuture, BiFunction)` | Combines the results of two independent futures once both complete. | Merges results into a single output. |
| `allOf(CompletableFuture<?>...)` | Waits for **all** given futures to complete. | Returns a `CompletableFuture<Void>` when the batch finishes. |
| `anyOf(CompletableFuture<?>...)` | Completes as soon as **any** of the given futures completes. | Returns a `CompletableFuture<Object>` with the fastest result. |


## Characteristics & Comparison

* **Non-Blocking Chaining**: Instead of calling `future.get()` and blocking your thread, you attach callbacks like `thenApply()` that automatically execute as soon as the data becomes available.
* **Declarative Pipeline**: You can write fluent, functional data pipelines (similar to Java Streams) across asynchronous boundaries.
* **Custom Executors**: By default, async operations run on the common `ForkJoinPool.commonPool()`, but you can easily pass your own custom `ExecutorService` as a second argument to `supplyAsync(task, executor)`.


## Key Differences: `Future` vs `CompletableFuture`

| Feature | Standard `Future` | `CompletableFuture` |
| :--- | :--- | :--- |
| **Retrieval** | Must block using `get()` to get results. | Non-blocking callbacks (`thenApply`, `thenAccept`). |
| **Pipeline Chaining** | Impossible (cannot chain subsequent tasks easily). | Native support for fluent chaining (`.thenApply().thenCompose()`). |
| **Exception Handling** | Manual try-catch around `future.get()`. | Built-in reactive operators (`exceptionally()`, `handle()`). |
| **Manual Completion** | Cannot manually trigger completion from outside. | Can be manually completed using `complete(value)`. |


## Exception Handling in `CompletableFuture`

Instead of wrapping errors in an `ExecutionException` upon retrieval, `CompletableFuture` provides fluent error-handling operators directly inside the pipeline:

* **`exceptionally(Function)`**: Catches any exception thrown earlier in the pipeline and returns a fallback/default value.
* **`handle(BiFunction)`**: Receives both the result (if successful) and the exception (if failed), allowing you to clean up or transform the outcome regardless of success or failure.

```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public class CompletableFutureExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        
        System.out.println("Main thread starts processing...");

        // 1. Asynchronous supply: Fetching user ID asynchronously
        CompletableFuture<String> futurePipeline = CompletableFuture.supplyAsync(() -> {
            System.out.println("Fetching user ID on thread: " + Thread.currentThread().getName());
            // Simulate delay
            try { Thread.sleep(500); } catch (InterruptedException e) { Thread.currentThread().interrupt(); }
            
            // Simulate error condition
            if (false) { throw new RuntimeException("Database down!"); }
            
            return "User_1049";
        })
        // 2. Chaining: Transform user ID to user profile data
        .thenApply(userId -> {
            System.out.println("Fetching profile for " + userId + " on thread: " + Thread.currentThread().getName());
            return "Profile Data for [" + userId + "]";
        })
        // 3. Error Handling fallback if anything above fails
        .exceptionally(ex -> {
            System.err.println("Error occurred: " + ex.getMessage());
            return "Default Guest Profile";
        });

        // Main thread can continue doing other work here...
        System.out.println("Main thread doing other work while async pipeline runs...");

        // 4. Blocking only at the very end to get final result (or use thenAccept for non-blocking end)
        String finalResult = futurePipeline.get();
        System.out.println("Final Result Received: " + finalResult);
    }
}
```


## Individual Methods Guide

### 1 `supplyAsync()`

**Characteristics & Behavior**
* **Asynchronous Execution with Result**: Runs a task asynchronously in the background using a `Supplier` (which returns a value) and returns a `CompletableFuture<T>`.
* **Default Pool**: By default, it runs on the common `ForkJoinPool.commonPool()`, but you can optionally pass a custom `ExecutorService` as a second argument.

**Exception Handling**
* Any runtime exception thrown inside the supplier is captured inside the returned future rather than crashing the calling thread. It can be caught later via `exceptionally()` or when calling `get()`.

**Code Example**
```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public class SupplyAsyncExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
            return "Result from supplyAsync";
        });

        // WITHOUT future.get():
        // 1. The main thread does not wait. It immediately finishes executing the main method.
        // 2. Since the main thread ends, the JVM shuts down instantly.
        // 3. The background thread is forcefully killed before it can finish, so nothing prints.

        System.out.println(future.get());
    }
}
```

### 2 `runAsync()`

**Characteristics & Behavior**
* **Fire-and-Forget / No Result**: Runs a task asynchronously using a `Runnable` (which performs an action but returns no result).
* **Return Type**: Returns a `CompletableFuture<Void>`.

**Exception Handling**
* Captures any exceptions thrown during execution. Calling `future.get()` will throw an `ExecutionException` wrapping the original error.

**Code Example**
```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public class RunAsyncExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Void> future = CompletableFuture.runAsync(() -> {
            System.out.println("Running background task without return value...");
        });

        future.get();
    }
}
```

### 3 `thenApply()`

**Characteristics & Behavior**
* **Result Transformation**: Transforms the result of the previous stage when it completes by applying a `Function`.
* **Pipeline Flow**: Takes the output of stage A, maps it to a new type or value, and returns a new `CompletableFuture`.

**Exception Handling**
* If the previous stage fails with an exception, `thenApply()` is bypassed, and the exception propagates down the chain.

**Code Example**
```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public class ThenApplyExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Integer> future = CompletableFuture.supplyAsync(() -> 5)
            .thenApply(val -> val * 10);

        System.out.println(future.get()); // 50
    }
}
```

### 4 `thenAccept()`

**Characteristics & Behavior**
* **Terminal Consumer**: Consumes the final result of the previous stage without returning anything new (`void`).
* **Usage**: Typically used as the final step in a pipeline to process, save, or log data.

**Exception Handling**
* Skipped if upstream tasks throw an unhandled exception.

**Code Example**
```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public class ThenAcceptExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Void> future = CompletableFuture.supplyAsync(() -> "User Data")
            .thenAccept(data -> {
                System.out.println("Consuming: " + data);
            });

        future.get();
    }
}
```

### 5 `thenCombine()`

**Characteristics & Behavior**
* **Merging Two Futures**: Waits for two independent futures to complete, then combines their results using a `BiFunction` to produce a merged output.
* **Concurrency**: Both source futures execute independently and concurrently before combining.

**Exception Handling**
* If either of the two input futures fails, the combined future completes exceptionally.

**Code Example**
```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public class ThenCombineExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<String> task1 = CompletableFuture.supplyAsync(() -> "Hello");
        CompletableFuture<String> task2 = CompletableFuture.supplyAsync(() -> " World");

        CompletableFuture<String> combined = task1.thenCombine(task2, (s1, s2) -> s1 + s2);

        System.out.println(combined.get()); // Hello World
    }
}
```

### 6 `allOf()`

**Characteristics & Behavior**
* **Batch Completion Check**: Accepts a variable number of futures and returns a new `CompletableFuture<Void>` that completes only when all of the given futures have finished.
* **Return Value**: Does not combine results automatically; it simply signals completion of the entire batch.

**Exception Handling**
* If any single future in the collection fails, calling `.get()` on the `allOf` future will throw an `ExecutionException`.

**Code Example**
```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public class AllOfExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<Void> f1 = CompletableFuture.runAsync(() -> {});
        CompletableFuture<Void> f2 = CompletableFuture.runAsync(() -> {});

        CompletableFuture<Void> all = CompletableFuture.allOf(f1, f2);

        all.get();
        System.out.println("All tasks finished!");
    }
}
```

### 7 `anyOf()`

**Characteristics & Behavior**
* **Race Condition / Fastest Wins**: Accepts a variable number of futures and completes as soon as any one of them finishes.
* **Return Type**: Returns a `CompletableFuture<Object>` containing the result of whichever task won the race.

**Exception Handling**
* Completes exceptionally if the fastest task throws an exception.

**Code Example**
```java
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.ExecutionException;

public class AnyOfExample {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        CompletableFuture<String> slow = CompletableFuture.supplyAsync(() -> "Slow");
        CompletableFuture<String> fast = CompletableFuture.supplyAsync(() -> "Fast");

        CompletableFuture<Object> winner = CompletableFuture.anyOf(slow, fast);

        System.out.println(winner.get()); // Fast
    }
}
```



---




# Production Warning: The Danger of Unmanaged Asynchronous Tasks (ForkJoinPool Starvation, Thread Exhaustion & OOM)

In our toy command-line examples, we call `future.get()` to block the `main` thread so the JVM doesn't shut down before background tasks finish.

However, in a real-world **production application** (like a Spring Boot web
server or enterprise service), the main thread never exits. This introduces a
critical architectural danger if you fire-and-forget asynchronous tasks
without proper lifecycle management.

### 1. The Threat to the Common Pool (`ForkJoinPool.commonPool()`)

By default, all asynchronous methods like `supplyAsync()` and `runAsync()` execute tasks on the shared **`ForkJoinPool.commonPool()`**.


#### What is a pool?
A set of reusable worker threads. Instead of creating a new thread per task, Java reuses a few threads.

#### What is "shared"?
Java creates one pool automatically at startup, and the **whole JVM** uses it — your code, libraries, and the JDK. That pool is `ForkJoinPool.commonPool()`.

#### Why do async methods use it?
When you call `supplyAsync()` without giving your own executor, Java still needs *some* thread to run the task. The common pool already exists, so Java uses it by default.

```text
No custom executor given  ->  common pool is used
Custom executor given     ->  your pool is used
```

#### Why it's a problem:

* The common pool has only `CPU cores - 1` threads.
* If your application fires off long-running tasks or blocking I/O calls (e.g., waiting on a slow database or external API) without a custom executor, **all threads in the common pool can become blocked**.
* This starves the entire JVM, causing unrelated features that rely on the common pool to freeze or fail.

**Fix:** Pass your own bounded `ExecutorService` for blocking or long-running work.


#### No Custom Executor vs Custom Executor

- **No custom executor given → common pool is used**

```java
CompletableFuture.supplyAsync(() -> {
    return fetchData();          // runs on ForkJoinPool.commonPool()
});
```

- **Custom executor given → your pool is used**

```java
ExecutorService myPool = Executors.newFixedThreadPool(10);

CompletableFuture.supplyAsync(() -> {
    return fetchData();          // runs on myPool, NOT the common pool
}, myPool);
```

**The only difference:** the second argument — your own `ExecutorService`.

### 2. Infinite Loops & Resource Exhaustion

If a background asynchronous task gets stuck in an infinite loop, throws unhandled errors repeatedly, or waits indefinitely for a dead resource:

* The thread remains occupied forever.
* If new requests continuously trigger this asynchronous operation, you will exhaust the thread pool.
* Completed futures or their results that are retained by long-lived references (static lists, caches, listeners) will consume heap memory over time, eventually resulting in an **`OutOfMemoryError` (OOM)** or complete
  server crash.

### 3. Best Practices for Production

* **Never use the default pool for blocking I/O**: Always pass a dedicated, bounded custom `ExecutorService` to your async tasks so you isolate resource limits:

  ```java
  ExecutorService customExecutor = new ThreadPoolExecutor(
      10, 10,                       // core = max = 10
      0L, TimeUnit.MILLISECONDS,
      new ArrayBlockingQueue<>(100) // bounded queue
  );
  CompletableFuture.supplyAsync(() -> fetchExternalData(), customExecutor);
  ```

* **Always set timeouts**: Never let an async operation wait forever.

  ```java
  // Fails with TimeoutException
  CompletableFuture.supplyAsync(() -> slowOperation())
      .orTimeout(3, TimeUnit.SECONDS)
      .exceptionally(ex -> fallbackResponse());

  // Completes normally with a fallback value
  CompletableFuture.supplyAsync(() -> slowOperation())
      .completeOnTimeout("default", 3, TimeUnit.SECONDS);
  ```

* **Proper Shutdown**: Ensure your custom thread pools are gracefully shut down using `shutdown()` and `awaitTermination()` when your application context closes to prevent dangling background threads.

