# Tree Sort

## 1. Overview

Tree Sort is a sorting algorithm that builds a Binary Search Tree (BST) from the input elements and then performs an in-order traversal to produce sorted output. It combines the benefits of BST structure with traversal properties.

### Key Characteristics
- **Type**: Comparison-based, tree-based sort
- **In-place**: No (requires O(n) space for tree)
- **Stable**: No (without modifications)
- **Online**: Yes (can process elements as they arrive)

## 2. Mathematical Foundation

### 2.1 BST Property

For any node N in a BST:
- All values in left subtree < N.value
- All values in right subtree > N.value

### 2.2 In-Order Traversal

In-order traversal visits: Left subtree → Node → Right subtree

This produces elements in sorted order due to BST property.

### 2.3 Tree Height Analysis

| Tree Type | Height | Sort Time |
|-----------|--------|-----------|
| Balanced | O(log n) | O(n log n) |
| Skewed (sorted input) | O(n) | O(n²) |

## 3. Algorithm Description

### 3.1 Pseudocode

```
TREE_SORT(A)
    root ← null
    
    // Build BST
    for each element x in A do
        root ← INSERT(root, x)
    
    // In-order traversal
    result ← []
    INORDER(root, result)
    return result

INSERT(node, value)
    if node = null then
        return new Node(value)
    if value < node.value then
        node.left ← INSERT(node.left, value)
    else
        node.right ← INSERT(node.right, value)
    return node

INORDER(node, result)
    if node ≠ null then
        INORDER(node.left, result)
        result.append(node.value)
        INORDER(node.right, result)
```

### 3.2 Step-by-Step Example

Sorting `[5, 3, 7, 1, 4, 6, 8]`:

```
Build BST:
Insert 5:       5
Insert 3:       5
               /
              3
Insert 7:       5
               / \
              3   7
Insert 1:       5
               / \
              3   7
             /
            1
Insert 4:       5
               / \
              3   7
             / \
            1   4
Insert 6:       5
               / \
              3   7
             / \ /
            1  4 6
Insert 8:       5
               / \
              3   7
             / \ / \
            1  4 6  8

In-order traversal: 1 → 3 → 4 → 5 → 6 → 7 → 8

Result: [1, 3, 4, 5, 6, 7, 8]
```

## 4. Complexity Analysis

| Case | Time | Condition |
|------|------|-----------|
| **Best** | O(n log n) | Balanced tree |
| **Average** | O(n log n) | Random input |
| **Worst** | O(n²) | Sorted/reverse sorted input |
| **Space** | O(n) | Tree storage |

### 4.1 Self-Balancing Variant

Using AVL or Red-Black tree guarantees O(n log n) time:

| Tree Type | Insert | Total |
|-----------|--------|-------|
| BST | O(n) worst | O(n²) worst |
| AVL | O(log n) | O(n log n) |
| Red-Black | O(log n) | O(n log n) |

## 5. Implementation

```rust
use std::cmp::Ordering;

struct Node<T> {
    value: T,
    left: Option<Box<Node<T>>>,
    right: Option<Box<Node<T>>>,
}

impl<T: Ord> Node<T> {
    fn new(value: T) -> Self {
        Node {
            value,
            left: None,
            right: None,
        }
    }

    fn insert(&mut self, value: T) {
        match value.cmp(&self.value) {
            Ordering::Less => {
                if let Some(ref mut left) = self.left {
                    left.insert(value);
                } else {
                    self.left = Some(Box::new(Node::new(value)));
                }
            }
            _ => {
                if let Some(ref mut right) = self.right {
                    right.insert(value);
                } else {
                    self.right = Some(Box::new(Node::new(value)));
                }
            }
        }
    }

    fn inorder(&self, result: &mut Vec<T>)
    where
        T: Clone,
    {
        if let Some(ref left) = self.left {
            left.inorder(result);
        }
        result.push(self.value.clone());
        if let Some(ref right) = self.right {
            right.inorder(result);
        }
    }
}

pub fn tree_sort<T: Ord + Clone>(arr: &mut [T]) {
    if arr.is_empty() {
        return;
    }

    // Build BST
    let mut root = Node::new(arr[0].clone());
    for item in arr.iter().skip(1) {
        root.insert(item.clone());
    }

    // In-order traversal
    let mut result = Vec::with_capacity(arr.len());
    root.inorder(&mut result);

    // Copy back
    arr.clone_from_slice(&result);
}
```

## 6. Advantages and Limitations

### Advantages
| Advantage | Description |
|-----------|-------------|
| Online | Can sort as elements arrive |
| Natural | Follows intuitive tree structure |
| Self-balancing option | Can guarantee O(n log n) |

### Limitations
| Limitation | Description |
|------------|-------------|
| O(n) space | Tree nodes require memory |
| O(n²) worst case | Without self-balancing |
| Not cache-friendly | Pointer chasing |
| Clone required | Elements need to be cloned |

## 7. Comparison with Heap Sort

| Aspect | Tree Sort | Heap Sort |
|--------|-----------|-----------|
| Data Structure | BST | Binary Heap |
| Worst Case | O(n²) or O(n log n) | O(n log n) |
| Space | O(n) | O(1) in-place |
| Stability | No | No |
| Online | Yes | Partial |

## 8. Applications

1. **Database indexing**: B-trees for sorted access
2. **File systems**: Directory organization
3. **Streaming data**: Online sorting
4. **When BST already exists**: Leverage existing structure

## 9. Variations

### 9.1 Self-Balancing Tree Sort
Use AVL or Red-Black tree for guaranteed O(n log n).

### 9.2 Ternary Tree Sort
Use a ternary tree (3-way comparison) for handling duplicates more elegantly.

## 10. References

1. Knuth, D. (1998). *The Art of Computer Programming, Vol. 3*.
2. Cormen, T. H. (2009). *Introduction to Algorithms*, Chapter 12.
3. Sedgewick, R. (2011). *Algorithms*, 4th Edition.

## 11. Source Code

**Implementation**: [src/sorting/tree_sort.rs](../../src/sorting/tree_sort.rs)
