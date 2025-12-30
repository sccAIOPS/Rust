# Data Structures

This directory contains comprehensive documentation for all data structure implementations in TheAlgorithms/Rust repository.

## Overview

Data structures are fundamental building blocks in computer science that organize and store data efficiently. This module contains implementations of classic and advanced data structures with various time-space tradeoffs.

## Categories

### Tree-Based Structures

| Structure | File | Description | Key Operations |
|-----------|------|-------------|----------------|
| [Binary Search Tree](binary_search_tree.md) | `binary_search_tree.rs` | Basic ordered tree | O(h) search/insert |
| [AVL Tree](avl_tree.md) | `avl_tree.rs` | Self-balancing BST with height tracking | O(log n) guaranteed |
| [Red-Black Tree](rb_tree.md) | `rb_tree.rs` | Self-balancing BST with color properties | O(log n) guaranteed |
| [B-Tree](b_tree.md) | `b_tree.rs` | Multi-way search tree | O(log n) disk-optimized |
| [Treap](treap.md) | `treap.rs` | Randomized BST with heap priorities | O(log n) expected |
| [Van Emde Boas Tree](veb_tree.md) | `veb_tree.rs` | Integer set with universe | O(log log U) operations |

### Prefix/Dictionary Structures

| Structure | File | Description | Key Operations |
|-----------|------|-------------|----------------|
| [Trie](trie.md) | `trie.rs` | Prefix tree for sequences | O(k) where k = key length |
| [Hash Table](hash_table.md) | `hash_table.rs` | Key-value store with hashing | O(1) average |

### Linear Structures

| Structure | File | Description | Key Operations |
|-----------|------|-------------|----------------|
| [Linked List](linked_list.md) | `linked_list.rs` | Doubly-linked list | O(1) insert at ends |
| [Queue](queue.md) | `queue.rs` | FIFO data structure | O(1) enqueue/dequeue |
| [Stack](stack.md) | `stack_using_singly_linked_list.rs` | LIFO data structure | O(1) push/pop |

### Heap Structures

| Structure | File | Description | Key Operations |
|-----------|------|-------------|----------------|
| [Heap](heap.md) | `heap.rs` | Binary heap (min/max) | O(log n) insert, O(1) peek |

### Range Query Structures

| Structure | File | Description | Key Operations |
|-----------|------|-------------|----------------|
| [Segment Tree](segment_tree.md) | `segment_tree.rs` | Range queries with point updates | O(log n) query/update |
| [Lazy Segment Tree](lazy_segment_tree.md) | `lazy_segment_tree.rs` | Range queries with range updates | O(log n) with lazy propagation |
| [Fenwick Tree](fenwick_tree.md) | `fenwick_tree.rs` | Binary Indexed Tree for prefix sums | O(log n) query/update |
| [Range Minimum Query](range_minimum_query.md) | `range_minimum_query.rs` | Sparse table for static RMQ | O(1) query after O(n log n) build |

### Union-Find Structures

| Structure | File | Description | Key Operations |
|-----------|------|-------------|----------------|
| [Union-Find](union_find.md) | `union_find.rs` | Disjoint Set Union | O(α(n)) ≈ O(1) amortized |

### Graph Structures

| Structure | File | Description | Key Operations |
|-----------|------|-------------|----------------|
| [Graph](graph.md) | `graph.rs` | Directed/Undirected graph | Adjacency list representation |

### Probabilistic Structures

| Structure | File | Description | Key Operations |
|-----------|------|-------------|----------------|
| [Skip List](skip_list.md) | `skip_list.rs` | Probabilistic balanced list | O(log n) expected |
| [Bloom Filter](bloom_filter.md) | `probabilistic/bloom_filter.rs` | Probabilistic set membership | O(k) with false positives |
| [Count-Min Sketch](count_min_sketch.md) | `probabilistic/count_min_sketch.rs` | Approximate frequency counting | O(1) with overestimation |

### Cycle Detection

| Algorithm | File | Description |
|-----------|------|-------------|
| [Floyd's Algorithm](floyds_algorithm.md) | `floyds_algorithm.rs` | Cycle detection in linked lists |

## Complexity Summary

### Time Complexity Comparison

| Operation | Array | Linked List | BST (avg) | AVL/RB | Hash Table |
|-----------|-------|-------------|-----------|--------|------------|
| Access | O(1) | O(n) | O(log n) | O(log n) | O(1) |
| Search | O(n) | O(n) | O(log n) | O(log n) | O(1) |
| Insert | O(n) | O(1)* | O(log n) | O(log n) | O(1) |
| Delete | O(n) | O(1)* | O(log n) | O(log n) | O(1) |

*At known position

### Space Complexity

| Structure | Space | Notes |
|-----------|-------|-------|
| BST/AVL/RB | O(n) | n nodes |
| B-Tree | O(n) | Better cache locality |
| Hash Table | O(n) | Load factor dependent |
| Segment Tree | O(n) | 2n or 4n array |
| Fenwick Tree | O(n) | n+1 array |
| Bloom Filter | O(m) | m bits, constant |
| Skip List | O(n) | Expected with levels |

## Usage Guidelines

### When to Use Each Structure

- **BST**: Simple ordered data, educational purposes
- **AVL Tree**: Frequent lookups, fewer modifications
- **Red-Black Tree**: Balanced read/write operations
- **B-Tree**: Disk-based storage, databases
- **Hash Table**: Fast key-value lookups, no ordering needed
- **Trie**: Prefix matching, autocomplete
- **Heap**: Priority queues, top-k problems
- **Segment Tree**: Range queries with updates
- **Fenwick Tree**: Prefix sums, simpler than segment tree
- **Union-Find**: Connected components, Kruskal's MST
- **Skip List**: Concurrent data structures
- **Bloom Filter**: Membership tests with space constraints

## Implementation Notes

All implementations in this module:
- Use Rust's ownership system effectively
- Provide generic type parameters where appropriate
- Include comprehensive test suites
- Follow the project's coding conventions

## References

- Cormen, T. H., et al. "Introduction to Algorithms" (CLRS)
- Sedgewick, R., & Wayne, K. "Algorithms"
- Knuth, D. E. "The Art of Computer Programming"
