# Prüfer Code (Prüfer Sequence)

## 1. Overview

Prüfer code is a bijective encoding of labeled trees as sequences of integers. Every labeled tree on $n$ vertices can be uniquely represented by a sequence of $n-2$ integers, each between $1$ and $n$. This provides a proof that there are exactly $n^{n-2}$ labeled trees on $n$ vertices (Cayley's formula).

The encoding was discovered by Heinz Prüfer in 1918.

## 2. Mathematical Foundation

### 2.1 Cayley's Formula

The number of distinct labeled trees on $n$ vertices is $n^{n-2}$.

This follows from the Prüfer code bijection:
- Each of the $n-2$ positions can have any of $n$ values
- Total: $n^{n-2}$ unique sequences

### 2.2 Bijection Property

```
Labeled Trees on n vertices  ←→  Sequences of length n-2 over {1,2,...,n}
```

## 3. Encoding Algorithm (Tree → Sequence)

### 3.1 Steps

1. Find the leaf with smallest label
2. Add the label of its neighbor to the sequence
3. Remove the leaf from the tree
4. Repeat until 2 nodes remain

### 3.2 Pseudocode

```
TREE-TO-PRUFER(adj_list, n):
    sequence = []
    degree = [count neighbors for each node]
    
    // Use min-heap or pointer for efficiency
    ptr = 0  // Smallest potential leaf
    
    for i = 0 to n-3:
        // Find smallest leaf
        while degree[ptr] != 1:
            ptr += 1
        
        leaf = ptr
        neighbor = find_neighbor(adj_list, leaf)
        
        sequence.append(neighbor)
        degree[leaf] = 0
        degree[neighbor] -= 1
        
        // Optimization: if neighbor becomes leaf and < ptr
        if degree[neighbor] == 1 and neighbor < ptr:
            sequence.append(find_neighbor(adj_list, neighbor))
            degree[neighbor] = 0
            // Update neighbor's neighbor degree
        else:
            ptr += 1
    
    return sequence
```

## 4. Decoding Algorithm (Sequence → Tree)

### 4.1 Key Insight

A vertex appears in the Prüfer sequence exactly (degree - 1) times.
Vertices not in sequence are leaves.

### 4.2 Steps

1. Count occurrences of each number in sequence
2. The missing numbers are leaves
3. Match smallest leaf with first sequence element
4. Remove leaf and first element, repeat

### 4.3 Pseudocode

```
PRUFER-TO-TREE(sequence, n):
    degree = [1, 1, ..., 1]  // All start as potential leaves
    
    for v in sequence:
        degree[v] += 1
    
    edges = []
    ptr = 0  // Smallest potential leaf
    
    for v in sequence:
        // Find smallest leaf
        while degree[ptr] != 1:
            ptr += 1
        
        leaf = ptr
        edges.append((leaf, v))
        
        degree[leaf] -= 1
        degree[v] -= 1
        
        if degree[v] == 1 and v < ptr:
            edges.append((v, find_next_sequence_element))
            // Similar optimization as encoding
        else:
            ptr += 1
    
    // Add final edge between two remaining nodes
    remaining = [i for i in 1..n if degree[i] == 1]
    edges.append((remaining[0], remaining[1]))
    
    return edges
```

## 5. Example

### 5.1 Encoding

```
Tree (n=6):
    1 — 2
    |   |
    3   4
   / \
  5   6

Adjacency: 1:[2,3], 2:[1,4], 3:[1,5,6], 4:[2], 5:[3], 6:[3]
Degrees: 1→2, 2→2, 3→3, 4→1, 5→1, 6→1

Step 1: Smallest leaf = 4, neighbor = 2 → seq = [2]
        Remove 4, degrees: 2→1

Step 2: Smallest leaf = 2, neighbor = 1 → seq = [2,1]
        Remove 2, degrees: 1→1

Step 3: Smallest leaf = 1, neighbor = 3 → seq = [2,1,3]
        Remove 1, degrees: 3→2

Step 4: Smallest leaf = 5, neighbor = 3 → seq = [2,1,3,3]
        Remove 5, degrees: 3→1

Remaining: 3, 6

Prüfer sequence: [2, 1, 3, 3]
```

### 5.2 Decoding

```
Sequence: [2, 1, 3, 3], n = 6

Count degrees: 1→2, 2→2, 3→3, 4→1, 5→1, 6→1
Initial leaves (degree=1): 4, 5, 6

Step 1: Smallest leaf = 4, first = 2 → edge (4,2), remove 4
Step 2: Smallest leaf = 2, first = 1 → edge (2,1), remove 2  
Step 3: Smallest leaf = 1, first = 3 → edge (1,3), remove 1
Step 4: Smallest leaf = 5, first = 3 → edge (5,3), remove 5
Final:  Remaining = 3, 6 → edge (3,6)

Edges: (4,2), (2,1), (1,3), (5,3), (3,6)
```

## 6. Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Encoding | O(n) | O(n) |
| Decoding | O(n) | O(n) |

With proper optimization (pointer technique), both achieve linear time.

## 7. Implementation

```rust
pub fn tree_to_prufer(adj: &[Vec<usize>]) -> Vec<usize> {
    let n = adj.len();
    if n <= 2 {
        return vec![];
    }

    let mut degree: Vec<usize> = adj.iter().map(|neighbors| neighbors.len()).collect();
    let mut removed = vec![false; n];
    let mut sequence = Vec::with_capacity(n - 2);
    let mut ptr = 0;

    for _ in 0..n - 2 {
        // Find smallest leaf
        while ptr < n && (removed[ptr] || degree[ptr] != 1) {
            ptr += 1;
        }

        let leaf = ptr;
        
        // Find neighbor of leaf
        let neighbor = adj[leaf]
            .iter()
            .find(|&&u| !removed[u])
            .copied()
            .unwrap();

        sequence.push(neighbor);
        removed[leaf] = true;
        degree[neighbor] -= 1;

        // Optimization: check if neighbor becomes valid leaf
        if degree[neighbor] == 1 && neighbor < ptr {
            let next_neighbor = adj[neighbor]
                .iter()
                .find(|&&u| !removed[u])
                .copied()
                .unwrap();
            sequence.push(next_neighbor);
            removed[neighbor] = true;
            degree[next_neighbor] -= 1;
        }
    }

    sequence
}

pub fn prufer_to_tree(sequence: &[usize], n: usize) -> Vec<(usize, usize)> {
    if n <= 1 {
        return vec![];
    }
    if n == 2 {
        return vec![(0, 1)];
    }

    let mut degree = vec![1usize; n];
    for &v in sequence {
        degree[v] += 1;
    }

    let mut edges = Vec::with_capacity(n - 1);
    let mut ptr = 0;
    let mut seq_idx = 0;

    for _ in 0..sequence.len() {
        // Find smallest leaf
        while degree[ptr] != 1 {
            ptr += 1;
        }

        let leaf = ptr;
        let v = sequence[seq_idx];
        seq_idx += 1;

        edges.push((leaf, v));
        degree[leaf] -= 1;
        degree[v] -= 1;

        // Optimization
        if degree[v] == 1 && v < ptr {
            if seq_idx < sequence.len() {
                let next_v = sequence[seq_idx];
                seq_idx += 1;
                edges.push((v, next_v));
                degree[v] -= 1;
                degree[next_v] -= 1;
            }
        }

        ptr += 1;
    }

    // Add final edge
    let remaining: Vec<_> = (0..n).filter(|&i| degree[i] == 1).collect();
    if remaining.len() == 2 {
        edges.push((remaining[0], remaining[1]));
    }

    edges
}
```

## 8. Applications

1. **Counting trees:** Proving Cayley's formula
2. **Random tree generation:** Generate random Prüfer code, decode to tree
3. **Tree enumeration:** Systematic enumeration of all labeled trees
4. **Network design:** Spanning tree enumeration
5. **Combinatorics:** Bijective proofs involving trees

## 9. Properties of Prüfer Sequences

| Property | Description |
|----------|-------------|
| Length | Always n-2 |
| Elements | Each in range [0, n-1] or [1, n] |
| Leaf detection | Missing elements are leaves |
| Degree formula | degree(v) = count(v in seq) + 1 |

## 10. Generating Random Trees

```rust
use rand::Rng;

pub fn random_labeled_tree(n: usize) -> Vec<(usize, usize)> {
    let mut rng = rand::thread_rng();
    let sequence: Vec<usize> = (0..n.saturating_sub(2))
        .map(|_| rng.gen_range(0..n))
        .collect();
    prufer_to_tree(&sequence, n)
}
```

## 11. Edge Cases

| Case | Sequence | Edges |
|------|----------|-------|
| n = 1 | [] | [] |
| n = 2 | [] | [(0,1)] |
| Star graph | [center, center, ...] | All connected to center |
| Path | Various | Linear chain |

## 12. Common Pitfalls

1. **0 vs 1 indexing:** Be consistent with vertex labels
2. **Degree initialization:** Leaves have degree 1 initially
3. **Final edge:** Don't forget to add edge between last two nodes
4. **Pointer optimization:** Essential for O(n) complexity

## 13. References

- Prüfer, H. (1918). "Neuer Beweis eines Satzes über Permutationen"
- Cayley, A. (1889). "A theorem on trees"
- Stanley, R. P. "Enumerative Combinatorics"
