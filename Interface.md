# Interface in Java

An **interface** is a reference type in Java that acts as a pure contract or blueprint. It specifies *what* a class should do, but not *how* it should do it, enabling multiple inheritance and loose coupling.

## Key Characteristics

* **Instantiation Limitations**: You cannot create an object of an interface using the `new` keyword (e.g., `Vehicle v = new Vehicle();` results in a compile-time error). It serves strictly as a specification to be implemented by other classes.
* **Mixed Method Types**: Traditionally can only contain **abstract methods** (implicitly `public` and `abstract` without a body). However, since Java 8+, it can also contain **concrete methods** using the `default` and `static` keywords, as well as `private` methods (Java 9+).
* **Mandatory Subclass Implementation**: Any non-abstract class implementing an interface *must* provide implementations for all of its abstract methods. If any abstract method is left unimplemented, the implementing class itself must be declared abstract.
* **No Constructor Support**: Interfaces **cannot have constructors** and **cannot maintain instance state**. They do not execute any instantiation logic via `super()` because they contain no instance variables or instance initializers to set up.
* **Variable Restrictions**: Interfaces **cannot have instance variables (fields)**. Any field declared inside an interface is implicitly **`public`**, **`static`**, and **`final`** (a constant). It must be initialized immediately at declaration.
* **Inheritance Hierarchy & Interface Extensions**: An interface **can extend multiple other interfaces** using the `extends` keyword. Unlike classes, a single class can also **implement multiple interfaces**, allowing a form of multiple inheritance that is forbidden with abstract classes.



### Examples

#### 1. Instantiation Limitations Example

```java
interface Vehicle {
    void start();
}

public class Example {
    public static void main(String[] args) {
        // Compile-time error: Vehicle is abstract; cannot be instantiated
        // Vehicle v = new Vehicle(); 
    }
}
```

#### 2. Mixed Method Types Example

```java
interface Vehicle {
    // 1. Abstract method: Implicitly public and abstract, has no body
    void start();

    // 2. Concrete method (Java 8+): Declared using the 'default' keyword
    default void honk() {
        System.out.println("Beep beep! (Default sound)");
    }
}

class Car implements Vehicle {
    @Override
    public void start() {
        System.out.println("Car engine started.");
    }
}

public class Example {
    public static void main(String[] args) {
        Car myCar = new Car();
        myCar.start(); // Invoking implemented abstract method
        myCar.honk();  // Invoking inherited default method
    }
}
```

#### 3. Mandatory Subclass Implementation Example

```java
interface Vehicle {
    void accelerate();
    void brake();
}

// Subclass leaves 'brake()' unimplemented, so it MUST be declared abstract
abstract class PrototypeCar implements Vehicle {
    @Override
    public void accelerate() {
        System.out.println("Speeding up...");
    }
}

// Concrete subclass must implement the missing 'brake()' method to compile
class ProductionCar extends PrototypeCar {
    @Override
    public void brake() {
        System.out.println("Stopping the vehicle...");
    }
}
```

#### 4. No Constructor Support Example

```java
interface Vehicle {
    // Compile-time error: Interfaces cannot have constructors
    /*
    Vehicle() {
        System.out.println("Interface constructor called");
    }
    */
    void drive();
}

class Sedan implements Vehicle {
    Sedan() {
        super(); // Calls Object's constructor, NOT the interface
        System.out.println("Sedan constructor called.");
    }

    @Override
    public void drive() {
        System.out.println("Driving sedan smoothly.");
    }
}
```

#### 5. Variable Restrictions Example

```java
interface Vehicle {
    int MAX_SPEED = 120; // Implicitly public, static, and final (Constant)
    // int currentSpeed; // Compile-time error: Variable must be initialized
}

class SportsCar implements Vehicle {
    void checkSpeed() {
        // MAX_SPEED = 150; // Compile-time error: Cannot assign a value to a final variable
        System.out.println("Max allowed speed: " + MAX_SPEED);
    }
}

public class Example {
    public static void main(String[] args) {
        // Accessed directly via the Interface name because it is static
        System.out.println(Vehicle.MAX_SPEED); // 120
    }
}
```

#### 6. Inheritance Hierarchy & Interface Extensions Example

```java
interface Engine {
    void startEngine();
}

interface GPS {
    void getCoordinates();
}

// An interface can extend MULTIPLE other interfaces
interface SmartVehicle extends Engine, GPS {
    void connectToWifi();
}

// Concrete class must implement ALL accumulated abstract methods
class AutonomousCar implements SmartVehicle {
    @Override
    public void startEngine() { System.out.println("Engine online."); }

    @Override
    public void getCoordinates() { System.out.println("Latitude: 40.7128"); }

    @Override
    public void connectToWifi() { System.out.println("Connected to 5G network."); }
}
```

---

# Why Use Interfaces?

1. **Achieves 100% Total Abstraction & Loose Coupling**

Interfaces isolate the API definition entirely from the code implementation detail. The consumer interacts strictly with the specification, making components easily interchangeable.

2. **Enforces a Common Contract**

An interface abstract method establishes a rigorous rule set. Any implementing concrete class **must** provide the implementation logic; otherwise, the compiler rejects it.

```text
Interface abstract method has no body
    -> an implementing class must provide the body
    -> if a class implements the interface and wants to be CONCRETE,
       it must implement ALL inherited abstract methods
    -> if it does NOT implement all of them,
       the implementing class itself must be declared abstract
```
