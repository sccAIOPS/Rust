# Fractional Knapsack Problem

## 1. Overview

The Fractional Knapsack problem is a variant of the classic knapsack where items can be divided into fractions. Unlike the 0/1 Knapsack, this problem can be solved optimally using a greedy approach, making it significantly more efficient.

**File**: `src/dynamic_programming/fractional_knapsack.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- $n$ items with weights $w_1, w_2, \ldots, w_n$
- Values $v_1, v_2, \ldots, v_n$
- Knapsack capacity $W$

Find fractions $x = (x_1, x_2, \ldots, x_n)$ where $0 \leq x_i \leq 1$ that:

$$\text{Maximize: } \sum_{i=1}^{n} v_i \cdot x_i$$
$$\text{Subject to: } \sum_{i=1}^{n} w_i \cdot x_i \leq W$$

### 2.2 Greedy Choice Property

**Theorem**: The greedy choice of always selecting the item with the highest value-to-weight ratio leads to an optimal solution.

**Proof**: 
Let $r_i = v_i / w_i$ be the value density of item $i$. Consider an optimal solution $X^*$ that doesn't take the maximum possible amount of the highest-density item $j$.

We can construct a new solution $X'$ by:
1. Increasing $x_j$ by some amount $\delta$
2. Decreasing some other $x_k$ (where $r_k < r_j$) by $\delta \cdot (w_j / w_k)$

The new solution has value:
$$V' = V^* + \delta \cdot v_j - \delta \cdot (w_j / w_k) \cdot v_k = V^* + \delta \cdot w_j (r_j - r_k) > V^*$$

This contradicts the optimality of $X^*$. ∎

### 2.3 Value Density

The key insight is the **value density** (value-to-weight ratio):
$$\rho_i = \frac{v_i}{w_i}$$

Higher density items are more "efficient" in terms of value per unit weight.

## 3. Algorithm Description

### 3.1 Intuition

The greedy approach:
1. Calculate value-to-weight ratio for each item
2. Sort items by ratio in descending order
3. Take items greedily until capacity is exhausted
4. For the last item, take only the fraction that fits

### 3.2 Pseudocode

```
FUNCTION fractional_knapsack(capacity, weights, values)
    n ← length(weights)
    
    // Calculate ratios and pair with weights
    items ← []
    FOR i ← 0 TO n-1 DO
        items.append((weights[i], values[i] / weights[i]))
    END FOR
    
    // Sort by ratio descending
    SORT items BY ratio DESCENDING
    
    knapsack_value ← 0
    remaining_capacity ← capacity
    
    FOR (weight, ratio) IN items DO
        IF weight < remaining_capacity THEN
            // Take entire item
            remaining_capacity ← remaining_capacity - weight
            knapsack_value ← knapsack_value + weight × ratio
        ELSE
            // Take fraction of item
            knapsack_value ← knapsack_value + remaining_capacity × ratio
            BREAK
        END IF
    END FOR
    
    RETURN knapsack_value
END FUNCTION
```

### 3.3 Step-by-Step Example

**Input**:
- Capacity: 50
- Items: 
  - Item 1: weight=10, value=60 (ratio=6)
  - Item 2: weight=20, value=100 (ratio=5)
  - Item 3: weight=30, value=120 (ratio=4)

**Sorted by ratio**: Item 1 (6) → Item 2 (5) → Item 3 (4)

| Step | Item | Weight | Ratio | Remaining Cap | Action | Value Added |
|------|------|--------|-------|---------------|--------|-------------|
| 1 | 1 | 10 | 6 | 50 | Take all | +60 |
| 2 | 2 | 20 | 5 | 40 | Take all | +100 |
| 3 | 3 | 30 | 4 | 20 | Take 2/3 | +80 |

**Result**: Total value = 60 + 100 + 80 = **240**

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Complexity |
|-----------|------------|
| Calculate ratios | O(n) |
| Sort | O(n log n) |
| Greedy selection | O(n) |
| **Total** | **O(n log n)** |

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Ratio storage | O(n) |
| Sorting | O(log n) to O(n) |
| **Total** | **O(n)** |

### 4.3 Comparison with 0/1 Knapsack

| Aspect | Fractional | 0/1 |
|--------|------------|-----|
| Time | O(n log n) | O(nW) |
| Space | O(n) | O(nW) |
| Approach | Greedy | DP |
| Always Optimal? | Yes | Yes |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

**Floating Point Handling**:
```rust
weights.sort_unstable_by(|a, b| {
    b.1.partial_cmp(&a.1).expect("Encountered NaN")
});
```

**Debug Output** (present in implementation):
```rust
dbg!(&weights);
dbg!(&w.0, &knapsack_value);
```

⚠️ The current implementation includes `dbg!` macros for debugging.

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| All items fit | Take everything, no fractions |
| Single item | Take it entirely or partial |
| Zero capacity | Returns 0.0 |
| NaN values | Panics (by design) |

### 5.3 Precision Considerations

```rust
// Using f64 for precision
pub fn fractional_knapsack(
    mut capacity: f64,
    weights: Vec<f64>,
    values: Vec<f64>
) -> f64
```

For very precise applications, consider using a rational number library.

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Resource Scheduling**: Allocating CPU time fractions
2. **Memory Allocation**: Distributing memory among processes
3. **Bandwidth Allocation**: Network resource distribution
4. **Load Balancing**: Distributing workload across servers

### 6.2 Industry Applications

| Industry | Application |
|----------|-------------|
| Finance | Portfolio allocation |
| Logistics | Partial shipment loading |
| Agriculture | Mixing feed ingredients |
| Manufacturing | Alloy composition |

### 6.3 Related Problems

| Problem | Relationship |
|---------|--------------|
| 0/1 Knapsack | Binary selection (DP required) |
| Continuous Knapsack | Same problem, different name |
| Linear Programming | More general framework |

## 7. Why Greedy Works

### Visual Comparison

```
Fractional Knapsack (Greedy Works)
──────────────────────────────────
Items by value density:
Item A: ████████ (high density)
Item B: █████░░░ (medium density)  
Item C: ███░░░░░ (low density)

Solution: [████████][█████][██]
          Full A + Full B + Partial C

0/1 Knapsack (Greedy Fails)
───────────────────────────
Same items, but no splitting:
Item A fits entirely: ████████
Item B doesn't fit: [████████][XXXXX]
                    No room!
Better: Skip A, take B and C
```

### Mathematical Insight

The greedy solution is optimal because:
1. The objective function is linear
2. Constraints are linear
3. Items are divisible (relaxed integrality constraint)

This makes fractional knapsack a **linear program** with a simple solution.

## 8. Optimizations

### Early Termination

If total weight ≤ capacity, take everything:
```rust
let total_weight: f64 = weights.iter().sum();
if total_weight <= capacity {
    return values.iter().sum();
}
```

### Pre-computation for Multiple Queries

For multiple queries with same items:
```rust
// Pre-sort once
let sorted_items: Vec<(f64, f64)> = /* sorted by ratio */;

// Query function
fn query(capacity: f64, sorted_items: &[(f64, f64)]) -> f64 {
    // O(n) per query instead of O(n log n)
}
```

## 9. References

1. Dantzig, G. B. (1957). Discrete-Variable Extremum Problems
2. [Wikipedia - Continuous knapsack problem](https://en.wikipedia.org/wiki/Continuous_knapsack_problem)
3. Cormen, T. H. et al. (2009). *Introduction to Algorithms*, Chapter 16
