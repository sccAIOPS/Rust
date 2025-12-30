# Optimal Binary Search Tree (OBST)

## 1. Overview

The Optimal Binary Search Tree problem constructs a BST that minimizes the expected search cost given access probabilities for keys. This is a classic interval DP problem with applications in database indexing.

**File**: `src/dynamic_programming/optimal_bst.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- Keys $k_1 < k_2 < \ldots < k_n$
- Search probabilities $p_1, p_2, \ldots, p_n$ where $\sum p_i = 1$

Find: BST structure minimizing expected search cost:

$$E[\text{cost}] = \sum_{i=1}^{n} p_i \cdot (\text{depth}(k_i) + 1)$$

### 2.2 Recurrence

Let $cost[i][j]$ = minimum cost for BST containing keys $k_i, \ldots, k_j$

For each possible root $r$ in range $[i, j]$:

$$cost[i][j] = \min_{r=i}^{j} \left( cost[i][r-1] + cost[r+1][j] + \sum_{k=i}^{j} p_k \right)$$

The sum term accounts for all nodes going one level deeper when a root is chosen.

### 2.3 Base Case

- $cost[i][i-1] = 0$ (empty tree)
- $cost[i][i] = p_i$ (single node)

## 3. Algorithm Description

### 3.1 Pseudocode

```
FUNCTION optimal_bst(keys, probs)
    n ← len(keys)
    cost[0..n+1][0..n+1] ← 0
    sum[0..n+1][0..n+1] ← 0  // prefix sums for efficiency
    
    // Precompute prefix sums
    FOR i ← 1 TO n DO
        sum[i][i] ← probs[i]
        FOR j ← i+1 TO n DO
            sum[i][j] ← sum[i][j-1] + probs[j]
        END FOR
    END FOR
    
    // Single keys
    FOR i ← 1 TO n DO
        cost[i][i] ← probs[i]
    END FOR
    
    // Increasing lengths
    FOR len ← 2 TO n DO
        FOR i ← 1 TO n-len+1 DO
            j ← i + len - 1
            cost[i][j] ← ∞
            FOR r ← i TO j DO
                c ← cost[i][r-1] + cost[r+1][j] + sum[i][j]
                IF c < cost[i][j] THEN
                    cost[i][j] ← c
                END IF
            END FOR
        END FOR
    END FOR
    
    RETURN cost[1][n]
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input**: 
- Keys: [10, 20, 30]
- Probabilities: [0.3, 0.2, 0.5]

**Build prefix sums**:
```
sum[1][1] = 0.3
sum[1][2] = 0.5
sum[1][3] = 1.0
sum[2][2] = 0.2
sum[2][3] = 0.7
sum[3][3] = 0.5
```

**Single keys**:
```
cost[1][1] = 0.3
cost[2][2] = 0.2
cost[3][3] = 0.5
```

**Length 2**:
```
cost[1][2]: root=1: 0 + 0.2 + 0.5 = 0.7
            root=2: 0.3 + 0 + 0.5 = 0.8
            min = 0.7 (root=1)

cost[2][3]: root=2: 0 + 0.5 + 0.7 = 1.2
            root=3: 0.2 + 0 + 0.7 = 0.9
            min = 0.9 (root=3)
```

**Length 3**:
```
cost[1][3]: root=1: 0 + 0.9 + 1.0 = 1.9
            root=2: 0.3 + 0.5 + 1.0 = 1.8
            root=3: 0.7 + 0 + 1.0 = 1.7
            min = 1.7 (root=3)
```

**Result**: 1.7

**Optimal Tree**:
```
    30 (root, prob=0.5)
   /
  10 (prob=0.3)
   \
    20 (prob=0.2)
```

Expected cost: 0.5×1 + 0.3×2 + 0.2×3 = 1.7 ✓

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(n³)** - Three nested loops

### 4.2 Space Complexity
- **O(n²)** for cost and sum tables

### 4.3 Knuth's Optimization

Can reduce to O(n²) using monotonicity of optimal root:

```
root[i][j-1] ≤ root[i][j] ≤ root[i+1][j]
```

This limits the inner loop search range.

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn optimal_bst(keys: &[char], probs: &[f64]) -> f64 {
    let n = keys.len();
    if n == 0 { return 0.0; }
    
    let mut cost = vec![vec![0.0; n + 2]; n + 2];
    let mut sum = vec![vec![0.0; n + 2]; n + 2];
    
    // Prefix sums
    for i in 0..n {
        sum[i][i] = probs[i];
        for j in i + 1..n {
            sum[i][j] = sum[i][j - 1] + probs[j];
        }
    }
    
    // Single keys
    for i in 0..n {
        cost[i][i] = probs[i];
    }
    
    // Increasing lengths
    for len in 2..=n {
        for i in 0..=n - len {
            let j = i + len - 1;
            cost[i][j] = f64::MAX;
            
            for r in i..=j {
                let left = if r > i { cost[i][r - 1] } else { 0.0 };
                let right = if r < j { cost[r + 1][j] } else { 0.0 };
                let c = left + right + sum[i][j];
                
                if c < cost[i][j] {
                    cost[i][j] = c;
                }
            }
        }
    }
    
    cost[0][n - 1]
}
```

### 5.2 Edge Cases

| Case | Result |
|------|--------|
| Empty keys | 0.0 |
| Single key | probability of that key |
| Uniform probs | Balanced tree optimal |
| One high prob | That key near root |

## 6. Comparison: OBST vs Standard BST

| Aspect | Standard BST | Optimal BST |
|--------|--------------|-------------|
| Construction | O(n log n) | O(n³) |
| Search | O(log n) avg | O(1) to O(n) depending on probs |
| Best for | Unknown access patterns | Known, stable access patterns |

## 7. Applications

1. **Database Systems**: Index structures for frequently queried columns
2. **Compilers**: Symbol table organization
3. **Spell Checkers**: Dictionary lookup optimization
4. **Network Routing**: Routing table organization

## 8. With Dummy Keys (Full OBST)

Include probabilities for unsuccessful searches (between keys):

```
Keys:     k1   k2   k3
Dummies: d0  d1  d2  d3

d0 = search < k1
d1 = k1 < search < k2
d2 = k2 < search < k3
d3 = search > k3
```

Recurrence extends to include dummy key costs.

## 9. Reconstruction

Track root choices to build actual tree:

```rust
struct BSTNode {
    key: char,
    left: Option<Box<BSTNode>>,
    right: Option<Box<BSTNode>>,
}

fn build_tree(keys: &[char], root_table: &[Vec<usize>], 
              i: usize, j: usize) -> Option<Box<BSTNode>> {
    if i > j { return None; }
    let r = root_table[i][j];
    Some(Box::new(BSTNode {
        key: keys[r],
        left: build_tree(keys, root_table, i, r.saturating_sub(1)),
        right: build_tree(keys, root_table, r + 1, j),
    }))
}
```

## 10. Related Problems

| Problem | Description |
|---------|-------------|
| Matrix Chain | Optimal parenthesization |
| Huffman Coding | Optimal prefix codes (greedy) |
| Optimal Merge | Merge sorted lists optimally |

## 11. References

1. Knuth, D.E. (1971). "Optimum binary search trees"
2. Cormen et al. "Introduction to Algorithms" - Chapter 15
3. [Wikipedia - Optimal BST](https://en.wikipedia.org/wiki/Optimal_binary_search_tree)
