# Locks

Locks in Java are synchronization mechanisms used to control concurrent access to shared resources by multiple threads, preventing race conditions, data corruption, and ensuring thread safety.


## Types of Locks in Java

Locks in Java are broadly classified into two high-level categories based on how they are implemented and managed: **Intrinsic Locks** (implicit/built-in) and **Explicit Locks**.

### 1. Intrinsic Locks (Implicit / Monitor Locks)
- **What they are:** Built directly into every Java object via a structural component called a monitor. They are used via the **`synchronized`** keyword.
- **How they work:** The JVM handles everything automatically. The lock is acquired when entering a `synchronized` block or method and is automatically released when exiting the block—even if an exception is thrown.
- **Characteristics:** Simple to use, but rigid (blocking, no timeouts, no fairness control, and no way to interrupt a thread waiting for the lock).

### 2. Explicit Locks
- **What they are:** Advanced locking mechanisms introduced in the `java.util.concurrent.locks` package (implementing the `Lock` interface). They give developers explicit, programmatic control over lock acquisition and release.
- **How they work:** You must manually call `.lock()` to acquire the lock and `.unlock()` (typically inside a `try-finally` block) to release it.

Explicit locks branch out into specialized implementation types:

* **`ReentrantLock`:** 
  - A standard mutual exclusion lock that allows the same thread to re-acquire the lock multiple times without deadlocking itself. 
  - Supports advanced features like fairness policies (queuing threads in order), timed lock attempts (`tryLock`), and interruptible waiting.
* **`ReadWriteLock` (`ReentrantReadWriteLock`):** 
  - Splits access into two separate locks: a **Read Lock** (shared by multiple threads concurrently) and a **Write Lock** (exclusive). 
  - Greatly improves performance in read-heavy applications.
* **`StampedLock`:** 
  - Introduced in Java 8, it provides read and write locks along with an **optimistic reading** mode that doesn't block writers, making it ultra-fast for specific high-concurrency scenarios.

## Comparison: Intrinsic Locks vs. Explicit Locks

| Feature / Characteristic | Intrinsic Locks (`synchronized`) | Explicit Locks (`ReentrantLock`, etc.) |
| :--- | :--- | :--- |
| **Implementation** | Built directly into every Java object via monitor headers. | Implemented as classes in the `java.util.concurrent.locks` package. |
| **Acquisition & Release** | **Implicit:** Automatically acquired upon entering a block/method and released upon exit. | **Explicit:** Must be manually invoked via `.lock()` and released via `.unlock()` (ideally in a `try-finally` block). |
| **Reentrancy** | **Yes:** A thread can re-enter a synchronized block/method it already holds without deadlocking. | **Yes:** Tracks hold counts; requires matching `.unlock()` calls for every `.lock()`. |
| **Fairness Policy** | **Unfair only:** No control over thread ordering; thread starvation is possible. | **Configurable:** Can be initialized as fair (`new ReentrantLock(true)`) or unfair. |
| **Interruptibility** | **No:** Threads blocked waiting for a lock cannot be interrupted. | **Yes:** Supports interruptible lock acquisition via `lockInterruptibly()`. |
| **Timed Locking** | **No:** Cannot specify a timeout; blocks indefinitely until the lock is available. | **Yes:** Supports timed attempts via `tryLock(long timeout, TimeUnit unit)`. |
| **Condition Queues** | **Basic:** Uses a single built-in wait set per object (`wait()`, `notify()`, `notifyAll()`). | **Advanced:** Supports multiple independent wait queues per lock using `Condition` objects (`lock.newCondition()`). |

---

# Part 1 — The Lock Interface

## The `Lock` Interface (`java.util.concurrent.locks.Lock`)

```java
package java.util.concurrent.locks;

import java.util.concurrent.TimeUnit;

public interface Lock {

    /**
     * Acquires the lock.
     *
     * If the lock is free:
     *     → The current thread gets the lock immediately.
     *
     * If another thread already has the lock:
     *     → The current thread waits until the lock is released.
     *
     * Example:
     *     Thread-1 has the lock.
     *     Thread-2 calls lock().
     *     Thread-2 waits.
     *     Thread-1 calls unlock().
     *     Thread-2 can then get the lock.
     *
     * Important:
     *     → This method does not stop waiting when the thread is interrupted.
     */
    void lock();


    /**
     * Acquires the lock, but allows the waiting thread to be interrupted.
     *
     * If the lock is free:
     *     → The current thread gets the lock immediately.
     *
     * If another thread already has the lock:
     *     → The current thread waits.
     *
     * If the waiting thread is interrupted:
     *     → It stops waiting.
     *     → InterruptedException is thrown.
     *
     * Difference from lock():
     *     → lock() keeps waiting even if interrupted.
     *     → lockInterruptibly() can stop waiting when interrupted.
     */
    void lockInterruptibly() throws InterruptedException;


    /**
     * Tries to acquire the lock without waiting.
     *
     * If the lock is free:
     *     → The current thread gets the lock.
     *     → Returns true.
     *
     * If another thread already has the lock:
     *     → The current thread does NOT wait.
     *     → Returns false immediately.
     *
     * Example:
     *     if (lock.tryLock()) {
     *         // Lock was successfully acquired
     *     } else {
     *         // Lock is busy, so do something else
     *     }
     */
    boolean tryLock();


    /**
     * Tries to acquire the lock and waits for a limited amount of time.
     *
     * If the lock is free:
     *     → The current thread gets the lock immediately.
     *     → Returns true.
     *
     * If another thread already has the lock:
     *     → The current thread waits.
     *
     * If the lock becomes free before the timeout:
     *     → The current thread gets the lock.
     *     → Returns true.
     *
     * If the timeout expires before getting the lock:
     *     → The thread stops waiting.
     *     → Returns false.
     *
     * If the thread is interrupted while waiting:
     *     → It stops waiting.
     *     → InterruptedException is thrown.
     *
     * Example:
     *     tryLock(10, TimeUnit.SECONDS)
     *
     *     → Try to get the lock.
     *     → If busy, wait up to 10 seconds.
     *     → Got the lock within 10 seconds → true.
     *     → Still busy after 10 seconds → false.
     */
    boolean tryLock(long time, TimeUnit unit) throws InterruptedException;


    /**
     * Releases the lock.
     *
     * Usually called after the thread finishes using the shared resource.
     *
     * Example:
     *     Thread-1 has the lock.
     *     Thread-1 finishes its work.
     *     Thread-1 calls unlock().
     *     Another waiting thread can now get the lock.
     *
     * Best practice:
     *     → Put unlock() inside a finally block.
     *     → This makes sure the lock is released even if an exception occurs.
     *
     * Example:
     *
     *     lock.lock();
     *
     *     try {
     *         // Work that requires the lock
     *     } finally {
     *         lock.unlock();
     *     }
     */
    void unlock();


    /**
     * Creates a Condition associated with this Lock.
     *
     * A Condition is used when a thread needs to WAIT for something
     * to happen before continuing.
     *
     * Example:
     *     Imagine a queue:
     *
     *     notEmpty → used when a consumer waits for data.
     *     notFull  → used when a producer waits for free space.
     *
     * One Lock can have multiple Conditions.
     *
     * Example:
     *
     *     Lock lock = new ReentrantLock();
     *
     *     Condition notEmpty = lock.newCondition();
     *     Condition notFull = lock.newCondition();
     *
     *     Here, the same lock has two separate waiting conditions.
     */
    Condition newCondition();
}
```


## Understanding Each `Lock` Method with Code Examples

The examples below use `ReentrantLock`, which is a common implementation of the `Lock` interface.

```java
import java.util.concurrent.TimeUnit;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class LockExamples {

    private static final Lock lock = new ReentrantLock();

    public static void main(String[] args) {

        // Examples will be shown below.
    }
}
```

---

### 1. `lock()`

Use `lock()` when the thread **must wait until it gets the lock**.

```java
Lock lock = new ReentrantLock();

lock.lock();

try {
    // Critical section
    System.out.println("Thread has the lock.");
    System.out.println("Doing some work...");
} finally {
    lock.unlock();
}
```

#### What happens?

```text
lock.lock()
     ↓
Is the lock free?
     ↓
   YES → Get the lock
     ↓
Do the work
     ↓
unlock()
```

If another thread already has the lock:

```text
Thread-1 → has the lock

Thread-2 → lock()
              ↓
           waits
              ↓
Thread-1 → unlock()
              ↓
Thread-2 → gets the lock
```

So:

```java
lock.lock();
```

means:

> **"Give me the lock. If it is busy, I will wait until I get it."**

---

### 2. `lockInterruptibly()`

Use `lockInterruptibly()` when the thread can wait for the lock, but you also want the ability to **cancel that waiting by interrupting the thread**.

```java
Lock lock = new ReentrantLock();

try {
    lock.lockInterruptibly();

    try {
        // Critical section
        System.out.println("Thread has the lock.");
    } finally {
        lock.unlock();
    }

} catch (InterruptedException e) {
    System.out.println("Thread was interrupted while waiting for the lock.");
}
```

#### Example situation

Suppose Thread-1 already has the lock.

```text
Thread-1 → has the lock

Thread-2 → lockInterruptibly()
                ↓
             waiting
                ↓
          Thread-2 is interrupted
                ↓
       InterruptedException
```

The important difference is:

```text
lock()
    → Wait until lock is available.
    → Interruption does not stop the waiting.

lockInterruptibly()
    → Wait until lock is available.
    → Interruption can stop the waiting.
```

---

### 3. `tryLock()`

Use `tryLock()` when you **do not want to wait**.

It simply checks whether the lock is available.

```java
Lock lock = new ReentrantLock();

if (lock.tryLock()) {

    try {
        System.out.println("Lock acquired.");
        System.out.println("Doing some work...");

    } finally {
        lock.unlock();
    }

} else {

    System.out.println("Lock is busy.");
    System.out.println("I will not wait.");
}
```

#### What happens?

If the lock is free:

```text
tryLock()
    ↓
Lock is free
    ↓
Get lock
    ↓
return true
```

If another thread has the lock:

```text
tryLock()
    ↓
Lock is busy
    ↓
Do NOT wait
    ↓
return false
```

So:

```java
if (lock.tryLock()) {
    // Got the lock
} else {
    // Could not get the lock
}
```

means:

> **"Try to get the lock right now. If I cannot get it, I will do something else."**

---

### 4. `tryLock(long time, TimeUnit unit)`

Use this when you are willing to **wait for a limited amount of time**.

For example:

```java
Lock lock = new ReentrantLock();

try {

    if (lock.tryLock(10, TimeUnit.SECONDS)) {

        try {
            System.out.println("Lock acquired.");
            System.out.println("Doing some work...");

        } finally {
            lock.unlock();
        }

    } else {

        System.out.println("Could not get the lock within 10 seconds.");

    }

} catch (InterruptedException e) {

    System.out.println("Thread was interrupted while waiting.");

}
```

Here:

```java
lock.tryLock(10, TimeUnit.SECONDS)
```

means:

> **"Try to get the lock. If it is busy, wait for up to 10 seconds."**

#### Situation 1 — Lock is immediately available

```text
tryLock(10 seconds)
        ↓
Lock is free
        ↓
Get lock immediately
        ↓
return true
```

#### Situation 2 — Lock is busy but becomes free after 3 seconds

```text
tryLock(10 seconds)
        ↓
Lock is busy
        ↓
Wait
        ↓
3 seconds later
        ↓
Lock becomes free
        ↓
Get lock
        ↓
return true
```

#### Situation 3 — Lock stays busy for more than 10 seconds

```text
tryLock(10 seconds)
        ↓
Lock is busy
        ↓
Wait
        ↓
10 seconds pass
        ↓
Still busy
        ↓
return false
```

#### Compare the two `tryLock()` methods

```text
tryLock()
    → Do not wait.
    → Try once.
    → true / false

tryLock(10, TimeUnit.SECONDS)
    → Wait if necessary.
    → Wait maximum 10 seconds.
    → true / false
```

---

### 5. `unlock()`

`unlock()` releases the lock so that another waiting thread can use it.

Usually, it is used together with `lock()`:

```java
Lock lock = new ReentrantLock();

lock.lock();

try {

    System.out.println("Working while holding the lock...");

} finally {

    lock.unlock();
}
```

Think of it like:

```text
lock()
  ↓
"I am taking control of the shared resource."
  ↓
Do work
  ↓
unlock()
  ↓
"I am finished. Another thread can use it."
```

#### Why use `finally`?

Consider this:

```java
lock.lock();

try {

    // Some work
    // An exception might occur here

} finally {

    lock.unlock();
}
```

Even if an exception occurs during the work, `finally` still executes.

Therefore:

```java
lock.lock();

try {
    // Critical section
} finally {
    lock.unlock();
}
```

is the standard pattern.

---

### 6. `newCondition()`

`newCondition()` is used when threads need to **wait for a particular condition**.

For example, imagine a queue.

A consumer cannot remove an item if the queue is empty.

A producer cannot add an item if the queue is full.

We can create two conditions:

```java
Lock lock = new ReentrantLock();

Condition notEmpty = lock.newCondition();
Condition notFull = lock.newCondition();
```

Now we have:

```text
                Lock
                 |
        -------------------
        |                 |
     notEmpty           notFull
        |                 |
   Queue has data     Queue has space
```

A consumer can wait for the queue to become non-empty:

```java
lock.lock();

try {

    while (queue.isEmpty()) {
        notEmpty.await();
    }

    // Remove item from queue

} finally {

    lock.unlock();
}
```

A producer can signal that data is now available:

```java
lock.lock();

try {

    // Add item to queue

    notEmpty.signal();

} finally {

    lock.unlock();
}
```

The important idea is:

```text
newCondition()
     ↓
Creates a Condition
     ↓
Condition allows threads to wait
     ↓
Another thread can signal them
```

One lock can have multiple conditions:

```java
Lock lock = new ReentrantLock();

Condition notEmpty = lock.newCondition();
Condition notFull = lock.newCondition();
```

This allows different groups of threads to wait for different situations.

---

# Part 2 — Lock Implementations

## ReentrantLock

`ReentrantLock` is the standard implementation of the `Lock` interface.

- **Reentrant:** A thread that already holds the lock can acquire it again without deadlocking itself.
- **Fairness:** Can be constructed as fair (`new ReentrantLock(true)`) to enforce FIFO ordering, or unfair (default) for higher throughput.
- **Advanced features:** Supports `tryLock`, `lockInterruptibly`, timed waits, and multiple `Condition` queues.

```java
import java.util.concurrent.locks.ReentrantLock;

ReentrantLock lock = new ReentrantLock();       // unfair (default)
ReentrantLock fair = new ReentrantLock(true);   // fair
```

**Reentrancy example:**

```java
ReentrantLock lock = new ReentrantLock();

lock.lock();           // hold count = 1
try {
    lock.lock();       // hold count = 2 — SAME thread, allowed
    try {
        // nested work
    } finally {
        lock.unlock(); // hold count = 1
    }
} finally {
    lock.unlock();     // hold count = 0 → fully released
}
```

---

## `ReadWriteLock` and `ReentrantReadWriteLock`

The `ReadWriteLock` interface (`java.util.concurrent.locks.ReadWriteLock`) maintains a **pair of associated locks** — one for **read-only operations** and one for **write operations**.

The idea is simple:

- **Multiple threads can read at the same time** (readers don't block each other).
- **Only one thread can write at a time** (writers block everyone).
- **A writer excludes both readers and other writers**.

This dramatically improves performance when you have **many reads and few writes**.

---

### Why `ReadWriteLock` Exists

With a normal `Lock` (or `synchronized`), only **one thread** can access the critical section at a time — even if all threads are just reading.

```text
Normal Lock:
   Reader-1 ─┐
   Reader-2 ─┤──►  LOCK  ──►  Only ONE thread at a time
   Reader-3 ─┘
```

But with `ReadWriteLock`:

```text
ReadWriteLock:
   Reader-1 ─┐
   Reader-2 ─┤──►  READ LOCK  ──►  All readers can enter together
   Reader-3 ─┘

   Writer-1 ────►  WRITE LOCK ──►  Alone — blocks all readers and writers
```

**Bottom line:** Use `ReadWriteLock` when reads are far more frequent than writes.

---

## The Interface

```java
package java.util.concurrent.locks;

public interface ReadWriteLock {

    /**
     * Returns the lock used for reading.
     * Multiple threads can hold the read lock simultaneously
     * as long as no thread holds the write lock.
     */
    Lock readLock();

    /**
     * Returns the lock used for writing.
     * Only one thread can hold the write lock at a time,
     * and it excludes all readers.
     */
    Lock writeLock();
}
```

---

### The Rules

```text
                    ReadWriteLock Rules
                            │
        ┌───────────────────┼───────────────────┐
        │                   │                   │
   Read Lock            Write Lock          Combination
        │                   │                   │
   Shared.              Exclusive.          Read + Write
   Many readers         Only one writer     NEVER at the
   at once.             at a time.          same time.
```

Concretely:

```text
Reader + Reader   →  ALLOWED   (both run concurrently)
Reader + Writer   →  BLOCKED   (writer must wait, or reader must wait)
Writer + Writer   →  BLOCKED   (only one writer at a time)
Writer + Reader   →  BLOCKED   (same as above)
```

---

### Common Implementation: `ReentrantReadWriteLock`

```java
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

ReadWriteLock rwLock = new ReentrantReadWriteLock();

Lock readLock  = rwLock.readLock();
Lock writeLock = rwLock.writeLock();
```

`ReentrantReadWriteLock` is the standard implementation. It supports:

- **Reentrancy** — a thread can re-acquire a lock it already holds.
- **Fairness mode** — optional (constructor with `true`).
- **Lock downgrading** — a writer can acquire the read lock, then release the write lock.
- **Lock upgrading is NOT allowed** — a reader cannot acquire the write lock while holding the read lock (this causes deadlock).

---

### Basic Usage Pattern

#### Reading

```java
Lock readLock = rwLock.readLock();

readLock.lock();

try {
    // Read the shared data
    // Multiple threads can be here at the same time
    System.out.println("Reading: " + sharedData);
} finally {
    readLock.unlock();
}
```

#### Writing

```java
Lock writeLock = rwLock.writeLock();

writeLock.lock();

try {
    // Modify the shared data
    // Only ONE thread can be here, and no readers allowed
    sharedData = newValue;
    System.out.println("Writing: " + sharedData);
} finally {
    writeLock.unlock();
}
```

---

### Full Working Example

```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class ReadWriteLockExample {

    // Shared resource
    private static final Map<String, String> cache = new HashMap<>();

    // The ReadWriteLock
    private static final ReadWriteLock rwLock = new ReentrantReadWriteLock();

    // Extract the two locks
    private static final Lock readLock  = rwLock.readLock();
    private static final Lock writeLock = rwLock.writeLock();

    // READ operation — many threads can do this together
    public static String read(String key) {
        readLock.lock();
        try {
            System.out.println(Thread.currentThread().getName()
                    + " is READING key=" + key);
            return cache.get(key);
        } finally {
            readLock.unlock();
        }
    }

    // WRITE operation — only one thread at a time
    public static void write(String key, String value) {
        writeLock.lock();
        try {
            System.out.println(Thread.currentThread().getName()
                    + " is WRITING key=" + key + " value=" + value);
            cache.put(key, value);
        } finally {
            writeLock.unlock();
        }
    }

    public static void main(String[] args) throws InterruptedException {

        // Seed the cache
        write("A", "Apple");

        // Spawn many reader threads
        for (int i = 0; i < 5; i++) {
            new Thread(() -> {
                System.out.println("Read result: " + read("A"));
            }, "Reader-" + i).start();
        }

        // Spawn a writer thread
        new Thread(() -> {
            write("A", "Apricot");
        }, "Writer-1").start();

        Thread.sleep(2000);
    }
}
```

#### Sample Output (order may vary)

```text
main is WRITING key=A value=Apple
Reader-0 is READING key=A
Reader-1 is READING key=A
Reader-2 is READING key=A
Reader-3 is READING key=A
Reader-4 is READING key=A
Read result: Apple
Read result: Apple
Read result: Apple
Read result: Apple
Read result: Apple
Writer-1 is WRITING key=A value=Apricot
```

Notice that **all 5 readers ran concurrently** — the writer had to wait for them to finish.

---

### Lock Downgrading vs Upgrading

#### Downgrading (Allowed)

A writer can acquire the read lock **before** releasing the write lock:

```java
writeLock.lock();
try {
    sharedData = newValue;
    readLock.lock();      // downgrade — still holding write lock
} finally {
    writeLock.unlock();   // release write lock, still hold read lock
}

try {
    // read safely — no writer can slip in
} finally {
    readLock.unlock();
}
```

#### Upgrading (NOT Allowed)

A reader **cannot** acquire the write lock while holding the read lock — this causes deadlock.

```java
readLock.lock();
try {
    writeLock.lock();   // ❌ DEADLOCK — never do this
} finally {
    readLock.unlock();
}
```

Why? Multiple readers may hold the read lock. They would all wait for the write lock, but the write lock waits for readers to release → circular wait.

---

### Fairness Mode

```java
ReadWriteLock unfair = new ReentrantReadWriteLock();       // default
ReadWriteLock fair   = new ReentrantReadWriteLock(true);   // FIFO ordering
```

- **Unfair:** Higher throughput, but writers may starve if readers keep arriving.
- **Fair:** Slower, but no starvation.






















