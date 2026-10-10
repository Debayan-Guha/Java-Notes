### 1. What are Access Modifiers and what is their visibility scope?
**Access modifiers control the visibility (scope) of classes, methods, constructors, and variables in Java.** They help enforce encapsulation by restricting access to sensitive data and methods, ensuring that internal object states can only be altered through approved, controlled entry points.

Java provides four distinct access levels controlled by three keywords (`public`, `protected`, `private`) and one implicit state (package-private/default):

| Modifier | Inside Same Class | Inside Same Package | Outside Package (Subclass Only) | Outside Package (World) |
| :--- | :---: | :---: | :---: | :---: |
| **`private`** | Yes | No | No | No |
| **Default (No keyword)** | Yes | Yes | No | No |
| **`protected`** | Yes | Yes | Yes (via Inheritance) | No |
| **`public`** | Yes | Yes | Yes | Yes |

---

### 2. What is the fundamental difference between Default (Package-Private) and `protected` access?
* **Default Access**: A member (variable, method, or class) without any access modifier is visible **only to classes within the exact same package**. Subclasses located in a different package *cannot* see or access default members.
* **`protected` Access**: A member is visible to all classes within the exact same package **AND** to subclasses located in completely different packages. 

```text
Package boundaries:

Same Package (com.app):
[BaseClass (protected/default members)] ◄─── Access Granted ─── [AnyClassInPackage]

Different Package (com.utils):
[SubClass extends BaseClass] ─── Can access protected members ───► (Access Granted)
[SubClass extends BaseClass] ─── Cannot see default members ────► (COMPILE ERROR)
```

```java
package packageA;

public class Parent {
    int defaultVar = 10;
    protected int protectedVar = 20;
}
```

```java
package packageB;
import packageA.Parent;

public class Child extends Parent {
    void display() {
        // System.out.println(defaultVar); // COMPILE-TIME ERROR: Not visible outside packageA
        System.out.println(protectedVar);  // ALLOWED: Visible through inheritance hierarchy
    }
}
```

---

### 3. What is the critical rule regarding protected members accessed from a different package?
Even though a subclass in a different package inherits a `protected` member, it can **only** access that member using its own subclass reference or through `super`. It **cannot** access the protected member using a parent class reference variable.

```java
package packageB;
import packageA.Parent;

public class StrangerChild extends Parent {
    void checkAccess() {
        this.protectedVar = 100; // ALLOWED: Accessing via its own reference
        super.protectedVar = 200; // ALLOWED: Accessing via super keyword
        
        Parent p = new Parent();
        // p.protectedVar = 300; // COMPILE-TIME ERROR: Parent reference cannot cross package border
    }
}
```

---

### 4. What are the rules for access modifiers when overriding methods in a subclass?
When overriding a method, the subclass method **cannot assign a weaker (more restrictive) access privilege** to it. It can either keep the same access level or make it wider (more public).

#### The Visibility Expansion Chain Rule:
`private` ──► `default` ──► `protected` ──► `public` *(Widest)*

```java
class SuperClass {
    protected void performAction() {
        System.out.println("Action performed");
    }
}

class SubClass extends SuperClass {
    // COMPILE-TIME ERROR: Attempting to assign weaker access ('default')
    /*
    @Override
    void performAction() { } 
    */

    // ALLOWED: Keeping it matching ('protected') or expanding it ('public')
    @Override
    public void performAction() {
        System.out.println("Public action performed");
    }
}
```

---

### 5. Can top-level classes or interfaces be marked as `private` or `protected`? Provide reasons.
**No, top-level classes and interfaces can only be declared as `public` or package-private (default).**

#### Structural Reasons:
* **Why not `private`?** A `private` top-level class would be visible only within the file it was declared in. Since a top-level class is the boundary of that file, making it private means no other class in the entire application could ever reference, instantiate, or extend it, rendering it completely useless dead code.
* **Why not `protected`?** The `protected` modifier is explicitly designed to grant access to subclasses across package boundaries. At the top-level file system layer, there is no parent class or surrounding context to establish inheritance before the class itself is defined, making `protected` logically invalid.

*Note: Inner/nested classes or interfaces located inside a class body scope **can** be marked as `private` or `protected`.*

```java
// Top-Level File Level
public class ValidClass { }     // Allowed
class AnotherValidClass { }     // Allowed (default package-private)

// protected class InvalidClass { } // COMPILE-TIME ERROR: Illegal modifier
// private class BadClass { }       // COMPILE-TIME ERROR: Illegal modifier
```

---

### 6. What is the interaction between `private` methods and method overriding?
**`private` methods cannot be overridden.** 

Because a `private` method is completely invisible to any subclass, a method with the exact same name and signature inside a subclass is treated as a completely brand-new declaration. It has no architectural connection to the parent class method, and adding the `@Override` annotation will cause a compile-time failure.

```java
class Alpha {
    private void compute() { }
}

class Beta extends Alpha {
    // This is NOT an override. It is just a completely distinct method belonging to Beta.
    // @Override // IF UNCOMMENTED: Causes a compile-time error
    public void compute() {
        System.out.println("Beta computation logic");
    }
}
```

---

### 7. Can a method have a higher access level than its class?
**Yes, a method can have a higher (wider) access modifier than its containing class, but its effective visibility is completely bottlenecked by the class's access level.**

For example, if you declare a `public` method inside a default (package-private) class, the method compiles perfectly. However, classes outside that package cannot see or instantiate the outer class, meaning they can never reach or call that `public` method anyway.

```java
class PackagePrivateClass { // Accessible only within the same package
    
    public void publicMethod() { 
        // Compiles fine, but effectively acts as package-private 
        // because outsiders cannot see 'PackagePrivateClass' to call it.
        System.out.println("High access method");
    }
}
```

---

### 8. Can we create an interface inside a class?
**Yes, you can declare an interface inside a class.** This is known as a **Nested Interface**. 

A nested interface is **implicitly `static`** and **implicitly `public`** by default, even if you omit those keywords. It is used to logically bind an interface to a specific class where its functional contract is highly relevant (e.g., a listener interface tied to a specific UI Component class).

```java
class Window {
    // Nested interface inside a class
    public interface VectorListener {
        void onResize(int width, int height);
    }
}

// Implementing the nested interface outside using dot (.) notation
class AppFrame implements Window.VectorListener {
    @Override
    public void onResize(int w, int h) {
        System.out.println("Dimensions changed to: " + w + "x" + h);
    }
}
```

---

### 9. What is the significance of access modifiers in code security and code maintenance?

#### Security Significance:
Access modifiers act as architectural security walls within your codebase. By keeping critical backend engine variables strictly `private`, you prevent external malicious code or unauthorized system modules from directly reading or corrupting sensitive values (like passwords, keys, or direct balance variables). This ensures your objects can only modify their state using your vetted, validated business methods.

#### Code Maintenance Significance:
* **Controlled APIs**: By minimizing `public` exposures and maximizing `private`/`protected` isolations, you reduce the surface area of your public API contract. 
* **Safe Refactoring**: When internal logic details are kept `private`, you can completely modify, delete, or optimize those private methods and variables anytime without breaking any external client code that consumes your class.
* **System Stability**: Restricting access levels limits the tightly coupled dependencies throughout your application, making bug isolation and updates vastly simpler.
