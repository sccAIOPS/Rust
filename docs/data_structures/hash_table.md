# Hash Table

## 1. Overview

A Hash Table (Hash Map) is a data structure that provides average O(1) time complexity for insert, delete, and lookup operations by using a hash function to compute an index into an array of buckets. It's one of the most important data structures in computer science, forming the backbone of dictionaries, sets, caches, and more.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a universe $U$ of possible keys and a set $S \subset U$ of $n$ keys to store, design a data structure supporting:
- **Insert(k, v)**: Associate value $v$ with key $k$
- **Search(k)**: Return value associated with $k$, or NOT_FOUND
- **Delete(k)**: Remove key $k$ and its value

### 2.2 Mathematical Model

**Hash Function**:
$$h: U \rightarrow \{0, 1, ..., m-1\}$$

Maps keys to indices in a table of size $m$.

**Load Factor**:
$$\alpha = \frac{n}{m}$$

Ratio of stored elements to table size.

**Ideal Properties of h**:
1. **Deterministic**: Same key always produces same hash
2. **Uniform Distribution**: Keys spread evenly across indices
3. **Efficient**: O(1) to compute

### 2.3 Collision Resolution

When $h(k_1) = h(k_2)$ for $k_1 \neq k_2$ (collision):

**Chaining**: Each bucket holds a list of colliding entries
**Open Addressing**: Find alternative slot (linear probing, quadratic probing, double hashing)

## 3. Algorithm Description

### 3.1 Intuition

Think of a hash table like a hotel with numbered rooms. The hash function is like a receptionist who assigns room numbers based on guest names. Most guests get unique rooms (fast access), but occasionally two guests get the same room number (collision) and need alternative arrangements.

### 3.2 Chaining Visualization

```
Hash table with chaining (m=5):
Insert: (15, "A"), (22, "B"), (30, "C"), (37, "D")

h(k) = k mod 5:
h(15) = 0, h(22) = 2, h(30) = 0, h(37) = 2

Index | Bucket (linked list)
------|---------------------
  0   | [15→A] → [30→C]
  1   | ∅
  2   | [22→B] → [37→D]
  3   | ∅
  4   | ∅
```

### 3.3 Open Addressing (Linear Probing)

```
Hash table (m=7):
Insert: 10, 22, 31, 4, 15

h(k) = k mod 7:
h(10)=3, h(22)=1, h(31)=3 (collision!), h(4)=4, h(15)=1 (collision!)

Step by step:
Insert 10: slot 3 empty → [_, _, _, 10, _, _, _]
Insert 22: slot 1 empty → [_, 22, _, 10, _, _, _]
Insert 31: slot 3 occupied, try 4 → [_, 22, _, 10, 31, _, _]
Insert 4:  slot 4 occupied, try 5 → [_, 22, _, 10, 31, 4, _]
Insert 15: slot 1 occupied, try 2 → [_, 22, 15, 10, 31, 4, _]
```

### 3.4 Pseudocode

**Chaining**:
```
INSERT(table, key, value):
    index = hash(key) mod table.size
    for entry in table[index]:
        if entry.key == key:
            entry.value = value  // Update
            return
    table[index].append((key, value))
    n++
    if n / table.size > LOAD_THRESHOLD:
        RESIZE(table)

SEARCH(table, key):
    index = hash(key) mod table.size
    for entry in table[index]:
        if entry.key == key:
            return entry.value
    return NOT_FOUND

DELETE(table, key):
    index = hash(key) mod table.size
    for i, entry in enumerate(table[index]):
        if entry.key == key:
            table[index].remove(i)
            n--
            return
```

**Linear Probing**:
```
INSERT(table, key, value):
    index = hash(key) mod table.size
    while table[index] is not EMPTY and table[index].key != key:
        index = (index + 1) mod table.size
    table[index] = (key, value)

SEARCH(table, key):
    index = hash(key) mod table.size
    while table[index] is not EMPTY:
        if table[index].key == key:
            return table[index].value
        index = (index + 1) mod table.size
    return NOT_FOUND
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Average | Worst Case |
|-----------|---------|------------|
| Insert    | O(1)    | O(n)       |
| Search    | O(1)    | O(n)       |
| Delete    | O(1)    | O(n)       |

**Analysis**:
- Average assumes uniform hashing and $\alpha < 1$
- Worst case: all keys hash to same index
- With good hash function, expected chain length = $\alpha$

### 4.2 Space Complexity

- **Storage**: O(n + m) where n = elements, m = table size
- **Chaining**: Extra space for linked list pointers
- **Open Addressing**: No extra pointers, but needs empty slots

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::collections::hash_map::DefaultHasher;
use std::hash::{Hash, Hasher};

const INITIAL_SIZE: usize = 32;
const LOAD_FACTOR_THRESHOLD: f64 = 0.75;

struct Entry<K, V> {
    key: K,
    value: V,
}

pub struct HashTable<K, V> {
    buckets: Vec<Vec<Entry<K, V>>>,
    count: usize,
}

impl<K: Hash + Eq, V> HashTable<K, V> {
    fn hash_index(&self, key: &K) -> usize {
        let mut hasher = DefaultHasher::new();
        key.hash(&mut hasher);
        (hasher.finish() as usize) % self.buckets.len()
    }
    
    pub fn insert(&mut self, key: K, value: V) {
        if self.load_factor() > LOAD_FACTOR_THRESHOLD {
            self.resize();
        }
        
        let index = self.hash_index(&key);
        for entry in &mut self.buckets[index] {
            if entry.key == key {
                entry.value = value;
                return;
            }
        }
        self.buckets[index].push(Entry { key, value });
        self.count += 1;
    }
}
```

**Key Design Patterns**:
- `Hash` trait for custom types
- `DefaultHasher` for SipHash (DoS-resistant)
- `Vec<Vec<...>>` for chaining
- Automatic resizing at load threshold

### 5.2 Hash Function Considerations

```rust
// Custom hash for composite type
#[derive(Hash, Eq, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

// Manual implementation
impl Hash for CustomType {
    fn hash<H: Hasher>(&self, state: &mut H) {
        self.field1.hash(state);
        self.field2.hash(state);
    }
}
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Empty table | Search returns None |
| Key not found | Return None, don't error |
| Duplicate key | Update value |
| High load factor | Resize table |
| Deletion with probing | Use tombstone or rehash |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Databases**: Index structures
2. **Caching**: Memoization, LRU cache
3. **Symbol Tables**: Compilers, interpreters
4. **Networking**: IP routing tables
5. **Deduplication**: Detecting duplicates
6. **Frequency Counting**: Word counts, analytics

### 6.2 Choosing Collision Resolution

| Method | Pros | Cons |
|--------|------|------|
| **Chaining** | Simple, α > 1 OK | Pointer overhead |
| **Linear Probing** | Cache-friendly | Clustering |
| **Quadratic Probing** | Less clustering | Secondary clustering |
| **Double Hashing** | Best distribution | Two hash functions |
| **Cuckoo Hashing** | O(1) worst search | Complex, two tables |
| **Robin Hood** | Low variance | Complex deletion |

### 6.3 Standard Library

```rust
use std::collections::HashMap;

let mut map = HashMap::new();
map.insert("key", "value");
let v = map.get("key");  // Some(&"value")
map.remove("key");

// Entry API for efficient update-or-insert
map.entry("count").and_modify(|v| *v += 1).or_insert(1);
```

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Poor Hash Function**: Leads to clustering
2. **Hash DoS**: Predictable hashing allows attacks
3. **Mutable Keys**: Changing key after insert breaks lookup
4. **Ignoring Load Factor**: Performance degrades as α increases
5. **Deletion in Open Addressing**: Requires tombstones

### 7.2 Optimization Opportunities

**SIMD Probing**: Use SIMD to check multiple slots at once

**Power-of-2 Table Size**: Use bitwise AND instead of modulo
```rust
index = hash & (size - 1)  // Instead of hash % size
```

**Swiss Table (Google)**: 
- Groups of 16 slots with metadata byte each
- SIMD parallel probing
- Used in Rust's `hashbrown` (backing `HashMap`)

### 7.3 Robin Hood Hashing

Reduces variance in probe lengths:
```
If inserting element has traveled farther than current element,
swap and continue inserting the displaced element.
```

Results in more predictable performance.

### 7.4 Cryptographic vs Non-Cryptographic Hash

| Type | Example | Speed | Collision Resistance |
|------|---------|-------|---------------------|
| Cryptographic | SHA-256 | Slow | High |
| Non-Crypto | FxHash, xxHash | Fast | Moderate |
| Rust Default | SipHash | Medium | DoS-resistant |

Use non-cryptographic for hash tables (unless DoS is a concern).

## 8. References

- Cormen, T. H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Chapter 11.
- Knuth, D. E. (1998). *The Art of Computer Programming, Volume 3: Sorting and Searching* (2nd ed.). Section 6.4.
- Sedgewick, R. (1988). *Algorithms* (2nd ed.). Addison-Wesley. Chapter 14.
- Abseil Swiss Tables: https://abseil.io/about/design/swisstables
