### 1. `final`

**What it means:** "Cannot be changed."

**Three uses:**

```java
final int x = 10;              // variable  -> value cannot be reassigned
final class MyClass { }        // class     -> cannot be extended
final void method() { }        // method    -> cannot be overridden
```

**Rules:**
- Final variable must be assigned exactly once.
- Final class cannot be subclassed (e.g., `String`, `Integer`).
- Final method cannot be overridden by a subclass.

**Common trap:**
```java
final List<String> list = new ArrayList<>();
list.add("A");       // allowed  -> modifying the object
list = new ArrayList<>();   // compile error -> reassigning the reference
```

`final` makes the **reference** constant, not the object.

---

### 2. `static`

**What it means:** Belongs to the class, not to any instance.

**Uses:**

```java
static int count;              // shared by all instances
static void method() { }       // called via ClassName.method()
static class Nested { }        // nested class without outer instance
static { /* init */ }          // static initializer block
```

**Key points:**
- Static members are loaded once when the class is loaded.
- Cannot access instance variables directly from a static context.
- Common for utility methods (`Math.max`) and constants (`Integer.MAX_VALUE`).

---

### 3. `volatile`

**What it means:** Always read/write directly from main memory, bypassing CPU cache.

**Guarantees:**
- Visibility — one thread's write is immediately seen by others.
- Ordering — prevents instruction reordering around the variable.

**Does NOT guarantee:**
- Atomicity — `count++` is still not thread-safe.

**Use case:** Simple flags shared between threads.

---

### 4. `synchronized`

**What it means:** Only one thread can execute the block at a time.

**Uses:**

```java
synchronized void method() { }           // locks on 'this'
synchronized (lock) { }                  // locks on 'lock'
static synchronized void method() { }    // locks on ClassName.class
```

**Guarantees:**
- Mutual exclusion.
- Visibility (a thread sees changes from the previous lock holder).
- Atomicity of the block.

---


### 5. `instanceof`

**What it means:** Checks whether an object is an instance of a class or interface.

```java
if (obj instanceof String) {
    String s = (String) obj;
}
```

**Java 16+ pattern matching:**

```java
if (obj instanceof String s) {
    System.out.println(s.length());    // no cast needed
}
```

Returns `false` if `obj` is `null`.

---

### 6. `this`

**What it means:** A reference to the current object.

**Uses:**
```java
this.name = name;                 // differentiate field from parameter
this();                           // call another constructor
return this;                      // return the current object
```

Cannot be used inside a static context.

---

### 7. `super`

**What it means:** A reference to the parent class.

**Uses:**
```java
super.method();                   // call parent method
super(args);                      // call parent constructor
super.field;                      // access parent field
```

Must be the first statement in a constructor when calling `super(...)`.

---

### 8. `throws` vs `throw`

**`throw`** — actually throws an exception.

```java
throw new IllegalArgumentException("bad input");
```

**`throws`** — declares that a method might throw an exception.

```java
void read() throws IOException { }
```

**Rule:** `throw` is used inside the method body. `throws` is used in the method signature.

---

### 9. `try` / `catch` / `finally`

```java
try {
    // risky code
} catch (IOException e) {
    // handle
} finally {
    // always runs
}
```

**Order:**
1. `try` runs.
2. On exception, matching `catch` runs.
3. `finally` runs in both cases (exception or no exception).
4. If no matching catch, exception propagates after `finally` runs.

**Java 7+ try-with-resources:**

```java
try (BufferedReader br = new BufferedReader(new FileReader("f.txt"))) {
    // br is closed automatically
}
```

---

### 10. `this()` vs `super()`

**`this()`** — calls another constructor in the **same** class.

**`super()`** — calls a constructor in the **parent** class.

Both must be the **first statement** in a constructor. You cannot use both in the same constructor.
