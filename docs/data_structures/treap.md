# Treap

## 1. Overview

A Treap (Tree + Heap) is a randomized binary search tree that combines BST ordering on keys with heap ordering on randomly assigned priorities. This randomization provides expected O(log n) operations without explicit balancing, making it simpler to implement than deterministic balanced trees.

Invented by Cecilia Aragon and Raimund Seidel in 1989.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Maintain a balanced BST using randomization instead of explicit balancing rotations, achieving O(log n) expected time for all operations.

### 2.2 Mathematical Model

**Treap Properties**:
1. **BST Property**: For each node, left subtree keys < node key < right subtree keys
2. **Heap Property**: For each node, node priority ≥ children priorities (max-heap)

**Random Priority Assignment**:
Each inserted key is assigned a random priority from a continuous distribution. The resulting tree structure is equivalent to inserting keys in the order of their priorities.

**Expected Height**:
$$E[\text{height}] = O(\log n)$$

**Proof Sketch**: A treap is equivalent to a random BST built by inserting keys in random order (determined by priorities). Random BSTs have expected height O(log n).

### 2.3 Uniqueness Theorem

**Theorem**: Given distinct keys and distinct priorities, there exists a unique treap satisfying both BST and heap properties.

**Proof**: The root must be the element with the highest priority. Its key partitions remaining elements into left (smaller keys) and right (larger keys) subtrees. Apply recursively.

## 3. Algorithm Description

### 3.1 Intuition

Imagine building a BST by inserting elements in random order—the resulting tree is likely balanced. A treap simulates this by assigning random priorities at insertion time. When priority order differs from insertion order, rotations restore the heap property while maintaining BST property.

### 3.2 Rotations

```
Right Rotation:           Left Rotation:
     y                         x
    / \                       / \
   x   C   --->              A   y
  / \                           / \
 A   B                         B   C
```

Rotations preserve BST property while adjusting heap property.

### 3.3 Pseudocode

```
INSERT(tree, key):
    if tree is NULL:
        return new Node(key, random_priority())
    
    if key < tree.key:
        tree.left = INSERT(tree.left, key)
        if tree.left.priority > tree.priority:
            tree = RIGHT_ROTATE(tree)
    else if key > tree.key:
        tree.right = INSERT(tree.right, key)
        if tree.right.priority > tree.priority:
            tree = LEFT_ROTATE(tree)
    // else: duplicate key, no insert
    
    return tree

DELETE(tree, key):
    if tree is NULL:
        return NULL
    
    if key < tree.key:
        tree.left = DELETE(tree.left, key)
    else if key > tree.key:
        tree.right = DELETE(tree.right, key)
    else:
        // Found the key to delete
        if tree.left is NULL:
            return tree.right
        if tree.right is NULL:
            return tree.left
        
        // Both children exist: rotate smaller priority down
        if tree.left.priority > tree.right.priority:
            tree = RIGHT_ROTATE(tree)
            tree.right = DELETE(tree.right, key)
        else:
            tree = LEFT_ROTATE(tree)
            tree.left = DELETE(tree.left, key)
    
    return tree
```

### 3.4 Step-by-Step Example

Insert (key, priority) pairs: (5, 10), (3, 20), (7, 15), (2, 5)

```
Insert (5, 10):     Insert (3, 20):
   (5,10)              (5,10)
                       /
                    (3,20)
                    
Priority 20 > 10, rotate right:
   (3,20)
       \
      (5,10)

Insert (7, 15):              Insert (2, 5):
   (3,20)                       (3,20)
       \                       /     \
      (5,10)                (2,5)  (5,10)
          \                            \
         (7,15)                       (7,15)

Priority 15 > 10, rotate left at (5,10):
   (3,20)
   /    \
(2,5)  (7,15)
       /
    (5,10)
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Expected | Worst Case |
|-----------|----------|------------|
| Search    | O(log n) | O(n)       |
| Insert    | O(log n) | O(n)       |
| Delete    | O(log n) | O(n)       |
| Split     | O(log n) | O(n)       |
| Merge     | O(log n) | O(n)       |

**Note**: Worst case O(n) has probability approaching 0 as n increases.

### 4.2 Space Complexity

- **Storage**: O(n) for n nodes
- **Per Node**: key + priority + 2 pointers
- **Auxiliary**: O(log n) expected for recursion

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
struct TreapNode<T: Ord> {
    value: T,
    priority: usize,
    left: Option<Box<TreapNode<T>>>,
    right: Option<Box<TreapNode<T>>>,
}

pub struct Treap<T: Ord> {
    root: Option<Box<TreapNode<T>>>,
    length: usize,
}
```

**Key Design Patterns**:
- Random priority via `SystemTime::now().subsec_nanos()` (or use `rand` crate)
- `mem::swap` for in-place rotation
- `Side` enum to abstract left/right operations
- `Option<Box<...>>` for optional children

**Random Priority Generation**:
```rust
fn rand() -> usize {
    SystemTime::now()
        .duration_since(UNIX_EPOCH)
        .unwrap()
        .subsec_nanos() as usize
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty tree | Insert creates root |
| Duplicate priority | Extremely rare with random priorities |
| All same priority | Degenerates to unbalanced BST |
| Delete leaf | Simply remove |
| Delete internal | Rotate down until leaf, then remove |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Randomized Algorithms**: Monte Carlo simulations
2. **Compiler Optimization**: Randomized data structures in JIT
3. **Game Development**: Quick spatial indexing
4. **Concurrent Data Structures**: Lock-free variations exist
5. **Educational**: Simpler to understand than AVL/RB

### 6.2 Extended Operations

**Split**: Divide treap into two based on key
```
SPLIT(tree, key) → (left_tree, right_tree)
```

**Merge**: Combine two treaps where all keys in left < all keys in right
```
MERGE(left, right) → combined_tree
```

These operations enable:
- Efficient range operations
- Persistent data structures
- Rope-like string operations

### 6.3 Related Algorithms

| Structure | Comparison |
|-----------|------------|
| **AVL Tree** | Deterministic O(log n), complex rotations |
| **Red-Black Tree** | Deterministic, more cases to handle |
| **Skip List** | Also randomized, different structure |
| **Splay Tree** | Amortized O(log n), self-adjusting |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Poor RNG**: Must use good random number generator
2. **Priority Collisions**: Should use 64-bit priorities
3. **Forgetting Rotations**: Must rotate after recursive call returns
4. **Delete Complexity**: Rotation direction matters

### 7.2 Optimization Opportunities

- **Implicit Keys**: Store subtree sizes, use index as implicit key
- **Persistent Treaps**: Path copying for immutable versions
- **Implicit Treap**: Sequence operations (like rope)
- **Weighted Treap**: Non-random priorities for biased structure

### 7.3 Implicit Key Treap

Store only values, not keys. Position in in-order traversal is the implicit key:
```rust
struct ImplicitTreap<T> {
    value: T,
    priority: usize,
    size: usize,  // subtree size
    left: ...,
    right: ...,
}
```

Enables O(log n) array operations:
- Insert at position
- Delete at position
- Reverse range
- Cut and concatenate

## 8. References

- Aragon, C. R., & Seidel, R. (1989). "Randomized Search Trees". *FOCS '89*.
- Seidel, R., & Aragon, C. R. (1996). "Randomized Search Trees". *Algorithmica*.
- Martínez, C., & Roura, S. (1997). "Randomized Binary Search Trees". *JACM*.
- Blelloch, G. E., & Reid-Miller, M. (1998). "Fast Set Operations Using Treaps". *SPAA '98*.
