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


















