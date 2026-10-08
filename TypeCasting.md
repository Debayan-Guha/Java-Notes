# Upcasting

The process of casting an object of a subclass to a reference variable of it superclass.

Because a subclass inherently contains everything a superclass has (an "is-a" relationship, such as a `Dog` *is-a* `Animal`), Java allows this type conversion implicitly without requiring any explicit cast operator.


## Core Characteristics & Behavior

* **Implicit Conversion**: You do not need to write explicit cast syntax (like `(SuperClass)`); the compiler handles it automatically.
* **Always Safe**: Since every subclass object satisfies the superclass contract, upcasting can never cause a `ClassCastException` at runtime.
* **Loss of Subclass Specifics**: After upcasting, the reference variable can only access methods and variables defined in the **superclass** (or overridden methods in the subclass). Any methods exclusive to the subclass become hidden behind the superclass reference.

## Purpose & Use Cases

* **Primary Mechanism for Polymorphism**: Upcasting is primarily used to achieve polymorphism, allowing you to write generic code—meaning code that applies generally to a whole class or group rather than being unique to one particular item—that can operate on objects of different subclasses through a common superclass reference.
* **Method Parameter Reusability**: Enables a single method or data structure to accept multiple different types of objects as long as they share a parent class or interface.
* **Enables Runtime Method Overriding**: By treating subclass objects as their superclass type, it allows dynamic method dispatch, ensuring that overridden methods execute based on the actual runtime object rather than the reference type.

## Code Example

```java
// Superclass
class Animal {
    void sound() {
        System.out.println("Animal makes a sound");
    }
}

// Subclass
class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Dog barks");
    }

    // Subclass-specific method
    void fetch() {
        System.out.println("Dog is fetching the ball");
    }
}

public class UpcastingExample {
    public static void main(String[] args) {
        // 1. Creating a subclass object
        Dog myDog = new Dog();

        // 2. Upcasting: Assigning Dog reference to an Animal reference (Implicit)
        Animal myAnimal = myDog; // Same as: Animal myAnimal = (Animal) myDog;

        // 3. Calling overridden method: calling dog method due to polymorphism
        // Dynamic Binding (or Late Binding) happens here at runtime: 
        // Even though 'myAnimal' is of reference type Animal, the JVM looks at the 
        // actual object in memory (Dog) at runtime and executes Dog's version of sound().
        myAnimal.sound();

        // COMPILE-TIME ERROR if attempted:
        // myAnimal.fetch(); 
        // Reason: compile time error not override method (only override method can be called, but method only present in subclass
        // causes compile time error)
    }
}
```

### Important Note: Reference Type vs Runtime Type

Java decides which methods are **callable** based on the **reference type** (`Animal`), but it decides which **implementation** to run based on the **runtime object type** (`Dog`).

```text
Reference type (Animal)   ->  decides what methods you can call
Runtime type   (Dog)      ->  decides which overridden method actually runs
```

So even though the reference is `Animal`, the overridden `sound()` from `Dog` runs at runtime. This is called **dynamic binding** (or **late binding**).



## Methods vs Variables: Binding

Methods use **dynamic binding** — the JVM resolves overridden methods at **runtime** based on the actual object type.

Variables use **static binding** — the JVM resolves variables at **compile time** based on the reference type.

```java
class Animal {
    String name = "Animal";

    void print() {
        System.out.println("Animal method");
    }
}

class Dog extends Animal {
    String name = "Dog";   // hides Animal's 'name', does NOT override it

    @Override
    void print() {
        System.out.println("Dog method");
    }
}

public class InstanceVariableExample {
    public static void main(String[] args) {
        Animal myAnimal = new Dog();          // Upcasting

        myAnimal.print();                     // Output: Dog method
                                              // -> method resolved at RUNTIME
                                              // -> uses actual object type (Dog)

        System.out.println(myAnimal.name);    // Output: Animal
                                              // -> variable resolved at COMPILE TIME
                                              // -> uses reference type (Animal)
    }
}
```
```text
Methods   ->  dynamic binding  ->  resolved at runtime  ->  object type decides
Variables ->  static binding   ->  resolved at compile time  ->  reference type decides
```

## Dynamic Method Dispatch: Quick-Recall Memory Hacks

* **Core Definition**: The runtime mechanism where the JVM looks past the reference type and executes the overridden method belonging to the **actual object in memory**.

* **Code Example**:
  ```java
  Animal myAnimal = new Dog(); // Upcasting
  myAnimal.sound();            // Output: "Dog barks" (Subclass version runs at runtime)
  ```

* **Why It Works**: Upcasting sets up the reference, and Dynamic Binding resolves the method call at runtime using the object's real type.

* **Interview Trap**: This only applies to methods, never to instance variables (variables use static binding based on the reference type).


---



# Downcasting

**Downcasting** is the process of casting a reference variable of a superclass back to a reference type of its subclass. 

Because a superclass reference does not automatically know if it points to a specific subclass object, downcasting is **explicit** and carries runtime risk.


## Core Characteristics & Behavior

* **Explicit Conversion Required**: You must explicitly provide the target subclass type in parentheses (e.g., `(Dog) myAnimal`). The compiler will not do this automatically.
* **Runtime Risk (`ClassCastException`)**: Unlike upcasting, downcasting can fail at runtime. If the superclass reference actually points to a different subclass (or just the base superclass itself), Java throws a `ClassCastException`.
* **Restores Subclass Specifics**: Once successfully downcast, you regain full access to all methods and variables exclusive to the subclass that were hidden behind the superclass reference.


## Why Downcasting is Needed

* **Accessing Subclass-Specific Features**: After upcasting an object to pass it into a generic method, you may later need to invoke a method that exists *only* in the subclass (e.g., calling `fetch()` which is defined in `Dog`, but not in `Animal`).
* **Type Recovery**: Recovering the precise concrete type after generic processing or container retrieval (like storing objects in a collection of type `Object` or `SuperClass`).


## The Safety Check: `instanceof` Operator

To prevent a fatal `ClassCastException` at runtime, you should **always** verify the actual object type using the `instanceof` operator before performing a downcast.


## Code Example

```java
// Superclass
class Animal {
    void sound() {
        System.out.println("Animal makes a sound");
    }
}

// Subclass
class Dog extends Animal {
    @Override
    void sound() {
        System.out.println("Dog barks");
    }

    // Subclass-specific method not present in Animal
    void fetch() {
        System.out.println("Dog is fetching the ball");
    }
}

public class DowncastingExample {
    public static void main(String[] args) {
        // 1. Upcasting: Dog object assigned to Animal reference
        Animal myAnimal = new Dog(); 

        // myAnimal.fetch(); -> COMPILE-TIME ERROR (Animal reference cannot see fetch)

        // 2. Safe Downcasting using instanceof check
        if (myAnimal instanceof Dog) {
            // Explicit cast to Dog to access subclass-specific methods
            Dog myDog = (Dog) myAnimal; 
            
            // Successfully calling subclass-specific method
            myDog.fetch(); // Output: Dog is fetching the ball
        }

        // 3. Unsafe Downcasting Example (Causes ClassCastException at runtime)
        Animal genericAnimal = new Animal();
        
        // COMPILES fine, but CRASHES AT RUNTIME with ClassCastException 
        // because genericAnimal is NOT a Dog instance!
        // Dog badDog = (Dog) genericAnimal; 
    }
}
```









