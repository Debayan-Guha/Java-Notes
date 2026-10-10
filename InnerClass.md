### 1. What are the different types of Nested Classes in Java?
Java allows you to define a class inside another class. These are broadly divided into two categories:

* **Static Nested Classes**: Declared with the `static` modifier. They behave like normal top-level classes but are packaged inside the outer class for logical grouping.
* **Inner Classes (Non-Static Nested Classes)**: Bound to the instance of the outer class. They are further divided into three types:
  1. **Member Inner Class**: Declared outside methods but inside the class body.
  2. **Local Inner Class**: Declared inside a method block.
  3. **Anonymous Inner Class**: A class without a name declared and instantiated in a single expression.

---

### 2. What is the fundamental difference between a Static Nested Class and a Member Inner Class?
* **Instance Association**: A Member Inner Class object is strictly tied to a specific instance of the Outer class. You cannot instantiate it without creating an Outer class object first. A Static Nested Class does not require an instance of the Outer class; it can be instantiated independently.
* **Access to Outer Members**: A Member Inner Class has direct access to **all** fields and methods of the Outer class, including `private` instance variables. A Static Nested Class **cannot** access non-static (instance) members of the Outer class directly; it can only access static members.

```text
Memory Association Layout:

Member Inner Class (Tied to an object):
Outer Object [ Instance Variables ] ──► Inner Object [ Can access Outer's state ]

Static Nested Class (Tied to the class type):
Outer Class Type [ Static Variables ] ──► Static Nested Object [ Cannot see Outer instances ]
```

```java
class Outer {
    private String instanceMsg = "Instance Variable";
    private static String staticMsg = "Static Variable";

    // 1. Member Inner Class (Non-static)
    class Inner {
        void display() {
            System.out.println(instanceMsg); // Direct access allowed
            System.out.println(staticMsg);   // Direct access allowed
        }
    }

    // 2. Static Nested Class
    static class Nested {
        void display() {
            // System.out.println(instanceMsg); // COMPILE-TIME ERROR: Cannot make static reference
            System.out.println(staticMsg);      // Allowed
        }
    }
}

public class Example {
    public static void main(String[] args) {
        // Instantiating a Member Inner Class (Requires Outer Object)
        Outer outerObj = new Outer();
        Outer.Inner innerObj = outerObj.new Inner();
        innerObj.display();

        // Instantiating a Static Nested Class (Does NOT require Outer Object)
        Outer.Nested nestedObj = new Outer.Nested();
        nestedObj.display();
    }
}
```

---

### 3. How do you instantiate a Member Inner Class from outside the Outer class?
Because a Member Inner Class requires an active enclosing Outer instance, you must instantiate the Outer class first, and then use the special `.new` syntax bound to that instance.

```java
class Outer {
    class Inner {
        void hello() { System.out.println("Hello from Inner Class"); }
    }
}

public class Main {
    public static void main(String[] args) {
        // Syntax Variant 1: Step-by-step
        Outer outer = new Outer();
        Outer.Inner inner1 = outer.new Inner();

        // Syntax Variant 2: Inline single statement
        Outer.Inner inner2 = new Outer().new Inner();
        
        inner1.hello();
    }
}
```

---

### 4. What is a Local Inner Class, and what are its scope restrictions?
A **Local Inner Class** is defined inside the body of a block, typically inside a method. It is only visible and instantiation-eligible within that specific method scope.

#### Core Rules for Local Inner Classes:
1. **Access Modifiers Forbidden**: You cannot mark a local class as `public`, `protected`, `private`, or `static`.
2. **Access to Local Variables ("Effectively Final")**: A local inner class can access local variables of the enclosing method *only* if those variables are **`final`** or **effectively final** (meaning their value is never altered after initialization). If you modify the local variable after the class reads it, the compiler throws an error.

```java
class MethodHolder {
    void processData() {
        int number = 42; // Effectively final local variable

        // Local Inner Class declared INSIDE the method
        class LocalInner {
            void print() {
                // Allowed because 'number' is never changed
                System.out.println("Local number: " + number); 
            }
        }

        // Must instantiate inside the method to use it
        LocalInner li = new LocalInner();
        li.print();
        
        // number = 100; // IF UNCOMMENTED: Causes compile error in line 10!
    }
}
```

---

### 5. What is an Anonymous Inner Class, and when should you use it?
An **Anonymous Inner Class** is a local class without a name that is declared and instantiated at the exact same time. It is used to override methods of an existing class or provide an implementation for an interface without writing a completely separate class file.

```java
interface Greeting {
    void sayHello();
}

public class AnonymousExample {
    public static void main(String[] args) {
        // Declaring and instantiating an anonymous implementation of the Greeting interface
        Greeting defaultGreeting = new Greeting() {
            @Override
            public void sayHello() {
                System.out.println("Hello World from Anonymous Class!");
            }
        }; // Notice the semicolon ending the statement

        defaultGreeting.sayHello();
    }
}
```

---

### 6. Can an Inner Class declare static fields or static methods?
* **Before Java 16**: No. Standard Member Inner Classes were prohibited from defining `static` methods or fields (unless they were compile-time constants marked `static final`). Static members belong to a class type standalone context, which contradicted the fact that inner classes are implicitly dependent on an outer instance lifecycle.
* **Java 16 and Later**: **Yes.** Java 16 introduced the ability for non-static inner classes to declare `static` members (methods, initializers, fields) to simplify application code patterns.

```java
class OuterContainer {
    class ModernInner {
        // Valid in modern Java versions (Java 16+)
        static int innerCounter = 0; 
        
        static void increment() {
            innerCounter++;
        }
    }
}
```

---

### 7. How do you resolve variable shadowing inside an Inner Class? (The `this` keyword rules)
If an inner class defines a variable with the exact same name as a variable in the outer class, the inner variable shadows the outer variable. To bypass shadowing and reference the outer class instance variable explicitly, use the syntax `OuterClassName.this.variableName`.

```java
class ShadowClass {
    int value = 10; // Outer instance variable

    class InnerShadow {
        int value = 20; // Inner instance variable shadows outer variable

        void printValues() {
            int value = 30; // Local variable shadows both

            System.out.println("Local variable: " + value);               // 30
            System.out.println("Inner Class variable: " + this.value);    // 20
            System.out.println("Outer Class variable: " + ShadowClass.this.value); // 10
        }
    }
}

public class Example {
    public static void main(String[] args) {
        new ShadowClass().new InnerShadow().printValues();
    }
}
```
