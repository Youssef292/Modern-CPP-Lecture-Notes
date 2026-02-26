# Generic Handling, Smart Pointers & System Design – Professional Notes

## Agenda

* Generic handling
* Smart pointers

  * `unique_ptr`
  * `shared_ptr`
* Modularity, linking, and system design

---

# 1) Generic Handling

## Generic Programming in C

### Compile-Time Generic (C)

* Achieved using **macros**
* Compared to **templates in C++**
* Weak type safety
* Debugging can be difficult

### Run-Time Generic (C)

* Achieved using `void*` (void pointer)
* `void*` is called an **incomplete type**
* Requires manual casting
* Not type-safe
* Error-prone and requires programmer discipline

### Example: Dynamic Array API in C

```c
typedef struct
{
    void* data;
    int size;
    int capacity;
    int elementSize; // Critical for generic handling
} DA_T;

// Example conceptual usage
// DA_T arr = CreateArray(sizeof(int));

void array_get(DA_T* arr, int index, void* output)
{
    char* base = (char*)arr->data; // byte-level access
    void* elementPtr = base + (index * arr->elementSize);
    memcpy(output, elementPtr, arr->elementSize);
}
```

Key Idea:

* Casting to `char*` allows byte-wise pointer arithmetic.
* `elementSize` enables generic memory manipulation.
* Still unsafe because type correctness depends on user discipline.

---

# Generic Solution in C++

C++ provides safer alternatives:

* Templates (compile-time generic)
* `std::any` (run-time generic abstraction)

---

## std::any

Header:

```cpp
#include <any>
```

* Abstraction over `void*`
* Type-safe wrapper
* Uses RTTI (Run-Time Type Information)
* Stores any copyable type

Example:

```cpp
std::any x = 10;

x = 3.15;
x = std::string("string");
```

### Reading from std::any

Casting is mandatory:

```cpp
std::any x = 10;
int value = std::any_cast<int>(x);
```

If type does not match → throws `std::bad_any_cast`.

---

## RTTI (Run-Time Type Information)

* Allows querying type at runtime
* Used via:

  * `typeid()`
  * `std::any::type()`

Example:

```cpp
std::cout << x.type().name() << std::endl;
```

---

## Difference Between auto and std::any

* `auto`:

  * Compile-time type deduction
  * Type is fixed after initialization
  * No runtime polymorphism

* `std::any`:

  * Type stored at runtime
  * Can change type dynamically
  * Uses internal storage (small object optimization):

    * Small objects → stored inline (stack-like)
    * Large objects → allocated on heap

---

## Template Wrapper Around std::any

```cpp
template <typename T>
T anyget(std::any x)
{
    T d{};
    if (x.type() == typeid(T))
    {
        std::cout << "any data: " << std::any_cast<T>(x) << std::endl;
        d = std::any_cast<T>(x);
        return d;
    }
    return d;
}
```

Usage:

```cpp
std::any x = 10;
std::cout << "any int " << std::any_cast<int>(x) << std::endl;

x = 1.1;
std::cout << "any now has type: " << std::string(x.type().name()) << std::endl;

double dd = anyget<double>(x);

x = std::string("Hello");
std::string s = anyget<std::string>(x);
```

Note:

* Return type alone is not part of function signature.
* Template parameter ensures correct type resolution.

---

## std::vector[std::any](std::any) Example

```cpp
std::vector<std::any> any_vec;

any_vec.push_back(10);                     // int
any_vec.push_back(10.3);                   // double
any_vec.push_back('A');                    // char
any_vec.push_back(std::string("Ahmed"));  // std::string

for (auto& element : any_vec)
{
    if (element.type() == typeid(int))
        std::cout << std::any_cast<int>(element) << " ";
    else if (element.type() == typeid(double))
        std::cout << std::any_cast<double>(element) << " ";
    else if (element.type() == typeid(char))
        std::cout << std::any_cast<char>(element) << " ";
    else if (element.type() == typeid(std::string))
        std::cout << std::any_cast<std::string>(element) << " ";
}
```

Key Takeaways:

* `std::any` enables runtime polymorphism without inheritance.
* Requires explicit type checks and casts.
* Safer and more expressive than raw `void*`.

---

# Compile-Time Generic Typing & Variant Type – Organized Notes

---

# 1) Compile-Time Generic Typing (Tagged Union Idea)

```cpp
struct
{
    data_t data;
    std::string tag;
} data_t;

enum { INT, FLOAT, STRING };
```

## Concept

* This approach simulates a **tagged union**.
* A tag (enum) determines the active stored type.
* The structure contains:

  * The actual data
  * A tag describing the stored type

## Problem With This Approach

* Requires manual getter and setter APIs.
* Programmer must ensure tag matches stored data.
* Error-prone.
* No automatic type safety.

This idea evolved into what is known as a **variant type**.

---

# 2) Variant Type

```cpp
#include <variant>

// tuple is the template generalization for struct
// variant is the template generalization for union

std::variant<int, float, std::string> var;

var = 43; // int selected at compile time

// Access methods
std::get<index>();
std::get<datatype>();

std::get<0>(var);
std::get<int>(var);
```

---

## Variant APIs

### 1) index()

```cpp
var.index();
```

* Returns the index of the currently active type.
* Provides compile-time type information (CTTI concept).

---

### 2) std::get

* Throws exception: `std::bad_variant_access` if wrong type accessed.

---

### 3) std::get_if

```cpp
std::get_if<int>(&var);
```

* Returns pointer to stored value.
* Returns `nullptr` if type does not match.

```cpp
int* i = std::get_if<int>(&var);
if (i != nullptr)
{
    // safe to use *i
}
```

---

### 4) std::holds_alternative

```cpp
std::holds_alternative<int>(var);
```

* Returns `true` or `false`.
* Used to check active type safely.

---

## Example

```cpp
std::variant<int,float,std::string> var;

var = 10;
std::cout << "has int? "
          << (std::holds_alternative<int>(var) ? true : false)
          << std::endl;

var = "hello";
std::cout << "has int? "
          << (std::holds_alternative<int>(var) ? true : false)
          << std::endl;

if (std::holds_alternative<std::string>(var))
{
    std::cout << "The String in variant is: "
              << std::get<std::string>(var)
              << std::endl;
}
```

---

# 3) Variant Access Example (Visitor Pattern)

```cpp
struct Point2D
{
    int x, y;
};

struct Point3D
{
    int x, y, z;
};

struct PointVisitor
{
    // overload the call operator for each data type

    void operator()(Point2D p2d) const
    {
        std::cout << " ( "
                  << p2d.x << ","
                  << p2d.y << " ) "
                  << std::endl;
    }

    void operator()(Point3D p3d) const
    {
        std::cout << " ( "
                  << p3d.x << ","
                  << p3d.y << ","
                  << p3d.z << " ) "
                  << std::endl;
    }
};

int main()
{
    // visitor pattern
    std::variant<Point2D, Point3D> var_point;

    Point2D p1 = {10, 20};
    Point3D p2 = {10, 20, 30};

    var_point = p1;
    std::visit(PointVisitor(), var_point);

    var_point = p2;
    std::visit(PointVisitor(), var_point);

    return 0;
}
```

## Key Idea

* `std::visit` applies the correct overload.
* No inheritance required.
* Compile-time polymorphism.

---

# 4) Practical Usage

### Example: Data Serialization

* Storing different data types safely.
* Type-checked access.

### Project Idea

Application that converts between different data types.

---

## Example: Generic Visitor with Lambda

```cpp
int main()
{
    typedef std::variant<int, float, std::string> Variant;
    std::vector<Variant> var_vec;

    std::string v;
    int num;
    float fnum;
    std::string str;

    while (std::getline(std::cin, v))
    {
        std::stringstream ss(v);
        ss >> num >> fnum >> str;
    }

    var_vec.push_back(num);
    var_vec.push_back(fnum);
    var_vec.push_back(str);

    auto visitor = [](auto &v)
    {
        std::cout << "I am "
                  << typeid(v).name()
                  << " datatype"
                  << std::endl;
    };

    std::for_each(var_vec.begin(),
                  var_vec.end(),
                  [&](Variant& v)
                  {
                      std::visit(visitor, v);
                  });

    return 0;
}
```

---

# 5) Comparison Table

| Feature        | std::variant                   | std::any                 |
| -------------- | ------------------------------ | ------------------------ |
| Type Set       | Fixed at compile time          | Any type                 |
| Type Safety    | Strong compile-time guarantees | Runtime type checking    |
| Access Method  | std::visit / std::get          | any_cast                 |
| Error Handling | bad_variant_access             | bad_any_cast             |
| Performance    | Typically faster               | Slightly slower          |
| Use Case       | Known alternatives             | Completely dynamic types |

# Smart Pointers

## What is the Problem with Raw Pointers?

* Ownership does not exist.
* It is unclear which function or object is responsible for deleting the pointer.

### Example

```cpp
void fun1(int* ptr)
{
  // some code
}

void fun2(int* ptr)
{
  // some code
  // deleted the ptr
}
```

* Function 2 acted as the owner of the pointer.
* This may lead to a segmentation fault (double delete or invalid access).

---

## How to Achieve Ownership?

### Features in C++ That Help Avoid This Problem:

1. Template classes and abstraction
2. RAII Pattern (Resource Acquisition Is Initialization)

---

# Types of Smart Pointers

## 1) `std::unique_ptr`

* Acts as the single owner of an object pointer.
* Only one `unique_ptr` can own a resource at a time.

### Basic Usage

```cpp
std::unique_ptr<int> un_int;
un_int = std::make_unique<int>();
*un_int = 10;

struct point
{
  int x, y;

  void display() {
    std::cout << x << " " << y;
  }
};

std::unique_ptr<point> un_point = std::make_unique<point>(point{2,3});
un_point->x = 5;
un_point->display();
```

---

## Benefits

### 1) Ownership is Exclusive

* Since ownership is with `unique_ptr` → object is not copyable.

```cpp
#include <memory>
#include <utility>

std::unique_ptr<std::string> un_str1 = std::make_unique<std::string>("Hello");
std::unique_ptr<std::string> un_str2;

// un_str2 = un_str1;
// Compiler error (copy constructor is deleted)

// To transfer ownership:
un_str2 = std::move(un_str1);

// After move:
// un_str2 points to "Hello"
// un_str1 becomes null

// Accessing raw pointer (for C API cases)
un_str1.get();

// If manually freed, it is programmer responsibility
free(un_str1.get());

// In normal raw pointers:
// Two pointers may point to the same object.
// If one deletes it, the other becomes dangling.
```

---

## Example 1

```cpp
#include <iostream>
#include <memory>
#include <sstream>

struct Point3D
{
    int x, y, z;

    std::string display()
    {
        std::stringstream ss;
        ss << " ( " << x << "," << y << "," << z << "," << ")";
        return ss.str();
    }
};

int main()
{
    std::unique_ptr<int> un_int = std::make_unique<int>();
    std::cout << *un_int << std::endl;
    std::cout << un_int.get() << std::endl;
    std::cout << un_int.get() << std::endl;

    std::unique_ptr<int> un_int2;

    // un_int = un_int2;
    // Compile error: operator deleted

    std::unique_ptr<Point3D> un_p3d(new Point3D); // Fair but not recommended
    std::cout << "Point is : " << un_p3d->display() << std::endl;

    std::unique_ptr<Point3D> un_p3d2 = std::move(un_p3d);
    std::cout << "Point is : " << un_p3d2->display() << std::endl;

    if (!un_p3d)
        std::cout << "Old ptr is now null" << std::endl;

    return 0;
}
```

---

## Example 2

```cpp
#include <iostream>
#include <memory>

class CUARTPort {
public:
    int data1 = 10;
    int data2 = 20;

    void Communicate() {
        std::cout << data1 << " " << data2 << std::endl;
    }
};

class CUARTManager {
public:
    std::unique_ptr<CUARTPort> un_uartport = std::make_unique<CUARTPort>();
};

int main()
{
    CUARTManager manager;
    manager.un_uartport->Communicate();

    return 0;
}
```


# Shared Pointers

## 2) `std::shared_ptr`

```cpp
std::shared_ptr<int> sh_int = std::make_shared<int>(20);
std::shared_ptr<int> sh_int2 = sh_int;

// example garbage collector idea (try as a project)
/*
  inside the shared pointer there is a reference counter.
  each time a new assignment like sh_int2 = sh_int happens,
  the reference counter increments.
*/

std::cout << sh_int.use_count();
```

---

## Concept

* `std::shared_ptr` allows **multiple owners** of the same resource.
* Internally maintains a **reference count**.
* When a new shared pointer copies the same resource → reference count increases.
* When a shared pointer is destroyed or reset → reference count decreases.
* When reference count reaches zero → memory is automatically deleted.

---

## Example

```cpp
#include <iostream>
#include <memory>

int main()
{
    std::shared_ptr<int> sh_int = std::make_shared<int>(20);
    std::shared_ptr<int> sh_int2 = sh_int;

    std::cout << "ref count: " << sh_int.use_count() << std::endl;
    std::cout << "ref count: " << sh_int2.use_count() << std::endl;
    //* output is 2 in both

    std::shared_ptr<int> sh_int3(new int(50));
    std::cout << *sh_int3 << " Ref count "
              << sh_int3.use_count() << std::endl;
    //^ shared count will still be 1 because it did not share with another shared pointer

    sh_int2.reset();
    if (!sh_int2)
        std::cout << "sh_int2 is now null and ref: "
                  << sh_int.use_count()
                  << std::endl;
    //! reference count reset to one

    return 0;
}
```

---

## Notes

* `use_count()` returns the current reference count.
* `reset()` decreases the reference count and releases ownership.
* When last owner releases the resource → memory is deleted automatically.
* Can be used to simulate a simple garbage collector idea as a project.
* Commonly used in multithreading environments where shared ownership is required.

# Namespace

## Definition

* Namespace is a feature in C++ introduced to solve the name collision problem.

---

## Name Collision Problem

```cpp
A
{
  int x;
}

B
{
  int x;
}

#include "A"
#include "B"

int main
{
  x; // collision will happen
}
```

### Namespace Solves It

```cpp
namespace A
{
  int x;
}

namespace B
{
  int x;
}

#include "A"
#include "B"

int main
{
  A::x;
}
```

---

## Benefits of Namespace

1. Solves name collision
2. Code becomes more structured
3. Improves readability

---

## Example

```cpp
namespace MyApp
{
    int x = 1;

    void print()
    {
        std::cout << "My App" << std::endl;
    }

    class mydata
    {
    public:
        static void data()
        {
            std::cout << "Class" << std::endl;
        }
    };
};

void MyApp::print()
{
    // body
}

namespace MyApp   // Expand namespace
{
    void stop()
    {

    }
}

namespace MyApp2
{
    void stop()
    {

    }
}

using MyApp::stop;
using namespace MyApp;
using namespace MyApp2;

int main()
{
    MyApp::x = 20;
    MyApp::print();

    MyApp::mydata c;
    c.data();
    MyApp::mydata::data();

    stop(); // lead to collision so don't use using namespace std blindly
    MyApp2::stop(); // correct usage

    return 0;
}
```

---

## Nested Namespaces

```cpp
namespace Company
{
    namespace Project
    {
        namespace Module
        {
            void run()
            {
                std::cout << "Nested" << std::endl;
            }
        }
    }
}

int main()
{
    Company::Project::Module::run(); // old style before C++17
    return 0;
}
```

### C++17 Nested Namespace Syntax

```cpp
namespace Company::Project::Module
{

}
```

---

## Anonymous Namespace

```cpp
namespace // anonymous namespace has internal linkage
{
    int x;
}

int main()
{
   x = 10;
   return 0;
}
```

---

## Namespace Alias

```cpp
namespace MYnamespace::Proj::mod
{
    void print()
    {

    }
}

namespace m = MYnamespace::Proj::mod; // alias namespace

int main()
{
   m::x = 10;
   return 0;
}
```

---

# Modularity

## Definition

* Modularity is a program design technique.
* Before modularity, all code was written in one file.

## Problems With Old Way

* Any change requires recompiling the whole project.
* Poor reusability.
* Dependency issues.

## In Modularity

* `interface.h` → header file for declarations.
* `source.c` → implementation file.

Now the project is separated into modules.

* Compiler recompiles only changed module.
* Provides abstraction to the project.

---

## Modularity Provides

### 1) Coupling

* All functions in file serve one goal.

### 2) Cohesion

* Each function does one thing.

---

## Example

### mod.hpp

```cpp
#pragma once

class MyData
{
public:
    MyData(int x) : x(x) {}

    int getx();
    void setx(int x);

private:
    int x;
};
```

### Implementation File

```cpp
#include "mod.hpp"
#include <iostream>

int MyData::getx()
{
  return x;
}

void MyData::setx(int x)
{
  this->x = x;
}
```

---

# Storage Class

## Definition

* Describes specifics related to an object:

  1. Linkage
  2. Scope
  3. Storage definition (allocation)

---

## Linkage Types

1. External linkage
2. Internal linkage

---

## Static Library vs Dynamic Library

* Static library:

  * Example: iostream
  * Entire library code is included into the executable at build time.

* Dynamic library:

  * Loaded at runtime.

