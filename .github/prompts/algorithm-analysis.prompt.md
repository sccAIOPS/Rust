# Algorithm Analysis Expertise

## Context
Use this prompt when performing deep analysis of algorithm implementations, complexity verification, or optimization recommendations.

## Complexity Analysis Framework

### Time Complexity Classes
| Class | Name | Example Operations |
|-------|------|-------------------|
| O(1) | Constant | Array access, hash lookup |
| O(log n) | Logarithmic | Binary search, balanced tree ops |
| O(n) | Linear | Linear search, single loop |
| O(n log n) | Linearithmic | Efficient sorts (merge, heap, quick avg) |
| O(n²) | Quadratic | Nested loops, simple sorts |
| O(n³) | Cubic | Matrix multiplication (naive) |
| O(2ⁿ) | Exponential | Subset enumeration, naive recursion |
| O(n!) | Factorial | Permutation generation |

### Amortized Analysis Patterns
```markdown
## Dynamic Array (Vec) Analysis

| Operation | Worst Case | Amortized |
|-----------|------------|-----------|
| push | O(n) | O(1) |
| pop | O(1) | O(1) |
| insert | O(n) | O(n) |
| remove | O(n) | O(n) |

Explanation: Push is O(1) amortized because resizing doubles capacity,
spreading the O(n) cost over n insertions.
```

### Recurrence Relations
| Recurrence | Solution | Example |
|------------|----------|---------|
| T(n) = T(n-1) + O(1) | O(n) | Linear recursion |
| T(n) = T(n-1) + O(n) | O(n²) | Selection sort |
| T(n) = 2T(n/2) + O(1) | O(n) | Tree traversal |
| T(n) = 2T(n/2) + O(n) | O(n log n) | Merge sort |
| T(n) = T(n/2) + O(1) | O(log n) | Binary search |
| T(n) = T(n/2) + O(n) | O(n) | Median finding |

## Space Complexity Patterns

### In-Place vs Out-of-Place
```markdown
## In-Place Algorithms (O(1) extra space)
- Heap sort
- Quick sort (O(log n) stack)
- Insertion sort
- Selection sort
- Bubble sort

## Out-of-Place Algorithms (O(n) extra space)
- Merge sort
- Counting sort
- Radix sort
- Bucket sort
```

### Stack Space in Recursion
```markdown
## Recursion Depth Analysis

| Algorithm | Depth | Stack Space |
|-----------|-------|-------------|
| Binary search | O(log n) | O(log n) |
| Quick sort (balanced) | O(log n) | O(log n) |
| Quick sort (worst) | O(n) | O(n) |
| DFS on tree | O(h) | O(h) |
| DFS on graph | O(V) | O(V) |
```

## Algorithm Comparison Matrix

### Sorting Algorithms
| Algorithm | Best | Average | Worst | Space | Stable |
|-----------|------|---------|-------|-------|--------|
| Quick Sort | O(n log n) | O(n log n) | O(n²) | O(log n) | No |
| Merge Sort | O(n log n) | O(n log n) | O(n log n) | O(n) | Yes |
| Heap Sort | O(n log n) | O(n log n) | O(n log n) | O(1) | No |
| Tim Sort | O(n) | O(n log n) | O(n log n) | O(n) | Yes |
| Insertion Sort | O(n) | O(n²) | O(n²) | O(1) | Yes |
| Counting Sort | O(n+k) | O(n+k) | O(n+k) | O(k) | Yes |
| Radix Sort | O(d(n+k)) | O(d(n+k)) | O(d(n+k)) | O(n+k) | Yes |

### Searching Algorithms
| Algorithm | Best | Average | Worst | Prerequisite |
|-----------|------|---------|-------|--------------|
| Linear Search | O(1) | O(n) | O(n) | None |
| Binary Search | O(1) | O(log n) | O(log n) | Sorted |
| Jump Search | O(1) | O(√n) | O(√n) | Sorted |
| Interpolation | O(1) | O(log log n) | O(n) | Sorted, uniform |
| Exponential | O(1) | O(log n) | O(log n) | Sorted |

### Tree Operations
| Structure | Search | Insert | Delete | Space |
|-----------|--------|--------|--------|-------|
| BST (balanced) | O(log n) | O(log n) | O(log n) | O(n) |
| BST (worst) | O(n) | O(n) | O(n) | O(n) |
| AVL Tree | O(log n) | O(log n) | O(log n) | O(n) |
| Red-Black | O(log n) | O(log n) | O(log n) | O(n) |
| B-Tree | O(log n) | O(log n) | O(log n) | O(n) |
| Trie | O(m) | O(m) | O(m) | O(ALPHABET×m×n) |

### Graph Algorithms
| Algorithm | Time | Space | Notes |
|-----------|------|-------|-------|
| BFS | O(V+E) | O(V) | Unweighted shortest path |
| DFS | O(V+E) | O(V) | Connectivity, cycles |
| Dijkstra | O((V+E)log V) | O(V) | Non-negative weights |
| Bellman-Ford | O(VE) | O(V) | Handles negative edges |
| Floyd-Warshall | O(V³) | O(V²) | All-pairs shortest path |
| Kruskal MST | O(E log E) | O(V) | Uses union-find |
| Prim MST | O((V+E)log V) | O(V) | Uses priority queue |

## Common Pitfalls Checklist

### Sorting Pitfalls
- [ ] Quick sort: O(n²) on sorted/nearly sorted input
- [ ] Quick sort: Stack overflow on deep recursion
- [ ] Merge sort: High memory usage for large arrays
- [ ] Counting sort: Large range causes memory issues
- [ ] Heap sort: Poor cache locality

### Search Pitfalls
- [ ] Binary search: Integer overflow in mid calculation
- [ ] Binary search: Off-by-one in bounds
- [ ] Interpolation search: Division by zero
- [ ] Jump search: Suboptimal block size

### Tree Pitfalls
- [ ] BST: Degenerates to O(n) on sorted insertion
- [ ] Recursive traversal: Stack overflow on deep trees
- [ ] AVL/RB: Complex rotation logic errors

### Graph Pitfalls
- [ ] DFS: Infinite loop without visited tracking
- [ ] BFS: Memory issues on wide graphs
- [ ] Dijkstra: Incorrect with negative weights
- [ ] Shortest path: Not handling disconnected components

## Optimization Strategies

### Level 1: Quick Wins
| Optimization | Benefit | Effort |
|--------------|---------|--------|
| Pre-allocate containers | Avoid reallocations | Low |
| Use iterators over indices | Better cache usage | Low |
| Remove unnecessary clones | Reduce allocations | Low |
| Early termination | Avoid unnecessary work | Low |

### Level 2: Algorithm Improvements
| Optimization | Benefit | Effort |
|--------------|---------|--------|
| Switch to better algorithm | Complexity reduction | Medium |
| Use appropriate data structure | Operation speedup | Medium |
| Add memoization | Avoid recomputation | Medium |
| Tail recursion | Prevent stack overflow | Medium |

### Level 3: Advanced Techniques
| Optimization | Benefit | Effort |
|--------------|---------|--------|
| Cache-oblivious algorithms | Better memory access | High |
| SIMD operations | Parallelism | High |
| Custom allocators | Reduced allocation overhead | High |
| Arena allocation | Better locality | High |

## Analysis Report Template

```markdown
# Algorithm Analysis: [Name]

## Summary
- **Current Complexity**: Time O(?), Space O(?)
- **Claimed Complexity**: [What documentation says]
- **Verified**: [Yes/No]
- **Issues Found**: [Count]

## Complexity Verification

### Time Complexity
| Operation | Code Location | Complexity | Evidence |
|-----------|--------------|------------|----------|
| Main loop | Line X | O(n) | Iterates over all elements |
| Inner operation | Line Y | O(1) | Constant-time hash lookup |
| **Total** | - | O(n) | Loop × inner |

### Space Complexity
| Allocation | Code Location | Size | Necessary |
|------------|--------------|------|-----------|
| Result vector | Line X | O(n) | Yes, output |
| Temp variable | Line Y | O(1) | Yes |
| **Total** | - | O(n) | - |

## Identified Issues

### Issue 1: [Title]
- **Severity**: High/Medium/Low
- **Location**: Line X
- **Problem**: [Description]
- **Impact**: [Performance/Correctness]
- **Solution**: [Recommendation]

## Recommendations

### Priority 1 (Must Fix)
1. [Recommendation]

### Priority 2 (Should Fix)
1. [Recommendation]

### Priority 3 (Nice to Have)
1. [Recommendation]

## Benchmarks (if applicable)

| Input Size | Current | Expected | Status |
|------------|---------|----------|--------|
| 100 | Xms | <1ms | ✅ |
| 1,000 | Xms | <10ms | ✅ |
| 10,000 | Xms | <100ms | ⚠️ |
```

## Edge Case Analysis Categories

### Numeric Edge Cases
- Zero
- One
- Maximum value (e.g., i32::MAX)
- Minimum value (e.g., i32::MIN)
- Negative numbers
- Overflow scenarios

### Collection Edge Cases
- Empty
- Single element
- Two elements
- All same elements
- Already sorted
- Reverse sorted
- Random order

### Graph Edge Cases
- Empty graph
- Single node
- Disconnected components
- Self-loops
- Cycles
- Complete graph
- Sparse vs dense

### String Edge Cases
- Empty string
- Single character
- Unicode
- Very long strings
- Repeated patterns
