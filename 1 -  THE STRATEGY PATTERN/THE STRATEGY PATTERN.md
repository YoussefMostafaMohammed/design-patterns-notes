# Modern C++ Strategy Pattern & Reusability Guide

## 📋 Table of Contents
1. [Designing for Reusability in Modern C++](#designing-for-reusability-in-modern-c)
2. [The Strategy Pattern: Overview](#the-strategy-pattern-overview)
3. [Implementation Approaches](#implementation-approaches)
   - [3.1 Customizability through Interfaces](#31-customizability-through-interfaces)
     - [Dependency Inversion vs Dependency Injection](#dependency-inversion-vs-dependency-injection)
     - [Modern Implementation Example](#modern-implementation-example)
   - [3.2 Customizability through Callbacks](#32-customizability-through-callbacks)
     - [What Is a Callback?](#what-is-a-callback)
     - [Types of Callbacks in C++](#types-of-callbacks-in-c)
     - [Why Use `std::function`?](#why-use-stdfunction)
     - [Callbacks in the Strategy Design Pattern](#callbacks-in-the-strategy-design-pattern)
     - [Summary Table](#summary-table)
   - [3.3 Customizability through Template Parameters](#33-customizability-through-template-parameters)
     - [Templates with Callable Parameters](#templates-with-callable-parameters)
     - [Templates with Types (Policy Classes)](#templates-with-types-policy-classes)
     - [Why Template Customizability Matters](#why-template-customizability-matters)
4. [Comparison: When to Use What](#comparison-when-to-use-what)
5. [Modern C++ Best Practices](#modern-c-best-practices)
6. [Key Takeaways](#key-takeaways)

---

## Designing for Reusability in Modern C++

Reusable and maintainable code is a core principle of modern software development. Whether you're building a small class or a full subsystem, design your code so it can be used again—by you or by other developers. This idea follows well-known principles:

* **Write once, use often.**
* **Avoid duplication.**
* **DRY — Don’t Repeat Yourself.**

### Why Reusability Matters

1. **You will need the same functionality again.**
   Reusable code saves you from rewriting the same logic in future projects.

2. **It reduces long-term effort.**
   A clean design prevents the need to reinvent solutions later.

3. **It supports teamwork.**
   Reusable components make it easier for others to integrate and understand your work.

4. **It avoids maintenance issues.**
   Duplicated code creates multiple points of failure. Centralizing logic prevents this.

5. **You benefit the most.**
   Over time, reusable components form a personal toolkit that speeds up future development.

---

## The Strategy Pattern: Overview

The strategy design pattern is one way to support the dependency inversion principle (DIP). With this pattern, interfaces are used to invert dependency relationships. Interfaces are created for every provided service. If a component needs a set of services, interfaces to those services are injected into the component—a mechanism called dependency injection. Using the strategy pattern makes unit testing easier, as you can easily mock services away. As an example, this section discusses a logging mechanism implemented with the strategy pattern.

The idea is:

**Write your code so users can change its behavior without modifying the code itself.**

So instead of hard-coding how something works, you let the caller choose how they want it to behave.

There are several ways to do this in C++.

---

## Implementation Approaches

### 3.1 Customizability through Interfaces

This classic object-oriented approach uses runtime polymorphism via abstract base classes.

#### Dependency Inversion vs Dependency Injection

These two concepts are related but **not the same**. One is a **principle**, the other is a **technique / implementation**.

##### 1. Dependency Inversion Principle (DIP)

The **Dependency Inversion Principle** is a **design principle** that states:

1. **High-level modules should not depend on low-level modules. Both should depend on abstractions.**
2. **Abstractions should not depend on details. Details should depend on abstractions.**

```cpp
class ILogger {   // abstraction
public:
    virtual void log(const std::string& message) = 0;
    virtual ~ILogger() = default;
};

// High-level module
class Foo {
    ILogger* logger;   // depends on abstraction
public:
    explicit Foo(ILogger* l) : logger(l) {}
    void doSomething() { logger->log("Hello DIP!"); }
};
```

##### 2. Dependency Injection (DI)

**Dependency Injection** is a **technique to implement DIP**. It means **providing dependencies from outside instead of creating them inside**.

```cpp
class ConsoleLogger : public ILogger {
public:
    void log(const std::string& message) override {
        std::cout << message << "\n";
    }
};

int main() {
    ConsoleLogger consoleLogger;
    Foo foo(&consoleLogger);  // Inject dependency
    foo.doSomething();
}
```

##### 3. Key Differences

| Aspect | Dependency Inversion (DIP) | Dependency Injection (DI) |
|--------|---------------------------|---------------------------|
| **Type** | Design principle | Implementation technique |
| **Goal** | High-level modules depend on abstractions | Supply dependencies externally to a class/module |
| **Scope** | Conceptual | Practical implementation |
| **Example** | `Foo` depends on `ILogger` instead of `ConsoleLogger` | `Foo` receives a `ConsoleLogger` instance via constructor |

**In short**: DIP tells you *what* to do ("depend on abstractions"), DI shows *how* to do it ("pass the dependency in").

#### Modern Implementation Example

Instead of doing this:

```cpp
class MyComponent {
private:
    ErrorLogger logger; // fixed logger (can't change!)
};
```

You do this:

```cpp
class ErrorLogger {
public:
    virtual void log(const std::string& msg) = 0;
    virtual ~ErrorLogger() = default;
};

class MyComponent {
private:
    ErrorLogger* logger;   // accepts any logger
public:
    MyComponent(ErrorLogger* l) : logger(l) {}

    void run() {
        logger->log("Running...");
    }
};
```

Now **anyone can inject their own logger**:

```cpp
class FileLogger : public ErrorLogger {
public:
    void log(const std::string& msg) override {
        std::cout << "File log: " << msg << '\n';
    }
};

int main() {
    FileLogger fl;
    MyComponent c(&fl); // inject the logging behavior
    c.run();
}
```

This is called **Dependency Injection (DI)**. Clients choose **how** logging happens.

---

### 3.2 Customizability through Callbacks

In C++, a **callback** is a **function that you pass to another function or object to be called later**. This allows you to **customize behavior** without modifying the original code. Callbacks are widely used in event handling, error handling, asynchronous programming, and implementing **strategic design patterns**.

#### What Is a Callback?

A callback is:

* A piece of code (function, lambda, or object with `operator()`)
* Passed to another function or class
* Called **later by that function/class** when some action occurs

Example in plain words: Imagine you hire a chef (callback) and tell a waiter (your function) to "call the chef when an order arrives." You don't care how the chef cooks; the waiter just calls them at the right time.

#### Types of Callbacks in C++

C++ supports three main kinds of callbacks:

##### a) Function Pointer Callback

A **function pointer** is a pointer that stores the address of a function. The function must have a fixed signature.

```cpp
#include <iostream>

void myCallback(int x) {
    std::cout << "Function pointer callback: " << x << "\n";
}

void doWork(void (*callback)(int)) {
    callback(42); // call the callback
}

int main() {
    doWork(myCallback);
}
```

**Key points:**
* Works with **normal functions** only
* Simple and efficient
* Cannot capture external variables

##### b) Function Object Callback (Functor)

A **function object** is an object that defines the `operator()`. It behaves like a function.

```cpp
#include <iostream>

struct MyFunctor {
    void operator()(int x) const {
        std::cout << "Function object callback: " << x << "\n";
    }
};

template <typename Callback>
void doWork(Callback cb) {
    cb(42); // call the callback
}

int main() {
    MyFunctor f;
    doWork(f);
}
```

**Key points:**
* Can store **state** (member variables)
* Works with templates
* More flexible than function pointers

##### c) Lambda Callback

A **lambda** in C++ is an **inline anonymous function** that can be defined **at the point of use**. Lambdas are a modern C++ favorite for implementing callbacks because they are concise, flexible, and can capture variables from their surrounding scope.

**Key characteristics:**
1. **Anonymous**: No separate function declaration is required
2. **Inline**: Defined at the place you need it
3. **Captures variables**: By value `[var]` or by reference `[&var]`
4. **Flexible parameters**: Can take zero or more parameters, including references
5. **Works with `std::function`**: Can store or pass lambdas anywhere a callable is expected

###### Basic Lambda Example

```cpp
#include <iostream>
#include <functional>

void doWork(const std::function<void(int)>& callback) {
    callback(42); // call the lambda
}

int main() {
    int factor = 10;
    doWork([factor](int x) {
        std::cout << "Lambda callback: " << x * factor << "\n";
    });
}
```

* `[factor]` → captures `factor` **by value** (a copy)
* `(int x)` → the lambda accepts **one parameter**

###### Lambda with Multiple Parameters and References

```cpp
#include <iostream>
#include <functional>

void doWork(const std::function<void(int&, int&)>& callback) {
    int a = 5, b = 10;
    callback(a, b); // call the lambda
    std::cout << "After callback: a = " << a << ", b = " << b << "\n";
}

int main() {
    int factor = 3;
    doWork([&factor](int& x, int& y) {
        x *= factor;
        y *= factor;
    });
}
```

**Output:**
```
After callback: a = 15, b = 30
```

* `[&factor]` → captures `factor` **by reference**
* `int& x, int& y` → parameters passed **by reference**, allowing modification

#### Why Use `std::function`?

* Wraps **any callable type** (function, lambda, functor)
* Can store the callback internally
* Supports **dynamic polymorphism for callbacks**

```cpp
void doWork(const std::function<void(int)>& callback) {
    callback(42); // call whatever was passed
}
```

The `const &` avoids copying the callable.

#### Callbacks in the Strategy Design Pattern

The **Strategy pattern** allows selecting an algorithm's behavior at runtime.

```cpp
#include <iostream>
#include <functional>

class Processor {
public:
    using ErrorHandler = std::function<void(const std::string&)>;

    Processor(ErrorHandler handler)
        : errorHandler(handler) {}

    void run() {
        errorHandler("Something went wrong");
    }

private:
    ErrorHandler errorHandler;
};

int main() {
    Processor p([](const std::string& msg){
        std::cerr << "[Custom Error] " << msg << "\n";
    });
    p.run(); // calls the lambda strategy
}
```

**Key points:**
* The `Processor` class **does not know** how errors are handled
* The client **injects the strategy** (callback)
* You can easily swap different strategies without changing the `Processor` code

#### Summary Table

| Callback Type    | Syntax / Example                             | Notes                                   |
| ---------------- | -------------------------------------------- | --------------------------------------- |
| Function Pointer | `void (*cb)(int)`                            | Works with plain functions              |
| Function Object  | `struct Functor { void operator()(int){} };` | Can store state, works with templates   |
| Lambda           | `[](int x){}`                                | Can capture variables, concise          |
| `std::function`  | `std::function<void(int)> cb`                | Most flexible, wraps all callable types |

**Use in Strategy Pattern:**
* The callback represents a **strategy (behavior)**
* Injecting different callbacks changes behavior **at runtime**
* Encourages **reusability and clean code**

---

### 3.3 Customizability through Template Parameters

C++ templates allow you to write **generic, flexible, and reusable code**. With templates, the **caller can customize types, behaviors, or policies** without modifying the implementation. This approach enables both **compile-time strategies** and **behavior injection**.

Templates enable two main types of customization:
1. **Behavior injection via callables** (function pointers, functors, lambdas)
2. **Behavior injection via types / policy classes** (strategy-style customization)

#### Templates with Callable Parameters

Templates can accept **callable objects** to define behavior dynamically.

```cpp
#include <iostream>
#include <algorithm>
#include <vector>
#include <functional>

// Template accepting any callable as a callback
template <typename Compare>
void sortVector(std::vector<int>& v, Compare comp) {
    std::sort(v.begin(), v.end(), comp);
}

int main() {
    std::vector<int> v = {1, 5, 3, 2};

    // Lambda as a callback
    int factor = 10;
    sortVector(v, [factor](int a, int b) {
        return (a * factor) < (b * factor);
    });

    // Functor as a callback
    struct Descending {
        bool operator()(int a, int b) const { return a > b; }
    };
    sortVector(v, Descending());

    // Function pointer as a callback
    auto ascending = [](int a, int b) { return a < b; };
    sortVector(v, ascending);
}
```

**Key points:**
* **Callbacks** define how the template behaves: function pointers, functors, or lambdas
* Multiple parameters and references are fully supported
* **Type-safe** and efficient

#### Templates with Types (Policy Classes)

Templates can accept **types** as parameters, allowing compile-time strategy injection.

```cpp
#include <vector>
#include <memory>
#include <iostream>

// Custom allocator policy
template <typename T>
class MyAllocator {
public:
    using value_type = T;
    MyAllocator() = default;

    T* allocate(std::size_t n) {
        std::cout << "Allocating " << n << " elements\n";
        return static_cast<T*>(::operator new(n * sizeof(T)));
    }

    void deallocate(T* ptr, std::size_t n) {
        std::cout << "Deallocating " << n << " elements\n";
        ::operator delete(ptr);
    }
};

// Vector template with customizable allocator
template <typename T, typename Allocator = std::allocator<T>>
using MyVector = std::vector<T, Allocator>;

int main() {
    MyVector<int> defaultVec;                 // default allocator
    MyVector<int, MyAllocator<int>> customVec; // custom allocator
}
```

**Explanation:**
1. `MyAllocator<int>` is a **policy class** defining memory allocation strategy
2. `MyVector` delegates allocation to the **provided type**
3. This is a **compile-time Strategy Pattern**: the container relies on a type that implements a specific interface
4. The caller can inject **different behaviors** without modifying the template

#### Why Template Customizability Matters

* **Extreme flexibility**: Fully change behavior at compile time without changing code
* **Compile-time efficiency**: Templates avoid runtime overhead (no virtual calls)
* **Reusability**: Same template can be used with different behaviors or policies
* **Stateful strategies**: Functors or lambdas can store state while maintaining efficiency
* **Supports modern C++ design patterns**: Implements **strategy, policy-based design, and dependency injection** at compile time

---

## Comparison: When to Use What

| Approach | Runtime Overhead | Flexibility | Testability | Use Case |
|----------|------------------|-------------|-------------|----------|
| **Interfaces (DI)** | Virtual calls (minimal) | High | Excellent | Large systems, mocking required |
| **Callbacks** | Type-erasure (`std::function`) | Very High | Good | Event handlers, small strategies |
| **Templates** | None (compile-time) | Medium* | Harder† | Performance-critical, policy-based |

*Flexibility: Strategy fixed at compile time  
†Testing: Requires template-heavy test frameworks or explicit instantiation

**Decision Tree:**
```plaintext
Need runtime polymorphism? → Yes → Use Interfaces
                                    ↓
Need maximum performance? → Yes → Use Templates
                                    ↓
Need quick, inline behavior? → Yes → Use Callbacks/Lambdas
```

---

## Modern C++ Best Practices

### ✅ Do's

- **Use `std::unique_ptr` for ownership**: Clear, safe, self-documenting
- **Prefer `std::function` for callbacks**: Type-erased, flexible
- **Leverage lambdas**: Capture state concisely with `[&]` or `[=]`
- **Apply `std::move`**: Efficiently transfer ownership of strategies
- **Document with `[[nodiscard]]`**: For factory functions
- **Concepts (C++20)**: Constrain template parameters
  ```cpp
  template <typename T>
  concept Logger = requires(T t, std::string_view msg) {
      { t.log(msg) } -> std::same_as<void>;
  };
  ```

### ❌ Don'ts

- **Raw pointers for ownership**: Use smart pointers
- **Static/global state**: Makes testing impossible
- **Overuse `std::function`**: For simple templates, prefer direct callable types
- **Capture all by reference `[&]`**: Be explicit about captures
- **Forget virtual destructors**: Base classes must have `virtual ~Base() = default;`

---

## Key Takeaways

1. **Strategy Pattern = Dependency Injection**: Decouple behavior from implementation.
2. **Modern C++ = Multiple Tools**: Interfaces, callbacks, and templates are complementary.
3. **Smart Pointers are Mandatory**: Use `unique_ptr` for DI; use `shared_ptr`/`weak_ptr` for observers.
4. **Lambdas are King**: For 90% of callbacks, lambdas provide the best balance of clarity and power.
5. **Performance Matters**: Use templates when virtual calls are unacceptable.
6. **Testability is Crucial**: Design strategies to be mockable from day one.

---

## 📚 Resources

- **Book**: *Professional C++ 6th Edition* (Marc Gregoire) - Chapter on Design Patterns
- **C++ Core Guidelines**: [R.11: Avoid calling `new` and `delete` explicitly](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines#R11)
- **Pattern Catalog**: [Refactoring.Guru - Strategy Pattern](https://refactoring.guru/design-patterns/strategy)
- **Testing**: Use **GoogleMock** with interface-based strategies

---

## 🎯 Example: Putting It All Together

```cpp
// Modern, flexible, testable error handler
class ErrorHandler {
public:
    virtual ~ErrorHandler() = default;
    virtual void handle(std::string_view error) = 0;
};

class Application {
public:
    // Injects both interface and callback for maximum flexibility
    explicit Application(
        std::unique_ptr<ErrorHandler> handler,
        std::function<void()> onStartup = []{})
        : errorHandler_(std::move(handler))
        , startupCallback_(std::move(onStartup)) {}

    void run() {
        startupCallback_();
        try {
            // ... app logic ...
        } catch (const std::exception& e) {
            errorHandler_->handle(e.what());
        }
    }

private:
    std::unique_ptr<ErrorHandler> errorHandler_;
    std::function<void()> startupCallback_;
};
```