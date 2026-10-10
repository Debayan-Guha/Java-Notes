### 1. What is Inheritance, and what is its primary purpose?
**Inheritance is a structural mechanism in Java that allows one class to acquire the properties (fields) and behaviors (methods) of another class.** 

It establishes an **"is-a" relationship** (e.g., a `Dog` *is-a* `Animal`) between a child subclass and a parent superclass. Its primary purposes are:
* **Code Reusability**: Subclasses inherit existing code directly without duplication.
* **Polymorphism Support**: Provides the structural hierarchy required to achieve runtime dynamic method overriding and upcasting.

---

### 2. What are the different types of Inheritance supported by Java?
Java supports three types of inheritance with classes, while completely blocking two others due to structural ambiguity:

* **Supported via Classes**:
  1. **Single Inheritance**: A class extends exactly one parent class (`A ◄── B`).
  2. **Multilevel Inheritance**: A class extends a subclass, creating a chain (`A ◄── B ◄── C`).
  3. **Hierarchical Inheritance**: Multiple subclasses extend a single parent class (`A` is extended by both `B` and `C`).

* **NOT Supported via Classes**:
  1. **Multiple Inheritance**: A single class trying to extend more than one parent class directly (`class C extends A, B`).
  2. **Hybrid Inheritance**: A combination of multiple and multilevel inheritance paths forming a diamond shape.

```text
Class Inheritance Limitations Layout:

Supported (Multilevel):         Blocked (Multiple):
┌───────────┐                     ┌───────────┐     ┌───────────┐
│  Class A  │                     │  Class A  │     │  Class B  │
└─────▲─────┘                     └─────▲─────┘     └─────▲─────┘
      │                                 │                 │
┌─────┴─────┐                           └────────┬────────┘
│  Class B  │                                    │  (Illegal syntax)
└─────▲─────┘                              ┌─────┴─────┐
      │                                    │  Class C  │
┌─────┴─────┐                              └───────────┘
│  Class C  │
└───────────┘
```

---

### 3. Why does Java not support Multiple Inheritance with classes?
Java blocks multiple inheritance of classes to prevent the **Diamond Problem** (implementation ambiguity). 

If two separate parent classes (`Class A` and `Class B`) define the exact same method signature with different code bodies, and a child class (`Class C`) attempts to inherit from both, the JVM cannot decide which parent method implementation to execute when called on a child instance. To eliminate this ambiguity entirely, Java limits class inheritance to a single parent.

*Note: Multiple inheritance of **interface contracts** is fully allowed because interfaces do not force multiple state representations.*

---

### 4. How does Constructor Execution order work in an inheritance chain?
When a subclass object is instantiated, **the parent class constructor always executes first**, followed by the subclass constructor. 

This happens because the very first line of any subclass constructor implicitly contains a **`super()`** call inserted by the compiler. This ensures that the parent class establishes its initial memory fields and state details securely before the subclass code appends its specialized properties.

```java
class Ancestor {
    Ancestor() {
        System.out.println("1. Ancestor memory initialized.");
    }
}

class Descendant extends Ancestor {
    Descendant() {
        // super(); is implicitly placed here by the compiler
        System.out.println("2. Descendant memory initialized.");
    }
}

public class ExecutionDemo {
    public static void main(String[] args) {
        new Descendant();
    }
}
/*
 * Output:
 * 1. Ancestor memory initialized.
 * 2. Descendant memory initialized.
 */
```

---

### 5. What is the difference between Method Overriding and Instance Variable Hiding?
* **Methods (Overriding)**: Resolved via **Dynamic Binding (Runtime)**. The JVM looks past the reference variable type definition and executes the method based on the actual concrete runtime object type allocated on the heap.
* **Instance Variables (Hiding/Shadowing)**: Resolved via **Static Binding (Compile-time)**. Variables cannot be overridden. If a subclass declares a field with the exact same name as a parent field, it **hides** it. The value accessed depends strictly on the declared **reference variable type**.

```java
class Parent {
    String message = "Parent Field Data";
    
    void show() { System.out.println("Parent Method Called"); }
}

class Child extends Parent {
    String message = "Child Hidden Field Data"; // Hides parent variable
    
    @Override
    void show() { System.out.println("Child Method Called"); }
}

public class BindingDemo {
    public static void main(String[] args) {
        Parent reference = new Child(); // Upcasting

        // Dynamic Binding: Object type decides method execution
        reference.show(); // Output: Child Method Called

        // Static Binding: Reference type decides variable access
        System.out.println(reference.message); // Output: Parent Field Data
    }
}
```

---

### 6. What is the role of the `super` keyword in Inheritance?
The `super` keyword is a reference variable used to explicitly target members of the immediate parent class. It is utilized in three scenarios:
1. **Invoking Parent Constructors**: Used as `super()` or `super(args)` to chain constructor parameters to the parent class.
2. **Accessing Hidden Fields**: Bypasses local variable shadowing to access hidden parent instance variables (`super.fieldName`).
3. **Invoking Overridden Methods**: Calls the original parent class method implementation from within the overridden subclass method (`super.methodName()`).

```java
class Base {
    void display() { System.out.println("Base message"); }
}

class Derived extends Base {
    @Override
    void display() {
        super.display(); // Calls original parent implementation first
        System.out.println("Derived addition");
    }
}
```

---

### 7. Which class members are NOT inherited by a subclass?
A subclass inherits all non-private members of its parent class, with the following exceptions:
* **Constructors**: Constructors are never inherited; they belong strictly to their declaring class. They are only invoked via constructor chaining (`super()`).
* **Private Members**: Fields and methods marked `private` are encapsulated within the parent class body and remain completely hidden from the subclass.
* **Static Initialization Blocks**: Class blocks run when their specific class is loaded into memory, not via subclass initialization loops.

#### Code Example Demonstrating Non-Inherited Members

```java
class Parent {
    // 1. Private member (Encapsulated, not inherited)
    private String secretToken = "Vault-123";

    // 2. Static Initialization Block (Belongs to class type loading, not instance inheritance)
    static {
        System.out.println("Parent Class loaded into memory.");
    }

    // 3. Parent Constructor (Belongs strictly to Parent class)
    Parent() {
        System.out.println("Parent constructor invoked.");
    }

    private void privateMethod() {
        System.out.println("This is private.");
    }
}

class Child extends Parent {
    Child() {
        // Implicitly calls super() to chain up, but does NOT inherit the constructor structure
        super(); 
        System.out.println("Child constructor invoked.");
    }

    void testInheritance() {
        // COMPILE-TIME ERRORS if attempted:
        // System.out.println(this.secretToken);  // ERROR: secretToken has private access in Parent
        // this.privateMethod();                  // ERROR: cannot find symbol method privateMethod()
    }
}

public class Main {
    public static void main(String[] args) {
        System.out.println("--- Instantiating Child ---");
        Child c = new Child();
    }
}

/*
 * Output Sequence:
 * Parent Class loaded into memory.
 * --- Instantiating Child ---
 * Parent constructor invoked.
 * Child constructor invoked.
 */
```

---

### 8. What is the ultimate root class of the Java inheritance hierarchy?
Every single class in Java implicitly or explicitly extends the **`java.lang.Object`** class. If your class does not declare an explicit `extends` keyword, the compiler automatically appends `extends Object` to its definition. This guarantees that every Java object inherits baseline utility functionalities such as `toString()`, `equals()`, `hashCode()`, and thread synchronization markers like `wait()` and `notify()`.


#### Code Example Demonstrating the Implicit Object Root Class

```java
// No explicit 'extends' declared. The compiler automatically changes this to:
// class CustomRecord extends java.lang.Object
class CustomRecord {
    String title;

    CustomRecord(String title) {
        this.title = title;
    }
}

public class ObjectRootDemo {
    public static void main(String[] args) {
        CustomRecord record = new CustomRecord("Interview Prep");

        // Proof 1: You can call methods like toString() and hashCode() 
        // even though they were never explicitly declared inside CustomRecord.
        System.out.println("Implicit toString(): " + record.toString());
        System.out.println("Implicit hashCode(): " + record.hashCode());

        // Proof 2: An Object reference type variable can hold any instance in Java (Upcasting)
        Object objRef = record; 
        
        // Proof 3: Verification using the instanceof check
        if (record instanceof Object) {
            System.out.println("Verification Success: CustomRecord IS-A java.lang.Object");
        }
    }
}

/*
 * Sample Output:
 * Implicit toString(): CustomRecord@6bc7c054
 * Implicit hashCode(): 1808253012
 * Verification Success: CustomRecord IS-A java.lang.Object
 */
```


---

### 9. What are Sealed Classes, and how do they restrict inheritance? (Java 17+)
Introduced as a standard feature in **Java 17**, **Sealed Classes** allow a class or interface to explicitly declare and restrict **which specific subclasses are permitted to extend it**. This breaks the traditional binary model where a class was either wide open for anyone to inherit or completely locked down using the `final` keyword.

```java
// Permitting ONLY Circle and Square to extend Shape
public sealed class Shape permits Circle, Square { }

// Permitted subclass must be declared final, sealed, or non-sealed
public final class Circle extends Shape { }
public final class Square extends Shape { }

// COMPILE-TIME ERROR: Triangle is not allowed to extend Shape
// public final class Triangle extends Shape { }
```
