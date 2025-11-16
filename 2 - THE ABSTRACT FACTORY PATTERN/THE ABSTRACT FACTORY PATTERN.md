# Abstract Factory Design Pattern

## Table of Contents

- [What is the Abstract Factory Pattern?](#what-is-the-abstract-factory-pattern)
- [Car Factory Simulation - Your Reference Example](#car-factory-simulation---your-reference-example)
- [Complete C++ Implementation](#complete-c-implementation)
- [Benefits of Abstract Factory](#benefits-of-abstract-factory)
- [Drawbacks](#drawbacks)
- [When to Use Abstract Factory](#when-to-use-abstract-factory)
- [Why Not Just Use Simple Factories?](#why-not-just-use-simple-factories)
- [Comparison with Other Patterns](#comparison-with-other-patterns)
- [Additional Examples](#additional-examples)
- [Best Practices](#best-practices)
- [Modern C++ Enhancements](#modern-c-enhancements)
- [Testing with Abstract Factory](#testing-with-abstract-factory)
- [Summary Flowchart](#summary-flowchart)
- [How to Compile and Run](#how-to-compile-and-run)
- [Recommended Reading](#recommended-reading)
- [Key Takeaways](#key-takeaways)

---

## What is the Abstract Factory Pattern?

The **Abstract Factory** is a creational design pattern that provides an interface for creating families of related or dependent objects without specifying their concrete classes. Think of it as a "factory of factories" or a super-factory that creates other factories.

### Key Idea
Instead of creating objects directly using `new`, you ask a factory object to create them for you. The abstract factory ensures that all products created by a concrete factory work together correctly.

---

## Car Factory Simulation - Your Reference Example

### The Problem
You need to create cars from different manufacturers (Ford, Toyota), and each manufacturer makes multiple car types (Sedan, SUV). You want to ensure that:
- A Ford factory only creates Ford cars
- A Toyota factory only creates Toyota cars
- The client code doesn't need to know the specific car type or brand

### The Solution Structure

```
┌─────────────────────────────────────────────────────────────────┐
│                    CLIENT CODE                                  │
│  (Uses ICarFactory, doesn't know concrete brands)              │
└─────────────────────────────────────────────────────────────────┘
                                   │
                                   ▼
┌─────────────────────────────────────────────────────────────────┐
│              ABSTRACT FACTORY (ICarFactory)                     │
│  - makeSuv()   → returns ICar*                                  │
│  - makeSedan() → returns ICar*                                  │
└─────────────────────────────────────────────────────────────────┘
           ▲                               ▲
           │                               │
    ┌──────┴──────┐                 ┌──────┴──────┐
    │             │                 │             │
┌───┴────┐   ┌────┴────┐       ┌───┴────┐   ┌────┴────┐
│FORD    │   │TOYOTA   │       │FORD    │   │TOYOTA   │
│FACTORY │   │FACTORY  │       │SEDAN   │   │SEDAN    │
└───┬────┘   └────┬────┘       └───┬────┘   └────┬────┘
    │             │                 │             │
    └──────┬──────┘                 └──────┬──────┘
           │                               │
    ┌──────▼───────────────────────┬───────▼────────┐
    │    CONCRETE PRODUCTS         │   PRODUCTS     │
    │  - FordSuv                   │  - ToyotaSuv   │
    │  - FordSedan                 │  - ToyotaSedan │
    └──────────────────────────────┴────────────────┘
```

### Class Hierarchy Diagram

```
┌──────────────────────────────────────────────────────────────────────┐
│                            ICar (Abstract Product)                   │
│   <<interface>>                                                      │
│   + info() : string                                                  │
│   + ~ICar() {virtual}                                                │
└──────────────────┬───────────────────────────────────────────────────┘
                   │
        ┌──────────┴──────────┬───────────────┐
        │                     │               │
    ┌───▼────────┐      ┌─────▼──────┐       │
    │   Ford     │      │   Toyota   │       │
    │(Abstract)  │      │(Abstract)  │       │
    └────┬───────┘      └─────┬──────┘       │
         │                    │              │
    ┌────┴──────┐        ┌────┴──────┐      │
    │           │        │           │      │
┌───▼──────┐ ┌──▼────────┐ ┌──▼──────────┐ │
│FordSedan │ │ FordSuv   │ │ToyotaSedan  │ │
└──────────┘ └───────────┘ └─────────────┘ │
                                           │
                                           │
┌──────────────────────────────────────────┘
│
┌──────────────────────────────────────────────────────────────────────┐
│                     ICarFactory (Abstract Factory)                   │
│   <<interface>>                                                      │
│   + makeSuv() : unique_ptr<ICar>                                     │
│   + makeSedan() : unique_ptr<ICar>                                   │
│   + ~ICarFactory() {virtual}                                         │
└──────────────────┬───────────────────────────────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
    ┌───▼────────┐      ┌─────▼────────┐
    │FordFactory │      │ToyotaFactory │
    └────────────┘      └──────────────┘
```

---

## Complete C++ Implementation

### Step 1: The Product Hierarchy (Cars)

```cpp
// File: Cars.hpp
#pragma once
#include <string>
#include <memory>

// Abstract Product: Base interface for all cars
export class ICar
{
public:
    virtual ~ICar() = default;  // ALWAYS virtual destructor in base classes!
    virtual std::string info() const = 0;
};

// Concrete Products: Ford cars
export class Ford : public ICar { };  // Abstract base for Ford cars

export class FordSedan : public Ford
{
public:
    std::string info() const override { 
        return "Ford Sedan"; 
    }
};

export class FordSuv : public Ford
{
public:
    std::string info() const override { 
        return "Ford SUV"; 
    }
};

// Concrete Products: Toyota cars
export class Toyota : public ICar { };  // Abstract base for Toyota cars

export class ToyotaSedan : public Toyota
{
public:
    std::string info() const override { 
        return "Toyota Sedan";
    }
};

export class ToyotaSuv : public Toyota
{
public:
    std::string info() const override { 
        return "Toyota SUV"; 
    }
};
```

### Step 2: The Abstract Factory

```cpp
// File: ICarFactory.hpp
#pragma once
#include "Cars.hpp"
#include <memory>

// Abstract Factory: Creates families of related cars
export class ICarFactory
{
public:
    virtual ~ICarFactory() = default;  // Virtual destructor!
    
    // Creates SUVs without knowing the concrete brand
    virtual std::unique_ptr<ICar> makeSuv() = 0;
    
    // Creates Sedans without knowing the concrete brand
    virtual std::unique_ptr<ICar> makeSedan() = 0;
};
```

### Step 3: Concrete Factories

```cpp
// File: FordFactory.hpp
#pragma once
#include "ICarFactory.hpp"
#include "Cars.hpp"

export class FordFactory : public ICarFactory
{
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

export class ToyotaFactory : public ICarFactory
{
public:
    std::unique_ptr<ICar> makeSuv() override {
        return std::make_unique<ToyotaSuv>(); }
    std::unique_ptr<ICar> makeSedan() override {
        return std::make_unique<ToyotaSedan>(); }
};
```

### Step 4: Client Code (How to Use It)

```cpp
// File: main.cpp
#include <iostream>
#include <memory>
#include "FordFactory.hpp"
#include "ToyotaFactory.hpp"

// Client function that works with ANY car factory
// This is the POWER of Abstract Factory - complete decoupling!
void createAndDisplayCars(ICarFactory& factory)
{
    std::cout << "Creating vehicles...\n";
    
    // Create SUV
    auto suv = factory.makeSuv();
    std::cout << "SUV: " << suv->info() << std::endl;
    
    // Create Sedan
    auto sedan = factory.makeSedan();
    std::cout << "Sedan: " << sedan->info() << std::endl;
}

int main()
{
    std::cout << "===== FORD FACTORY =====\n";
    FordFactory fordFactory;
    createAndDisplayCars(fordFactory);
    
    std::cout << "\n===== TOYOTA FACTORY =====\n";
    ToyotaFactory toyotaFactory;
    createAndDisplayCars(toyotaFactory);
    
    return 0;
}
```

### Expected Output:
```
===== FORD FACTORY =====
Creating vehicles...
SUV: Ford SUV
Sedan: Ford Sedan

===== TOYOTA FACTORY =====
Creating vehicles...
SUV: Toyota SUV
Sedan: Toyota Sedan
```

---

## Benefits of Abstract Factory

| Benefit | Explanation |
|---------|-------------|
| **Encapsulation** | Details of object creation are hidden from the client |
| **Consistency** | Guarantees that products from one family work together |
| **Flexibility** | Easy to switch between entire families of products |
| **Scalability** | Adding new product families is straightforward |
| **Testability** | Easy to mock factories for unit testing |

---

## Drawbacks

| Drawback | Explanation |
|----------|-------------|
| **Complexity** | More classes and interfaces to manage |
| **Extensibility** | Hard to add new product types (requires changing the abstract factory interface) |
| **Redundancy** | Can lead to duplicate code in concrete factories |
| **Overkill** | May be unnecessary for simple object creation scenarios |

---

## When to Use Abstract Factory

**Use when:**
- System needs to be independent of how its products are created
- System needs to work with multiple families of related products
- Products from one family must be used together
- You want to provide a library of products without exposing implementation

**Don't use when:**
- When you only have one product type, or products are not related
- When object creation logic is trivial and doesn't need abstraction
- When you need to add new product types frequently (this requires changing the abstract factory interface, which affects all concrete factories)

---

## Why Not Just Use Simple Factories?

**Q: Why not just use simple factories?**

**A:** Simple factories create one type of object. Abstract Factory creates RELATED families. It ensures you don't mix incompatible objects.

A simple factory might look like this:
```cpp
class SimpleCarFactory {
public:
    ICar* createCar(std::string type) {
        if (type == "fordSuv") return new FordSuv();
        if (type == "toyotaSedan") return new ToyotaSedan();
        // ... but this can lead to mixing incompatible objects!
    }
};
```

**The key difference**: Abstract Factory ensures you don't mix incompatible objects. With a simple factory, you could accidentally create a FordSuv and ToyotaSedan in the same "set," breaking consistency. The Abstract Factory pattern guarantees that if you use a `FordFactory`, you'll ONLY get Ford products that work together.

---

## Comparison with Other Patterns

### Abstract Factory vs. Factory Method

**Q: When should I NOT use it?**

**A:** When you only have one product type, or products are not related, or when object creation logic is trivial.

| Abstract Factory | Factory Method |
|------------------|----------------|
| Creates families of related objects | Creates a single type of object |
| Uses composition (factory object) | Uses inheritance (subclass decides) |
| Focus on "WHAT" to create | Focus on "WHO" creates it |

```cpp
// Factory Method
class Car {
    virtual Car* createCar() = 0;  // One product
};

// Abstract Factory
class CarFactory {
    virtual SUV* createSUV() = 0;      // Product family
    virtual Sedan* createSedan() = 0;  // Multiple related products
};
```

### Abstract Factory vs. Builder

**Q: How is it different from Builder?**

**A:** Builder constructs complex objects step-by-step. Abstract Factory creates complete, ready-to-use objects immediately.

- **Abstract Factory**: Creates objects in one step, focuses on **WHAT** family of objects
- **Builder**: Creates objects step-by-step, focuses on **HOW** to construct complex objects

```cpp
// Abstract Factory: One-step creation
auto fordSuv = fordFactory.makeSuv();  // Complete object

// Builder: Multi-step construction
CarBuilder builder;
builder.setEngine("V8");
builder.setWheels(4);
builder.setColor("Red");
auto car = builder.build();  // Step-by-step
```

---

## Additional Examples

### Example 1: GUI Component Library
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

// macOS Implementation (similar structure)
class MacButton : public IButton { /* ... */ };
class MacCheckbox : public ICheckbox { /* ... */ };
class MacFactory : public IGUIFactory { /* ... */ };

// Client Code
void createUI(IGUIFactory& factory) {
    auto button = factory.createButton();
    auto checkbox = factory.createCheckbox();
    button->paint();
    checkbox->paint();
}
```

### Example 2: Database Connection Factory
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

class SqlFactory : public IDbFactory { /* ... */ };

// For PostgreSQL
class PostgresConnection : public IDbConnection { /* ... */ };
class PostgresCommand : public IDbCommand { /* ... */ };
// ... and so on
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
Factories should focus on creation, not complex logic.

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

---

## Testing with Abstract Factory

```cpp
// Mock Factory for Testing
class MockCarFactory : public ICarFactory {
public:
    std::unique_ptr<ICar> makeSuv() override {
        auto mock = std::make_unique<Mock<ICar>>();
        mock->info() = "Mock SUV";
        return mock;
    }
    // ...
};

// Test
void testCarCreation() {
    MockCarFactory mockFactory;
    createAndDisplayCars(mockFactory);  // Works seamlessly!
}
```

---

## Summary Flowchart

```
                    Client Needs Objects
                            │
                            ▼
              Client Calls Abstract Factory
                            │
                            ▼
         ┌──────────────────┼──────────────────┐
         │                  │                  │
    ┌────▼────┐        ┌────▼─────┐      ┌────▼─────┐
    │ Ford    │        │ Toyota   │      │  Honda   │
    │ Factory │        │ Factory  │      │ Factory  │
    └────┬────┘        └────┬─────┘      └────┬─────┘
         │                  │                  │
         └──────────┬───────┴──────────┬───────┘
                    ▼                  ▼
            ┌──────────────┐    ┌──────────────┐
            │  Ford SUV    │    │ Toyota SUV   │
            │  Ford Sedan  │    │ Toyota Sedan │
            └──────────────┘    └──────────────┘
                    │
                    ▼
            Client Uses Objects
           (No knowledge of concrete types)
```

---

## How to Compile and Run

### Using g++ (with C++20 modules support):
```bash
# For older C++ versions without modules
g++ -std=c++17 -o car_factory main.cpp Cars.cpp ICarFactory.cpp FordFactory.cpp ToyotaFactory.cpp

# Run
./car_factory
```

### Using CMake:
```cmake
cmake_minimum_required(VERSION 3.15)
project(CarFactory)

set(CMAKE_CXX_STANDARD 17)

add_executable(car_factory 
    main.cpp
    Cars.hpp
    ICarFactory.hpp
    FordFactory.hpp
    ToyotaFactory.hpp
)
```

---

## Key Takeaways

1. **Abstract Factory = Interface for creating families of related objects**
2. **Ensures compatibility** between created objects
3. **Promotes consistency** - you can't accidentally mix Ford engines with Toyota chassis
4. **Enables switching** entire product families easily
5. **Increases complexity** - use only when warranted
