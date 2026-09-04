# Lecture 2.1: C++ STL Containers & Abstract Data Types

## Overview

Containers are data structures provided by the C++ Standard Template Library (STL) to store, organize, and manage collections of objects with automatic memory management. 

In this module, we explore:
1. **The Four STL Container Categories:** Sequence containers, associative containers, unordered containers, and container adapters.
2. **Access Patterns & Selection Criteria:** How architectural requirements (random access, uniqueness, ordering, lookup vs. mutation frequency) dictate container choice.
3. **Containers as Abstract Data Types (ADTs):** Defining interfaces, formal axioms, and operational guarantees.
4. **Behavioral Subtleties in STL:** Map subscripting side-effects (`operator[]` vs. `.at()`).
5. **Unified Interfaces & Generic Algorithms:** Standardized iterator abstractions enabling decoupled, reusable algorithms.

### Summary Taxonomy of STL Containers

| Family | Organizational Principle | Key Characteristics | Standard Library Containers |
|---|---|---|---|
| **Sequence Containers** | Linear, position-based | Preserves insertion order; direct positional access | `std::vector`, `std::array`, `std::deque`, `std::list`, `std::forward_list` |
| **Associative Containers** | Key-ordered (Red-Black Trees) | Automatically sorted; strict weak ordering; $O(\log n)$ | `std::set`, `std::multiset`, `std::map`, `std::multimap` |
| **Unordered Associative** | Hashed (Hash Tables) | Key-based lookup via hashing; average $O(1)$ | `std::unordered_set`, `std::unordered_map`, `std::unordered_multiset`, `std::unordered_multimap` |
| **Container Adapters** | Constrained interfaces | Restricts access to specific protocols (LIFO/FIFO/Heap) | `std::stack`, `std::queue`, `std::priority_queue` |

---

## 1. Classification of STL Containers

The C++ Standard Library organizes containers into four architectural families based on memory layout and data access guarantees.

### 1. Sequence Containers
Sequence containers store elements in a linear sequence where an element's position depends on when and where it was inserted.

- **`std::vector`:** Dynamically resizable contiguous array.
  - **Complexity:** $O(1)$ random access, amortized $O(1)$ append at end, $O(n)$ insertion/removal in middle.
  - **Default Choice:** Offers superior CPU cache locality due to contiguous memory allocation.
- **`std::array`:** Fixed-size compile-time sequence allocated directly on the stack without heap allocation overhead.
- **`std::deque`:** Double-ended queue structured as segmented chunks of indexed arrays.
  - **Complexity:** $O(1)$ random access, $O(1)$ push/pop at both front and back.
- **`std::list`:** Doubly linked list.
  - **Complexity:** $O(1)$ node insertion/removal anywhere given an iterator; $O(n)$ traversal/search. No random access.
- **`std::forward_list`:** Singly linked list. More memory-efficient than `std::list` by omitting backward pointers.

---

### 2. Associative Containers
Associative containers store sorted elements based on strict weak ordering of keys. They are typically implemented as self-balancing binary search trees (specifically, **Red-Black Trees**).

- **`std::set`:** Unique keys stored in sorted order.
- **`std::multiset`:** Keys stored in sorted order, allowing duplicates.
- **`std::map`:** Unique key-value pairs (`std::pair<const Key, Value>`) sorted by key.
- **`std::multimap`:** Sorted key-value pairs allowing duplicate keys.
- **Complexity:** Search, insertion, and deletion all take strictly $O(\log n)$ worst-case time.

---

### 3. Unordered Associative Containers
Introduced in C++11, these organize elements into bucketed **hash tables**. Elements have no guaranteed ordering.

- **`std::unordered_set`:** Unique keys organized via a hash function.
- **`std::unordered_multiset`:** Hashed keys allowing duplicates.
- **`std::unordered_map`:** Unique key-value pairs accessed via key hashing.
- **`std::unordered_multimap`:** Key-value pairs allowing duplicate hashed keys.
- **Complexity:** Average-case $O(1)$ search, insert, and delete. Degrades to $O(n)$ in the worst case under frequent hash collisions.

---

### 4. Container Adapters
Adapters provide constrained, specialized interfaces on top of underlying sequence containers (defaulting to `std::deque` or `std::vector`). They encapsulate internal mechanics and do not expose iterators directly.

- **`std::stack`:** LIFO (Last In, First Out) interface providing `.push()`, `.pop()`, and `.top()`.
- **`std::queue`:** FIFO (First In, First Out) interface providing `.push()`, `.pop()`, `.front()`, and `.back()`.
- **`std::priority_queue`:** Max-heap (or custom comparator heap) maintaining the highest-priority element at `.top()`.
  - **Complexity:** $O(\log n)$ insertion (`.push()`) and removal (`.pop()`), $O(1)$ top inspection (`.top()`).

---

### Complexity & Trade-Off Summary

| Container | Underlying Data Structure | Random Access | Search Time | Insert / Erase Time | Key Advantage |
|---|---|:---:|:---:|:---:|---|
| **`std::vector`** | Contiguous heap array | $O(1)$ | $O(n)$ | Amortized $O(1)$ back, $O(n)$ middle | Cache locality, minimal overhead |
| **`std::deque`** | Map of fixed-size chunks | $O(1)$ | $O(n)$ | $O(1)$ front and back | Non-relocating buffer growth |
| **`std::list`** | Doubly linked list | $O(n)$ | $O(n)$ | $O(1)$ at known iterator | Iterator/pointer validity stability |
| **`std::set` / `std::map`** | Red-Black Tree | No | $O(\log n)$ | $O(\log n)$ | Always sorted, $O(\log n)$ range queries |
| **`std::unordered_map`** | Hash Table (Chaining) | No | Avg $O(1)$, Worst $O(n)$ | Avg $O(1)$, Worst $O(n)$ | Constant-time key lookup |

---

### Example: Using `std::set`

```cpp
#include <iostream>
#include <set>

int main() {
    std::set<int> s;

    // Inserting elements in reverse order:
    for (int i = 5; i >= 1; i--) {
        s.insert(i * 10); // s automatically stores sorted: {10, 20, 30, 40, 50}
    }

    s.insert(20); // Duplicate key: rejected, no new element inserted
    s.erase(20);  // Removes 20: {10, 30, 40, 50}

    // C++20 lookup method:
    if (s.contains(40)) {
        std::cout << "s contains 40!\n";
    }

    // In-order traversal:
    for (int val : s) {
        std::cout << val << " ";
    }
    std::cout << "\n";

    return 0;
}
```

---

## 2. Selection Criteria: Choosing the Right Container

Container selection is governed by access patterns, memory constraints, and frequency of mutations.

### Container Decision Matrix

| Primary Requirement | Sub-Condition | Best Container Choice | Rationale |
|---|---|---|---|
| **Linear Sequence** | Frequent index lookups & back appends | `std::vector` | Contiguous memory, cache locality |
| **Linear Sequence** | Fast insertion/removal at both ends | `std::deque` | Indexed chunk buffers, non-relocating |
| **Linear Sequence** | Frequent middle insertions/splices | `std::list` | $O(1)$ pointer unlinking with iterator |
| **Key-Value Mapping** | Sorted key order required | `std::map` | Red-Black Tree ($O(\log n)$ range queries) |
| **Key-Value Mapping** | Maximum lookup speed (order irrelevant) | `std::unordered_map` | Hash Table (Average $O(1)$ lookup) |
| **Unique Key Set** | Sorted keys required | `std::set` | In-order tree traversal ($O(\log n)$) |
| **Unique Key Set** | Maximum membership check speed | `std::unordered_set` | Hash Table (Average $O(1)$) |
| **Protocol Queue** | Strict FIFO arrival order | `std::queue` | Encapsulated front/back access |
| **Priority Queue** | Dynamic largest/highest element access | `std::priority_queue` | Binary Heap ($O(\log n)$ push/pop) |

---

### Technical Decision Factors

1. **Enforcing Uniqueness:**
   - When duplicate items are forbidden, using `std::vector` requires an $O(n)$ linear scan prior to every insertion.
   - **Recommended:** `std::set` (ordered) or `std::unordered_set` (hash-based $O(1)$ average).

2. **Preserving Insertion Order:**
   - Associative and unordered containers reorder elements via keys or hash buckets, destroying the temporal insertion timeline.
   - **Recommended:** `std::vector`, `std::deque`, or `std::queue` (FIFO).

3. **Random Access vs. Sequential Traversal:**
   - If an algorithm relies on indexed access (`data[i]`) or binary search, contiguous memory (`std::vector`, `std::array`) or chunked memory (`std::deque`) is mandatory.
   - Node-based structures (`std::list`, `std::set`) incur pointer chasing and lack $O(1)$ random indexing.

4. **Small Static Collections with Frequent Lookups:**
   - While `std::unordered_set` offers theoretical $O(1)$ lookup, hash computations and pointer indirection introduce overhead on small collections ($N \le 100$).
   - **Recommended:** A sorted `std::vector` combined with `std::binary_search` often outperforms hash sets due to cache locality and zero memory allocation overhead.

5. **High-Throughput Insertions with Sporadic Deletions:**
   - Erasing an element from the middle of a `std::vector` requires shifting $O(n)$ elements.
   - **Recommended:** `std::unordered_map` / `std::unordered_set` ($O(1)$ average erasure without shifting memory) or `std::list` ($O(1)$ node splice if iterator is held).

---

### Exercise 2.3: Scenario-Based Analysis

| Scenario | Recommended Container | Architectural Rationale |
|---|---|---|
| **1. CPUs in a machine** | `std::vector<CPU>` (or `std::array<CPU, N>`) | The CPU count is established at initialization and remains static. Access is strictly by core ID / numerical index (`cpus[0]`, `cpus[1]`), demanding $O(1)$ index access and cache locality. |
| **2. Incoming service requests** | `std::queue<Request>` (or `std::priority_queue<Request>`) | Requests must be processed in arrival order (FIFO). If requests carry priority levels (e.g., latency-critical vs. background), `std::priority_queue` ensures highest-priority execution. |
| **3. Food items on a menu** | `std::unordered_map<ItemID, MenuItem>` or `std::map<ItemID, MenuItem>` | A menu maps unique keys (Item ID / Name) to attributes (price, ingredients). Use `std::unordered_map` for fast $O(1)$ lookup, or `std::map` if the menu must be displayed in alphabetical order. |
| **4. Shopping cart on an e-commerce site** | `std::unordered_map<ItemID, int>` | A cart maps product IDs to quantities. Adding items requires checking existence ($O(1)$), incrementing quantities, or inserting new pairs. Removing items is also an $O(1)$ average operation. |

---

## 3. Containers as Abstract Data Types (ADTs)

An **Abstract Data Type (ADT)** is a mathematical model for data structures where behavior is defined by its **interface and operational guarantees (axioms)** rather than its concrete physical implementation.

### ADT Interface vs. Implementations

| Abstract Data Type | Core Abstract Operations | STL Realization | Physical Implementation |
|---|---|---|---|
| **Set ADT** | `insert(v)`, `erase(v)`, `contains(v)` | `std::set` | Red-Black Tree (Balanced BST) |
| **Set ADT** | `insert(v)`, `erase(v)`, `contains(v)` | `std::unordered_set` | Hash Table with separate chaining |
| **Map ADT** | `put(k, v)`, `get(k)`, `remove(k)` | `std::map` | Red-Black Tree (Key-value nodes) |
| **Map ADT** | `put(k, v)`, `get(k)`, `remove(k)` | `std::unordered_map` | Hash Table (Buckets of pairs) |

---

### Example 2.2: Formal Axioms of the Set ADT

A `Set` ADT with operations `insert(v)`, `erase(v)`, and `contains(v)` satisfies the following formal algebraic axioms (for all elements $u, v$ where $u \ne v$):

1. **Initial State (Empty Set):**
   $$\text{Set } s; \implies s.\text{contains}(v) = \text{false}$$
2. **Direct Insert Guarantee:**
   $$s.\text{insert}(v); \implies s.\text{contains}(v) = \text{true}$$
3. **Independent Insert Invariance:**
   $$\text{let } x = s.\text{contains}(u); \quad s.\text{insert}(v); \implies s.\text{contains}(u) = x \quad (\text{for } u \ne v)$$
4. **Direct Erase Guarantee:**
   $$s.\text{erase}(v); \implies s.\text{contains}(v) = \text{false}$$
5. **Independent Erase Invariance:**
   $$\text{let } x = s.\text{contains}(u); \quad s.\text{erase}(v); \implies s.\text{contains}(u) = x \quad (\text{for } u \ne v)$$

> [!NOTE]
> **Axiomatic Consistency:** These five axioms form a minimal and complete formal specification. They guarantee that modifying element $v$ leaves the membership state of all distinct elements $u \ne v$ completely unchanged.

---

## 4. Behavioral Nuances: `std::map` Subscripting Side-Effects

A critical design characteristic in C++ `std::map` and `std::unordered_map` is the side-effect of `operator[]`.

```cpp
#include <iostream>
#include <string>
#include <map>

int main() {
    std::map<std::string, int> cart;

    // Initial population:
    cart["soap"] = 2;
    cart["salt"] = 1;
    cart.insert(std::make_pair("pen", 10));
    cart.erase("salt");

    std::cout << "Soap: " << cart["soap"] << "\n"; // Output: 2

    // Accessing a NON-EXISTENT key via operator[]:
    std::cout << "Hat: "  << cart["hat"]  << "\n"; // Output: 0 (Mutates map!)

    // Accessing via .at():
    std::cout << "Hat: "  << cart.at("hat") << "\n"; // Succeeds, prints 0

    return 0;
}
```

### Critical Analysis of `cart["hat"]` vs. `cart.at("hat")`

> [!WARNING]
> **Subscripting Mutates the Data Structure:**
> When `cart["hat"]` is evaluated on a key that does not exist:
> 1. `operator[]` automatically creates and inserts a new entry `{"hat", 0}` with a default-constructed value (`int()` $= 0$).
> 2. It returns a reference to this newly created value.
> 
> Because `cart["hat"]` secretly inserts `"hat"`, the subsequent call `cart.at("hat")` succeeds. If line 16 (`cart["hat"]`) were removed, `cart.at("hat")` would throw `std::out_of_range`.

### Map Lookup Method Behavior Comparison

| Lookup Expression | Key Exists? | Action Performed | Return Value | Container Mutated? |
|---|:---:|---|---|:---:|
| **`map[key]`** | **Yes** | Returns existing mapped value | `Value&` | No |
| **`map[key]`** | **No** | **Inserts `(key, Value())` default node** | `Value&` | **YES (Mutates!)** |
| **`map.at(key)`** | **Yes** | Returns existing mapped value | `Value&` / `const Value&` | No |
| **`map.at(key)`** | **No** | **Throws `std::out_of_range`** | None (Exception) | No |
| **`map.find(key)`** | **Yes** | Returns iterator to key-value node | `iterator` | No |
| **`map.find(key)`** | **No** | Returns end iterator | `map.end()` | No |

---

## 5. Unified Interfaces & Generic Programming

One of the defining design philosophies of the C++ Standard Template Library is the **Unified Interface**.

Even though `std::vector`, `std::list`, `std::set`, and `std::deque` maintain entirely different physical memory structures, they share identical method signatures, type definitions, and traversal semantics.

### Standardized API Vocabulary

| Category | Member Function / Type | Universal Behavior Across Containers |
|---|---|---|
| **Capacity** | `.size()`, `.empty()`, `.max_size()` | Inspect container volume and emptiness |
| **Iterators** | `.begin()`, `.end()`, `.rbegin()`, `.rend()` | Obtain forward and reverse traversal handles |
| **Mutations** | `.insert()`, `.erase()`, `.clear()`, `.swap()` | Uniform insertion, deletion, and buffer reset |
| **Type Aliases** | `::value_type`, `::iterator`, `::size_type` | Meta-programming type extraction |

```cpp
std::vector<int> v = {1, 2, 3};
std::list<int>   l = {1, 2, 3};
std::set<int>    s = {1, 2, 3};

// Identical syntax across completely different memory layouts:
bool e1 = v.empty();
bool e2 = l.empty();
bool e3 = s.empty();
```

---

### Decoupling Algorithms via Iterators

Because of unified interfaces, algorithms in `<algorithm>` are implemented as templates parameterized over iterator categories rather than specific containers:

```cpp
#include <algorithm>
#include <iostream>
#include <vector>
#include <list>

int main() {
    std::vector<int> vec = {10, 20, 30};
    std::list<int>   lst = {10, 20, 30};

    // The identical generic template operates on both:
    auto it1 = std::find(vec.begin(), vec.end(), 20);
    auto it2 = std::find(lst.begin(), lst.end(), 20);

    if (it1 != vec.end()) std::cout << "Found in vector: " << *it1 << "\n";
    if (it2 != lst.end()) std::cout << "Found in list: "   << *it2 << "\n";

    return 0;
}
```

---

### Ease of Refactoring (Container Swapping)

Unified interfaces allow developers to change the underlying container backend via `using` type aliases without altering business logic:

```cpp
// Change container implementation in one place:
// using Container = std::vector<std::string>;
using Container = std::deque<std::string>;

Container items;
items.push_back("Alpha");
items.push_back("Beta");

for (auto it = items.begin(); it != items.end(); ++it) {
    std::cout << *it << "\n";
}
```

---

## 6. Key Takeaways

1. **Memory Informs Performance:** Choose `std::vector` by default for sequential data due to cache locality; switch to node-based or hash structures only when specific access patterns demand them.
2. **ADTs Encapsulate Invariants:** Abstract data types define formal contracts and axioms ($u \ne v$ independence), allowing underlying implementations to change freely.
3. **Be Aware of Map Mutation:** `operator[]` inserts default values upon missing key access. Use `.find()` or `.contains()` for non-mutating lookups, and `.at()` when missing keys represent errors.
4. **Leverage Unified Interfaces:** Write generic client code using iterator ranges (`.begin()`, `.end()`) and STL algorithms to ensure maintainability and seamless refactoring.
