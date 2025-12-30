# AVL Tree

## 1. Overview

An AVL tree (named after inventors **A**delson-**V**elsky and **L**andis, 1962) is a self-balancing binary search tree where the heights of the two child subtrees of any node differ by at most one. This height-balance property guarantees O(log n) time complexity for search, insert, and delete operations.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Maintain a BST with the additional constraint that tree height remains O(log n), ensuring worst-case O(log n) operations.

### 2.2 Mathematical Model

**AVL Property** (Balance Invariant):
For every node $n$:
$$|\text{height}(\text{left}(n)) - \text{height}(\text{right}(n))| \leq 1$$

**Balance Factor**:
$$\text{BF}(n) = \text{height}(\text{right}(n)) - \text{height}(\text{left}(n))$$

Valid balance factors: $\{-1, 0, +1\}$

**Height Bound**:
An AVL tree with $n$ nodes has height:
$$h < 1.44 \log_2(n+2) - 0.328$$

This is derived from Fibonacci numbers: minimum nodes at height $h$ is $N_h = F_{h+2} - 1$.

### 2.3 Correctness Proof

**Theorem**: After any insert or delete operation followed by rebalancing, the AVL property is restored.

**Proof Sketch**:
1. Operations may increase/decrease heights by at most 1 per level
2. Rebalancing (rotations) reduce height differences back to $\leq 1$
3. At most O(log n) rotations needed (one per ancestor)

## 3. Algorithm Description

### 3.1 Intuition

AVL trees "fix" the BST's worst-case scenario by performing rotations whenever the tree becomes unbalanced. Think of rotations as "pivoting" a subtree to redistribute nodes more evenly.

### 3.2 Rotations

Four rotation cases handle imbalance:

**Case 1: Left-Left (LL)** - Right rotation
```
        z                      y
       / \                   /   \
      y   T4   Right        x     z
     / \       Rotate      / \   / \
    x   T3     ------>    T1 T2 T3 T4
   / \
  T1  T2
```

**Case 2: Right-Right (RR)** - Left rotation
```
    z                          y
   / \                       /   \
  T1  y       Left          z     x
     / \      Rotate       / \   / \
    T2  x     ------>     T1 T2 T3 T4
       / \
      T3  T4
```

**Case 3: Left-Right (LR)** - Left then Right rotation
```
      z                z                 x
     / \             / \               /   \
    y   T4   Left   x   T4   Right    y     z
   / \       --->  / \       --->    / \   / \
  T1  x           y   T3            T1 T2 T3 T4
     / \         / \
    T2  T3      T1  T2
```

**Case 4: Right-Left (RL)** - Right then Left rotation
```
    z                 z                   x
   / \               / \                /   \
  T1  y    Right    T1  x     Left     z     y
     / \   --->       / \     --->    / \   / \
    x   T4           T2  y           T1 T2 T3 T4
   / \                  / \
  T2  T3               T3  T4
```

### 3.3 Pseudocode

```
INSERT(tree, value):
    if tree is empty:
        return new Node(value, height=1)
    
    if value < tree.value:
        tree.left = INSERT(tree.left, value)
    else if value > tree.value:
        tree.right = INSERT(tree.right, value)
    else:
        return tree  // Duplicate, no insert
    
    tree.height = 1 + max(height(tree.left), height(tree.right))
    
    return REBALANCE(tree)

REBALANCE(node):
    balance = BALANCE_FACTOR(node)
    
    // Left Heavy
    if balance < -1:
        if BALANCE_FACTOR(node.left) > 0:  // LR case
            node.left = LEFT_ROTATE(node.left)
        return RIGHT_ROTATE(node)
    
    // Right Heavy
    if balance > 1:
        if BALANCE_FACTOR(node.right) < 0:  // RL case
            node.right = RIGHT_ROTATE(node.right)
        return LEFT_ROTATE(node)
    
    return node  // Already balanced
```

### 3.4 Step-by-Step Example

Insert sequence: [10, 20, 30, 40, 50, 25]

```
Insert 10:        Insert 20:       Insert 30 (imbalance!):
    10               10                  10
                      \                   \
                      20                  20
                                           \
                                           30
                                    BF(10) = 2 → Left Rotate

After rotation:   Insert 40:       Insert 50 (imbalance!):
    20               20                  20
   /  \             /  \                /  \
  10  30           10  30              10  30
                        \                   \
                        40                  40
                                             \
                                             50
                                    BF(30) = 2 → Left Rotate

After rotation:   Insert 25:
    20                 20
   /  \               /  \
  10  40             10  40
     /  \               /  \
    30  50             30  50
                       /
                      25
                 (balanced, BF ∈ {-1,0,1})
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Time Complexity |
|-----------|-----------------|
| Search    | O(log n)        |
| Insert    | O(log n)        |
| Delete    | O(log n)        |
| Min/Max   | O(log n)        |

**All operations are O(log n) worst-case** due to the height bound.

**Rotation Cost**: Each rotation is O(1), and at most O(log n) rotations per operation.

### 4.2 Space Complexity

- **Storage**: O(n) for n nodes
- **Per Node Overhead**: height field (1 integer)
- **Auxiliary (recursive)**: O(log n) stack space

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
struct AVLNode<T: Ord> {
    value: T,
    height: usize,
    left: Option<Box<AVLNode<T>>>,
    right: Option<Box<AVLNode<T>>>,
}

pub struct AVLTree<T: Ord> {
    root: Option<Box<AVLNode<T>>>,
    length: usize,
}
```

**Key Design Patterns**:
- `mem::swap` for efficient node rotation without cloning
- `Side` enum for abstracting left/right operations
- Height stored in each node for O(1) balance factor calculation
- `FromIterator` implementation for easy construction

**Rotation Implementation**:
```rust
fn rotate(&mut self, side: Side) {
    let mut subtree = self.child_mut(!side).take().unwrap();
    *self.child_mut(!side) = subtree.child_mut(side).take();
    self.update_height();
    mem::swap(self, subtree.as_mut());
    *self.child_mut(side) = Some(subtree);
    self.update_height();
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty tree | Insert creates root |
| Duplicate value | Reject (return false) |
| Delete leaf | Simply remove |
| Delete node with one child | Replace with child |
| Delete node with two children | Replace with in-order successor |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **In-Memory Databases**: Fast ordered access with guaranteed performance
2. **File System Metadata**: Directory indexing (ext4 uses related structures)
3. **Compiler Symbol Tables**: Maintaining sorted identifiers
4. **Memory Allocators**: Tracking free memory blocks by size
5. **Real-Time Systems**: Predictable O(log n) guarantees

### 6.2 Related Algorithms

| Structure | Comparison |
|-----------|------------|
| **Red-Black Tree** | Less strict balancing (height ≤ 2 log n), faster inserts |
| **B-Tree** | More children, better for disk I/O |
| **Splay Tree** | Self-adjusting, good for skewed access |
| **Treap** | Randomized, simpler implementation |

### AVL vs Red-Black Trade-offs

| Aspect | AVL | Red-Black |
|--------|-----|-----------|
| Height | ≤ 1.44 log n | ≤ 2 log n |
| Lookups | Faster | Slower |
| Insertions | More rotations | Fewer rotations |
| Use case | Read-heavy | Write-heavy |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Incorrect Rotation Order**: LR/RL cases need two rotations
2. **Height Update Timing**: Must update after modifying children
3. **Balance Factor Sign**: Consistent convention needed

### 7.2 Optimization Opportunities

- **Iterative Implementation**: Avoid recursion stack overhead
- **Parent Pointers**: O(1) access to ancestors for bottom-up rebalancing
- **Lazy Height Update**: Only recalculate when needed
- **Cache-Friendly Layout**: Array-based storage for better locality

## 8. References

- Adelson-Velsky, G. M., & Landis, E. M. (1962). "An algorithm for the organization of information". *Proceedings of the USSR Academy of Sciences*, 146, 263-266.
- Cormen, T. H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Chapter 13.
- Knuth, D. E. (1997). *The Art of Computer Programming, Volume 3*. Section 6.2.3.
