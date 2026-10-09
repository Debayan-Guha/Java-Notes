# Abstract Class in Java

An **abstract class** is a restricted class that cannot be instantiated on its own and serves as a common template or blueprint for subclasses to extend and implement.


## Key Characteristics

* **Instantiation Limitations**: You cannot create an object of an abstract class using the `new` keyword (e.g., `Shape s = new Shape();` results in a compile-time error). It serves strictly as a blueprint to be extended by other classes.
* **Mixed Method Types**: Can contain both **abstract methods** (methods without a body that *must* be implemented by subclasses) and **concrete methods** (fully implemented methods that subclasses inherit directly).
* **Mandatory Subclass Implementation**: Any non-abstract subclass extending an abstract class *must* provide implementations for all inherited abstract methods in the chain. If any abstract method is left unimplemented, the subclass itself must be declared abstract.
* **Constructor Support**: Abstract classes **can have constructors**, which are automatically **called when a concrete subclass is instantiated** (via `super(...)`), even though you **cannot** create an object of the abstract class directly. This structural support means an abstract class can safely house constructors, instance variables, and static methods, all of which execute, resolve, or initialize within the runtime context of the subclass execution.
* **Variable Flexibility**: Abstract classes **can have instance variables (fields)**. Unlike interfaces where fields are implicitly public, static, and final, fields in an abstract class can be **`final`**, **`static`**, or **non-final** (regular fields) with any access modifier.
* **Inheritance Hierarchy & Abstract Extensions**: Abstract classes **can extend other abstract classes**, allowing you to build multi-layered type structures where methods can remain unimplemented. However, a concrete subclass down the line **must implement all abstract methods in the entire hierarchy**—not just the ones declared in its immediate parent. If any abstract method from any ancestor is left unimplemented, the subclass cannot be instantiated and must also be declared abstract.


### Examples

#### 1. Instantiation Limitations Example

```java
abstract class Shape {
    abstract void draw();
}

public class Example {
    public static void main(String[] args) {
        // Compile-time error: Shape is abstract; cannot be instantiated
        // Shape s = new Shape(); 
    }
}
```

#### 2. Mixed Method Types Example

```java
abstract class Shape {
    // 1. Abstract method: Has no body, must be overridden by a subclass
    abstract double calculateArea();

    // 2. Concrete method: Fully implemented, inherited directly by subclasses
    void printDetails() {
        System.out.println("This is a geometric shape tool.");
    }
}

class Circle extends Shape {
    double radius = 5.0;

    @Override
    double calculateArea() {
        return Math.PI * radius * radius;
    }
}

public class Example {
    public static void main(String[] args) {
        Circle c = new Circle();
        c.printDetails(); // Invoking inherited concrete method
    }
}
```

#### 3. Mandatory Subclass Implementation Example

```java
abstract class Shape {
    abstract void render();
    abstract void resize();
}

// Subclass leaves 'resize()' unimplemented, so it MUST be declared abstract
abstract class GraphicElement extends Shape {
    @Override
    void render() {
        System.out.println("Rendering graphic element...");
    }
}

// Concrete subclass must implement the missing 'resize()' method to compile
class ScreenBox extends GraphicElement {
    @Override
    void resize() {
        System.out.println("Resizing screen box...");
    }
}
```

#### 4. Constructor Support Example

```java
abstract class Shape {
    String color;

    // Abstract class constructor
    Shape(String color) {
        this.color = color;
        System.out.println("Shape constructor called for: " + color);   // Shape constructor called for: Red
    }

    abstract double calculateArea();
}

class Circle extends Shape {
    double radius;

    Circle(String color, double radius) {
        super(color);   // calls abstract class constructor
        this.radius = radius;
        System.out.println("Circle constructor called");                // Circle constructor called
    }

    @Override
    double calculateArea() {
        return Math.PI * radius * radius;
    }
}

public class Example {
    public static void main(String[] args) {
        Circle c = new Circle("Red", 5.0);
    }
}

/*
 * Output:
 * Shape constructor called for: Red
 * Circle constructor called
 */
```

#### 5. Variable Flexibility Example

```java
abstract class Shape {
    String color;                          // non-final instance variable
    final int sides = 0;                   // final instance variable
    static int shapeCount = 0;             // static variable

    Shape(String color) {
        this.color = color;
        shapeCount++;
    }

    abstract double calculateArea();
}

class Circle extends Shape {
    double radius;

    Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    double calculateArea() {
        return Math.PI * radius * radius;
    }
}

public class Example {
    public static void main(String[] args) {
        Circle c = new Circle("Red", 5.0);

        System.out.println(c.color);            // Red       (non-final)
        System.out.println(c.sides);            // 0         (final)
        System.out.println(Shape.shapeCount);   // 1         (static)
    }
}
```

#### 6. Inheritance Hierarchy & Abstract Extensions Example

```java
// Level 1: abstract class
abstract class Shape {
    abstract double calculateArea();       // abstract method
}

// Level 2: abstract class extends abstract class
abstract class TwoDShape extends Shape {
    abstract double calculatePerimeter();  // another abstract method
}

// Level 3: concrete class -> must implement ALL abstract methods
// from Shape (calculateArea) AND TwoDShape (calculatePerimeter)
class Circle extends TwoDShape {
    double radius;

    @Override
    double calculateArea() {
        return Math.PI * radius * radius;
    }

    @Override
    double calculatePerimeter() {
        return 2 * Math.PI * radius;
    }
}

public class Example {
    public static void main(String[] args) {
        Circle c = new Circle();
        c.radius = 5;

        System.out.println("Area: " + c.calculateArea());           // 78.53...
        System.out.println("Perimeter: " + c.calculatePerimeter()); // 31.41...
    }
}
```

**Hierarchy Structure:**

```text
Shape (abstract)      -> calculateArea()
   |
   v
TwoDShape (abstract)  -> calculatePerimeter()
   |
   v
Circle (concrete)     -> implements calculateArea()
                      -> implements calculatePerimeter()
                      -> no abstract methods left -> can be instantiated
```

**Rule Definition:**

```text
Abstract extends abstract    -> allowed, may leave methods unimplemented
Concrete extends abstract    -> must implement ALL abstract methods in the hierarchy
```


---


# Why Use Abstract Classes?

1. **Code Reusability & Avoids Duplication**

By providing concrete methods and instance variables in the abstract class, all subclasses inherit them automatically without rewriting code (DRY principle).

2. **Enforces a Common Contract**

An abstract method has no body — it is only a declaration. Any concrete subclass **must** implement all inherited abstract methods, otherwise it also has to be declared abstract.

```text
Abstract method has no body
    -> someone must provide the body
    -> if a class extends the abstract class and wants to be CONCRETE,
       it must implement ALL inherited abstract methods
    -> if it does NOT implement all of them,
       the class itself must also be declared abstract
```

Example:

```java
abstract class Shape {
    abstract double calculateArea();     // no body
    abstract double calculatePerimeter(); // no body
}

// Concrete class -> implements BOTH abstract methods
class Circle extends Shape {
    double radius;

    @Override
    double calculateArea() { return Math.PI * radius * radius; }

    @Override
    double calculatePerimeter() { return 2 * Math.PI * radius; }
}

// Abstract class -> implements only ONE, so it must stay abstract
abstract class PartialShape extends Shape {
    @Override
    double calculateArea() { return 0; }

    // calculatePerimeter() still not implemented
    // -> class must be declared abstract, otherwise compile error
}
```
```text
Concrete subclass  ->  MUST implement ALL inherited abstract methods
Abstract subclass  ->  may leave some (or all) unimplemented
```

Concrete methods, on the other hand, already have a body. Subclasses **inherit them automatically** — you do not need to override or rewrite them. You simply **call them** and they run. You can override a concrete method **only if** you want different behavior. Overriding is optional.

```java
abstract class Shape {

    // Abstract method -> subclass MUST implement
    abstract double calculateArea();

    // Concrete method -> subclass inherits it, override is OPTIONAL
    void displayColor() {
        System.out.println("Shape color");
    }
}

class Circle extends Shape {
    double radius;

    // Forced to implement (abstract method)
    @Override
    double calculateArea() {
        return Math.PI * radius * radius;
    }

    // Option 1: inherit as-is (no override) -> uses Shape's displayColor()
}

class Rectangle extends Shape {
    double length, width;

    // Forced to implement (abstract method)
    @Override
    double calculateArea() {
        return length * width;
    }

    // Option 2: override because we want different behavior
    @Override
    void displayColor() {
        System.out.println("Rectangle color");
    }
}

public class Example {
    public static void main(String[] args) {
        Circle c = new Circle();
        c.radius = 5;

        Rectangle r = new Rectangle();
        r.length = 4;
        r.width = 6;

        // Without calling displayColor(), nothing is printed.
        // Inheritance alone does NOT run the method.

        // Circle did NOT override -> calls Shape's version
        c.displayColor();   // Output: Shape color

        // Rectangle DID override -> calls its own version
        r.displayColor();   // Output: Rectangle color
    }
}
```

**Summary:**

```text
Abstract method -> subclass MUST implement (or be declared abstract itself)

Concrete method -> Option 1: inherit as-is (just call it to run it)
                -> Option 2: override it (only if you want different behavior)

Inheriting a method -> subclass HAS access to it
Calling a method    -> the method actually RUNS
No call             -> nothing happens, no output
```

3. **Partial Implementation**

An abstract class can provide **some** methods fully implemented (concrete) and leave **some** methods as declarations only (abstract). This mix is called **partial implementation** — the abstract class gives the skeleton, and subclasses fill in the missing details.

In other words, an abstract class can contain **both concrete methods and abstract methods together**. The concrete methods are shared logic that all subclasses reuse; the abstract methods are placeholders that each subclass must fill in.

```text
Abstract class = partial implementation
    |
    +-- Concrete methods  -> already implemented (shared logic)
    +-- Abstract methods  -> left blank for subclasses to fill in
```

**Code Example: Partial Implementation**

```java
// Abstract Class serving as a blueprint with partial implementation
abstract class Shape {
    String color;

    // Shared constructor (common setup)
    Shape(String color) {
        this.color = color;
    }

    /*
     * CONCRETE METHOD (Shared / Fully Implemented):
     * All subclasses automatically get this exact behavior without writing it.
     */
    void displayColor() {
        System.out.println("Shape color: " + color);
    }

    /*
     * ABSTRACT METHOD (Forced Contract):
     * No body here; subclasses are forced to provide their own custom implementation.
     */
    abstract double calculateArea();
}

// Concrete Subclass 1
class Circle extends Shape {
    double radius;

    Circle(String color, double radius) {
        super(color);
        this.radius = radius;
    }

    @Override
    double calculateArea() {
        return Math.PI * radius * radius;
    }
}

// Concrete Subclass 2
class Rectangle extends Shape {
    double length, width;

    Rectangle(String color, double length, double width) {
        super(color);
        this.length = length;
        this.width = width;
    }

    @Override
    double calculateArea() {
        return length * width;
    }
}

public class AbstractWhyExample {
    public static void main(String[] args) {
        Shape circle = new Circle("Red", 5.0);
        Shape rectangle = new Rectangle("Blue", 4.0, 6.0);

        // Both subclasses reuse the shared concrete method from the abstract class
        circle.displayColor();    // Output: Shape color: Red
        rectangle.displayColor(); // Output: Shape color: Blue

        // Both execute their own specific implementation of the abstract method
        System.out.println("Circle Area: " + circle.calculateArea());
        System.out.println("Rectangle Area: " + rectangle.calculateArea());
    }
}
```


















