# Standard Template Library (STL) in C++

## Overview
* **Standard Template Library (STL):** A core component for Data Structures and Algorithms (DSA).
* **Computer Science Core:** Focused on problem-solving, primarily divided into:
    1.  **Architecture-based problems**
    2.  **Data manipulation**

---

## 1. Architecture
Architecture focuses on the structural design and organization of software.

### Types of Architecture
* **Dynamic Architecture**
* **Static Architecture:** Often refers to Layered Architecture.

### Key Architectural Concepts
* **Concurrency:** Managing multiple tasks via threading and asynchronous programming.
* **Design Principles:**
    * **Coupling and Cohesion:** Measuring the dependencies and responsibilities of modules.
    * **Object-Oriented Programming (OOP):** Implementing SOLID principles.
* **Note on Performance:** While essential for "Clean Code" and maintainability, these architectural patterns are generally distinct from raw Input/Output (I/O) or data processing performance.

---

## 2. Data Manipulation
This section focuses on how data is stored and processed efficiently.

### Core Components
* **Algorithms:** Operations performed on Data Structures.
* **Containers:** Structures that hold data (e.g., `vector`, `list`, `map`, `set`, `stack`, `queue`).
* **Iterators:** * A subclass used to traverse a container.
    * Every container has its own specific iterator.

> **Key Rule:** Algorithms iterate on containers.

### Implementation
* **New Header:** `<algorithm>` (formerly referenced as `<algorithm.h>`).


## Important Intro: How to Study Data Structures and Algorithms (DSA)

To master DSA, the focus should be on comparing the implementation of different containers and evaluating their operations through the lens of time complexity.

### 1. Data Structure Operations
Every Data Structure (DS) possesses its own unique set of operations, generally categorized into:

1.  **Lifecycle:** Create and Destroy.
2.  **Modification (Size):** Insert and Erase.
    * *Note:* These operations change the actual size/capacity of the container.
3.  **Access (Values):** Read and Write.
    * *Note:* Unlike modification, access involves only reading or editing existing values without changing the container's size.
4.  **Search Algorithms:** Locating specific data within the structure.
5.  **Sort Algorithms:** Organizing data (primarily numbers) based on specific formats/orders.

---

### 2. Time Complexity: Big O Notation
Big O notation, expressed as $O(f(n))$, represents the **worst-case scenario** for an algorithm's performance.

* **n:** The number of elements in the container being operated upon.
* **Steps:** The number of operations taken per iteration.

#### Common Complexity Classes


| Notation | Name | Description |
| :--- | :--- | :--- |
| **$O(1)$** | Constant | Performance is independent of the number of elements; a horizontal line on a graph. |
| **$O(n)$** | Linear | Performance scales proportionally with the number of elements. |
| **$O(\log n)$** | Logarithmic | Performance increases slowly as the data set grows (e.g., Binary Search). |

> **Space Complexity:** A measure of how much additional memory is required to solve a problem relative to the input size.

---

### 3. Code Example: Complexity Analysis
```cpp
// Example of an iteration loop
while (i < 20) 
{
    if(i == 5) 
    {
        // Perform an action
    }
}
```
# Containers

## 1. std::array
A container that encapsulates fixed-size arrays, providing a safer and more feature-rich alternative to C-style arrays.

### Key Characteristics
* **Contiguous Elements:** Elements are stored in a continuous block of memory, allowing for $O(1)$ random access.
    
* **Fixed Size:** The size is part of the type definition and must be a constant expression known at compile-time. It cannot grow or shrink.
* **Memory Safety:** * **C-style Arrays (Carray):** These are considered "bad" or "unsafe" because they lack boundary checking. Accessing an index outside the range leads to undefined behavior or memory corruption.
    * **std::array Improvement:** While the `[]` operator still doesn't check bounds (for performance), `std::array` provides the `.at()` member function which performs boundary checking and throws an exception if the index is out of bounds.

### Syntax & Initialization
```cpp
#include <array>

// Syntax: std::array<T, N> 
// T: Type of elements
// N: Number of elements (must be a constant integer expression)
std::array<int, 5> arr = {1, 2, 3, 4, 5};
```
| Feature            | C-style Array                          | std::array                                      |
|--------------------|----------------------------------------|-------------------------------------------------|
| Size Retrieval     | Manual or sizeof tricks                | `.size()` method                                |
| Boundary Checking  | None (Unsafe)                          | Supported via `.at()` (throws exception)        |
| Iterators          | Raw pointers only                      | `.begin()`, `.end()`, etc.                      |
| Assignment         | Not supported (requires `memcpy`)      | Supported (`arr1 = arr2`)                       |
| Safe Access        | `arr[i]` (no check)                    | `arr.at(i)` (boundary check)                    |


# std::array Example in C++

```cpp
#include <array>

std::array<int, 10> arr = {10, 20, 30};

int Test_main()
{
    size_t size = arr.size();

    // The ability of iterators --> it's an abstraction of pointer (pointer-like object)
    // makes loop generalization possible

    // Range-based for loop (C++11)
    for (int& val : arr)
    {
        std::cout << val << ' ';
    }
    cout << std::endl;

    // Normal loop
    // Note: use != not < (it works with arrays but with other containers it will not)
    for (auto it = arr.begin(); it != arr.end() l ++it)
    {
        std::cout << *it << ' ';
    }
    std::cout << std::endl;
}
```
# std::array Methods & Notes

```cpp
arr.at(2);      // Throws exception if out of range

arr.front();    // Returns first element
arr.back();     // Returns last element
                // (values, not pointers like begin() and end())

arr.fill(0);    // Fill all elements with the same value

// Important trick:
arr.empty();    // Tricky

if (!arr.empty())   // Always false for std::array
{
    // This definition is tricky because std::array
    // cannot be empty (size is fixed at compile time)
}

// [] operator is NOT safe (no boundary check)

int* data = arr.data();   // Get pointer to underlying data 
```
# Notes & Concepts

- That’s a pointer to the underlying data, so be careful when dealing with it.
- Good practice: try to make such access read-only whenever possible.

---

## Example Concept

We have a vector of floats (4 bytes per float).

**Task:** Send float values over UART.

**Intuition:**  
Deal with the data as raw bits.

---

## std::array Design Notes

- `std::array` has a fixed size.
- There are no built-in insert or erase methods.
- Any insert/erase behavior must be implemented manually by the programmer.
- The container itself does not provide dynamic resizing operations.

---

## Additional References

- CPPCON on YouTube  
- Research topic: **C++26 → std::meta**

## Vector in C++ 
- It is a dynamically allocated array.  
- Provides almost the same APIs as `std::array`.  
- Supports dynamic resizing.  
- Manages memory automatically.  
- Allows insertion and removal of elements.  
- Safe access via `.at()` and unsafe via `[]`.  
- Can get underlying data pointer using `.data()`.  

```cpp
#include <vector>

std::vector<int> v;

v.push_back(5);
v.push_back(10);
v.push_back(11);

int last = v.back();   // Returns the last element

v.pop_back();          // Removes the last element (returns void)

// What if we do:
v.at(3);               // Throws an exception if index is out of range
```
## Vector Capacity Notes

- Imagine a `std::vector` as a bottle:  
  - **Capacity** → the maximum it can hold before needing to grow.  
  - **Size** → the current number of elements stored.  
- Capacity grows automatically based on growth factors, which are compiler-dependent.  
- Good practice: use `v.reserve(n)` to preallocate memory when the number of elements is known, reducing reallocations.  









