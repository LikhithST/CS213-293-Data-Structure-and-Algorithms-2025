# Lecture 2.3: Pointers, References, and Smart Pointers in C++

## Overview

Memory management is a core pillar of high-performance systems programming and algorithm implementation in C++. Unlike languages with automated runtime garbage collection (such as Java or Python), C++ gives developers direct control over physical memory addresses and object lifetimes.

In this module, we cover:
1. **Raw Pointers and Memory Mechanics:** Address-of (`&`), dereferencing (`*`), pointer arithmetic, and hazardous states (`nullptr`, wild pointers, dangling pointers).
2. **References vs. Pointers:** Syntax, rebindability, and parameter passing semantics.
3. **RAII and Smart Pointers (Exercise 2.6):** Emulating garbage collection deterministically using `std::unique_ptr`, `std::shared_ptr`, `std::weak_ptr`, and the historical caveats of `std::auto_ptr`.
4. **Circular References & Memory Leaks:** Resolving cyclic ownership graphs using `weak_ptr`.
5. **Modern C++ Memory Ownership Guidelines:** Practical decision rules for assigning ownership in data structures.

### Summary Taxonomy of C++ Memory Handles

| Category | Type | Ownership Model | Rebindable? | Nullable? | Lifecycle Management |
|---|---|---|:---:|:---:|---|
| **Raw Pointer** | `T*` | Non-owning / Observer | Yes | Yes (`nullptr`) | Manual (`new`/`delete` or stack address) |
| **Reference** | `T&` | Non-owning alias | No | No | Tied to aliased object's lifetime |
| **Unique Smart Pointer** | `std::unique_ptr<T>` | Exclusive ownership | Yes (via move) | Yes | Automatic destruction upon scope exit |
| **Shared Smart Pointer** | `std::shared_ptr<T>` | Shared ownership | Yes | Yes | Automatic destruction when reference count $= 0$ |
| **Weak Smart Pointer** | `std::weak_ptr<T>` | Non-owning observer | Yes | Yes | Breaks circular dependencies; requires `.lock()` |

---

## 1. Raw Pointers: Memory Addressing

A **pointer** is a variable whose value is the **memory address** of another variable. Instead of holding raw application data, a pointer holds a locator telling the CPU where that data resides in RAM.

### Core Operators
- **Address-of Operator (`&`):** Retrieves the physical memory address of an existing variable.
- **Dereference Operator (`*`):** Accesses or modifies the value stored at the memory address pointed to by the pointer.

```cpp
#include <iostream>

int main() {
    int score = 42;
    int* ptr = &score; // ptr holds the memory address of score

    std::cout << "Value of score:          " << score << '\n';  // 42
    std::cout << "Address of score (&):    " << &score << '\n'; // e.g., 0x7ffd5e2a
    std::cout << "Value stored in ptr:     " << ptr << '\n';   // e.g., 0x7ffd5e2a
    std::cout << "Dereferenced value (*):  " << *ptr << '\n';  // 42

    // Modifying target value through the pointer:
    *ptr = 99;
    std::cout << "New score value:         " << score << '\n';  // 99

    return 0;
}
```

```text
Memory Address       Variable Name       Stored Value
-----------------------------------------------------------------
0x7ffd5e2a           score               99
0x7ffd5e30           ptr                 0x7ffd5e2a  ----(points to score)
```

---

### Passing by Pointer in Functions
By default, C++ passes arguments **by value** (creating a copy). Passing a pointer enables functions to mutate caller variables directly without copying large structs:

```cpp
void applyDamage(int* hp, int damage) {
    if (hp != nullptr) {
        *hp -= damage; // Directly mutates caller's memory
    }
}

int main() {
    int playerHp = 100;
    applyDamage(&playerHp, 25);
    std::cout << "Remaining HP: " << playerHp << '\n'; // 75
}
```

---

### Pointer States & Safety

| Pointer State | Definition / Origin | Risk Level | Behavior on Dereference | Safe Practice |
|---|---|:---:|---|---|
| **`nullptr`** | Explicit zero address initialization | **Safe** | Predictable crash / caught by `if (p != nullptr)` | Always initialize unused pointers to `nullptr` |
| **Wild Pointer** | Uninitialized pointer containing garbage bits | **Critical** | Undefined behavior / random memory corruption | Never declare pointers without immediate assignment |
| **Dangling Pointer** | Points to deallocated or out-of-scope memory | **Critical** | Use-after-free vulnerability / segmentation fault | Clear pointers to `nullptr` after `delete`; use smart pointers |

#### Examples of Pointer Pitfalls

```cpp
// 1. nullptr (Safe if checked):
int* ptr = nullptr;
if (ptr != nullptr) {
    *ptr = 10;
}

// 2. Wild pointer (DANGEROUS):
int* wild; // Holds random bits -> *wild = 10 causes undefined behavior!

// 3. Dangling pointer (DANGEROUS):
int* getDanglingPointer() {
    int localVal = 42;
    return &localVal; // BUG: localVal destroyed when function returns!
}
```

---

### Pointers and Arrays
In C++, an array name automatically decays into a pointer pointing to its initial element ($A \equiv \&A[0]$). Pointer arithmetic advances addresses by multiples of `sizeof(T)`:

$$*(p + i) \equiv p[i]$$

```cpp
int numbers[] = {10, 20, 30};
int* p = numbers; // Points to &numbers[0]

std::cout << *p;       // 10
std::cout << *(p + 1); // 20 (Advances by sizeof(int) = 4 bytes)
std::cout << *(p + 2); // 30
```

---

## 2. References vs. Pointers

A **reference** (`T&`) is an alias (an alternative name) for an existing variable. Once initialized, it binds permanently to its target object.

```cpp
#include <iostream>

int main() {
    int original = 42;
    int& ref = original; // ref is an immutable alias for original

    ref = 99; // Mutates original directly
    std::cout << "Original: " << original << '\n'; // 99
    std::cout << "Address:  " << &original << " == " << &ref << '\n'; // Identical addresses

    return 0;
}
```

### Detailed Comparison

| Dimension | Pointer (`T*`) | Reference (`T&`) |
|---|---|---|
| **Rebindability** | Can be reassigned to point to different memory addresses | Permanently bound to the initial object upon declaration |
| **Nullability** | Can be `nullptr` (can represent "no object") | Cannot be null; must always alias a valid object |
| **Syntax** | Requires explicit `*` (dereference) and `&` (address-of) | Clean, transparent variable syntax |
| **Storage** | Has its own memory address and consumes 8 bytes (on 64-bit) | Typically compiled away; operates as direct alias |
| **Arithmetic** | Supports arithmetic (`ptr++`, `ptr + i`) | Does not support reference arithmetic |

---

## 3. Smart Pointers & RAII (Exercise 2.6)

In standard C++, there is no background garbage collection thread. Instead, C++ utilizes **RAII (Resource Acquisition Is Initialization)**: resources are acquired in constructors and automatically released in destructors when objects go out of scope.

Smart pointers, defined in `<memory>`, wrap raw pointers and manage heap memory deterministically.

---

### 1. `std::unique_ptr` (Exclusive Ownership)

- **Ownership Model:** Strictly exclusive. Exactly **one** `unique_ptr` owns the heap resource at any time.
- **Copy Semantics:** **Non-copyable** (copy constructor and copy assignment are deleted).
- **Move Semantics:** Ownership is transferred explicitly using `std::move`.
- **Overhead:** Zero runtime cost (identical size and speed to a raw pointer).
- **Factory Function:** `std::make_unique<T>(...)` (C++14).

```cpp
#include <iostream>
#include <memory>

struct Node {
    int value;
    Node(int v) : value(v) { std::cout << "Node(" << value << ") created\n"; }
    ~Node() { std::cout << "Node(" << value << ") destroyed\n"; }
};

int main() {
    // 1. Create unique_ptr
    std::unique_ptr<Node> u1 = std::make_unique<Node>(10);

    // std::unique_ptr<Node> u2 = u1; // COMPILE ERROR: Cannot copy unique_ptr!

    // 2. Transfer ownership via move:
    std::unique_ptr<Node> u2 = std::move(u1); // u1 is now nullptr, u2 owns Node(10)

    if (!u1) {
        std::cout << "u1 is now empty (nullptr)\n";
    }
    std::cout << "u2 value: " << u2->value << '\n';

    return 0;
} // u2 goes out of scope here -> Node(10) deleted automatically!
```

---

### 2. `std::shared_ptr` (Shared Ownership via Reference Counting)

- **Ownership Model:** Shared among multiple owners.
- **Mechanism:** Maintains a heap-allocated **Control Block** containing a strong reference counter (`use_count`) and a weak counter.
- **Copy Semantics:** Copying increments `use_count`. When any `shared_ptr` is destroyed, `use_count` decrements.
- **Deallocation Rule:** The underlying object is deleted when `use_count` reaches **$0$**.
- **Factory Function:** `std::make_shared<T>(...)` (allocates object and control block in a single contiguous memory chunk).

```text
       shared_ptr p1 ----\
                          +---> [ Control Block: use_count = 2, weak_count = 0 ] ---> [ int: 100 ]
       shared_ptr p2 ----/
```

```cpp
#include <iostream>
#include <memory>

int main() {
    std::shared_ptr<int> p1 = std::make_shared<int>(100);
    std::cout << "Initial use_count: " << p1.use_count() << '\n'; // 1

    {
        std::shared_ptr<int> p2 = p1; // Copy: count becomes 2
        std::cout << "Inside block use_count: " << p1.use_count() << '\n'; // 2
        std::cout << "p2 value: " << *p2 << '\n';
    } // p2 goes out of scope -> use_count decrements to 1

    std::cout << "Outside block use_count: " << p1.use_count() << '\n'; // 1

    return 0;
} // p1 goes out of scope -> use_count hits 0 -> int(100) deleted!
```

---

### 3. `std::weak_ptr` (Non-Owning Observer)

- **Ownership Model:** Non-owning observer referencing an object managed by a `std::shared_ptr`.
- **Mechanism:** Observes the resource without incrementing `use_count`.
- **Access Rule:** Cannot be dereferenced directly. Must call `.lock()` to obtain a temporary `std::shared_ptr`, verifying the resource is still alive.

```cpp
#include <iostream>
#include <memory>

int main() {
    std::weak_ptr<int> weak;

    {
        std::shared_ptr<int> shared = std::make_shared<int>(42);
        weak = shared; // weak observes shared, use_count remains 1

        std::cout << "use_count: " << shared.use_count() << '\n'; // 1

        // Accessing object via .lock():
        if (auto locked = weak.lock()) {
            std::cout << "Observed value: " << *locked << '\n'; // 42
        }
    } // shared destroyed here -> int(42) deleted!

    // Attempting to access after object destruction:
    if (auto locked = weak.lock()) {
        std::cout << "Object still alive: " << *locked << '\n';
    } else {
        std::cout << "Object has expired and was freed!\n"; // Executes
    }

    return 0;
}
```

---

### 4. Circular References: Why `weak_ptr` is Essential

When two objects hold `std::shared_ptr` references to each other, a **cyclic reference** is formed. Neither object's `use_count` can ever reach $0$, producing a permanent memory leak.

```text
Circular Leak:
[ Node A (use_count=1) ] === shared_ptr ===> [ Node B (use_count=1) ]
[ Node A (use_count=1) ] <=== shared_ptr === [ Node B (use_count=1) ]
(Neither count reaches 0 -> Memory Leak!)

Cycle Broken with weak_ptr:
[ Node A (use_count=1) ] === shared_ptr ===> [ Node B (use_count=1) ]
[ Node A (use_count=1) ] <... weak_ptr ..... [ Node B (use_count=1) ]
(weak_ptr does not increment count -> Clean Destruction!)
```

```cpp
#include <iostream>
#include <memory>

struct B; // Forward declaration

struct A {
    std::shared_ptr<B> ptrB;
    ~A() { std::cout << "A destroyed\n"; }
};

struct B {
    std::weak_ptr<A> ptrA; // Using weak_ptr breaks the circular ownership cycle!
    ~B() { std::cout << "B destroyed\n"; }
};

int main() {
    auto a = std::make_shared<A>();
    auto b = std::make_shared<B>();

    a->ptrB = b;
    b->ptrA = a; // Non-owning reference: avoids cycle leak!

    return 0;
} // Both A and B are destroyed cleanly!
```

---

### 5. `std::auto_ptr` (Historical Flaw & Removal)

In C++98, `std::auto_ptr` was the initial attempt at an exclusive-ownership smart pointer. However, because C++98 lacked move semantics (`std::move`), `std::auto_ptr` implemented ownership transfer inside its **copy constructor**:

```cpp
// Deprecated in C++11, completely removed in C++17
std::auto_ptr<int> p1(new int(50));
std::auto_ptr<int> p2 = p1; // DANGEROUS: p1 silently became nullptr!

// Accessing p1 now causes a runtime crash:
// *p1 = 20; // Segmentation Fault!
```

> [!CAUTION]
> **Why `std::auto_ptr` was Removed:**
> 1. Silent destructive copying made passing `auto_ptr` by value into functions or standard algorithms dangerous.
> 2. `std::auto_ptr` could not be stored in STL containers (such as `std::vector`), because sorting or copying the vector would destroy elements.
> 3. Completely replaced by `std::unique_ptr`, which makes ownership transfers explicit via `std::move`.

---

## 4. Summary Matrix: Smart Pointers in C++

| Smart Pointer | Standard | Ownership Model | Copyable? | Movable? | Control Block Overhead | Typical Applications |
|---|:---:|---|:---:|:---:|:---:|---|
| **`std::unique_ptr`** | C++11 | Exclusive (single owner) | **No** | **Yes** | Zero (size of raw pointer) | Factory methods, local heap objects, tree nodes |
| **`std::shared_ptr`** | C++11 | Shared (reference counted) | **Yes** | **Yes** | 2 pointers + heap control block | Shared graphs, multithreaded resource pools |
| **`std::weak_ptr`** | C++11 | Non-owning observer | **Yes** | **Yes** | References shared control block | Cache systems, breaking circular references |
| **`std::auto_ptr`** | C++98 | Destructive copy (flawed) | Broken | N/A | Zero | **Removed in C++17; do not use** |

---

## 5. Modern C++ Memory Best Practices

1. **Default to `std::unique_ptr`:** Use `std::unique_ptr` for all dynamically allocated heap resources. It incurs zero runtime cost and provides clear exclusive ownership.
2. **Use `std::shared_ptr` Only When Necessary:** Reserve `std::shared_ptr` for scenarios where multiple independent entities genuinely share ownership of an object's lifetime.
3. **Break Cycles with `std::weak_ptr`:** In graph, tree-parent, or observer patterns, use `std::weak_ptr` for back-references to prevent cyclic memory leaks.
4. **Use Raw Pointers/References for Non-Owning Parameters:** When passing objects into functions that only inspect or use data without affecting object lifetime, pass by `const T&` or raw non-owning `T*`.
5. **Never Use `auto_ptr` or Raw `new`/`delete`:** Rely on `std::make_unique` and `std::make_shared` to eliminate explicit `new`/`delete` calls.
