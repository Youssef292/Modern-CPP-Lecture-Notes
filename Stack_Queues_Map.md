# Vectors & Containers – Professional Notes

## 1. Vectors & 2D Arrays

* Which is better for a 2D matrix?

  * `std::vector<std::vector<int>> matrix1;`
  * `std::vector<int> matrix2;` (flattened 1D array)
* **Reason:**

  * At the C level, `vector<vector<int>>` creates multiple allocations → non-contiguous memory → poor cache locality.
  * Flattened 1D array improves cache performance (important in data-oriented design, e.g., game development).
  * Access formula: `matrix[row * num_cols + col]`

---

## 2. Container Categorization

* **Sequential Containers** – ordered by index: `array`, `vector`, `list`, `stack`, `queue`, `deque`
* **Associative Containers** – key-value mapping: `map`, `set`, `multimap`, `multiset`
* Note: `stack` and `queue` are adapters, can be implemented on `vector` or `list`.

---

## 3. Stack (LIFO)

```cpp
#include <stack>
std::stack<int> st;

st.push(x);     // insert element
st.top();       // access top
st.pop();       // remove top
```

* Task example: parsing expressions, RPN notation.

---

## 4. Queue (FIFO)

```cpp
#include <queue>
std::queue<int> q;

q.push(x);      // insert at back
q.pop();        // remove front
q.front();      // access front
q.back();       // access back
```

* Example: moving average computation using queues.
* C++20 alternative: `std::span`.

---

## 5. Map (Associative Container)

* Stores key-value pairs.
* Types:

  * `map` → keys sorted
  * `unordered_map` → keys unsorted
* Key: `int`, `string`, `char`, or custom type (with constraints)
* Value: any associated data

```cpp
#include <map>
#include <string>

std::map<std::string, int> mp = { {"youssef", 24}, {"ramez", 23} };

mp["youssef"] = 23;
mp.count("youssef");   // 1 if key exists, 0 otherwise

// Iterate
for(auto it = mp.begin(); it != mp.end(); ++it) {
    std::cout << "Key: " << it->first << ", Value: " << it->second << std::endl;
}
```

* Built internally as `std::pair<key, value>`
* Functions:

  * `insert(key, value)`
  * `[]` → returns reference to value (creates key if missing)
  * `find()` → returns iterator to key-value pair
  * `count()` → number of keys (for multimap, multiple keys allowed)

---

## 6. Callable Objects in Maps

* **Callable objects** generalize function pointers.
* Can store function pointers, lambdas, or functors.

```cpp
#include <functional>
#include <map>
#include <vector>
#include <string>
#include <iostream>

void handleCreate(std::vector<std::string>& args) { /* implementation */ }
void say_hello() { std::cout << "Hello world" << std::endl; }

std::function<void(void)> func;
func = say_hello;
func();

std::map<std::string, std::function<void(std::vector<std::string>&)>> m_command = {
    {"create", handleCreate}
};
```

* Drawback: all stored functions must match the same prototype.


