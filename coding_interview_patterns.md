# Coding Interview Patterns - Complete Study Guide

## How to Use This Guide
For each pattern you'll find:
- **When to recognize it** — signals in the problem statement
- **Core template** — the skeleton code you adapt every time
- **Variations** — how the same pattern morphs across different questions
- **Practice problems** — ordered easy → hard

---

# Part 0: C++ STL Fundamentals

Before learning data structures and algorithms, you need to be comfortable with C++ STL syntax. This section covers the concepts that trip up beginners.

## Variables: Types, `auto`, and Declarations

In C++, every variable needs a type. You can either write it explicitly or let the compiler figure it out with `auto`.

```cpp
// Explicit types:
int x = 5;
string name = "hello";
vector<int> nums = {1, 2, 3};
unordered_map<int, int> seen;

// Using auto (compiler deduces the type):
auto x = 5;                             // int
auto name = "hello"s;                   // string (the 's' suffix matters)
auto it = seen.find(3);                 // unordered_map<int,int>::iterator

// When to use auto:
// - Iterator types (they're long and ugly)
// - When the type is obvious from the right side
// - Structured bindings (covered below)

// WRONG: forgetting the type entirely
index = seen.find(3);                   // ERROR: what is 'index'? No type!
auto index = seen.find(3);              // OK: auto deduces the type
```

---

## Iterators: The Pointer-Like Objects

**What is an iterator?** It's an object that "points to" an element inside a container. Think of it as a bookmark in a book — it marks a position.

```cpp
vector<int> v = {10, 20, 30, 40, 50};

// v.begin() → iterator to first element (10)
// v.end()   → iterator PAST the last element (not 50, but after it!)
//
//   begin()                         end()
//     ↓                               ↓
//    [10] [20] [30] [40] [50]       [???]
//     0    1    2    3    4      (past the end)

auto it = v.begin();        // points to 10
*it;                        // 10  (dereference: get the value it points to)
it++;                       // now points to 20
*it;                        // 20
it + 2;                     // points to 40 (random access, vector only)
```

### The Key Operations

```cpp
vector<int> v = {10, 20, 30, 40, 50};

// Dereference: get the value
auto it = v.begin();
int val = *it;              // 10

// Move forward/backward
it++;                       // next element
it--;                       // previous element
advance(it, 3);             // jump forward 3 positions

// Arithmetic (vector/deque only — not for set/map!)
auto it2 = v.begin() + 3;  // points to 40
int dist = it2 - it;       // 3 (distance between iterators)

// Comparison
if (it == v.end()) { /* reached the end, element doesn't exist */ }
if (it != v.end()) { /* iterator is valid, can dereference it */ }

// Convert to index
int index = it - v.begin();    // e.g., 0, 1, 2, ...
```

### Iterators for Different Containers

```cpp
// VECTOR: random access, fast arithmetic
vector<int> v = {10, 20, 30};
auto it = v.begin() + 1;       // OK: points to 20
int idx = it - v.begin();      // OK: idx = 1

// SET/MAP: bidirectional only — NO arithmetic!
set<int> s = {10, 20, 30};
auto it = s.begin();            // points to 10
it++;                           // OK: points to 20
// it + 1;                      // ERROR: can't do arithmetic on set iterators
next(it);                       // OK: returns iterator to next element
prev(it);                       // OK: returns iterator to previous element

// This is why we use next() and prev() with set/map:
auto it2 = next(s.begin());    // points to 20
auto it3 = prev(s.end());      // points to 30 (last element)
```

---

## `.` vs `->` (Dot vs Arrow)

**Simple rule:**
- **`.`** = access member of an **object**
- **`->`** = access member of a **pointer/iterator**

```cpp
// Object → use dot
pair<int, int> p = {3, 7};
p.first;                        // 3
p.second;                       // 7

string s = "hello";
s.size();                       // 5
s.empty();                      // false

vector<int> v = {1, 2, 3};
v.push_back(4);
v.size();                       // 4

// Pointer/Iterator → use arrow
auto it = myMap.find(key);
it->first;                      // the key
it->second;                     // the value

ListNode* node = new ListNode(5);
node->val;                      // 5
node->next;                     // pointer to next node

// WHY? An iterator is like a pointer. You can't do pointer.member,
// you need pointer->member (which is shorthand for (*pointer).member)

// These two are equivalent:
it->second;                     // cleaner (preferred)
(*it).second;                   // same thing, uglier
```

---

## `pair` — Two Values Bundled Together

Many STL structures use pairs internally (maps, priority queues with custom data).

```cpp
#include <utility>

// Creation
pair<int, string> p1 = {42, "hello"};
pair<int, string> p2 = make_pair(42, "hello");   // older style, same thing
auto p3 = make_pair(42, "hello");                // auto works too

// Access
p1.first;                      // 42
p1.second;                     // "hello"

// In a map, each entry IS a pair
unordered_map<string, int> mp = {{"apple", 3}, {"banana", 5}};
for (auto& [key, value] : mp) {    // structured binding (C++17)
    // key = "apple", value = 3, etc.
}
// Without structured binding:
for (auto& p : mp) {
    p.first;                    // key
    p.second;                   // value
}

// Pairs compare lexicographically (first, then second)
// Useful for sorting by two criteria
vector<pair<int,int>> v = {{3,1}, {1,5}, {3,0}};
sort(v.begin(), v.end());
// Result: {1,5}, {3,0}, {3,1}  (sorted by first, then second)
```

---

## Structured Bindings (C++17) — Unpacking Values

```cpp
// Instead of accessing .first and .second, unpack directly:

// With pairs
pair<int, string> p = {42, "hello"};
auto [num, text] = p;               // num=42, text="hello"

// With maps (each element is a pair)
unordered_map<string, int> mp = {{"a", 1}, {"b", 2}};
for (auto& [key, value] : mp) {
    cout << key << ": " << value;   // much cleaner than p.first, p.second
}

// With tuples
auto [a, b, c] = make_tuple(1, 2.0, "three");

// With find() on maps
auto it = mp.find("a");
if (it != mp.end()) {
    auto& [key, value] = *it;       // dereference iterator, then unpack
    cout << key << " " << value;
}
```

---

## The `find()` Pattern — Searching in Containers

`find()` returns an **iterator**. If the element isn't found, it returns `end()`. Always check before using!

```cpp
// === VECTOR: use std::find (free function) ===
vector<int> v = {10, 20, 30};
auto it = find(v.begin(), v.end(), 20);     // note: find(), not v.find()
if (it != v.end()) {
    int index = it - v.begin();             // 1
    int value = *it;                        // 20
} else {
    // not found
}

// === MAP/SET: use .find() (member function) ===
unordered_map<string, int> mp = {{"apple", 3}, {"banana", 5}};
auto it = mp.find("apple");                 // note: mp.find(), not find()
if (it != mp.end()) {
    string key = it->first;                 // "apple"
    int value = it->second;                 // 3
} else {
    // not found
}

// === Shortcut: .count() for just checking existence ===
if (mp.count("apple")) {                   // returns 1 (exists) or 0 (doesn't)
    // exists! access with mp["apple"]
}
// .count() is simpler when you just need yes/no
// .find() is needed when you also want the iterator (to get the value, erase, etc.)

// === Common mistake ===
auto it = mp.find("apple");
if (it != mp.end()) {
    // CORRECT: use the iterator you already have
    int val = it->second;

    // WASTEFUL: looking up again with []
    int val = mp["apple"];      // works but does a second lookup
}
```

---

## `()` vs No `()` — Functions vs Members

```cpp
// () means CALL a function
v.size();                       // call size(), returns a number
v.empty();                      // call empty(), returns true/false
v.begin();                      // call begin(), returns an iterator
s.find("hello");                // call find(), returns position

// No () means ACCESS a data member
p.first;                        // access the first element of a pair
p.second;                       // access the second element of a pair
node->val;                      // access the val field of a node
node->next;                     // access the next pointer
node->left;                     // access the left child pointer

// How to know which is which?
// - Built-in data fields (first, second, val, next, left, right) → NO ()
// - Everything else is a function → YES ()
// - If it DOES something (search, count, check) → function → ()
// - If it IS something (a stored value) → member → no ()
```

---

## References (`&`) — Avoid Copies

```cpp
// Without &: COPIES the whole thing (slow for big containers)
for (auto p : mp) { ... }              // copies each pair
for (string s : words) { ... }         // copies each string

// With &: uses a REFERENCE (fast, no copy)
for (auto& p : mp) { ... }             // reference to each pair
for (string& s : words) { ... }        // reference to each string
for (auto& [key, val] : mp) { ... }    // reference with structured binding

// const& when you only need to read:
for (const auto& p : mp) { ... }       // can't modify, but fast

// In function parameters:
void process(vector<int>& nums);        // reference: modifies original, no copy
void process(const vector<int>& nums);  // const ref: read-only, no copy
void process(vector<int> nums);         // copy: slow! copies entire vector
```

**Rule of thumb:** Always use `&` in for-each loops and function parameters unless you specifically need a copy.

---

## Common Return Types Cheat Sheet

```cpp
// What does each function return? Quick reference:

// --- Searching ---
v.find(val)         → iterator (vector: free function find())
mp.find(key)        → iterator to {key, value} pair
s.count(val)        → int (0 or 1, or more for multiset)
mp.count(key)       → int (0 or 1)

// --- Bounds (sorted containers or sorted vectors) ---
lower_bound(b, e, val)  → iterator to first element >= val
upper_bound(b, e, val)  → iterator to first element > val
s.lower_bound(val)      → iterator (set/map member version, faster)

// --- Inserting ---
v.push_back(val)        → void (nothing returned)
s.insert(val)           → pair<iterator, bool>  (iterator + was it new?)
mp.insert({k, v})       → pair<iterator, bool>
mp[key] = val           → reference to value (creates entry if missing!)

// --- Erasing ---
v.erase(it)             → iterator to next element
s.erase(it)             → iterator to next element
s.erase(val)            → int (count of elements removed)
mp.erase(key)           → int (0 or 1)

// --- Size ---
v.size()                → size_t (unsigned integer)
v.empty()               → bool
```

---

## Putting It All Together — The Two Sum Example Explained

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        unordered_map<int, int> seen;           // value → index

        for (int i = 0; i < nums.size(); i++) {
            int complement = target - nums[i];

            auto it = seen.find(complement);    // auto: deduces iterator type
                                                // .find(): returns iterator
            if (it != seen.end()) {             // check: was it found?
                                                // seen.end(), NOT nums.end()!
                return {it->second, i};         // it->second: the stored index
                                                // ->: because it's an iterator
                                                // .second: member, not function, no ()
            }
            seen[nums[i]] = i;                  // store value→index
        }
        return {};
    }
};

// Every STL concept in this solution:
// 1. auto           — deduce iterator type
// 2. .find()        — search, returns iterator
// 3. != .end()      — check if found
// 4. ->             — access through iterator
// 5. .second        — pair member (no parentheses)
// 6. seen[key]=val  — insert/update in map
// 7. &              — nums passed by reference (no copy)
// 8. {}             — return empty vector
```

---

# Part 1: Data Structures — Syntax, Operations & When to Use

Every data structure is a trade-off. The key question: **which operations do I need to be fast?**

## DS 1: Array / Vector

**What it is:** Contiguous block of memory. Elements stored side by side. Fastest random access.

**Trade-off:** Fast read by index, slow insert/delete in middle (must shift elements).

| Operation | Time | Notes |
|---|---|---|
| Access by index `v[i]` | O(1) | Direct memory offset |
| Push back | O(1) amortized | Doubles capacity when full |
| Pop back | O(1) | |
| Insert at middle | O(n) | Must shift elements right |
| Delete at middle | O(n) | Must shift elements left |
| Search (unsorted) | O(n) | Linear scan |
| Search (sorted) | O(log n) | Binary search |

```cpp
#include <vector>
#include <algorithm>

// --- Creation ---
vector<int> v;                          // empty
vector<int> v(10, 0);                   // 10 zeros
vector<int> v = {1, 2, 3, 4, 5};       // initializer list
vector<vector<int>> grid(m, vector<int>(n, 0));  // m x n 2D grid of zeros

// --- Basic operations ---
v.push_back(6);                         // add to end:       [1,2,3,4,5,6]
v.pop_back();                           // remove from end:  [1,2,3,4,5]
v.size();                               // 5
v.empty();                              // false
v.front();                              // 1  (first element)
v.back();                               // 5  (last element)

// --- Inserting and erasing ---
v.insert(v.begin() + 2, 99);           // insert 99 at index 2: [1,2,99,3,4,5]
v.erase(v.begin() + 2);                // erase index 2:        [1,2,3,4,5]
v.erase(v.begin() + 1, v.begin() + 3); // erase range [1,3):   [1,4,5]

// --- Sorting ---
sort(v.begin(), v.end());              // ascending:  [1,2,3,4,5]
sort(v.begin(), v.end(), greater<>());  // descending: [5,4,3,2,1]

// custom sort: sort by absolute value
sort(v.begin(), v.end(), [](int a, int b) {
    return abs(a) < abs(b);
});

// --- Searching (sorted array) ---
binary_search(v.begin(), v.end(), 3);          // true/false: is 3 present?
lower_bound(v.begin(), v.end(), 3);            // iterator to first >= 3
upper_bound(v.begin(), v.end(), 3);            // iterator to first > 3

// --- Useful algorithms ---
*max_element(v.begin(), v.end());               // max value
*min_element(v.begin(), v.end());               // min value
accumulate(v.begin(), v.end(), 0);              // sum (needs <numeric>)
reverse(v.begin(), v.end());                    // reverse in-place
unique(v.begin(), v.end());                     // remove adjacent dups (sort first!)

// --- 2D vector tricks ---
int rows = grid.size();
int cols = grid[0].size();
// 4-direction traversal (used in grid BFS/DFS)
int dirs[] = {0, 1, 0, -1, 0};
for (int d = 0; d < 4; d++) {
    int nr = r + dirs[d], nc = c + dirs[d + 1];
}
```

**When to use:** Default choice. Use unless you need fast insert/delete in the middle or fast lookup by value.

### When & Why: Real Use Cases

**1. Sorting unlocks easier solutions**
Many problems become trivial once sorted.
```
Problem: "Find if any two elements sum to target" (Two Sum variant on sorted array)
Unsorted: O(n²) brute force checking all pairs
Sorted:   O(n) with two pointers

[1, 4, 2, 7, 5, 3]  → sort → [1, 2, 3, 4, 5, 7], target = 9
 left=1, right=7 → sum=8 < 9 → move left
 left=2, right=7 → sum=9 = target ✓
```

**2. `lower_bound` / `upper_bound` — finding boundaries in sorted data**

These replace hand-written binary search in most cases.
```cpp
// USE CASE: Count occurrences of 5 in sorted array
vector<int> v = {1, 3, 5, 5, 5, 7, 9};
int count = upper_bound(v.begin(), v.end(), 5)    // first > 5 → points to 7 (index 5)
          - lower_bound(v.begin(), v.end(), 5);   // first >= 5 → points to 5 (index 2)
// count = 5 - 2 = 3

// USE CASE: Find nearest value to target
// sorted: [1, 3, 7, 9], target = 5
auto it = lower_bound(v.begin(), v.end(), 5);     // points to 7
// Check both neighbors: 7 and prev(it)=3, pick closer → 3

// USE CASE: Where to insert to keep sorted?
// sorted: [1, 3, 7, 9], inserting 5
auto pos = lower_bound(v.begin(), v.end(), 5);    // points to 7 (index 2)
// insert before index 2 → [1, 3, 5, 7, 9]

// USE CASE: Longest Increasing Subsequence in O(n log n)
int lengthOfLIS(vector<int>& nums) {
    vector<int> tails;
    for (int num : nums) {
        auto it = lower_bound(tails.begin(), tails.end(), num);
        if (it == tails.end()) tails.push_back(num);
        else *it = num;     // replace to keep tails as small as possible
    }
    return tails.size();
}
// [10, 9, 2, 5, 3, 7, 101, 18]
// tails: [10]→[9]→[2]→[2,5]→[2,3]→[2,3,7]→[2,3,7,101]→[2,3,7,18]
// answer: 4
```
**Memory trick:** `lower_bound` = "where does this value **start**?", `upper_bound` = "where does this value **end**?"

**3. Remove duplicates in-place — the two-pointer trick**
When you can't use extra space, use slow/fast pointers:
```cpp
// [1, 1, 2, 3, 3, 4] → [1, 2, 3, 4, ...]
int slow = 0;
for (int fast = 1; fast < nums.size(); fast++) {
    if (nums[fast] != nums[slow]) {
        slow++;
        nums[slow] = nums[fast];
    }
}
// Trace:
// [1, 1, 2, 3, 3, 4]    s=0, f=1: same, skip
//  s  f
// [1, 1, 2, 3, 3, 4]    s=0, f=2: diff! s++, copy → [1,2,2,3,3,4]
//  s     f
// [1, 2, 2, 3, 3, 4]    s=1, f=3: diff! s++, copy → [1,2,3,3,3,4]
//     s     f
// [1, 2, 3, 3, 3, 4]    s=2, f=4: same, skip
//        s     f
// [1, 2, 3, 3, 3, 4]    s=2, f=5: diff! s++, copy → [1,2,3,4,3,4]
//        s        f
// Result: first 4 elements are unique. Return slow+1 = 4.
```

**4. 2D grids — the direction array trick**
Most grid problems need to visit 4 neighbors. Instead of writing 4 if-statements:
```cpp
// Instead of this mess:
if (r+1 < m) dfs(r+1, c);
if (r-1 >= 0) dfs(r-1, c);
if (c+1 < n) dfs(c+1, c);
if (c-1 >= 0) dfs(r, c-1);

// Use this:
int dirs[] = {0, 1, 0, -1, 0};
for (int d = 0; d < 4; d++) {
    int nr = r + dirs[d], nc = c + dirs[d + 1];
    if (nr >= 0 && nr < m && nc >= 0 && nc < n)
        dfs(nr, nc);
}
// dirs encodes: right(0,1), down(1,0), left(0,-1), up(-1,0)
```

---

## DS 2: String

**What it is:** Essentially a vector of characters with extra methods for text manipulation.

| Operation | Time | Notes |
|---|---|---|
| Access `s[i]` | O(1) | |
| Append `s += "abc"` | O(k) | k = length of appended string |
| Substring `s.substr(i, len)` | O(len) | Creates a new string |
| Find `s.find("abc")` | O(n*m) | |
| Compare `s == t` | O(n) | |

```cpp
#include <string>

// --- Creation ---
string s = "hello";
string s(5, 'a');                       // "aaaaa"

// --- Basic operations ---
s.size();                               // 5 (same as s.length())
s.empty();                              // false
s.push_back('!');                       // "hello!"
s.pop_back();                           // "hello"
s += " world";                          // "hello world"

// --- Substrings and searching ---
s.substr(0, 5);                         // "hello" (start, length)
s.find("world");                        // 6 (index of first occurrence)
s.find("xyz");                          // string::npos (not found)
s.find('o');                            // 4

// --- Modification ---
s[0] = 'H';                            // "Hello world"
s.insert(5, ",");                       // "Hello, world"
s.erase(5, 1);                         // "Hello world"
reverse(s.begin(), s.end());            // "dlrow olleH"

// --- Conversion ---
to_string(123);                         // "123"
stoi("123");                            // 123 (string to int)
stol("123456789");                      // long
stod("3.14");                           // double

// --- Character checks (useful in many problems) ---
isalpha(c);    // is letter?
isdigit(c);    // is digit?
isalnum(c);    // is letter or digit?
tolower(c);    // lowercase version
toupper(c);    // uppercase version

// --- Splitting trick (no built-in split in C++) ---
stringstream ss("hello world foo");
string token;
while (ss >> token) {
    // token = "hello", "world", "foo"
}

// --- Common patterns ---
// Character frequency count
int freq[26] = {};
for (char c : s) freq[c - 'a']++;

// Check if two strings are anagrams
bool isAnagram(string s, string t) {
    if (s.size() != t.size()) return false;
    int freq[26] = {};
    for (int i = 0; i < s.size(); i++) {
        freq[s[i] - 'a']++;
        freq[t[i] - 'a']--;
    }
    for (int f : freq) if (f != 0) return false;
    return true;
}
```

### When & Why: Real Use Cases

**1. Character frequency — the `freq[26]` array trick**
Instead of a hash map, use a fixed array for lowercase letters. Faster and simpler.
```cpp
// Problem: Are two strings anagrams? ("listen" and "silent")
// Why freq array: 26 letters = 26 slots, O(1) per lookup, no hash overhead
int freq[26] = {};
for (char c : s) freq[c - 'a']++;    // c - 'a' converts 'a'→0, 'b'→1, ..., 'z'→25
for (char c : t) freq[c - 'a']--;
// If all zeros → anagram. If any non-zero → not anagram.

// This works because:  'c' - 'a' = 2,  'd' - 'a' = 3, etc.
// It maps each letter to an index 0-25.
```

**2. String building — avoid `s += char` in a loop when possible**
```cpp
// BAD: O(n²) — each += may copy the entire string
string result;
for (int i = 0; i < n; i++)
    result += someChar;     // could be O(n) each time

// GOOD: O(n) — reserve space, or use push_back (amortized O(1))
string result;
result.reserve(n);          // pre-allocate
for (int i = 0; i < n; i++)
    result.push_back(someChar);

// In practice, C++ compilers optimize += for single chars well,
// but for substrings, be aware of the cost.
```

**3. Palindrome checking — two common approaches**
```cpp
// Approach 1: Two pointers (O(n) time, O(1) space)
bool isPalindrome(string& s) {
    int left = 0, right = s.size() - 1;
    while (left < right) {
        if (s[left] != s[right]) return false;
        left++; right--;
    }
    return true;
}

// Approach 2: Reverse and compare (O(n) time, O(n) space)
string rev = s;
reverse(rev.begin(), rev.end());
return s == rev;    // simpler but uses extra space
```

**4. The `c - 'a'` and `c - '0'` conversions**
```cpp
// These come up constantly:
char c = 'f';
int index = c - 'a';       // 5 (maps 'a'-'z' to 0-25)
int digit = '7' - '0';     // 7 (maps '0'-'9' to 0-9)

// Reverse:
char letter = 'a' + 5;     // 'f'
char digitChar = '0' + 7;  // '7'
```

---

## DS 3: Hash Map (`unordered_map`) and Hash Set (`unordered_set`)

**What it is:** Key-value storage using hashing. Average O(1) for insert, delete, and lookup.

**Trade-off:** Fast everything, but unordered. Uses more memory. Worst case O(n) due to hash collisions.

| Operation | Average | Worst |
|---|---|---|
| Insert | O(1) | O(n) |
| Delete | O(1) | O(n) |
| Lookup | O(1) | O(n) |
| Iterate all | O(n) | O(n) |

```cpp
#include <unordered_map>
#include <unordered_set>

// --- Hash Map (key → value) ---
unordered_map<string, int> mp;

mp["apple"] = 3;                        // insert or update
mp["banana"] = 5;
mp.insert({"cherry", 2});              // insert (won't overwrite if exists)

mp["apple"];                            // 3 (access value)
mp.count("apple");                      // 1 (exists) or 0 (doesn't)
mp.find("apple") != mp.end();          // true (another way to check)

mp.erase("apple");                      // remove key
mp.size();                              // number of entries

// --- Iteration ---
for (auto& [key, value] : mp) {         // structured bindings (C++17)
    cout << key << ": " << value << endl;
}

// --- Common pattern: frequency count ---
unordered_map<int, int> freq;
for (int num : nums) freq[num]++;       // count occurrences

// --- Common pattern: two sum with hash map ---
unordered_map<int, int> seen;           // value → index
for (int i = 0; i < nums.size(); i++) {
    int complement = target - nums[i];
    if (seen.count(complement))
        return {seen[complement], i};
    seen[nums[i]] = i;
}

// --- Hash Set (just keys, no values) ---
unordered_set<int> s;

s.insert(5);                            // add element
s.erase(5);                             // remove element
s.count(5);                             // 1 or 0
s.size();

// --- Common pattern: check for duplicates ---
unordered_set<int> seen;
for (int num : nums)
    if (!seen.insert(num).second)       // insert returns {iterator, was_inserted}
        return true;                    // duplicate found
```

**When to use:**
- Need O(1) lookup by key → hash map
- Need to track "have I seen this?" → hash set
- Frequency counting → hash map
- Two sum / complement problems → hash map

**Ordered alternative:** Use `map` / `set` (red-black tree) when you need sorted order. O(log n) operations instead of O(1).

### When & Why: Real Use Cases

**1. Two Sum — THE classic hash map problem**
"Find two numbers that add up to target." Without hash map: O(n²). With: O(n).
```cpp
// nums = [2, 7, 11, 15], target = 9
// Walk through array. For each number, ask: "have I seen my complement?"
unordered_map<int, int> seen;   // value → index
for (int i = 0; i < nums.size(); i++) {
    int complement = target - nums[i];
    if (seen.count(complement))
        return {seen[complement], i};
    seen[nums[i]] = i;
}
// i=0: need 9-2=7, not seen. Store {2:0}
// i=1: need 9-7=2, seen[2]=0! Return {0, 1} ✓
//
// Why it works: instead of checking all pairs, we store what we've seen
// and check in O(1) if the complement exists.
```

**2. Frequency count — "top K", "most common", "group by"**
```cpp
// Problem: Find the most frequent element
// nums = [1, 3, 2, 3, 1, 3]
unordered_map<int, int> freq;
for (int n : nums) freq[n]++;
// freq = {1:2, 3:3, 2:1}
// Now find max: iterate freq, track highest count → 3 appears 3 times

// Problem: Group anagrams ("eat","tea","tan","ate","nat","bat")
// Key insight: anagrams have the same sorted form
unordered_map<string, vector<string>> groups;
for (string& s : strs) {
    string key = s;
    sort(key.begin(), key.end());   // "eat" → "aet", "tea" → "aet"
    groups[key].push_back(s);
}
// groups = {"aet": ["eat","tea","ate"], "ant": ["tan","nat"], "abt": ["bat"]}
```

**3. Hash set — "have I seen this before?"**
```cpp
// Problem: Contains Duplicate — return true if any value appears twice
unordered_set<int> seen;
for (int num : nums) {
    if (seen.count(num)) return true;   // already seen!
    seen.insert(num);
}
return false;

// Problem: Longest Consecutive Sequence [100, 4, 200, 1, 3, 2] → 4 (sequence 1,2,3,4)
unordered_set<int> numSet(nums.begin(), nums.end());
int best = 0;
for (int num : numSet) {
    if (numSet.count(num - 1)) continue;    // only start from sequence beginning
    int len = 1;
    while (numSet.count(num + len)) len++;
    best = max(best, len);
}
// num=1: no 0 in set → start counting: 1,2,3,4 → length 4
// num=4: 3 exists → skip (not a starting point)
// O(n) because each element is visited at most twice
```

**4. When to use `map` (ordered) vs `unordered_map`**
```
Use unordered_map when:     Use map when:
  - Just need lookup/insert   - Need sorted order of keys
  - Don't care about order    - Need lower_bound/upper_bound
  - Want O(1) average         - Need to iterate in order
                               - Keys aren't hashable (pairs, etc.)
```

---

## DS 4: Stack

**What it is:** LIFO (Last In, First Out). Like a stack of plates — you can only add/remove from the top.

| Operation | Time |
|---|---|
| Push (add to top) | O(1) |
| Pop (remove from top) | O(1) |
| Top (peek) | O(1) |

```cpp
#include <stack>

stack<int> st;

st.push(10);                            // [10]
st.push(20);                            // [10, 20]
st.push(30);                            // [10, 20, 30]

st.top();                               // 30 (peek, doesn't remove)
st.pop();                               // removes 30 → [10, 20]
st.size();                              // 2
st.empty();                             // false

// --- Common pattern: matching parentheses ---
bool isValid(string s) {
    stack<char> st;
    for (char c : s) {
        if (c == '(' || c == '[' || c == '{') {
            st.push(c);
        } else {
            if (st.empty()) return false;
            char top = st.top(); st.pop();
            if ((c == ')' && top != '(') ||
                (c == ']' && top != '[') ||
                (c == '}' && top != '{'))
                return false;
        }
    }
    return st.empty();
}

// --- Common pattern: evaluate reverse polish notation ---
// push numbers, pop two operands when you see an operator
```

**When to use:**
- Matching brackets/parentheses
- Undo operations
- Expression evaluation
- Monotonic stack problems (next greater element, largest rectangle)
- DFS iteratively (instead of recursion)

### When & Why: Real Use Cases

**1. Matching & nesting problems — "does this close what was last opened?"**
```cpp
// The key insight: the LAST thing opened must be the FIRST thing closed → LIFO → stack
// Valid:   ( [ { } ] )   — each closer matches the most recent opener
// Invalid: ( [ { ] } )   — ] tries to close [ but { was opened more recently

// This extends beyond brackets:
// - HTML tag matching: <div><span></span></div>
// - Directory navigation: cd a/b/c, then cd .. goes back to b
```

**2. Monotonic stack — "for each element, find the next greater/smaller"**
```cpp
// Problem: Stock Span — for each day, how many consecutive days before had price ≤ today?
// prices = [100, 80, 60, 70, 60, 75, 85]
// spans  = [  1,  1,  1,  2,  1,  4,  6]
//
// Day 6 (price 85): look back — 75,60,70,60,80 are all ≤ 85 → span = 6
// Brute force: O(n²). With monotonic stack: O(n).
//
// Stack stores indices of "walls" — prices we haven't been able to see past.
// When current price beats the stack top, pop it (we can see past it now).
stack<int> st;
vector<int> span(n);
for (int i = 0; i < n; i++) {
    while (!st.empty() && prices[st.top()] <= prices[i])
        st.pop();
    span[i] = st.empty() ? i + 1 : i - st.top();
    st.push(i);
}
```

**3. Expression evaluation — "calculate 3 + 2 * 4"**
```cpp
// Two stacks: one for numbers, one for operators
// When you see a higher-precedence op on the stack, evaluate first
// This is how calculators work internally
// Simplified for + and *:
// "3 + 2 * 4" → push 3, push +, push 2, see * (higher precedence, push),
//                push 4, end → evaluate: 2*4=8, then 3+8=11
```

**4. DFS without recursion — use stack to simulate the call stack**
```cpp
// Recursive DFS:                       Stack-based DFS:
// void dfs(node) {                     stack<Node*> st;
//     visit(node);                     st.push(root);
//     for (child : node.children)      while (!st.empty()) {
//         dfs(child);                      auto node = st.top(); st.pop();
// }                                        visit(node);
//                                          for (child : node.children)
//                                              st.push(child);
//                                      }
// Use when recursion might hit stack overflow (deep trees/graphs)
```

---

## DS 5: Queue

**What it is:** FIFO (First In, First Out). Like a line at a store — first person in is first person out.

| Operation | Time |
|---|---|
| Push (add to back) | O(1) |
| Pop (remove from front) | O(1) |
| Front (peek front) | O(1) |
| Back (peek back) | O(1) |

```cpp
#include <queue>

queue<int> q;

q.push(10);                             // back: [10]
q.push(20);                             // back: [10, 20]
q.push(30);                             // back: [10, 20, 30]

q.front();                              // 10
q.back();                               // 30
q.pop();                                // removes 10 → [20, 30]
q.size();                               // 2

// --- Common pattern: BFS level-by-level ---
queue<TreeNode*> q;
q.push(root);
while (!q.empty()) {
    int levelSize = q.size();           // number of nodes at this level
    for (int i = 0; i < levelSize; i++) {
        auto node = q.front(); q.pop();
        // process node
        if (node->left) q.push(node->left);
        if (node->right) q.push(node->right);
    }
    // one level fully processed
}
```

**When to use:**
- BFS (always uses a queue)
- Level-order traversal
- Sliding window maximum (use deque)

### When & Why: Real Use Cases

**1. BFS — "find the shortest path in an unweighted graph/grid"**
```
Why queue? BFS explores level by level. The queue ensures we process all nodes
at distance 1 before distance 2, distance 2 before distance 3, etc.

Problem: Shortest path in a maze from S to E
###########
#S  #     #
# # # ### #
# #   #   #
# ##### # #
#       # E#
###########

BFS queue processes: (start is distance 0)
  Level 0: S
  Level 1: all cells 1 step from S
  Level 2: all cells 2 steps from S
  ...
First time we reach E → that's the shortest path!

Why NOT DFS? DFS might find a path, but it could be a long winding one.
BFS guarantees the SHORTEST path is found first.
```

**2. Level-order processing — "do something per level of a tree"**
```cpp
// Problem: Find the average value at each level of a binary tree
//        3
//       / \
//      9  20
//        /  \
//       15   7
// Output: [3.0, 14.5, 11.0]

queue<TreeNode*> q;
q.push(root);
while (!q.empty()) {
    int size = q.size();            // KEY: capture level size before processing
    double sum = 0;
    for (int i = 0; i < size; i++) {
        auto node = q.front(); q.pop();
        sum += node->val;
        if (node->left) q.push(node->left);
        if (node->right) q.push(node->right);
    }
    result.push_back(sum / size);
}
// The size = q.size() trick separates levels — without it, you can't tell
// where one level ends and the next begins.
```

### Deque (Double-ended queue)
```cpp
#include <deque>

deque<int> dq;
dq.push_back(1);                        // add to back
dq.push_front(0);                       // add to front
dq.pop_back();                          // remove from back
dq.pop_front();                         // remove from front
dq.front();                             // peek front
dq.back();                              // peek back
dq[i];                                  // random access O(1)

// --- Common pattern: sliding window maximum ---
// Maintain a monotonic decreasing deque of indices
deque<int> dq;  // stores indices
for (int i = 0; i < nums.size(); i++) {
    while (!dq.empty() && dq.front() <= i - k) dq.pop_front();  // out of window
    while (!dq.empty() && nums[dq.back()] <= nums[i]) dq.pop_back(); // smaller = useless
    dq.push_back(i);
    if (i >= k - 1) result.push_back(nums[dq.front()]);  // front = max of window
}
```

---

## DS 6: Priority Queue (Heap)

**What it is:** A tree-based structure where the top element is always the min (or max). Not fully sorted — only guarantees the top.

**Trade-off:** Fast access to min/max. But can't search or access arbitrary elements.

| Operation | Time |
|---|---|
| Push | O(log n) |
| Pop (remove top) | O(log n) |
| Top (peek) | O(1) |

```cpp
#include <queue>

// --- Max-Heap (default in C++) ---
priority_queue<int> maxHeap;
maxHeap.push(3);                         // {3}
maxHeap.push(1);                         // {3, 1}
maxHeap.push(5);                         // {5, 3, 1}
maxHeap.top();                           // 5 (largest)
maxHeap.pop();                           // removes 5

// --- Min-Heap ---
priority_queue<int, vector<int>, greater<int>> minHeap;
minHeap.push(3);                         // {3}
minHeap.push(1);                         // {1, 3}
minHeap.push(5);                         // {1, 3, 5}
minHeap.top();                           // 1 (smallest)

// --- Custom comparator heap ---
// Example: min-heap of pairs, sorted by second value
auto cmp = [](pair<int,int>& a, pair<int,int>& b) {
    return a.second > b.second;          // > for min-heap, < for max-heap
};
priority_queue<pair<int,int>, vector<pair<int,int>>, decltype(cmp)> pq(cmp);

// --- Common pattern: kth largest ---
priority_queue<int, vector<int>, greater<int>> minHeap;  // size k
for (int num : nums) {
    minHeap.push(num);
    if (minHeap.size() > k) minHeap.pop();  // evict smallest
}
// minHeap.top() = kth largest
```

**When to use:**
- Need repeated access to min or max element
- Kth largest/smallest problems
- Merge K sorted structures
- Dijkstra's algorithm
- Task scheduling

### When & Why: Real Use Cases

**1. Kth largest — "why min-heap and not max-heap?"**
```
Problem: Find the 3rd largest in [3, 1, 5, 2, 8, 7, 4]
Answer: 5

Intuition: Keep a min-heap of size 3 (the top 3 largest elements).
The smallest of those 3 = the 3rd largest overall.

Walk through:
  push 3 → heap: [3]
  push 1 → heap: [1, 3]
  push 5 → heap: [1, 3, 5]          (size = k = 3, full)
  push 2 → heap: [1, 2, 3, 5] → pop min → [2, 3, 5]
  push 8 → heap: [2, 3, 5, 8] → pop min → [3, 5, 8]
  push 7 → heap: [3, 5, 7, 8] → pop min → [5, 7, 8]
  push 4 → heap: [4, 5, 7, 8] → pop min → [5, 7, 8]

  top() = 5 = 3rd largest ✓

Why min-heap? Because we want the SMALLEST of the top K to be easily accessible.
When a new number comes in and it's bigger than the heap top, it deserves to be
in the top K, so we swap it in (push + pop).
If we used max-heap, we'd have no efficient way to evict the smallest of the top K.
```

**2. Running median — "split into two halves"**
```
Problem: Numbers arrive one by one. After each, report the median.

Idea: Keep two heaps:
  - max-heap "lo": stores the smaller half
  - min-heap "hi": stores the larger half

  lo.top() = largest of small half
  hi.top() = smallest of large half

  Numbers: 5, 2, 8, 1, 4

  Add 5: lo=[5], hi=[]           median = 5
  Add 2: lo=[2], hi=[5]          median = (2+5)/2 = 3.5
  Add 8: lo=[2,5], hi=[8]        median = 5  (lo has more, take lo.top)
  Add 1: lo=[1,2], hi=[5,8]      median = (2+5)/2 = 3.5
  Add 4: lo=[1,2,4], hi=[5,8]    median = 4

Why two heaps? We need the middle element(s). By splitting into halves,
the middle is always at the top of one or both heaps → O(1) access.
```

**3. Dijkstra's algorithm — "always process the nearest unvisited node"**
```
Graph:  A --1-- B --2-- D
        |       |
        4       1
        |       |
        C --3-- E

Shortest path from A to D?
Min-heap processes: (distance, node)
  (0, A) → neighbors: B(1), C(4) → push (1,B), (4,C)
  (1, B) → neighbors: D(3), E(2) → push (3,D), (2,E)
  (2, E) → neighbors: C(5) → 5 > 4 already known, skip
  (3, D) → FOUND! Distance = 3

Why heap? We need to always pick the unvisited node with smallest distance.
Heap gives us that in O(log n) instead of O(n) linear scan.
```

---

## DS 7: Linked List

**What it is:** Nodes connected by pointers. Each node stores data + pointer to next (and optionally previous).

**Trade-off:** O(1) insert/delete if you have the pointer, but O(n) to find an element (no random access).

| Operation | Time | Notes |
|---|---|---|
| Access by index | O(n) | Must traverse from head |
| Insert at head | O(1) | |
| Insert after known node | O(1) | |
| Delete known node | O(1) | |
| Search | O(n) | |

```cpp
// --- Node definition ---
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

// --- Dummy head trick (avoids edge cases for head insertion) ---
ListNode dummy(0);
ListNode* tail = &dummy;
// ... build list by setting tail->next = new node; tail = tail->next;
return dummy.next;  // real head

// --- Reverse a linked list ---
ListNode* reverse(ListNode* head) {
    ListNode* prev = nullptr;
    while (head) {
        ListNode* next = head->next;
        head->next = prev;
        prev = head;
        head = next;
    }
    return prev;
}

// --- Find middle (fast/slow pointers) ---
ListNode* findMiddle(ListNode* head) {
    auto slow = head, fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;  // middle node
}

// --- Detect cycle ---
bool hasCycle(ListNode* head) {
    auto slow = head, fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}

// --- Merge two sorted lists ---
ListNode* merge(ListNode* l1, ListNode* l2) {
    ListNode dummy(0);
    auto* tail = &dummy;
    while (l1 && l2) {
        if (l1->val <= l2->val) { tail->next = l1; l1 = l1->next; }
        else { tail->next = l2; l2 = l2->next; }
        tail = tail->next;
    }
    tail->next = l1 ? l1 : l2;
    return dummy.next;
}
```

**When to use:** When the problem gives you linked lists. Key techniques: dummy head, fast/slow pointers, reversal.

### When & Why: Real Use Cases

**1. Dummy head — "eliminates all head-related edge cases"**
```cpp
// Problem: Remove all nodes with value 6 from [6, 1, 6, 3, 6]
// Without dummy: need special handling if head itself should be removed
// With dummy: treat every node the same way

ListNode dummy(0);
dummy.next = head;
ListNode* prev = &dummy;
while (prev->next) {
    if (prev->next->val == 6)
        prev->next = prev->next->next;  // skip the node
    else
        prev = prev->next;
}
return dummy.next;
// dummy → 6 → 1 → 6 → 3 → 6
// dummy → 1 → 6 → 3 → 6      (removed first 6)
// dummy → 1 → 3 → 6            (removed second 6)
// dummy → 1 → 3                 (removed third 6)
// return 1 → 3 ✓
```

**2. Fast/slow pointers — "tortoise and hare"**
```
Why does this work for finding the middle?
Slow moves 1 step, fast moves 2 steps.
When fast reaches the end, slow is at the halfway point.

1 → 2 → 3 → 4 → 5
s
f
1 → 2 → 3 → 4 → 5
    s
        f
1 → 2 → 3 → 4 → 5
        s              ← slow is at middle!
                f      ← fast is at end

Why does this detect cycles?
If there's a cycle, fast will eventually "lap" slow inside the cycle.
Like two runners on a circular track — the faster one catches up.

1 → 2 → 3 → 4
        ↑     ↓
        6 ← 5

s=1 f=1 → s=2 f=3 → s=3 f=5 → s=4 f=3 → s=5 f=5 MEET! cycle detected.
```

**3. Reversal — "the three-pointer dance"**
```
Reverse: 1 → 2 → 3 → 4 → null

Step by step (prev, curr, next):
prev=null   curr=1    next=2     →  null ← 1   2 → 3 → 4
prev=1      curr=2    next=3     →  null ← 1 ← 2   3 → 4
prev=2      curr=3    next=4     →  null ← 1 ← 2 ← 3   4
prev=3      curr=4    next=null  →  null ← 1 ← 2 ← 3 ← 4

Return prev (4), which is the new head.

This pattern is used in: Reverse Linked List, Palindrome Linked List
(reverse second half, compare), Reverse Nodes in K-Group.
```

---

## DS 8: Tree & Binary Search Tree

**What it is:** Hierarchical structure. Each node has children. A binary tree has at most 2 children. A BST keeps left < root < right.

```cpp
// --- Node definition ---
struct TreeNode {
    int val;
    TreeNode* left;
    TreeNode* right;
    TreeNode(int x) : val(x), left(nullptr), right(nullptr) {}
};

// --- Three traversal orders (DFS) ---
void inorder(TreeNode* node) {           // left → root → right
    if (!node) return;                   // BST: gives sorted order!
    inorder(node->left);
    process(node->val);
    inorder(node->right);
}

void preorder(TreeNode* node) {          // root → left → right
    if (!node) return;                   // useful for serialization
    process(node->val);
    preorder(node->left);
    preorder(node->right);
}

void postorder(TreeNode* node) {         // left → right → root
    if (!node) return;                   // useful for deletion, calculating sizes
    postorder(node->left);
    postorder(node->right);
    process(node->val);
}

// --- BFS (level-order) ---
void levelOrder(TreeNode* root) {
    queue<TreeNode*> q;
    q.push(root);
    while (!q.empty()) {
        int size = q.size();
        for (int i = 0; i < size; i++) {
            auto node = q.front(); q.pop();
            process(node->val);
            if (node->left) q.push(node->left);
            if (node->right) q.push(node->right);
        }
    }
}

// --- Height / Max Depth ---
int height(TreeNode* node) {
    if (!node) return 0;
    return 1 + max(height(node->left), height(node->right));
}

// --- BST: Search ---
TreeNode* search(TreeNode* node, int target) {
    if (!node || node->val == target) return node;
    if (target < node->val) return search(node->left, target);
    return search(node->right, target);
}

// --- BST: Validate ---
bool isValidBST(TreeNode* node, long lo = LONG_MIN, long hi = LONG_MAX) {
    if (!node) return true;
    if (node->val <= lo || node->val >= hi) return false;
    return isValidBST(node->left, lo, node->val) &&
           isValidBST(node->right, node->val, hi);
}
```

**Key properties to know:**
- **Complete binary tree:** All levels full except possibly last (filled left to right). Used in heaps.
- **Balanced BST:** Height = O(log n). Guarantees O(log n) search/insert.
- **Inorder traversal of BST** = sorted order. This is used in many BST problems.

### When & Why: Real Use Cases

**1. Traversal order matters — each one answers a different question**
```
Given this tree:
        4
       / \
      2    6
     / \  / \
    1   3 5   7

Inorder   (Left, Root, Right): 1, 2, 3, 4, 5, 6, 7  → SORTED! Use for BST problems.
Preorder  (Root, Left, Right): 4, 2, 1, 3, 6, 5, 7  → Use to COPY/SERIALIZE a tree.
Postorder (Left, Right, Root): 1, 3, 2, 5, 7, 6, 4  → Use to DELETE or compute sizes (children first).
Level-order (BFS):             4, 2, 6, 1, 3, 5, 7  → Use for level-based questions.
```

**2. The recursive "pass info up" pattern**
```cpp
// Most tree problems follow this: compute something for left and right subtrees,
// then combine at the current node.

// Problem: Is this tree balanced? (height difference ≤ 1 at every node)
int checkHeight(TreeNode* node) {
    if (!node) return 0;
    int left = checkHeight(node->left);
    if (left == -1) return -1;                      // left subtree unbalanced
    int right = checkHeight(node->right);
    if (right == -1) return -1;                     // right subtree unbalanced
    if (abs(left - right) > 1) return -1;           // THIS node unbalanced
    return 1 + max(left, right);                    // pass height up
}
// Return -1 as a "signal" that something is wrong — it bubbles up.
// This avoids computing height separately + checking balance = O(n) instead of O(n²).
```

**3. BST property — "everything left is smaller, everything right is bigger"**
```cpp
// Problem: Kth Smallest Element in BST
// Key insight: inorder traversal gives sorted order → kth element in inorder = answer
int kthSmallest(TreeNode* root, int k) {
    stack<TreeNode*> st;
    auto node = root;
    while (node || !st.empty()) {
        while (node) { st.push(node); node = node->left; }    // go left
        node = st.top(); st.pop();
        if (--k == 0) return node->val;                       // kth node
        node = node->right;
    }
    return -1;
}
// For k=3 on tree [4,2,6,1,3,5,7]:
// inorder: 1, 2, 3 ← stop here! answer = 3
```

**4. When to choose DFS vs BFS for trees**
```
Use DFS (recursion) when:         Use BFS (queue) when:
  - Path from root to leaf          - Level-by-level processing
  - Comparing subtree properties    - Finding nearest node
  - Most "does this tree have..."   - Zigzag/right-side view
  - Easier to write recursively     - Need to know which level you're on
```

---

## DS 9: Graph Representations

**What it is:** Nodes (vertices) connected by edges. Can be directed or undirected, weighted or unweighted.

```cpp
// --- Adjacency List (most common, best for sparse graphs) ---
int n = 5; // number of nodes
vector<vector<int>> adj(n);              // unweighted
adj[0].push_back(1);                     // edge 0 → 1
adj[0].push_back(2);                     // edge 0 → 2
adj[1].push_back(3);
// For undirected: add both directions
adj[0].push_back(1); adj[1].push_back(0);

// --- Weighted adjacency list ---
vector<vector<pair<int,int>>> adj(n);    // {neighbor, weight}
adj[0].push_back({1, 5});               // edge 0→1, weight 5

// --- Building from edge list (common input format) ---
// edges = [[0,1], [0,2], [1,3], [2,3]]
vector<vector<int>> adj(n);
for (auto& e : edges) {
    adj[e[0]].push_back(e[1]);
    adj[e[1]].push_back(e[0]);          // undirected
}

// --- Adjacency Matrix (use when n is small or need O(1) edge lookup) ---
vector<vector<int>> adj(n, vector<int>(n, 0));
adj[0][1] = 1;                          // edge 0 → 1
adj[0][1] = 5;                          // weighted edge

// --- Degree counting ---
// Undirected: degree of node i = adj[i].size()
// Directed: indegree[v]++ for each edge u→v (used in topological sort)
```

**Choosing representation:**

| | Adjacency List | Adjacency Matrix |
|---|---|---|
| Space | O(V + E) | O(V²) |
| Check if edge exists | O(degree) | O(1) |
| Iterate neighbors | O(degree) | O(V) |
| Best for | Sparse graphs (most problems) | Dense graphs, small n |

### When & Why: Real Use Cases

**1. Recognizing a graph problem in disguise**
```
Many problems don't say "graph" but ARE graph problems:

"Word Ladder: change 'hit' to 'cog' one letter at a time"
→ Graph! Each word is a node, edges connect words that differ by 1 letter.
  hit — hot — dot — dog — cog       (BFS → shortest path = 5)

"Course prerequisites: [1→0] means take 0 before 1"
→ Graph! Courses are nodes, prerequisites are directed edges.
  Topological sort tells you the valid order.

"Number of islands in a grid"
→ Graph! Each land cell is a node, adjacent land cells are edges.
  DFS/BFS from each unvisited land cell counts components.

"Friend circles / social network groups"
→ Graph! People are nodes, friendships are edges.
  Connected components via Union-Find or DFS.
```

**2. Building the adjacency list — the first step in every graph problem**
```cpp
// Input format 1: Edge list [[0,1],[0,2],[1,3]]
vector<vector<int>> adj(n);
for (auto& edge : edges) {
    adj[edge[0]].push_back(edge[1]);
    adj[edge[1]].push_back(edge[0]);    // omit for directed graphs
}

// Input format 2: Prerequisites [[1,0],[2,0],[3,1]]  (take 0 before 1, etc.)
vector<vector<int>> adj(n);
for (auto& p : prerequisites) {
    adj[p[1]].push_back(p[0]);          // p[1] must come before p[0]
}

// Input format 3: Grid (implicit graph — no adjacency list needed)
// Each cell (r,c) has neighbors (r±1,c) and (r,c±1)
// Just use the direction array trick to visit neighbors
```

**3. Detecting cycles — the key question for many graph problems**
```
Undirected graph: cycle exists if during DFS, you visit an already-visited node
                  that isn't the parent you came from.

Directed graph:   cycle exists if during DFS, you reach a node that's currently
                  "in progress" (on the current path). Use 3-state coloring:
                  WHITE=unvisited, GRAY=in-progress, BLACK=done.
                  Hitting a GRAY node = cycle!

Why it matters:
  - Course schedule: cycle = impossible to complete all courses
  - Topological sort: only works on DAGs (Directed Acyclic Graphs)
  - Deadlock detection: cycle in resource dependency = deadlock
```

---

## DS 10: Ordered Containers (`set`, `map`)

**What it is:** Self-balancing BST (red-black tree). Elements are always sorted. O(log n) for everything.

```cpp
#include <set>
#include <map>

// --- Ordered Set ---
set<int> s;
s.insert(5);                             // {5}
s.insert(3);                             // {3, 5}
s.insert(7);                             // {3, 5, 7}

s.count(5);                              // 1
s.erase(5);                              // {3, 7}

// --- Ordered iteration (always sorted) ---
for (int x : s) cout << x << " ";       // 3 7

// --- Powerful: lower_bound and upper_bound ---
s = {1, 3, 5, 7, 9};
auto it = s.lower_bound(4);             // iterator to 5 (first >= 4)
auto it2 = s.upper_bound(5);            // iterator to 7 (first > 5)
auto it3 = s.lower_bound(5);            // iterator to 5 (first >= 5)

// predecessor: prev(it) gives the element before
// successor: next(it) gives the element after
// s.begin() → smallest, prev(s.end()) or *s.rbegin() → largest

// --- Multiset (allows duplicates) ---
multiset<int> ms;
ms.insert(3); ms.insert(3);             // {3, 3}
ms.erase(ms.find(3));                   // removes ONE 3 → {3}
ms.erase(3);                            // removes ALL 3s → {}

// --- Ordered Map ---
map<string, int> m;
m["banana"] = 2;
m["apple"] = 5;
// iteration is sorted by key: apple → banana

// --- Common usage: sliding window with sorted access ---
// Use multiset when you need the min/max of a sliding window
// and also need to remove arbitrary elements
```

**When to use over unordered variants:**
- Need elements in sorted order
- Need `lower_bound` / `upper_bound` (finding nearest element)
- Need min/max with dynamic insert/delete
- Need range queries

### When & Why: Real Use Cases

**1. `set` with `lower_bound` — "find nearest element dynamically"**
```cpp
// Problem: You're inserting numbers one by one. After each insert,
// find the smallest difference between any two numbers.
// set keeps everything sorted → just check neighbors of the new element.

set<int> s;
int minDiff = INT_MAX;
for (int num : nums) {
    auto it = s.insert(num).first;
    if (it != s.begin())
        minDiff = min(minDiff, num - *prev(it));    // left neighbor
    if (next(it) != s.end())
        minDiff = min(minDiff, *next(it) - num);    // right neighbor
}
// Insert 5: s={5},         minDiff=INT_MAX
// Insert 1: s={1,5},       minDiff=4      (|5-1|)
// Insert 3: s={1,3,5},     minDiff=2      (|3-1| or |5-3|)
// Insert 8: s={1,3,5,8},   minDiff=2
//
// Without set: after each insert, scan all pairs → O(n²)
// With set: just check 2 neighbors → O(n log n) total
```

**2. `multiset` — "sorted container that allows duplicates"**
```cpp
// Problem: Sliding Window Median — find median of each window of size k
// Need: sorted access + ability to remove specific elements as window slides
//
// Why not priority_queue? Heap can't remove arbitrary elements efficiently.
// Multiset can: erase(find(val)) removes exactly one copy in O(log n).

multiset<int> window;
for (int i = 0; i < nums.size(); i++) {
    window.insert(nums[i]);
    if (window.size() > k)
        window.erase(window.find(nums[i - k]));    // remove element leaving window
    if (window.size() == k) {
        auto mid = next(window.begin(), k / 2);    // median position
        // ... compute median
    }
}
// multiset = "heap that supports deletion" — use when you need both.
```

**3. `map` — "sorted key-value store"**
```cpp
// Problem: My Calendar — book events, return false if double-booking
// Events: [start, end). Check if new event overlaps any existing.
map<int, int> calendar;  // start → end, sorted by start

bool book(int start, int end) {
    auto it = calendar.lower_bound(start);

    // Check event starting at or after 'start'
    if (it != calendar.end() && it->first < end) return false;  // overlap

    // Check event starting before 'start'
    if (it != calendar.begin() && prev(it)->second > start) return false;

    calendar[start] = end;
    return true;
}
// lower_bound finds the right spot in O(log n)
// Only need to check 2 neighbors — the power of sorted order
```

**4. When to use `set` vs `priority_queue` vs `unordered_set`**
```
Need just "is X present?" →  unordered_set  (O(1))
Need min/max only        →  priority_queue  (O(1) peek, O(log n) push/pop)
Need min/max AND delete  →  multiset        (O(log n) everything)
Need sorted iteration    →  set/map         (O(log n) everything)
Need nearest element     →  set/map         (lower_bound in O(log n))
```

---

## DS Quick Reference: Which to Use?

| I need... | Use | Why |
|---|---|---|
| Fast access by index | `vector` | O(1) random access |
| Fast lookup "is X present?" | `unordered_set` | O(1) average |
| Fast lookup key → value | `unordered_map` | O(1) average |
| Sorted order + fast insert | `set` / `map` | O(log n), always sorted |
| Fast min or max | `priority_queue` | O(1) peek, O(log n) push/pop |
| LIFO (last in, first out) | `stack` | O(1) push/pop |
| FIFO (first in, first out) | `queue` | O(1) push/pop |
| Both ends fast | `deque` | O(1) push/pop front and back |
| Sorted + allow duplicates | `multiset` | O(log n), keeps all copies |
| Fast insert/delete at position | `list` (linked list) | O(1) with iterator |

---

# Part 2: Core Concepts — What You Need to Understand

Before diving into patterns, make sure you understand these foundational concepts.

## Concept 1: Time & Space Complexity (Big-O)

**What it measures:** How the runtime/memory grows as input size n increases. We care about the **worst case** and **dominant term**.

**The hierarchy (fastest to slowest):**
```
O(1) < O(log n) < O(n) < O(n log n) < O(n²) < O(n³) < O(2ⁿ) < O(n!)
```

**How to determine complexity:**

```cpp
// O(1) — constant: doesn't depend on n
int x = arr[0] + arr[1];

// O(log n) — logarithmic: halves the problem each step
while (n > 0) { n /= 2; }              // binary search

// O(n) — linear: one pass through data
for (int i = 0; i < n; i++) { }

// O(n log n) — linearithmic: sort, then process
sort(arr.begin(), arr.end());           // comparison-based sort

// O(n²) — quadratic: nested loops
for (int i = 0; i < n; i++)
    for (int j = 0; j < n; j++) { }

// O(2ⁿ) — exponential: subsets, brute-force recursion
// O(n!) — factorial: permutations
```

**Quick rules:**
- Single loop over n → O(n)
- Nested loop over n → O(n²)
- Halving each step → O(log n)
- Loop × halving → O(n log n)
- Each element branches into 2 choices → O(2ⁿ)

**Constraint → complexity mapping (use this in contests):**

| n ≤ | Max complexity | Typical approach |
|---|---|---|
| 10 | O(n!) | Brute force, permutations |
| 20 | O(2ⁿ) | Bitmask, backtracking |
| 100 | O(n³) | Triple nested loops, Floyd-Warshall |
| 1,000 | O(n²) | DP with 2D table, brute force pairs |
| 10,000 | O(n²) barely | Careful O(n²) or optimize |
| 100,000 | O(n log n) | Sort + binary search, segment tree |
| 1,000,000 | O(n) | Hash map, two pointers, prefix sum |
| 10⁸+ | O(n) or O(log n) | Math, binary search on answer |

---

## Concept 2: Recursion

**What it is:** A function that calls itself. Every recursive solution has:
1. **Base case** — when to stop
2. **Recursive case** — break problem into smaller subproblems

```cpp
// Factorial: n! = n * (n-1)!
int factorial(int n) {
    if (n <= 1) return 1;               // base case
    return n * factorial(n - 1);        // recursive case
}

// Fibonacci: fib(n) = fib(n-1) + fib(n-2)
int fib(int n) {
    if (n <= 1) return n;               // base cases: fib(0)=0, fib(1)=1
    return fib(n - 1) + fib(n - 2);    // O(2ⁿ) — bad! use DP to fix
}

// Power: x^n
int power(int x, int n) {
    if (n == 0) return 1;
    int half = power(x, n / 2);
    if (n % 2 == 0) return half * half;         // x^n = (x^(n/2))²
    else return half * half * x;                 // x^n = (x^(n/2))² * x
}
// O(log n) — fast exponentiation!
```

**How to think recursively:**
1. Assume the recursive call works correctly for smaller inputs (leap of faith)
2. Use that result to solve the current problem
3. Handle the smallest case directly (base case)

**The call stack:** Each recursive call adds a frame to the stack. Too deep = stack overflow. Max safe depth ≈ 10,000 in most systems.

---

## Concept 3: Backtracking

**What it is:** Recursion + "undo". Explore all possible paths, abandon (backtrack) as soon as you know a path can't lead to a solution.

**Mental model:** Walking through a maze. At each fork, pick a direction. If it's a dead end, walk back and try the next direction.

```
                start
               /     \
           choice A   choice B
            /    \         \
         A1      A2        B1
        (dead)  (goal!)   (dead)
```

**The three-step pattern (every backtracking problem):**
```cpp
void backtrack(state) {
    if (goal_reached) { save_result; return; }

    for (each choice) {
        if (invalid) continue;           // PRUNE: skip bad choices early

        make_choice();                   // 1. DO
        backtrack(new_state);            // 2. RECURSE
        undo_choice();                   // 3. UNDO
    }
}
```

**Backtracking vs brute force:** Brute force tries EVERYTHING. Backtracking **prunes** — it skips entire branches that can't possibly work. This is what makes it fast enough.

---

## Concept 4: Dynamic Programming (DP)

**What it is:** Optimized recursion. When a recursive solution recomputes the same subproblems, DP stores the results to avoid redundant work.

**Two signals that a problem is DP:**
1. **Overlapping subproblems** — same state computed multiple times
2. **Optimal substructure** — optimal solution uses optimal solutions to subproblems

**Fibonacci — the simplest DP example:**
```cpp
// Pure recursion: O(2ⁿ) — recomputes same values
int fib(int n) {
    if (n <= 1) return n;
    return fib(n-1) + fib(n-2);
}

// Top-down DP (memoization): add a cache
int memo[101];
memset(memo, -1, sizeof(memo));
int fib(int n) {
    if (n <= 1) return n;
    if (memo[n] != -1) return memo[n];              // already computed
    return memo[n] = fib(n-1) + fib(n-2);           // store result
}

// Bottom-up DP (tabulation): fill table iteratively
int fib(int n) {
    if (n <= 1) return n;
    int dp[n + 1];
    dp[0] = 0; dp[1] = 1;
    for (int i = 2; i <= n; i++)
        dp[i] = dp[i-1] + dp[i-2];
    return dp[n];
}

// Space-optimized: only need last two values
int fib(int n) {
    if (n <= 1) return n;
    int a = 0, b = 1;
    for (int i = 2; i <= n; i++) {
        int c = a + b;
        a = b;
        b = c;
    }
    return b;
}
```

**How to approach any DP problem:**
1. Can I write a recursive solution? → Write it
2. Are there overlapping subproblems? → Add memoization (now it's DP)
3. Can I convert to bottom-up? → Fill table iteratively
4. Can I optimize space? → Usually only need previous row/values

---

## Concept 5: Greedy Algorithms

**What it is:** At each step, make the locally best choice. Unlike DP, no backtracking — commit to each decision.

**When it works:** When the locally optimal choice leads to the globally optimal solution. This isn't always true — you must prove (or intuit) the greedy choice is safe.

```cpp
// Example: Activity Selection (pick max non-overlapping intervals)
// Greedy choice: always pick the interval that ENDS earliest
int maxActivities(vector<vector<int>>& intervals) {
    sort(intervals.begin(), intervals.end(),
         [](auto& a, auto& b) { return a[1] < b[1]; });  // sort by end time

    int count = 1, lastEnd = intervals[0][1];
    for (int i = 1; i < intervals.size(); i++) {
        if (intervals[i][0] >= lastEnd) {       // no overlap
            count++;
            lastEnd = intervals[i][1];
        }
    }
    return count;
}

// Example: Jump Game (can you reach the end?)
bool canJump(vector<int>& nums) {
    int maxReach = 0;
    for (int i = 0; i <= maxReach && i < nums.size(); i++) {
        maxReach = max(maxReach, i + nums[i]);  // greedily extend reach
    }
    return maxReach >= nums.size() - 1;
}
```

**Greedy vs DP:**
- **Greedy:** Make one choice, never look back. Faster but only works when greedy choice is provably optimal.
- **DP:** Consider all choices, combine optimally. Slower but always correct when applicable.

**Common greedy signals:** Interval scheduling, Huffman coding, minimum spanning tree, tasks with deadlines, "is it possible to reach/achieve..."

### The "Track Best So Far" Greedy Pattern

One of the most common greedy patterns: walk through the array once, maintaining a running best (min, max, or sum), and compute the answer at each step.

```cpp
// Example 1: Best Time to Buy and Sell Stock (LC 121)
// Track the lowest price seen so far. At each price, check profit if selling today.
int maxProfit(vector<int>& prices) {
    int minPrice = INT_MAX, maxProfit = 0;
    for (int price : prices) {
        minPrice = min(minPrice, price);              // best buy so far
        maxProfit = max(maxProfit, price - minPrice);  // best profit if selling today
    }
    return maxProfit;
}
// [7, 1, 5, 3, 6, 4]
// min: 7→1→1→1→1→1   profit: 0→0→4→2→5→3   answer: 5

// Example 2: Maximum Subarray / Kadane's Algorithm (LC 53)
// Track the best subarray ending here. At each element, either extend or start fresh.
int maxSubArray(vector<int>& nums) {
    int currentSum = nums[0], maxSum = nums[0];
    for (int i = 1; i < nums.size(); i++) {
        currentSum = max(nums[i], currentSum + nums[i]);  // extend or restart
        maxSum = max(maxSum, currentSum);                  // update global best
    }
    return maxSum;
}
// [-2, 1, -3, 4, -1, 2, 1, -5, 4]
// curr: -2→1→-2→4→3→5→6→1→5   max: -2→1→1→4→4→5→6→6→6   answer: 6

// The common shape:
//   single loop through array
//   maintain running state (min, max, sum)
//   at each step, compute candidate answer and update best
```

**Problems using this pattern:**
- Best Time to Buy and Sell Stock (LC 121) — track min price
- Maximum Subarray (LC 53) — track current sum (Kadane's)
- Best Sightseeing Pair (LC 1014) — track best score + index
- Maximum Product Subarray (LC 152) — track min AND max (negatives flip sign)

---

## Concept 6: Bit Manipulation

**What it is:** Operating directly on binary representation of numbers. Extremely fast (single CPU instruction).

```cpp
// --- Basic operations ---
int a = 5;                               // binary: 101

a & b;    // AND:  bits set in BOTH
a | b;    // OR:   bits set in EITHER
a ^ b;    // XOR:  bits set in EXACTLY ONE
~a;       // NOT:  flip all bits
a << k;   // Left shift:  multiply by 2^k
a >> k;   // Right shift: divide by 2^k

// --- Common tricks ---
// Check if ith bit is set
bool isSet = (n >> i) & 1;

// Set the ith bit
n |= (1 << i);

// Clear the ith bit
n &= ~(1 << i);

// Toggle the ith bit
n ^= (1 << i);

// Check if power of 2
bool isPow2 = n > 0 && (n & (n - 1)) == 0;  // power of 2 has exactly one bit set

// Count set bits
__builtin_popcount(n);                   // GCC built-in, O(1)

// Lowest set bit
int lowest = n & (-n);                   // isolates rightmost 1-bit

// Remove lowest set bit
n = n & (n - 1);

// --- XOR properties (super useful) ---
// a ^ a = 0       (anything XOR itself = 0)
// a ^ 0 = a       (anything XOR 0 = itself)
// a ^ b ^ a = b   (cancels out)

// Find single number (all others appear twice)
int singleNumber(vector<int>& nums) {
    int result = 0;
    for (int n : nums) result ^= n;     // pairs cancel, single remains
    return result;
}

// --- Subsets via bitmask ---
// Enumerate all subsets of array of size n
vector<int> arr = {1, 2, 3};
int n = arr.size();
for (int mask = 0; mask < (1 << n); mask++) {  // 0 to 2^n - 1
    vector<int> subset;
    for (int i = 0; i < n; i++)
        if (mask & (1 << i))
            subset.push_back(arr[i]);
    // subset is one possible subset
}
// mask=0: {}  mask=1: {1}  mask=2: {2}  mask=3: {1,2}
// mask=4: {3} mask=5: {1,3} mask=6: {2,3} mask=7: {1,2,3}
```

---

## Concept 7: Two Pointers & Sliding Window (Conceptual)

**Two Pointers:** Use two indices to scan data, avoiding nested loops. Reduces O(n²) to O(n).

Three flavors:
1. **Opposite ends:** Start from both sides, move inward (sorted array sums, palindromes)
2. **Same direction:** Slow/fast pointer (remove duplicates, linked list cycle)
3. **Sliding window:** Left/right boundary of a range (subarray/substring problems)

**Why it works:** By maintaining a condition, we can skip positions that can't possibly be the answer. Each pointer moves at most n times → O(n) total.

---

## Concept 8: Binary Search (Conceptual)

**What it is:** Repeatedly halve the search space. Works whenever you can answer "is this value too small or too big?"

**The two forms:**
1. **Search in sorted array** — classic binary search for a target
2. **Search on answer** — binary search on the result value itself. "What's the minimum X such that condition is true?"

**Key insight for "search on answer":** If the answer space has a **monotonic property** (all values below threshold fail, all above succeed), you can binary search the threshold.

---

## Competitive Coding Boilerplate

```cpp
#include <bits/stdc++.h>
using namespace std;

// Type shortcuts
typedef long long ll;
typedef pair<int,int> pii;
typedef vector<int> vi;
typedef vector<vector<int>> vvi;

// Constants
const int MOD = 1e9 + 7;
const int INF = 1e9;
const ll LINF = 1e18;

// Direction arrays (4-dir and 8-dir)
int dx4[] = {0, 0, 1, -1};
int dy4[] = {1, -1, 0, 0};
int dx8[] = {0, 0, 1, -1, 1, 1, -1, -1};
int dy8[] = {1, -1, 0, 0, 1, -1, 1, -1};

// Fast I/O
ios_base::sync_with_stdio(false);
cin.tie(NULL);

// Common macros
#define all(x) (x).begin(), (x).end()
#define sz(x) (int)(x).size()
#define pb push_back
#define mp make_pair
#define fi first
#define se second
```

---

# Part 3: Algorithm Patterns

## Pattern 1: Sliding Window

### When to recognize
- Subarray / substring problems
- "Maximum/minimum of contiguous sequence"
- "Longest substring with at most K..."
- "Find anagram / permutation in string"
- Input is linear (array or string), answer is a contiguous range

### Core template (variable size window)
```cpp
int left = 0, best = 0;
// state tracking (hash map, count, sum, etc.)

for (int right = 0; right < n; right++) {
    // expand: add nums[right] or s[right] to state

    while (/* window is invalid */) {
        // shrink: remove nums[left] from state
        left++;
    }
    best = max(best, right - left + 1);
}
```

### Core template (fixed size window K)
```cpp
// build first window of size K
for (int i = 0; i < k; i++) { /* add to state */ }

for (int i = k; i < n; i++) {
    // add element i (enters window)
    // remove element i - k (leaves window)
    // update answer
}
```

### Variations and what changes

| Variation | Window type | State tracking | Shrink condition |
|---|---|---|---|
| Max sum subarray of size K | Fixed | Running sum | N/A (fixed) |
| Longest substring without repeating | Variable | Hash set of chars | Duplicate found |
| Longest with at most K distinct | Variable | Hash map char→count | Distinct count > K |
| Minimum window substring | Variable | Char frequency map | All target chars covered |
| Find all anagrams | Fixed | Char frequency match | N/A (fixed) |
| Max consecutive ones (K flips) | Variable | Count of zeros | Zero count > K |

### Key insight
The "shrink condition" is the only thing that really changes between problems. Master identifying what makes a window invalid.

### Solved Example: Longest Substring Without Repeating Characters (LC 3)
```
Input: s = "abcabcbb"
Output: 3  (the substring "abc")
```
```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        unordered_set<char> window;         // state: chars in current window
        int left = 0, best = 0;

        for (int right = 0; right < s.size(); right++) {
            // shrink condition: duplicate found
            while (window.count(s[right])) {
                window.erase(s[left]);
                left++;
            }
            window.insert(s[right]);        // expand
            best = max(best, right - left + 1);
        }
        return best;
    }
};
// Template mapping:
//   state       = unordered_set of chars in window
//   expand      = insert s[right]
//   shrink when = s[right] already in set (duplicate)
//   answer      = max window size seen
```

### Practice problems
1. Maximum Average Subarray I (LC 643) — fixed window warmup
2. Longest Substring Without Repeating Characters (LC 3)
3. Permutation in String (LC 567)
4. Longest Repeating Character Replacement (LC 424)
5. Minimum Window Substring (LC 76) — the boss fight

---

## Pattern 2: Two Pointers

### When to recognize
- Sorted array + find pair with some property
- "Two sum" variants on sorted data
- Palindrome checking
- Merging sorted arrays
- Partitioning (Dutch National Flag)
- "Container with most water", "trapping rain water"

### Core templates

**Opposite ends (converging):**
```cpp
int left = 0, right = n - 1;
while (left < right) {
    if (/* condition met */) return result;
    else if (/* need bigger */) left++;
    else right--;
}
```

**Same direction (fast/slow):**
```cpp
int slow = 0;
for (int fast = 0; fast < n; fast++) {
    if (/* fast points to wanted element */) {
        arr[slow] = arr[fast];
        slow++;
    }
}
// slow = new length
```

**Fast/slow for cycles:**
```cpp
auto slow = head, fast = head;
while (fast && fast->next) {
    slow = slow->next;
    fast = fast->next->next;
    if (slow == fast) return true; // cycle
}
```

### Variations

| Variation | Pointer type | Key decision |
|---|---|---|
| Two Sum (sorted) | Converging | Sum too small → left++, too big → right-- |
| 3Sum | Fix one + converging | Sort + fix i, two-pointer on rest, skip duplicates |
| Container With Most Water | Converging | Move the shorter side |
| Remove Duplicates in-place | Same direction | Slow writes, fast reads |
| Linked List Cycle | Fast/slow | Fast moves 2x, collision = cycle |
| Find middle of linked list | Fast/slow | When fast reaches end, slow is at middle |
| Palindrome check | Converging | Compare left and right, move inward |
| Trapping Rain Water | Converging | Process shorter side, track max heights |

### Solved Example: 3Sum (LC 15)
```
Input: nums = [-1, 0, 1, 2, -1, -4]
Output: [[-1, -1, 2], [-1, 0, 1]]
```
```cpp
class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        vector<vector<int>> result;
        sort(nums.begin(), nums.end());         // must sort first

        for (int i = 0; i < (int)nums.size() - 2; i++) {
            if (i > 0 && nums[i] == nums[i - 1]) continue; // skip dup for i

            int left = i + 1, right = nums.size() - 1;     // converging pointers
            while (left < right) {
                int sum = nums[i] + nums[left] + nums[right];
                if (sum < 0) left++;                        // need bigger
                else if (sum > 0) right--;                  // need smaller
                else {
                    result.push_back({nums[i], nums[left], nums[right]});
                    while (left < right && nums[left] == nums[left + 1]) left++;   // skip dup
                    while (left < right && nums[right] == nums[right - 1]) right--; // skip dup
                    left++; right--;
                }
            }
        }
        return result;
    }
};
// Template mapping:
//   Fix one element (i), then use converging two pointers on the rest
//   Dedup: skip same values at each level (i, left, right)
```

### Practice problems
1. Two Sum II - Sorted (LC 167)
2. Remove Duplicates from Sorted Array (LC 26)
3. 3Sum (LC 15)
4. Container With Most Water (LC 11)
5. Trapping Rain Water (LC 42)

---

## Pattern 3: Binary Search

### When to recognize
- Sorted array + find element
- "Minimum/maximum value that satisfies condition" — binary search on answer
- Anything where you can **halve the search space** based on a condition
- Rotated sorted array problems

### Core templates

**Standard binary search:**
```cpp
int lo = 0, hi = n - 1;
while (lo <= hi) {
    int mid = lo + (hi - lo) / 2;
    if (nums[mid] == target) return mid;
    else if (nums[mid] < target) lo = mid + 1;
    else hi = mid - 1;
}
return -1;
```

**Binary search on answer (find minimum valid):**
```cpp
int lo = MIN_POSSIBLE, hi = MAX_POSSIBLE;
while (lo < hi) {
    int mid = lo + (hi - lo) / 2;
    if (isValid(mid)) hi = mid;      // mid could be answer, search left
    else lo = mid + 1;               // mid too small
}
return lo; // first valid value
```

### Variations

| Variation | What changes | Key insight |
|---|---|---|
| Find exact element | Standard | Basic halving |
| Find first/last occurrence | Modified bounds | Don't return early, keep searching |
| Search rotated array | Check which half is sorted | Compare mid with lo/hi |
| Find peak element | Compare mid with neighbors | Move toward ascending side |
| Koko eating bananas | Binary search on answer | Check if speed K works in H hours |
| Split array largest sum | Binary search on answer | Check if max-sum ≤ mid with ≤ K splits |
| Find median of two sorted | Binary search on partition | Partition both arrays |

### Solved Example: Koko Eating Bananas (LC 875)
```
Input: piles = [3,6,7,11], h = 8
Output: 4  (eating speed of 4 bananas/hour finishes in 8 hours)
```
```cpp
class Solution {
public:
    int minEatingSpeed(vector<int>& piles, int h) {
        int lo = 1, hi = *max_element(piles.begin(), piles.end());

        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (canFinish(piles, mid, h)) hi = mid;    // mid works, try smaller
            else lo = mid + 1;                          // mid too slow
        }
        return lo;
    }

    bool canFinish(vector<int>& piles, int speed, int h) {
        int hours = 0;
        for (int p : piles)
            hours += (p + speed - 1) / speed;   // ceil division
        return hours <= h;
    }
};
// Template mapping:
//   Binary search on answer: search space is [1, max(piles)]
//   isValid(mid) = canFinish at speed mid within h hours
//   Find minimum valid → hi = mid pattern
```

### Binary search on answer — recognition trick
If the problem says "minimize the maximum" or "what's the minimum X such that..." you can often:
1. Binary search on the answer value
2. Write a `canAchieve(mid)` function
3. Find the boundary

### Practice problems
1. Binary Search (LC 704)
2. First Bad Version (LC 278)
3. Search in Rotated Sorted Array (LC 33)
4. Find Peak Element (LC 162)
5. Koko Eating Bananas (LC 875)
6. Median of Two Sorted Arrays (LC 4)

---

## Pattern 4: Backtracking

### When to recognize
- "Generate all..." / "Find all valid..."
- "List all combinations / permutations / subsets"
- Constraint satisfaction (Sudoku, N-Queens)
- Problems with choices at each step and constraints to respect

### Core template
```cpp
void backtrack(vector<...>& result, State& current, int start, ...) {
    if (/* goal reached */) {
        result.push_back(current);
        return;
    }
    for (int i = start; i < choices.size(); i++) {
        if (/* prune: skip invalid */) continue;

        current.add(choices[i]);          // make choice
        backtrack(result, current, ...);  // recurse
        current.remove(choices[i]);       // undo choice
    }
}
```

### Variations — the complete family

| Problem | Choices | Start index | Pruning | Dedup needed? |
|---|---|---|---|---|
| **Subsets** | Include/exclude each | i + 1 | None | No |
| **Subsets II** (with dups) | Include/exclude each | i + 1 | Skip if nums[i]==nums[i-1] and i>start | Yes |
| **Permutations** | Any unused element | 0 (use visited[]) | Skip visited | No |
| **Permutations II** (with dups) | Any unused element | 0 | Sort + skip same value at same level | Yes |
| **Combinations (n choose k)** | Pick next number | i + 1 | Stop when remaining < needed | No |
| **Combination Sum** (reuse OK) | Pick a number | i (same index) | Sum > target | No |
| **Combination Sum II** (no reuse) | Pick a number | i + 1 | Sum > target + sort/skip dups | Yes |
| **Palindrome Partition** | Choose split point | i + 1 | Substring not palindrome | No |
| **Generate Parentheses** | ( or ) | N/A | open ≤ n, close ≤ open | No |
| **N-Queens** | Column for each row | N/A | Column/diagonal conflicts | No |
| **Sudoku Solver** | 1-9 for empty cell | N/A | Row/col/box conflicts | No |
| **Word Search** | 4 directions | N/A | Out of bounds / visited / wrong char | No |

### Solved Example: Combination Sum (LC 39)
```
Input: candidates = [2,3,6,7], target = 7
Output: [[2,2,3],[7]]
```
```cpp
class Solution {
public:
    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        vector<vector<int>> result;
        vector<int> current;
        backtrack(result, current, candidates, target, 0);
        return result;
    }

    void backtrack(vector<vector<int>>& result, vector<int>& current,
                   vector<int>& candidates, int remaining, int start) {
        if (remaining == 0) {               // goal reached
            result.push_back(current);
            return;
        }
        for (int i = start; i < candidates.size(); i++) {
            if (candidates[i] > remaining) continue;   // prune: too big

            current.push_back(candidates[i]);           // make choice
            backtrack(result, current, candidates,
                      remaining - candidates[i], i);    // i (not i+1): reuse allowed
            current.pop_back();                         // undo choice
        }
    }
};
// Template mapping:
//   choices     = candidates[i..n]
//   start index = i (same index, since reuse is allowed)
//   prune       = candidates[i] > remaining
//   goal        = remaining == 0
//   No dedup needed: distinct candidates + controlled start index
```

### Solved Example: Subsets II with Dedup (LC 90)
```
Input: nums = [1,2,2]
Output: [[],[1],[1,2],[1,2,2],[2],[2,2]]
```
```cpp
class Solution {
public:
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        sort(nums.begin(), nums.end());     // MUST sort for dedup
        vector<vector<int>> result;
        vector<int> current;
        backtrack(result, current, nums, 0);
        return result;
    }

    void backtrack(vector<vector<int>>& result, vector<int>& current,
                   vector<int>& nums, int start) {
        result.push_back(current);          // every subset is valid
        for (int i = start; i < nums.size(); i++) {
            if (i > start && nums[i] == nums[i - 1]) continue; // skip dup at same level
            current.push_back(nums[i]);
            backtrack(result, current, nums, i + 1);
            current.pop_back();
        }
    }
};
// Dedup rule: if nums[i] == nums[i-1] AND i > start, skip.
// "i > start" means: only skip duplicates at the SAME recursion level,
// not across different depths.
```

### The deduplication pattern (memorize this)
When input has duplicates and you need unique results:
```cpp
sort(nums.begin(), nums.end()); // MUST sort first
// in the loop:
if (i > start && nums[i] == nums[i - 1]) continue; // skip dups at same level
```

### Practice problems
1. Subsets (LC 78)
2. Permutations (LC 46)
3. Combination Sum (LC 39)
4. Subsets II (LC 90)
5. Palindrome Partitioning (LC 131)
6. N-Queens (LC 51)

---

## Pattern 5: BFS (Breadth-First Search)

### When to recognize
- "Shortest path" in unweighted graph/grid
- "Minimum number of steps/moves"
- Level-order traversal of tree
- "Nearest" anything
- Spreading/infection problems (rotting oranges)

### Core template
```cpp
queue<State> q;
set<State> visited;
q.push(start);
visited.insert(start);
int steps = 0;

while (!q.empty()) {
    int size = q.size(); // process level by level
    for (int i = 0; i < size; i++) {
        auto curr = q.front(); q.pop();
        if (/* goal */) return steps;
        for (auto& next : neighbors(curr)) {
            if (visited.count(next)) continue;
            visited.insert(next);
            q.push(next);
        }
    }
    steps++;
}
return -1; // unreachable
```

### Variations

| Variation | State | Neighbors | Notes |
|---|---|---|---|
| Grid shortest path | (row, col) | 4 directions | Check bounds + walls |
| Word Ladder | current word | Change 1 char | Use word set for O(1) lookup |
| Tree level order | node pointer | left, right children | No visited needed |
| Rotting oranges | (row, col) | 4 directions | Multi-source: start with ALL rotten |
| Min Knight moves | (row, col) | 8 L-shapes | Prune with bounds |
| Open the lock | lock state "0000" | ±1 each digit | Deadends = pre-visited |
| Shortest path with state | (row, col, keys) | 4 dir + pick up key | State includes collected items |
| Bidirectional BFS | Two frontiers | Expand smaller side | Meet in middle for speed |

### Solved Example: Rotting Oranges (LC 994)
```
Input: grid = [[2,1,1],[1,1,0],[0,1,1]]
Output: 4  (minutes until all oranges rot)
```
```cpp
class Solution {
public:
    int orangesRotting(vector<vector<int>>& grid) {
        int m = grid.size(), n = grid[0].size();
        queue<pair<int,int>> q;
        int fresh = 0;

        // multi-source: push ALL rotten oranges as starting points
        for (int r = 0; r < m; r++)
            for (int c = 0; c < n; c++) {
                if (grid[r][c] == 2) q.push({r, c});
                else if (grid[r][c] == 1) fresh++;
            }

        int minutes = 0;
        int dirs[] = {0, 1, 0, -1, 0};     // 4-direction trick

        while (!q.empty() && fresh > 0) {
            int size = q.size();            // process level by level
            for (int i = 0; i < size; i++) {
                auto [r, c] = q.front(); q.pop();
                for (int d = 0; d < 4; d++) {
                    int nr = r + dirs[d], nc = c + dirs[d + 1];
                    if (nr < 0 || nr >= m || nc < 0 || nc >= n) continue;
                    if (grid[nr][nc] != 1) continue;
                    grid[nr][nc] = 2;       // mark rotten (also acts as visited)
                    fresh--;
                    q.push({nr, nc});
                }
            }
            minutes++;
        }
        return fresh == 0 ? minutes : -1;
    }
};
// Template mapping:
//   State     = (row, col)
//   Neighbors = 4 directions
//   Visited   = grid itself (change 1→2)
//   Multi-source BFS: enqueue all rotten at start
//   Level-by-level: each level = 1 minute
```

### Multi-source BFS trick
When there are multiple starting points (e.g., all rotten oranges), push ALL of them into the queue before starting. This finds shortest distance from any source.

### Practice problems
1. Binary Tree Level Order Traversal (LC 102)
2. Number of Islands (LC 200) — can also use DFS
3. Rotting Oranges (LC 994)
4. Word Ladder (LC 127)
5. Open the Lock (LC 752)
6. Shortest Path in Binary Matrix (LC 1091)

---

## Pattern 6: DFS (Depth-First Search)

### When to recognize
- "Does a path exist?"
- Tree traversals (inorder, preorder, postorder)
- Connected components / counting islands
- Detecting cycles
- Problems where you need to explore exhaustively

### Core template (graph/grid)
```cpp
void dfs(vector<vector<int>>& grid, int r, int c, vector<vector<bool>>& visited) {
    if (r < 0 || r >= grid.size() || c < 0 || c >= grid[0].size()) return;
    if (visited[r][c] || grid[r][c] == 0) return;
    visited[r][c] = true;
    dfs(grid, r + 1, c, visited);
    dfs(grid, r - 1, c, visited);
    dfs(grid, r, c + 1, visited);
    dfs(grid, r, c - 1, visited);
}
```

### Core template (tree)
```cpp
// Most tree problems are just: process node + recurse on children
int dfs(TreeNode* node) {
    if (!node) return BASE_CASE;
    int left = dfs(node->left);
    int right = dfs(node->right);
    // combine left, right, and node->val
    return RESULT;
}
```

### Variations

| Variation | Key twist |
|---|---|
| Number of islands | DFS from each unvisited land, count components |
| Max area of island | DFS returns size, track max |
| Clone graph | DFS + hash map (old node → new node) |
| Course schedule (cycle) | DFS with 3 states: unvisited/in-progress/done |
| Tree max depth | return 1 + max(left, right) |
| Tree diameter | Track max(left + right) across all nodes |
| Path sum | Subtract from target as you go down |
| Lowest common ancestor | Return node if found, bubble up |
| Serialize/deserialize tree | Preorder DFS with null markers |

### Solved Example: Number of Islands (LC 200)
```
Input: grid = [["1","1","0","0","0"],
               ["1","1","0","0","0"],
               ["0","0","1","0","0"],
               ["0","0","0","1","1"]]
Output: 3
```
```cpp
class Solution {
public:
    int numIslands(vector<vector<char>>& grid) {
        int count = 0;
        for (int r = 0; r < grid.size(); r++)
            for (int c = 0; c < grid[0].size(); c++)
                if (grid[r][c] == '1') {
                    dfs(grid, r, c);    // sink the entire island
                    count++;            // one more island found
                }
        return count;
    }

    void dfs(vector<vector<char>>& grid, int r, int c) {
        if (r < 0 || r >= grid.size() || c < 0 || c >= grid[0].size()) return;
        if (grid[r][c] != '1') return;
        grid[r][c] = '0';              // mark visited by sinking
        dfs(grid, r + 1, c);
        dfs(grid, r - 1, c);
        dfs(grid, r, c + 1);
        dfs(grid, r, c - 1);
    }
};
// Template mapping:
//   Each DFS call explores one connected component
//   Visited = modify grid in-place ('1' → '0')
//   Count components = count how many times DFS is triggered
```

### Solved Example: Lowest Common Ancestor (LC 236)
```
Input: root = [3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1
Output: 3
```
```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (!root || root == p || root == q) return root;   // base: found or null
        TreeNode* left = lowestCommonAncestor(root->left, p, q);
        TreeNode* right = lowestCommonAncestor(root->right, p, q);
        if (left && right) return root;     // p and q on different sides → root is LCA
        return left ? left : right;         // both on same side → bubble up
    }
};
// Key insight: if p is in left subtree and q is in right subtree,
// current node is the LCA. Otherwise, the answer bubbles up from whichever side has both.
```

### DFS on trees — the "global variable" trick
Many tree problems need you to track a global best while recursing:
```cpp
int globalBest = 0;
int dfs(TreeNode* node) {
    if (!node) return 0;
    int left = dfs(node->left);
    int right = dfs(node->right);
    globalBest = max(globalBest, left + right + node->val); // update global
    return max(left, right) + node->val; // return path to parent
}
```
Used in: diameter, max path sum, longest univalue path.

### Practice problems
1. Max Depth of Binary Tree (LC 104)
2. Number of Islands (LC 200)
3. Path Sum I/II/III (LC 112, 113, 437)
4. Lowest Common Ancestor (LC 236)
5. Course Schedule (LC 207)
6. Binary Tree Maximum Path Sum (LC 124)

---

## Pattern 7: Dynamic Programming

### When to recognize
- "How many ways to..."
- "Minimum/maximum cost to..."
- "Is it possible to..." (with many overlapping paths)
- Optimal substructure: answer depends on answers to smaller subproblems
- Overlapping subproblems: same state gets recomputed

### Step-by-step approach
1. **Define state**: What variables describe a subproblem? (Usually index, remaining capacity, etc.)
2. **Define recurrence**: How does dp[i] relate to smaller subproblems?
3. **Define base case**: What's the answer for the smallest subproblem?
4. **Define order**: Fill table so dependencies are computed first
5. **Optimize space** if possible (often only need previous row)

### The major DP families

#### Family 1: Linear DP
State: `dp[i]` = answer considering elements 0..i
```
dp[i] = some function of dp[i-1], dp[i-2], ...
```

| Problem | Recurrence | Base |
|---|---|---|
| Climbing stairs | dp[i] = dp[i-1] + dp[i-2] | dp[0]=1, dp[1]=1 |
| House robber | dp[i] = max(dp[i-1], dp[i-2] + nums[i]) | dp[0]=nums[0] |
| Max subarray (Kadane) | dp[i] = max(nums[i], dp[i-1] + nums[i]) | dp[0]=nums[0] |
| Longest increasing subseq | dp[i] = max(dp[j]+1) for j<i where nums[j]<nums[i] | dp[i]=1 |
| Word break | dp[i] = any dp[j] && s[j..i] in dict | dp[0]=true |
| Decode ways | dp[i] = dp[i-1] (if valid 1-digit) + dp[i-2] (if valid 2-digit) | dp[0]=1 |

**Solved Example: House Robber (LC 198)**
```
Input: nums = [2,7,9,3,1]
Output: 12  (rob houses 0, 2, 4 → 2+9+1=12)
```
```cpp
class Solution {
public:
    int rob(vector<int>& nums) {
        int n = nums.size();
        if (n == 1) return nums[0];
        vector<int> dp(n);
        dp[0] = nums[0];
        dp[1] = max(nums[0], nums[1]);
        for (int i = 2; i < n; i++)
            dp[i] = max(dp[i - 1],             // skip house i
                        dp[i - 2] + nums[i]);   // rob house i
        return dp[n - 1];
    }
};
// dp[i] = max money robbing from houses 0..i
// Choice: rob house i (get nums[i] + best from 0..i-2)
//         or skip it (best from 0..i-1)
// Space optimization: only need dp[i-1] and dp[i-2], so use two variables
```

#### Family 2: Two-sequence / Grid DP
State: `dp[i][j]` = answer for first i of seq1 and first j of seq2

| Problem | Recurrence | Base |
|---|---|---|
| Longest common subseq | match: dp[i-1][j-1]+1, else max(dp[i-1][j], dp[i][j-1]) | dp[0][j]=dp[i][0]=0 |
| Edit distance | match: dp[i-1][j-1], else 1+min(dp[i-1][j], dp[i][j-1], dp[i-1][j-1]) | dp[i][0]=i, dp[0][j]=j |
| Grid unique paths | dp[i][j] = dp[i-1][j] + dp[i][j-1] | dp[0][j]=dp[i][0]=1 |
| Min path sum | dp[i][j] = grid[i][j] + min(dp[i-1][j], dp[i][j-1]) | dp[0][0]=grid[0][0] |

**Solved Example: Edit Distance (LC 72)**
```
Input: word1 = "horse", word2 = "ros"
Output: 3  (horse → rorse → rose → ros)
```
```cpp
class Solution {
public:
    int minDistance(string word1, string word2) {
        int m = word1.size(), n = word2.size();
        vector<vector<int>> dp(m + 1, vector<int>(n + 1));

        for (int i = 0; i <= m; i++) dp[i][0] = i;     // delete all
        for (int j = 0; j <= n; j++) dp[0][j] = j;     // insert all

        for (int i = 1; i <= m; i++) {
            for (int j = 1; j <= n; j++) {
                if (word1[i - 1] == word2[j - 1])
                    dp[i][j] = dp[i - 1][j - 1];       // chars match, no op
                else
                    dp[i][j] = 1 + min({dp[i - 1][j],      // delete
                                        dp[i][j - 1],       // insert
                                        dp[i - 1][j - 1]}); // replace
            }
        }
        return dp[m][n];
    }
};
// dp[i][j] = min operations to convert word1[0..i-1] to word2[0..j-1]
// Three choices when chars don't match: delete, insert, replace
// Base cases: converting empty string to/from a string of length k costs k ops
```

#### Family 3: Knapsack DP
State: `dp[i][w]` = best value using items 0..i with capacity w

| Variant | Choice | Recurrence |
|---|---|---|
| 0/1 Knapsack | Take or skip item | dp[i][w] = max(dp[i-1][w], dp[i-1][w-wt[i]] + val[i]) |
| Unbounded knapsack | Take any number | dp[w] = max(dp[w], dp[w-wt[i]] + val[i]) for all i |
| Subset sum | Take or skip | dp[i][s] = dp[i-1][s] \|\| dp[i-1][s-nums[i]] |
| Coin change (min) | Use any coin | dp[a] = min(dp[a], dp[a-coin]+1) for all coins |
| Coin change (count ways) | Use any coin | dp[a] += dp[a-coin] for all coins |
| Partition equal subset | Take or skip | Reduce to subset sum = total/2 |

**Solved Example: Coin Change (LC 322)**
```
Input: coins = [1,2,5], amount = 11
Output: 3  (5+5+1)
```
```cpp
class Solution {
public:
    int coinChange(vector<int>& coins, int amount) {
        vector<int> dp(amount + 1, amount + 1);     // amount+1 = "impossible"
        dp[0] = 0;                                   // 0 coins for amount 0

        for (int a = 1; a <= amount; a++)
            for (int coin : coins)
                if (coin <= a)
                    dp[a] = min(dp[a], dp[a - coin] + 1);

        return dp[amount] > amount ? -1 : dp[amount];
    }
};
// dp[a] = min coins to make amount a
// For each amount, try every coin: dp[a] = min(dp[a - coin] + 1)
// This is unbounded knapsack: each coin can be used unlimited times
```

#### Family 4: Interval DP
State: `dp[i][j]` = answer for subarray/substring i..j
```
dp[i][j] = best over all split points k in [i, j)
```

| Problem | Key idea |
|---|---|
| Longest palindromic subseq | dp[i][j] = dp[i+1][j-1]+2 if match, else max of shrinking either end |
| Longest palindromic substring | Expand from center, or dp[i][j] = true if s[i]==s[j] and dp[i+1][j-1] |
| Burst balloons | dp[i][j] = max over k of dp[i][k] + dp[k][j] + nums[i]*nums[k]*nums[j] |
| Matrix chain multiplication | Try all split points, minimize cost |

**Solved Example: Longest Palindromic Substring (LC 5)**
```
Input: s = "babad"
Output: "bab" (or "aba")
```
```cpp
class Solution {
public:
    string longestPalindrome(string s) {
        int n = s.size(), start = 0, maxLen = 1;
        // dp[i][j] = true if s[i..j] is a palindrome
        vector<vector<bool>> dp(n, vector<bool>(n, false));

        // base: single chars
        for (int i = 0; i < n; i++) dp[i][i] = true;

        // base: two chars
        for (int i = 0; i < n - 1; i++)
            if (s[i] == s[i + 1]) {
                dp[i][i + 1] = true;
                start = i; maxLen = 2;
            }

        // fill for lengths 3..n (must go by increasing length)
        for (int len = 3; len <= n; len++)
            for (int i = 0; i <= n - len; i++) {
                int j = i + len - 1;
                if (s[i] == s[j] && dp[i + 1][j - 1]) {
                    dp[i][j] = true;
                    start = i; maxLen = len;
                }
            }
        return s.substr(start, maxLen);
    }
};
// Interval DP: dp[i][j] depends on dp[i+1][j-1] (inner substring)
// Fill order: increasing length, so inner results are ready
// Alternative O(n) approach: Manacher's algorithm (rarely asked)
```

#### Family 5: DP on Trees
Process children first (postorder), combine results at each node.

| Problem | State at node |
|---|---|
| House robber III | (rob_this_node, skip_this_node) |
| Binary tree cameras | (covered, has_camera, needs_cover) |
| Diameter of tree | left_depth + right_depth, track global max |

**Solved Example: House Robber III (LC 337)**
```
Input:     3         Output: 7 (rob nodes 3+4=7, or 4+5 is not adjacent?
          / \                  actually: rob root(3) + right-right(4) = 3+4=7
         2   3                 skipping the direct children)
          \   \
           3   1
```
```cpp
class Solution {
public:
    int rob(TreeNode* root) {
        auto [skip, take] = dfs(root);
        return max(skip, take);
    }

    pair<int,int> dfs(TreeNode* node) {  // returns {skip_this, rob_this}
        if (!node) return {0, 0};
        auto [leftSkip, leftTake] = dfs(node->left);
        auto [rightSkip, rightTake] = dfs(node->right);

        int skip = max(leftSkip, leftTake) + max(rightSkip, rightTake);
        int take = node->val + leftSkip + rightSkip;  // can't rob children
        return {skip, take};
    }
};
// Each node returns two values: best if we skip it, best if we rob it
// Rob this node → must skip both children
// Skip this node → children can be robbed or skipped (take best of each)
```

#### Family 6: Bitmask DP
State: `dp[mask]` where mask represents a subset of items.
Used when n ≤ 20.

| Problem | State |
|---|---|
| Traveling salesman | dp[mask][i] = min cost visiting set `mask` ending at city i |
| Assign tasks to people | dp[mask] = min cost assigning first k tasks to people in mask |
| Shortest superstring | dp[mask][i] = shortest string containing all strings in mask, ending with i |

### Top-down vs bottom-up decision
- **Top-down (memoization)**: Easier to write, only computes needed states. Use when state space is sparse.
- **Bottom-up (tabulation)**: Slightly faster (no recursion overhead), easier to optimize space. Use when you need all states anyway.

### Practice problems
1. Climbing Stairs (LC 70) — warmup
2. House Robber (LC 198)
3. Coin Change (LC 322)
4. Longest Common Subsequence (LC 1143)
5. Longest Increasing Subsequence (LC 300)
6. Edit Distance (LC 72)
7. Partition Equal Subset Sum (LC 416)
8. Burst Balloons (LC 312)

---

## Pattern 8: Graph Algorithms

### Topological Sort
**When:** Dependencies, ordering, "course schedule" type problems.
```cpp
// Kahn's Algorithm (BFS-based)
vector<int> topoSort(int n, vector<vector<int>>& adj) {
    vector<int> indegree(n, 0), order;
    for (int u = 0; u < n; u++)
        for (int v : adj[u]) indegree[v]++;

    queue<int> q;
    for (int i = 0; i < n; i++)
        if (indegree[i] == 0) q.push(i);

    while (!q.empty()) {
        int u = q.front(); q.pop();
        order.push_back(u);
        for (int v : adj[u])
            if (--indegree[v] == 0) q.push(v);
    }
    return order.size() == n ? order : vector<int>{}; // empty = cycle
}
```

### Dijkstra's (weighted shortest path)
**When:** Shortest path with non-negative weights.
```cpp
vector<int> dijkstra(int n, vector<vector<pair<int,int>>>& adj, int src) {
    vector<int> dist(n, INT_MAX);
    priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
    dist[src] = 0;
    pq.push({0, src});

    while (!pq.empty()) {
        auto [d, u] = pq.top(); pq.pop();
        if (d > dist[u]) continue;
        for (auto [v, w] : adj[u]) {
            if (dist[u] + w < dist[v]) {
                dist[v] = dist[u] + w;
                pq.push({dist[v], v});
            }
        }
    }
    return dist;
}
```

### Union-Find
**When:** "Are these connected?", dynamic connectivity, counting components.
```cpp
class UnionFind {
    vector<int> parent, rank_;
public:
    UnionFind(int n) : parent(n), rank_(n, 0) {
        iota(parent.begin(), parent.end(), 0);
    }
    int find(int x) {
        if (parent[x] != x) parent[x] = find(parent[x]); // path compression
        return parent[x];
    }
    bool unite(int x, int y) {
        int px = find(x), py = find(y);
        if (px == py) return false;
        if (rank_[px] < rank_[py]) swap(px, py);
        parent[py] = px;
        if (rank_[px] == rank_[py]) rank_[px]++;
        return true;
    }
};
```

### Solved Example: Course Schedule (LC 207)
```
Input: numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
Output: true  (can take 0→1→2→3)
```
```cpp
class Solution {
public:
    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        vector<vector<int>> adj(numCourses);
        vector<int> indegree(numCourses, 0);

        for (auto& p : prerequisites) {
            adj[p[1]].push_back(p[0]);      // p[1] → p[0] (prereq → course)
            indegree[p[0]]++;
        }

        queue<int> q;
        for (int i = 0; i < numCourses; i++)
            if (indegree[i] == 0) q.push(i);   // no prerequisites

        int completed = 0;
        while (!q.empty()) {
            int course = q.front(); q.pop();
            completed++;
            for (int next : adj[course])
                if (--indegree[next] == 0) q.push(next);
        }
        return completed == numCourses;     // false if cycle exists
    }
};
// Topological sort via Kahn's algorithm
// If we can process all nodes, no cycle → can finish all courses
// If completed < numCourses, there's a cycle → impossible
```

### Solved Example: Network Delay Time — Dijkstra (LC 743)
```
Input: times = [[2,1,1],[2,3,1],[3,4,1]], n = 4, k = 2
Output: 2  (signal from node 2 reaches all nodes in 2 units)
```
```cpp
class Solution {
public:
    int networkDelayTime(vector<vector<int>>& times, int n, int k) {
        vector<vector<pair<int,int>>> adj(n + 1);
        for (auto& t : times)
            adj[t[0]].push_back({t[1], t[2]});     // {dest, weight}

        vector<int> dist(n + 1, INT_MAX);
        priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
        dist[k] = 0;
        pq.push({0, k});

        while (!pq.empty()) {
            auto [d, u] = pq.top(); pq.pop();
            if (d > dist[u]) continue;              // stale entry
            for (auto [v, w] : adj[u]) {
                if (dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                    pq.push({dist[v], v});
                }
            }
        }

        int ans = *max_element(dist.begin() + 1, dist.end());
        return ans == INT_MAX ? -1 : ans;
    }
};
// Dijkstra from source k, then answer = max distance to any node
// If any node unreachable (INT_MAX), return -1
```

### Practice problems
1. Course Schedule I & II (LC 207, 210)
2. Network Delay Time (LC 743)
3. Number of Connected Components (LC 323)
4. Redundant Connection (LC 684)
5. Cheapest Flights Within K Stops (LC 787)
6. Accounts Merge (LC 721)

---

## Pattern 9: Stack-Based Patterns

### Monotonic Stack
**When:** "Next greater/smaller element", "largest rectangle", stock span.

```cpp
// Next greater element for each position
vector<int> nextGreater(vector<int>& nums) {
    int n = nums.size();
    vector<int> result(n, -1);
    stack<int> st; // stores indices

    for (int i = 0; i < n; i++) {
        while (!st.empty() && nums[st.top()] < nums[i]) {
            result[st.top()] = nums[i];
            st.pop();
        }
        st.push(i);
    }
    return result;
}
```

| Problem | Stack stores | Pop condition |
|---|---|---|
| Next greater element | Indices of unresolved | Current > top |
| Daily temperatures | Indices of unresolved | Current temp > top |
| Largest rectangle in histogram | Indices of bars | Current height < top |
| Trapping rain water (stack) | Indices of bars | Current > top (fill layer by layer) |
| Valid parentheses | Opening brackets | Matching close bracket found |
| Min stack | (val, current_min) pairs | Normal stack ops |

### Solved Example: Daily Temperatures (LC 739)
```
Input: temperatures = [73,74,75,71,69,72,76,73]
Output:               [ 1, 1, 4, 2, 1, 1, 0, 0]
```
```cpp
class Solution {
public:
    vector<int> dailyTemperatures(vector<int>& temps) {
        int n = temps.size();
        vector<int> result(n, 0);
        stack<int> st;                      // indices of unresolved days

        for (int i = 0; i < n; i++) {
            // pop all days that current temp resolves
            while (!st.empty() && temps[i] > temps[st.top()]) {
                result[st.top()] = i - st.top();    // days until warmer
                st.pop();
            }
            st.push(i);
        }
        return result;
    }
};
// Monotonic decreasing stack: stack always has decreasing temps
// When we find a warmer day, it resolves all colder days on the stack
// Days left on stack at end → no warmer day exists (already 0)
```

### Solved Example: Largest Rectangle in Histogram (LC 84)
```
Input: heights = [2,1,5,6,2,3]
Output: 10  (rectangle of height 5, width 2 at indices 2-3)
```
```cpp
class Solution {
public:
    int largestRectangleArea(vector<int>& heights) {
        stack<int> st;
        int maxArea = 0, n = heights.size();

        for (int i = 0; i <= n; i++) {
            int h = (i == n) ? 0 : heights[i];     // sentinel: 0 flushes stack
            while (!st.empty() && h < heights[st.top()]) {
                int height = heights[st.top()]; st.pop();
                int width = st.empty() ? i : i - st.top() - 1;
                maxArea = max(maxArea, height * width);
            }
            st.push(i);
        }
        return maxArea;
    }
};
// Stack stores indices of increasing heights
// When height decreases: the popped bar's rectangle extends from
//   (previous stack top + 1) to (current index - 1)
// Sentinel value 0 at end forces all remaining bars to be processed
```

### Practice problems
1. Valid Parentheses (LC 20)
2. Daily Temperatures (LC 739)
3. Next Greater Element I (LC 496)
4. Largest Rectangle in Histogram (LC 84)
5. Maximal Rectangle (LC 85)

---

## Pattern 10: Heap / Priority Queue

### When to recognize
- "Kth largest/smallest"
- "Top K frequent"
- Merge K sorted things
- Running median
- "Schedule to minimize..."

### Key insight
- Want K largest? Use a **min-heap of size K** (smallest of the large ones falls out)
- Want K smallest? Use a **max-heap of size K**

### Templates

**Kth largest:**
```cpp
priority_queue<int, vector<int>, greater<int>> minHeap; // size K
for (int num : nums) {
    minHeap.push(num);
    if (minHeap.size() > k) minHeap.pop();
}
return minHeap.top(); // kth largest
```

**Merge K sorted lists:**
```cpp
auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);
// push head of each list, then always pop min and push its next
```

**Running median (two heaps):**
```cpp
priority_queue<int> lo;                              // max-heap: smaller half
priority_queue<int, vector<int>, greater<int>> hi;   // min-heap: larger half

void addNum(int num) {
    lo.push(num);
    hi.push(lo.top()); lo.pop();
    if (hi.size() > lo.size()) { lo.push(hi.top()); hi.pop(); }
}
double findMedian() {
    return lo.size() > hi.size() ? lo.top() : (lo.top() + hi.top()) / 2.0;
}
```

### Solved Example: Top K Frequent Elements (LC 347)
```
Input: nums = [1,1,1,2,2,3], k = 2
Output: [1,2]
```
```cpp
class Solution {
public:
    vector<int> topKFrequent(vector<int>& nums, int k) {
        unordered_map<int, int> freq;
        for (int n : nums) freq[n]++;

        // min-heap of size k: (frequency, value)
        priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
        for (auto& [val, cnt] : freq) {
            pq.push({cnt, val});
            if (pq.size() > k) pq.pop();   // evict least frequent
        }

        vector<int> result;
        while (!pq.empty()) {
            result.push_back(pq.top().second);
            pq.pop();
        }
        return result;
    }
};
// Step 1: Count frequencies with hash map
// Step 2: Min-heap of size k keeps the k most frequent
//   - When heap exceeds k, the smallest frequency gets evicted
//   - What remains = top k frequent elements
// Time: O(n log k) — better than O(n log n) sorting when k << n
```

### Solved Example: Merge K Sorted Lists (LC 23)
```
Input: lists = [[1,4,5],[1,3,4],[2,6]]
Output: [1,1,2,3,4,4,5,6]
```
```cpp
class Solution {
public:
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        auto cmp = [](ListNode* a, ListNode* b) { return a->val > b->val; };
        priority_queue<ListNode*, vector<ListNode*>, decltype(cmp)> pq(cmp);

        for (auto* list : lists)
            if (list) pq.push(list);        // push head of each list

        ListNode dummy(0);
        ListNode* tail = &dummy;

        while (!pq.empty()) {
            ListNode* smallest = pq.top(); pq.pop();
            tail->next = smallest;
            tail = tail->next;
            if (smallest->next) pq.push(smallest->next);   // push next node
        }
        return dummy.next;
    }
};
// Min-heap always holds one node from each list (at most k nodes)
// Pop smallest, append to result, push its next node
// Time: O(n log k) where n = total nodes, k = number of lists
```

### Practice problems
1. Kth Largest Element (LC 215)
2. Top K Frequent Elements (LC 347)
3. Merge K Sorted Lists (LC 23)
4. Find Median from Data Stream (LC 295)
5. Task Scheduler (LC 621)

---

## Pattern 11: Trie (Prefix Tree)

### When to recognize
- Prefix matching / autocomplete
- "Word search" with dictionary
- Longest common prefix
- Counting words with prefix

### Core template
```cpp
struct TrieNode {
    TrieNode* children[26] = {};
    bool isEnd = false;
};

class Trie {
    TrieNode* root = new TrieNode();
public:
    void insert(string& word) {
        auto node = root;
        for (char c : word) {
            if (!node->children[c - 'a'])
                node->children[c - 'a'] = new TrieNode();
            node = node->children[c - 'a'];
        }
        node->isEnd = true;
    }
    bool search(string& word) {
        auto node = find(word);
        return node && node->isEnd;
    }
    bool startsWith(string& prefix) {
        return find(prefix) != nullptr;
    }
private:
    TrieNode* find(string& s) {
        auto node = root;
        for (char c : s) {
            if (!node->children[c - 'a']) return nullptr;
            node = node->children[c - 'a'];
        }
        return node;
    }
};
```

### Solved Example: Word Search II (LC 212) — Trie + Backtracking
```
Input: board = [["o","a","a","n"],["e","t","a","e"],["i","h","k","r"],["i","f","l","v"]],
       words = ["oath","pea","eat","rain"]
Output: ["eat","oath"]
```
```cpp
class Solution {
    struct TrieNode {
        TrieNode* children[26] = {};
        string* word = nullptr;             // store word at leaf (avoids rebuilding)
    };

public:
    vector<string> findWords(vector<vector<char>>& board, vector<string>& words) {
        // Build trie from all words
        TrieNode root;
        for (auto& w : words) {
            auto* node = &root;
            for (char c : w) {
                if (!node->children[c - 'a'])
                    node->children[c - 'a'] = new TrieNode();
                node = node->children[c - 'a'];
            }
            node->word = &w;
        }

        vector<string> result;
        int m = board.size(), n = board[0].size();
        for (int r = 0; r < m; r++)
            for (int c = 0; c < n; c++)
                dfs(board, r, c, &root, result);
        return result;
    }

    void dfs(vector<vector<char>>& board, int r, int c,
             TrieNode* node, vector<string>& result) {
        if (r < 0 || r >= board.size() || c < 0 || c >= board[0].size()) return;
        char ch = board[r][c];
        if (ch == '#' || !node->children[ch - 'a']) return;     // visited or no path

        node = node->children[ch - 'a'];
        if (node->word) {
            result.push_back(*node->word);
            node->word = nullptr;           // dedup: don't find same word twice
        }

        board[r][c] = '#';                  // mark visited
        dfs(board, r + 1, c, node, result);
        dfs(board, r - 1, c, node, result);
        dfs(board, r, c + 1, node, result);
        dfs(board, r, c - 1, node, result);
        board[r][c] = ch;                   // restore
    }
};
// Combines two patterns:
//   Trie: efficient prefix lookup (prune paths that can't form any word)
//   DFS/Backtracking: explore grid in all directions
// Without trie, you'd search for each word separately → much slower
```

### Practice problems
1. Implement Trie (LC 208)
2. Word Search II (LC 212) — trie + backtracking
3. Design Add and Search Words (LC 211)

---

## Pattern 12: Intervals

### When to recognize
- Merging overlapping intervals
- Inserting into intervals
- Meeting rooms / scheduling
- Minimum platforms / rooms needed

### Core approach: sort by start time, then sweep

**Merge intervals:**
```cpp
sort(intervals.begin(), intervals.end());
vector<vector<int>> merged = {intervals[0]};
for (int i = 1; i < intervals.size(); i++) {
    if (intervals[i][0] <= merged.back()[1])
        merged.back()[1] = max(merged.back()[1], intervals[i][1]);
    else
        merged.push_back(intervals[i]);
}
```

| Problem | Sort by | Technique |
|---|---|---|
| Merge intervals | Start | Extend end if overlap |
| Insert interval | Already sorted | Find overlap region |
| Meeting rooms (can attend all?) | Start | Check any overlap |
| Meeting rooms II (min rooms) | Events | Sort start/end, count active |
| Non-overlapping intervals (min removals) | End | Greedy: keep earliest ending |

### Solved Example: Meeting Rooms II — Min Rooms Needed (LC 253)
```
Input: intervals = [[0,30],[5,10],[15,20]]
Output: 2  (meetings [0,30] and [5,10] overlap → need 2 rooms)
```
```cpp
class Solution {
public:
    int minMeetingRooms(vector<vector<int>>& intervals) {
        vector<int> starts, ends;
        for (auto& i : intervals) {
            starts.push_back(i[0]);
            ends.push_back(i[1]);
        }
        sort(starts.begin(), starts.end());
        sort(ends.begin(), ends.end());

        int rooms = 0, maxRooms = 0, e = 0;
        for (int s = 0; s < starts.size(); s++) {
            if (starts[s] < ends[e]) rooms++;       // new meeting starts before one ends
            else e++;                                // reuse a room (one meeting ended)
            maxRooms = max(maxRooms, rooms);
        }
        return maxRooms;
    }
};
// Sweep line approach: separate starts and ends, sort independently
// Walk through starts: if a start comes before the next end, need +1 room
// Alternative: use a min-heap of end times (pop if current start >= heap top)
```

### Practice problems
1. Merge Intervals (LC 56)
2. Insert Interval (LC 57)
3. Non-overlapping Intervals (LC 435)
4. Meeting Rooms II (LC 253)
5. Minimum Number of Arrows (LC 452)

---

## Pattern 13: Prefix Sum

### When to recognize
- "Subarray sum equals K"
- Range sum queries
- "Number of subarrays with sum..."

### Core idea
```
prefix[i] = nums[0] + nums[1] + ... + nums[i-1]
sum(l, r) = prefix[r+1] - prefix[l]
```

### With hash map — "subarray sum = K" pattern
```cpp
int subarraySum(vector<int>& nums, int k) {
    unordered_map<int, int> prefixCount;
    prefixCount[0] = 1; // empty prefix
    int sum = 0, count = 0;
    for (int num : nums) {
        sum += num;
        count += prefixCount[sum - k]; // how many prefixes give us target
        prefixCount[sum]++;
    }
    return count;
}
```

### Solved Example: Product of Array Except Self (LC 238)
```
Input: nums = [1,2,3,4]
Output: [24,12,8,6]  (each position = product of all OTHER elements)
```
```cpp
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        int n = nums.size();
        vector<int> result(n, 1);

        // Pass 1: prefix products (left to right)
        int prefix = 1;
        for (int i = 0; i < n; i++) {
            result[i] = prefix;
            prefix *= nums[i];
        }

        // Pass 2: suffix products (right to left)
        int suffix = 1;
        for (int i = n - 1; i >= 0; i--) {
            result[i] *= suffix;
            suffix *= nums[i];
        }
        return result;
    }
};
// result[i] = (product of everything left of i) * (product of everything right of i)
// Pass 1 fills prefix products, Pass 2 multiplies in suffix products
// O(n) time, O(1) extra space (result array doesn't count)
// This is prefix/suffix product — same idea as prefix sum but with multiplication
```

### Practice problems
1. Range Sum Query (LC 303)
2. Subarray Sum Equals K (LC 560)
3. Continuous Subarray Sum (LC 523)
4. Product of Array Except Self (LC 238) — prefix + suffix product

---

## Meta Strategy: The Decision Flowchart

```
START: Read the problem
  │
  ├── Array/String problem?
  │   ├── Sorted? → Two pointers or Binary search
  │   ├── Subarray/substring? → Sliding window or Prefix sum
  │   ├── Need all combinations? → Backtracking
  │   └── Optimization? → DP (check which family)
  │
  ├── Graph/Tree problem?
  │   ├── Shortest path unweighted? → BFS
  │   ├── Shortest path weighted? → Dijkstra
  │   ├── Ordering/dependencies? → Topological sort
  │   ├── Connected components? → Union-Find or DFS
  │   ├── Tree traversal/property? → DFS (recursive)
  │   └── Level-based? → BFS
  │
  ├── Design problem?
  │   ├── LRU/LFU cache → Hash map + Doubly linked list
  │   ├── Autocomplete/prefix → Trie
  │   └── Min/Max tracking → Monotonic stack/deque or Heap
  │
  └── Look at constraints (n)
      ├── n ≤ 15-20 → Bitmask DP or brute force
      ├── n ≤ 100 → O(n³) ok
      ├── n ≤ 1,000 → O(n²) ok
      ├── n ≤ 100,000 → O(n log n) needed
      └── n ≤ 1,000,000+ → O(n) needed
```

---

## Study Plan (8 weeks)

| Week | Pattern | # Problems | Focus |
|---|---|---|---|
| 1 | Two pointers + Sliding window | 8 | Get comfortable with pointer manipulation |
| 2 | Binary search + Sorting | 6 | Master "search on answer" technique |
| 3 | BFS + DFS on trees | 8 | Tree problems are interview staples |
| 4 | Backtracking | 6 | Master the template, learn dedup |
| 5 | DP: Linear + Knapsack | 8 | Core DP skills |
| 6 | DP: Grid + Interval + advanced | 6 | Complete DP coverage |
| 7 | Graphs + Heap + Stack | 8 | Advanced patterns |
| 8 | Mixed practice | 10 | Identify patterns without hints |

**Total: ~60 problems, covering all major patterns.**

### Tips for each practice session
1. Set a timer: 25 min for medium, 40 min for hard
2. If stuck after 10 min, look at the pattern hint (which pattern), try again
3. If stuck after 20 min, look at the approach, implement yourself
4. After solving, read 2-3 other solutions — learn alternative approaches
5. Redo problems you struggled with after 3-5 days (spaced repetition)
