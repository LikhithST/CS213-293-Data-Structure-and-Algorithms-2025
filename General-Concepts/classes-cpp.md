# Object-Oriented Programming (OOP) in C++

## Overview

C++ is a multi-paradigm systems programming language that provides comprehensive support for **Object-Oriented Programming (OOP)** with direct memory control and zero-cost abstractions. 

OOP structures software around **objects**—data structures that bundle state (member variables) together with behavior (member functions/methods).

This guide explores the four fundamental pillars of OOP and their concrete mechanics in C++:
1. **Classes & Encapsulation:** Data hiding, access specifiers (`private`, `protected`, `public`), and object state protection.
2. **Object Lifecycle Management:** Constructors (default, parameterized, copy, move), destructors, and RAII.
3. **Inheritance & Reusability:** Deriving classes, inheritance access modes, and hierarchical models.
4. **Polymorphism (Static vs. Dynamic):** Function overloading, operator overloading, `virtual` functions, and runtime dynamic dispatch.
5. **Abstraction & Interfaces:** Separating interface contracts from concrete physical implementations using abstract classes.

### The Four Pillars of OOP in C++

| Pillar | Core Architectural Purpose | C++ Mechanism / Keywords | Primary Benefit |
|---|---|---|---|
| **Encapsulation** | Bundles data and methods; restricts direct external mutation | `class`, `private`, `protected`, `public` | Data integrity, modularity, maintainability |
| **Abstraction** | Hides implementation complexity behind clean interfaces | Abstract classes, Pure virtual functions (`= 0`) | Decoupled architecture, simplified client code |
| **Inheritance** | Establishes hierarchical "is-a" relationships; code reuse | `: public / protected / private Base` | Eliminates redundancy, structural polymorphism |
| **Polymorphism** | Provides a unified interface for entity-specific behaviors | Function overloading, templates, `virtual`, `override` | Extensibility, flexible dynamic runtime dispatch |

---

## 1. Classes, Objects & Access Control

A **class** is a user-defined blueprint or type specification; an **object** is a concrete instance of that class allocated in memory (on the stack, heap, or data segment).

```cpp
#include <iostream>
#include <string>

class Car {
private:
    std::string brand;
    int speed;

public:
    // Parameterized constructor
    Car(const std::string& b, int s) : brand(b), speed(s) {}

    void accelerate(int delta) {
        if (delta > 0) {
            speed += delta;
        }
    }

    void displayStatus() const {
        std::cout << "Brand: " << brand << ", Speed: " << speed << " km/h\n";
    }
};

int main() {
    Car myCar("Toyota", 60); // Instantiation of Car object on the stack
    myCar.accelerate(20);
    myCar.displayStatus();   // Output: Brand: Toyota, Speed: 80 km/h
    return 0;
}
```

---

### Access Specifiers & Data Hiding

Access specifiers enforce encapsulation boundaries by controlling which scopes can access specific members:

| Access Specifier | Accessible Inside Own Class? | Accessible Inside Derived Classes? | Accessible Outside (Client Code)? | Typical Usage |
|---|:---:|:---:|:---:|---|
| **`private`** (default for `class`) | **Yes** | **No** | **No** | Internal state variables, private helper methods |
| **`protected`** | **Yes** | **Yes** | **No** | Base properties customized by derived subclasses |
| **`public`** (default for `struct`) | **Yes** | **Yes** | **Yes** | Public interface, getters, setters, API methods |

> [!NOTE]
> **`struct` vs. `class` in C++:** In C++, the **only** language difference between a `struct` and a `class` is the default access level: `struct` members and base classes default to `public`, whereas `class` members and base classes default to `private`.

---

## 2. Object Lifecycle: Constructors, Destructors & RAII

C++ does not rely on non-deterministic garbage collectors. Object creation and teardown are deterministically managed through constructors and destructors.

```cpp
#include <iostream>

class DynamicArray {
private:
    int* data;
    size_t size;

public:
    // 1. Parameterized Constructor (Resource Acquisition)
    DynamicArray(size_t s) : size(s), data(new int[s]()) {
        std::cout << "Allocated " << size << " integers on the heap.\n";
    }

    // 2. Destructor (Resource Release / RAII)
    ~DynamicArray() {
        delete[] data;
        std::cout << "Deallocated heap array cleanly.\n";
    }

    // Disable unsafe shallow copying (Rule of Three):
    DynamicArray(const DynamicArray&) = delete;
    DynamicArray& operator=(const DynamicArray&) = delete;
};
```

---

### Special Member Functions (The Rule of 0 / 3 / 5)

When a class directly manages raw system resources (heap memory, file handles, sockets), it must explicitly define the resource management lifecycle:

| Special Member Function | Syntax Signature | Role |
|---|---|---|
| **Default Constructor** | `T()` | Initializes an object with default state |
| **Parameterized Constructor** | `T(args...)` | Initializes object state with user arguments |
| **Destructor** | `~T()` | Cleans up resources when the object exits scope |
| **Copy Constructor** | `T(const T& other)` | Constructs a new deep copy of an existing object |
| **Copy Assignment Operator** | `T& operator=(const T& other)` | Copies state from an existing object, replacing current state |
| **Move Constructor** | `T(T&& other) noexcept` | Transfers ownership of resources from a temporary rvalue |
| **Move Assignment Operator** | `T& operator=(T&& other) noexcept` | Releases current resource and steals ownership from temporary |

---

## 3. Inheritance & Class Hierarchies

Inheritance allows a derived (child) class to inherit member variables and methods from a base (parent) class, establishing an "is-a" structural relationship.

```cpp
#include <iostream>
#include <string>

// Base Class
class Animal {
protected:
    std::string name;

public:
    Animal(const std::string& n) : name(n) {}

    void eat() const {
        std::cout << name << " is eating...\n";
    }
};

// Derived Class
class Dog : public Animal {
public:
    Dog(const std::string& n) : Animal(n) {} // Forwarding to base constructor

    void bark() const {
        std::cout << name << " says: Woof! Woof!\n";
    }
};

int main() {
    Dog d("Buddy");
    d.eat();  // Inherited from Animal base
    d.bark(); // Defined directly in Dog
    return 0;
}
```

---

### Inheritance Access Modes

When inheriting from a base class (`class Derived : <mode> Base`), the access mode determines the maximum accessibility of inherited members in the derived class:

| Base Member Visibility | `public` Inheritance | `protected` Inheritance | `private` Inheritance |
|---|---|---|---|
| **`public`** in Base | Stays **`public`** in Derived | Becomes **`protected`** in Derived | Becomes **`private`** in Derived |
| **`protected`** in Base | Stays **`protected`** in Derived | Stays **`protected`** in Derived | Becomes **`private`** in Derived |
| **`private`** in Base | **Inaccessible** (hidden) | **Inaccessible** (hidden) | **Inaccessible** (hidden) |

- **`public` inheritance:** Models a true subtyping relationship ("is-a").
- **`private` / `protected` inheritance:** Models implementation reuse ("implemented-in-terms-of"), hiding the base class API from client code.

---

### Inheritance Topologies in C++

| Topology | Description | Example Structure |
|---|---|---|
| **Single Inheritance** | Derived class inherits from exactly one base class | `Dog` $\to$ `Animal` |
| **Multiple Inheritance** | Derived class inherits from two or more base classes | `Smartphone` $\to$ `Phone`, `Camera` |
| **Multilevel Inheritance** | A derived class acts as a base class for another class | `SportsCar` $\to$ `Car` $\to$ `Vehicle` |
| **Hierarchical Inheritance** | Multiple derived classes branch from a single base class | `Cat` $\to$ `Animal`, `Dog` $\to$ `Animal` |
| **Hybrid (Diamond) Inheritance** | Combination of multiple and hierarchical inheritance (resolvable via `virtual` inheritance) | `ScannerPrinter` $\to$ `Scanner`, `Printer` $\to$ `Device` |

---

## 4. Polymorphism: Compile-Time vs. Run-Time

Polymorphism ("many forms") enables the same interface to exhibit different behaviors depending on the concrete types involved.

---

### 1. Compile-Time Polymorphism (Static Dispatch)

Resolved entirely by the compiler during compilation based on static types and arguments. Incurs **zero runtime overhead**.

#### A. Function Overloading
Multiple functions in the same scope sharing the identical name with distinct parameter lists:
```cpp
class Printer {
public:
    void show(int val)               { std::cout << "Integer: " << val << '\n'; }
    void show(double val)            { std::cout << "Double:  " << val << '\n'; }
    void show(const std::string& val){ std::cout << "String:  " << val << '\n'; }
};
```

#### B. Operator Overloading
Customizing standard C++ operators (`+`, `-`, `<<`, `==`) for user-defined classes:
```cpp
struct Point {
    int x, y;
    Point operator+(const Point& rhs) const {
        return Point{x + rhs.x, y + rhs.y};
    }
};
```

---

### 2. Run-Time Polymorphism (Dynamic Dispatch via `virtual`)

Resolved at runtime when invoking methods through base class pointers (`Base*`) or base class references (`Base&`).

```cpp
#include <iostream>
#include <memory>

class Base {
public:
    // Virtual destructor is mandatory when deleting derived objects via Base*
    virtual ~Base() = default;

    virtual void speak() const {
        std::cout << "Base entity speaks.\n";
    }
};

class Derived : public Base {
public:
    void speak() const override { // Explicit override specifier
        std::cout << "Derived entity speaks specialized sound.\n";
    }
};

int main() {
    std::unique_ptr<Base> obj = std::make_unique<Derived>();
    obj->speak(); // Output: "Derived entity speaks specialized sound." (Dynamic Dispatch)
    return 0;
}
```

---

### Static vs. Dynamic Polymorphism Comparison

| Dimension | Compile-Time (Static) Polymorphism | Run-Time (Dynamic) Polymorphism |
|---|---|---|
| **Mechanism** | Function / Operator Overloading, Templates | Virtual Functions, `override`, Virtual Table (`vtable`) |
| **Resolution Time** | Compile time (Compiler chooses exact function call) | Run time (Lookup via `vptr` pointer indirection) |
| **Execution Speed** | **Optimal** (Direct calls, eligible for inline expansion) | Minor overhead ($1$ pointer dereference in `vtable`) |
| **Memory Overhead** | Zero per-object memory overhead | $8$ bytes per object on 64-bit for the `vptr` table pointer |
| **Flexibility** | Types must be known at compile time | Decoupled; can load dynamic subclasses at runtime |

---

## 5. Abstraction & Abstract Base Classes

Abstraction separates **what** an entity does (its interface) from **how** it performs it (its implementation).

A class containing at least one **pure virtual function** (`virtual void f() = 0`) is an **abstract base class** and cannot be directly instantiated:

```cpp
#include <iostream>
#include <vector>
#include <memory>

// Abstract Base Class (Interface Contract)
class Shape {
public:
    virtual ~Shape() = default;

    // Pure virtual functions:
    virtual double area() const = 0;
    virtual void draw() const = 0;
};

// Concrete Derived Class 1
class Circle : public Shape {
private:
    double radius;
public:
    Circle(double r) : radius(r) {}
    double area() const override { return 3.14159265 * radius * radius; }
    void draw() const override   { std::cout << "Drawing Circle(r=" << radius << ")\n"; }
};

// Concrete Derived Class 2
class Rectangle : public Shape {
private:
    double width, height;
public:
    Rectangle(double w, double h) : width(w), height(h) {}
    double area() const override { return width * height; }
    void draw() const override   { std::cout << "Drawing Rectangle(" << width << "x" << height << ")\n"; }
};

int main() {
    // Shape s; // COMPILE ERROR: Cannot instantiate abstract class Shape!

    std::vector<std::unique_ptr<Shape>> shapes;
    shapes.push_back(std::make_unique<Circle>(5.0));
    shapes.push_back(std::make_unique<Rectangle>(4.0, 6.0));

    for (const auto& shape : shapes) {
        shape->draw();
        std::cout << "Area: " << shape->area() << "\n\n";
    }
    return 0;
}
```

---

## 6. Key Takeaways & Best Practices

1. **Always Declare Virtual Destructors in Base Classes:** Any class with virtual functions must have `virtual ~Base() = default;` to prevent undefined behavior and memory leaks when deleting derived objects through base pointers.
2. **Use the `override` Keyword:** Always mark overridden virtual methods in derived classes with `override` so the compiler catches mismatched parameter types or missing `const` qualifiers at compile time.
3. **Prefer Composition Over Inheritance:** Use public inheritance only for true polymorphic "is-a" relationships. For code reuse without subtyping, prefer member composition ("has-a").
4. **Encapsulate Invariants:** Keep member variables `private` by default. Provide `const`-correct getters and validated mutating methods to preserve internal object invariants.
5. **Manage Resources with Smart Pointers:** Store polymorphic hierarchies in standard containers using `std::vector<std::unique_ptr<Base>>` to guarantee automatic, exception-safe deallocation.

