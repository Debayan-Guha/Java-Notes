### 1. What is Abstraction, and what is its primary purpose?
**Abstraction is the process of hiding the complex internal implementation details of a system and showing only the essential, high-level features to the user.** 

Its primary purpose is to **reduce complexity and isolate system modules**. By focusing on *what* an object does rather than *how* it does it, abstraction allows developers to interact with simplified interfaces without needing to understand the underlying backend code execution engine.

---

### 2. How do Abstraction and Encapsulation differ?
While both concepts deal with isolation, they operate at completely different conceptual layers:

* **Abstraction (Design Layer)**: Focuses on hiding complexity by creating a simplified boundary or interface contract (*"What does this object do?"*). Achieved via **Abstract Classes** and **Interfaces**.
* **Encapsulation (Implementation Layer)**: Focuses on hiding data access variables by bundling state data and code together securely within a defensive boundary (*"How do I secure this object's fields?"*). Achieved via **Private Variables** and **Public Accessors**.

```text
Conceptual Comparison:

[ Abstraction: The Outer Dashboard ] ──► Exposes only the clean contract buttons.
    └─► [ Encapsulation: The Internal Safe ] ──► Locks down private fields behind methods.
```

---

### 3. What is the difference between Partial Abstraction and 100% Total Abstraction?
Java achieves different scales of abstraction depending on the blueprint tool chosen:
* **Partial Abstraction (0% to 100%)**: Provided by **Abstract Classes**. Because an abstract class can mix both abstract methods (unimplemented contracts) and concrete methods (fully implemented behaviors), it offers a hybrid, multi-layered abstraction model.
* **Total Abstraction (100% Contract)**: Historically provided by **Interfaces**. An interface traditionally contains strictly abstract method declarations, acting as a completely pure specification contract. 

*Note: With the introduction of `default` and `static` methods in Java 8+, interfaces can now carry implementation details, shifting modern interface architecture closer to a hybrid model.*

---

### 4. Summary Guide: Abstract Class vs. Interface

| Feature | Abstract Class | Interface |
| :--- | :--- | :--- |
| **Abstraction Level** | Partial to Total (0% - 100%). | 100% pure contract (Pre-Java 8). |
| **Multiple Inheritance** | **Not supported** (A class can only extend one parent). | **Supported** (A class can implement multiple interfaces). |
| **Variables** | Can have instance fields (`final`, `static`, or non-final). | Fields are strictly implicitly `public static final` constants. |
| **Constructors** | **Supported** (Invoked during subclass instantiation chaining). | **Not supported** (Cannot maintain instance states). |
