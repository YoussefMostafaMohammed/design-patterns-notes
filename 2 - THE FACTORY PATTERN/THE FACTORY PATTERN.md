# Factory Patterns

## 📋 Table of Contents

- [Why Design Patterns Matter](#why-design-patterns-matter)
- [The 5 Core Problems That Led to Factory Patterns](#the-5-core-problems-that-led-to-factory-patterns)
- [Factory Patterns Overview](#factory-patterns-overview)
- [Simple Factory Pattern](#simple-factory-pattern)
- [Factory Method Pattern](#factory-method-pattern)
- [Abstract Factory Pattern](#abstract-factory-pattern)
- [Detailed Comparison: Factory Method vs Abstract Factory](#detailed-comparison-factory-method-vs-abstract-factory)
- [Additional Real-World Examples](#additional-real-world-examples)
- [Best Practices](#best-practices)
- [Modern C++ Enhancements](#modern-c-enhancements)
- [Testing with Factory Patterns](#testing-with-factory-patterns)
- [Summary Flowchart](#summary-flowchart)
- [How to Compile and Run](#how-to-compile-and-run)
- [Key Takeaways](#key-takeaways)

---

## Why Design Patterns Matter

Design patterns are not theoretical concepts—they are battle-tested solutions to recurring problems that developers face in real-world software development. The Factory patterns emerged because object creation became too complex, too duplicated, too coupled, and too dynamic to manage with simple `new` statements.

---

## The 5 Core Problems That Led to Factory Patterns

| Problem | Why It's Bad | How Factory Solves It |
|---------|--------------|------------------------|
| **Tight coupling to concrete classes** | Code depends on specific classes, breaks when they change | Factory hides concrete types behind interfaces |
| **Complex creation logic everywhere** | Repetition of parameter validation, config reading, resource allocation | Centralizes creation logic in one place |
| **Need runtime object selection** | Sprawling switch/if-else blocks throughout codebase | Factory decides dynamically based on input |
| **Changing implementations** | Requires editing many files to swap implementations | Swap implementation in factory only |
| **Testing difficulty** | Cannot mock or inject fake objects | Factory enables dependency injection for tests |

**Bottom line:** Object creation became too complicated, too duplicated, too coupled, and too dynamic. Factories make creation central, flexible, decoupled, configurable, and test-friendly.

---

## Factory Patterns Overview

### Feature Comparison Table

| Pattern | Use Case | Structure | Key Benefit | Flexibility |
|---------|----------|-----------|-------------|-------------|
| **Simple Factory** | Single product, simple logic | Single class with `create()` method | Hides implementation | Low |
| **Factory Method** | Subclasses decide creation | Class hierarchy with virtual method | Polymorphic creation | Medium |
| **Abstract Factory** | **Families** of related products | **Parallel hierarchies** | **Guarantees compatibility** | **High** |

### Decision Flowchart

```
Do you need to create objects?
    |
    ▼
Are the objects **related** (must work together)?
    |
    ├─ YES → Do you have **multiple families**?
    |        |
    |        ├─ YES → Use **Abstract Factory**
    |        |
    |        └─ NO → Use **Factory Method**
    |
    └─ NO → Do you need **polymorphic creation**?
            |
            ├─ YES → Use **Factory Method**
            |
            └─ NO → Use **Simple Factory / Factory Function**
```

---

## Simple Factory Pattern

**Note:** Simple Factory is not a true Gang of Four design pattern, but a common programming idiom.

### What is Simple Factory?

A Simple Factory uses a **single class** with a `create()` method that decides which object to instantiate based on parameters. It has no polymorphic factory hierarchy.

### Implementation (Factory Function)

```cpp
// Public interface
class Foo {
public:
    virtual ~Foo() = default;
    static std::unique_ptr<Foo> create(); // Factory function
    virtual void bar() = 0;
protected:
    Foo() = default; // Protected constructor
};

// Hidden implementation
class FooImpl : public Foo {
public:
    void bar() override { /* implementation */ }
};

std::unique_ptr<Foo> Foo::create() {
    return std::make_unique<FooImpl>();
}

// Client usage
auto fooInstance = Foo::create();
fooInstance->bar();
```

### When to Use

- Single product type with simple creation logic
- No need for extensibility or multiple factory variants
- Goal is to hide implementation details (pImpl idiom)
- Object creation is based on simple parameters or configuration

---

## Factory Method Pattern

### What is Factory Method?

The **Factory Method** defines an interface for creating a single object, but lets **subclasses decide which class to instantiate**. It uses polymorphism: a base class declares a factory method, and subclasses override it to create concrete objects.

**Key distinction:** Creates **one product**, but subclasses decide the exact type at runtime.

### Second Car Factory Simulation (Factory Method)

#### The Problem

- You want to request a car without specifying the exact type
- The **factory itself decides** what to create (e.g., least busy factory, random, config-based)
- Need **virtual constructors**

#### Solution Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                          CLIENT CODE                                │
│  "I want A car, I don't care which brand"                         │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────┼──────────────────────────────────────┐
│       FACTORY HIERARCHY      │                                      │
│                              ▼                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    CarFactory (Abstract)                     │  │
│  │  + requestCar() : unique_ptr<ICar>                           │  │
│  │  - createCar() : unique_ptr<ICar> [virtual]                  │  │
│  │  + getNumberOfCarsProduced() : unsigned                      │  │
│  └────────────────────────┬───────────────────────────────────────┘  │
│                           │                                         │
│        ┌──────────────────┼──────────────────┐                      │
│        ▼                  ▼                  ▼                      │
│  ┌─────────────┐   ┌──────────────┐   ┌──────────────┐            │
│  │ FordFactory │   │ToyotaFactory │   │ LeastBusy    │            │
│  │             │   │              │   │   Factory    │            │
│  └──────┬──────┘   └──────┬───────┘   └──────┬───────┘            │
│         │                 │                    │                    │
│         │  creates        │  creates           │  delegates         │
│         ▼                 ▼                    ▼                    │
└─────────┼─────────────────┼────────────────────┼────────────────────┘
          │                 │                    │
          ▼                 ▼                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  CONCRETE PRODUCTS (Cars)                           │
│                                                                     │
│  Ford (Car)               Toyota (Car)               ICar (any)   │
└─────────────────────────────────────────────────────────────────────┘
```

#### Complete C++ Implementation (Factory Method)

```cpp
// ---------- PRODUCT HIERARCHY ----------
class ICar {
public:
    virtual ~ICar() = default;
    virtual std::string info() const = 0;
};

class Ford : public ICar {
public:
    std::string info() const override { return "Ford"; }
};

class Toyota : public ICar {
public:
    std::string info() const override { return "Toyota"; }
};

// ---------- FACTORY HIERARCHY ----------
class CarFactory {
public:
    virtual ~CarFactory() = default;
    
    std::unique_ptr<ICar> requestCar() {
        ++m_numberOfCarsProduced;
        return createCar();  // Virtual constructor idiom (NVI)
    }
    
    unsigned getNumberOfCarsProduced() const { 
        return m_numberOfCarsProduced; 
    }
    
private:
    virtual std::unique_ptr<ICar> createCar() = 0;
    unsigned m_numberOfCarsProduced { 0 };
};

// ---------- CONCRETE FACTORIES ----------
class FordFactory final : public CarFactory {
private:
    std::unique_ptr<ICar> createCar() override {
        return std::make_unique<Ford>();
    }
};

class ToyotaFactory final : public CarFactory {
private:
    std::unique_ptr<ICar> createCar() override {
        return std::make_unique<Toyota>();
    }
};

// ---------- ADVANCED: LEAST BUSY FACTORY ----------
class LeastBusyFactory final : public CarFactory {
public:
    explicit LeastBusyFactory(std::vector<std::unique_ptr<CarFactory>> factories)
        : m_factories { std::move(factories) } {
        if (m_factories.empty()) {
            throw std::runtime_error { "No factories provided." };
        }
    }
    
private:
    std::unique_ptr<ICar> createCar() override {
        auto leastBusy = std::min_element(m_factories.begin(), m_factories.end(),
            [](const auto& f1, const auto& f2) {
                return f1->getNumberOfCarsProduced() < 
                       f2->getNumberOfCarsProduced();
            });
        return (*leastBusy)->requestCar();
    }
    
    std::vector<std::unique_ptr<CarFactory>> m_factories;
};

// ---------- CLIENT CODE ----------
int main() {
    // Simple usage
    ToyotaFactory myFactory;
    auto myCar = myFactory.requestCar();
    std::cout << myCar->info() << std::endl;  // Toyota
    
    // Advanced: Least busy factory
    std::vector<std::unique_ptr<CarFactory>> factories;
    factories.push_back(std::make_unique<FordFactory>());
    factories.push_back(std::make_unique<FordFactory>());
    factories.push_back(std::make_unique<ToyotaFactory>());
    
    // Pre-order some cars
    for (size_t i : {0, 0, 0, 1, 1, 2}) {
        factories[i]->requestCar();
    }
    
    LeastBusyFactory leastBusyFactory { std::move(factories) };
    
    // Build 10 cars from least busy factory
    for (unsigned i { 0 }; i < 10; ++i) {
        auto theCar = leastBusyFactory.requestCar();
        std::cout << theCar->info() << std::endl;
    }
    // Output: Toyota, Ford, Toyota, Ford, Ford, Toyota, Ford, Ford, Ford, Toyota
}
```

---

## Abstract Factory Pattern

### What is Abstract Factory?

The **Abstract Factory** is a creational design pattern that provides an interface for creating **families** of related or dependent objects without specifying their concrete classes. Think of it as a "factory of factories" or a super-factory that creates other factories.

**Key distinction:** While a Simple Factory creates **one type** of object, Abstract Factory creates **complete families** of objects that must work together.

### Key Concepts

1. **Abstract Products**: Interfaces defining the contract for each product type (e.g., `ICar`, `IEngine`)
2. **Concrete Products**: Implementation classes belonging to specific families (e.g., `FordCar`, `ToyotaEngine`)
3. **Abstract Factory**: Interface declaring creation methods for each product in the family
4. **Concrete Factories**: Implementation classes that create concrete products from one family
5. **Client**: Uses only abstract interfaces, never concrete classes

### When to Use Abstract Factory

**Use when:**
- System needs to be independent of how its products are created
- System needs to work with multiple families of related products
- Products from one family must be used together (cannot mix brands)
- You want to provide a library of products without exposing implementation

**Don't use when:**
- You only have one product type
- Products are not related or don't need to work together
- Object creation logic is trivial and doesn't need abstraction
- You need to add new product types frequently (requires changing the abstract factory interface)

### Car Factory Simulation (Abstract Factory)

#### The Problem

You need to create cars from different manufacturers (Ford, Toyota), and each manufacturer makes multiple car types (Sedan, SUV). You want to ensure that:
- A Ford factory only creates Ford cars
- A Toyota factory only creates Toyota cars
- The client code doesn't need to know the specific car type or brand

#### Solution Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                             CLIENT CODE                             │
│  "I need a family of cars, but I don't care about the brand"      │
└─────────────────────────────────────┬───────────────────────────────┘
                                      │
                                      ▼
┌─────────────────────────────────────┼───────────────────────────────┐
│        ABSTRACT FACTORY INTERFACE   │                               │
│                                     ▼                               │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    ICarFactory (Abstract)                    │  │
│  │  + makeSuv()   : unique_ptr<ICar>                            │  │
│  │  + makeSedan() : unique_ptr<ICar>                            │  │
│  └────────────────────────┬───────────────────────────────────────┘  │
│                           │                                         │
│        ┌──────────────────┼──────────────────┐                      │
│        ▼                  ▼                  ▼                      │
│  ┌─────────────┐   ┌──────────────┐   ┌──────────────┐            │
│  │ FordFactory │   │ToyotaFactory │   │ HondaFactory │            │
│  │ (Concrete)  │   │ (Concrete)   │   │  (Concrete)  │            │
│  └──────┬──────┘   └──────┬───────┘   └──────┬───────┘            │
│         │                 │                    │                    │
│         │  CREATES        │  CREATES           │  CREATES           │
│         ▼                 ▼                    ▼                    │
└─────────┼─────────────────┼────────────────────┼────────────────────┘
          │                 │                    │
          ▼                 ▼                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                     CONCRETE PRODUCTS                               │
│                                                                     │
│  FordSuv, FordSedan          ToyotaSuv, ToyotaSedan          ...   │
│  (All implement ICar)          (All implement ICar)                 │
└─────────────────────────────────────────────────────────────────────┘
```

#### Class Hierarchy

```
┌─────────────────────────────────────────────────────────────────────┐
│                    PRODUCT HIERARCHY                                │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                           ICar (Abstract)                     │  │
│  │                       <<interface>>                           │  │
│  │                     + info() : string                         │  │
│  │                     + ~ICar() {virtual}                       │  │
│  └──────────────────────────┬────────────────────────────────────┘  │
│                             │                                       │
│              ┌──────────────┴──────────────┐                        │
│              │                             │                        │
│        ┌─────▼──────┐             ┌────────▼──────┐                │
│        │   Ford     │             │   Toyota      │                │
│        │(Abstract)  │             │(Abstract)     │                │
│        └─────┬──────┘             └───────┬───────┘                │
│              │                             │                        │
│      ┌───────┴────────┐            ┌───────┴──────────┐            │
│      │                │            │                  │            │
│ ┌────▼──────┐   ┌─────▼──────┐ ┌───▼──────────┐ ┌────▼────────┐   │
│ │FordSedan  │   │  FordSuv   │ │ToyotaSedan   │ │ ToyotaSuv   │   │
│ │(Concrete) │   │ (Concrete) │ │ (Concrete)   │ │  (Concrete) │   │
│ └───────────┘   └────────────┘ └──────────────┘ └─────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
                                   ▲
                                   │
                        ┌──────────┼──────────┐
                        │          │          │
┌───────────────────────┼──────────┼──────────┼───────────────────────┐
│  FACTORY HIERARCHY    ▼          ▼          ▼                       │
│                                                                     │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    ICarFactory (Abstract)                    │  │
│  │                        <<interface>>                         │  │
│  │               + makeSuv() : unique_ptr<ICar>                │  │
│  │               + makeSedan() : unique_ptr<ICar>              │  │
│  │               + ~ICarFactory() {virtual}                    │  │
│  └───────────────────────────────┬───────────────────────────────┘  │
│                                  │                                  │
│        ┌─────────────────────────┴─────────────────────────┐        │
│        ▼                    ▼                              ▼        │
│  ┌──────────────┐     ┌──────────────┐              ┌──────────┐  │
│  │ FordFactory  │     │ToyotaFactory │              │ LeastBusy│  │
│  │              │     │              │              │  Factory │  │
│  └──────────────┘     └──────────────┘              └──────────┘  │
└─────────────────────────────────────────────────────────────────────┘
```

### Product vs Factory Relationship

| Aspect | Product Hierarchy (Cars) | Factory Hierarchy (Factories) |
|--------|--------------------------|-------------------------------|
| **Purpose** | Defines WHAT can be created | Defines WHO creates them |
| **Base Type** | `ICar` (abstract interface) | `ICarFactory` (abstract interface) |
| **Concrete Types** | `FordSuv`, `ToyotaSedan`, etc. | `FordFactory`, `ToyotaFactory` |
| **Relationship** | Is-a: FordSuv *is a* ICar | Creates-a: FordFactory *creates* FordSuv |
| **Client Knows** | Only ICar interface | Only ICarFactory interface |
| **Key Benefit** | Polymorphic car objects | Guarantees compatible families |

#### Complete C++ Implementation (Abstract Factory)

```cpp
// ---------- PRODUCT HIERARCHY (Cars) ----------
// File: Cars.hpp
#pragma once
#include <string>
#include <memory>

// Abstract Product: Base interface for all cars
class ICar {
public:
    virtual ~ICar() = default;  // ALWAYS virtual destructor in base classes!
    virtual std::string info() const = 0;
};

// Concrete Products: Ford cars
class Ford : public ICar { };  // Abstract base for Ford cars

class FordSedan : public Ford {
public:
    std::string info() const override { 
        return "Ford Sedan"; 
    }
};

class FordSuv : public Ford {
public:
    std::string info() const override { 
        return "Ford SUV"; 
    }
};

// Concrete Products: Toyota cars
class Toyota : public ICar { };  // Abstract base for Toyota cars

class ToyotaSedan : public Toyota {
public:
    std::string info() const override { 
        return "Toyota Sedan";
    }
};

class ToyotaSuv : public Toyota {
public:
    std::string info() const override { 
        return "Toyota SUV"; 
    }
};

// ---------- ABSTRACT FACTORY INTERFACE ----------
// File: ICarFactory.hpp
#pragma once
#include "Cars.hpp"
#include <memory>

// Abstract Factory: Creates families of related cars
class ICarFactory {
public:
    virtual ~ICarFactory() = default;  // Virtual destructor!
    
    // Creates SUVs without knowing the concrete brand
    virtual std::unique_ptr<ICar> makeSuv() = 0;
    
    // Creates Sedans without knowing the concrete brand
    virtual std::unique_ptr<ICar> makeSedan() = 0;
};

// ---------- CONCRETE FACTORIES ----------
// File: FordFactory.hpp
#pragma once
#include "ICarFactory.hpp"
#include "Cars.hpp"

class FordFactory : public ICarFactory {
public:
    std::unique_ptr<ICar> makeSuv() override {
        return std::make_unique<FordSuv>(); }
    std::unique_ptr<ICar> makeSedan() override {
        return std::make_unique<FordSedan>(); }
};

// File: ToyotaFactory.hpp
#pragma once
#include "ICarFactory.hpp"
#include "Cars.hpp"

class ToyotaFactory : public ICarFactory {
public:
    std::unique_ptr<ICar> makeSuv() override {
        return std::make_unique<ToyotaSuv>(); }
    std::unique_ptr<ICar> makeSedan() override {
        return std::make_unique<ToyotaSedan>(); }
};

// ---------- CLIENT CODE ----------
// File: main.cpp
#include <iostream>
#include <memory>
#include "FordFactory.hpp"
#include "ToyotaFactory.hpp"

// Client function that works with ANY car factory
// This is the POWER of Abstract Factory - complete decoupling!
void createAndDisplayCars(ICarFactory& factory) {
    std::cout << "Creating vehicles...\n";
    
    // Create SUV
    auto suv = factory.makeSuv();
    std::cout << "SUV: " << suv->info() << std::endl;
    
    // Create Sedan
    auto sedan = factory.makeSedan();
    std::cout << "Sedan: " << sedan->info() << std::endl;
}

int main() {
    std::cout << "===== FORD FACTORY =====\n";
    FordFactory fordFactory;
    createAndDisplayCars(fordFactory);
    
    std::cout << "\n===== TOYOTA FACTORY =====\n";
    ToyotaFactory toyotaFactory;
    createAndDisplayCars(toyotaFactory);
    
    return 0;
}

// Expected Output:
// ===== FORD FACTORY =====
// Creating vehicles...
// SUV: Ford SUV
// Sedan: Ford Sedan
//
// ===== TOYOTA FACTORY =====
// Creating vehicles...
// SUV: Toyota SUV
// Sedan: Toyota Sedan
```

---

## Detailed Comparison: Factory Method vs Abstract Factory

### Why They Look the Same in C++

Both patterns are implemented using:
- Interfaces / abstract base classes
- `virtual` factory functions
- Polymorphism

Because C++ doesn't have "interfaces" like Java or C#, we use abstract classes for both. This makes them look identical syntactically:

```cpp
// Factory Method
class Creator {
public:
    virtual ICar* createCar() = 0;
};

class FordFactory : public Creator {
public:
    ICar* createCar() override { return new FordCar(); }
};

// Abstract Factory
class CarFactory {
public:
    virtual ICar* createCar() = 0;
    virtual IEngine* createEngine() = 0;
};
```

But their **intent**, **scope**, and **use-cases** are different.

### The One Sentence Difference

**Factory Method:** A single virtual function to create **ONE product**.

**Abstract Factory:** A family of virtual functions to create **RELATED products** (car + engine + wheels + seats... same brand).

### Technical Explanation of Each Pattern

#### Factory Method — Purpose

Let subclasses decide **which specific product class** to instantiate, without changing the code that uses the factory.

**Key Points:**
- Produces **a single type** of product
- You override **one virtual method** like `create()`
- It's about **customizing object creation** in a subclass

**Minimal structure:**
```cpp
class Product {
public:
    virtual void run() = 0;
    virtual ~Product() = default;
};

class Creator {
public:
    virtual std::unique_ptr<Product> create() = 0;
    virtual ~Creator() = default;
};
```

**When to use:** When your class needs to create objects, but you **want subclasses to choose the exact concrete type**.

#### Abstract Factory — Purpose

Provide an interface to create an **entire family of related products**, *ensuring they are compatible*.

**Key Points:**
- Produces **multiple products** (family)
- Each product has its own abstract type
- The factory provides **multiple creation functions**

```cpp
class UIAbstractFactory {
public:
    virtual std::unique_ptr<Button> createButton() = 0;
    virtual std::unique_ptr<TextField> createTextField() = 0;
    virtual ~UIAbstractFactory() = default;
};
```

**When to use:** When your system must work with **different product families** and must **never mix incompatible ones**.

### Implementation Differences

#### Factory Method Implementation Characteristics

**Key Implementation Points:**
- Uses **Template Method pattern** (public non-virtual method calling private virtual method)
- **Single creation method** per factory
- Factory hierarchy **inherits** creation behavior
- **Subclass decides** the exact product type

```cpp
// 1. Single product interface
class ICar { /* ... */ };

class Ford : public ICar { /* ... */ };
class Toyota : public ICar { /* ... */ };

// 2. Base factory with NVI (Non-Virtual Interface) pattern
class CarFactory {
public:
    // Public non-virtual interface
    std::unique_ptr<ICar> requestCar() {
        ++m_count;
        return createCar();  // Calls private virtual method
    }
    
private:
    // Private virtual method (Template Method pattern)
    virtual std::unique_ptr<ICar> createCar() = 0;
};

// 3. Subclasses override the creation method
class FordFactory : public CarFactory {
private:
    std::unique_ptr<ICar> createCar() override {
        return std::make_unique<Ford>();
    }
};
```

#### Abstract Factory Implementation Characteristics

**Key Implementation Points:**
- **Multiple creation methods** in one interface
- **No factory hierarchy inheritance** of creation logic (each concrete factory implements all methods)
- **Parallel hierarchies**: Product hierarchy AND Factory hierarchy
- **Factory decides** the entire family of products

```cpp
// 1. Multiple product interfaces
class ICar { /* ... */ };
class IEngine { /* ... */ };
class IWheels { /* ... */ };

// 2. Abstract factory interface with multiple creation methods
class ICarFactory {
public:
    virtual ~ICarFactory() = default;
    virtual std::unique_ptr<ICar> makeSuv() = 0;
    virtual std::unique_ptr<ICar> makeSedan() = 0;
    virtual std::unique_ptr<IEngine> makeEngine() = 0;
    virtual std::unique_ptr<IWheels> makeWheels() = 0;
};

// 3. Concrete factory implements ALL creation methods
class FordFactory : public ICarFactory {
public:
    std::unique_ptr<ICar> makeSuv() override {
        return std::make_unique<FordSuv>();
    }
    std::unique_ptr<ICar> makeSedan() override {
        return std::make_unique<FordSedan>();
    }
    std::unique_ptr<IEngine> makeEngine() override {
        return std::make_unique<FordEngine>();
    }
    // ... all other methods
};
```

### When to Use Each Pattern

#### Use Factory Method When:

1. **Single Product Type**: You only need to create one kind of object
2. **Subclass Variation**: Different subclasses should create different variants of that object
3. **Virtual Constructor**: You need to create objects polymorphically through a base class pointer
4. **Extensibility**: You want to allow future subclasses to change the product type without modifying base code
5. **Simpler Inheritance**: You have an existing class hierarchy and want to add creation capability

**Example Scenarios:**
- Document application where `Document` base class has `createDocument()` factory method, and `PDFDocument`, `WordDocument` subclasses override it
- Game character spawning where `EnemySpawner` base class has `createEnemy()`, and `EasySpawner`, `HardSpawner` create different enemy types
- Plugin systems where the plugin decides which objects to instantiate

#### Use Abstract Factory When:

1. **Product Families**: You need to create **multiple related objects** that must work together
2. **Compatibility Guarantee**: You must ensure objects from incompatible families aren't mixed (e.g., no Ford engine in a Toyota car)
3. **Runtime Family Switching**: You need to switch entire families of products at runtime
4. **System Independence**: Your system should be independent of how its products are created
5. **Library Design**: You're providing a library and want to hide implementation details while exposing a clean API

**Example Scenarios:**
- Cross-platform GUI libraries (WindowsFactory creates WindowsButton, WindowsCheckbox; MacFactory creates MacButton, MacCheckbox)
- Database abstraction layers (SqlServerFactory creates SqlConnection, SqlCommand; PostgresFactory creates PostgresConnection, PostgresCommand)
- Automotive manufacturing systems where you need entire vehicle families (chassis, engine, wheels) from one manufacturer
- Theme systems in applications (DarkThemeFactory, LightThemeFactory creating related UI components)

### Concrete Code Comparison

#### Factory Method Example (Single Product)

```cpp
// Client code - works with ANY factory
void buildCar(CarFactory& factory) {
    auto car = factory.requestCar();  // ONE product
    car->info();
}

// Usage
FordFactory ford;
ToyotaFactory toyota;

buildCar(ford);    // Builds a Ford
buildCar(toyota);  // Builds a Toyota
```

#### Abstract Factory Example (Product Family)

```cpp
// Client code - works with ANY factory
void buildCompleteVehicle(ICarFactory& factory) {
    auto car = factory.makeSuv();      // Product 1
    auto engine = factory.makeEngine(); // Product 2
    auto wheels = factory.makeWheels(); // Product 3
    
    // All are guaranteed to be compatible
    car->info();
    engine->start();
    wheels->rotate();
}

// Usage
FordFactory ford;
ToyotaFactory toyota;

buildCompleteVehicle(ford);    // All Ford parts
buildCompleteVehicle(toyota);  // All Toyota parts
```

### Why Abstract Factory Often Uses Factory Methods Inside

**Abstract Factory = a collection of Factory Methods.**

Example:

```cpp
class CarAbstractFactory {
public:
    virtual std::unique_ptr<SUV> createSUV() = 0;          // factory method
    virtual std::unique_ptr<Sedan> createSedan() = 0;      // factory method
    virtual std::unique_ptr<Truck> createTruck() = 0;      // factory method
};
```

So the pattern names refer to **intent**, not syntax.

### Summary of Differences

| Comparison Point | Factory Method | Abstract Factory |
|------------------|----------------|------------------|
| **Number of Products** | One per factory | Multiple related products per factory |
| **Primary Goal** | Defer instantiation to subclasses | Create families of compatible objects |
| **Design Pattern** | Creational (based on inheritance) | Creational (based on object composition) |
| **Flexibility** | Medium - new subclasses can change product | High - can switch entire families at runtime |
| **Complexity** | Lower (single hierarchy) | Higher (parallel hierarchies) |
| **Client Code** | Calls single method | Calls multiple methods to get related objects |
| **Extensibility** | Easy to add new factory subclasses | Hard to add new product types (requires interface change) |

---

## Additional Real-World Examples

### GUI Component Library

Creating cross-platform UI elements that match the operating system's look and feel.

```cpp
// Abstract Products
class IButton { public: virtual void paint() = 0; };
class ICheckbox { public: virtual void paint() = 0; };

// Abstract Factory
class IGUIFactory {
public:
    virtual std::unique_ptr<IButton> createButton() = 0;
    virtual std::unique_ptr<ICheckbox> createCheckbox() = 0;
};

// Windows Implementation
class WindowsButton : public IButton {
public: void paint() override { std::cout << "Windows Button\n"; }
};
class WindowsCheckbox : public ICheckbox {
public: void paint() override { std::cout << "Windows Checkbox\n"; }
};

class WindowsFactory : public IGUIFactory {
public:
    std::unique_ptr<IButton> createButton() override {
        return std::make_unique<WindowsButton>();
    }
    std::unique_ptr<ICheckbox> createCheckbox() override {
        return std::make_unique<WindowsCheckbox>();
    }
};

// Mac Implementation
class MacButton : public IButton {
public: void paint() override { std::cout << "Mac Button\n"; }
};
class MacCheckbox : public ICheckbox {
public: void paint() override { std::cout << "Mac Checkbox\n"; }
};

class MacFactory : public IGUIFactory {
public:
    std::unique_ptr<IButton> createButton() override {
        return std::make_unique<MacButton>();
    }
    std::unique_ptr<ICheckbox> createCheckbox() override {
        return std::make_unique<MacCheckbox>();
    }
};

// Client Code
void createUI(IGUIFactory& factory) {
    auto button = factory.createButton();
    auto checkbox = factory.createCheckbox();
    button->paint();
    checkbox->paint();
}

int main() {
    std::string os = detectOS();
    std::unique_ptr<IGUIFactory> factory;
    
    if (os == "windows") {
        factory = std::make_unique<WindowsFactory>();
    } else if (os == "mac") {
        factory = std::make_unique<MacFactory>();
    }
    
    createUI(*factory);
}
```

### Database Connection Factory

Creating database-specific connection objects, commands, and readers.

```cpp
class IDbConnection { public: virtual void connect() = 0; };
class IDbCommand { public: virtual void execute() = 0; };
class IDataReader { public: virtual void read() = 0; };

class IDbFactory {
public:
    virtual std::unique_ptr<IDbConnection> createConnection() = 0;
    virtual std::unique_ptr<IDbCommand> createCommand() = 0;
    virtual std::unique_ptr<IDataReader> createReader() = 0;
};

// For SQL Server
class SqlConnection : public IDbConnection { /* ... */ };
class SqlCommand : public IDbCommand { /* ... */ };
class SqlDataReader : public IDataReader { /* ... */ };

class SqlFactory : public IDbFactory {
public:
    std::unique_ptr<IDbConnection> createConnection() override {
        return std::make_unique<SqlConnection>();
    }
    std::unique_ptr<IDbCommand> createCommand() override {
        return std::make_unique<SqlCommand>();
    }
    std::unique_ptr<IDataReader> createReader() override {
        return std::make_unique<SqlDataReader>();
    }
};

// For PostgreSQL
class PostgresConnection : public IDbConnection { /* ... */ };
class PostgresCommand : public IDbCommand { /* ... */ };
class PostgresDataReader : public IDataReader { /* ... */ };

class PostgresFactory : public IDbFactory {
public:
    std::unique_ptr<IDbConnection> createConnection() override {
        return std::make_unique<PostgresConnection>();
    }
    std::unique_ptr<IDbCommand> createCommand() override {
        return std::make_unique<PostgresCommand>();
    }
    std::unique_ptr<IDataReader> createReader() override {
        return std::make_unique<PostgresDataReader>();
    }
};

// Client code can switch databases seamlessly
void processData(IDbFactory& dbFactory) {
    auto conn = dbFactory.createConnection();
    auto cmd = dbFactory.createCommand();
    auto reader = dbFactory.createReader();
    
    conn->connect();
    cmd->execute();
    reader->read();
}
```

---

## Best Practices

### 1. Always Use Virtual Destructors
```cpp
virtual ~ICarFactory() = default;  // Prevents memory leaks!
```

### 2. Prefer `unique_ptr` for Return Values
```cpp
// Good: Clear ownership transfer
virtual std::unique_ptr<ICar> makeSuv() = 0;

// Bad: Raw pointers, unclear ownership
virtual ICar* makeSuv() = 0;
```

### 3. Keep Factories Lightweight
Factories should focus on creation, not complex business logic.

### 4. Consider Dependency Injection
```cpp
// Inject the factory instead of hardcoding
class CarDealership {
private:
    ICarFactory& factory;  // Reference to abstract factory
public:
    CarDealership(ICarFactory& f) : factory(f) {}
    
    void showInventory() {
        auto suv = factory.makeSuv();
        auto sedan = factory.makeSedan();
        // ...
    }
};
```

### 5. Use Enums for Factory Selection
```cpp
enum class CarBrand { FORD, TOYOTA };

std::unique_ptr<ICarFactory> createFactory(CarBrand brand) {
    switch(brand) {
        case CarBrand::FORD: return std::make_unique<FordFactory>();
        case CarBrand::TOYOTA: return std::make_unique<ToyotaFactory>();
        default: throw std::invalid_argument("Unknown brand");
    }
}
```

### 6. Document Product Families
Clearly document which products belong to which families and why they must be used together.

### 7. Avoid Over-Engineering
Use the simplest pattern that satisfies your requirements. Simple Factory is often sufficient for basic cases.

---

## Modern C++ Enhancements

### Using `std::expected` (C++23) for Error Handling
```cpp
#include <expected>

std::expected<std::unique_ptr<ICar>, std::string> 
ToyotaFactory::makeSuv() {
    try {
        return std::make_unique<ToyotaSuv>();
    } catch(...) {
        return std::unexpected("Failed to create Toyota SUV");
    }
}
```

### Using Concepts (C++20) for Constraints
```cpp
template<typename T>
concept CarConcept = std::is_base_of_v<ICar, T>;

template<CarConcept T>
class Factory {
    std::unique_ptr<ICar> create() {
        return std::make_unique<T>();
    }
};
```

### Using Modules (C++20)
```cpp
// Instead of #pragma once
export module CarFactory;

export class ICar { /* ... */ };
```

### Using `std::variant` for Type-Safe Factories
```cpp
using CarVariant = std::variant<FordSuv*, ToyotaSuv*, HondaSuv*>;

class AdvancedCarFactory {
public:
    virtual CarVariant createCar(const std::string& type) = 0;
};
```

---

## Testing with Factory Patterns

### Mock Factory for Testing
```cpp
class MockCarFactory : public ICarFactory {
public:
    std::unique_ptr<ICar> makeSuv() override {
        auto mock = std::make_unique<Mock<ICar>>();
        mock->info() = "Mock SUV";
        return mock;
    }
    std::unique_ptr<ICar> makeSedan() override {
        auto mock = std::make_unique<Mock<ICar>>();
        mock->info() = "Mock Sedan";
        return mock;
    }
};

// Test
void testCarCreation() {
    MockCarFactory mockFactory;
    createAndDisplayCars(mockFactory);  // Works seamlessly!
}
```

### Google Test Example
```cpp
TEST(CarFactoryTest, AbstractFactoryCreatesCompatibleFamily) {
    FordFactory factory;
    auto suv = factory.makeSuv();
    auto sedan = factory.makeSedan();
    
    // Verify both are Ford brand
    EXPECT_TRUE(dynamic_cast<Ford*>(suv.get()));
    EXPECT_TRUE(dynamic_cast<Ford*>(sedan.get()));
}

TEST(CarFactoryTest, FactoryMethodPolymorphicCreation) {
    FordFactory fordFactory;
    ToyotaFactory toyotaFactory;
    
    auto fordCar = fordFactory.requestCar();
    auto toyotaCar = toyotaFactory.requestCar();
    
    EXPECT_EQ(fordCar->info(), "Ford");
    EXPECT_EQ(toyotaCar->info(), "Toyota");
}
```

### Testing LeastBusyFactory
```cpp
TEST(CarFactoryTest, LeastBusyFactoryDistributesLoad) {
    std::vector<std::unique_ptr<CarFactory>> factories;
    factories.push_back(std::make_unique<FordFactory>());
    factories.push_back(std::make_unique<ToyotaFactory>());
    
    // Pre-use one factory
    factories[0]->requestCar();
    factories[0]->requestCar();
    
    LeastBusyFactory leastBusy(std::move(factories));
    
    // Should create from least busy (Toyota)
    auto car = leastBusy.requestCar();
    EXPECT_EQ(car->info(), "Toyota");
}
```

---

## Summary Flowchart

```
┌─────────────────────────────────────────────────────────────────────┐
│  STEP 1: CLIENT CODE                                                │
│  "I need a family of cars, but I don't care about the brand"      │
│  createAndDisplayCars(factory);                                     │
└───────────────────────────────┬─────────────────────────────────────┘
                                │
                                ▼
┌───────────────────────────────┼─────────────────────────────────────┐
│  STEP 2: ABSTRACT FACTORY INTERFACE                                 │
│  factory.makeSuv();                                                 │
│  factory.makeSedan();                                               │
└───────────────────────────────┬─────────────────────────────────────┘
                                │
        ┌───────────────────────┼───────────────────────┐
        │                       │                       │
        ▼                       ▼                       ▼
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│  FordFactory    │   │ ToyotaFactory   │   │  HondaFactory   │
│  (if selected)  │   │  (if selected)  │   │  (if selected)  │
└────────┬────────┘   └────────┬────────┘   └────────\┬────────┘
         │                     │                      │
         │  creates_family_of  │  creates_family_of   │  creates_family_of
         ▼                     ▼                      ▼
┌─────────────────┐   ┌─────────────────┐   ┌─────────────────┐
│  FordSuv        │   │  ToyotaSuv      │   │  HondaSuv       │
│  FordSedan      │   │  ToyotaSedan    │   │  HondaSedan     │
└─────────────────┘   └─────────────────┘   └─────────────────┘
         │                     │                      │
         └───────────┬─────────┴──────────┬───────────┘
                     │                    │
                     ▼                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│  STEP 3: CLIENT USES PRODUCTS                                       │
│  suv->info();  // "Ford SUV" or "Toyota SUV" or ...               │
│  sedan->info(); // "Ford Sedan" or "Toyota Sedan" or ...          │
│  (Client still doesn't know the concrete type!)                    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## How to Compile and Run

### Using g++

```bash
# For C++17 and above (without modules)
g++ -std=c++17 -o car_factory main.cpp Cars.cpp ICarFactory.cpp FordFactory.cpp ToyotaFactory.cpp

# Run
./car_factory

# For C++20 with modules support
g++ -std=c++20 -fmodules-ts -o car_factory main.cpp

# With optimizations and warnings
g++ -std=c++17 -O2 -Wall -Wextra -Wpedantic -o car_factory main.cpp

# Debug build
g++ -std=c++17 -g -DDEBUG -o car_factory main.cpp
```

### Using CMake

```cmake
cmake_minimum_required(VERSION 3.15)
project(CarFactory)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Create executable
add_executable(car_factory 
    main.cpp
    Cars.hpp
    ICarFactory.hpp
    FordFactory.hpp
    ToyotaFactory.hpp
)

# For modular approach, use:
# add_executable(car_factory main.cpp)
# target_compile_features(car_factory PUBLIC cxx_std_20)

# Add testing
enable_testing()
add_executable(test_car_factory test.cpp)
target_link_libraries(test_car_factory gtest gtest_main)
add_test(NAME CarFactoryTest COMMAND test_car_factory)
```

### Using Make

```makefile
CXX = g++
CXXFLAGS = -std=c++17 -Wall -Wextra -Wpedantic -O2
DEBUGFLAGS = -std=c++17 -g -DDEBUG
TARGET = car_factory
SOURCES = main.cpp Cars.cpp ICarFactory.cpp FordFactory.cpp ToyotaFactory.cpp
TEST_SOURCES = test.cpp Cars.cpp ICarFactory.cpp FordFactory.cpp ToyotaFactory.cpp

all: $(TARGET)

$(TARGET): $(SOURCES)
	$(CXX) $(CXXFLAGS) -o $(TARGET) $(SOURCES)

debug: $(SOURCES)
	$(CXX) $(DEBUGFLAGS) -o $(TARGET) $(SOURCES)

test: test_car_factory
	./test_car_factory

test_car_factory: $(TEST_SOURCES)
	$(CXX) $(CXXFLAGS) -o test_car_factory $(TEST_SOURCES) -lgtest -lgtest_main

clean:
	rm -f $(TARGET) test_car_factory

run: $(TARGET)
	./$(TARGET)
```

### Using Clang

```bash
clang++ -std=c++17 -stdlib=libc++ -o car_factory main.cpp
```

### Using Visual Studio

```powershell
cl /std:c++17 /EHsc main.cpp Cars.cpp ICarFactory.cpp FordFactory.cpp ToyotaFactory.cpp
```

---

## Key Takeaways

1. **Abstract Factory** = Interface for creating **families** of related objects
2. **Factory Method** = Subclasses decide which **single product** to create
3. **Simple Factory** = **One class** decides based on parameters (not a true pattern)
4. Use **Abstract Factory** when products must work together (no mixing brands)
5. Use **Factory Method** when subclasses should control object instantiation
6. Use **Simple Factory** when you just need to hide implementation details
7. All factory patterns **decouple** client code from concrete classes
8. They **centralize** object creation logic and make code **testable**
9. Abstract Factory is the most **complex** but provides the strongest **consistency guarantees**
10. Choose the **simplest pattern** that satisfies your requirements (KISS principle)
11. **Abstract Factory often uses Factory Methods inside it** - it's a collection of Factory Methods
12. The pattern names refer to **intent**, not syntax - both use virtual functions but solve different problems
13. **Parallel hierarchies** are the hallmark of Abstract Factory (Product hierarchy + Factory hierarchy)
14. **Template Method pattern** is the hallmark of Factory Method (NVI - Non-Virtual Interface)
15. **Factory Method** is inheritance-based, **Abstract Factory** is composition-based