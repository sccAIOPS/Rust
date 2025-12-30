# 0/1 Knapsack Problem

## 1. Overview

The 0/1 Knapsack problem is a fundamental combinatorial optimization problem. Given a set of items with weights and values, determine the most valuable combination that fits within a weight capacity. Each item can only be taken once (hence "0/1" - either take it or don't).

**File**: `src/dynamic_programming/knapsack.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- $n$ items with weights $w_1, w_2, \ldots, w_n$
- Values $v_1, v_2, \ldots, v_n$
- Knapsack capacity $W$

Find a binary vector $x = (x_1, x_2, \ldots, x_n)$ where $x_i \in \{0, 1\}$ that:

$$\text{Maximize: } \sum_{i=1}^{n} v_i \cdot x_i$$
$$\text{Subject to: } \sum_{i=1}^{n} w_i \cdot x_i \leq W$$

### 2.2 Mathematical Model

**Optimal Substructure**: Let $K(i, w)$ be the maximum value achievable using items $1, 2, \ldots, i$ with capacity $w$.

**Recurrence Relation**:
$$K(i, w) = \begin{cases} 
0 & \text{if } i = 0 \text{ or } w = 0 \\
K(i-1, w) & \text{if } w_i > w \\
\max(K(i-1, w), v_i + K(i-1, w - w_i)) & \text{otherwise}
\end{cases}$$

### 2.3 Correctness Proof

**Theorem**: The DP solution produces an optimal solution.

**Proof by Strong Induction**:

*Base Case*: $K(0, w) = 0$ and $K(i, 0) = 0$ are trivially optimal.

*Inductive Step*: Assume $K(i-1, w')$ is optimal for all $w' \leq W$. For item $i$ with weight $w_i$:
- If $w_i > w$: Cannot include item $i$, so $K(i, w) = K(i-1, w)$ is optimal.
- If $w_i \leq w$: Either item $i$ is in optimal solution or not:
  - If not included: Optimal is $K(i-1, w)$
  - If included: Optimal is $v_i + K(i-1, w - w_i)$
  
Taking maximum covers both cases. ∎

## 3. Algorithm Description

### 3.1 Intuition

Build a 2D table where each cell $(i, w)$ stores the maximum value achievable:
- Using only the first $i$ items
- With exactly $w$ weight capacity remaining

For each item, decide whether including it yields a better result than excluding it.

### 3.2 Pseudocode

```
FUNCTION knapsack(capacity, items)
    n ← length(items)
    
    // Create DP table
    K[0..n][0..capacity] ← 0
    
    // Fill table
    FOR i ← 1 TO n DO
        FOR w ← 0 TO capacity DO
            IF items[i].weight > w THEN
                K[i][w] ← K[i-1][w]
            ELSE
                include ← items[i].value + K[i-1][w - items[i].weight]
                exclude ← K[i-1][w]
                K[i][w] ← MAX(include, exclude)
            END IF
        END FOR
    END FOR
    
    // Backtrack to find included items
    included ← []
    w ← capacity
    FOR i ← n DOWN TO 1 DO
        IF K[i][w] ≠ K[i-1][w] THEN
            included.append(i)
            w ← w - items[i].weight
        END IF
    END FOR
    
    RETURN (K[n][capacity], included)
END FUNCTION
```

### 3.3 Step-by-Step Example

**Input**:
- Capacity: 10
- Items: [(weight=6, value=7), (weight=4, value=9), (weight=5, value=10)]

**DP Table Construction**:

| Item\\Cap | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|-----------|---|---|---|---|---|---|---|---|---|---|-----|
| 0 (none)  | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0   |
| 1 (6,7)   | 0 | 0 | 0 | 0 | 0 | 0 | 7 | 7 | 7 | 7 | 7   |
| 2 (4,9)   | 0 | 0 | 0 | 0 | 9 | 9 | 9 | 9 | 9 | 9 | 16  |
| 3 (5,10)  | 0 | 0 | 0 | 0 | 9 | 10| 10| 10| 10| 19| 19  |

**Backtracking**:
- At K[3][10]=19: K[3][10] ≠ K[2][10]=16 → Include item 3
- At K[2][5]=9: K[2][5] ≠ K[1][5]=0 → Include item 2
- At K[1][1]=0: K[1][1] = K[0][1]=0 → Skip item 1

**Result**: Items 2 and 3 → Total value: 19, Total weight: 9

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Explanation |
|------|------------|-------------|
| Best | O(n × W) | Always fills entire table |
| Average | O(n × W) | Same as best |
| Worst | O(n × W) | Same as best |

Where $n$ is the number of items and $W$ is the capacity.

### 4.2 Space Complexity

| Component | Space | Notes |
|-----------|-------|-------|
| DP Table | O(n × W) | Full 2D table |
| Backtracking | O(n) | Recursion/result storage |
| **Total** | **O(n × W)** | Dominant factor |

**Space Optimization**: Can reduce to O(W) by using rolling array, but loses backtracking capability.

### 4.3 NP-Completeness

The knapsack problem is **NP-Complete** in general. The DP solution is **pseudo-polynomial** because:
- Runtime depends on $W$ (numeric value), not $\log W$ (input size)
- If $W$ is exponential in input size, runtime is exponential

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Item Structure**:
```rust
pub struct Item {
    weight: usize,
    value: usize,
}
```

**Solution Structure**:
```rust
pub struct KnapsackSolution {
    optimal_profit: usize,
    total_weight: usize,
    item_indices: Vec<usize>,  // 1-indexed
}
```

**Functional Table Generation**:
```rust
(0..=num_items).fold(
    vec![vec![0; capacity + 1]; num_items + 1],
    |mut matrix, item_index| {
        // Fill matrix...
        matrix
    },
)
```

### 5.2 Edge Cases

| Case | Behavior |
|------|----------|
| Empty items | Returns (0, 0, []) |
| Zero capacity | Returns (0, 0, []) |
| All items too heavy | Returns (0, 0, []) |
| Single item fits | Returns that item's value |
| Greedy fails | DP finds optimal solution |

**Greedy Doesn't Work Example**:
```rust
// Items: (weight=10, value=15), (weight=6, value=7), (weight=4, value=9)
// Capacity: 10
// Greedy by value/weight: Take items 2+3 → value=16
// Optimal: Take item 1 → value=15? NO!
// Actually optimal: Take items 2+3 → value=16 ✓
```

### 5.3 Index Convention

⚠️ **Note**: The implementation returns **1-indexed** item indices.

```rust
KnapsackSolution {
    item_indices: vec![1, 2, 3, 4, 6]  // 1-indexed!
}
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Resource Allocation**: Allocating CPU/memory to processes
2. **Project Selection**: Selecting projects under budget constraints
3. **Cutting Stock**: Optimizing material usage in manufacturing
4. **Cargo Loading**: Maximizing value of shipped goods

### 6.2 Industry Applications

| Industry | Application |
|----------|-------------|
| Finance | Portfolio optimization |
| Logistics | Vehicle loading |
| Cloud Computing | VM resource allocation |
| Bioinformatics | Gene selection |

### 6.3 Related Algorithms

| Problem | Difference |
|---------|------------|
| Fractional Knapsack | Items can be divided (greedy works) |
| Unbounded Knapsack | Unlimited copies of each item |
| Multi-dimensional Knapsack | Multiple capacity constraints |
| Subset Sum | All values = weights |

## 7. Variants Comparison

```
┌─────────────────────────────────────────────────────────────┐
│              Knapsack Problem Variants                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  0/1 Knapsack          Fractional           Unbounded        │
│  ┌───────────┐         ┌───────────┐        ┌───────────┐   │
│  │ ████ 1    │         │ ██▒▒ 0.5  │        │ ████ ∞    │   │
│  │ ████ 0    │         │ ████ 1.0  │        │ ████ ∞    │   │
│  │ ░░░░ 1    │         │ ▓▓░░ 0.75 │        │ ░░░░ ∞    │   │
│  └───────────┘         └───────────┘        └───────────┘   │
│  DP: O(nW)             Greedy: O(n log n)   DP: O(nW)       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## 8. Optimization Techniques

### Space Optimization (O(W))
```rust
let mut dp = vec![0; capacity + 1];
for item in items {
    for w in (item.weight..=capacity).rev() {
        dp[w] = dp[w].max(dp[w - item.weight] + item.value);
    }
}
```

### Meet-in-the-Middle (O(n × 2^(n/2)))
For small n with large W:
1. Split items into two halves
2. Enumerate all subsets of each half
3. Use binary search to combine

## 9. References

1. Martello, S., & Toth, P. (1990). *Knapsack Problems: Algorithms and Computer Implementations*
2. [Wikipedia - Knapsack problem](https://en.wikipedia.org/wiki/Knapsack_problem)
3. Kellerer, H., Pferschy, U., & Pisinger, D. (2004). *Knapsack Problems*
