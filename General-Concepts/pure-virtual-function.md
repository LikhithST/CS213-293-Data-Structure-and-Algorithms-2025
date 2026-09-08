# Pure Virtual Functions & Abstract Classes in C++

## Overview

In C++, **pure virtual functions** form the foundation of abstract data types (ADTs), interface design, and dynamic polymorphism. A pure virtual function acts as an explicit contract: it enforces that all non-abstract derived classes **must** provide a concrete implementation for the function.

This guide provides a comprehensive analysis of:
1. **Pure Virtual Syntax & The Pure Specifier (`= 0`):** Language syntax, compiler mechanics, and declaration semantics.
2. **Abstract Base Classes (ABCs):** Instantiation barriers and inheritance rules.
3. **Derived Class Overriding Requirements:** When a derived class becomes concrete vs. remains abstract.
4. **Pure Virtual Functions with Bodies:** Providing optional shared fallback logic while preserving the override mandate.
5. **Pure Virtual Destructors:** Designing abstract base classes with no other pure members.
6. **Interface Simulation in C++:** Building decoupled interfaces without dedicated `interface` keywords.

---

## 1. Syntax Breakdown of Pure Virtual Functions

A pure virtual function is declared within a class definition by appending the **pure specifier** (`= 0`) to a virtual function prototype:

```cpp
virtual double area() const = 0;
```

### Component Breakdown

| Syntax Component | Language Role | Compiler Behavior |
|---|---|---|
| **`virtual`** | Dynamic Dispatch Keyword | Inserts a slot for the function in the class's Virtual Method Table (**vtable**), enabling runtime polymorphism. |
| **`double area() const`** | Member Function Signature | Specifies the return type (`double`), function identifier (`area`), parameter list, and `const` qualifier. |
| **`= 0`** | **Pure Specifier** | Sets the corresponding entry in the class's vtable to null / `__cxa_pure_virtual`, marking the function as having no default body and making the enclosing class **abstract**. |

---

## 2. Abstract Classes & Instantiation Rules

A class containing **at least one pure virtual function** is defined as an **Abstract Base Class (ABC)**.

```cpp
#include <iostream>

// Abstract Base Class
class Shape {
public:
    virtual ~Shape() = default; // Mandatory virtual destructor

    virtual double area() const = 0; // Pure virtual function
    virtual void draw() const = 0;   // Pure virtual function
};
```

### Instantiation Restrictions

The C++ compiler strictly forbids the direct instantiation of an abstract class:

```cpp
int main() {
    // Shape s;            // COMPILE ERROR: Cannot declare variable 's' to be of abstract type 'Shape'
    // auto s = new Shape; // COMPILE ERROR: Cannot allocate an object of abstract type 'Shape'

    Shape* ptr = nullptr;  // OK: Base pointers and references are valid and used for polymorphism
}
```

> [!IMPORTANT]
> **Why Base Pointers are Allowed:** While abstract classes cannot exist as standalone physical objects in memory, abstract pointers (`Shape*`) and references (`Shape&`) are fully permitted because they point to concrete instances of derived classes.

---

## 3. Concrete Derived Classes vs. Abstract Subclasses

When a derived class inherits from an abstract base class, it has two choices:
1. **Implement every inherited pure virtual function** $\implies$ The derived class becomes **concrete** and can be instantiated.
2. **Leave one or more pure virtual functions unimplemented** $\implies$ The derived class **remains abstract** and cannot be instantiated.

```cpp
#include <iostream>

// 1. Concrete Derived Class: Implements ALL pure virtual functions
class Circle : public Shape {
private:
    double radius;

public:
    Circle(double r) : radius(r) {}

    double area() const override {
        return 3.14159265 * radius * radius;
    }

    void draw() const override {
        std::cout << "Drawing Circle of radius " << radius << '\n';
    }
};

// 2. Incomplete Derived Class: Omits draw() -> Remains Abstract!
class SemiShape : public Shape {
public:
    double area() const override {
        return 0.0;
    }
    // draw() is NOT overridden here -> SemiShape is still an abstract class!
};

int main() {
    Circle c(5.0); // OK: Circle is concrete
    c.draw();      // Output: Drawing Circle of radius 5

    // SemiShape s; // COMPILE ERROR: Cannot instantiate abstract class 'SemiShape'
    return 0;
}
```

---

## 4. Subtlety: Pure Virtual Functions with a Body

In standard C++, a pure virtual function **can still have an implementation defined outside the class body**.

### Why Define a Body for a Pure Virtual Function?
- It forces derived classes to explicitly override the method (preventing accidental omission of custom logic).
- It provides a **reusable fallback or common baseline implementation** that derived classes can invoke deliberately.

```cpp
#include <iostream>
#include <string>

class Logger {
public:
    virtual ~Logger() = default;

    // Pure virtual declaration:
    virtual void log(const std::string& message) = 0;
};

// Providing an out-of-line body for the pure virtual function:
void Logger::log(const std::string& message) {
    std::cout << "[Default Log Header]: " << message << '\n';
}

class FileLogger : public Logger {
public:
    void log(const std::string& message) override {
        // Explicitly calling base class pure virtual implementation:
        Logger::log(message);
        std::cout << "Writing '" << message << "' to disk...\n";
    }
};

int main() {
    FileLogger fl;
    fl.log("System startup complete.");
    return 0;
}
```

---

## 5. Pure Virtual Destructors

Sometimes an architectural design requires a class to be abstract, but every member function naturally has a sensible default implementation. 

In such cases, declare the **destructor as pure virtual**:

```cpp
class AbstractBase {
public:
    // Pure virtual destructor makes the class abstract without other pure methods:
    virtual ~AbstractBase() = 0;
};

// Mandatory: Must provide an out-of-line definition for the pure virtual destructor!
AbstractBase::~AbstractBase() {}
```

> [!CAUTION]
> **Pure Virtual Destructors MUST Have a Definition:**
> Unlike normal member functions, derived class destructors always implicitly invoke the base class destructor during object destruction. If the base pure virtual destructor lacks a definition, linking fails with an `undefined reference to AbstractBase::~AbstractBase()` error.

---

## 6. Building Interfaces in Modern C++

Unlike Java or C#, C++ does not feature a dedicated `interface` keyword. Instead, an **interface** is defined as an abstract class that:
1. Contains **only** pure virtual functions (`= 0`).
2. Declares **no non-static member variables** (zero state).
3. Provides a public `virtual ~Interface() = default;` destructor.

```cpp
#include <iostream>
#include <memory>
#include <vector>

// Pure Interface 1
class Drawable {
public:
    virtual ~Drawable() = default;
    virtual void draw() const = 0;
};

// Pure Interface 2
class Serializable {
public:
    virtual ~Serializable() = default;
    virtual void serialize() const = 0;
};

// Concrete Class Implementing Multiple Interfaces
class Document : public Drawable, public Serializable {
public:
    void draw() const override {
        std::cout << "Rendering document view on display.\n";
    }

    void serialize() const override {
        std::cout << "Serializing document data to JSON stream.\n";
    }
};

int main() {
    std::unique_ptr<Document> doc = std::make_unique<Document>();

    // Using via Drawable interface handle:
    Drawable* d = doc.get();
    d->draw();

    // Using via Serializable interface handle:
    Serializable* s = doc.get();
    s->serialize();

    return 0;
}
```

---

## 7. Comparative Taxonomy: Function Types in C++

| Function Type | Syntax Example | Body Required in Class? | Can be Overridden? | Must be Overridden? | Makes Class Abstract? | Resolution Mechanism |
|---|---|:---:|:---:|:---:|:---:|---|
| **Non-Virtual Function** | `void print();` | **Yes** | No (Shadowed, not overridden) | No | No | Static Compile-Time Binding |
| **Virtual Function** | `virtual void print();` | **Yes** | **Yes** | No (Optional) | No | Dynamic Dispatch via `vtable` |
| **Pure Virtual Function** | `virtual void print() = 0;` | **No** (Optional out-of-line) | **Yes** | **Yes** (in concrete subclasses) | **Yes** | Dynamic Dispatch via `vtable` |
| **Pure Virtual Destructor** | `virtual ~Base() = 0;` | **Yes** (Out-of-line required) | **Yes** | Generated automatically | **Yes** | Reverse destructor chain |

---

## 8. Key Takeaways

1. **`= 0` Creates an Abstract Contract:** The pure specifier prevents direct class instantiation and transfers the implementation obligation to derived subclasses.
2. **Base Pointers Enable Polymorphism:** Even though an abstract class cannot be instantiated, `std::unique_ptr<AbstractBase>` and `AbstractBase*` handles provide the foundation for dynamic runtime polymorphism.
3. **Always Include Virtual Destructors:** Abstract base classes and interfaces must always declare a `virtual ~Interface() = default;` to prevent resource leaks when deleting objects polymorphically.
4. **Out-of-Line Bodies are Permitted:** A pure virtual function can supply base fallback logic defined outside the class definition (`void Base::func() { ... }`).
5. **Always Use `override`:** When fulfilling a pure virtual contract in a derived class, mark the method with `override` to let the compiler verify exact signature matching.

