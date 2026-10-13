### 1. What is Encapsulation, and what is its primary purpose?
**Encapsulation is the mechanism of wrapping data (instance variables) and the code acting on that data (methods) together into a single unit (class).** 

Its primary purpose is to **hide the internal implementation details** of an object from direct outside access. Instead of allowing external components to reach inside and mutate state fields directly, encapsulation forces them to interact through a controlled, authorized gateway of public methods.

---

### 2. How do you implement strict Encapsulation in Java?
To implement a strictly encapsulated class structure, you must follow two essential steps:
1. **Private Fields**: Mark all instance variables with the **`private`** access modifier so they cannot be accessed or altered from outside the class boundary.
2. **Public Accessors**: Provide public getter and setter methods (`getX()` and `setX()`) to allow controlled retrieval and modification of those values.

```java
class BankAccount {
    // 1. Private variables block direct outside manipulation
    private double balance;

    // 2. Controlled access gateway via Public Methods
    public double getBalance() {
        return this.balance;
    }

    public void deposit(double amount) {
        // Enforcing security rules and data validation rules before mutating state
        if (amount > 0) {
            this.balance += amount;
            System.out.println("Deposited: " + amount);
        } else {
            System.out.println("Invalid deposit amount.");
        }
    }
}
```

---

### 3. What is Data Hiding versus Encapsulation? Are they the same?
**No, they are not completely identical. Data Hiding is a subset of Encapsulation.**
* **Encapsulation**: Focuses on **grouping/bundling** data and code together into a cohesive object. It is a broader design concept.
* **Data Hiding**: Focuses strictly on **restricting access levels** to an object's internal variables. You can achieve encapsulation without achieving complete data hiding if you lazily mark your variables as `public`. True encapsulation requires data hiding to be effective.

#### Code Example Demonstrating the Difference

```java
// Example 1: Encapsulation WITHOUT Data Hiding
class LazyAccount {
    // Data and methods are bundled together (Encapsulation is present)
    public double balance; // CRITICAL MISTAKE: Public access bypasses Data Hiding

    public void display() {
        System.out.println("Balance: " + balance);
    }
}

// Example 2: True Encapsulation WITH Data Hiding
class SecureAccount {
    // 1. Data Hiding implemented: The variable is hidden from outside view
    private double balance; 

    // 2. Encapsulated gateway method to view data securely
    public double getBalance() {
        return this.balance;
    }

    // 3. Encapsulated gateway method to validate modifications securely
    public void deposit(double amount) {
        if (amount > 0) {
            this.balance += amount;
        }
    }
}

public class Main {
    public static void main(String[] args) {
        // Test 1: Bypassing Data Hiding easily
        LazyAccount lazy = new LazyAccount();
        lazy.balance = -99999.0; // Allowed! Direct corruption because fields aren't hidden.

        // Test 2: Protected by Data Hiding
        SecureAccount secure = new SecureAccount();
        // secure.balance = 5000.0; // COMPILE-TIME ERROR: balance has private access in SecureAccount
        secure.deposit(500.0); // Allowed: State is updated only through the authorized gateway
    }
}
```


---

### 4. What are the key architectural advantages of using Encapsulation?
* **Data Validation**: Setter methods allow you to guard variables against garbage inputs or corrupt configurations by rejecting unexpected payloads before assignment.
* **Flexibility & Maintainability**: You can completely alter your internal data structures or variable types (e.g., changing an integer field to a long field) without breaking client apps, provided you keep your public method signatures matching.
* **Read-Only / Write-Only Classes**: You can make a class entirely read-only by omitting all setter methods, or write-only by omitting all getter methods.

```java
// Completely Read-Only Class Shape
public class SecureToken {
    private String hash;

    public SecureToken(String hash) {
        this.hash = hash;
    }

    // No setter provided means the value is completely unchangeable after instantiation
    public String getHash() {
        return this.hash;
    }
}
```

---

### 5. The Encapsulation Leak Trap: How mutable object references can break encapsulation
**Simply making a variable `private` and adding a getter does not guarantee perfect encapsulation if that variable points to a mutable object reference (like an array or a `java.util.Date`).**

If a public getter returns a direct reference to an internal mutable object, external code can modify the internal state of your encapsulated object **without using any setter method**, completely bypassing your validation rules.

#### Incorrect Version (Encapsulation Leak)
```java
import java.util.ArrayList;
import java.util.List;

class StudentProfile {
    private List<String> courses; // Mutable reference object

    public StudentProfile(List<String> courses) {
        this.courses = courses;
    }

    // CRITICAL LEAK: Returning the exact reference array pointer
    public List<String> getCourses() {
        return this.courses; 
    }
}

public class Main {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("Java");
        
        StudentProfile student = new StudentProfile(list);
        
        // LEAK EXPLOITATION: External modification without using a setter!
        student.getCourses().add("Hacking Code"); 
        
        System.out.println(student.getCourses()); // Output: [Java, Hacking Code]
    }
}
```

#### Correct Version (Safe Encapsulation via Defensive Copying)
To fix this security vulnerability, your getter must perform **Defensive Copying**—returning a brand-new copy of the collection data or wrapping it inside an unmodifiable wrapper so that outsiders cannot mutate your internal state arrays.

```java
import java.util.ArrayList;
import java.util.Collections;
import java.util.List;

class StudentProfileSecure {
    private List<String> courses;

    public StudentProfileSecure(List<String> courses) {
        // Defensive copy during object construction
        this.courses = new ArrayList<>(courses);
    }

    // SAFE ACCESSOR: Returns a read-only unmodifiable view wrapper
    public List<String> getCourses() {
        return Collections.unmodifiableList(this.courses); 
    }
}

public class MainSecure {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("Java");
        
        StudentProfileSecure student = new StudentProfileSecure(list);
        
        // This will now throw a Runtime Exception safely protecting the internal state!
        // student.getCourses().add("Hacking Code"); // java.lang.UnsupportedOperationException
    }
}
```
