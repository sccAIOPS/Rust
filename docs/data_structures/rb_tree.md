# Red-Black Tree

## 1. Overview

A Red-Black Tree is a self-balancing binary search tree where each node has an additional "color" attribute (red or black). The tree maintains balance through a set of color-based rules, ensuring that the longest path from root to leaf is at most twice the shortest path. Invented by Rudolf Bayer in 1972 (as symmetric binary B-trees) and refined by Guibas and Sedgewick in 1978.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Maintain a balanced BST that guarantees O(log n) operations with fewer rotations than AVL trees during modifications.

### 2.2 Mathematical Model

**Red-Black Properties** (Invariants):
1. Every node is either **red** or **black**
2. The **root** is always **black**
3. All **leaves** (NIL/null) are **black**
4. If a node is **red**, both its children are **black** (no consecutive reds)
5. Every path from any node to its descendant leaves contains the same number of **black nodes** (black-height)

**Black-Height**: The number of black nodes on any path from a node to a leaf (not counting the node itself).

**Height Bound**:
A Red-Black tree with $n$ internal nodes has height:
$$h \leq 2 \log_2(n+1)$$

**Proof**: Property 4 ensures at least half the nodes on any path are black. Property 5 ensures black-height is consistent. Combined: height is at most $2 \times \text{black-height}$.

### 2.3 Correctness Proof

**Theorem**: Red-Black properties can be maintained in O(log n) time per operation.

**Key Insight**: Violations only propagate upward, and at most O(log n) levels need adjustment. Each level requires O(1) work (recoloring or rotation).

## 3. Algorithm Description

### 3.1 Intuition

Red-Black trees use color to encode structural information that would otherwise require height tracking. The rules ensure that no path is more than twice as long as any other, providing "good enough" balance with fewer rotations than AVL.

### 3.2 Insertion Algorithm

After standard BST insertion (new node is always **red**):

**Case 1**: Uncle is red → Recolor
```
       G(B)                G(R)
      /   \               /   \
    P(R)  U(R)   →     P(B)  U(B)
    /                   /
  N(R)                N(R)
Then continue fixing at G
```

**Case 2**: Uncle is black, node is inner child → Rotate to Case 3
```
     G(B)               G(B)
    /   \              /   \
  P(R)  U(B)   →    N(R)  U(B)
    \               /
   N(R)           P(R)
```

**Case 3**: Uncle is black, node is outer child → Rotate and recolor
```
      G(B)              P(B)
     /   \             /   \
   P(R)  U(B)   →   N(R)  G(R)
   /                        \
 N(R)                       U(B)
```

### 3.3 Pseudocode

```
INSERT_FIXUP(tree, node):
    while node.parent is RED:
        if node.parent == node.grandparent.left:
            uncle = node.grandparent.right
            if uncle is RED:                    // Case 1
                node.parent.color = BLACK
                uncle.color = BLACK
                node.grandparent.color = RED
                node = node.grandparent
            else:
                if node == node.parent.right:   // Case 2
                    node = node.parent
                    LEFT_ROTATE(tree, node)
                node.parent.color = BLACK       // Case 3
                node.grandparent.color = RED
                RIGHT_ROTATE(tree, node.grandparent)
        else:
            // Symmetric cases for right subtree
            ...
    tree.root.color = BLACK

DELETE_FIXUP(tree, node):
    while node != tree.root and node.color == BLACK:
        if node == node.parent.left:
            sibling = node.parent.right
            // Handle 4 cases based on sibling and its children colors
            ...
        else:
            // Symmetric cases
            ...
    node.color = BLACK
```

### 3.4 Step-by-Step Example

Insert sequence: [7, 3, 18, 10, 22, 8, 11, 26]

```
Insert 7 (root):     Insert 3:         Insert 18:
    7(B)                7(B)              7(B)
                       /                 /   \
                     3(R)              3(R) 18(R)

Insert 10:           After fixup (Case 1):
    7(B)                   7(B)
   /   \                  /   \
 3(R) 18(R)             3(B) 18(B)
      /                      /
    10(R)                 10(R)

Insert 22:              Insert 8:
    7(B)                    7(B)
   /   \                   /   \
 3(B) 18(B)              3(B) 18(B)
      /  \                    /  \
   10(R) 22(R)            10(R) 22(R)
                          /
                        8(R)
                        ↑ Triggers fixup

After rotation:
       7(B)
      /   \
    3(B)  18(B)
         /   \
       10(B) 22(B)
       /
     8(R)
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Time Complexity |
|-----------|-----------------|
| Search    | O(log n)        |
| Insert    | O(log n)        |
| Delete    | O(log n)        |

**Rotation Analysis**:
- Insert: At most 2 rotations + O(log n) recolorings
- Delete: At most 3 rotations + O(log n) recolorings

### 4.2 Space Complexity

- **Storage**: O(n) for n nodes
- **Per Node Overhead**: 1 bit for color (often rounded to byte)
- **Auxiliary**: O(log n) for recursion or O(1) with parent pointers

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
#[derive(Copy, Clone)]
enum Color {
    Red,
    Black,
}

pub struct RBNode<K: Ord, V> {
    key: K,
    value: V,
    color: Color,
    parent: *mut RBNode<K, V>,
    left: *mut RBNode<K, V>,
    right: *mut RBNode<K, V>,
}
```

**Key Design Patterns**:
- Raw pointers (`*mut`) for parent links (necessary for efficient traversal)
- `unsafe` blocks required for pointer manipulation
- `Box::into_raw` / `Box::from_raw` for ownership transfer
- Null sentinel pattern for leaves

**Unsafe Operations**:
```rust
unsafe fn left_rotate<K: Ord, V>(tree: &mut RBTree<K, V>, x: *mut RBNode<K, V>) {
    let y = (*x).right;
    (*x).right = (*y).left;
    if !(*y).left.is_null() {
        (*(*y).left).parent = x;
    }
    (*y).parent = (*x).parent;
    // ... update parent's child pointer
    (*y).left = x;
    (*x).parent = y;
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Delete root | Ensure new root is black |
| Delete red leaf | Simply remove (no fixup needed) |
| Delete black leaf | Complex fixup required |
| All nodes same color | Invalid state, invariant broken |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Linux Kernel**: `rbtree` used extensively (scheduler, memory management)
2. **C++ STL**: `std::map` and `std::set` implementations
3. **Java**: `TreeMap` and `TreeSet`
4. **Nginx**: Timers and event management
5. **Filesystems**: ext3/4 extent trees

### 6.2 Related Algorithms

| Structure | Trade-off |
|-----------|-----------|
| **AVL Tree** | Stricter balance (1.44 log n height), more lookups |
| **2-3-4 Tree** | Red-Black is isomorphic to 2-3-4 trees |
| **B-Tree** | Generalization for disk storage |
| **Splay Tree** | Amortized O(log n), self-adjusting |

### Red-Black vs AVL Trade-offs

| Aspect | Red-Black | AVL |
|--------|-----------|-----|
| Height | ≤ 2 log n | ≤ 1.44 log n |
| Rotations/Insert | ≤ 2 | O(log n) |
| Rotations/Delete | ≤ 3 | O(log n) |
| Best for | Write-heavy | Read-heavy |
| Used in | std::map, Linux | Databases, lookups |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Null Pointer Dereferencing**: Leaves are null, must check
2. **Parent Pointer Updates**: Often forgotten during rotations
3. **Color Inheritance**: New nodes must be red
4. **Root Coloring**: Must always be black after operations

### 7.2 Optimization Opportunities

- **Null Sentinels**: Use single black null node instead of null checks
- **Color in Pointer**: Store color in low bit of pointer (alignment trick)
- **Left-Leaning Red-Black Trees**: Simplified variant by Sedgewick
- **Intrusive Containers**: Embed node in data to reduce indirection

### 7.3 Left-Leaning Red-Black Trees (LLRB)

Simplified variant where red links lean left:
- Only 3 cases instead of 6
- Easier to implement
- Slightly higher constant factors

## 8. References

- Guibas, L. J., & Sedgewick, R. (1978). "A dichromatic framework for balanced trees". *FOCS '78*.
- Cormen, T. H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Chapter 13.
- Sedgewick, R. (2008). "Left-leaning Red-Black Trees". *Dagstuhl Workshop on Data Structures*.
- Bayer, R. (1972). "Symmetric binary B-Trees: Data structure and maintenance algorithms". *Acta Informatica*.
