# `std::pair` in Data Structures

-   `std::pair` is a data structure commonly used inside associative
    containers like `std::map`, but it can also be used independently.
-   It is originally implemented as a **template struct**, not a class.
-   It contains two public data members:
    -   `first`
    -   `second`

## Example

``` cpp
std::pair<std::string, int> p;
p.first;
p.second;
```

------------------------------------------------------------------------

# When Two Elements Are Not Enough → `std::tuple`

If you need a template that:

-   Takes **any number of data types**
-   Preserves the **order** of those types
-   Accepts a **variable number of different arguments**

Then you use `std::tuple`.

`std::tuple` allows storing multiple heterogeneous values in a single
object.

------------------------------------------------------------------------

# Accessing Tuple Elements

To access elements in a `std::tuple`, use:

``` cpp
std::get<>()
```

-   `std::get<>` is a **template function** that operates on a tuple.
-   It can take:
    -   An **index**
    -   Or a **data type**

## Example

``` cpp
std::tuple<int, std::string, double> t;

std::get<0>(t);        // Access by index
std::get<int>(t);      // Access by type (if unique)
```

------------------------------------------------------------------------

# Use Cases

-   When you have a simple struct-like grouping of values.
-   When you want to return multiple values from a function.
-   One of the most popular use cases is returning a **result object**
    containing multiple pieces of data.
-   Useful when you don't want to define a custom struct just to group
    values temporarily.

------------------------------------------------------------------------

# Summary

-   `std::pair` → exactly two values (`first`, `second`)
-   `std::tuple` → any number of values, ordered, heterogeneous
-   `std::get<>` → used to access tuple elements (by index or type)

# `std::tuple` Initialization & Usage Examples

## Initialization Syntax

### Incorrect Example (Syntax Issue)

``` cpp
std::tuple<std::string , int , float> t = make_tuple<std::string,int,float>({"Youssef",30,123,33}); 
```

> ❌ This is incorrect: - `make_tuple` does not require explicit
> template arguments. - The number of arguments does not match the tuple
> type. - The brace initialization is invalid here.

------------------------------------------------------------------------

### Correct Initialization

``` cpp
std::tuple<std::string, int, float> t =
    std::make_tuple("Youssef", 30, 123.33f);

auto name   = std::get<0>(t);
auto age    = std::get<int>(t);
auto length = std::get<float>(t);

std::cout << name << " " << age << " " << length << std::endl;
```

------------------------------------------------------------------------

# Accessing Elements Inside a Loop (Common Error)

``` cpp
for(int i = 0; i < 4; i++)
{
    auto ele = std::get<i>(t); // ❌ ERROR
}
```

❌ This causes an error because:

-   `std::get<i>` requires a **compile-time constant**
-   `i` is a runtime variable
-   Template arguments must be known at compile time

------------------------------------------------------------------------

# Packing & Unpacking Syntax

## Structured Binding (Unpacking)

``` cpp
auto& [name, age, length] = t;
std::cout << name << " " << age << " " << length << std::endl;
```

-   This is called **structured binding**
-   It unpacks the tuple into separate variables
-   Introduced in C++17

------------------------------------------------------------------------

# Structured Binding in Loops

If you have a container of tuples:

``` cpp
for (auto [i, j, k] : matrix)
{
    // use i, j, k directly
}
```

-   Very useful when iterating over:
    -   Containers of tuples
    -   `std::map` (key-value pairs)
    -   Structured data

# Struct Example: Grouping Related Data

## Basic Struct Design

``` cpp
struct Phase
{
    int current;
    int voltage;
    int power;
};

struct PhaseVector
{
    Phase ph1;
    Phase ph2;
    Phase ph3;
};
```

### Usage

``` cpp
PhaseVector pv;

pv.ph1.current = 20;
```

------------------------------------------------------------------------

# Problem: Iterating Over Struct Members

Attempting this:

``` cpp
for (int i = 0; i < pv.size(); i++)
{
    pv[i].current = 0;
    pv[i].voltage = 0;
    pv[i].power   = 0;
}
```

❌ Issues:

-   `PhaseVector` does not have a `size()` function.
-   It is not an array or container.
-   `pv[i]` is invalid because no indexing operator is defined.

------------------------------------------------------------------------

# Solution: Operator Overloading

To make `PhaseVector` indexable like an array, we overload `operator[]`.

## Example Implementation

``` cpp
struct PhaseVector
{
    Phase ph1;
    Phase ph2;
    Phase ph3;

    Phase& operator[](int i)
    {
        switch (i)
        {
        case 0: return ph1;
        case 1: return ph2;
        case 2: return ph3;
        default:
            throw std::out_of_range("Index out of range");
        }
    }
};
```

### Why Return by Reference?

``` cpp
Phase& operator[](int i)
```

-   Returning by reference allows modification.
-   If returned by value, changes would affect only a copy.

------------------------------------------------------------------------

# After Overloading

Now this works:

``` cpp
for (int i = 0; i < 3; i++)
{
    pv[i].current = 0;
    pv[i].voltage = 0;
    pv[i].power   = 0;
}
```

------------------------------------------------------------------------

# Key Concepts Demonstrated

-   Struct grouping
-   Nested struct usage
-   Operator overloading
-   Returning by reference
-   Enabling array-like behavior for custom types

## C++ Algorithms & Callable Objects – Professional Notes

## General Rule When Writing Loops

Before writing a `for` loop, ask:

> Does `<algorithm>` already provide a function for this task?

Prefer using standard algorithms instead of manual loops. This:

* Improves readability
* Reduces bugs
* Encourages declarative style
* Leads to cleaner and shorter code

When iteration is required, prefer:

* `std::for_each` over raw `for` loops (when appropriate)

---

# `<algorithm>` and `<numeric>` Overview

```cpp
#include <algorithm>
#include <numeric>
```

Common operations:

* `std::accumulate`
* `std::reverse`
* `std::copy`
* `std::copy_if`
* `std::copy_n`
* `std::sort`
* `std::transform`
* `std::for_each`
* `std::generate`

---

# 1) std::accumulate

Header:

```cpp
#include <numeric>
```

### Syntax

```cpp
std::accumulate(begin, end, initial_value);
std::accumulate(begin, end, initial_value, binary_function);
```

* The return type is the type of `initial_value`.
* Commonly used for summation.
* Can also be used for string concatenation or custom reductions.

---

## Example: Join Strings (CSV Style)

```cpp
#include <iostream>
#include <vector>
#include <numeric>

std::string join(std::string acc, std::string next)
{
    return acc + ", " + next;
}

int main()
{
    std::vector<std::string> csv_line = {"Name", "Age", "Length"};

    std::string joined = std::accumulate(
        csv_line.begin() + 1,
        csv_line.end(),
        csv_line[0],
        join
    );

    std::cout << joined << std::endl;
}
```

---

# 2) std::reverse

### Syntax

```cpp
std::reverse(begin, end);
```

Reverses elements in-place.

---

## Reversing a File (Concept)

If reversing file lines:

* Avoid loading entire file into a vector if memory is a concern.
* Use file streaming (`std::ifstream`, `std::ofstream`).
* Use file seeking (`seekg`) to move within the file.
* Reverse lines manually while reading.

This is more about stream handling than containers.

---

# 3) Copy Algorithms

## std::copy

```cpp
std::copy(src.begin(), src.end(), dest.begin());
```

Copies all elements.

---

## std::copy_if

```cpp
std::copy_if(src.begin(), src.end(), dest.begin(), predicate);
```

* Copies only elements where predicate returns `true`.
* Predicate must return `bool`.

---

## std::copy_n

```cpp
std::copy_n(src.begin(), count, dest.begin());
```

Copies a fixed number of elements.

---

## reserve vs resize

* `reserve(n)` → Allocates memory but does not change size.
* `resize(n)` → Changes container size (may initialize elements).

---

# Callable Objects in C++

A callable is anything that can be invoked like a function.

## 1) C Function Pointer

```cpp
bool (*fp)(int);

bool isEven(int a) { return a % 2 == 0; }

fp = isEven;
fp(5);
```

Drawbacks:

* No state
* Limited flexibility
* No type safety enhancements

---

## 2) std::function

```cpp
#include <functional>

std::function<bool(int)> f;
f = isEven;

if (f)
{
    f(10);
}
```

* Type-erased callable wrapper
* Can store function pointers, lambdas, functors
* Slight runtime overhead

---

## 3) Function Object (Functor)

A class that overloads `operator()`.

```cpp
class NumberChecker
{
public:
    enum Type { CHECK_EVEN, CHECK_ODD, CHECK_PRIME };

    bool operator()(Type check, int num)
    {
        switch (check)
        {
        case CHECK_EVEN:  return isEven(num);
        case CHECK_ODD:   return isOdd(num);
        case CHECK_PRIME: return isPrime(num);
        default:          return false;
        }
    }

private:
    bool isEven(int n)  { return n % 2 == 0; }
    bool isOdd(int n)   { return !isEven(n); }
    bool isPrime(int n) { return false; }
};
```

Usage:

```cpp
NumberChecker check;
std::cout << std::boolalpha
          << check(NumberChecker::CHECK_EVEN, 40)
          << std::endl;
```

---

## 4) Lambda Expressions

### Basic Syntax

```cpp
[capture](parameters) { body };
```

Example:

```cpp
auto isEven = [](int x)
{
    return x % 2 == 0;
};
```

---

## Capture List

Example:

```cpp
int y = 10;

auto lam = [y](int x)
{
    return x + y;
};
```

### Capture Rules

* `[y]` → capture by value (read-only unless `mutable`)
* `[&y]` → capture by reference
* `[&]` → capture everything by reference
* `[=]` → capture everything by value
* `[&, z]` → everything by reference, `z` by value

Captured-by-value variables are `const` unless lambda is marked `mutable`:

```cpp
auto lam = [y](int x) mutable
{
    y = 20; // allowed
    return x + y;
};
```

---

## How Lambdas Work Internally

The compiler generates an anonymous class:

* Captured variables become member variables
* `operator()` is generated
* Capture list defines what becomes part of the class

---

# std::transform

### Syntax

```cpp
std::transform(src.begin(), src.end(), dest.begin(), operation);
```

* Applies a callable to each element
* Stores result in destination container
* Usually returns the same type as the source
* Does not modify source unless source == destination

Example:

```cpp
std::transform(v.begin(), v.end(), v.begin(),
               [](int x) { return x * 2; });
```

---

# std::invoke

Generic calling interface:

```cpp
std::invoke(callable, args...);
```

* Can invoke:

  * Function pointers
  * Member functions
  * Functors
  * Lambdas

Useful when writing generic template code that accepts any callable.

---

# Summary

## Prefer Algorithms Over Loops

Instead of writing manual loops:

* Use `accumulate` for reductions
* Use `transform` for element-wise modifications
* Use `copy_if` for filtering
* Use `reverse` for reversing ranges
* Use `sort` for ordering

## Callable Types in C++

1. Function pointer (C-style)
2. `std::function`
3. Functor (function object)
4. Lambda expression

Each provides increasing flexibility and abstraction.
