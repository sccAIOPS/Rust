# Binary Search Tree

## 1. Overview

A Binary Search Tree (BST) is a node-based binary tree data structure where each node has at most two children (left and right), and for every node:
- All values in the left subtree are less than the node's value
- All values in the right subtree are greater than the node's value

BSTs were first introduced in the 1960s and remain fundamental to computer science education and practical applications.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a set $S$ of comparable elements, maintain a data structure that supports:
- **Search**: Determine if element $x \in S$
- **Insert**: Add element $x$ to $S$
- **Min/Max**: Find the smallest/largest element in $S$
- **Floor/Ceiling**: Find the largest element $\leq x$ / smallest element $\geq x$

### 2.2 Mathematical Model

**Input/Output Specifications**:
- Elements must implement the `Ord` trait (total ordering)
- Tree maintains the BST property at all times

**BST Property** (Invariant):
For any node $n$ with value $v$:
$$\forall x \in \text{left}(n): x < v$$
$$\forall y \in \text{right}(n): y > v$$

### 2.3 Correctness Proof

**Invariant**: The BST property is maintained after every operation.

**Proof for Insert**:
1. Base case: Empty tree → single node (trivially satisfies BST property)
2. Inductive step: Given a valid BST, inserting $x$:
   - If $x < \text{root}$, insert into left subtree (maintains ordering)
   - If $x \geq \text{root}$, insert into right subtree (maintains ordering)
   - By induction, subtrees remain valid BSTs

## 3. Algorithm Description

### 3.1 Intuition

Think of a BST as a decision tree where each node asks "Is the target less than, equal to, or greater than me?" and directs the search accordingly. This binary decision halves the search space at each step (in balanced trees).

### 3.2 Pseudocode

```
SEARCH(node, value):
    if node is NULL:
        return false
    if value == node.value:
        return true
    else if value < node.value:
        return SEARCH(node.left, value)
    else:
        return SEARCH(node.right, value)

INSERT(node, value):
    if node is NULL:
        return new Node(value)
    if value < node.value:
        node.left = INSERT(node.left, value)
    else:
        node.right = INSERT(node.right, value)
    return node

MINIMUM(node):
    if node.left is NULL:
        return node.value
    return MINIMUM(node.left)

MAXIMUM(node):
    if node.right is NULL:
        return node.value
    return MAXIMUM(node.right)
```

### 3.3 Step-by-Step Example

Inserting values [5, 3, 7, 1, 4, 6, 8]:

```
Step 1: Insert 5          Step 2: Insert 3          Step 3: Insert 7
       5                         5                         5
                                /                         / \
                               3                         3   7

Step 4-7: Insert 1, 4, 6, 8
           5
          / \
         3   7
        / \ / \
       1  4 6  8
```

Searching for 4:
1. Compare 4 with 5: 4 < 5, go left
2. Compare 4 with 3: 4 > 3, go right
3. Compare 4 with 4: Found!

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Best Case | Average Case | Worst Case |
|-----------|-----------|--------------|------------|
| Search    | O(1)      | O(log n)     | O(n)       |
| Insert    | O(1)      | O(log n)     | O(n)       |
| Min/Max   | O(1)      | O(log n)     | O(n)       |
| Floor/Ceil| O(1)      | O(log n)     | O(n)       |

**Worst Case**: Occurs when the tree degenerates into a linked list (e.g., inserting sorted data: 1, 2, 3, 4, 5).

```
1
 \
  2
   \
    3
     \
      4  ← Height = n, not log(n)
```

**Average Case Derivation**:
For a randomly built BST with $n$ nodes, the expected height is $O(\log n)$.
The expected number of comparisons for a search is approximately $2 \ln n \approx 1.39 \log_2 n$.

### 4.2 Space Complexity

- **Storage**: O(n) for n nodes
- **Auxiliary (recursive calls)**: O(h) where h is tree height
  - Best/Average: O(log n)
  - Worst: O(n)

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub struct BinarySearchTree<T: Ord> {
    value: Option<T>,
    left: Option<Box<BinarySearchTree<T>>>,
    right: Option<Box<BinarySearchTree<T>>>,
}
```

**Key Design Choices**:
- Uses `Option<Box<...>>` for optional child nodes (heap allocation)
- Generic over any type implementing `Ord` trait
- Value wrapped in `Option` to handle empty trees
- Implements iterator via in-order traversal stack

**Ownership Patterns**:
- `Box<T>` provides heap allocation with single ownership
- References returned for search operations (`&T`)
- `match` expressions for safe pattern matching on `Option`

### 5.2 Edge Cases

| Case | Behavior |
|------|----------|
| Empty tree | `search` returns `false`, `min`/`max` return `None` |
| Single node | All operations work on that one node |
| Duplicate values | Inserted to the right (≥ comparison) |
| Already present | Search returns `true`, insert adds duplicate |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Database Indexing**: BSTs form the basis for B-tree indexes used in databases
2. **Symbol Tables**: Compilers use BSTs for symbol table management
3. **File Systems**: Directory structures often use tree-based indexing
4. **Autocomplete Systems**: BSTs can store dictionaries for prefix matching
5. **Game Development**: BSTs for spatial partitioning (though specialized variants preferred)

### 6.2 Related Algorithms

| Structure | Use When |
|-----------|----------|
| **AVL Tree** | Need guaranteed O(log n) operations |
| **Red-Black Tree** | Balanced tree with less strict balancing |
| **B-Tree** | Disk-based storage, many children per node |
| **Hash Table** | O(1) average, no ordering needed |
| **Skip List** | Probabilistic, good for concurrent access |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Degenerate Trees**: Sorted input creates O(n) height
   - *Mitigation*: Use self-balancing trees (AVL, Red-Black)

2. **Stack Overflow on Deep Trees**: Recursive operations
   - *Mitigation*: Iterative implementation or tail recursion

3. **Memory Overhead**: Box per node adds allocation overhead
   - *Mitigation*: Arena allocation for many nodes

### 7.2 Optimization Opportunities

- **Iterative Search**: Avoid stack frames for deep trees
- **Parent Pointers**: Enable efficient successor/predecessor
- **Thread Safety**: Use `Arc<RwLock<...>>` for concurrent access

## 8. References

- Cormen, T. H., Leiserson, C. E., Rivest, R. L., & Stein, C. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Chapter 12.
- Knuth, D. E. (1997). *The Art of Computer Programming, Volume 3: Sorting and Searching* (2nd ed.). Addison-Wesley.
- Sedgewick, R., & Wayne, K. (2011). *Algorithms* (4th ed.). Addison-Wesley.
