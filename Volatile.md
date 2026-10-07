# The `volatile` Keyword

In Java, multi-threaded applications use CPU caches to speed up data access. However, this means different threads might hold separate, cached copies of the same variable, leading to visibility bugs.

The **`volatile`** keyword is a lightweight synchronization mechanism that instructs the JVM and CPU: *"Always read this variable directly from main memory, and always write updates straight back to main memory."*


## Core Features & Characteristics

* **Visibility**: Ensures that any change made to a `volatile` variable by one thread is **immediately visible** to all other reading threads, bypassing local CPU cache lines.
* **Prevents Instruction Reordering**: Acts as a memory barrier (fence). It prevents the JVM and CPU from reordering instructions around reads and writes of the `volatile` variable, preserving program execution order.
* **Lightweight Synchronization**: Unlike `synchronized` or explicit locks (`Lock`), `volatile` **does not cause thread blocking or context switching**, making it much faster and cheaper to use.


## The Problem Without `volatile` (Visibility Issue)

When a thread modifies a regular variable, the update might sit in that specific thread's CPU cache and not propagate to main memory. Other threads checking that variable will read their stale, cached copy, causing infinite loops or missed state changes.

### Code Example: Thread Hangs Without `volatile`

```java
public class WithoutVolatileExample {
    // Regular variable (not volatile)
    private static boolean running = true;

    public static void main(String[] args) throws InterruptedException {
        Thread workerThread = new Thread(() -> {
            System.out.println("Worker thread started...");
            while (running) {
                // Without volatile, the worker thread might never see running = false
                // because it caches 'running' in its CPU register/cache.
            }
            System.out.println("Worker thread stopped.");
        });

        workerThread.start();

        Thread.sleep(1000);
        running = false; // Main thread tries to stop the worker
        System.out.println("Main thread set running to false.");

        workerThread.join();
    }
}
```


## The Solution With `volatile`

By marking the flag as `volatile`, we force all reads and writes to go directly to main memory, ensuring the worker thread instantly detects the change.

### Code Example: Clean Shutdown With `volatile`

```java
public class WithVolatileExample {
    // Volatile flag guarantees immediate cross-thread visibility
    private static volatile boolean running = true;

    public static void main(String[] args) throws InterruptedException {
        Thread workerThread = new Thread(() -> {
            System.out.println("Worker thread started...");
            while (running) {
                // Instantly sees when 'running' becomes false
            }
            System.out.println("Worker thread stopped successfully.");
        });

        workerThread.start();

        Thread.sleep(1000);
        running = false; // Immediately visible to the worker thread
        System.out.println("Main thread set running to false.");

        workerThread.join();
    }
}
```


## Limitation: Why `volatile` Fails for Variable Increments (`count++`)

While `volatile` guarantees visibility, it does **not** guarantee atomicity.

An operation like `count++` is actually three distinct steps:

1. Read the current value of `count` from memory.
2. Increment the value locally (`count + 1`).
3. Write the new value back to memory.

If multiple threads perform `count++` simultaneously on a `volatile` variable, their execution steps will interleave. Threads will overwrite each other's updates, causing lost increments and an incorrect final count.

### Code Example: `volatile` Fails During Concurrent Increment

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

public class VolatileIncrementFailureExample {
    // Volatile does NOT make compound operations (like count++) atomic!
    private static volatile int count = 0;

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(10);

        // Submit 1000 tasks, each incrementing count
        for (int i = 0; i < 1000; i++) {
            executor.submit(() -> {
                count++; // NOT thread-safe! Read -> Increment -> Write race condition
            });
        }

        executor.shutdown();
        executor.awaitTermination(2, TimeUnit.SECONDS);

        // EXPECTED: 1000
        // ACTUAL: Usually less than 1000 due to race conditions (e.g., 982, 965)
        System.out.println("Final Counter Value (Incorrect): " + count);
    }
}
```

| Time / Step | Thread A Action | Thread B Action | Shared Memory (`count`) |
| :--- | :--- | :--- | :--- |
| **Initial State** | — | — | `count = 0` |
| **Step 1** | Reads `count` (gets `0`) | — | `count = 0` |
| **Step 2** | — | Reads `count` (gets `0` because Thread A hasn't written back yet) | `count = 0` |
| **Step 3** | Increments local value: `0 + 1 = 1` | — | `count = 0` |
| **Step 4** | — | Increments local value: `0 + 1 = 1` | `count = 0` |
| **Step 5** | Writes its local value (`1`) back to `count` | — | `count = 1` |
| **Step 6** | — | Writes its local value (`1`) back to `count`, completely overwriting Thread A's update | `count = 1` (Should have been `2`, but became `1`) |

### Fix for Increments

For atomic counter updates, you must use:

* `AtomicInteger`

  ```java
  AtomicInteger count = new AtomicInteger();
  count.incrementAndGet();
  ```

* or proper locking mechanisms (`synchronized`).


#### Code: Atomic Counter Using `AtomicInteger`

```java
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

public class AtomicCounterExample {
    // AtomicInteger makes compound operations (like increment) atomic!
    private static AtomicInteger count = new AtomicInteger(0);

    public static void main(String[] args) throws InterruptedException {
        ExecutorService executor = Executors.newFixedThreadPool(10);

        // Submit 1000 tasks, each incrementing count
        for (int i = 0; i < 1000; i++) {
            executor.submit(() -> {
                count.incrementAndGet(); // Thread-safe! Atomic read-modify-write
            });
        }

        executor.shutdown();
        executor.awaitTermination(2, TimeUnit.SECONDS);

        // EXPECTED: 1000
        // ACTUAL: 1000 (always correct, every run)
        System.out.println("Final Counter Value (Correct): " + count.get());
    }
}
```

The value is **always 1000** on every run because `incrementAndGet()` performs the read-modify-write as a single atomic operation.
