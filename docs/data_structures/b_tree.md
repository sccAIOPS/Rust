# B-Tree

## 1. Overview

A B-Tree is a self-balancing tree data structure that maintains sorted data and allows searches, sequential access, insertions, and deletions in logarithmic time. Unlike binary trees, B-Tree nodes can have more than two children, making it ideal for storage systems that read and write large blocks of data (disks, databases).

Invented by Rudolf Bayer and Edward McCreight at Boeing Research Labs in 1970.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Design a balanced search tree optimized for systems where:
- Data access is in blocks (e.g., disk pages)
- Minimizing the number of disk accesses is critical
- Tree height should be kept minimal

### 2.2 Mathematical Model

**B-Tree of Order m** (or minimum degree t):

For a B-Tree of minimum degree $t \geq 2$:
1. Every node has at most $2t - 1$ keys
2. Every node (except root) has at least $t - 1$ keys
3. Root has at least 1 key (if non-empty)
4. All leaves appear at the same depth
5. A non-leaf node with $k$ keys has $k + 1$ children

**Height Bound**:
$$h \leq \log_t \frac{n+1}{2}$$

For $n$ keys and minimum degree $t$.

### 2.3 Key Properties

- **Branching Factor**: High branching factor (2t) reduces tree height
- **Disk Optimization**: Node size typically matches disk block size
- **Sequential Access**: In-order traversal visits keys in sorted order

## 3. Algorithm Description

### 3.1 Intuition

B-Trees are like "wide" BSTs. Instead of making one comparison per level, you make up to $2t-1$ comparisons but visit far fewer levels. For disk I/O, one node access = one disk read, so fewer levels = fewer disk reads.

### 3.2 Node Structure

```
[k₁ | k₂ | ... | kₙ]
 /    |       |    \
c₀   c₁     ...    cₙ
```
- Keys are sorted: $k_1 < k_2 < ... < k_n$
- Child $c_i$ contains keys in range $(k_i, k_{i+1})$

### 3.3 Pseudocode

```
SEARCH(node, key):
    i = 0
    while i < node.num_keys and key > node.keys[i]:
        i = i + 1
    if i < node.num_keys and key == node.keys[i]:
        return (node, i)
    if node.is_leaf:
        return NOT_FOUND
    return SEARCH(node.children[i], key)

INSERT(tree, key):
    if tree.root is full:
        new_root = allocate_node()
        new_root.children[0] = tree.root
        SPLIT_CHILD(new_root, 0)
        tree.root = new_root
    INSERT_NON_FULL(tree.root, key)

INSERT_NON_FULL(node, key):
    i = node.num_keys - 1
    if node.is_leaf:
        // Insert key in sorted position
        while i >= 0 and key < node.keys[i]:
            node.keys[i+1] = node.keys[i]
            i = i - 1
        node.keys[i+1] = key
        node.num_keys++
    else:
        // Find child to recurse into
        while i >= 0 and key < node.keys[i]:
            i = i - 1
        i = i + 1
        if node.children[i] is full:
            SPLIT_CHILD(node, i)
            if key > node.keys[i]:
                i = i + 1
        INSERT_NON_FULL(node.children[i], key)

SPLIT_CHILD(parent, index):
    full_child = parent.children[index]
    new_child = allocate_node()
    mid = t - 1
    
    // Move right half of keys to new child
    for j = 0 to t-2:
        new_child.keys[j] = full_child.keys[mid+1+j]
    
    // Move right half of children (if not leaf)
    if not full_child.is_leaf:
        for j = 0 to t-1:
            new_child.children[j] = full_child.children[mid+1+j]
    
    // Insert middle key into parent
    for j = parent.num_keys downto index+1:
        parent.keys[j] = parent.keys[j-1]
    parent.keys[index] = full_child.keys[mid]
    
    // Link new child to parent
    for j = parent.num_keys+1 downto index+2:
        parent.children[j] = parent.children[j-1]
    parent.children[index+1] = new_child
```

### 3.4 Step-by-Step Example

B-Tree with t=2 (2-3-4 tree), inserting: [10, 20, 30, 5, 6, 7, 15]

```
Insert 10:        Insert 20:        Insert 30 (split!):
  [10]             [10|20]            [20]
                                     /    \
                                  [10]   [30]

Insert 5:         Insert 6:         Insert 7 (split!):
    [20]              [20]               [6|20]
   /    \            /    \             /  |   \
 [5|10] [30]      [5|6|10] [30]      [5] [10] [30]

Insert 15:
      [6|20]
     /  |   \
   [5] [10|15] [30]
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Comparisons | Disk Accesses |
|-----------|-------------|---------------|
| Search    | O(t log_t n) | O(log_t n)    |
| Insert    | O(t log_t n) | O(log_t n)    |
| Delete    | O(t log_t n) | O(log_t n)    |

**Key Insight**: While comparisons per node are O(t), the number of nodes visited is O(log_t n), which is much smaller than O(log_2 n) for large t.

### 4.2 Space Complexity

- **Storage**: O(n) for keys
- **Node Overhead**: O(t) per node for pointers
- **Total Nodes**: O(n/t) to O(n/(t-1))

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
struct Node<T> {
    keys: Vec<T>,
    children: Vec<Node<T>>,
}

pub struct BTree<T> {
    root: Node<T>,
    props: BTreeProps,
}

struct BTreeProps {
    degree: usize,
    max_keys: usize,
    mid_key_index: usize,
}
```

**Key Design Patterns**:
- `Vec` for dynamic key/children storage
- Separate `BTreeProps` struct to avoid borrow checker issues
- Generic over `T: Ord + Copy + Default`
- Binary search within nodes for efficiency

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty tree | Search returns false, insert creates root |
| Root is full | Split root, increase tree height |
| Leaf is full | Split before inserting (proactive) |
| Minimum keys | Borrow/merge during deletion |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Databases**: MySQL InnoDB, PostgreSQL indexes
2. **Filesystems**: NTFS, HFS+, ext4 (B+ tree variant)
3. **Key-Value Stores**: LMDB, BerkeleyDB
4. **Search Engines**: Lucene segment indexes

### 6.2 Variants

| Variant | Modification |
|---------|--------------|
| **B+ Tree** | Keys only in leaves, leaves linked |
| **B* Tree** | Minimum 2/3 full, redistribute before split |
| **B-link Tree** | Concurrency-optimized with sibling pointers |
| **Counted B-Tree** | Stores subtree sizes for rank queries |

### B-Tree vs B+ Tree

| Aspect | B-Tree | B+ Tree |
|--------|--------|---------|
| Keys in internal nodes | Yes | No (only in leaves) |
| Leaf linkage | No | Yes (linked list) |
| Range queries | Slower | Faster |
| Point queries | Same | Same |
| Space | Slightly less | Slightly more |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Off-by-One Errors**: Indices for keys vs children differ
2. **Split Timing**: Must split before recursive insert (proactive splitting)
3. **Minimum Degree**: t ≥ 2 required for properties to hold
4. **Memory Alignment**: Node size should match disk block

### 7.2 Optimization Opportunities

- **Bulk Loading**: Build tree bottom-up for sorted input
- **Prefix Compression**: Store key prefixes in internal nodes
- **Cache-Oblivious B-Trees**: Better CPU cache utilization
- **Write-Optimized B-Trees**: LSM-tree hybrid approaches

## 8. References

- Bayer, R., & McCreight, E. (1970). "Organization and maintenance of large ordered indices". *Boeing Scientific Research Laboratories*.
- Comer, D. (1979). "The Ubiquitous B-Tree". *ACM Computing Surveys*.
- Cormen, T. H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Chapter 18.
- Graefe, G. (2011). "Modern B-Tree Techniques". *Foundations and Trends in Databases*.
