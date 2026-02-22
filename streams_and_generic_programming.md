# Streams in C++

## What is a Stream
- A stream is a sequential data source with unknown start or end (appears infinite, but start and end can be known).  
- Sequential: data must be accessed one after another, not random access.  
- Imagine it as a data pipeline for input/output.  
- Streams are an abstraction of files in C.

## Data Sources
- **Console:** `iostream` (user `cin`/`cout`)  
- **Files:** Disk I/O (stored data in non-volatile memory)  
  - In Unix, everything is a file (common interface of the OS)  
  - `fstream`, `ifstream`, `ofstream`  
- **Socket:** Network protocols  
- **Character Device:** UART  
- **Memory Stream:** Read directly from memory  
  - `stringstream`

## Stream Types
- **Text:** fixed-size data chunks  
- **Binary:** variable-size data chunks (specific encoding to the app/file)  

**Note:** Any process in any operating system has 3 standard streams:  
- `stdin` → 0  
- `stdout` → 1  
- `stderr` → 2  

## File I/O & Redirection
- **I/O Redirection:**  
  - Output: `program.exe > output`  
  - Input: `program.exe < input > output`  

**Example Task:**  
- Input file: `20+30`  
- Output: `50`  
- Error: `sum is 50 / sub`

## Templates [MetaPorgramming]
- complie time instant instant 

# C++ Templates & SafeArray Example

```cpp
#include <iostream>

// Generic add function
template <typename T>
T add(T a, T b)
{
    return a + b;
}

// Generic scale function with default template parameters
template <typename T = int, const int scaler = 10>
T scale(T a)
{
    return a * scaler;
}

// SafeArray class template
template <typename T, const int arr_sz>
class SafeArray
{
    T data[arr_sz];
    const int size = arr_sz;

public:
    // Overloaded subscript operator with bounds checking
    T& operator[](int index)
    {
        if (index < arr_sz && index >= 0)
        {
            return data[index];
        }
        // Optionally: throw an exception here for out-of-bounds access
        throw std::out_of_range("Index out of bounds");
    }
};

int main()
{
    int z = add(5, 6);             // Correct usage

    int s = scale<int, 10>(5);     // Correct syntax
    float f = scale<float, 50>(5);

    SafeArray<int, 10> my_array;
    my_array[0] = 42;
    std::cout << my_array[0] << std::endl;

    return 0;
}
```





