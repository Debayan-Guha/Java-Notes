### 1. Can an interface be final?
**No, an interface cannot be `final`.** 

An interface is designed to be a contract that other classes *implement* or other interfaces *extend*. Marking it as `final` would prevent any class from implementing it, making the interface completely useless. If you attempt to use the `final` modifier in an interface header, the Java compiler will throw a **compile-time error** (`modifier final not allowed here`).

---

### 2. Can an interface extend a class?
**No, an interface cannot extend a class.**

Interfaces can only extend other interfaces. A class represents a concrete implementation blueprint that can have instance state (fields) and constructors, which goes against the pure contract design of an interface. An interface can only use the `extends` keyword followed by **other interfaces**. 

---

### 3. What happens if a class implements 2 interfaces with the same default method?
If a class implements two interfaces that declare the exact same `default` method signature, Java will throw a **compile-time conflict error** (`class inherits unrelated defaults`). 

Because Java does not allow multiple inheritance of state/implementation ambiguity (known as the *Diamond Problem*), the compiler forces you to **manually resolve the ambiguity** by overriding the method in the implementing class.

```java
interface InterfaceA {
    default void log() {
        System.out.println("Logging from A");
    }
}

interface InterfaceB {
    default void log() {
        System.out.println("Logging from B");
    }
}

// Throws compile error unless log() is explicitly overridden
class Logger implements InterfaceA, InterfaceB {
    
    @Override
    public void log() {
        // Option 1: Provide a completely new implementation
        System.out.println("Custom class logging logic");
        
        // Option 2: Explicitly resolve to a specific interface's version using super
        InterfaceA.super.log(); 
    }
}
```

---

### 4. Can an interface be declared inside a class?
**Yes, an interface can be declared inside a class.** This is known as a **Nested (or Inner) Interface**. 

It is used to group interfaces logically with the class they are closely related to (for example, the `Map.Entry` interface inside the `Map` class).

```java
class OuterClass {
    // Nested interface declared inside a class
    interface NestedInterface {
        void display();
    }
}

// To implement it, you reference it using the OuterClass name
class Implementer implements OuterClass.NestedInterface {
    @Override
    public void display() {
        System.out.println("Inside nested interface implementation!");
    }
}
```

---

### 5. Can a nested interface be non-static?
**No, nested interfaces are implicitly static.**

Even if you explicitly omit the `static` keyword when declaring a nested interface inside a class, the compiler automatically treats it as `static`. 

A non-static nested structure (like an inner class) requires an instance of the outer class to exist. Because interfaces do not have instance states or constructors, they cannot belong to a specific object instance of the outer class. They belong strictly to the class type itself.

```java
class Window {
    // Even without the 'static' keyword, this is implicitly static!
    interface Listener {
        void onClick();
    }
}

public class Example {
    public static void main(String[] args) {
        // You do NOT need to create a 'new Window()' object to reference it
        Window.Listener action = () -> System.out.println("Click detected!");
        action.onClick();
    }
}
```

---

### 6. Is an interface inside an interface possible?
**Yes, an interface can be declared inside another interface.** This is another variation of a **Nested Interface**.

Just like interfaces nested inside classes, an interface nested inside another interface is **implicitly `public` and `static`**, even if you do not explicitly type those keywords. It is utilized to logically group related behaviors under a single top-level namespace.

```java
interface CustomMap {
    void clear();

    // Nested interface declared inside another interface
    interface Entry {
        Object getKey();
        Object getValue();
    }
}

// A class can implement the nested interface directly using the dot (.) notation
class MapEntry implements CustomMap.Entry {
    @Override
    public Object getKey() {
        return "KeyData";
    }

    @Override
    public Object getValue() {
        return "ValueData";
    }
}
```

---

### 7. Can an interface have private methods? (Java 9+)
**Yes.** Starting from Java 9, interfaces can contain `private` and `private static` methods. They are used strictly as helper methods to encapsulate common code shared between multiple `default` or `static` methods within the same interface, preventing code duplication.

```java
interface Database {
    default void connect() {
        validateCredentials(); // Calling the private helper
        System.out.println("Connected to Database.");
    }

    default void disconnect() {
        validateCredentials(); // Reusing the private helper
        System.out.println("Disconnected.");
    }

    // Private helper method - hidden from implementing classes
    private void validateCredentials() {
        System.out.println("Validating security tokens...");
    }
}
```

---

### 8. What is a Marker Interface?
A **Marker Interface** (also known as a Tagging Interface) is an interface that **contains absolutely no methods or fields**. 

Implementing a marker interface tells the Java Virtual Machine (JVM) or a framework that the implementing class possesses a specific capability or behavior. Examples include `Serializable`, `Cloneable`, and `Remote`.

```java
// Implementing Cloneable markers this class as safe to duplicate via object.clone()
class UserProfile implements Cloneable {
    String username;
    
    // JVM checks if 'this instanceof Cloneable' behind the scenes
}
```

---

### 9. Can you declare static methods in an interface? What are the rules?
**Yes, since Java 8, you can declare static methods inside an interface.**

Interface static methods are designed to provide utility or helper functionalities directly related to the interface's domain, completely eliminating the need to create separate utility classes (like `Collections` or `Math`).

#### Core Rules for Interface Static Methods:
1. **Must Have a Body**: Unlike abstract methods, a static method inside an interface cannot be blank; it must provide a complete block implementation.
2. **Not Inherited by Implementing Classes**: Unlike `default` methods or standard class inheritance, interface static methods are **not** inherited by classes that implement the interface. They belong strictly to the interface container itself.
3. **Invocation via Interface Name Only**: Because they are not inherited, you cannot call an interface static method using an object reference variable or a subclass name. It must be called exclusively using the format `InterfaceName.methodName()`.
4. **Cannot Be Overridden**: Implementing classes can define a method with the exact same name and signature, but it is treated as a completely fresh class method (method hiding), not an override.

```java
interface UtilityService {
    // Declaring a valid static method with a body inside an interface
    static void printSystemLog(String message) {
        System.out.println("[SYSTEM LOG]: " + message);
    }
}

class ServiceRunner implements UtilityService {
    // Normal class logic
}

public class Example {
    public static void main(String[] args) {
        // RULE 3 IN ACTION: Must call via Interface Name
        UtilityService.printSystemLog("App initialized."); // Output: [SYSTEM LOG]: App initialized.

        ServiceRunner runner = new ServiceRunner();
        // runner.printSystemLog("Failed"); 
        // COMPILE-TIME ERROR: The method printSystemLog(String) is undefined for the type ServiceRunner
    }
}
```
