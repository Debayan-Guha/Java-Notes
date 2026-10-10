### 1. What is Polymorphism, and what are its two core types?
**Polymorphism is the capability of an object to take on multiple forms.** In Java, it allows a single interface or parent reference type to represent distinct underlying concrete implementations. 

Java supports two core variations:
1. **Compile-time Polymorphism (Static Binding)**: Resolved by the compiler during compilation. Achieved via **Method Overloading**.
2. **Runtime Polymorphism (Dynamic Binding)**: Resolved by the JVM at execution runtime. Achieved via **Method Overriding** combined with **Upcasting**.

---

### 2. What is Method Overloading (Compile-time Polymorphism)?
Method Overloading occurs when a single class contains multiple methods sharing the **exact same name but possessing different parameter lists** (signatures). 

#### Compilation Rules for Overloading:
* **Signature Variance**: Overloaded methods **must** vary by their parameter lists in one of three ways:
  1. The **number** of input arguments.
  2. The **data types** of input arguments.
  3. The **sequence/order** of different argument data types.
* **Return Type & Modifiers**: Changing *only* the return type, access modifiers, or threw-exceptions list does **not** count as overloading. If the parameter signatures match exactly, changing the return type results in a **compile-time duplicate method error**.

```java
class Calculator {
    // Base Method
    int add(int a, int b) { return a + b; }

    // 1. Overloaded by parameter count
    int add(int a, int b, int c) { return a + b + c; }

    // 2. Overloaded by parameter data type
    double add(double a, double b) { return a + b; }

    // COMPILE-TIME ERROR: Duplicate method declaration (Changing return type only is illegal)
    // void add(int x, int y) { } 
}
```

---

### 3. What is Method Overriding (Runtime Polymorphism)?
Method Overriding occurs when a subclass provides a specialized, custom implementation for a method that it already inherited from its parent class.

#### Structural Rules for Overriding:
1. **Exact Signature Match**: The subclass method **must** match the parent method’s name, parameter count, and parameter data types exactly.
2. **Covariant Return Types**: The return type must match the parent's return type or be a **subclass** of it (known as a covariant return type).
3. **Access Modifier Rules**: The subclass method **cannot assign a weaker access privilege** than the parent method. It can only expand it (`private` ──► `default` ──► `protected` ──► `public`).
4. **Exception Constraints**: The overriding subclass method cannot throw broader or brand-new **checked exceptions** than the parent method.

```java
class Vehicle {
    protected Object fuelType() { return "Generic Fuel"; }
}

class Tesla extends Vehicle {
    // ALLOWED: Expanded visibility to 'public' and specialized return type to 'String' (Covariant)
    @Override
    public String fuelType() { return "Electricity"; } 
}
```

---

### 4. What is Dynamic Method Dispatch, and how does it execute?
**Dynamic Method Dispatch is the structural mechanism where an overridden method call is resolved dynamically at runtime rather than early at compile time.** 

This relies on **Upcasting** (assigning a subclass instance to a superclass reference variable). When you invoke an overridden method through an upcast reference variable, the JVM bypasses the variable's reference type declaration and looks directly at the **actual concrete object allocated in memory** to determine which implementation to run.

```java
class Animal {
    void makeSound() { System.out.println("Animal sound"); }
}

class Dog extends Animal {
    @Override
    void makeSound() { System.out.println("Dog barks"); }
}

public class DispatchDemo {
    public static void main(String[] args) {
        // Upcasting: Reference type is Animal, Runtime object type is Dog
        Animal myPet = new Dog();

        // Dynamic Binding occurs here: Prints "Dog barks"
        myPet.makeSound(); 
    }
}
```

---

### 5. Summary Check: Reference Type vs. Runtime Object Type
During upcasting and polymorphic code execution, memorize this strict behavioral rule:

```text
Reference Type (SuperClass)  ──► Decides WHAT methods/variables you are ALLOWED to call.
Runtime Type   (SubClass)    ──► Decides WHICH overridden method implementation ACTUALLY runs.
```

If a method is exclusive to the subclass and not declared inside the parent reference type, you will get a **compile-time error** if you attempt to call it directly through an upcast reference.

---

### 6. Do Instance Variables use Polymorphic Dynamic Binding?
**No. Instance variables do not support runtime polymorphism.** 

Variables use **Static Binding (Compile-time binding)**. When you access a field variable on an object, the JVM resolves the reference value strictly based on the **declared variable reference type**, completely ignoring the actual object type in memory. Subclasses cannot override instance variables; they can only **hide** (shadow) them.

```java
class Parent {
    String value = "Parent Field";
}

class Child extends Parent {
    String value = "Child Field"; // Hides parent variable
}

public class VariableBindingDemo {
    public static void main(String[] args) {
        Parent target = new Child(); // Upcasting
        
        // Static Binding: Prints "Parent Field" because the reference type is Parent
        System.out.println(target.value); 
    }
}
```

---

### 7. Can private, static, or final methods be overridden?
**No. Polymorphic dynamic dispatch is impossible for private, static, or final methods.**

* **`private` methods**: Are completely invisible to any subclass, meaning a matching signature in a child class is treated as a completely separate method rather than an override.
* **`static` methods**: Belong strictly to the class type blueprint rather than an instance. Redefining a static method in a subclass is known as **Method Hiding**, which resolves early via compile-time static binding.
* **`final` methods**: Explicitly carry a compiler lock that rejects any subclass overriding attempts, causing an instant compile-time crash.

```java
class Super {
    static void print() { System.out.println("Static Parent"); }
}

class Sub extends Super {
    // Method Hiding, NOT overriding
    static void print() { System.out.println("Static Child"); }
}

public class StaticBindingDemo {
    public static void main(String[] args) {
        Super obj = new Sub(); // Upcasting
        
        // Static Binding: Prints "Static Parent" based on the reference variable type
        obj.print(); 
    }
}
```

---

### 8. How does Method Overloading handle Type Promotion vs Autoboxing vs Varargs?
When an overloaded method is called, Java uses a strict precedence resolution chain to find the matching method. It prefers exact matching first, and falls back to more generalized alternatives in a precise order:

```text
Overloading Match Priority Order:
1. Exact Match ──► 2. Widening (Type Promotion) ──► 3. Autoboxing ──► 4. Varargs (Variable Arguments)
```

```java
class OverloadPriority {
    void process(long n) { System.out.println("1. Widening (long)"); }
    void process(Integer n) { System.out.println("2. Autoboxing (Integer)"); }
    void process(int... n) { System.out.println("3. Varargs (int...)"); }

    public static void main(String[] args) {
        OverloadPriority demo = new OverloadPriority();
        int primitiveInt = 10;
        
        // An int primitive prefers Widening over Autoboxing or Varargs!
        // Output: "1. Widening (long)"
        demo.process(primitiveInt); 
    }
}
```

---

### 9. The Constructor Polymorphism Trap: What happens when calling an overridden method inside a constructor?
**Never call an overridable method inside a constructor declaration.** 

When a parent class constructor executes during instantiation chaining, the object's subclass fields have **not yet been initialized** (they still contain blank binary default values like `0` or `null`). If the parent constructor invokes a polymorphic method that has been overridden in the child class, the dynamic binder will execute the child's version of that method using uninitialized fields.

```java
class Base {
    Base() {
        // DANGER TRAP: Calling an overridable method inside constructor
        init(); 
    }
    void init() { System.out.println("Base Init"); }
}

class Derived extends Base {
    String data = "Special Token"; // Field initialization happens AFTER 'super()'

    @Override
    void init() {
        // Prints: "Derived Init -> Data value: null"
        System.out.println("Derived Init -> Data value: " + data);
    }
}

public class ConstructorPolymorphismDemo {
    public static void main(String[] args) {
        new Derived();
    }
}
```

---

### 10. What is the Diamond Problem in Runtime Polymorphism, and how do Interfaces handle it? (Java 8+)
The **Diamond Problem** occurs when a class inherits multiple copies of a method with the same signature from different paths, creating implementation ambiguity. 

Because Java classes only allow single inheritance (`extends`), the diamond problem does not occur with classes. However, since Java 8, interfaces can contain **`default` methods with implementations**, which reintroduced this conflict. If a class implements two interfaces sharing the exact same `default` method signature, the compiler rejects the class with an **inheritance conflict error**.

```java
interface Alpha {
    default void execute() { System.out.println("Alpha Logic"); }
}

interface Beta {
    default void execute() { System.out.println("Beta Logic"); }
}

// Throws compile error unless 'execute()' is explicitly overridden to resolve ambiguity
class SystemTask implements Alpha, Beta {
    
    @Override
    public void execute() {
        // Manually resolving the conflict path via 'super' notation
        Alpha.super.execute(); 
    }
}

/*
 * Output:
 * Alpha Logic
 */
```

