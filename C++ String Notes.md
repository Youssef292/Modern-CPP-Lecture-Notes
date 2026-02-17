# C++ String Notes

---

# 1) C-Style Strings (C)

## Definition

```c
char c[10];
```

- Fixed-size character array.
- If local → contains garbage values.
- If global → initialized with zeros.

---

## Initialization

```c
char c[10] = "Youssef";
```

---

## Assignment Problem

```c
char c2[10];
c2 = c;   // ❌ Error
```

### Why?

- The array name is an alias to the first memory address.
- Arrays cannot be assigned after declaration.
- Assignment rule:

```
Lval = Rval
Lval → Location
Rval → Data value
```

---

## Invalid Operation

```c
c[10] = "Some thing";  // ❌ Error
```

---

## Accessing Characters

- Must use loops.
- No boundary checking.

---

## Important C String Functions

- `strstr()` → Finds substring.
- `sprintf(myString,"%format", numVar);` → Convert int to string.
- `sscanf()` → Parse formatted input.
- `atoi()` → Convert string to int.
- `atod()` → Convert string to double.

---

# 2) C++ Strings — std::string

## Overview

- C++ provides a class: `std::string`
- Contains:
  - Data members
  - Member functions
- Must include:

```cpp
#include <string>
```

- Define:

```cpp
std::string myString;
```

> You can use C libraries in C++, but it may cause compatibility issues.

---

# 3) Creating std::string Objects

## Different Initialization Methods

```cpp
std::string str1 = "youssef";
std::string str2 = {"youssef"};
std::string str3("youssef");
```

---

## Copying and Assignment

```cpp
std::string str3(str2);
std::string str5 = str2;
str5 = str1;
```

All are valid in C++.

---

## Assignment After Declaration

```cpp
std::string str7;
str7 = "hello";  // ✅ Valid
```

---

## Construct With Repeated Character

```cpp
std::string str8(5, 'a');  // "aaaaa"
```

---

## Character List Initialization

```cpp
std::string str9{'a','b'};
```

---

# 4) std::string Functional Categories

---

# A) Storage & Capacity

```cpp
std::string s("Youssef Abdelhakem");
```

## Size Functions

```cpp
s.size();
s.length();
s.capacity();
```

### Explanation

- `size()` → Number of characters.
- `length()` → Same as size.
- `capacity()` → Allocated memory size.

---

## Capacity Concept

- C → Fixed size.
- C++ → Dynamic resizing.
- Memory allocated in heap.
- Size can change at runtime.

---

## Small String Optimization (SSO)

- If string size < 15 bytes:
  - Stored in stack memory.
- If larger:
  - Allocated in heap memory.

Example:

```cpp
std::string s = "short";
std::cout << s.size();
std::cout << s.capacity();
```

---

## reserve()

```cpp
s.reserve(100);
```

- Sets capacity.
- Avoids repeated reallocations.
- Usually forces heap allocation.

---

## resize()

```cpp
s.resize(50);
```

- Changes string size.
- May add or remove characters.

---

## Performance Problem

```cpp
for(int i = 0; i < 100000; i++){
    s2 += "a";  // Reallocates many times
}
```

### Better Solution

```cpp
s2.reserve(100000);
```

---

# B) Accessing Characters

## C-Style

```cpp
char c[10];
```

- No boundary checking.

---

## C++ Style

```cpp
std::string s("Nabil");
```

### No Boundary Check

```cpp
s[0];
```

### With Boundary Check

```cpp
s.at(8);  // Throws exception if out of range
```

---

## Other Access Methods

- `s.front()` → First character
- `s.back()` → Last character
- `s.c_str()` → Returns `const char*`
- `s.data()` → C++17 returns `char*` (modifiable)

---

# C) Modification Functions

```cpp
std::string s("youssef");
```

## Append

```cpp
s.append("s");
s += "a";
```

---

## Concatenation

```cpp
std::string s1("Abdelhakem");
s = s + s1;
```

---

## Comparison

```cpp
if (s == s1)
```

---

## Insert

```cpp
s.insert(5, "x");
```

---

## Replace

```cpp
s.replace(pos, number_of_characters, "a");
```

Example:

```cpp
s.replace(5, 3, "a");
```

---

## Push & Pop

- `push_back()` → Add character at end
- `pop_back()` → Remove last character

---

# 5) Searching in Strings (Parsing)

```cpp
std::string s = "hello, World";
```

---

## find()

- Returns index if found.
- Returns `std::string::npos` if not found.

```cpp
s.find("hello");  // 0
s.find('o', 5);
```

---

## find_first_of()

```cpp
s.find_first_of("llo");
```

Returns first matching index.

---

## rfind()

- Searches from the end.
- Returns position counted from the beginning.

---

# 6) Substring

```cpp
std::string text = "hello world";
std::string sub = text.substr(index, length);
```

Example:

```cpp
std::string text = "hello, world";
std::string sub = text.substr(6, 5);

std::cout << text;
std::cout << sub;
```

Another example:

```cpp
std::string text = "data,data,data";
```

---

# 7) Conversion Utilities

- `stoi()` → string to int
- `stod()` → string to double
- `std::to_string(int | double)` → number to string
- `atoi()`
- `atod()`

---

# 8) Raw Strings

## Escaping Characters

```cpp
printf("Hello, \"World\"");
```

---

## Raw String Literal

```cpp
std::string path = R"(c:/name/)";
```

Useful for:
- File paths
- Regex
- Development environments

---

# 9) C++ Standard Updates

---

## C++11

- UTF-16 / UTF-32 support
- Raw strings
- `to_string`
- `front()`, `back()`
- `pop_back()`, `push_back()`

---

## C++14

```cpp
using namespace std::literals::string_literals;
auto s = "hello"s;
```

---

## C++17

- `str.data()` → returns `char*`
- `std::string_view`

---

## C++20 / C++23

- `contains()` replaces common `find()` usage:

```cpp
"hello".contains('h');  // true
```

---

# Final Summary

- C strings → fixed-size arrays.
- std::string → dynamic class.
- Capacity ≠ size.
- Use `reserve()` for performance.
- Use `at()` for safe access.
- Use `substr()`, `find()`, `replace()` for parsing.
- Modern C++ improves usability with `string_view`, `contains()`, and raw strings.

---
