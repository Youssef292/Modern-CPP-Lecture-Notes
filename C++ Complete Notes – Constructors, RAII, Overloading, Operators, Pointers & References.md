# Constructors and Destructors

- A **constructor** is a special function that initializes an object in memory when we instantiate an object.
- Its name is the same as its class name.
- **Naming convention**: Class names typically use **PascalCase** (e.g., `CPerson`, `CSensorReading`).  
  (Side note: Some use a leading 'C' for classes, but it's not mandatory.)

- Constructors can have arguments to provide initial values for private members.
- Constructors can call other constructors (this adds complexity when classes are nested—instantiating a class inside another triggers its constructors).

## Example: `CSensorReading` Class

```cpp
class CSensorReading {
public:
    // Constructor using an initialization list
    CSensorReading(int val, int unit, int timestamp)
        : m_val(val), m_unit(unit), m_timestamp(timestamp) {}

private:
    int m_val;
    int m_unit;
    int m_timestamp;  // Note: naming convention m_ for member variables is common in C++
};
```
## Example: CSensorNode Class
```cpp
class CSensorNode {
public:
    // Constructor initializes members, including calling another constructor for m_reading
    CSensorNode(int id, int loc)
        : m_id(id), m_location(loc),
          m_reading(0, 0, 0),     // explicit constructor call (adds complexity)
          m_reading2(m_reading)   // copy constructor
    {
        // Inside the constructor body: application-specific initialization logic
        // This is like startup code executed when the object is created.
    }

    // Lazy initialization: some logic we want to run later
    void init(/*...*/) {
        // ...
    }

private:
    int m_id;
    int m_location;
    CSensorReading m_reading;
    CSensorReading m_reading2;
};
```
> “We use a constructor initialization list because if members need to be initialized before the constructor body executes, it ensures they are initialized properly. The compiler also requires const and reference members to be initialized this way, so the initialization list handles these cases correctly and efficiently.”

## Destructors
A universal pattern: if there is a constructor, there is often a destructor.
Destructor syntax: `~ClassName()`

```cpp
~CSensorNode() {
    delete m_id;   // Example cleanup
}
```
- **Lazy deinitialization**: We may want to delay cleanup. How to implement it? (Search topic: "lazy deinitialization in destructor?")

### RAII (Resource Acquisition Is Initialization)

This is a common idiom in C++:

- **Resource**: memory, files, mutexes, semaphores, etc. (OS resources)
- **Acquisition**: reserving a resource must happen during initialization.
- **Release**: when the resource is no longer needed, it must be deinitialized.

The destructor is where we release resources to apply the RAII idiom.

## Overloading

- In **C**, a function cannot have multiple definitions with the same name—it would cause a linker error.
- In **C++**, it is normal to define multiple functions with the same name but different parameter lists (overloading). Each has its own implementation.

```cpp
int add(int a, int b) {
    return a + b;
}

float add(float a, float b) {
    return a + b;
}
```
### Usage:
```cpp
int x = 5, y = 6;
int z = add(x, y);          // Calls int version

float x2 = 5.5f, y2 = 6.5f;
float z2 = add(x2, y2);     // Calls float version

// float z3 = add(x, y2);   // Error: no matching function (mixed types)
// float z3 = add(x2, 6);   // Also error
```

- The compiler uses **name mangling** to distinguish overloaded functions (e.g., `_Z3addii` for `add(int, int)`). Explore on [Compiler Explorer](https://godbolt.org/).

## Overloading in Classes

- Methods can be overloaded.
- Constructors (ctors) can be overloaded.
- Operators can be overloaded: `+`, `-`, `*`, `[]`, `->`, `>>`, `<<`, etc.
- (Note: operators are just functions that the compiler calls; overloading them is possible.)

### Example: Operator Overloading
```cpp
class CCartesianPoint {
public:
    CCartesianPoint(int x, int y) : m_x(x), m_y(y) {}

    // Overload the + operator
    CCartesianPoint operator+(const CCartesianPoint& other) {
        return CCartesianPoint(this->m_x + other.m_x, this->m_y + other.m_y);
    }

private:
    int m_x, m_y;
};

void test_operator_overloading() {
    CCartesianPoint p1(2, 2), p2(3, 3);
    CCartesianPoint p3 = p1 + p2;  // Now works because operator+ is defined
}
```
## Tasks for Practice

1. **Dot product**: Overload `operator*` for a vector class.

2. **Counter class**: Overload prefix `++` and postfix `++`.

### Example Skeleton:
```cpp
class Counter {
private:
    int count;
    int step;

public:
    // Constructor with default parameters
    Counter(int start = 0, int s = 1) : count(start), step(s) {}

    // Prefix increment
    void operator++() {
        count += step;
    }

    // Postfix increment (dummy int parameter)
    void operator++(int) {
        count += step;
        // Note: The 'int' is unused; to avoid warning:
        // (void) /*parameter_name*/; // but here it's unnamed
    }
};
```
- **Prefix**: `++c1`
- **Postfix**: `c1++` (needs a dummy `int` parameter)


### Overloading can be done in multiple ways: normal functions, methods, constructors, operators.
- Default parameters can also be used to reduce the number of overloads:

```cpp
Counter(int start = 0, int s = 1) {}   // One constructor covers both Counter(5) and Counter(5,10)
```
- The compiler resolves calls based on argument types and count.

## Conclusion on Operator Overloading

1. **It is a useful feature** when used correctly.

2. **Readability can be a double‑edged sword**.
   - Example: `cout << "Hello"` reuses the shift operator, which might be confusing in other contexts.

3. **Always consider the impact** on your team and yourself when overloading operators.

# Pointers and References

## Pointer Pitfalls in C

 **1- Dangling pointer** – pointing to memory that is no longer valid.

```c
int* fun() {
    int x;          // local variable on stack
    return &x;      // after function returns, x is destroyed → dangling pointer
}
```
**Solution:** Use dynamically allocated memory (heap) or return a pointer to a global/static variable.

**2- Wild pointer** – a pointer with an undefined value.
```c
int* ptr;           // uninitialized
printf("%d", *ptr); // undefined behavior
```
 **Solution:** Always initialize pointers (e.g., to `NULL` or `nullptr`) and check before dereferencing.

# References in C++

- References were introduced to solve some pointer issues.
```cpp
int x = 10;
int* ptr = &x;     // pointer: need address-of operator
int& ref = x;      // reference: must be initialized at declaration
```
## Benefits of References

- **No manual dereferencing needed:** A reference automatically refers to the object.  
- **Cannot be left uninitialized:** Attempting to do so results in a compiler error.  
- **Helps clarify ownership:**
  - `fun(object*)` – unclear who owns the object (caller or callee).  
  - `fun(object&)` – you don't own the object, but you can modify it (side effects).

```cpp
int& func() {
    int x;
    int& ref = x;
    return ref;    // Dangling reference! x is local, destroyed after return.
}
```
- Returning a reference to a local variable causes the same dangling problem as pointers.

## References vs. Pointers

- References solve wild pointer issues (they must be initialized), but dangling references are still possible.  
- There is no "null reference" (though a reference bound to a null pointer is undefined behavior).  
- No pointer-to-reference or reference-to-reference (though `int&&` exists – it's an rvalue reference, covered later).  

### Why Still Use Pointers?

- **`this` pointer:** Inside a class, `this` is a pointer (cannot be a reference due to language rules).  
- **Dynamic memory allocation:** Low-level memory management (`new`/`delete`, `malloc`/`free`) works with pointers.  
- **Low-level hardware access:** Memory-mapped I/O in embedded systems.  
- **Callbacks and function pointers:** No such thing as a reference to a function.


