## 1. What are the different ways to create string objects?

- **Using string literal:** `String s1 = "decode";`

When a string is created using a string literal, the JVM checks the String Constant Pool for the same string value. If the value already exists, the existing String object inside the pool is reused, and its reference is assigned to the variable. If the value does not exist, a new String object is created inside the String Constant Pool (not as a separate object in the regular heap area).

- **Using the `new` keyword:** `String s1 = new String("D");`

When a string is created using the `new` keyword, the JVM creates a new String object in the heap memory. The string literal `"D"` is checked in the String Constant Pool, and if `"D"` is not already present, it is added to the pool. The string object created in the heap memory then references the object in the string pool that stores the literal value. However, `s1` refers to the new String object in the heap, not directly to the object in the String Pool.

If we want `s1` to refer to the pooled String object, we can use the `intern()` method.

```java
public class StringInternExample {

    public static void main(String[] args) {

        // 1. Create a string object in the heap memory
        // It also ensures "D" is in the String Constant Pool (SCP)
        String s1 = new String("D");

        // 2. Use intern() to get the reference from the String Constant Pool
        String s2 = s1.intern();

        // 3. Create a string literal (directly points to the SCP)
        String s3 = "D";

        // Verification
        System.out.println("s1 == s3: " + (s1 == s3)); // false (Heap address != SCP address)
        System.out.println("s2 == s3: " + (s2 == s3)); // true  (Both point to the same SCP address)
    }
}
```


## 2. What is String Constant Pool?

The String Constant Pool (also known as the String Pool) is a special memory area located inside the **Heap memory** where Java stores string literals. No two string objects can have the same value in a string constant pool. Because strings are heavily used in applications, creating a new object every time can waste a lot of memory. The String Constant Pool optimizes this by **reusing existing string objects**. If a string literal already exists in the pool, Java returns its reference instead of creating a duplicate object. 


## 3. Why is Java provided with a String Constant Pool since we can store objects in heap memory?

- **Memory Optimization and Efficiency:** 
Strings are the most frequently used data types in Java applications. If every literal created a brand new object in the standard heap memory, it would consume a massive amount of RAM and severely degrade performance. The String Constant Pool avoids this by allowing multiple variables to share the exact same string object.

- **Caching and Reusability:** 
The pool acts as a built-in cache. When you use string literals, the JVM automatically checks the pool first and reuses existing instances, reducing unnecessary object creation and lessening the burden on the Garbage Collector _(the background service in Java that automatically finds and deletes unused objects to free up RAM)_.

- **Safety via Immutability:** 
Because strings are immutable (cannot be changed), it is completely safe for multiple references to point to the exact same object in the pool without the risk of one reference accidentally modifying the data for another.


## 4. How many objects are created in the following snippet?

```java
String s1 = new String("Decode");
StringBuffer s2 = new StringBuffer("Decode");
```
Assuming `"Decode"` is **not already present** in the String Constant Pool.

**Statement 1 :-**

```java
String s1 = new String("Decode");
```

Creates **2 objects**:

1. `"Decode"` → String object in the **String Constant Pool**
2. `new String("Decode")` → new String object in the **heap**

```text
 String Constant Pool (Inside Heap)
┌─────────────────────────────────┐
│        ┌───────────────┐        │
│        │   "Decode"    │        │
│        └───────────────┘        │
└────────────────▲────────────────┘
                 │
                 │ (Internal Reference)
                 │
 Heap             │
┌────────────────┴────────────────┐
│        ┌───────────────┐        │
│        │  new String() │        │
│        └───────────────┘        │
└────────────────▲────────────────┘
                 │
                 │
                s1 (Stack Variable)
```

**Statement 2 :-**

```java
StringBuffer s2 = new StringBuffer("Decode");
```

Creates **1 new object**:

1. `new StringBuffer("Decode")` → StringBuffer object in the **heap**

The existing `"Decode"` String object from the String Constant Pool is used to initialize the StringBuffer.

```text
String Constant Pool
┌───────────────┐
│   "Decode"    │◄──────────────────┐
└───────────────┘                   │
                                    │
                                    │ References internal value
                                    │ (char array backing)
                                    │
Heap                                │
┌───────────────────────────────────┴┐
│           StringBuffer             │
└────────────────────────────────────┘
                  ▲
                  │
                  │
                 s2 (Stack)
```

**Total**

```text
String s1 = new String("Decode");              → 2 objects
StringBuffer s2 = new StringBuffer("Decode");  → 1 object

Total = 3 objects
```


## 5. How to compare two Strings in Java?

In Java, strings can be compared using different methods depending on whether you want to check for **reference equality** (memory address) or **content equality** (actual text value).

- **Using the `==` Operator (Reference Comparison)**
  - **What it does:** Checks if both string references point to the **exact same object in memory** (same memory address).
  - **Behavior:** It does **not** compare the actual characters inside the strings.
  - **Example:**
  ```java
  String s1 = "Hello";
  String s2 = "Hello"; // Pointing to the same pooled object
  String s3 = new String("Hello"); // Creates a new object in heap

  System.out.println(s1 == s2); // true (same reference in String Pool)
  System.out.println(s1 == s3); // false (different memory locations)
  ```
  
```txt
  String Constant Pool (Inside Heap)
┌─────────────────────────────────┐
│        ┌───────────────┐        │
│        │    "Hello"    │◄───────┼────────┐
│        └───────────────┘        │        │
└────────────────▲────────────────┘        │
                 │                         │
                 │                         │
                 │ Direct Pool Reference   │ Direct Pool Reference
                 │                         │
Heap             │                         │
┌────────────────┼────────────────┐        │
│        ┌───────┴───────┐        │        │
│        │  new String() │        │        │
│        └───────▲───────┘        │        │
└────────────────┼────────────────┘        │
                 │                         │
                 │ Internal                │
                 │ Reference               │
                 │                         │
                s3                         s2
              (Heap)                   (Literal)
                 ▲
                 │
                s1 (Literal Points to Pool)
```
  
- **Using the `equals()` Method (Content Comparison)**
  - **What it does:** Compares the actual content/characters of two strings.
  - **Behavior:** It checks case sensitivity. If the characters match exactly, it returns `true`; otherwise, it returns `false`.
  - **Example:**
```java
  String s1 = "Hello";
  String s3 = new String("Hello");

  System.out.println(s1.equals(s3)); // true (contents are identical)
```
```txt
String Constant Pool (Inside Heap)
┌─────────────────────────────────┐
│        ┌───────────────┐        │
│        │    "Hello"    │◄───────┐
│        └───────────────┘        │
└────────────────▲────────────────┘
                 │                │
                 │                │ .equals() checks if the underlying
                 │ Internal       │ characters inside both targets 
                 │ Reference      │ match exactly ("Hello" == "Hello")
                 │                │
 Heap            │                │
┌────────────────┼────────────────┘
│        ┌───────┴───────┐        │
│        │  new String() │◄─────────── [ .equals() comparison ]
│        └───────▲───────┘        │                 ▲
└────────────────┼────────────────┘                 │
                 │                                  │
                 │                                  │
                s3                                 s1
           (Heap Object)                    (Pool Reference)

```


## 6. Why are strings made immutable in Java?

Strings in Java are immutable (unmodifiable), meaning once a `String` object is created, its internal state and character content cannot be changed. If you try to modify a string (such as using `concat()` or `toUpperCase()`), Java actually creates a brand-new string object in memory rather than altering the original one. 

Security is a major reason for string immutability. In Java, strings are frequently used to store sensitive information—such as usernames, passwords, and connection credentials—as well as to reference critical system resources like files, databases, and network connections. Because strings are immutable, once these values or resource paths are created and authenticated, they are locked and cannot be maliciously altered or tampered with by any other part of the application.

**Example**

```java
String s1 = "Decode";

s1 = s1.concat(" Java");
```
**What happens?**

Initially:
```txt
String Constant Pool

┌───────────────┐
│   "Decode"    │
└───────────────┘
        ▲
        │
        │ refers to
        │
       s1
```
After:
```java
s1 = s1.concat(" Java");
```
The original "Decode" object is not changed.

Instead, a new String object "Decode Java" is created.

```txt
String Constant Pool

┌───────────────┐
│   "Decode"    │ ◄────────────── Original object
└───────────────┘
        │
        │
        │

┌────────────────────┐
│   "Decode Java"    │ ◄──────── New object
└────────────────────┘
          ▲
          │
          │ refers to
          │
         s1
```
So:

Before concat():
```txt
s1 ─────► "Decode"
```

After concat():
```txt
"Decode"       ← remains unchanged
"Decode Java"  ← new String object

s1 ─────► "Decode Java"
```



## 7. Is String thread safe or not?

Yes, the `String` class in Java is **thread-safe**. 

Because Java strings are **immutable** (their internal state and character values cannot be changed after creation), multiple threads can read, share, and pass string objects concurrently across different parts of an application without any risk of data corruption. There is no need to write complex synchronization code or use explicit locks when sharing string references among threads, as their state is guaranteed to remain constant throughout their lifecycle.


## 8. when to use StringBuffer & StringBuilder?

While the standard `String` class is immutable (every modification creates a new object in memory), `StringBuffer` and `StringBuilder` are designed for scenarios where you need to perform **frequent modifications, concatenations, or manipulations** of strings without creating unnecessary garbage objects.

- **When to use `StringBuilder`**
  - **What it is:** A mutable sequence of characters that is **not thread-safe** and not synchronized.
  - **When to use it:** Use `StringBuilder` in **single-threaded** environments or local methods where performance and speed are the top priorities. Because it does not waste CPU cycles on synchronization locks, it executes significantly faster than `StringBuffer`.
  - **Example:**
```java
  StringBuilder sb = new StringBuilder("Hello");
  sb.append(" World"); // Modifies the existing object directly
  System.out.println(sb.toString()); // Output: Hello World
```

- **When to use `StringBuffer`**
  - **What it is:** A mutable sequence of characters that is **thread-safe** and synchronized.
  - **When to use it:** Use `StringBuffer` in **multi-threaded** environments where multiple threads might be modifying the same string buffer concurrently. Its methods are synchronized, ensuring that data corruption or race conditions do not occur (though this comes with a slight performance penalty due to lock overhead).
  - **Example:**
```java
  StringBuffer sb = new StringBuffer("Thread");
  sb.append("-Safe"); // Safely modified across multiple threads
  System.out.println(sb.toString()); // Output: Thread-Safe
```


**Quick Summary**

* **Use `String`** when the value is constant and rarely changes.
* **Use `StringBuilder`** for heavy string manipulation in a single thread (best performance).
* **Use `StringBuffer`** only when multiple threads need to safely modify the same string sequence concurrently.


## 9. What is String Interning?

**String interning** is a memory-optimization technique in Java where a string object is explicitly added to the **String Constant Pool** (if it isn't already there) so that it can be shared and reused.

When you call the `intern()` method on a `String` object, the JVM performs the following steps:
1. It checks if an identical string literal already exists in the String Constant Pool.
2. If it **exists**, the JVM returns the reference of the pooled string object.
3. If it **does not exist**, the JVM adds a copy of that string to the pool and returns its reference.

**Why use `intern()`?**

- **Memory Conservation:** If you have many duplicate string objects created in the heap (for example, parsed from a file or network response using the `new` keyword), interning allows them to share a single pooled instance, freeing the heap copies for garbage collection.
- **Faster Comparisons:** Once two strings are interned, you can use reference equality (`==`) instead of content equality (`equals()`), which compares memory addresses directly and executes much faster.

**Example:**
```java
String s1 = new String("Hello"); // Created in Heap (and pool if not present)
String s2 = s1.intern();         // Returns the reference from the String Constant Pool

String s3 = "Hello";             // Points directly to the pooled instance

System.out.println(s1 == s2);    // false (s1 is heap object, s2 is pool reference)
System.out.println(s2 == s3);    // true  (both point to the exact same pooled object)
```


## 10. If the garbage collector does not collect unreferenced objects from the string pool, will memory usage increase and eventually cause a crash?

**Yes, but with a major caveat:** Modern JVM implementations manage the String Constant Pool using a specialized internal cache (the `StringTable`), and unreferenced pooled strings **are** eligible for garbage collection, just like regular objects in the heap. 

**Before Java 7**

* **Memory Location:** The String Constant Pool was located in a separate, fixed-size memory area called **PermGen** (Permanent Generation) space.
* **Garbage Collection Behavior:** Because **PermGen** was distinct from the main **Heap memory**, strings inside the pool were rarely cleaned up by the **Garbage Collector**. 
* **The Problem:** If an application heavily **interned** strings or loaded too many literals, it would easily exhaust the tight **PermGen** limit, causing the application to crash with a `java.lang.OutOfMemoryError: PermGen space`.

**After Java 7 (Java 7 and Later)**

* **Memory Location:** The String Constant Pool was moved directly into the main **Heap memory** area.
* **Garbage Collection Behavior:** Because it now shares space with regular objects, pooled strings are fully subject to standard **Garbage Collection**. 
* **How Cleanup Works:** If a string is **interned** or created via a literal, but no variables reference it anymore, the **Garbage Collector** can safely sweep it away to reclaim memory. It will delete the string provided it is no longer referenced anywhere in the application or by the **JVM's internal tables**.










