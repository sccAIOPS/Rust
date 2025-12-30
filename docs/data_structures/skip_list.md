# Skip List

## 1. Overview

A Skip List is a probabilistic data structure that allows O(log n) average-case search, insertion, and deletion in a sorted sequence. It uses multiple levels of linked lists with increasingly sparse elements, allowing "express lane" traversal that skips over many elements.

Invented by William Pugh in 1989 as a simpler alternative to balanced trees.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Maintain a sorted set supporting:
- **Search(x)**: Find if $x$ exists
- **Insert(x)**: Add $x$ to the set
- **Delete(x)**: Remove $x$ from the set
- **Predecessor/Successor**: Find nearest elements

### 2.2 Mathematical Model

**Structure**:
- Multiple levels (0, 1, 2, ..., max_level)
- Each element appears at level 0
- Each element at level $i$ appears at level $i+1$ with probability $p$ (typically 0.5)

**Expected Height of Element**:
$$E[\text{height}] = \frac{1}{1-p} = 2 \text{ (for } p=0.5)$$

**Expected Maximum Level**:
$$E[\text{max level}] = \log_{1/p} n = \log_2 n \text{ (for } p=0.5)$$

**Expected Search Time**:
$$O\left(\frac{\log n}{|\log p|}\right) = O(\log n)$$

### 2.3 Key Insight

At each level, we expect to skip about $1/p$ elements. With $\log_{1/p} n$ levels, total steps ≈ $\log n / |\log p|$ = O(log n).

## 3. Algorithm Description

### 3.1 Intuition

Imagine a sorted linked list where some elements are "promoted" to express lanes above. To find an element, start at the top express lane and move right until you'd overshoot, then drop down a level and repeat. Like using highways + local roads for navigation.

### 3.2 Structure Visualization

```
Skip List containing [1, 3, 4, 6, 7, 9, 12, 17, 19, 21, 25]:

Level 3: HEAD ────────────────────────────→ 17 ─────────────────→ NIL
Level 2: HEAD ──────────→ 6 ──────────────→ 17 ──────────→ 25 ──→ NIL  
Level 1: HEAD ────→ 3 ──→ 6 ────→ 9 ──────→ 17 ───→ 21 ──→ 25 ──→ NIL
Level 0: HEAD → 1 → 3 → 4 → 6 → 7 → 9 → 12 → 17 → 19 → 21 → 25 → NIL

Search for 19:
1. Start at HEAD level 3, go right to 17
2. 17 < 19, but 17's next is NIL, drop to level 2
3. At 17 level 2, next is 25 > 19, drop to level 1
4. At 17 level 1, next is 21 > 19, drop to level 0
5. At 17 level 0, next is 19, found!
```

### 3.3 Pseudocode

```
RANDOM_LEVEL():
    level = 0
    while random() < p and level < MAX_LEVEL:
        level++
    return level

SEARCH(list, target):
    current = list.head
    for level from MAX_LEVEL down to 0:
        while current.next[level] != NIL and current.next[level].key < target:
            current = current.next[level]
    current = current.next[0]
    if current != NIL and current.key == target:
        return current
    return NOT_FOUND

INSERT(list, key, value):
    update = array of size MAX_LEVEL + 1
    current = list.head
    
    // Find insertion position at each level
    for level from MAX_LEVEL down to 0:
        while current.next[level] != NIL and current.next[level].key < key:
            current = current.next[level]
        update[level] = current
    
    current = current.next[0]
    if current != NIL and current.key == key:
        current.value = value  // Update existing
        return
    
    // Insert new node
    new_level = RANDOM_LEVEL()
    new_node = Node(key, value, new_level)
    
    for level from 0 to new_level:
        new_node.next[level] = update[level].next[level]
        update[level].next[level] = new_node

DELETE(list, key):
    update = array of size MAX_LEVEL + 1
    current = list.head
    
    for level from MAX_LEVEL down to 0:
        while current.next[level] != NIL and current.next[level].key < key:
            current = current.next[level]
        update[level] = current
    
    current = current.next[0]
    if current == NIL or current.key != key:
        return NOT_FOUND
    
    for level from 0 to current.level:
        update[level].next[level] = current.next[level]
```

### 3.4 Step-by-Step Example

**Insert 8 into the skip list above**:

```
1. Generate random level: say level = 2

2. Find predecessors at each level:
   Level 3: HEAD (8 < 17)
   Level 2: 6 (8 > 6, 8 < 17)
   Level 1: 6 (8 > 6, 8 < 9)
   Level 0: 7 (8 > 7, 8 < 9)
   
   update = [node(7), node(6), node(6)]

3. Insert new node at levels 0, 1, 2:
   
Before:                          After:
Level 2: 6 ────────→ 17         Level 2: 6 ──→ 8 ────→ 17
Level 1: 6 ────→ 9 ──→ 17       Level 1: 6 ──→ 8 ──→ 9 ──→ 17
Level 0: 7 → 9                   Level 0: 7 → 8 → 9
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Average | Worst Case |
|-----------|---------|------------|
| Search    | O(log n) | O(n)      |
| Insert    | O(log n) | O(n)      |
| Delete    | O(log n) | O(n)      |

**Worst case O(n)** occurs when random levels are all 0 (degenerates to linked list). Probability is negligible for reasonable n.

### 4.2 Space Complexity

- **Expected**: O(n) total pointers
- **Each element**: Expected 2 pointers (for p=0.5)
- **Total**: O(n × 1/(1-p)) = O(n) for constant p

### 4.3 Comparison with Balanced Trees

| Aspect | Skip List | AVL/Red-Black |
|--------|-----------|---------------|
| Average time | O(log n) | O(log n) |
| Worst time | O(n) | O(log n) |
| Implementation | Simpler | Complex |
| Memory | ~2n pointers | ~3n pointers |
| Concurrency | Easier | Harder |
| Cache | Worse | Better |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::cmp::Ordering;

const MAX_LEVEL: usize = 16;
const P: f64 = 0.5;

struct Node<K, V> {
    key: K,
    value: V,
    forward: Vec<Option<Box<Node<K, V>>>>,
}

pub struct SkipList<K, V> {
    head: Box<Node<K, V>>,
    level: usize,
    length: usize,
}

impl<K: Ord + Default, V: Default> SkipList<K, V> {
    fn random_level() -> usize {
        let mut level = 0;
        while rand::random::<f64>() < P && level < MAX_LEVEL {
            level += 1;
        }
        level
    }
    
    pub fn insert(&mut self, key: K, value: V) {
        let mut update: Vec<*mut Node<K, V>> = vec![std::ptr::null_mut(); MAX_LEVEL + 1];
        let mut current = &mut *self.head as *mut Node<K, V>;
        
        // Find position (using raw pointers for flexibility)
        for i in (0..=self.level).rev() {
            unsafe {
                while let Some(ref mut next) = (*current).forward[i] {
                    if next.key < key {
                        current = &mut **next;
                    } else {
                        break;
                    }
                }
                update[i] = current;
            }
        }
        
        let new_level = Self::random_level();
        if new_level > self.level {
            for i in (self.level + 1)..=new_level {
                update[i] = &mut *self.head;
            }
            self.level = new_level;
        }
        
        // Create and link new node
        // ... (full implementation continues)
    }
}
```

**Key Challenges in Rust**:
- Multiple mutable references during traversal
- Solutions: raw pointers, `Rc<RefCell<>>`, or index-based approach

### 5.2 Index-Based Alternative

```rust
struct Node<K, V> {
    key: K,
    value: V,
    forward: Vec<Option<usize>>,  // Indices instead of pointers
}

pub struct SkipList<K, V> {
    nodes: Vec<Node<K, V>>,
    head: usize,
    free_list: Vec<usize>,
    level: usize,
}
```

This is more Rust-friendly (no unsafe), but requires manual memory management.

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Empty list | Search returns None |
| Duplicate key | Update value or reject |
| Delete non-existent | Return None |
| Very unlucky random | Still correct, just slow |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Redis**: Sorted sets use skip lists
2. **LevelDB/RocksDB**: MemTable implementation
3. **Concurrent Maps**: Lock-free skip lists
4. **Priority Queues**: Alternative to heaps
5. **Database Indexes**: In-memory indexes

### 6.2 Why Redis Uses Skip Lists

From Redis creator Antirez:
1. Simpler to implement and debug
2. Easy to modify for range operations
3. Similar performance to balanced trees
4. Works well with their memory allocator

### 6.3 Concurrent Skip Lists

Skip lists are naturally suited for concurrent access:
- Lock-free implementations possible
- Can lock individual nodes (not whole subtrees)
- Java's `ConcurrentSkipListMap` uses this

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Poor RNG**: Must use good random number generator
2. **Fixed MAX_LEVEL**: Should be ≥ log_{1/p}(expected n)
3. **Memory Overhead**: Each level adds pointers
4. **Cache Misses**: Pointer chasing is slow

### 7.2 Optimization Opportunities

**Deterministic Skip List**: Use pattern instead of random
- E.g., every 2nd element at level 1, every 4th at level 2
- Removes randomness, guarantees O(log n)
- But loses simplicity

**Unrolled Skip List**: Store multiple elements per node
- Better cache utilization
- Fewer pointer traversals

**Finger Search**: Cache recent search position
- O(log d) search where d = distance from finger
- Great for sequential access patterns

### 7.3 Choosing p Value

| p | Avg pointers/node | Levels for 1M elements |
|---|-------------------|------------------------|
| 0.25 | 1.33 | 10 |
| 0.5 | 2.0 | 20 |
| 0.75 | 4.0 | 40 |

- Lower p = fewer pointers, more comparisons
- Higher p = more pointers, fewer comparisons
- p = 0.5 or 0.25 commonly used

## 8. References

- Pugh, W. (1990). "Skip Lists: A Probabilistic Alternative to Balanced Trees". *Communications of the ACM*.
- Pugh, W. (1990). "Concurrent Maintenance of Skip Lists". Technical Report CS-TR-2222.
- Herlihy, M., et al. (2006). "A Provably Correct Scalable Concurrent Skip List". *OPODIS 2006*.
- Redis Documentation: https://redis.io/docs/data-types/sorted-sets/
