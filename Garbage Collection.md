## 1. What is Garbage Collection and what are its advantages?

**Garbage Collection** is an automated memory management process in Java. In languages like C or C++, developers must manually allocate and free up memory using functions like `free()` or `delete`. If a developer forgets to free memory, it leads to memory leaks. 

Java automates this process through the **JVM (Java Virtual Machine)**. The Garbage Collector runs in the background, automatically tracking objects in the heap memory to identify which objects are still actively used by the application and which ones are unreferenced (dead). It then destroys the unreferenced objects and frees up their memory space.

**Advantages**

1. **Automatic Memory Management:**
   - Developers do not have to manually write code to release memory, which significantly reduces human error and development time.

2. **Prevention of Memory Leaks:**
   - By automatically sweeping away unreferenced objects, GC prevents memory from slowly filling up with abandoned data, protecting the application from running out of RAM.

3. **Enhanced Program Stability and Safety:**
   - It eliminates common manual memory errors such as **dangling pointers** (referencing memory that has already been freed) and **double-free errors** (attempting to free the same memory twice), making Java applications much more stable.

4. **Cleaner and More Maintainable Code:**
   - Developers can focus entirely on business logic and application features rather than spending time managing memory allocation and deallocation.
  


## 2. Where are objects created in memory? On Stack or Heap?

In Java, memory is divided into different sections, but when it comes to object creation, the short answer is: **All objects are created on the Heap, while references to those objects and local variables live on the Stack.**

To understand how Java manages memory during execution, let's break down the distinct roles of the Stack and the Heap:

1. **The Stack Memory**
   - **What it stores:** The Stack is responsible for storing **primitive values** (like `int`, `boolean`, `double`) and **reference variables** (the variable names that point to objects, such as `s1`, `s2`). It also tracks the execution of methods (keeping track of method call frames and local variables).
   - **Lifetime:** Stack memory has a strict **LIFO (Last-In, First-Out)** lifecycle. When a method is called, a new block (frame) is pushed onto the stack. As soon as the method finishes execution, that entire block is instantly popped off and wiped clean.
   - **Speed:** Allocation and deallocation on the stack are extremely fast.

2. **The Heap Memory**
   - **What it stores:** The Heap is a large, shared pool of memory dedicated to storing **all physical objects and arrays** created in Java (e.g., when you use the `new` keyword, or create string literals for the String Constant Pool).
   - **Lifetime:** Objects stored in the heap do not disappear when a method finishes. They persist as long as there is at least one active reference pointing to them. Once they become unreferenced, they are cleaned up asynchronously by the **Garbage Collector**.
   - **Speed:** Allocation on the heap is slower compared to the stack.


**Code Example**

```java
public class MemoryDemo {
    public static void main(String[] args) {
        int age = 25;                           // Primitive variable
        String s1 = new String("Java");         // Object creation via 'new'
    }
}
```

**Memory Diagram**

```text
                  JAVA MEMORY DURING MAIN METHOD EXECUTION
 
   STACK MEMORY (LIFO)                    HEAP MEMORY (Shared Pool)
┌────────────────────────┐             ┌───────────────────────────────┐
│                        │             │                               │
│  [ main() Frame ]      │             │  [ General Heap Area ]        │
│  ┌──────────────────┐  │             │  ┌─────────────────────┐      │
│  │ age = 25         │  │             │  │   new String()      │      │
│  │ (Primitive Value)│  │             │  │  (Physical Object)  │      │
│  ├──────────────────┤  │             │  └──────────┬──────────┘      │
│  │ s1 = [Reference] │──┼─────────────┼─────────────┘                 │
│  │ (Address Pointer)│  │             │             │                 │
│  └──────────────────┘  │             │             │                 │
│                        │             │             ▼ (Internal Ref)  │
│                        │             │  ┌─────────────────────────┐  │
│                        │             │  │  String Constant Pool   │  │
│                        │             │  │   ┌─────────────────┐   │  │
│                        │             │  │   │     "Java"      │   │  │
│                        │             │  │   │ (Literal Value) │   │  │
│                        │             │  │   └─────────────────┘   │  │
│                        │             │  └─────────────────────────┘  │
└────────────────────────┘             └───────────────────────────────┘
  (When main() finishes,                  (Objects persist here until 
   this frame is deleted.)                 swept by Garbage Collector.)
```


## 3. Which part of the memory is involved Garbage Collection?

The **Heap memory** is the primary region involved in Garbage Collection. 

While Java has other memory areas like the Stack, Metaspace, and Program Counter (PC) registers, the Garbage Collector exclusively focuses on managing and cleaning up the **Heap**, where all objects and arrays reside.



## 4. Who manages Garbage Collector?

The Garbage Collector is managed entirely and automatically by the **JVM (Java Virtual Machine)**. 

Developers do not manually allocate or free memory, nor do they directly control when the Garbage Collector runs. The JVM's runtime environment continuously monitors heap memory allocation, object lifespans, and memory pressure in the background to decide when and how to execute garbage collection.


## 5. How can be the Garbage Collector be requested?

While developers cannot *force* the JVM to run the Garbage Collector immediately, Java provides standard mechanisms to **request** garbage collection. 

You can make this request using either of the following methods:
1. `System.gc();` : This is a static convenience method provided by the `System` class. It is shorter, cleaner, and much more commonly used in everyday Java programming.
2. `Runtime.getRuntime().gc();` : This calls the `gc()` method on the singleton `Runtime` instance representing the current Java application environment. `System.gc()` is just a wrapper around this exact call.

Both methods do the exact same thing under the hood—they send a hint to the JVM that memory cleanup is requested. However, whether the JVM chooses to run the Garbage Collector at that exact moment is entirely up to its internal algorithm and memory management policies.

**Code Example**

```java
public class GarbageCollectionRequestDemo {
    
    public static void main(String[] args) {
        // Create an object
        GarbageCollectionRequestDemo obj = new GarbageCollectionRequestDemo();
        
        // Unreference the object making it eligible for GC
        obj = null;
        
        System.out.println("Object unreferenced. Requesting Garbage Collection...");
        
        // Requesting the JVM to run the Garbage Collector
        System.gc();
        
        // Alternatively, you can use:
        // Runtime.getRuntime().gc();
        
        System.out.println("GC request sent. (Note: JVM may or may not run it immediately).");
    }

    @Override
    protected void finalize() throws Throwable {
        // This method is called by the Garbage Collector just before the object is destroyed
        System.out.println("Garbage Collector has collected the object and finalized it.");
    }
}
```



## 6. Does `finalize()` being called mean GC happened?

**Yes.** If an object's `finalize()` method runs, it means the Garbage Collector *has* identified that the object is unreachable (dead), and it is in the process of clearing it out of memory. 

## 7. What are the different ways to make an object eligible for GC when it is no longer needed?

An object in Java becomes eligible for Garbage Collection the moment it loses all active references pointing to it, making it unreachable from any root references (such as local variables, active threads, or static fields). 

1. **Nullifying the Reference Variable**
   - **How it works:** You explicitly assign `null` to the reference variable pointing to the object. This severs the link between the stack variable and the heap object, leaving the object with zero references.
   - **Example:**
  ```java
  MyClass obj = new MyClass();
  // ... use the object ...
  obj = null; // The object is now unreferenced and eligible for GC
  ```

2. **Reassigning the Reference Variable**
   - **How it works:** If a reference variable is reassigned to point to a completely new object, its old object loses that reference. If no other references point to the old object, it immediately becomes eligible for GC.
   - **Example:**
  ```java
  MyClass obj = new MyClass();
  obj = new MyClass(); // The first object is abandoned and eligible for GC
  ```

3. **Going Out of Scope (Method Scope)**
   - **How it works:** Objects created inside a method (local variables) are automatically bound to that method's stack frame. Once the method finishes execution, its stack frame is destroyed, and all local reference variables cease to exist. If those objects are not returned or stored globally, they instantly become eligible for GC.
   - **Example:**
  ```java
  public void calculate() {
      MyClass localObj = new MyClass(); // Created on heap, referenced locally
      // ... do work ...
  } // Method ends here: localObj reference is destroyed, object becomes eligible for GC
  ```

4. **Island of Isolation (Unreachable Object Clusters)**
   - **How it works:** Sometimes, two or more objects reference each other but have no active references coming from outside (e.g., from stack variables). Even though they technically still have references, they form an "island of isolation" that is completely cut off from the rest of the application. The JVM's Garbage Collector is smart enough to detect these isolated clusters and sweep them all away together.
   - **Example:**
  ```java
  public class Test {
      Test reference;

      public static void main(String[] args) {
          Test obj1 = new Test();
          Test obj2 = new Test();

          obj1.reference = obj2; // obj1 points to obj2
          obj2.reference = obj1; // obj2 points to obj1

          obj1 = null;
          obj2 = null; 
          // Both objects reference each other, but have zero external roots. 
          // They form an isolated island and are eligible for GC.
      }
  }
  ```


## 8. What is the purpose of overriding `finalize()`?

The historical purpose of overriding the `finalize()` method was to perform **cleanup and resource deallocation** (such as closing database connections, closing file streams, or releasing network sockets) just before an unreferenced object was destroyed by the Garbage Collector.

When an object became eligible for collection, the Garbage Collector would automatically invoke its `finalize()` method, giving the object one last chance to clean up external resources before its memory was permanently reclaimed from the heap.


## 9. Is Garbage Collector a foreground / background thread?

The Garbage Collector runs as a **background thread** (specifically, a daemon thread) within the JVM. It operates asynchronously behind the scenes, continuously monitoring heap allocations and cleaning up unreferenced objects while your application's main foreground threads (user thread) execute business logic. 


## 10. How Garbage Collection works?

**Phase 1:** Marking (Identifying Live vs. Dead Objects)

The GC starts by figuring out what needs to stay and what can be deleted:

- **GC Roots:** The process starts from entry points called **GC Roots** (which include local stack variables, active thread references, static variables, and JNI references).
- **Reachability Analysis:** The GC traverses the reference graph outward from these roots. Any object reachable through an active chain of references is marked as **live (reachable)**. 
- Anything that cannot be reached from the GC Roots is considered unreferenced and marked as dead/garbage.

**Phase 2:** Sweeping and Compacting (Reclaiming Memory)

Once dead objects are identified, the collector cleans them up:

- **Sweeping:** The memory space occupied by unmarked objects is reclaimed, making those blocks available for future object allocations.
- **Compacting:** To prevent **memory fragmentation** (where free memory is scattered in tiny, unusable gaps between live objects), many collectors shift remaining live objects closer together to create a contiguous block of free memory.



## 11. Can you name commonly used Oracle's JVM & which GC strategy is used by it?

The most widely used implementation of Oracle's JVM is **HotSpot JVM** (delivered via Oracle JDK or open-source builds like OpenJDK).






