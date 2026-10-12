### 1. Explain Collection Hierarchy ?

It starts with the collection Interface. Collection Interface is the root of Java Collection Framework and most of the collections in javaare inherited from  this interface except Map interface.

```txt
                         java.lang.Iterable
                                |
                                v
                       java.util.Collection   (root interface)
                                |
        +-----------------------+-----------------------+
        |                       |                       |
        v                       v                       v
      List                    Set                    Queue
    (Interface)           (Interface)             (Interface)
        |                       |                       |
        |                       |                       +--> Deque (Interface)
        |                       |                       |        |
        |             +---------+---------+             |        +--> ArrayDeque
        |             |         |         |             |        +--> LinkedList
        |             v         v         v             v
        |         HashSet  TreeSet  LinkedHashSet   PriorityQueue
        |             |
        |             v
        |      LinkedHashSet
        |
        +--> ArrayList
        +--> LinkedList
        +--> Vector
        |      |
        |      v
        |    Stack
        +--> CopyOnWriteArrayList


                    java.util.Map   (separate root — does NOT extend Collection)
                         |
        +----------------+----------------+----------------+
        |                |                |                |
        v                v                v                v
     HashMap         TreeMap        LinkedHashMap     Hashtable
        |                                                 |
        v                                                 v
  (also: WeakHashMap, IdentityHashMap,             Properties
   EnumMap, ConcurrentHashMap)
```

---


### 2. Explain the List hierarchy.

```txt
java.lang.Iterable
   |
   v
java.util.Collection
   |
   v
java.util.List (Interface)
   |
   +-- java.util.ArrayList (Concrete Class)
   |
   +-- java.util.LinkedList (Concrete Class)
   |        |
   |        +-- java.util.Deque (Interface)
   |
   +-- java.util.Vector (Concrete Class — legacy, synchronized)
   |      |
   |      v
   |   java.util.Stack (Concrete Class — legacy, LIFO)
   |
   +-- java.util.concurrent.CopyOnWriteArrayList
              (Concrete Class — thread-safe)
```

#### Characteristics

- `List` is an **ordered collection** that maintains insertion order.
- Allows **duplicate** elements and **null** elements (depending on the implementation).
- Supports **positional (index-based) access and modification** using `get(int)`, `set(int, E)`, `add(int, E)`, and `remove(int)`. Elements can be retrieved, inserted, removed, or replaced at a specific index. When elements are inserted or removed, the positions of other elements may shift to maintain the order.
- Supports **index-based search** using `indexOf(Object)` and `lastIndexOf(Object)`, which return the index of the first and last occurrence of an element, respectively.
- Supports **random access** in implementations like `ArrayList`, allowing elements to be retrieved directly by index without traversing the list one element at a time.
- Provides a special iterator called `ListIterator` for **bidirectional traversal**, allowing movement both forward and backward through the list, as well as element modification during iteration.

```java
import java.util.ArrayList;
import java.util.List;
import java.util.ListIterator;

public class ListCharacteristicsExample {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("A");
        list.add("B");
        list.add("C");
        list.add("B");   // duplicates allowed
        list.add(null);  // null allowed

        // Positional access
        System.out.println(list.get(0));           // A

        // Index-based search
        System.out.println(list.indexOf("B"));     // 1
        System.out.println(list.lastIndexOf("B")); // 3

        // Insert / remove / replace anywhere
        list.add(1, "X");        // insert at index 1  -> [A, X, B, C, B, null]
        list.set(2, "Y");        // replace index 2    -> [A, X, Y, C, B, null]
        list.remove(3);          // remove index 3     -> [A, X, Y, B, null]

        // ListIterator: bidirectional traversal
        ListIterator<String> it = list.listIterator();
        while (it.hasNext()) {
            System.out.println("Forward: " + it.next());
        }
        while (it.hasPrevious()) {
            System.out.println("Backward: " + it.previous());
        }
    }
}
```


##### 1. `ArrayList`

**Internal Structure:** Resizable array

```text
Index:   0     1     2     3     4
       +-----+-----+-----+-----+-----+
       | A   | B   | C   | D   | E   |
       +-----+-----+-----+-----+-----+
```

##### Characteristics:

- **Constant-time positional access** — `get(int)` is **O(1)**, because the internal array allows direct index lookup. This means you can jump straight to any index in the list instantly, without scanning through the other elements.
- **Fastest for random access and iteration** among all `List` implementations.
- **Maintains insertion order** and **allows null and duplicate** elements.
- **Default initial capacity: 10**; when the internal array fills up, it **grows by 50%** (i.e., new capacity = old capacity + old capacity / 2). (for example, from 10 to 15, then 15 to 22, and so on).
- **Not synchronized (not thread-safe)** — if multiple parts of a program try to modify the list at the same time, things could get messy (race conditions, corrupted data, or unexpected results). Use `Collections.synchronizedList()` or `CopyOnWriteArrayList` if concurrent access is needed.

**When to use:** General-purpose list where random access is frequent.

```java
List<String> list = new ArrayList<>();
list.add("A");
list.add("B");
list.add("B");             // duplicate allowed
list.add(null);            // null allowed

list.get(1);               // O(1) -> B
list.indexOf("B");         // 1
list.lastIndexOf("B");     // 2
list.add(1, "X");          // insert at index 1
list.remove(2);            // remove index 2
System.out.println(list);  // [A, X, B, null]
```


##### 2. `LinkedList`

**Internal Structure:** Doubly-linked list of nodes

```text
       +------+     +------+     +------+
null <-| prev |<--->| prev |<--->| prev |-> null
       | data |     | data |     | data |
       | next |<--->| next |<--->| next |
       +------+     +------+     +------+
         Node A      Node B       Node C
```

##### Characteristics:

- **Linear-time positional access** — `get(int)` is **O(n)**. This means you cannot jump straight to an index; the list has to start from the beginning (or end) and walk through the nodes one by one until it reaches the position you asked for.
- **Constant-time add/remove at the beginning or end** — adding or removing at the head or tail is **O(1)** because the list only needs to update a couple of links.
- **Keeps insertion order** — elements stay in the order you added them.
- **Allows null and duplicate** elements.
- **Higher memory overhead per element** — each element is wrapped inside a `Node` object that stores the data plus two references (previous and next). So each element uses more memory than an `ArrayList`.
- **Also implements `Deque` and `Queue`** — so it can be used as a double-ended queue, a stack, or a FIFO queue.
- **Not synchronized (not thread-safe)** — if multiple threads try to modify the list at the same time, things could get messy. You might get corrupted links or unexpected results. Use `Collections.synchronizedList()` if you need safe concurrent access.

**When to use:** Frequent insertion/deletion at the ends; stack or queue usage.

```java
LinkedList<String> list = new LinkedList<>();
list.add("B");
list.add("C");
list.addFirst("A");        // O(1)
list.addLast("D");         // O(1)

list.get(2);               // O(n) -> C
list.addFirst("Z");        // O(1)
list.removeLast();         // O(1)
System.out.println(list);  // [Z, A, B, C]
```


##### 3. `Vector` (Legacy)

**Internal Structure:** Synchronized resizable array

```text
Index:   0     1     2     3     4
       +-----+-----+-----+-----+-----+
       | A   | B   | C   | D   | E   |   (all methods synchronized)
       +-----+-----+-----+-----+-----+
```

##### Characteristics:

- **Same structure as `ArrayList`** — backed by a resizable array — but **every method is synchronized**.
- **Thread-safe**, which means multiple threads can safely use it at the same time without corrupting the data. However, it is **slower than `ArrayList`** because of the locking overhead on every operation.
- **Default initial capacity is 10**.
- **Grows by 100% (doubles)** when the internal array becomes full. For example, from 10 to 20, then 20 to 40, and so on.
- **Legacy class** — it existed before the modern Collection Framework was introduced. It has largely been replaced by `ArrayList` (for single-threaded use) or `Collections.synchronizedList()` (for thread-safe use).

**When to use:** Rarely — only in legacy code. Prefer `ArrayList` or `CopyOnWriteArrayList`.

```java
Vector<String> vector = new Vector<>();
vector.add("A");
vector.add("B");
vector.add("C");

vector.get(1);             // B
vector.remove(0);          // remove index 0
System.out.println(vector); // [B, C]
```

##### 4. `Stack` (Legacy)

**Internal Structure:** Extends `Vector` — LIFO stack

```text
        push ->  +-----+
                 |  C  |  <- top
                 +-----+
                 |  B  |
                 +-----+
                 |  A  |
                 +-----+
                  stack
        pop  <-  removes from top
```

##### Characteristics:

- **Extends `Vector`** — so it inherits the same resizable array structure, and it is **also synchronized** (thread-safe but slower due to locking overhead).
- **Follows LIFO (Last-In-First-Out) order** — the element added last is the one removed first. Think of it like a stack of plates: you place a new plate on top, and you take the top plate off first.
- **Legacy class** — it existed before the modern Collection Framework. It has largely been replaced by **`ArrayDeque`**, which is faster, not synchronized, and offers the same LIFO behavior through the `Deque` interface.

**When to use:** Rarely — prefer `ArrayDeque` for stack behavior.

```java
Stack<String> stack = new Stack<>();
stack.push("A");
stack.push("B");
stack.push("C");

stack.peek();              // C (not removed)
stack.pop();               // C (removed)
stack.pop();               // B
System.out.println(stack); // [A]
```


##### 5. `CopyOnWriteArrayList`

**Internal Structure:** Copy-on-write array

```text
Read  ->  Thread 1  --->  [ A, B, C ]   (original array, shared)
Read  ->  Thread 2  --->  [ A, B, C ]

Write ->  Thread 3  --->  [ A, B, C, D ]  (new copy created)
                          original array remains unchanged for readers
```

##### Characteristics:

- **Thread-safe without explicit synchronization** — you do not need to wrap it with `synchronized` blocks or `Collections.synchronizedList()`. It handles concurrency on its own.
- **Every write operation creates a new copy of the underlying array** — this is where the name "copy-on-write" comes from. Instead of modifying the existing array, it makes a fresh copy with the change and swaps it in.
- **Iterators never throw `ConcurrentModificationException`** — because iterators work on a **snapshot** of the array taken when the iterator was created. Even if another thread modifies the list while you are iterating, your iterator keeps working on its own copy.
- **Excellent for frequent reads, poor for frequent writes** — reads are fast because no locking is needed. Writes are expensive because each write copies the entire array, which is O(n).
- **Keeps insertion order** — elements stay in the order you added them.
- **Allows null and duplicate** elements.

**When to use:** Event-listener lists, observer patterns — where reads dominate and writes are rare.

```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
list.add("A");
list.add("B");
list.add("C");

Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String v = it.next();
    if (v.equals("B")) list.add("D");   // safe — does not affect iterator
}
System.out.println(list);  // [A, B, C, D]
```

---


### 3. Explain the Set hierarchy.

```txt
java.lang.Iterable
   |
   v
java.util.Collection
   |
   v
java.util.Set (Interface)
   |
   +-- java.util.HashSet (Concrete Class)
   |        |
   |        v
   |   java.util.LinkedHashSet (Concrete Class — insertion-ordered)
   |
   +-- java.util.TreeSet (Concrete Class — sorted)
   |
   +-- java.util.concurrent.CopyOnWriteArraySet
   |        (Concrete Class — thread-safe)
   |
   +-- java.util.EnumSet (Concrete Class — specialized for enums)
   |
   +-- java.util.concurrent.ConcurrentSkipListSet
            (Concrete Class — thread-safe, sorted)
```

#### Characteristics

- `Set` is a **collection that does not allow duplicate elements**.
- **Ordering depends on the implementation**: `HashSet` has no ordering, `LinkedHashSet` maintains insertion order, `TreeSet` maintains sorted order.
- **Null elements**: `HashSet` and `LinkedHashSet` allow one null; `TreeSet` does not allow null (with natural ordering).
- **No positional access** — there is no `get(int)` because a `Set` is not index-based.
- Uses `equals()` and `hashCode()` to decide whether an element is a duplicate.
- Supports **set operations** through `addAll`, `retainAll`, `removeAll` (union, intersection, difference).

```java
Set<String> set = new HashSet<>();
set.add("A");
set.add("B");
set.add("B");          // duplicate ignored
set.add(null);         // one null allowed

set.contains("A");     // true
set.remove("B");       // removes B
System.out.println(set);  // [A, null]  (order not guaranteed)

// Iteration
for (String s : set) {
    System.out.println(s);           // A, null  (order may vary)
}

Iterator<String> it = set.iterator();
while (it.hasNext()) {
    System.out.println(it.next());
}

// Set operations
Set<String> a = new HashSet<>(Arrays.asList("A", "B", "C"));
Set<String> b = new HashSet<>(Arrays.asList("B", "C", "D"));

a.retainAll(b);        // intersection -> [B, C]
a.addAll(b);           // union
a.removeAll(b);        // difference
```

---

##### 1. `HashSet`

**Internal Structure:** Hash table (backed by a `HashMap`)

```text
Bucket Index (hash-based)
   |
   +-- [0]  ->  null
   +-- [1]  ->  "A"
   +-- [2]  ->  "C" -> "D"   (collision handled by linked list / tree)
   +-- [3]  ->  null
   +-- [4]  ->  "B"
   ...
```

##### Characteristics:

- **Constant-time performance** — `add`, `remove`, `contains` are **O(1)** on average.
- **No ordering guarantee** — the iteration order can change when elements are added or removed.
- **Does not allow duplicates**.
- **Allows one null** element.
- **Default initial capacity: 16**; **load factor: 0.75** — when the table becomes 75% full, it doubles in size.
- **Not synchronized (not thread-safe)** — use `Collections.synchronizedSet()` or `ConcurrentHashMap.newKeySet()` for concurrent access.

**When to use:** General-purpose set when order does not matter and fast lookup is needed.

```java
Set<String> set = new HashSet<>();
set.add("A");
set.add("B");
set.add("B");              // duplicate ignored
set.add(null);             // one null allowed

set.contains("A");         // O(1) -> true
set.remove("B");           // O(1)
System.out.println(set);   // [A, null]  (order not guaranteed)
```


##### 2. `LinkedHashSet`

**Internal Structure:** Hash table with a doubly-linked list running through all entries

```text
Insertion Order:  A -> C -> B -> D

   +-----+     +-----+     +-----+     +-----+
   |  A  |<--->|  C  |<--->|  B  |<--->|  D  |
   +-----+     +-----+     +-----+     +-----+
      ^                                   ^
      |                                   |
     head                                tail
```

##### Characteristics:

- **Maintains insertion order** — elements iterate in the order they were added.
- **Constant-time performance** — `add`, `remove`, `contains` are **O(1)** on average, almost as fast as `HashSet`.
- **Does not allow duplicates**.
- **Allows one null** element.
- **Default initial capacity: 16**; **load factor: 0.75**.
- **Not synchronized (not thread-safe)**.
- **Slightly higher memory usage** than `HashSet` due to the extra linked list.

**When to use:** When you need set behavior but insertion order must be preserved.

```java
Set<String> set = new LinkedHashSet<>();
set.add("A");
set.add("C");
set.add("B");
set.add("A");              // duplicate ignored

System.out.println(set);   // [A, C, B]  (insertion order preserved)
set.contains("C");         // O(1) -> true
set.remove("B");           // O(1)
```


##### 3. `TreeSet`

**Internal Structure:** Red-black tree (self-balancing binary search tree)

```text
                    (B)
                   /   \
                (A)     (D)
                       /   \
                    (C)     (E)

In-order traversal: A, B, C, D, E   (sorted)
```

##### Characteristics:

- **Sorted order (Ascending by default)** — elements are automatically arranged in ascending order based on their natural ordering (e.g., adding `30, 10, 20` stores them as `10, 20, 30`). A custom `Comparator` can be used to define a different order, such as descending order (`30, 20, 10`).
- **Logarithmic performance** — `add`, `remove`, `contains` are **O(log n)**.
- **Does not allow duplicates**.
- **Does not allow null** (with natural ordering) — throws `NullPointerException`.
- **Not synchronized (not thread-safe)**.

**When to use:** When you need sorted iteration or range-based queries.

```java
// Natural ordering (Ascending order)
TreeSet<Integer> set = new TreeSet<>();
set.add(30);
set.add(10);
set.add(20);
set.add(10);               // duplicate ignored

System.out.println(set);   // [10, 20, 30] (ascending order)
set.first();               // 10
set.last();                // 30
set.higher(20);            // 30
set.lower(20);             // 10
set.headSet(20);           // [10]
set.tailSet(20);           // [20, 30]

// Custom Comparator (Descending order)
TreeSet<Integer> descSet = new TreeSet<>(Comparator.reverseOrder());
descSet.add(30);
descSet.add(10);
descSet.add(20);
descSet.add(10);           // duplicate ignored

System.out.println(descSet); // [30, 20, 10] (descending order)
descSet.first();             // 30
descSet.last();              // 10
descSet.higher(20);          // 10
descSet.lower(20);           // 30
descSet.headSet(20);         // [30]
descSet.tailSet(20);         // [20, 10]
```


##### 4. `CopyOnWriteArraySet`

**Internal Structure:** Backed by a `CopyOnWriteArrayList`

```text
Read  ->  Thread 1  --->  [ A, B, C ]   (original array, shared)
Read  ->  Thread 2  --->  [ A, B, C ]

Write ->  Thread 3  --->  [ A, B, C, D ]  (new copy created)
                          original array remains unchanged for readers
```

##### Characteristics:

- **Thread-safe without explicit synchronization**.
- **Every write creates a new copy** of the underlying array.
- **Iterators never throw `ConcurrentModificationException`** — they work on a snapshot.
- **Excellent for frequent reads, poor for frequent writes** — writes are **O(n)**.
- **Does not allow duplicates**.
- **Allows null** elements.
- **Maintains insertion order** (because it is backed by `CopyOnWriteArrayList`).

**When to use:** Small sets that are read often and modified rarely, in concurrent environments.

```java
CopyOnWriteArraySet<String> set = new CopyOnWriteArraySet<>();
set.add("A");
set.add("B");
set.add("C");

Iterator<String> it = set.iterator();
while (it.hasNext()) {
    String v = it.next();
    if (v.equals("B")) set.add("D");   // safe — does not affect iterator
}
System.out.println(set);  // [A, B, C, D]
```


##### 5. `EnumSet`

**Internal Structure:** Bit vector (a single `long` or array of `long`s)

```text
Enum: DAY { MON, TUE, WED, THU, FRI, SAT, SUN }
       bit:  0    1    2    3    4    5    6

EnumSet.of(MON, WED, FRI):
       bits:  1    0    1    0    1    0    0
             MON       WED       FRI
```

##### Characteristics:

- **Extremely fast and compact** — internally represented as a bit vector.
- **Only for enum types** — every element must be of the same enum.
- **Sorted order** — iterates in the natural order of the enum constants.
- **Does not allow null** elements.
- **Not synchronized (not thread-safe)**.
- Offers useful factory methods: `of()`, `range()`, `allOf()`, `noneOf()`, `complementOf()`, `copyOf()`.

**When to use:** When you are working with enum values and want maximum performance.

```java
enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

EnumSet<Day> weekend = EnumSet.of(Day.SAT, Day.SUN);
EnumSet<Day> workdays = EnumSet.range(Day.MON, Day.FRI);
EnumSet<Day> all = EnumSet.allOf(Day.class);

System.out.println(weekend);   // [SAT, SUN]
System.out.println(workdays);  // [MON, TUE, WED, THU, FRI]
System.out.println(all);       // [MON, TUE, WED, THU, FRI, SAT, SUN]
```

---



### 4. Explain the Queue hierarchy.

```txt
java.lang.Iterable
   |
   v
java.util.Collection
   |
   v
java.util.Queue (Interface)
   |
   +-- java.util.PriorityQueue (Concrete Class — priority-based)
   |
   +-- java.util.concurrent.ConcurrentLinkedQueue
   |        (Concrete Class — thread-safe, FIFO)
   |
   +-- java.util.concurrent.BlockingQueue (Interface)
   |        |
   |        +-- java.util.concurrent.ArrayBlockingQueue
   |        |        (Concrete Class — bounded, thread-safe)
   |        |
   |        +-- java.util.concurrent.LinkedBlockingQueue
   |        |        (Concrete Class — optionally bounded, thread-safe)
   |        |
   |        +-- java.util.concurrent.PriorityBlockingQueue
   |        |        (Concrete Class — thread-safe, priority-based)
   |        |
   |        +-- java.util.concurrent.SynchronousQueue
   |        |        (Concrete Class — zero capacity)
   |        |
   |        +-- java.util.concurrent.DelayQueue
   |        |        (Concrete Class — delay-based)
   |        |
   |        +-- java.util.concurrent.LinkedTransferQueue
   |                 (Concrete Class — transfer-based)
   |
   +-- java.util.Deque (Interface — double-ended queue)
            |
            +-- java.util.ArrayDeque (Concrete Class)
            |
            +-- java.util.LinkedList (Concrete Class — also a List)
            |
            +-- java.util.concurrent.ConcurrentLinkedDeque
            |        (Concrete Class — thread-safe)
            |
            +-- java.util.concurrent.BlockingDeque (Interface)
                     |
                     +-- java.util.concurrent.LinkedBlockingDeque
                              (Concrete Class — thread-safe)
```

#### Characteristics

- `Queue` is a **collection designed to hold elements before processing**.
- **Ordering**: Typically **FIFO (First-In-First-Out)**, but variants exist (`PriorityQueue` orders by priority, `Deque` allows both ends).
- **Duplicates allowed**; **null not allowed** in most implementations (`LinkedList` allows null, but its use as a queue discourages it).
- **No positional access** — there is no `get(int)`.
- Provides two sets of methods for insertion, removal, and inspection:
  - **Throws exception**: `add()`, `remove()`, `element()`
  - **Returns special value**: `offer()`, `poll()`, `peek()`

```java
Queue<String> queue = new LinkedList<>();
queue.offer("A");
queue.offer("B");
queue.offer("C");

queue.peek();              // A (not removed)
queue.poll();              // A (removed)
System.out.println(queue); // [B, C]

// Iteration
for (String s : queue) {
    System.out.println(s);           // B, C
}

Iterator<String> it = queue.iterator();
while (it.hasNext()) {
    System.out.println(it.next());   // B, C
}

// Deque — both ends
Deque<String> deque = new ArrayDeque<>();
deque.addFirst("X");
deque.addLast("Y");
deque.peekFirst();         // X
deque.peekLast();          // Y
```

##### 1. `PriorityQueue`

**Internal Structure:** Binary heap (stored in a resizable array)

```text
                (10)              <- smallest = highest priority
               /    \
            (20)    (30)
            /  \
         (40)  (50)

Array: [10, 20, 30, 40, 50]
```

##### Characteristics:

- **Priority-based ordering** — elements are ordered by natural ordering or a custom `Comparator`. Smallest element (or comparator-defined highest priority) is at the head.
- **Logarithmic performance** — `offer`, `poll` are **O(log n)**; `peek` is **O(1)**.
- **Duplicates ARE allowed**, but equal-priority elements may come out in any order.
- **Does not allow null** elements.
- **Not synchronized (not thread-safe)**.
- **Default initial capacity: 11**.
- Not stable — equal-priority elements have no guaranteed order.

**When to use:** Task scheduling by priority, Dijkstra's algorithm, event-driven simulations.

```java
PriorityQueue<Integer> pq = new PriorityQueue<>();
pq.offer(30);
pq.offer(10);
pq.offer(20);
pq.offer(10);              // duplicates allowed

System.out.println(pq.peek());  // 10 (smallest = highest priority)
pq.poll();                      // 10 (removed)
pq.poll();                      // 10
System.out.println(pq);         // [20, 30]

// Custom Comparator (descending)
PriorityQueue<Integer> maxHeap = new PriorityQueue<>(Comparator.reverseOrder());
maxHeap.offer(10);
maxHeap.offer(30);
maxHeap.offer(20);

System.out.println(maxHeap.peek());  // 30
```

##### 2. `ConcurrentLinkedQueue`

**Internal Structure:** Lock-free singly-linked list (CAS-based)

```text
head -> [A] -> [B] -> [C] -> null
                              ^
                             tail
```

##### Characteristics:

- **Thread-safe without locks** — uses CAS (Compare-And-Swap) operations.
- **FIFO order**.
- **Unbounded** — grows as needed; no capacity limit.
- **Iterators are weakly consistent** — never throw `ConcurrentModificationException`.
- **Does not allow null** elements.
- **Constant-time performance** — `offer`, `poll`, `peek` are **O(1)** on average.

**When to use:** High-throughput concurrent FIFO queues where blocking is not desired.

```java
ConcurrentLinkedQueue<String> queue = new ConcurrentLinkedQueue<>();
queue.offer("A");
queue.offer("B");
queue.offer("C");

System.out.println(queue.peek());  // A
queue.poll();                      // A removed
System.out.println(queue);         // [B, C]
```

##### 3. `ArrayBlockingQueue`

**Internal Structure:** Bounded array used as a circular buffer

```text
Capacity: 4

   +-----+-----+-----+-----+
   |  A  |  B  |     |     |
   +-----+-----+-----+-----+
     ^                 ^
   head              tail

put()  blocks when full
take() blocks when empty
```

##### Characteristics:

- **Thread-safe** — uses locks (`ReentrantLock`) internally.
- **Bounded** — capacity is fixed at construction and cannot change.
- **FIFO order**.
- **Blocking operations**:
  - `put()` blocks if the queue is full.
  - `take()` blocks if the queue is empty.
- **Does not allow null** elements.
- **Constant-time performance** — `offer`, `poll`, `put`, `take` are **O(1)**.
- Optionally supports **fairness** (threads served in FIFO order) — disabled by default.

**When to use:** Producer-consumer patterns where bounded buffering is required.

```java
BlockingQueue<String> queue = new ArrayBlockingQueue<>(3);
queue.put("A");            // blocks if full
queue.put("B");
queue.put("C");

System.out.println(queue.take()); // A (blocks if empty)
System.out.println(queue);        // [B, C]
```

##### 4. `LinkedBlockingQueue`

**Internal Structure:** Optionally bounded linked list with two locks (one for head, one for tail)

```text
head -> [A] -> [B] -> [C] -> null
                              ^
                             tail

putLock guards tail
takeLock guards head
```

##### Characteristics:

- **Thread-safe** — uses two separate locks for `put` and `take`, allowing higher concurrency than `ArrayBlockingQueue`.
- **Optionally bounded** — if no capacity is given, default is `Integer.MAX_VALUE` (effectively unbounded).
- **FIFO order**.
- **Blocking operations** — `put()` blocks if full; `take()` blocks if empty.
- **Does not allow null** elements.
- **Constant-time performance** — `offer`, `poll`, `put`, `take` are **O(1)**.
- **Higher throughput** than `ArrayBlockingQueue` for many producers/consumers.

**When to use:** Producer-consumer patterns, especially with many producers and consumers.

```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>(3);
queue.put("A");
queue.put("B");
queue.put("C");

System.out.println(queue.take()); // A
System.out.println(queue);        // [B, C]
```

##### 5. `PriorityBlockingQueue`

**Internal Structure:** Binary heap (thread-safe version of `PriorityQueue`)

```text
                (10)              <- smallest = highest priority
               /    \
            (20)    (30)
            /  \
         (40)  (50)

protected by a single lock
```

##### Characteristics:

- **Thread-safe priority queue** — combines `PriorityQueue`'s ordering with blocking operations.
- **Unbounded** — grows as needed; `put()` never blocks.
- **Logarithmic performance** — `offer`, `poll` are **O(log n)**.
- **Does not allow null** elements.
- **Does not guarantee order** among elements of equal priority.
- **`take()` blocks** when the queue is empty.
- Iterators are weakly consistent.

**When to use:** Priority-based task processing across multiple threads.

```java
BlockingQueue<Integer> pq = new PriorityBlockingQueue<>();
pq.offer(30);
pq.offer(10);
pq.offer(20);

System.out.println(pq.take()); // 10
System.out.println(pq.take()); // 20
System.out.println(pq.take()); // 30
```


---


### 5. Explain the Map hierarchy.

```txt
java.util.Map (Interface — separate root, does NOT extend Collection)
   |
   +-- java.util.HashMap (Concrete Class)
   |        |
   |        v
   |   java.util.LinkedHashMap (Concrete Class — insertion/access ordered)
   |
   +-- java.util.Hashtable (Concrete Class — legacy, synchronized)
   |        |
   |        v
   |   java.util.Properties (Concrete Class — String key-value pairs)
   |
   +-- java.util.SortedMap (Interface)
   |        |
   |        v
   |   java.util.NavigableMap (Interface)
   |        |
   |        v
   |   java.util.TreeMap (Concrete Class — sorted by key)
   |
   +-- java.util.WeakHashMap (Concrete Class — keys are weakly referenced)
   |
   +-- java.util.IdentityHashMap (Concrete Class — uses == instead of equals)
   |
   +-- java.util.EnumMap (Concrete Class — specialized for enum keys)
   |
   +-- java.util.concurrent.ConcurrentMap (Interface)
            |
            v
        java.util.concurrent.ConcurrentHashMap
                 (Concrete Class — thread-safe)
            |
            +-- java.util.concurrent.ConcurrentSkipListMap
                     (Concrete Class — thread-safe, sorted)
```

#### Characteristics

- `Map` is a **key-value pair mapping structure**. It does **not** extend `Collection`, but it is an official pillar of the Java Collection Framework.
- **Keys are unique** — a map cannot contain duplicate keys.
- **Values can be duplicated** — multiple keys can map to the same value.
- **Null handling depends on the implementation**: `HashMap` allows one null key and multiple null values; `Hashtable` allows neither; `TreeMap` does not allow a null key (with natural ordering) but allows null values.
- **Ordering depends on the implementation**: `HashMap` has no ordering, `LinkedHashMap` maintains insertion or access order, `TreeMap` sorts by key.
- Uses `equals()` and `hashCode()` (or `compareTo()`) to determine key uniqueness.
- Provides **three collection views**: `keySet()`, `values()`, and `entrySet()`.

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
map.put("C", 3);
map.put("A", 10);          // overwrites value for key "A"

map.get("A");              // 10
map.containsKey("B");      // true
map.containsValue(3);      // true
map.remove("C");           // removes key "C"
System.out.println(map);   // {A=10, B=2}  (order not guaranteed)

// Iteration
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}

for (String key : map.keySet()) {
    System.out.println(key);
}

for (Integer value : map.values()) {
    System.out.println(value);
}

Iterator<Map.Entry<String, Integer>> it = map.entrySet().iterator();
while (it.hasNext()) {
    Map.Entry<String, Integer> entry = it.next();
    System.out.println(entry.getKey() + " = " + entry.getValue());
}

// Convenience methods (Java 8+)
map.getOrDefault("Z", 0);              // 0
map.putIfAbsent("D", 4);               // adds only if key absent
map.computeIfAbsent("E", k -> 5);      // computes value if key absent
map.forEach((k, v) -> System.out.println(k + " = " + v));
```

##### 1. `HashMap`

**Internal Structure:** Hash table (array of buckets; each bucket is a linked list or a red-black tree after JDK 8 threshold)

```text
Bucket Index (hash-based)
   |
   +-- [0]  ->  null
   +-- [1]  ->  ("A", 1)
   +-- [2]  ->  ("C", 3) -> ("D", 4)    (collision handled by linked list / tree)
   +-- [3]  ->  null
   +-- [4]  ->  ("B", 2)
   ...
```

##### Characteristics:

- **Constant-time performance** — `put`, `get`, `remove`, `containsKey` are **O(1)** on average.
- **No ordering guarantee** — iteration order can change when entries are added or removed.
- **Does not allow duplicate keys**.
- **Allows one null key and multiple null values**.
- **Default initial capacity: 16**; **load factor: 0.75** — when the table becomes 75% full, it doubles in size.
- **Treeifies a bucket** when it contains 8 or more entries (JDK 8+) to keep worst-case performance at O(log n).
- **Not synchronized (not thread-safe)** — use `Collections.synchronizedMap()` or `ConcurrentHashMap` for concurrent access.

**When to use:** General-purpose map when order does not matter and fast lookup is needed.

```java
Map<String, Integer> map = new HashMap<>();
map.put("A", 1);
map.put("B", 2);
map.put("C", 3);
map.put("A", 10);          // overwrites value for "A"
map.put(null, 100);        // one null key allowed
map.put("D", null);        // multiple null values allowed

System.out.println(map.get("A"));          // 10
System.out.println(map.containsKey("B"));  // true
map.remove("C");
System.out.println(map);                   // order not guaranteed
```

##### 2. `LinkedHashMap`

**Internal Structure:** Hash table with a doubly-linked list running through all entries

```text
Insertion Order:  A -> C -> B -> D

   +------+     +------+     +------+     +------+
   | A=1  |<--->| C=3  |<--->| B=2  |<--->| D=4  |
   +------+     +------+     +------+     +------+
       ^                                      ^
      head                                   tail
```

##### Characteristics:

- **Maintains insertion order** — entries iterate in the order they were added.
- **Optionally access-ordered** — if `accessOrder` is `true`, entries are reordered on each `get` or `put` (useful for LRU caches).
- **Constant-time performance** — `put`, `get`, `remove` are **O(1)** on average, nearly as fast as `HashMap`.
- **Does not allow duplicate keys**.
- **Allows one null key and multiple null values**.
- **Default initial capacity: 16**; **load factor: 0.75**.
- **Slightly higher memory usage** than `HashMap` due to the extra linked list.

**When to use:** When insertion or access order matters; LRU cache implementation.

```java
Map<String, Integer> map = new LinkedHashMap<>();
map.put("A", 1);
map.put("C", 3);
map.put("B", 2);
map.put("A", 10);          // overwrites value, does not change position

System.out.println(map);   // {A=10, C=3, B=2}  (insertion order preserved)

// Access-ordered (LRU) map
Map<String, Integer> lru = new LinkedHashMap<>(16, 0.75f, true);
lru.put("A", 1);
lru.put("B", 2);
lru.put("C", 3);
lru.get("A");              // A moved to the end (most recently used)
System.out.println(lru);   // {B=2, C=3, A=1}
```


##### 3. `TreeMap`

**Internal Structure:** Red-black tree (self-balancing binary search tree) keyed by the map's keys

```text
                    (B, 2)
                   /      \
              (A, 1)      (D, 4)
                          /    \
                     (C, 3)    (E, 5)

In-order traversal: A, B, C, D, E   (sorted by key)
```

##### Characteristics:

- **Sorted order (Ascending by default)** — keys are automatically arranged in ascending order based on their natural ordering (e.g., adding keys `30, 10, 20` stores them as `10, 20, 30`). A custom `Comparator` can be used to define a different order, such as descending order (`30, 20, 10`).
- **Logarithmic performance** — `put`, `get`, `remove`, `containsKey` are **O(log n)**.
- **Does not allow duplicate keys**.
- **Does not allow a null key** (with natural ordering) — throws `NullPointerException`. Null values are allowed.
- **Not synchronized (not thread-safe)**.
- Implements `NavigableMap` — provides `firstKey()`, `lastKey()`, `higherKey()`, `lowerKey()`, `ceilingKey()`, `floorKey()`, `headMap()`, `tailMap()`, `subMap()`, `descendingMap()`.

**When to use:** When sorted key iteration or range-based queries are needed.

```java
// Natural ordering (Ascending order)
TreeMap<Integer, String> map = new TreeMap<>();
map.put(30, "C");
map.put(10, "A");
map.put(20, "B");
map.put(10, "Z");          // overwrites value for key 10

System.out.println(map);   // {10=Z, 20=B, 30=C} (ascending by key)
map.firstKey();            // 10
map.lastKey();             // 30
map.higherKey(20);         // 30
map.lowerKey(20);          // 10
map.headMap(20);           // {10=Z}
map.tailMap(20);           // {20=B, 30=C}

// Custom Comparator (Descending order)
TreeMap<Integer, String> descMap = new TreeMap<>(Comparator.reverseOrder());
descMap.put(30, "C");
descMap.put(10, "A");
descMap.put(20, "B");

System.out.println(descMap);   // {30=C, 20=B, 10=A} (descending by key)
descMap.firstKey();            // 30
descMap.lastKey();             // 10
```

##### 4. `EnumMap`

**Internal Structure:** Array indexed by enum ordinal

```text
Enum: DAY { MON, TUE, WED, THU, FRI, SAT, SUN }
      ordinal:  0    1    2    3    4    5    6

EnumMap<Day, String>:
       index:  0    1    2    3    4    5    6
       value:  X    Y    Z    -    -    -    -
```

##### Characteristics:

- **Extremely fast and compact** — internally represented as an array indexed by enum ordinal.
- **Only for enum keys** — keys must all be of the same enum type.
- **Natural order** — iterates in the natural order of the enum constants.
- **Does not allow null keys** (but allows null values).
- **Not synchronized (not thread-safe)**.
- Faster than `HashMap` when keys are enums.

**When to use:** When keys are enum constants and maximum performance is desired.

```java
enum Day { MON, TUE, WED, THU, FRI, SAT, SUN }

EnumMap<Day, String> map = new EnumMap<>(Day.class);
map.put(Day.MON, "Work");
map.put(Day.SAT, "Rest");

System.out.println(map);          // {MON=Work, SAT=Rest}  (enum order)
System.out.println(map.get(Day.MON));  // Work
```

##### 5. `ConcurrentHashMap`

**Internal Structure:** Segmented/bucketed hash table with CAS and synchronized bucket-level locking (no global lock in JDK 8+)

```text
Bucket Index (hash-based)
   |
   +-- [0]  ->  null
   +-- [1]  ->  ("A", 1)           (each bucket locked independently)
   +-- [2]  ->  ("C", 3) -> ("D", 4)
   +-- [3]  ->  null
   +-- [4]  ->  ("B", 2)
   ...
```

##### Characteristics:

- **Thread-safe without a global lock** — reads are lock-free; writes lock only the affected bucket.
- **High concurrency** — many threads can read and write simultaneously.
- **Does not allow null keys or null values**.
- **No ordering guarantee**.
- **Constant-time performance** — `put`, `get`, `remove` are **O(1)** on average.
- **Iterators are weakly consistent** — never throw `ConcurrentModificationException`.
- **Atomic compound operations**: `putIfAbsent()`, `computeIfAbsent()`, `compute()`, `merge()`, `replace()`.

**When to use:** High-concurrency maps — caches, shared registries, counters.

```java
ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();
map.put("A", 1);
map.put("B", 2);
// map.put(null, 100);   // NullPointerException
// map.put("C", null);   // NullPointerException

map.putIfAbsent("A", 99);              // does nothing — A already present
map.computeIfAbsent("C", k -> 3);      // adds C=3
map.merge("A", 10, Integer::sum);      // A = 1 + 10 = 11

System.out.println(map);               // order not guaranteed
```

---


### 6. Why doesn't `Map` extend the `Collection` interface?

- **Collection** is a group of individual values (you just add items like `add(item)`).
- **Map** is a key-value pair where every value needs a key to be stored (you use `put(key, value)`). 

Because they store data completely differently, `Map` is kept separate from the `Collection` interface.

#### What if `Map` *did* extend `Collection`?
If `Map` extended `Collection`, it would inherit the single-item `add(item)` method. This would break how maps work, because a map requires **both a key and a value** (`put(key, value)`), not just a single object. It would also force maps to inherit an iterator that returns individual values instead of key-value pairs, which breaks the fundamental purpose of a map.


---


### 7. What is the difference between `ArrayList` and `LinkedList`? Which one is faster?
The speed of these two structures depends entirely on the operational context, because their underlying memory tracking models are fundamentally distinct:

| Operational Feature | `ArrayList` | `LinkedList` |
| :--- | :--- | :--- |
| **Underlying Structure** | Resizable dynamic array packed in contiguous memory slots. | Doubly Linked List where nodes point to neighbors. |
| **Random Access (`get(index)`)**| **$O(1)$** - Instant access via index math. | **$O(n)$** - Must traverse nodes step-by-step. |
| **Insertion/Deletion (Middle)**| **$O(n)$** - Shifting array elements in memory slots. | **$O(1)$** - Once positioned, just updates pointers. |
| **Insertion/Deletion (End)** | **$O(1)$** amortized (Slows down only during array resizing). | **$O(1)$** via direct tracking pointer. |
| **Memory Overhead** | Low (Only allocates data arrays). | High (Every node must store data and two addresses). |

#### Code Selection Rules:
* Use `ArrayList` when your application performs **frequent search or read operations** with minimal index shifting.
* Use `LinkedList` if your workload relies on **constant additions and structural removals** at the terminal edges (acting as a Stack or Queue).



---


### 8. How does a `HashMap` work internally in Java? (Crucial Interview Topic)

A `HashMap` stores **key-value pairs** and lets you look them up quickly using the key. It does this using a technique called **hashing**. The idea is simple: instead of searching through all the entries one by one, we use the key to compute a **number** that tells us exactly where to store or find the value.


#### 1. The Internal Storage — Array of Buckets

Internally, a `HashMap` is just an **array of slots**. Each slot is called a **bucket**.

```text
Index 
  +---+
0 |   |   ----> Buket
  +---+
1 |   |
  +---+
2 |   |
  +---+
3 |   |
  +---+
4 |   |
  +---+
5 |   |
  +---+

Default size when we initialize a hashmap = 16 buckets (indexes 0 to 15)
```

Each bucket can hold **one entry** (a key-value pair). If two keys map to the same bucket, that is called a **collision**, and the bucket can hold a chain of entries.


#### 2. How `put(key, value)` Works — Step by Step

When you call `map.put("Alice", 25)`, the following happens.

**Step 1 — Compute the hash of the key**

```java
int hash = key.hashCode();
```

For example, if `"Alice".hashCode()` returns `63350368`, that is our raw hash.

**Step 2 — Convert the hash into a bucket index**

Java does some mixing on the hash and then uses bit-masking to fit it into the array size:

```java
int index = hash & (arrayLength - 1);   // if length is a power of 2
```

If the array has 16 buckets, only the low 4 bits of the hash are used to pick the index.

```text
hash   = ...1010 0011
length = 16  ->  mask = 15 = 0000 1111
index  = hash & mask = 0011 = 3
```

So the entry will go into bucket 3.

**Step 3 — Look inside the bucket**

Two cases.

**Case A — Bucket is empty**

```text
Bucket 3 is empty
  +---+
3 |   |   ->  place new Node("Alice", 25) here
  +---+
```

**Case B — Bucket already has an entry (collision)**

```text
Bucket 3 already contains Node("Bob", 30)
  +-----+
3 | Bob |
  +-----+
```

If the new key equals the existing key (`equals()` returns `true`), the value is **overwritten**.

If the keys are different, the new node is **appended to the chain** in that bucket.


#### 3. What Is a Collision?

A **collision** happens when two different keys produce the **same bucket index**. This is normal and expected because the array is much smaller than the space of all possible hashes.

```text
"Alice".hashCode()  ->  index 3
"Bob".hashCode()    ->  index 3     (collision!)
"Carol".hashCode()  ->  index 7
```

Both `"Alice"` and `"Bob"` land in bucket 3.


#### 4. How Collisions Are Handled — Separate Chaining

Java uses a technique called **separate chaining**. Instead of overwriting, each bucket can hold a **chain** of entries.

```text
Bucket 3:
  +-------+    +-------+    +-------+
  | Alice | -> |  Bob  | -> | Carol | -> null
  +-------+    +-------+    +-------+
```

Each node in the chain stores:

- the **hash** of the key,
- the **key** itself,
- the **value**,
- a **next** pointer to the following node.

```java
static class Node<K, V> {
    final int hash;
    final K key;
    V value;
    Node<K, V> next;
}
```
```txt
[ HashMap Array ]       [ Bucket at Index 2 ]       [ Node 1 (Inside Bucket) ]
                      
  +---+
0 |   |                  
  +---+                  
1 |   |                  
  +---+        ┌────────┐       +-----------------------+
2 | o-+───────►│ Bucket │──────►│ hash  = 3152          │
  +---+        └────────┘       │ key   = "name"        │
3 |   |                         │ value = "John"        │
  +---+                         │ next  = ----> [Node2] │
                                +-----------------------+
```


#### 5. How Lookup (`get`) Works

When you call `map.get("Bob")`:

1. Compute the hash of `"Bob"`.
2. Convert the hash into a bucket index.
3. Go to that bucket.
4. If the bucket is empty, return `null`.
5. If the bucket has entries, walk the chain and compare each key with `equals()`.
6. If a match is found, return its value.
7. If the chain ends with no match, return `null`.

```text
get("Bob")

  hash("Bob")  ->  index 3
       |
       v
  +-------+    +-------+    +-------+
  | Alice | -> |  Bob  | -> | Carol | -> null
  +-------+    +-------+    +-------+
                   ^
                   |
             equals("Bob")? yes
                   |
                   v
               return value
```

#### 6. Java 8+ Optimization — Treeification

When a bucket accumulates many entries, the chain becomes a **linked list**, and lookup inside that bucket becomes **O(n)**. If an attacker deliberately generates many keys with the same hash, this can slow the map to a crawl (a denial-of-service attack).

To fix this, Java 8 introduced **treeification**:

- If a single bucket's chain grows past **8 nodes**,
- **and** the total map capacity is at least **64 buckets**,
- the bucket is converted from a **linked list** into a **self-balancing Red-Black Tree**.
- Lookup inside that bucket becomes **O(log n)** instead of **O(n)**.
- If the bucket later shrinks below **6 nodes**, it is converted back to a linked list.

```text
Before treeification (linked list, O(n)):

  +-------+    +-------+    +-------+    +-------+
  | A     | -> | B     | -> | C     | -> | D     | -> ...
  +-------+    +-------+    +-------+    +-------+

After treeification (red-black tree, O(log n)):

              (C)
             /   \
          (A)     (E)
            \    /   \
           (B) (D)   (F)
```

#### 7. The Full Picture

```text
HashMap Internal Structure:

Index Buckets Array
 +---+
0|   |---> Empty
 +---+
1|   |---> Node [KeyA] -> Node [KeyB]        (linked list, collision chain)
 +---+
2|   |---> Node [KeyX] -> Node [KeyY]        (linked list, collision chain)
 +---+
3|   |---> Red-Black Tree node                (treeified, chain size > 8)
 +---+
4|   |---> Empty
 +---+
5|   |---> Node [KeyZ]
 +---+
...
 +---+
15|  |---> Empty
 +---+
```



#### 8. Summary

```text
1. HashMap stores data in an array of buckets.

2. put(key, value):
      - compute hash of key
      - map hash to a bucket index
      - if bucket is empty, insert new node
      - if bucket is occupied (collision), chain or overwrite

3. Collisions are handled with separate chaining (linked list).

4. Java 8+ treeifies a bucket when its chain grows past 8 nodes
   and the map has at least 64 buckets, giving O(log n) lookup.

5. Resizing happens when the map becomes 75% full:
   capacity doubles and all entries are rehashed.
```


---


### 9. What is the contract between `hashCode()` and `equals()`?

The contract between the two methods is:

- **Rule 1** — If two objects are equal according to `equals()`, then `hashCode()` on each **must return the same integer**.

```text
a.equals(b) == true   ->   a.hashCode() == b.hashCode()   (MUST hold)
```

- **Rule 2** — If two objects have the same `hashCode()`, they are **not necessarily equal**. That is just a collision, and `equals()` must still be checked.

```text
a.hashCode() == b.hashCode()   ->   a.equals(b) may be true or false
```

- **Rule 3** — If you override `equals()`, you **must** override `hashCode()` as well, so that equal objects produce equal hash codes.


#### What Happens If We Don't Override `hashCode()`?

**Setup**

```java
UserToken k1 = new UserToken(101);
UserToken k2 = new UserToken(101);
```

- `k1` and `k2` are **two separate objects in memory**.
- Both have `id = 101`.
- `equals()` is overridden to compare `id`, so it says they are **logically equal**.
- `hashCode()` is **NOT** overridden, so both objects inherit the default `hashCode()` from `Object`, which is based on their memory address.

```text
k1.equals(k2)   ->  true      (same content)
k1.hashCode()   ->  9871      (different!)
k2.hashCode()   ->  4482
```

Even though `equals()` says they are equal, they are **not the same object**. They are two distinct objects that merely hold the same value.


**Only `k1` is stored in the map**

When we call `map.put(k1, "Authorized User")`, only `k1` is stored:

```text
put(k1, "Authorized User"):

  k1.hashCode() = 9871  ->  bucket 3

  Bucket 3: [k1 -> "Authorized User"]

  k2 is NOT in the map.
  k2 is just a separate object in the program.
```

So the map contains exactly one entry: `k1 -> "Authorized User"` in bucket 3.


**Now we call `get(k2)`**.

`k2` is a different object from `k1`, but it holds the same value (`id = 101`). Logically, it represents the same key, so we expect the map to return `"Authorized User"`.


```text
get(k2):

  k2.hashCode() = 4482  ->  bucket 7

  Bucket 0: (empty)
  Bucket 1: (empty)
  Bucket 2: (empty)
  Bucket 3: [k1 -> "Authorized User"]   <-- k2 never looks here
  Bucket 4: (empty)
  ...
  Bucket 7: (empty)                     <-- lookup lands here

  Result: null   (WRONG)
```

`k2` is a different object, so its hash code sends the lookup to bucket 7. Bucket 7 is empty, so the map returns `null`. The lookup never reaches bucket 3 where `k1` is sitting.

So even though `k2.equals(k1)` is `true`, the map cannot find `k1` because `k2` and `k1` are sent to different buckets.


**Why**

```text
HashMap uses hashCode() FIRST to pick a bucket.
If k1 and k2 have different hash codes,
they land in different buckets,
and the lookup for k2 never finds k1.
```

The problem is not that `k1` and `k2` are compared and rejected — the problem is that they are never compared at all, because they never end up in the same bucket.


**Fix — override `hashCode()` too**

```java
@Override
public int hashCode() {
    return Objects.hash(id);
}
```

Now:

```text
k1.hashCode() = 9871
k2.hashCode() = 9871      (same)


put(k1)  ->  bucket 3
get(k2)  ->  bucket 3

  Bucket 3: [k1 -> "Authorized User"]
                 |
                 v
           k1.equals(k2)? yes
                 |
                 v
        return "Authorized User"   (CORRECT)
```


**Rule**

```text
equals()       ->  tells whether two objects are logically equal
hashCode()     ->  tells which bucket an object goes into

Two objects can be equal by equals() but still be different objects.
If their hashCode() differs, they land in different buckets,
and the map never compares them at all.

If you override equals(), you MUST override hashCode().
```

**Incorrect Version**  — only override  `equals()`

```java
import java.util.HashMap;

class UserToken {
    private int id;

    UserToken(int id) { this.id = id; }

    // equals() is overridden to compare content (id)
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        UserToken other = (UserToken) obj;
        return this.id == other.id;
    }

    // hashCode() is NOT overridden  ->  default from Object (memory address)
}

public class HashContractDemo {
    public static void main(String[] args) {

        HashMap<UserToken, String> map = new HashMap<>();

        // Step 1: store the value using k1
        UserToken k1 = new UserToken(101);
        map.put(k1, "Authorized User");

        // Step 2: later, we no longer have k1.
        //         we create a new object k2 with the same id to look up the value.
        UserToken k2 = new UserToken(101);

        System.out.println("k1.equals(k2): " + k1.equals(k2));   // true
        System.out.println("k1.hashCode(): " + k1.hashCode());   // e.g., 9871
        System.out.println("k2.hashCode(): " + k2.hashCode());   // e.g., 4482

        // Step 3: try to fetch the value using k2
        String result = map.get(k2);

        System.out.println("Result: " + result);                 // null  (WRONG)
    }
}
```



**Correct Version — override both `equals()` and `hashCode()`**

```java
import java.util.HashMap;
import java.util.Objects;

class UserToken {
    private int id;

    UserToken(int id) { this.id = id; }

    // equals() compares content (id)
    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        UserToken other = (UserToken) obj;
        return this.id == other.id;
    }

    // hashCode() is now overridden to match equals()
    @Override
    public int hashCode() {
        return Objects.hash(id);
    }
}

public class HashContractDemo {
    public static void main(String[] args) {

        HashMap<UserToken, String> map = new HashMap<>();

        // Step 1: store the value using k1
        UserToken k1 = new UserToken(101);
        map.put(k1, "Authorized User");

        // Step 2: later, we create a new object k2 with the same id
        UserToken k2 = new UserToken(101);

        // k1.equals(k2) is true
        System.out.println("k1.equals(k2): " + k1.equals(k2));   // true

        // k1.hashCode() == k2.hashCode()
        System.out.println("k1.hashCode(): " + k1.hashCode());   // 132
        System.out.println("k2.hashCode(): " + k2.hashCode());   // 132

        // Step 3: fetch the value using k2
        String result = map.get(k2);

        System.out.println("Result: " + result);                 // Authorized User
    }
}
```

**What happens inside the map**

```text
put(k1, "Authorized User"):
    k1.hashCode() = 132  ->  bucket 3
    Bucket 3: [k1 -> "Authorized User"]

get(k2):
    k2.hashCode() = 132  ->  bucket 3   (same bucket)

    Bucket 3: [k1 -> "Authorized User"]
                   |
                   v
             k1.equals(k2)?  yes
                   |
                   v
          return "Authorized User"   (CORRECT)
```

Because `hashCode()` is overridden to match `equals()`, both `k1` and `k2` land in the **same bucket**, and the lookup succeeds.


---


### 10. What is the difference between Fail-Fast and Fail-Safe Iterators?

When iterating over a collection, changes made to the structure by other thread contexts are handled via two tracking models:

* **Fail-Fast Iterators**: Operate directly on the raw live collection data structure itself. They maintain an internal tracking flag called **`modCount`** (modification counter). If the structure is modified (adding, deleting) while iterating without using the iterator's own structural methods, a **`ConcurrentModificationException`** is thrown instantly.
  * *Examples*: Iterators for `ArrayList`, `HashSet`, `HashMap`.
* **Fail-Safe / Fail-Weak Iterators**: Operate on a **detached clone copy** of the collection or use concurrent memory slot architectures. They can safely handle modifications mid-loop without throwing any exception, though they might not reflect changes made after the copy step.
  * *Examples*: Iterators for `CopyOnWriteArrayList`, `ConcurrentHashMap`.

#### Fail-Fast Code Example

```java
import java.util.ArrayList;
import java.util.Iterator;
import java.util.List;

public class FailFastDemo {
    public static void main(String[] args) {
        List<String> list = new ArrayList<>();
        list.add("A");
        list.add("B");

        Iterator<String> it = list.iterator();
        while (it.hasNext()) {
            String val = it.next();

            // CRITICAL TRAP: Mutating the raw list directly mid-iteration
            if (val.equals("A")) {
                list.remove(val);   // throws ConcurrentModificationException
            }
        }
    }
}
```

**Output**

```text
Exception in thread "main" java.util.ConcurrentModificationException
    at java.base/java.util.ArrayList$Itr.checkForComodification(ArrayList.java:...)
    at java.base/java.util.ArrayList$Itr.next(ArrayList.java:...)
    at FailFastDemo.main(FailFastDemo.java:...)
```

**What happens inside**

```text
modCount = 0

it.next()  ->  checkForComodification()
                expectedModCount == modCount ?  yes  ->  continue

list.remove("A")  ->  modCount++   (modCount = 1)

it.next()  ->  checkForComodification()
                expectedModCount (0) != modCount (1)  ->  throw
                ConcurrentModificationException
```


#### Fail-Safe Code Example

```java
import java.util.Iterator;
import java.util.List;
import java.util.concurrent.CopyOnWriteArrayList;

public class FailSafeDemo {
    public static void main(String[] args) {
        List<String> list = new CopyOnWriteArrayList<>();
        list.add("A");
        list.add("B");

        Iterator<String> it = list.iterator();
        while (it.hasNext()) {
            String val = it.next();

            // Mutating the list mid-iteration is SAFE here
            if (val.equals("A")) {
                list.remove(val);   // no exception
            }
        }

        System.out.println("Final list: " + list);
    }
}
```

**Output**

```text
Final list: [B]
```

**What happens inside**

```text
Iterator is created  ->  takes a snapshot of the array  [A, B]
                         (the snapshot is never modified)

list.remove("A")     ->  creates a NEW array  [B]
                         and swaps it in
                         (the snapshot is untouched)

Iterator keeps iterating over the snapshot [A, B]
                         (no exception, even though the list changed)

Final list:  [B]
```

#### Side-by-Side Comparison

```text
+---------------------+---------------------------+-----------------------------+
| Feature             | Fail-Fast                 | Fail-Safe                   |
+---------------------+---------------------------+-----------------------------+
| Operates on         | Live collection           | Snapshot / clone            |
| Detects modification| Yes, via modCount         | No                          |
| Throws exception    | ConcurrentModificationEx. | Never                       |
| Sees updates        | Yes, but throws           | No (works on stale copy)    |
| Examples            | ArrayList, HashSet,       | CopyOnWriteArrayList,       |
|                     | HashMap                   | ConcurrentHashMap           |
+---------------------+---------------------------+-----------------------------+
```


---


### 11. Difference between Synchronized Collection and Concurrent Collection.

In Java, both **Synchronized Collections** and **Concurrent Collections** provide thread-safe operations, meaning multiple threads can access or modify them without causing data corruption. However, they achieve thread safety and performance through fundamentally different mechanisms.



#### 1. Synchronized Collections

Synchronized collections are traditional thread-safe data structures wrapped using wrapper methods like `Collections.synchronizedList()` or `Collections.synchronizedMap()`.

##### How They Work
- **Mechanism:** They typically use a single, intrinsic lock (monitor lock) on the collection object itself for **every** operation (read, write, update).
- **Blocking:** When one thread is reading or writing to the collection, *all other threads* attempting any operation (even just reading) are blocked until the lock is released.

##### Key Characteristics
- **Performance:** Low concurrency and high contention. Because a single lock is shared across the entire collection, performance degrades significantly as the number of threads increases.
- **Fail-Fast Iterators:** Their iterators are **fail-fast**. If the collection is structurally modified (add/remove) by any thread while another thread is iterating over it, a `ConcurrentModificationException` is thrown.


##### Synchronized Collection — Code Example

```java
import java.util.Collections;
import java.util.List;
import java.util.ArrayList;

public class SynchronizedCollectionDemo {
    public static void main(String[] args) throws InterruptedException {

        // 1. Create a synchronized wrapper around a regular ArrayList
        List<String> list = Collections.synchronizedList(new ArrayList<>());

        list.add("A");
        list.add("B");
        list.add("C");

        // 2. Thread 1 — iterates over the list
        Thread reader = new Thread(() -> {
            synchronized (list) {              // manual lock for iteration
                for (String s : list) {
                    System.out.println("Reader: " + s);
                    try { Thread.sleep(100); } catch (InterruptedException e) {}
                }
            }
        });

        // 3. Thread 2 — modifies the list while Thread 1 is iterating
        Thread writer = new Thread(() -> {
            try { Thread.sleep(150); } catch (InterruptedException e) {}
            list.add("D");                      // synchronized method — allowed
            System.out.println("Writer added D");
        });

        reader.start();
        writer.start();

        reader.join();
        writer.join();

        System.out.println("Final list: " + list);
    }
}
```

**Output (with `synchronized (list)` around the loop)**

```text
Reader: A
Reader: B
Reader: C
Writer added D
Final list: [A, B, C, D]
```

The writer waits until the reader finishes iterating, because the reader holds the lock on `list` for the entire loop. No exception is thrown.


**Why iteration needs manual synchronization**

The wrapper synchronizes each individual method call (`add`, `get`, `remove`), but a loop calls `hasNext()` and `next()` as separate calls. Another thread can modify the list between those calls, causing a `ConcurrentModificationException`. So you must wrap the whole loop inside `synchronized (list) { ... }`.

```java
import java.util.Collections;
import java.util.List;
import java.util.ArrayList;

public class ManualSyncDemo {
    public static void main(String[] args) throws InterruptedException {

        List<String> list = Collections.synchronizedList(new ArrayList<>());
        list.add("A");
        list.add("B");
        list.add("C");

        // Reader thread — iterates WITHOUT manual synchronization
        Thread reader = new Thread(() -> {
            try {
                for (String s : list) {          // no synchronized (list)
                    System.out.println("Reader: " + s);
                    Thread.sleep(100);
                }
            } catch (Exception e) {
                System.out.println("Reader threw: " + e.getClass().getSimpleName());
            }
        });

        // Writer thread — modifies the list mid-iteration
        Thread writer = new Thread(() -> {
            try { Thread.sleep(150); } catch (InterruptedException e) {}
            list.add("D");                        // allowed — method is synchronized
            System.out.println("Writer added D");
        });

        reader.start();
        writer.start();
        reader.join();
        writer.join();

        System.out.println("Final list: " + list);
    }
}
```

**Output (typical)**

```text
Reader: A
Reader: B
Writer added D
Reader threw: ConcurrentModificationException
Final list: [A, B, C, D]
```

The wrapper locked each `hasNext()` and `next()` call individually, but not the whole loop. The writer slipped in between calls and changed the list, causing the exception.


#### 2. Concurrent Collections

Concurrent collections are part of the `java.util.concurrent` package (e.g., `ConcurrentHashMap`, `CopyOnWriteArrayList`, `BlockingQueue`).

##### How They Work
- **Mechanism:** They use modern concurrency techniques such as lock striping, compare-and-swap (CAS) operations, or fine-grained locking instead of locking the entire object.
- **Non-Blocking / Segmented Locking:** Different threads can often read and write concurrently to different parts of the collection without blocking each other (for example, `ConcurrentHashMap` locks only specific buckets/segments).

##### Key Characteristics
- **Performance:** High concurrency and scalability. They are optimized for multi-threaded environments with heavy read/write traffic.
- **Weakly Consistent Iterators:** Their iterators are **weakly consistent**. They do **not** throw `ConcurrentModificationException`. Instead, they reflect elements as they were at the iterator's creation or safely incorporate modifications made during iteration.



##### Concurrent Collection — Code Example with Threads

```java
import java.util.concurrent.ConcurrentHashMap;

public class ConcurrentCollectionDemo {
    public static void main(String[] args) throws InterruptedException {

        // 1. Create a ConcurrentHashMap (thread-safe without external locks)
        ConcurrentHashMap<String, Integer> map = new ConcurrentHashMap<>();

        map.put("A", 1);
        map.put("B", 2);
        map.put("C", 3);

        // 2. Thread 1 — iterates over the map (NO manual synchronization needed)
        Thread reader = new Thread(() -> {
            for (String key : map.keySet()) {
                System.out.println("Reader: " + key + " = " + map.get(key));
                try { Thread.sleep(100); } catch (InterruptedException e) {}
            }
        });

        // 3. Thread 2 — modifies the map while Thread 1 is iterating
        Thread writer = new Thread(() -> {
            try { Thread.sleep(150); } catch (InterruptedException e) {}
            map.put("D", 4);                     // safe — no exception
            System.out.println("Writer added D");
        });

        reader.start();
        writer.start();

        reader.join();
        writer.join();

        System.out.println("Final map: " + map);
    }
}
```

**Output (typical)**

```text
Reader: A = 1
Reader: B = 2
Reader: C = 3
Writer added D
Final map: {A=1, B=2, C=3, D=4}
```

No `ConcurrentModificationException`, and no `synchronized` block needed.


**Why no exception is thrown**

```text
ConcurrentHashMap uses a weakly consistent iterator.

When the reader iterates:
    - it does NOT snapshot the entire map
    - it walks the buckets directly, but tolerates concurrent changes

When the writer calls put("D", 4):
    - it locks only the specific bucket for "D"
    - other buckets remain available to readers

The reader may or may not see "D" during iteration,
but it will NEVER throw ConcurrentModificationException.
```

**What the reader actually sees**

```text
Reader: A = 1
Reader: B = 2
Reader: C = 3

(the reader may or may not see "D" — depends on timing)
```

The final map contains `D` because the writer's change was applied to the live map.

**Key takeaway**

```text
Synchronized Collection (e.g., synchronizedList):
    - needs synchronized (list) around the loop
    - otherwise  ->  ConcurrentModificationException

Concurrent Collection (e.g., ConcurrentHashMap):
    - no manual synchronization needed
    - iterator is weakly consistent
    - never throws ConcurrentModificationException
    - may or may not see updates made during iteration
```




#### Key Differences

```text
+------------------------+-------------------------------+-------------------------------+
| Feature                | Synchronized Collection       | Concurrent Collection         |
+------------------------+-------------------------------+-------------------------------+
| Locking                | Single lock on entire         | Fine-grained locks per bucket |
|                        | collection                    | or lock-free (CAS)            |
| Concurrency            | Only one thread at a time     | Many threads can read/write   |
|                        |                               | simultaneously                |
| Performance            | Slow under high contention    | High throughput               |
| Iteration              | NOT safe — must manually      | Safe — weakly consistent      |
|                        | synchronize the block         | iterators                     |
| Fail-Fast              | Yes                           | No (weakly consistent)        |
| Null support           | Depends on backing collection | ConcurrentHashMap does NOT    |
|                        |                               | allow null keys/values        |
| Introduced in          | Java 1.2 (Collections utility)| Java 1.5 (java.util.concurrent)|
| Examples               | synchronizedList,             | ConcurrentHashMap,            |
|                        | synchronizedMap,              | CopyOnWriteArrayList,         |
|                        | synchronizedSet               | ConcurrentLinkedQueue         |
+------------------------+-------------------------------+-------------------------------+
```
























