# Builder Design Pattern

## 📋 Table of Contents

- [Why Builder Design Pattern Matters](#why-builder-design-pattern-matters)
- [The 5 Core Problems That Led to Builder Pattern](#the-5-core-problems-that-led-to-builder-pattern)
- [Understanding the Builder Pattern](#understanding-the-builder-pattern)
  - [What is Builder Pattern?](#what-is-builder-pattern)
  - [Key Concepts](#key-concepts)
  - [When to Use Builder Pattern](#when-to-use-builder-pattern)
- [Car Construction Simulation - Reference Example](#car-construction-simulation---reference-example)
  - [The Problem](#the-problem)
  - [Solution Architecture](#solution-architecture)
  - [Class Hierarchy](#class-hierarchy)
  - [Builder vs Director Relationship](#builder-vs-director-relationship)
- [Complete C++ Implementation](#complete-c-implementation)
  - [Basic Builder Pattern](#basic-builder-pattern)
  - [Fluent Interface Builder](#fluent-interface-builder)
  - [Director-Based Construction](#director-based-construction)
  - [Building Immutable Objects](#building-immutable-objects)
  - [Optional Parameters Pattern](#optional-parameters-pattern)
- [Benefits and Drawbacks](#benefits-and-drawbacks)
- [Comparison with Other Patterns](#comparison-with-other-patterns)
  - [Builder vs Factory Method](#builder-vs-factory-method)
  - [Builder vs Abstract Factory](#builder-vs-abstract-factory)
  - [Builder vs Telescoping Constructor](#builder-vs-telescoping-constructor)
- [Additional Real-World Examples](#additional-real-world-examples)
  - [SQL Query Builder](#sql-query-builder)
  - [HTTP Request Builder](#http-request-builder)
  - [Document Builder](#document-builder)
- [Best Practices](#best-practices)
- [Modern C++ Enhancements](#modern-c-enhancements)
- [Testing with Builder Pattern](#testing-with-builder-pattern)
- [Summary Flowchart](#summary-flowchart)
- [How to Compile and Run](#how-to-compile-and-run)
- [Key Takeaways](#key-takeaways)

---

## Why Builder Design Pattern Matters

Design patterns are not theoretical concepts—they are battle-tested solutions to recurring problems that developers face in real-world software development. The Builder pattern emerged because object construction became too complex, too error-prone, too inflexible, and too difficult to maintain with traditional constructors and factory methods.

While Factory patterns focus on **which object to create**, Builder focuses on **how to assemble complex objects step-by-step**. It's the difference between ordering a car (Factory) and custom-building a car with specific options (Builder).

---

## The 5 Core Problems That Led to Builder Pattern

| Problem | Why It's Bad | How Builder Solves It |
|---------|--------------|------------------------|
| **Telescoping Constructors** | Too many constructor parameters, hard to read, easy to swap values | Provides named methods for each parameter |
| **Complex Object Assembly** | Objects require many steps to initialize properly | Breaks construction into clear, ordered steps |
| **Immutability Requirements** | Cannot create immutable objects with many parameters | Builds object in one final step after configuration |
| **Invalid Object State** | Partially constructed objects can be used incorrectly | Prevents access until fully constructed |
| **Mixing Construction & Representation** | Construction logic pollutes business logic | Separates assembly from final representation |

**Bottom line:** Object construction became too complicated, too error-prone, and too rigid. Builder makes construction **step-by-step, readable, flexible, safe, and clean**.

---

## Understanding the Builder Pattern

### What is Builder Pattern?

The **Builder** is a creational design pattern that provides a flexible solution to constructing complex objects. Unlike other creational patterns, Builder constructs objects step-by-step, allowing you to produce different types and representations of an object using the same construction code.

**Key distinction:** While Factory patterns create objects in a single step, Builder **assembles objects incrementally** and **returns the final object only after complete construction**.

### Key Concepts

1. **Product**: The complex object being built (e.g., `Car`, `House`, `Document`)
2. **Builder Interface**: Declares construction steps for building the product
3. **Concrete Builder**: Implements the builder interface, provides methods for each construction step
4. **Director** (optional): Orchestrates the construction process using a builder object
5. **Client**: Configures the builder or director, then retrieves the final product

### When to Use Builder Pattern

**Use when:**
- Objects have many optional parameters (more than 4-5)
- Objects require complex step-by-step initialization
- You need to create immutable objects with many parameters
- You want to prevent objects from being in an inconsistent state
- Construction process should be independent of the final representation
- Same construction process can create different representations

**Don't use when:**
- Objects are simple with few parameters (use constructor)
- Objects don't require step-by-step construction
- All parameters are mandatory (use constructor)
- Performance is critical and object creation is frequent (builder has overhead)

---

## Car Construction Simulation - Reference Example

### The Problem

You need to construct a `Car` object with many optional features:
- Engine type (required)
- Transmission (required)
- Color (optional)
- GPS (optional)
- Leather seats (optional)
- Sunroof (optional)
- Alloy wheels (optional)
- Premium sound system (optional)
- ... and many more

**Problems with traditional approaches:**
- **Constructor**: Would need 10+ parameters, hard to remember order
- **Telescoping constructors**: Too many overloaded versions
- **Setters**: Cannot create immutable objects, risk of incomplete state
- **Factory**: Doesn't support step-by-step customization

### Solution Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                          CLIENT CODE                                │
│  "I want to build a custom car step by step"                      │
└──────────────────────────────┬──────────────────────────────────────┘
                               │
                               ▼
┌──────────────────────────────┼──────────────────────────────────────┐
│      BUILDER INTERFACE       │                                      │
│                              ▼                                      │
│  ┌──────────────────────────────────────────────────────────────┐  │
│  │                    ICarBuilder (Abstract)                    │  │
│  │  + setEngine()       : void                                  │  │
│  │  + setTransmission() : void                                  │  │
│  │  + setColor()        : void                                  │  │
│  │  + addGPS()          : void                                  │  │
│  │  + addSunroof()      : void                                  │  │
│  │  + build()           : unique_ptr<Car>                       │  │
│  └────────────────────────┬───────────────────────────────────────┘  │
│                           │                                         │
│        ┌──────────────────┼──────────────────┐                      │
│        ▼                  ▼                  ▼                      │
│  ┌─────────────┐   ┌──────────────┐   ┌──────────────┐            │
│  │SportsCar    │   │SUVBuilder    │   │EconomyBuilder│            │
│  │Builder      │   │              │   │              │            │
│  └──────┬──────┘   └──────┬───────┘   └──────┬───────┘            │
│         │                 │                    │                    │
│         │  builds         │  builds            │  builds            │
│         ▼                 ▼                    ▼                    │
└─────────┼─────────────────┼────────────────────┼────────────────────┘
          │                 │                    │
          ▼                 ▼                    ▼
┌─────────────────────────────────────────────────────────────────────┐
│                       CONCRETE PRODUCT                              │
│                                                                     │
│                     Car (Complex Object)                           │
│  ┌───────────────────────────────────────────────────────────────┐ │
│  │ Engine, Transmission, Color, GPS, Sunroof, Seats, ...        │ │
│  │ All set via builder, returned as complete, valid object      │ │
│  └───────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

### Class Hierarchy

```
┌─────────────────────────────────────────────────────────────────────┐
│                      PRODUCT HIERARCHY                              │
│                                                                     │
│  ┌───────────────────────────────────────────────────────────────┐  │
│  │                           Car (Final)                         │  │
│  │                       <<complex object>>                      │  │
│  │  - Engine*                                                    │  │
│  │  - Transmission*                                              │  │
│  │  - Color                                                      │  │
│  │  - bool hasGPS                                                │  │
│  │  - bool hasSunroof                                            │  │
│  │  + info() : string                                            │  │
│  └──────────────────────────┬────────────────────────────────────┘  │
│                             │                                       │
│                ┌────────────┴────────────┐                          │
│                │                         │                          │
│        ┌───────▼────────┐       ┌────────▼───────┐                │
│        │  (no Car       │       │  (no Car       │                │
│        │   subtypes)    │       │   subtypes)    │                │
│        └────────────────┘       └────────────────┘                │
└─────────────────────────────────────────────────────────────────────┘
                                   ▲
                                   │
                                   │
┌──────────────────────────────────┼─────────────────────────────────┐
│         BUILDER HIERARCHY        │                                 │
│                                  │                                 │
│  ┌───────────────────────────────▼──────────────────────────────┐ │
│  │                    ICarBuilder (Abstract)                    │ │
│  │                        <<interface>>                         │ │
│  │               + setEngine() : void                          │ │
│  │               + setTransmission() : void                    │ │
│  │               + setColor() : void                           │ │
│  │               + addGPS() : void                             │ │
│  │               + addSunroof() : void                         │ │
│  │               + build() : unique_ptr<Car>                   │ │
│  │               + ~ICarBuilder() {virtual}                    │ │
│  └───────────────────────────────┬───────────────────────────────┘ │
│                                  │                                 │
│        ┌─────────────────────────┴─────────────────────────┐       │
│        ▼                    ▼                              ▼       │
│  ┌──────────────┐     ┌──────────────┐              ┌──────────┐ │
│  │SportsBuilder │     │SUVBuilder    │              │Economy   │ │
│  │              │     │              │              │Builder   │ │
│  └──────────────┘     └──────────────┘              └──────────┘ │
└─────────────────────────────────────────────────────────────────────┘
```

### Builder vs Director Relationship

| Aspect | Builder | Director |
|--------|---------|----------|
| **Responsibility** | Knows **how** to build each part | Knows **what** to build and in **which order** |
| **Knowledge** | Product details, specific configurations | Construction algorithm, business rules |
| **Flexibility** | Can be used standalone | Optional but provides guided construction |
| **Example** | `setEngine()`, `setColor()` | `makeSportsCar()`, `makeFamilyCar()` |
| **Client Usage** | Client calls builder methods directly | Client calls director methods, director uses builder |

---

## Complete C++ Implementation

### Basic Builder Pattern

```cpp
// File: Car.hpp
#pragma once
#include <string>
#include <memory>

class Engine {
public:
    Engine(const std::string& type) : type_(type) {}
    std::string getType() const { return type_; }
private:
    std::string type_;
};

class Transmission {
public:
    Transmission(const std::string& type) : type_(type) {}
    std::string getType() const { return type_; }
private:
    std::string type_;
};

// The complex product we're building
class Car {
public:
    // Getters
    std::string getEngine() const { return engine_ ? engine_->getType() : "None"; }
    std::string getTransmission() const { return transmission_ ? transmission_->getType() : "None"; }
    std::string getColor() const { return color_; }
    bool hasGPS() const { return hasGPS_; }
    bool hasSunroof() const { return hasSunroof_; }
    int getSeatCount() const { return seatCount_; }
    
    // Generate info string
    std::string info() const {
        std::string info = "Car with " + getEngine() + " engine, " + 
                          getTransmission() + " transmission, " + 
                          color_ + " color";
        if (hasGPS_) info += ", GPS";
        if (hasSunroof_) info += ", Sunroof";
        info += ", " + std::to_string(seatCount_) + " seats";
        return info;
    }
    
    // Note: Car's constructor is private, only Builder can create it
    friend class CarBuilder;
    
private:
    Car() = default;
    
    std::unique_ptr<Engine> engine_;
    std::unique_ptr<Transmission> transmission_;
    std::string color_ = "Black";
    bool hasGPS_ = false;
    bool hasSunroof_ = false;
    int seatCount_ = 4;
};

// Abstract Builder Interface
class ICarBuilder {
public:
    virtual ~ICarBuilder() = default;
    
    virtual void setEngine(const std::string& type) = 0;
    virtual void setTransmission(const std::string& type) = 0;
    virtual void setColor(const std::string& color) = 0;
    virtual void addGPS() = 0;
    virtual void addSunroof() = 0;
    virtual void setSeatCount(int count) = 0;
    virtual std::unique_ptr<Car> build() = 0;
    
    // Reset builder to initial state
    virtual void reset() = 0;
};

// Concrete Builder
class CarBuilder : public ICarBuilder {
public:
    CarBuilder() {
        reset();
    }
    
    void setEngine(const std::string& type) override {
        car_->engine_ = std::make_unique<Engine>(type);
    }
    
    void setTransmission(const std::string& type) override {
        car_->transmission_ = std::make_unique<Transmission>(type);
    }
    
    void setColor(const std::string& color) override {
        car_->color_ = color;
    }
    
    void addGPS() override {
        car_->hasGPS_ = true;
    }
    
    void addSunroof() override {
        car_->hasSunroof_ = true;
    }
    
    void setSeatCount(int count) override {
        car_->seatCount_ = count;
    }
    
    std::unique_ptr<Car> build() override {
        // Validate required fields
        if (!car_->engine_ || !car_->transmission_) {
            throw std::runtime_error("Car must have engine and transmission");
        }
        
        // Return the built car and create a new one for next build
        return std::move(car_);
    }
    
    void reset() override {
        car_ = std::make_unique<Car>();
    }
    
private:
    std::unique_ptr<Car> car_;
};

// Client code
// File: main.cpp
#include <iostream>
#include "Car.hpp"

void demonstrateBasicBuilder() {
    std::cout << "===== BASIC BUILDER DEMO =====\n";
    
    CarBuilder builder;
    
    // Build a sports car
    builder.setEngine("V8");
    builder.setTransmission("Manual");
    builder.setColor("Red");
    builder.addSunroof();
    builder.setSeatCount(2);
    
    auto sportsCar = builder.build();
    std::cout << sportsCar->info() << std::endl;
    
    // Build an economy car
    builder.reset();
    builder.setEngine("1.6L");
    builder.setTransmission("Automatic");
    builder.setColor("Blue");
    builder.setSeatCount(4);
    
    auto economyCar = builder.build();
    std::cout << economyCar->info() << std::endl;
    
    // Build a luxury SUV
    builder.reset();
    builder.setEngine("V6 Turbo");
    builder.setTransmission("Automatic");
    builder.setColor("Black");
    builder.addGPS();
    builder.addSunroof();
    builder.setSeatCount(7);
    
    auto luxurySUV = builder.build();
    std::cout << luxurySUV->info() << std::endl;
}
```

### Fluent Interface Builder

```cpp
// Fluent interface version
class FluentCarBuilder {
public:
    FluentCarBuilder& withEngine(const std::string& type) {
        builder_->setEngine(type);
        return *this;
    }
    
    FluentCarBuilder& withTransmission(const std::string& type) {
        builder_->setTransmission(type);
        return *this;
    }
    
    FluentCarBuilder& withColor(const std::string& color) {
        builder_->setColor(color);
        return *this;
    }
    
    FluentCarBuilder& withGPS() {
        builder_->addGPS();
        return *this;
    }
    
    FluentCarBuilder& withSunroof() {
        builder_->addSunroof();
        return *this;
    }
    
    FluentCarBuilder& withSeats(int count) {
        builder_->setSeatCount(count);
        return *this;
    }
    
    std::unique_ptr<Car> build() {
        return builder_->build();
    }
    
private:
    std::unique_ptr<ICarBuilder> builder_ = std::make_unique<CarBuilder>();
};

// Fluent usage
void demonstrateFluentBuilder() {
    std::cout << "\n===== FLUENT BUILDER DEMO =====\n";
    
    FluentCarBuilder builder;
    
    auto sportsCar = builder
        .withEngine("V12")
        .withTransmission("Manual")
        .withColor("Yellow")
        .withSunroof()
        .withSeats(2)
        .build();
    
    std::cout << sportsCar->info() << std::endl;
}
```

### Director-Based Construction

```cpp
// Director class
class CarDirector {
public:
    CarDirector(ICarBuilder& builder) : builder_(builder) {}
    
    // Predefined construction recipes
    void constructSportsCar() {
        builder_.reset();
        builder_.setEngine("V8");
        builder_.setTransmission("Manual");
        builder_.setColor("Red");
        builder_.addSunroof();
        builder_.setSeatCount(2);
    }
    
    void constructFamilySUV() {
        builder_.reset();
        builder_.setEngine("V6");
        builder_.setTransmission("Automatic");
        builder_.setColor("Silver");
        builder_.addGPS();
        builder_.addSunroof();
        builder_.setSeatCount(7);
    }
    
    void constructEconomyCar() {
        builder_.reset();
        builder_.setEngine("1.6L");
        builder_.setTransmission("Automatic");
        builder_.setColor("White");
        builder_.setSeatCount(4);
    }
    
private:
    ICarBuilder& builder_;
};

// Client code
void demonstrateDirector() {
    std::cout << "\n===== DIRECTOR DEMO =====\n";
    
    CarBuilder builder;
    CarDirector director(builder);
    
    // Build sports car using director
    director.constructSportsCar();
    auto sportsCar = builder.build();
    std::cout << sportsCar->info() << std::endl;
    
    // Build family SUV using director
    director.constructFamilySUV();
    auto familySUV = builder.build();
    std::cout << familySUV->info() << std::endl;
}
```

### Building Immutable Objects

```cpp
// Immutable Car - all fields const
class ImmutableCar {
public:
    // Getters
    std::string getEngine() const { return engine_; }
    std::string getTransmission() const { return transmission_; }
    std::string getColor() const { return color_; }
    bool hasGPS() const { return hasGPS_; }
    bool hasSunroof() const { return hasSunroof_; }
    int getSeatCount() const { return seatCount_; }
    
    std::string info() const {
        std::string info = "Immutable Car with " + engine_ + " engine, " + 
                          transmission_ + " transmission, " + color_ + " color";
        if (hasGPS_) info += ", GPS";
        if (hasSunroof_) info += ", Sunroof";
        info += ", " + std::to_string(seatCount_) + " seats";
        return info;
    }
    
    // Builder is nested class - only way to construct
    class Builder;
    
private:
    ImmutableCar(std::string engine, std::string transmission, std::string color,
                 bool gps, bool sunroof, int seats)
        : engine_(engine), transmission_(transmission), color_(color),
          hasGPS_(gps), hasSunroof_(sunroof), seatCount_(seats) {}
    
    const std::string engine_;
    const std::string transmission_;
    const std::string color_;
    const bool hasGPS_;
    const bool hasSunroof_;
    const int seatCount_;
};

// Nested builder for immutable object
class ImmutableCar::Builder {
public:
    Builder& engine(const std::string& type) {
        engine_ = type;
        return *this;
    }
    
    Builder& transmission(const std::string& type) {
        transmission_ = type;
        return *this;
    }
    
    Builder& color(const std::string& color) {
        color_ = color;
        return *this;
    }
    
    Builder& gps(bool has) {
        hasGPS_ = has;
        return *this;
    }
    
    Builder& sunroof(bool has) {
        hasSunroof_ = has;
        return *this;
    }
    
    Builder& seats(int count) {
        seatCount_ = count;
        return *this;
    }
    
    std::unique_ptr<ImmutableCar> build() {
        // Validate
        if (engine_.empty() || transmission_.empty()) {
            throw std::runtime_error("Engine and transmission required");
        }
        
        return std::make_unique<ImmutableCar>(
            engine_, transmission_, color_, hasGPS_, hasSunroof_, seatCount_
        );
    }
    
private:
    std::string engine_;
    std::string transmission_;
    std::string color_ = "Black";
    bool hasGPS_ = false;
    bool hasSunroof_ = false;
    int seatCount_ = 4;
};

void demonstrateImmutableBuilder() {
    std::cout << "\n===== IMMUTABLE BUILDER DEMO =====\n";
    
    auto car = ImmutableCar::Builder()
        .engine("V8")
        .transmission("Manual")
        .color("Blue")
        .gps(true)
        .seats(2)
        .build();
    
    std::cout << car->info() << std::endl;
    
    // Once built, object cannot be modified
    // car->engine_ = "V6"; // COMPILE ERROR - const field
}
```

### Optional Parameters Pattern

```cpp
// Builder with optional validation
class ValidatingCarBuilder : public ICarBuilder {
public:
    // ... same as CarBuilder ...
    
    std::unique_ptr<Car> build() override {
        // Enhanced validation
        if (!engine_) {
            throw std::runtime_error("Engine is required");
        }
        if (!transmission_) {
            throw std::runtime_error("Transmission is required");
        }
        if (seatCount_ < 2 || seatCount_ > 9) {
            throw std::runtime_error("Seat count must be between 2 and 9");
        }
        
        return std::move(car_);
    }
    
    // Optional: partial build for testing
    std::unique_ptr<Car> buildPartial() {
        return std::move(car_);
    }
    
private:
    std::unique_ptr<Car> car_ = std::make_unique<Car>();
};
```

---

## Benefits and Drawbacks

### Benefits

| Benefit | Explanation |
|---------|-------------|
| **Readability** | Named methods make construction self-documenting |
| **Flexibility** | Can set only needed parameters, skip optional ones |
| **Immutability** | Enables creation of immutable objects with many parameters |
| **Validation** | Builder can validate before creating final object |
| **Step-by-step** | Construction can be paused, resumed, or conditional |
| **Type Safety** | Compiler catches method name typos, not parameter order errors |
| **Refactoring** | Adding new parameters doesn't break existing code |

### Drawbacks

| Drawback | Explanation |
|----------|-------------|
| **Boilerplate** | Requires creating builder class with many methods |
| **Complexity** | Overhead for simple objects with few parameters |
| **Mutability** | Builder itself is mutable (stateful) |
| **Performance** | Slightly slower than direct constructor call |
| **Memory** | Builder holds temporary object during construction |
| **Learning Curve** | More complex than straightforward constructors |

---

## Comparison with Other Patterns

### Builder vs Factory Method

**Q: When should I use Builder instead of Factory Method?**

**A:** Use Factory Method when you need to decide **which type** of object to create. Use Builder when you need to decide **how to assemble** an object step-by-step.

```cpp
// Factory Method - decides WHICH car
class CarFactory {
public:
    virtual std::unique_ptr<ICar> createCar() = 0;
};

class FordFactory : public CarFactory {
    std::unique_ptr<ICar> createCar() override {
        return std::make_unique<Ford>();  // Fixed Ford car
    }
};

// Builder - decides HOW to build the car
class CarBuilder {
public:
    void setEngine(const std::string& engine) { /* ... */ }
    void setColor(const std::string& color) { /* ... */ }
    // ... many more options
    std::unique_ptr<Car> build() { /* ... */ }
};

// Usage difference:
FordFactory factory;
auto car = factory.createCar();  // Get a Ford (no customization)

CarBuilder builder;
builder.setEngine("V8");
builder.setColor("Red");
// ... more customization
auto customCar = builder.build();  // Get exactly what you specified
```

| Factory Method | Builder |
|----------------|---------|
| Creates objects in **one step** | Creates objects in **multiple steps** |
| Decides **which type** | Decides **how to configure** |
| No configuration needed | Heavy configuration needed |
| Fixed product | Customizable product |

### Builder vs Abstract Factory

**Q: How is Builder different from Abstract Factory?**

**A:** Abstract Factory creates **families of compatible objects**. Builder creates **one complex object step-by-step**.

```cpp
// Abstract Factory - creates compatible families
class CarFactory {
public:
    virtual std::unique_ptr<ICar> makeCar() = 0;
    virtual std::unique_ptr<IEngine> makeEngine() = 0;
    virtual std::unique_ptr<IWheels> makeWheels() = 0;
};

// Builder - creates one complex object
class CarBuilder {
public:
    void setEngine(std::unique_ptr<IEngine> engine) { /* ... */ }
    void setWheels(std::unique_ptr<IWheels> wheels) { /* ... */ }
    std::unique_ptr<ICar> build() { /* ... */ }
};
```

| Abstract Factory | Builder |
|------------------|---------|
| Creates **multiple related objects** | Creates **one complex object** |
| Guarantees **family compatibility** | Guarantees **valid state** |
| **Cannot** customize individual objects | **Can** customize each step |
| **Runtime** family selection | **Step-by-step** configuration |

### Builder vs Telescoping Constructor

**Q: Why not just use constructors with default parameters?**

**A:** Telescoping constructors become unreadable with many optional parameters. Builder provides named methods that are self-documenting.

```cpp
// Telescoping constructor - hard to read
class Car {
public:
    Car(string engine, string transmission);
    Car(string engine, string transmission, string color);
    Car(string engine, string transmission, string color, bool gps);
    Car(string engine, string transmission, string color, bool gps, bool sunroof);
    // ... dozens more overloaded constructors
};

// Usage - which parameter is which?
Car car("V8", "Manual", "Red", true, false, 2, true, false, true);
// What does "true, false, 2, true" mean?

// Builder - clear and readable
auto car = CarBuilder()
    .engine("V8")
    .transmission("Manual")
    .color("Red")
    .gps(true)
    .seats(2)
    .build();
```

| Telescoping Constructor | Builder |
|-------------------------|---------|
| **Hard to read** parameter order | **Self-documenting** method names |
| **Compile-time** errors for missing params | **Runtime** validation |
| **Cannot** skip optional params easily | **Can** skip any optional param |
| **Immutable** if using const | **Enables** immutability |
| **No** step-by-step construction | **Supports** step-by-step |

---

## Additional Real-World Examples

### SQL Query Builder

```cpp
class SQLQueryBuilder {
public:
    SQLQueryBuilder& select(const std::string& columns) {
        query_ += "SELECT " + columns + " ";
        return *this;
    }
    
    SQLQueryBuilder& from(const std::string& table) {
        query_ += "FROM " + table + " ";
        return *this;
    }
    
    SQLQueryBuilder& where(const std::string& condition) {
        if (!hasWhere_) {
            query_ += "WHERE ";
            hasWhere_ = true;
        }
        query_ += condition + " ";
        return *this;
    }
    
    SQLQueryBuilder& andWhere(const std::string& condition) {
        query_ += "AND " + condition + " ";
        return *this;
    }
    
    SQLQueryBuilder& orWhere(const std::string& condition) {
        query_ += "OR " + condition + " ";
        return *this;
    }
    
    SQLQueryBuilder& orderBy(const std::string& columns) {
        query_ += "ORDER BY " + columns + " ";
        return *this;
    }
    
    SQLQueryBuilder& limit(int count) {
        query_ += "LIMIT " + std::to_string(count);
        return *this;
    }
    
    std::string build() {
        return query_;
    }
    
private:
    std::string query_;
    bool hasWhere_ = false;
};

// Usage
std::string query = SQLQueryBuilder()
    .select("*")
    .from("users")
    .where("age > 18")
    .andWhere("active = 1")
    .orderBy("created_at DESC")
    .limit(10)
    .build();
// Output: SELECT * FROM users WHERE age > 18 AND active = 1 ORDER BY created_at DESC LIMIT 10
```

### HTTP Request Builder

```cpp
class HttpRequest {
public:
    class Builder {
    public:
        Builder& method(const std::string& method) {
            request_.method_ = method;
            return *this;
        }
        
        Builder& url(const std::string& url) {
            request_.url_ = url;
            return *this;
        }
        
        Builder& header(const std::string& key, const std::string& value) {
            request_.headers_[key] = value;
            return *this;
        }
        
        Builder& body(const std::string& body) {
            request_.body_ = body;
            return *this;
        }
        
        Builder& timeout(int seconds) {
            request_.timeout_ = seconds;
            return *this;
        }
        
        HttpRequest build() {
            if (request_.method_.empty() || request_.url_.empty()) {
                throw std::runtime_error("Method and URL are required");
            }
            return std::move(request_);
        }
        
    private:
        HttpRequest request_;
    };
    
    void send() {
        std::cout << "Sending " << method_ << " request to " << url_ << std::endl;
        // Actual HTTP implementation here
    }
    
private:
    std::string method_;
    std::string url_;
    std::unordered_map<std::string, std::string> headers_;
    std::string body_;
    int timeout_ = 30;
};

// Usage
auto request = HttpRequest::Builder()
    .method("POST")
    .url("https://api.example.com/users")
    .header("Content-Type", "application/json")
    .header("Authorization", "Bearer token123")
    .body(R"({"name": "John", "age": 30})")
    .timeout(60)
    .build();

request.send();
```

### Document Builder

```cpp
class Document {
public:
    class Builder {
    public:
        Builder& addTitle(const std::string& title) {
            doc_.content_ += "# " + title + "\n\n";
            return *this;
        }
        
        Builder& addSection(const std::string& heading) {
            doc_.content_ += "## " + heading + "\n\n";
            return *this;
        }
        
        Builder& addParagraph(const std::string& text) {
            doc_.content_ += text + "\n\n";
            return *this;
        }
        
        Builder& addList(const std::vector<std::string>& items) {
            for (const auto& item : items) {
                doc_.content_ += "- " + item + "\n";
            }
            doc_.content_ += "\n";
            return *this;
        }
        
        Builder& addCodeBlock(const std::string& code, const std::string& language = "") {
            if (!language.empty()) {
                doc_.content_ += "```" + language + "\n";
            } else {
                doc_.content_ += "```\n";
            }
            doc_.content_ += code + "\n```\n\n";
            return *this;
        }
        
        Document build() {
            doc_.wordCount_ = countWords();
            return std::move(doc_);
        }
        
    private:
        int countWords() {
            std::istringstream iss(doc_.content_);
            return std::distance(std::istream_iterator<std::string>(iss), 
                               std::istream_iterator<std::string>());
        }
        
        Document doc_;
    };
    
    void save(const std::string& filename) {
        std::ofstream file(filename);
        file << content_;
        std::cout << "Saved document with " << wordCount_ << " words to " << filename << std::endl;
    }
    
private:
    std::string content_;
    int wordCount_ = 0;
};

// Usage
auto doc = Document::Builder()
    .addTitle("Builder Pattern Guide")
    .addSection("Introduction")
    .addParagraph("The Builder pattern is a creational design pattern...")
    .addList({"Benefit 1: Readability", "Benefit 2: Flexibility"})
    .addCodeBlock("CarBuilder builder;\nbuilder.setEngine(\"V8\");", "cpp")
    .build();

doc.save("builder_guide.md");
```

---

## Best Practices

### 1. Validate at Build Time
```cpp
std::unique_ptr<Car> build() {
    if (!engine_) {
        throw std::runtime_error("Engine is required");
    }
    // ... more validation
    return std::move(car_);
}
```

### 2. Provide Sensible Defaults
```cpp
class CarBuilder {
public:
    CarBuilder() : color_("Black"), seatCount_(4), hasGPS_(false) {}
    // ... methods ...
};
```

### 3. Make Product Constructor Private
```cpp
class Car {
private:
    Car() = default;  // Only Builder can construct
    friend class CarBuilder;
};
```

### 4. Return Builder Reference for Fluent Interface
```cpp
CarBuilder& setEngine(const std::string& type) {
    engine_ = type;
    return *this;
}
```

### 5. Use Director for Complex Recipes
```cpp
class CarDirector {
public:
    void constructSportsCar(CarBuilder& builder) {
        builder.reset()
            .setEngine("V8")
            .setTransmission("Manual")
            .setSeatCount(2);
    }
};
```

### 6. Support Method Chaining
```cpp
auto car = CarBuilder()
    .setEngine("V8")
    .setColor("Red")
    .addGPS()
    .build();
```

### 7. Document Required vs Optional Fields
```cpp
// Required: engine, transmission
// Optional: color (default: Black), GPS (default: false), etc.
```

### 8. Consider Thread Safety
```cpp
class ThreadSafeCarBuilder {
public:
    std::unique_ptr<Car> build() {
        std::lock_guard<std::mutex> lock(mutex_);
        // ... build logic ...
    }
private:
    std::mutex mutex_;
};
```

### 9. Use move semantics for efficiency
```cpp
std::unique_ptr<Car> build() {
    return std::move(car_);  // Move, not copy
}
```

### 10. Provide reset() method
```cpp
void reset() {
    car_ = std::make_unique<Car>();
}
```

---

## Modern C++ Enhancements

### Using `std::optional` (C++17) for Optional Parameters

```cpp
#include <optional>

class ModernCarBuilder {
public:
    ModernCarBuilder& setEngine(const std::string& type) {
        engine_ = type;
        return *this;
    }
    
    ModernCarBuilder& setColor(const std::string& color) {
        color_ = color;
        return *this;
    }
    
    ModernCarBuilder& setSunroof(bool has) {
        hasSunroof_ = has;
        return *this;
    }
    
    std::unique_ptr<Car> build() {
        if (!engine_) {
            throw std::runtime_error("Engine is required");
        }
        
        auto car = std::make_unique<Car>();
        car->engine_ = std::make_unique<Engine>(*engine_);
        if (color_) car->color_ = *color_;
        car->hasSunroof_ = hasSunroof_.value_or(false);
        return car;
    }
    
private:
    std::optional<std::string> engine_;
    std::optional<std::string> color_;
    std::optional<bool> hasSunroof_;
};
```

### Using Concepts (C++20) for Builder Constraints

```cpp
template<typename T>
concept Buildable = requires(T builder) {
    { builder.build() } -> std::same_as<std::unique_ptr<Car>>;
    { builder.setEngine(std::string{}) } -> std::same_as<T&>;
    { builder.setTransmission(std::string{}) } -> std::same_as<T&>;
};

template<Buildable B>
class Director {
public:
    void constructSportsCar(B& builder) {
        builder.setEngine("V8").setTransmission("Manual");
    }
};
```

### Using `std::variant` for Type-Safe Options

```cpp
using EngineVariant = std::variant<V8Engine, V6Engine, Inline4Engine>;

class VariantCarBuilder {
public:
    VariantCarBuilder& setEngine(EngineVariant engine) {
        engine_ = engine;
        return *this;
    }
    
    std::unique_ptr<Car> build() {
        // Use std::visit to handle different engine types
        std::visit([this](auto&& engine) {
            // Configure car based on engine type
        }, engine_);
        return std::move(car_);
    }
    
private:
    EngineVariant engine_;
    std::unique_ptr<Car> car_ = std::make_unique<Car>();
};
```

### Using `std::expected` (C++23) for Error Handling

```cpp
#include <expected>

class SafeCarBuilder {
public:
    std::expected<std::unique_ptr<Car>, std::string> build() {
        if (!engine_) {
            return std::unexpected("Engine is required");
        }
        if (!validateConfiguration()) {
            return std::unexpected("Invalid configuration");
        }
        return std::move(car_);
    }
    
private:
    bool validateConfiguration() {
        // Complex validation logic
        return true;
    }
    
    std::unique_ptr<Car> car_ = std::make_unique<Car>();
    bool engine_ = false;
};
```

### Using Coroutines (C++20) for Async Building

```cpp
#include <coroutine>

class AsyncCarBuilder {
public:
    struct promise_type {
        std::unique_ptr<Car> car;
        auto get_return_object() { return AsyncCarBuilder{*this}; }
        auto initial_suspend() { return std::suspend_never{}; }
        auto final_suspend() noexcept { return std::suspend_never{}; }
        void unhandled_exception() { std::terminate(); }
    };
    
    AsyncCarBuilder(promise_type& p) : handle_(std::coroutine_handle<promise_type>::from_promise(p)) {}
    ~AsyncCarBuilder() { if (handle_) handle_.destroy(); }
    
    AsyncCarBuilder& setEngine(const std::string& type) {
        // Async engine configuration
        return *this;
    }
    
    std::unique_ptr<Car> build() {
        return std::move(handle_.promise().car);
    }
    
private:
    std::coroutine_handle<promise_type> handle_;
};

// Usage
auto builder = [](const std::string& engine) -> AsyncCarBuilder {
    co_await std::suspend_always{};
    co_return;
};
```

---

## Testing with Builder Pattern

### Testing Builder Validation

```cpp
TEST(CarBuilderTest, ThrowsWhenEngineNotSet) {
    CarBuilder builder;
    builder.setTransmission("Manual");
    
    EXPECT_THROW(builder.build(), std::runtime_error);
}

TEST(CarBuilderTest, BuildsValidCar) {
    CarBuilder builder;
    builder.setEngine("V8")
           .setTransmission("Manual")
           .setColor("Red");
    
    auto car = builder.build();
    EXPECT_EQ(car->getEngine(), "V8");
    EXPECT_EQ(car->getTransmission(), "Manual");
    EXPECT_EQ(car->getColor(), "Red");
}

TEST(CarBuilderTest, DefaultValuesWork) {
    CarBuilder builder;
    builder.setEngine("V6")
           .setTransmission("Automatic");
    
    auto car = builder.build();
    EXPECT_EQ(car->getColor(), "Black");  // Default
    EXPECT_FALSE(car->hasGPS());          // Default
    EXPECT_EQ(car->getSeatCount(), 4);    // Default
}
```

### Testing Director Recipes

```cpp
TEST(CarDirectorTest, ConstructsSportsCarCorrectly) {
    CarBuilder builder;
    CarDirector director(builder);
    
    director.constructSportsCar();
    auto car = builder.build();
    
    EXPECT_EQ(car->getSeatCount(), 2);
    EXPECT_TRUE(car->hasSunroof());
}

TEST(CarDirectorTest, ConstructsFamilySUVCorrectly) {
    CarBuilder builder;
    CarDirector director(builder);
    
    director.constructFamilySUV();
    auto car = builder.build();
    
    EXPECT_EQ(car->getSeatCount(), 7);
    EXPECT_TRUE(car->hasGPS());
    EXPECT_TRUE(car->hasSunroof());
}
```

### Testing Fluent Interface

```cpp
TEST(FluentBuilderTest, MethodChainingWorks) {
    FluentCarBuilder builder;
    
    auto car = builder.withEngine("V12")
                     .withTransmission("Manual")
                     .withColor("Yellow")
                     .withSunroof()
                     .withSeats(2)
                     .build();
    
    EXPECT_EQ(car->getEngine(), "V12");
    EXPECT_EQ(car->getTransmission(), "Manual");
    EXPECT_TRUE(car->hasSunroof());
}
```

### Testing Immutable Objects

```cpp
TEST(ImmutableBuilderTest, ObjectIsTrulyImmutable) {
    auto car = ImmutableCar::Builder()
        .engine("V8")
        .transmission("Manual")
        .color("Blue")
        .build();
    
    // Verify fields are const - can't be modified
    // car->engine_ = "V6"; // Should not compile
    
    EXPECT_EQ(car->getEngine(), "V8");
    EXPECT_EQ(car->getColor(), "Blue");
}
```

---

## Summary Flowchart

```
┌─────────────────────────────────────────────────────────────────────┐
│  STEP 1: CLIENT CODE                                                │
│  "I want to build a complex object step by step"                  │
│  CarBuilder builder;                                                │
└───────────────────────────────┬─────────────────────────────────────┘
                                │
                                ▼
┌───────────────────────────────┼─────────────────────────────────────┐
│  STEP 2: BUILDER INTERFACE                                          │
│  builder.setEngine("V8");                                           │
│  builder.setTransmission("Manual");                                 │
│  builder.setColor("Red");                                           │
│  builder.addGPS();                                                  │
│  builder.addSunroof();                                              │
└───────────────────────────────┬─────────────────────────────────────┘
                                │
                                ▼
┌───────────────────────────────┼─────────────────────────────────────┐
│  STEP 3: OPTIONAL DIRECTOR                                          │
│  Director director(builder);                                        │
│  director.constructSportsCar();  // Or manual steps above          │
└───────────────────────────────┬─────────────────────────────────────┘
                                │
                                ▼
┌───────────────────────────────┼─────────────────────────────────────┐
│  STEP 4: VALIDATION & BUILD                                          │
│  auto car = builder.build();  // Validates and returns product     │
└─────────────────────────────────────────────────────────────────────┘
                                │
                                ▼
┌───────────────────────────────┼─────────────────────────────────────┐
│  STEP 5: FINAL PRODUCT                                              │
│  car->info();  // Fully constructed, valid object                 │
│  (Cannot be modified if immutable)                                 │
└─────────────────────────────────────────────────────────────────────┘
```

---

## How to Compile and Run

### Using g++

```bash
# For C++17 and above
g++ -std=c++17 -o builder_demo main.cpp Car.cpp CarBuilder.cpp

# Run
./builder_demo

# With optimizations and warnings
g++ -std=c++17 -O2 -Wall -Wextra -Wpedantic -o builder_demo main.cpp

# Debug build
g++ -std=c++17 -g -DDEBUG -o builder_demo main.cpp

# With testing
g++ -std=c++17 -o test_builder test.cpp Car.cpp -lgtest -lgtest_main
```

### Using CMake

```cmake
cmake_minimum_required(VERSION 3.15)
project(CarBuilder)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Create executable
add_executable(builder_demo 
    main.cpp
    Car.hpp
    Car.cpp
    CarBuilder.hpp
    CarBuilder.cpp
)

# Add testing
enable_testing()
add_executable(test_builder test.cpp Car.cpp CarBuilder.cpp)
target_link_libraries(test_builder gtest gtest_main)
add_test(NAME BuilderTest COMMAND test_builder)
```

### Using Make

```makefile
CXX = g++
CXXFLAGS = -std=c++17 -Wall -Wextra -Wpedantic -O2
DEBUGFLAGS = -std=c++17 -g -DDEBUG
TARGET = builder_demo
SOURCES = main.cpp Car.cpp CarBuilder.cpp
TEST_SOURCES = test.cpp Car.cpp CarBuilder.cpp

all: $(TARGET)

$(TARGET): $(SOURCES)
	$(CXX) $(CXXFLAGS) -o $(TARGET) $(SOURCES)

debug: $(SOURCES)
	$(CXX) $(DEBUGFLAGS) -o $(TARGET) $(SOURCES)

test: test_builder
	./test_builder

test_builder: $(TEST_SOURCES)
	$(CXX) $(CXXFLAGS) -o test_builder $(TEST_SOURCES) -lgtest -lgtest_main

clean:
	rm -f $(TARGET) test_builder

run: $(TARGET)
	./$(TARGET)
```

### Using Clang

```bash
clang++ -std=c++17 -stdlib=libc++ -o builder_demo main.cpp Car.cpp
```

### Using Visual Studio

```powershell
cl /std:c++17 /EHsc main.cpp Car.cpp CarBuilder.cpp
```

---

## Key Takeaways

1. **Builder** = Step-by-step construction of complex objects
2. **Fluent interface** makes code readable and self-documenting
3. **Director** provides predefined construction recipes (optional)
4. **Enables immutability** even with many parameters
5. **Validates** object state before returning
6. **Separates** construction logic from business logic
7. **Prevents** telescoping constructors
8. **Supports** conditional and dynamic construction
9. **More boilerplate** than simple constructors but worth it for complex objects
10. **Different from Factory**: Factory decides **which type**, Builder decides **how to assemble**
11. **Different from Abstract Factory**: Abstract Factory creates **multiple related objects**, Builder creates **one complex object**
12. **Modern C++** features (optional, concepts, expected) enhance builder safety
13. **Testing** is easier with clear validation points
14. **Thread safety** must be considered if builder is shared
15. **Reset capability** allows builder reuse
16. **Nested builder pattern** is ideal for immutable objects
17. **Method chaining** is the key to fluent API
18. **Performance overhead** is minimal compared to readability benefits