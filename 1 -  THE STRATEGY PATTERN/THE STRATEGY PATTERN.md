# Designing for Reusability in Modern C++

Reusable and maintainable code is a core principle of modern software development. Whether you're building a small class or a full subsystem, design your code so it can be used again—by you or by other developers. This idea follows well-known principles:

* **Write once, use often.**
* **Avoid duplication.**
* **DRY — Don’t Repeat Yourself.**

## Why Reusability Matters

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

# The Strategy Pattern

The strategy design pattern is one way to support the dependency inversion principle (DIP) With this pattern, interfaces are used to invert dependency relationships. Interfaces are created for every provided service. If a component needs a set of services interfacesto those services are injected into the component, a mechanism called dependency injection. Using the strategy pattern makes unit testing easier, as you can easily mock services away. As an example, this section discusses a logging mechanism implemented with the strategy pattern.

The idea is:

**Write your code so users can change its behavior without modifying the code itself.**

So instead of hard-coding how something works, you let the caller choose how they want it to behave.

There are several ways to do this in C++:

---

# 1. **Customizability through Interfaces (Dependency Inversion + Dependency Injection)**

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

This is called **Dependency Injection (DI)**.
Clients choose **how** logging happens.

---

# 2. **Customizability through Callbacks**

# **Callbacks in Modern C++**

In C++, a **callback** is a **function that you pass to another function or object to be called later**. This allows you to **customize behavior** without modifying the original code. Callbacks are widely used in event handling, error handling, asynchronous programming, and implementing **strategic design patterns**.

---

## **1. What Is a Callback?**

A callback is:

* A piece of code (function, lambda, or object with `operator()`)
* Passed to another function or class
* Called **later by that function/class** when some action occurs

Example in plain words:

> Imagine you hire a chef (callback) and tell a waiter (your function) to “call the chef when an order arrives.” You don’t care how the chef cooks; the waiter just calls them at the right time.

---

## **2. Types of Callbacks in C++**

C++ supports three main kinds of callbacks:

---

### **a) Function Pointer Callback**

A **function pointer** is a pointer that stores the address of a function. The function must have a fixed signature (arguments and return type).

#### Example:

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
* Cannot capture external variables like lambdas

---

### **b) Function Object Callback (Functor)**

A **function object** is an object that defines the `operator()`. It behaves like a function.

#### Example:

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
* Works with templates (like `doWork<Callback>`)
* More flexible than function pointers

---

### **c) Lambda Callback**

A **lambda** in C++ is an **inline anonymous function** that can be defined **at the point of use**. Lambdas are a modern C++ favorite for implementing callbacks because they are concise, flexible, and can capture variables from their surrounding scope.

**Key characteristics of lambdas:**

1. **Anonymous**: No separate function declaration is required.
2. **Inline**: Defined at the place you need it.
3. **Captures variables**: You can capture variables from the surrounding scope **by value** (`[var]`) or **by reference** (`[&var]`).
4. **Flexible parameters**: Can take zero or more parameters, including references.
5. **Works with `std::function`**: You can store or pass lambdas anywhere a callable is expected.

---

#### **Basic Example**

```cpp
#include <iostream>
#include <functional>

void doWork(const std::function<void(int)>& callback) {
    callback(42); // call the lambda
}

int main() {
    int factor = 10;

    // Lambda captures 'factor' by value
    doWork([factor](int x) {
        std::cout << "Lambda callback: " << x * factor << "\n";
    });
}
```

* `[factor]` → captures `factor` **by value** (a copy).
* `(int x)` → the lambda accepts **one parameter**.
* Inside the lambda, `x` is multiplied by the captured `factor` and printed.

---

#### **Lambda with Multiple Parameters and References**

You can also define lambdas that accept multiple parameters, and even pass them **by reference**:

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

    // Lambda captures 'factor' by reference and parameters by reference
    doWork([&factor](int& x, int& y) {
        x *= factor;
        y *= factor;
    });

    return 0;
}
```

**Output:**

```
After callback: a = 15, b = 30
```

* `[&factor]` → captures `factor` **by reference**, so any changes inside the lambda affect the original variable.
* `int& x, int& y` → parameters passed **by reference**, allowing the lambda to modify `a` and `b` directly.

---

#### **Use in Strategy Design Pattern**

* Lambdas are perfect for the **Strategy pattern**, where a class delegates behavior to a callback.
* You can inject different behaviors at runtime without changing the class.

```cpp
#include <functional>
#include <iostream>

class Processor {
public:
    using Strategy = std::function<void(int&, int&)>;
    Processor(Strategy s) : strategy(s) {}
    
    void run() {
        int a = 2, b = 5;
        strategy(a, b);
        std::cout << "Result: " << a << ", " << b << "\n";
    }

private:
    Strategy strategy;
};

int main() {
    int factor = 10;
    Processor p([&factor](int& x, int& y) {
        x *= factor;
        y *= factor;
    });
    
    p.run(); // Calls the lambda strategy
}
```

* The `Processor` class **doesn’t know the details** of the strategy.
* The lambda provides a **custom behavior** injected at runtime.

---

#### **Key Points**

* Lambda functions are **concise and inline**.
* Can **capture external variables** by value (`[var]`) or reference (`[&var]`).
* Accept **multiple parameters**, including references.
* **Works seamlessly with `std::function`**, allowing maximum flexibility.
* Ideal for **strategy pattern implementations**, callbacks, and customizable behavior.

---

## **3. Why Use `std::function`?**

* It wraps **any callable type** (function, lambda, functor)
* Can store the callback internally
* Supports **dynamic polymorphism for callbacks**

Example:

```cpp
void doWork(const std::function<void(int)>& callback) {
    callback(42); // call whatever was passed
}
```

* The `const &` avoids copying the callable
* The caller can pass any type of callback

---

## **4. Callbacks in the Strategy Design Pattern**

The **Strategy pattern** is a **behavioral design pattern** that allows selecting an algorithm's behavior at runtime.

* Instead of hardcoding behavior, you **inject it as a callback**.
* Your main class delegates certain operations to the **callback (strategy)**.

#### Example: Error handling strategy

```cpp
#include <iostream>
#include <functional>

class Processor {
public:
    using ErrorHandler = std::function<void(const std::string&)>;

    Processor(ErrorHandler handler)
        : errorHandler(handler) {}

    void run() {
        // some logic
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

---

## **5. Summary Table**

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

**Takeaways**

1. Callbacks let your code be **flexible and reusable**.
2. Function pointers are simple but limited.
3. Functors are more flexible and can store state.
4. Lambdas are concise and capture variables.
5. `std::function` provides maximum flexibility.
6. Using callbacks is perfect for implementing the **Strategy design pattern**.

---

## 3. Customizability through Template Parameters

C++ templates allow you to write **generic, flexible, and reusable code**. With templates, the **caller can customize types, behaviors, or policies** without modifying the implementation. This approach enables both **compile-time strategies** and **behavior injection**, making templates a powerful tool for modern C++ design.

Templates enable two main types of customization:

1. **Behavior injection via callables** (function pointers, functors, lambdas).
2. **Behavior injection via types / policy classes** (strategy-style customization).

---

### **1. Templates with Callable Parameters**

Templates can accept **callable objects** (functions, function pointers, functors, lambdas) to define behavior dynamically.

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
        return (a * factor) < (b * factor); // captured variable, custom behavior
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

* **Callbacks** define how the template behaves. They can be:

  * **Function pointers**: simple, cannot store state.
  * **Functors** (objects with `operator()`): can store state and be reused.
  * **Lambdas**: concise, inline, can capture variables by value `[factor]` or reference `[&factor]`, support multiple parameters and references.
* Multiple parameters and references are fully supported, e.g.:

```cpp
auto multiplyAndCompare = [](int& a, int& b, int factor) {
    return (a * factor) < (b * factor);
};
```

This allows templates to handle **dynamic, caller-defined behavior** while remaining **type-safe** and efficient.

---

### **2. Templates with Types (Policy Classes / Strategy Types)**

Templates can also accept **types** as parameters, allowing the caller to inject a **behavior policy** or **strategy** at compile time.

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

1. `MyAllocator<int>` is a **policy class** defining memory allocation strategy.
2. `MyVector` delegates allocation to the **provided type**. The container itself doesn’t change, only its behavior does.
3. This is a **compile-time Strategy Pattern**: the container relies on a type that implements a specific interface (allocate/deallocate).
4. The caller can inject **different behaviors** (default allocator, custom allocator, logging strategy, error-handling strategy) without modifying the template.

---

### **3. Why Template Customizability Matters**

* **Extreme flexibility**: You can fully change behavior at compile time without changing the code.
* **Compile-time efficiency**: Templates avoid runtime overhead, unlike virtual function-based strategies.
* **Reusability**: Same template can be used with different behaviors or policies.
* **Stateful strategies**: Functors or lambdas can store state, giving dynamic control while maintaining efficiency.
* **Supports modern C++ design patterns**: Templates can implement **strategy, policy-based design, and dependency injection** at compile time.

---

### **4. Real-world Examples**

* **STL algorithms** like `std::sort` take custom comparators.
* **STL containers** allow custom memory allocators.
* **Custom logging or error handling**: define a logger policy and inject it into your classes/templates.
* **Mathematical or functional behavior**: inject custom functors/lambdas for calculation strategies.
