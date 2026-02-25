# STL Containers & Project Notes – Professional Version

## Agenda

* Continue map
* set
* pair
* tuple
* Algorithms:

  * sort
  * copy
  * find
  * reverse
  * transform

Goal: reduce manual loops using `<algorithm>` library.

---

## 1. push_back vs emplace_back

* `push_back()`:

  * Takes argument by value
  * Calls constructor + copy into container → 2 constructor calls
* `emplace_back()`:

  * Forwards arguments directly to constructor
  * Only one constructor call → more efficient

---

## 2. Map (Ordered Container)

* Keys are automatically sorted
* Can define custom comparison by overloading `<` operator or providing comparator class

```cpp
struct Address {
    std::string street;
    std::string country;
    std::string block_num;

    Address(std::string st, std::string ct, std::string bn)
        : street(st), country(ct), block_num(bn) {}

    bool operator<(const Address& other) const {
        return street < other.street
               && country < other.country
               && block_num < other.block_num;
    }

    std::string toString() const {
        std::stringstream ss;
        ss << country << ", " << street << ", " << block_num;
        return ss.str();
    }
};

struct Person {
    std::string name;
    int age;

    Person(std::string Pname, int Page) : name(Pname), age(Page) {}

    std::string toString() const {
        std::stringstream ss;
        ss << "Person: " << name << ", Age: " << age;
        return ss.str();
    }
};

std::map<Address, Person> google_map;
```

---

## 3. Unordered Map

* **Ordered map**: uses balanced binary tree (Red-Black Tree)

  * Search, insert, erase → O(log n)
* **Unordered map**: uses hash table

  * Access, insert, erase → O(1) on average
  * Requires unique key and hash function
  * Custom types require `==` and hash function
* Variants: `multimap`, `unordered_multimap`

---

## 4. Set

* **Set**: sorted unique elements
* **Unordered set**: unsorted unique elements
* Key is the value itself (no pairs)

```cpp
std::set<int> s = {10, 20, 30};
s.insert(10); // no effect, duplicates not allowed
```

* Real-world usage:

  * Deduplication in payment systems
  * Excel: remove duplicates
  * Git diff: compute differences between files
  * Unique elements in vectors

---

## 5. Project: Parking System Data Modeling

### Constants

```cpp
const double Rate_regular = 5.0;
const double Rate_vip = 10.0;
const double Rate_disabled = 3.0;
const double Min_charge = 5.0;
```

### Parking Session

```cpp
struct ParkingSession {
    std::string plate;
    int spotNum;
    int entryHour;
    int entryMinute;
    std::string spotType; // e.g., "regular", "vip", "disabled"
};
```

### Receipt Structure

```cpp
struct Receipt {
    std::string plate;
    int spotNum;
    std::string exitTime;
    double duration;
    double totalCost;
};
```

### Containers

```cpp
std::map<std::string, ParkingSession> currentParking;
std::map<std::string, std::vector<Receipt>> history;
std::map<int, bool> spots;
std::map<int, std::string> spotTypeMap; // e.g., 0-16: regular, 16-18: vip, 18-20: disabled
std::map<int, int> hourlyTraffic; // hours 0-23, traffic count
```

* Functionality:

  * Track car data
  * Generate receipts
  * Assign spot type
  * Count hourly traffic
