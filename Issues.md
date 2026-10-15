## 1. Multiple Bag Problem

### 1. What is Bag?

In JPA / Hibernate terminology, a **Bag** is simply a `List<T>` collection without any ordering and without a unique constraint on its elements.

#### Hibernate Collection Types Comparison

| Java Type Used in Entity | Hibernate Name | Ordered? | Duplicates Allowed? |
| :--- | :--- | :---: | :---: |
| `List<T>` (no `@OrderColumn`) | Bag | ❌ No | ✅ Yes |
| `List<T>` + `@OrderColumn` | List | ✅ Yes (by index) | ✅ Yes |
| `Set<T>` | Set | ❌ No | ❌ No |
| `Collection<T>` | Bag | ❌ No | ✅ Yes |

> **Key Takeaway:** If you just write `private List<Item> items;` in your entity — that's a Bag.


#### What is "Order" in Hibernate?

In Hibernate, **Order** refers strictly to the **sequence, position, or arrangement** of the items inside the collection (like 1st, 2nd, 3rd). 

Here is the simple, real-world difference between an **Unordered Collection (Bag)** and an **Ordered Collection (List)**:

##### 1. Unordered (Bag) = A Grocery Cart 🛒

* **In Hibernate:** If you save a Java `List` containing `["Apple", "Banana", "Milk"]`, the database just inserts three separate rows. When you fetch them tomorrow, the database might return them as `["Milk", "Apple", "Banana"]`. The original sequence is lost because the database table has no column to store *where* they were positioned.

##### 2. Ordered (List with `@OrderColumn`) = A Top-10 Music Chart 📊

Think of an **Ordered List** like a leaderboard or playlist. The exact position matters. If "Song A" is #1 and "Song B" is #2, you cannot mix them up.
* **In Hibernate:** To remember this sequence, Hibernate creates an extra column in your database table (usually called an index column).
* When you save `["Apple", "Banana", "Milk"]`, the database explicitly saves:
    * Apple ➡️ Position `0`
    * Banana ➡️ Position `1`
    * Milk ➡️ Position `2`
* When Hibernate fetches this data back, it uses `ORDER BY position_column` so that your Java List is reconstructed **exactly** in the same 1st, 2nd, 3rd sequence.



### 2. The Multiple Bag Fetch Exception Explained

When managing entity relationships in JPA/Hibernate, fetching multiple collections simultaneously can trigger a notorious runtime error.

#### 1. The Error Message

When you try to eagerly fetch an entity that contains two or more `List<T>` (bag) associations at the exact same time, Hibernate crashes with the following exception:

```text
org.hibernate.loader.MultipleBagFetchException: 
cannot simultaneously fetch multiple bags: 
[com.example.Order.items, com.example.Order.payments]
```

#### 2. What Triggers It

You will encounter this exception specifically when all of the following conditions are met:
1. An entity defines **two or more** `List<T>` collections without an `@OrderColumn` (meaning they are treated as Hibernate **Bags**).
2. You attempt to **eagerly load both collections** in a single database round-trip.

This can happen via a `JOIN FETCH` query, an `@EntityGraph`, or by marking both associations with `FetchType.EAGER`.

```java
// ❌ This triggers the MultipleBagFetchException!
@Query("SELECT o FROM Order o JOIN FETCH o.items JOIN FETCH o.payments")
List<Order> findAllWithItemsAndPayments();
```


#### 3. Deep Dive: Why Does Hibernate Block This? (The "More Explanation")

Hibernate is actually preventing you from accidentally destroying your application's performance. The root cause is a relational database concept called a **Cartesian Product (or Cartesian Explosion)**.

##### The Database Perspective
When you perform a SQL `JOIN` across two independent one-to-many relationships (e.g., an Order has N items and M payments), the database does not return two clean lists. Instead, it multiplies them together. The total number of rows returned by the query becomes:

Total Rows = Orders * N (Items) * M (Payments)

###### A Real-World Example
Imagine you have **1 Order**. 
* This order has **3 items** (Apple, Banana, Milk).
* This order has **2 payments** (Credit Card, PayPal).

Here is the core problem: **Items and Payments have absolutely no relationship to each other.** They are completely separate, independent entities that just happen to belong to the same parent Order.

If Hibernate ran the `JOIN FETCH` query, the database would return a flat matrix combining every item with every payment:

| Order ID | Item Name | Payment Method |
| :--- | :--- | :--- |
| 1 | Apple | Credit Card |
| 1 | Apple | PayPal |
| 1 | Banana | Credit Card |
| 1 | Banana | PayPal |
| 1 | Milk | Credit Card |
| 1 | Milk | PayPal |

Instead of 5 data elements (3 + 2), the database forces Hibernate to read **6 rows**. If an order had 50 items and 10 payments, it would return **500 rows** of heavily duplicated data.

##### The Bag Dilemma
Because a **Bag** allows duplicate elements and has no unique identity or tracking index, Hibernate looks at this flat table and cannot safely figure out which duplicates are real and which duplicates were caused by the SQL Join multiplier. To protect your application data from corruption and your memory from exhausting, Hibernate throws the `MultipleBagFetchException`.


### 4. How to Fix It

There are three industry-standard ways to resolve this issue:

#### Solution A: Use `Set<T>` instead of `List<T>` (Easiest)
If your collections do not need to allow duplicate elements, change the Java type from `List` to `Set`. Hibernate can easily filter out the SQL Cartesian duplicate rows because a `Set` enforces uniqueness.

```java
@OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
private Set<Item> items; // Changed to Set

@OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
private Set<Payment> payments; // Changed to Set
```

⚠️ **Crucial Catch: If you need to override `equals()` and `hashCode()` (whether for business purposes, sorting, or custom logic), then you must override them correctly!**
Because a Java `Set` relies on object equality to prevent duplicates, you must provide a safe custom implementation. *(Note: If you do not explicitly override them, Java safely falls back to its default memory reference check, which works fine within a single Hibernate session).*

* **The Golden Rule:** Do **NOT** use database-generated `@Id` columns (like an auto-increment column) inside your `equals()` and `hashCode()`. When you create a new entity, its ID is `null`. Once you save it, the database assigns an ID. Changing the ID changes the hashcode, which will completely break the `Set` in Java memory.
* **The Solution:** Use a **Business Key** (a unique, immutable property like a UUID, a SKU number, an email, or a transaction token). If no natural business key exists, use a programmatic UUID generated when the object is created in Java.


#### Solution B: Split the Fetch into Multiple Queries (Best for Performance)
Instead of forcing the database to multiply the records in one massive join, fetch the primary entity with one collection first, and then fetch the second collection using the initialized entity references.

```java
// Query 1: Fetch Orders and their Items
@Query("SELECT o FROM Order o JOIN FETCH o.items")
List<Order> findAllWithItems();

// Query 2: Fetch the same Orders and their Payments (Hibernate merges them in memory)
@Query("SELECT o FROM Order o JOIN FETCH o.payments WHERE o IN :orders")
List<Order> fetchPaymentsForOrders(@Param("orders") List<Order> orders);
```


#### Solution C: Keep `List<T>` and add `@OrderColumn`
If you want to keep using Java `List` collections, you can fix the issue by adding the `@OrderColumn` annotation to your relationship. This tells Hibernate to treat the collection as a true **Ordered List** instead of a **Bag**. Since Hibernate maintains a persistent tracking index for each element, it can easily distinguish between actual duplicates and the Cartesian product rows, bypassing the exception.

```java
@OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
@OrderColumn(name = "item_order_index") // Turns Bag into an Ordered List
private List<Item> items;

@OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
@OrderColumn(name = "payment_order_index") // Turns Bag into an Ordered List
private List<Payment> payments;
```
⚠️ **Trade-off:** Hibernate will now run extra SQL `UPDATE` statements to maintain the index column when items are inserted or removed from the middle of the list. For large lists, this introduces a significant performance cost.



#### Solution D: Use `@Fetch(FetchMode.SUBSELECT)` (Rarely Recommended)
By annotating your collections with `@Fetch(FetchMode.SUBSELECT)`, Hibernate changes its global fetching strategy for those fields. Instead of fetching collections inside the main query, it automatically runs exactly one subsequent subselect query for each collection when they are accessed, completely avoiding the Cartesian multiplication.

```java
@OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
@Fetch(FetchMode.SUBSELECT)
private List<Item> items;

@OneToMany(mappedBy = "order", fetch = FetchType.LAZY)
@Fetch(FetchMode.SUBSELECT)
private List<Payment> payments;
```

⚠️ **The Hidden Trap — Why this is rarely a good idea:**
* **Bans Manual `JOIN FETCH`:** It **does not fix** the `MultipleBagFetchException` if you keep using manual `JOIN FETCH` queries. If you try to write `SELECT o FROM Order o JOIN FETCH o.items JOIN FETCH o.payments`, Hibernate will completely ignore the `SUBSELECT` annotation and still crash your application.
* **Forces Reliance on Lazy Loading:** To make this work, you must remove `JOIN FETCH` from your repository layer entirely and rely strictly on Hibernate's automated lazy-loading triggers behind the scenes. This strips away your manual control and can introduce invisible query performance blind spots across your codebase.



---




## 2. The Circular Reference Problem 

### 1. What is a Circular Reference?

A **circular reference** happens when two (or more) objects point to each other, forming a loop.

#### In Code

```java
Order order = new Order();
OrderItem item = new OrderItem();

order.setItem(item);   // Order → Item
item.setOrder(order);  // Item → Order  ← CYCLE!
```

In JPA, this is **completely normal** — a parent-child bidirectional relationship *always* looks like this in memory. It's not a bug in your entities. The problem appears **when something tries to serialize, hash, or print this cycle.**


### 2. Why Does It Happen?

JPA relationships are **bidirectional** — the parent knows its children, and children know their parent. This is by design and is actually useful:

- **Parent → Child** navigation: `order.getItems()` — needed to list all items of an order.
- **Child → Parent** navigation: `item.getOrder()` — needed to know which order an item belongs to.

Both directions are legitimate. But three things do **not** understand cycles:

| Tool | What It Does | What Breaks |
|---|---|---|
| **Jackson** (JSON serialization) | Walks every field and serializes it | Infinite loop → `StackOverflowError` |
| **`toString()` / `equals()` / `hashCode()`** | Recursively walks fields | Infinite loop → `StackOverflowError` |
| **Lombok `@Data` / `@ToString` / `@EqualsAndHashCode`** | Auto-generates the above | Same crash, hidden behind generated code |


### 3. The Three Faces of the Problem

The "circular reference" problem is actually **three distinct problems** that look similar:

#### 🎭 Face 1: JSON Serialization Loop (Jackson)

When you return an entity from a REST controller, Spring uses **Jackson** to convert the object to JSON. Jackson doesn't know when to stop.

```json
{
  "id": 1,
  "customerName": "Alice",
  "items": [
    {
      "id": 10,
      "productName": "Book",
      "order": {
        "id": 1,
        "customerName": "Alice",
        "items": [
          {
            "id": 10,
            "productName": "Book",
            "order": {
              "id": 1,
              ...
              // ♻️ infinite loop → StackOverflowError
            }
          }
        ]
      }
    }
  ]
}
```

**Error you see:**
```
com.fasterxml.jackson.databind.JsonMappingException:
Infinite recursion (StackOverflowError)
```

#### 🎭 Face 2: `toString()` / `equals()` / `hashCode()` Loop

If you use Lombok's `@Data` or `@ToString`, or manually write these methods to include the back-reference, calling them explodes:

```java
Order order = repo.findById(1L).get();
System.out.println(order);  // 💥 StackOverflowError
```

Why? `Order.toString()` prints items → each `OrderItem.toString()` prints its order → that order prints its items → **loop**.

#### 🎭 Face 3: Lombok's Silent Trap

Lombok generates the dangerous methods **for you** without warning:

```java
@Data   // generates toString, equals, hashCode → ALL recursive
@Entity
public class Order {
    @OneToMany(mappedBy = "order")
    private List<OrderItem> items;
    // ...
}

@Data   // also recursive
@Entity
public class OrderItem {
    @ManyToOne
    private Order order;
    // ...
}
```

This is the **most common cause** in modern Spring Boot projects because `@Data` feels innocent.



### 4. Reproducing the Problem (Full Runnable Example)

#### 4.1 The Entities (with Lombok `@Data` — the trap)

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import lombok.Data;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "orders")
@Data   // ❌ generates recursive toString/equals/hashCode
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String customerName;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
}
```

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import lombok.Data;

@Entity
@Table(name = "order_items")
@Data   // ❌ also recursive
public class OrderItem {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String productName;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;   // ← back-reference
}
```

#### 4.2 The Controller (triggers JSON loop)

```java
package com.example.demo.controller;

import com.example.demo.entity.Order;
import com.example.demo.repository.OrderRepository;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/orders")
public class OrderController {

    private final OrderRepository repo;

    public OrderController(OrderRepository repo) {
        this.repo = repo;
    }

    @GetMapping
    public List<Order> getAll() {
        return repo.findAll();   // 💥 Jackson infinite loop here
    }
}
```

#### 4.3 Call It

```bash
curl http://localhost:8080/orders
```

**You get:**
```
com.fasterxml.jackson.databind.JsonMappingException:
Infinite recursion (StackOverflowError)
```

**And if you also log the entity:**
```java
Order o = repo.findById(1L).get();
System.out.println(o);   // 💥 Lombok toString → StackOverflowError
```

You have now reproduced **both Face 1 and Face 2** at once. 


### 5. Solutions

#### ✅ Solution A: `@JsonIgnore` on the Back-Reference (Simplest for JSON)

Tell Jackson to skip the child's reference back to the parent.

```java
package com.example.demo.entity;

import com.fasterxml.jackson.annotation.JsonIgnore;
import jakarta.persistence.*;
import lombok.Data;

@Entity
@Table(name = "order_items")
@Data
public class OrderItem {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String productName;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    @JsonIgnore   // ✅ Jackson skips this field
    private Order order;
}
```

**Result JSON:**
```json
{
  "id": 1,
  "customerName": "Alice",
  "items": [
    { "id": 10, "productName": "Book" },
    { "id": 11, "productName": "Pen" }
  ]
}
```

✅ Clean. No loop.
⚠️ **Trade-off:** You lose the ability to navigate from child to parent in your API responses. If a client needs to know `item.getOrder().getId()`, they can't.



#### ✅ Solution B: `@JsonManagedReference` + `@JsonBackReference` (Best for Parent-Child JSON)

Jackson's built-in **pair** annotation that explicitly tells it which side of the relationship is the "forward" (managed) and which is the "back" reference.

```java
package com.example.demo.entity;

import com.fasterxml.jackson.annotation.JsonManagedReference;
import jakarta.persistence.*;
import lombok.Data;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "orders")
@Data
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String customerName;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    @JsonManagedReference   // ✅ serialized forward
    private List<OrderItem> items = new ArrayList<>();
}
```

```java
package com.example.demo.entity;

import com.fasterxml.jackson.annotation.JsonBackReference;
import jakarta.persistence.*;
import lombok.Data;

@Entity
@Table(name = "order_items")
@Data
public class OrderItem {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String productName;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    @JsonBackReference   // ✅ skipped during serialization
    private Order order;
}
```

**Result:** Same JSON as Solution A, but the intent is more explicit. Jackson understands the pair, so it knows to write `items` but skip `order`.

⚠️ **Limitation:** `@JsonBackReference` fields are also **skipped during deserialization**. If you POST a JSON to create an `OrderItem` with an `order` field, Jackson **ignores it**. You must set the order in code or via a DTO.



#### ✅ Solution C: DTOs (The Cleanest — Best for Production)

Instead of returning entities directly, create **Data Transfer Objects** that only carry what the API needs. Entities never leave the service layer.

##### DTOs

```java
package com.example.demo.dto;

public record OrderItemDto(Long id, String productName) {}
```

```java
package com.example.demo.dto;

import java.util.List;

public record OrderDto(Long id, String customerName, List<OrderItemDto> items) {}
```

##### Mapper

```java
package com.example.demo.mapper;

import com.example.demo.dto.OrderDto;
import com.example.demo.dto.OrderItemDto;
import com.example.demo.entity.Order;
import org.springframework.stereotype.Component;
import java.util.stream.Collectors;

@Component
public class OrderMapper {

    public OrderDto toDto(Order o) {
        return new OrderDto(
                o.getId(),
                o.getCustomerName(),
                o.getItems().stream()
                        .map(i -> new OrderItemDto(i.getId(), i.getProductName()))
                        .collect(Collectors.toList())
        );
    }
}
```

##### Controller

```java
@RestController
@RequestMapping("/orders")
public class OrderController {

    private final OrderRepository repo;
    private final OrderMapper mapper;

    public OrderController(OrderRepository repo, OrderMapper mapper) {
        this.repo = repo;
        this.mapper = mapper;
    }

    @GetMapping
    public List<OrderDto> getAll() {
        return repo.findAll().stream()
                .map(mapper::toDto)
                .toList();
    }
}
```

✅ **No circular reference possible** — DTOs have no back-reference.
✅ **Full control over API shape.**
✅ **Decouples API from DB schema.**
⚠️ **Trade-off:** More boilerplate. But this is the **industry-standard** approach for non-trivial apps.


#### ✅ Solution D: Fix Lombok — `@ToString` / `@EqualsAndHashCode` Exclusions

If you want to keep using Lombok's `@Data`, tell it to **exclude the back-reference** from generated methods.

##### Option D1: Exclude fields from `@ToString`

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import lombok.Data;
import lombok.ToString;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "orders")
@Data
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String customerName;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    @ToString.Exclude   // ✅ Lombok skips this in toString → no recursion
    private List<OrderItem> items = new ArrayList<>();
}
```

```java
@Entity
@Table(name = "order_items")
@Data
public class OrderItem {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String productName;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    @ToString.Exclude   // ✅ Lombok skips this too
    @EqualsAndHashCode.Exclude  // ✅ and skips it in equals/hashCode
    private Order order;
}
```

##### Option D2: Replace `@Data` with safer annotations

`@Data` bundles `@ToString`, `@EqualsAndHashCode`, `@Getter`, `@Setter`, `@RequiredArgsConstructor`. A safer combo for JPA entities:

```java
@Entity
@Getter
@Setter
@ToString(onlyExplicitlyIncluded = true)
@EqualsAndHashCode(onlyExplicitlyIncluded = true)
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @ToString.Include
    @EqualsAndHashCode.Include
    private Long id;

    @ToString.Include
    private String customerName;

    @OneToMany(mappedBy = "order")
    private List<OrderItem> items = new ArrayList<>();
}
```

> **Why this is safer:** `onlyExplicitlyIncluded = true` means Lombok **only** uses fields you explicitly mark. Nothing is included by accident.

⚠️ **Still not safe for JSON** — this fixes Lombok, but Jackson still loops. Combine with Solution A, B, or C.



#### ✅ Solution E: `@JsonIdentityInfo` (Advanced — Handles Duplicates Too)

This Jackson annotation gives each object an **ID**, so when Jackson encounters the same object twice, it emits a reference instead of re-serializing.

```java
package com.example.demo.entity;

import com.fasterxml.jackson.annotation.JsonIdentityInfo;
import com.fasterxml.jackson.annotation.ObjectIdGenerators;
import jakarta.persistence.*;
import lombok.Data;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "orders")
@Data
@JsonIdentityInfo(
        generator = ObjectIdGenerators.PropertyGenerator.class,
        property = "id"
)
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String customerName;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
}
```

```java
@Entity
@Table(name = "order_items")
@Data
@JsonIdentityInfo(
        generator = ObjectIdGenerators.PropertyGenerator.class,
        property = "id"
)
public class OrderItem {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String productName;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;
}
```

**Result JSON:**
```json
{
  "id": 1,
  "customerName": "Alice",
  "items": [
    { "id": 10, "productName": "Book", "order": 1 },
    { "id": 11, "productName": "Pen",  "order": 1 }
  ]
}
```

Note: `"order": 1` — it's just the **ID**, not a nested object. No loop.

⚠️ **Trade-off:** Changes the JSON shape. Consumers must understand that some fields are IDs, not objects. Can be confusing on the client side.


### 6. Common Mistakes & Gotchas

#### ❌ Mistake 1: Using `@Data` on JPA entities

`@Data` looks harmless. It isn't. It generates `toString`, `equals`, and `hashCode` that recurse into **every field**, including lazy relationships. This alone causes 80% of circular reference bugs in Spring Boot projects.

✅ Use `@Getter` + `@Setter`, and be deliberate about `toString`/`equals`/`hashCode`.

#### ❌ Mistake 2: Using `@JsonIgnore` on the **parent** side by accident

```java
@OneToMany(mappedBy = "order")
@JsonIgnore   // ❌ you just hid all items from JSON
private List<OrderItem> items;
```

✅ Put `@JsonIgnore` on the **back-reference** (the `@ManyToOne` side), not the collection side.

#### ❌ Mistake 3: Forgetting that `@JsonBackReference` blocks deserialization

If you POST JSON with an `order` field on `OrderItem`, Jackson **silently drops it**. You must set the parent in code:

```java
OrderItem item = new OrderItem();
item.setProductName("Book");
item.setOrder(existingOrder);   // ← you must do this manually
```

#### ❌ Mistake 4: Mixing `@JsonIgnore` and `@JsonBackReference` on the same field

These do different things. Combining them can cause inconsistent behavior. Pick one.

#### ❌ Mistake 5: Thinking "just use `@JsonIgnoreProperties({"hibernateLazyInitializer", "handler"})`" fixes it

That annotation only hides Hibernate's proxy internals. It does **not** stop the parent-child loop. Different problem, different fix.


### 7. Quick Cheat Sheet

```
┌──────────────────────────────────────────────────────────────┐
│  PROBLEM: Circular Reference                                 │
│  SYMPTOM: StackOverflowError / Infinite recursion            │
└──────────────────────────────────────────────────────────────┘

THREE FACES:
  1. Jackson JSON serialization loop   → @JsonIgnore / @JsonManaged+BackReference / DTO
  2. Lombok-generated toString loop    → @ToString.Exclude
  3. equals/hashCode recursion         → @EqualsAndHashCode.Exclude

FIXES (pick one per face):

  A. @JsonIgnore on back-ref           (simplest, hides child→parent)
  B. @JsonManagedReference +           (explicit pair, hides back-ref)
     @JsonBackReference
  C. DTOs                              (cleanest, production standard)
  D. @ToString.Exclude +               (fixes Lombok-generated methods)
     @EqualsAndHashCode.Exclude
  E. @JsonIdentityInfo                 (emits ID on second encounter)

GOLDEN RULES:
 • NEVER use @Data on a JPA entity
 • NEVER return entities directly from a controller
 • Put @JsonIgnore / @JsonBackReference on the @ManyToOne side
 • Prefer DTOs for anything non-trivial
 • Keep FetchType.LAZY (EAGER makes the loop worse)
 • Don't mix @JsonIgnore and @JsonBackReference on same field
```


---



## 3. The Self-Invocation Problem in Spring Boot (@Transactional Inside the Same Class) 

### 1. What is the Self-Invocation Problem?

Imagine you have a service class with two methods:

```java
@Service
public class OrderService {

    public void placeOrder(Order order) {
        // ... some logic ...
        saveOrder(order);   // ← calling a @Transactional method inside the same class
    }

    @Transactional
    public void saveOrder(Order order) {
        // ... DB save ...
    }
}
```

**Question:** When `placeOrder()` calls `saveOrder()`, does the `@Transactional` annotation on `saveOrder()` actually work?

**Answer:** ❌ **NO.**

This is called the **self-invocation problem** — and it silently breaks `@Transactional`, `@Async`, `@Cacheable`, and `@PreAuthorize`. No error, no warning. The annotation is simply **ignored**.


### 2. Why Does It Happen? (The Proxy Explained)

Spring implements `@Transactional` using **AOP proxies**. When you inject `OrderService`, Spring doesn't give you the real object — it gives you a **proxy wrapper**.

```
┌─────────────────────────────────────────────┐
│  Outside caller (Controller)                │
│         ↓                                   │
│  ┌──────────────────────────────────────┐   │
│  │  PROXY (Spring-generated)            │   │
│  │  ┌────────────────────────────────┐  │   │
│  │  │ 1. Start transaction           │  │   │
│  │  │ 2. Call real method            │  │   │
│  │  │ 3. Commit / Rollback          │  │   │
│  │  └────────────────────────────────┘  │   │
│  └──────────────────────────────────────┘   │
│         ↓                                   │
│  ┌──────────────────────────────────────┐   │
│  │  REAL OrderService instance          │   │
│  │                                      │   │
│  │  placeOrder() {                      │   │
│  │    saveOrder(); // ← `this.saveOrder`│   │
│  │  }                                   │   │
│  │                                      │   │
│  │  saveOrder() { /* no proxy hit */ }  │   │
│  └──────────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

#### The Key Insight

When `placeOrder()` calls `saveOrder()` internally, it uses **`this.saveOrder()`** — a direct call on the **real object**, not through the proxy. The proxy never sees it. So the transaction advice never fires.

> 💡 **One-liner:** `@Transactional` only works when the method is called **from outside the bean**, through the Spring proxy.

#### Proof: Two Scenarios

| Caller | Path | Transaction Works? |
|---|---|---|
| `OrderController` calls `orderService.saveOrder()` | Controller → Proxy → Real → `saveOrder` | ✅ Yes |
| `OrderService.placeOrder()` calls `this.saveOrder()` | Real → Real → `saveOrder` (no proxy) | ❌ No |


### 3. Reproducing the Problem (Full Runnable Example)

#### 3.1 The Entities

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

@Entity
@Table(name = "orders")
@Getter
@Setter
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String customerName;
    private double amount;
}
```

#### 3.2 The Repository

```java
package com.example.demo.repository;

import com.example.demo.entity.Order;
import org.springframework.data.jpa.repository.JpaRepository;

public interface OrderRepository extends JpaRepository<Order, Long> {
}
```

#### 3.3 The Broken Service

```java
package com.example.demo.service;

import com.example.demo.entity.Order;
import com.example.demo.repository.OrderRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class OrderService {

    private final OrderRepository repo;

    public OrderService(OrderRepository repo) {
        this.repo = repo;
    }

    // ❌ This method is NOT @Transactional.
    // It calls saveOrder() internally — the transaction on saveOrder is IGNORED.
    public void placeOrder(Order order) {
        System.out.println("placeOrder() started");

        // If saveOrder() throws, the order is still saved.
        saveOrder(order);

        // Now something fails AFTER saveOrder succeeded
        throw new RuntimeException("Simulated failure after save");
    }

    @Transactional
    public void saveOrder(Order order) {
        System.out.println("saveOrder() called — is a transaction active? "
                + org.springframework.transaction.support.TransactionSynchronizationManager
                    .isActualTransactionActive());
        repo.save(order);
    }
}
```

#### 3.4 The Controller

```java
package com.example.demo.controller;

import com.example.demo.entity.Order;
import com.example.demo.service.OrderService;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/orders")
public class OrderController {

    private final OrderService orderService;

    public OrderController(OrderService orderService) {
        this.orderService = orderService;
    }

    @PostMapping
    public String place(@RequestBody Order order) {
        try {
            orderService.placeOrder(order);
        } catch (RuntimeException e) {
            return "Failed: " + e.getMessage();
        }
        return "OK";
    }
}
```

#### 3.5 The Bug in Action

Call it:

```bash
curl -X POST http://localhost:8080/orders \
  -H "Content-Type: application/json" \
  -d '{"customerName": "Alice", "amount": 100}'
```

**Expected behavior:** The exception in `placeOrder()` should roll back `saveOrder()` → no order in DB.

**Actual behavior:**
- Console shows: `saveOrder() called — is a transaction active? false`
- The order **IS saved** to the DB despite the exception.
- Response: `"Failed: Simulated failure after save"`

**But the DB has the order.** Data integrity broken. ❌

You have now reproduced the self-invocation problem.


### 4. Solutions

#### ✅ Solution A: Move `saveOrder()` to a Separate Bean (Best)

The cleanest fix. Extract the `@Transactional` method into its own service.

```java
package com.example.demo.service;

import com.example.demo.entity.Order;
import com.example.demo.repository.OrderRepository;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class OrderPersistenceService {

    private final OrderRepository repo;

    public OrderPersistenceService(OrderRepository repo) {
        this.repo = repo;
    }

    @Transactional
    public void saveOrder(Order order) {
        repo.save(order);
    }
}
```

```java
package com.example.demo.service;

import com.example.demo.entity.Order;
import org.springframework.stereotype.Service;

@Service
public class OrderService {

    private final OrderPersistenceService persistenceService;

    public OrderService(OrderPersistenceService persistenceService) {
        this.persistenceService = persistenceService;
    }

    public void placeOrder(Order order) {
        // ✅ This call goes through the proxy → @Transactional fires!
        persistenceService.saveOrder(order);

        throw new RuntimeException("Simulated failure after save");
    }
}
```

**Result:** The order is **rolled back** when the exception is thrown. ✅

✅ Simple, clean, no magic.
✅ Recommended by the Spring team.
⚠️ Trade-off: One extra class.



#### ✅ Solution B: Inject the Class Into Itself (`@Lazy` Self-Injection)

The trick you asked about. Inject the **proxy** into the class itself and call the method via the injected reference instead of `this`.

```java
package com.example.demo.service;

import com.example.demo.entity.Order;
import com.example.demo.repository.OrderRepository;
import org.springframework.context.annotation.Lazy;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
public class OrderService {

    private final OrderRepository repo;
    private final OrderService self;  // ← inject self

    public OrderService(OrderRepository repo,
                        @Lazy OrderService self) {  // ← @Lazy breaks the circular dependency
        this.repo = repo;
        this.self = self;
    }

    public void placeOrder(Order order) {
        // ✅ Call via the proxy, not `this`
        self.saveOrder(order);

        throw new RuntimeException("Simulated failure after save");
    }

    @Transactional
    public void saveOrder(Order order) {
        repo.save(order);
    }
}
```

**Why `@Lazy` is mandatory:**
Without it, Spring tries to construct `OrderService`, which needs an `OrderService` to construct — infinite loop, and you get:

```
BeanCurrentlyInCreationException:
Requested bean is currently in creation: Is there an unresolvable circular reference?
```

`@Lazy` tells Spring: *"Don't resolve this dependency until it's actually needed."* So Spring injects a **proxy** that will be resolved later.

✅ No extra classes needed.
✅ Works with `@Transactional`, `@Async`, `@Cacheable`, etc.
⚠️ Trade-off: Slightly confusing — some teams consider self-injection a code smell.
⚠️ The `@Lazy` is easy to forget, and the error message is cryptic.


### 5. Common Mistakes & Gotchas

#### ❌ Mistake 1: Forgetting `@Lazy` on self-injection

```java
public OrderService(OrderRepository repo, OrderService self) { ... }  // ❌
```

You get `BeanCurrentlyInCreationException`. The `@Lazy` is not optional.

#### ❌ Mistake 2: Thinking `private` or `package-private` methods work with `@Transactional`

```java
@Transactional
private void saveOrder(Order order) { ... }   // ❌ silently ignored
```

Spring AOP **only proxies public methods** by default. Private/protected/package-private methods with `@Transactional` are **silently ignored**.

✅ Always use `public` for `@Transactional` methods.

#### ❌ Mistake 3: Self-injection with constructor + no `@Lazy` and using field injection as "workaround"

```java
@Autowired
private OrderService self;  // ❌ no @Lazy — same circular issue
```

You still need `@Lazy`, regardless of injection style.


#### ❌ Mistake 4: Using self-injection across `@Async` and expecting a new thread

Self-injection **does** work with `@Async` — but only if the method is public, the bean is proxied, and `@EnableAsync` is enabled. Otherwise same silent failure.


### ❌ Mistake 5: Using `final` on a self-injected field with `@Lazy` + constructor injection

Actually this is fine. But make sure you don't mark `OrderService` itself as `final class` — Spring needs to subclass it for the CGLIB proxy (unless the class implements an interface). If your class is `final`, `@Transactional` won't work at all.


### 6. Quick Cheat Sheet

```
┌──────────────────────────────────────────────────────────────┐
│  PROBLEM: Self-Invocation                                    │
│  SYMPTOM: @Transactional / @Async / @Cacheable silently     │
│           does nothing when called via `this.method()`       │
└──────────────────────────────────────────────────────────────┘

WHY?
  → Spring AOP uses a proxy.
  → External calls go through the proxy → advice fires.
  → `this.method()` bypasses the proxy → advice never fires.

FIXES (pick one):

  A. Extract to a separate bean      (cleanest — RECOMMENDED)
  B. Self-inject with @Lazy          (no new class, easy)
  C. AopContext.currentProxy()       (needs exposeProxy = true)
  D. Merge the methods               (if truly same transaction)
  E. TransactionTemplate             (programmatic, explicit)

GOLDEN RULES:
 • @Transactional only works on PUBLIC methods
 • @Transactional only works when called from OUTSIDE the bean
 • Self-injection requires @Lazy — always
 • AopContext requires @EnableAspectJAutoProxy(exposeProxy = true)
 • Prefer extracting to a separate bean over self-injection
 • Propagate REQUIRES_NEW if you need a fresh transaction after extraction
```


---


## 4. The N+1 Query Problem

### 1. What is the N+1 Problem?

The **N+1 problem** is a performance anti-pattern where your application fires **1 query to fetch a list of N entities**, and then **N additional queries** — one for each entity — to fetch a related collection or association. Total queries = **N + 1**.

It's called "N+1" because:
- **1** query fetches the parent list.
- **N** queries fetch the children for each parent.

### In Code

```java
// 1 query — fetch all orders
List<Order> orders = orderRepository.findAll();

// N queries — one per order to load items
for (Order order : orders) {
    order.getItems().size();  // 💥 triggers a SELECT per order
}
```

If there are 100 orders → **101 SQL queries fired**. The application works — it's just silently 100× slower.


### 2. Why Does It Happen?

JPA associations are **lazy by default** for collections (`@OneToMany`, `@ManyToMany`) and `@ManyToOne`. This is intentional — loading the entire object graph eagerly would be worse.

But lazy loading has a hidden cost: **the first time you touch a lazy collection, Hibernate fires a query.**

```java
@OneToMany(mappedBy = "order", fetch = FetchType.LAZY)  // ← default
private List<OrderItem> items;
```

When you loop over orders and access `order.getItems()`:

- Hibernate sees the collection is not initialized.
- Hibernate fires `SELECT * FROM order_items WHERE order_id = ?`.
- It does this **once per order** because each order has a different ID.

No error. No warning. Just slow.

### The Three Things That Make It Worse

| Situation | Why It Gets Worse |
|---|---|
| **1000 orders in a loop** | 1001 queries instead of 2 |
| **Multiple lazy collections per entity** | N × M queries (one per collection per entity) |
| **`Open Session in View` enabled** | Hides the problem in dev — only explodes in prod |



### 3. The Three Faces of N+1


#### 🎭 Face 1: Classic Loop (Most Common)

```java
List<Order> orders = repo.findAll();  // 1 query
for (Order o : orders) {
    System.out.println(o.getItems().size());  // N queries
}
```

**Symptom:** Log shows `SELECT * FROM orders` followed by **many identical** `SELECT * FROM order_items WHERE order_id = ?` with different IDs.

#### 🎭 Face 2: Jackson Serialization

```java
@GetMapping("/orders")
public List<Order> getAll() {
    return repo.findAll();   // 💥 Jackson triggers lazy loads during serialization
}
```

**Symptom:** The controller looks innocent — one call to `findAll()`. But the logs show **hundreds of SELECTs** because Jackson walks every field and touches every lazy collection.

This is the **most common N+1 in production** because developers don't see it happen — the JSON output looks fine, just slow.

#### 🎭 Face 3: Nested Associations

```java
@ManyToOne(fetch = FetchType.LAZY)
private Customer customer;   // lazy parent
```

Looping over 100 orders and calling `order.getCustomer().getName()` fires **100 queries** to the `customers` table — even if all 100 orders belong to the same customer.

**Symptom:** Duplicate SELECTs on the same parent ID repeated N times.



### 4. Reproducing the Problem (Full Runnable Example)

#### 4.1 The Entities

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "orders")
@Getter
@Setter
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String customerName;

    // Default: LAZY — this is where N+1 originates
    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    private List<OrderItem> items = new ArrayList<>();
}
```

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

@Entity
@Table(name = "order_items")
@Getter
@Setter
public class OrderItem {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String productName;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "order_id")
    private Order order;
}
```

#### 4.2 The Repository

```java
package com.example.demo.repository;

import com.example.demo.entity.Order;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import java.util.List;

public interface OrderRepository extends JpaRepository<Order, Long> {

    // ❌ No JOIN FETCH — N+1 will happen if items are accessed
    List<Order> findAll();
}
```

#### 4.3 The Controller — Triggers N+1

```java
package com.example.demo.controller;

import com.example.demo.entity.Order;
import com.example.demo.repository.OrderRepository;
import org.springframework.web.bind.annotation.*;
import java.util.List;

@RestController
@RequestMapping("/orders")
public class OrderController {

    private final OrderRepository repo;

    public OrderController(OrderRepository repo) {
        this.repo = repo;
    }

    @GetMapping
    public List<Order> getAll() {
        return repo.findAll();   // 💥 Jackson will trigger N lazy loads
    }
}
```

#### 4.4 Enable SQL Logging

```yaml
# application.yml
spring:
  jpa:
    show-sql: true
    properties:
      hibernate:
        format_sql: true
```

#### 4.5 The Bug in Action

Insert 5 orders, each with 3 items. Then:

```bash
curl http://localhost:8080/orders
```

**Console output:**

```sql
-- Query 1: fetch all orders
SELECT * FROM orders;

-- Query 2: items for order 1
SELECT * FROM order_items WHERE order_id = 1;

-- Query 3: items for order 2
SELECT * FROM order_items WHERE order_id = 2;

-- Query 4: items for order 3
SELECT * FROM order_items WHERE order_id = 3;

-- Query 5: items for order 4
SELECT * FROM order_items WHERE order_id = 4;

-- Query 6: items for order 5
SELECT * FROM order_items WHERE order_id = 5;
```

**Total: 6 queries for 5 orders.** With 1000 orders, that's **1001 queries**. You have now reproduced the N+1 problem.



### 5. Solutions

#### ✅ Solution A: `JOIN FETCH` (Most Common Fix)

Tell Hibernate to load the parent and the collection in **one SQL query**.

```java
package com.example.demo.repository;

import com.example.demo.entity.Order;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import java.util.List;

public interface OrderRepository extends JpaRepository<Order, Long> {

    // ✅ One query loads orders + items together
    @Query("SELECT DISTINCT o FROM Order o JOIN FETCH o.items")
    List<Order> findAllWithItems();
}
```

**Result:** 1 query instead of N+1.

```sql
SELECT o.*, i.*
FROM orders o
INNER JOIN order_items i ON i.order_id = o.id;
```

**Why `DISTINCT`:** Without it, if an order has 3 items, it appears 3 times in the result. `DISTINCT` tells Hibernate to collapse the parent rows (Hibernate 6 handles this automatically, but `DISTINCT` is safe).

⚠️ **Warning:** If the entity has **two or more `List<>` (bag) collections**, `JOIN FETCH` on both throws `MultipleBagFetchException` (see Section 1).

⚠️ **Warning:** `JOIN FETCH` with pagination (`Pageable`) causes Hibernate to load **all rows into memory** and paginate in Java. Use a two-query approach instead.



#### ✅ Solution B: `@BatchSize` / `default_batch_fetch_size` (Best When You Can't JOIN)

Instead of firing one query per parent, fetch children in **batches** using an `IN` clause.

```java
package com.example.demo.entity;

import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;
import org.hibernate.annotations.BatchSize;
import java.util.ArrayList;
import java.util.List;

@Entity
@Table(name = "orders")
@Getter
@Setter
public class Order {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String customerName;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    @BatchSize(size = 20)   // ✅ Fetch items for up to 20 orders per query
    private List<OrderItem> items = new ArrayList<>();
}
```

Or **globally** in `application.yml`:

```yaml
spring:
  jpa:
    properties:
      hibernate:
        default_batch_fetch_size: 20
```

**Result for 100 orders:**

- Without `@BatchSize`: **100 queries** (one per order).
- With `@BatchSize(20)`: **5 queries** (100 / 20).

```sql
SELECT * FROM order_items WHERE order_id IN (1, 2, 3, ..., 20);
SELECT * FROM order_items WHERE order_id IN (21, 22, ..., 40);
-- ... 5 queries total
```

✅ Works with `LAZY` — no JOIN FETCH needed.
✅ Solves the `MultipleBagFetchException` case (you can't JOIN FETCH two bags, but you can `@BatchSize` both).
✅ No pagination issues.
⚠️ Still more than 1 query — but far fewer than N+1.

**This is the safest default for most production apps.**


#### ✅ Solution C: DTO Projection (Best for Read-Only Endpoints)

Skip entities entirely. Write a query that returns exactly what the API needs.

```java
package com.example.demo.dto;

import java.util.List;

public record OrderDto(Long id, String customerName, List<String> itemNames) {}
```

```java
package com.example.demo.repository;

import com.example.demo.dto.OrderDto;
import com.example.demo.entity.Order;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import java.util.List;

public interface OrderRepository extends JpaRepository<Order, Long> {

    @Query("""
        SELECT new com.example.demo.dto.OrderDto(
            o.id, o.customerName,
            (SELECT i.productName FROM OrderItem i WHERE i.order = o)
        )
        FROM Order o
    """)
    List<OrderDto> findAllAsDto();
}
```

✅ **Exactly one query.** No lazy loads. No proxies. No surprises.
✅ Decouples API shape from DB schema.
⚠️ More boilerplate.
⚠️ Only works for read paths.



### 5. Common Mistakes & Gotchas

#### ❌ Mistake 1: `Open Session in View` hides N+1 in dev

Spring Boot enables `spring.jpa.open-in-view=true` **by default**. This keeps the Hibernate session open during JSON serialization — so Jackson's lazy loads **work** in dev. Then in prod with real data, they become 1000 queries.

✅ Set `spring.jpa.open-in-view=false` and fix N+1 properly.

#### ❌ Mistake 2: `JOIN FETCH` on two bags

```java
@Query("SELECT o FROM Order o JOIN FETCH o.items JOIN FETCH o.payments")
List<Order> findAll();   // ❌ MultipleBagFetchException
```

✅ Use `Set` for both, or use `@BatchSize` instead.

#### ❌ Mistake 3: Adding `DISTINCT` and assuming it's free

`DISTINCT` on the SQL side is costly. In JPQL, Hibernate adds `DISTINCT` to the SQL — but Hibernate 6 auto-deduplicates without needing it in most cases. Test both.

#### ❌ Mistake 4: Using `FetchType.EAGER` to "fix" N+1

Eager loading doesn't fix N+1 — it just moves it. Now **every** `findAll()` loads the whole graph, including cases where you didn't need it.

✅ Keep `LAZY`, use `JOIN FETCH` or `@EntityGraph` selectively.

#### ❌ Mistake 5: N+1 via `@ManyToOne` back-references

```java
@ManyToOne(fetch = FetchType.LAZY)
private Customer customer;
```

Looping and calling `order.getCustomer().getName()` fires **one query per order**, even if all orders share the same customer.

✅ Use `JOIN FETCH o.customer` in the query.

#### ❌ Mistake 6: Forgetting that DTO projections skip lazy loading entirely

If you write a JPQL query that selects DTOs, Hibernate doesn't instantiate entities — no lazy loading, no N+1. This is why DTOs are the safest read-path approach.

#### ❌ Mistake 7: Logging `order.toString()` triggers N+1

If `toString()` includes lazy collections (and Lombok `@Data` does), **logging** your entity triggers N+1.

```java
log.info("Order loaded: {}", order);   // 💥 triggers lazy loads
```

✅ Use `@ToString.Exclude` on collections, or log only IDs.


#### ❌ Mistake 8: Using `@BatchSize` on `@ManyToOne`

`@BatchSize` works on collections, not on `@ManyToOne`. For lazy `@ManyToOne`, use `@BatchSize` at the class level or set `hibernate.default_batch_fetch_size` globally.



### 6. Quick Cheat Sheet

```
┌──────────────────────────────────────────────────────────────┐
│  PROBLEM: N+1 Query                                          │
│  SYMPTOM: 1 parent query + N child queries, silent slowdown │
└──────────────────────────────────────────────────────────────┘

WHY?
  → Collections are LAZY by default.
  → First access to each lazy collection fires a query.
  → Looping over N parents → N queries.

THREE FACES:
  1. Classic loop              → order.getItems() inside for-loop
  2. Jackson serialization     → controller returns entity, Jackson walks it
  3. Nested @ManyToOne         → order.getCustomer().getName() in loop

FIXES (pick one):

  A. JOIN FETCH                (1 query, no pagination)
  B. @EntityGraph              (declarative, works with Pageable)
  C. @BatchSize /              (fewest changes, safest default)
     default_batch_fetch_size
  D. DTO projection            (best for read endpoints)
  E. Two-query fetch           (JOIN FETCH + correct pagination)

GOLDEN RULES:
 • Set spring.jpa.open-in-view=false in production
 • Enable default_batch_fetch_size: 20 globally as a safety net
 • Use DTOs for list/read endpoints
 • Never JOIN FETCH + Pageable (loads everything in memory)
 • Never JOIN FETCH two bags (MultipleBagFetchException)
 • Keep LAZY, fetch selectively
 • Log only IDs, never whole entities with lazy collections
 • Watch show-sql in dev — repeated SELECTs = N+1
```































