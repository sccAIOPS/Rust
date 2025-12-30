# Linked List

## 1. Overview

A Linked List is a linear data structure where elements are stored in nodes, each containing data and a reference (pointer) to the next node. Unlike arrays, linked lists don't require contiguous memory, allowing efficient insertion and deletion at any position at the cost of sequential access.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Design a linear collection supporting:
- **Insert**: Add element at any position
- **Delete**: Remove element from any position
- **Access**: Retrieve element at position
- **Traverse**: Visit all elements in order

### 2.2 Mathematical Model

**Node Definition**:
$$\text{Node} = (\text{data}: T, \text{next}: \text{Option<Node>})$$

**List Definition**:
A sequence of nodes $n_0 \rightarrow n_1 \rightarrow ... \rightarrow n_{k-1} \rightarrow \text{None}$

**Variants**:
- **Singly Linked**: Each node points to next only
- **Doubly Linked**: Each node points to next and previous
- **Circular**: Last node points back to first

## 3. Algorithm Description

### 3.1 Intuition

Think of a linked list like a treasure hunt where each clue (node) tells you where to find the next clue. You can easily insert a new clue in the middle (just redirect pointers) but finding the 5th clue requires following the first 4 clues.

### 3.2 Structure Visualization

```
Singly Linked List:
  Head
   ↓
 [10|→]──→[20|→]──→[30|→]──→[40|/]
   
Doubly Linked List:
  Head                               Tail
   ↓                                  ↓
 [/|10|→]←→[←|20|→]←→[←|30|→]←→[←|40|/]
```

### 3.3 Pseudocode

```
PUSH_FRONT(list, value):
    new_node = Node(value)
    new_node.next = list.head
    list.head = new_node
    list.length++

PUSH_BACK(list, value):
    new_node = Node(value)
    if list.head is None:
        list.head = new_node
    else:
        current = list.head
        while current.next is not None:
            current = current.next
        current.next = new_node
    list.length++

POP_FRONT(list):
    if list.head is None:
        return None
    value = list.head.data
    list.head = list.head.next
    list.length--
    return value

INSERT_AT(list, index, value):
    if index == 0:
        PUSH_FRONT(list, value)
        return
    current = list.head
    for i in 0 to index - 2:
        if current is None:
            error "Index out of bounds"
        current = current.next
    new_node = Node(value)
    new_node.next = current.next
    current.next = new_node
    list.length++

GET(list, index):
    current = list.head
    for i in 0 to index - 1:
        if current is None:
            error "Index out of bounds"
        current = current.next
    return current.data
```

### 3.4 Step-by-Step Example

**Insert 30 between 20 and 40**:

```
Before: [10|→]──→[20|→]──→[40|/]
                  ↑
               current

1. Create new node [30|?]
2. new_node.next = current.next (points to 40)
   [30|→]──→[40|/]
3. current.next = new_node
   [20|→]──→[30|→]──→[40|/]

After: [10|→]──→[20|→]──→[30|→]──→[40|/]
```

**Delete node with value 30**:

```
Before: [10|→]──→[20|→]──→[30|→]──→[40|/]
                  ↑         ↑
                prev     current

1. prev.next = current.next
   [20|→]──→[40|/]
   (node 30 is now orphaned)

After: [10|→]──→[20|→]──→[40|/]
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Singly Linked | Doubly Linked | Array |
|-----------|---------------|---------------|-------|
| Access (index) | O(n) | O(n) | O(1) |
| Search | O(n) | O(n) | O(n) |
| Insert front | O(1) | O(1) | O(n) |
| Insert back | O(n)* | O(1)** | O(1)*** |
| Insert middle | O(n) | O(n) | O(n) |
| Delete front | O(1) | O(1) | O(n) |
| Delete back | O(n) | O(1)** | O(1) |
| Delete middle | O(n) | O(n) | O(n) |

*O(1) with tail pointer  
**With tail pointer  
***Amortized

### 4.2 Space Complexity

- **Storage**: O(n) for n elements
- **Per Node**: sizeof(T) + sizeof(pointer) × (1 or 2)
- **Overhead**: Pointer(s) per element (significant for small T)

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
type Link<T> = Option<Box<Node<T>>>;

struct Node<T> {
    val: T,
    next: Link<T>,
}

pub struct LinkedList<T> {
    head: Link<T>,
    length: usize,
}
```

**Key Design Patterns**:
- `Option<Box<Node<T>>>` for nullable owned pointers
- `Box` for heap allocation of nodes
- `Option::take()` for ownership transfer
- `as_ref()` and `as_mut()` for borrowing

**Common Operations**:
```rust
// Take ownership of next node
let old_head = self.head.take();

// Borrow next node
if let Some(ref node) = self.head {
    // use node
}

// Mutable borrow for modification
if let Some(ref mut node) = self.head {
    node.val = new_value;
}
```

### 5.2 Iterator Implementation

```rust
pub struct Iter<'a, T> {
    next: Option<&'a Node<T>>,
}

impl<T> LinkedList<T> {
    pub fn iter(&self) -> Iter<T> {
        Iter { next: self.head.as_deref() }
    }
}

impl<'a, T> Iterator for Iter<'a, T> {
    type Item = &'a T;
    
    fn next(&mut self) -> Option<Self::Item> {
        self.next.map(|node| {
            self.next = node.next.as_deref();
            &node.val
        })
    }
}
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Empty list | head = None, length = 0 |
| Single element | head.next = None |
| Insert at 0 | Update head pointer |
| Delete last | Traverse to second-to-last |
| Index out of bounds | Return None or error |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Implementation of Other Structures**: Stacks, queues, hash tables
2. **Undo Functionality**: Each action is a node
3. **Memory Allocators**: Free list management
4. **Music Playlists**: Next/previous song
5. **Browser History**: Back/forward navigation
6. **LRU Cache**: Doubly linked list + hash map

### 6.2 Comparison with Arrays

| Aspect | Linked List | Array/Vec |
|--------|-------------|-----------|
| Memory | Non-contiguous | Contiguous |
| Cache | Poor locality | Good locality |
| Random access | O(n) | O(1) |
| Insert/delete | O(1)* | O(n) |
| Memory overhead | High (pointers) | Low |
| Size | Dynamic | Fixed or amortized |

*At known position

### 6.3 Variants

| Variant | Extra Feature |
|---------|---------------|
| **Singly Linked** | Basic, minimal memory |
| **Doubly Linked** | Bidirectional traversal |
| **Circular** | Last connects to first |
| **Skip List** | Multiple levels for O(log n) search |
| **XOR Linked List** | Memory-efficient doubly linked |
| **Unrolled** | Multiple elements per node |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Lost References**: Modifying next before saving it
2. **Memory Leaks**: In languages without GC
3. **Null Pointer**: Forgetting to check for end
4. **Off-by-One**: Index calculations
5. **Rust Ownership**: Fighting the borrow checker

### 7.2 Rust-Specific Challenges

**The Doubly Linked List Problem**:
```rust
// This doesn't work in safe Rust:
struct Node<T> {
    val: T,
    next: Option<Box<Node<T>>>,
    prev: Option<Box<Node<T>>>,  // Who owns prev?
}
```

**Solutions**:
1. Use `Rc<RefCell<Node<T>>>` (reference counting)
2. Use `unsafe` with raw pointers
3. Use indices into a `Vec` instead of pointers
4. Use `std::collections::LinkedList`

### 7.3 When to Use (and When Not To)

**Use Linked List When**:
- Frequent insertions/deletions at known positions
- Don't need random access
- Can't predict size in advance
- Implementing other data structures

**Use Array/Vec When**:
- Need random access
- Iterating is common (cache efficiency)
- Memory overhead matters
- Size is relatively stable

### 7.4 Standard Library

```rust
use std::collections::LinkedList;

let mut list = LinkedList::new();
list.push_back(1);
list.push_front(0);
list.pop_front();  // Returns Some(0)

// Cursor API for O(1) operations
let mut cursor = list.cursor_front_mut();
cursor.insert_after(5);
```

## 8. References

- Knuth, D. E. (1997). *The Art of Computer Programming, Volume 1: Fundamental Algorithms* (3rd ed.). Section 2.2.
- Sedgewick, R. (1988). *Algorithms* (2nd ed.). Addison-Wesley. Chapter 3.
- "Learning Rust With Entirely Too Many Linked Lists". https://rust-unofficial.github.io/too-many-lists/
- Rust std::collections::LinkedList documentation.
