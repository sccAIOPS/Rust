# Longest Increasing Subsequence (LIS)

## 1. Overview

The Longest Increasing Subsequence problem finds the longest subsequence of a given sequence where elements are in strictly increasing order. This implementation achieves **O(n log n)** time complexity using binary search optimization.

**File**: `src/dynamic_programming/longest_increasing_subsequence.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a sequence $A = (a_1, a_2, \ldots, a_n)$, find the longest subsequence $(a_{i_1}, a_{i_2}, \ldots, a_{i_k})$ where:
- $i_1 < i_2 < \ldots < i_k$ (indices are increasing)
- $a_{i_1} < a_{i_2} < \ldots < a_{i_k}$ (values are strictly increasing)

### 2.2 Mathematical Model

**Naive DP Approach** (O(n²)):
Let $dp[i]$ = length of LIS ending at index $i$.

$$dp[i] = 1 + \max_{j < i, a_j < a_i} dp[j]$$

**Optimized Approach** (O(n log n)):
Maintain an array `tails` where `tails[i]` is the smallest tail element of any increasing subsequence of length $i+1$.

## 3. Algorithm Description

### 3.1 Binary Search Optimization

Key insight: We can use binary search because `tails` is always sorted.

For each element:
1. If larger than all tails, append it
2. Otherwise, replace the first tail ≥ current element

### 3.2 Pseudocode

```
FUNCTION longest_increasing_subsequence(array)
    n ← length(array)
    IF n ≤ 1 THEN RETURN array
    
    // tails[i] = (smallest tail of LIS of length i+1, original index)
    tails ← []
    prev ← [0..n] // prev[i] = index of previous element in LIS ending at i
    
    FOR i ← 0 TO n-1 DO
        value ← array[i]
        IF tails is empty OR value > tails.last().value THEN
            prev[i] ← tails.last().index (if exists)
            tails.append((value, i))
        ELSE
            // Binary search for position to replace
            pos ← binary_search(tails, value)
            tails[pos] ← (value, i)
            prev[i] ← tails[pos-1].index (if pos > 0)
        END IF
    END FOR
    
    // Reconstruct LIS
    RETURN reconstruct(array, tails, prev)
END FUNCTION
```

### 3.3 Step-by-Step Example

**Input**: [10, 9, 2, 5, 3, 7, 101, 18]

| Step | Element | Tails Array | Action |
|------|---------|-------------|--------|
| 0 | 10 | [(10,0)] | Append |
| 1 | 9 | [(9,1)] | Replace 10 with 9 |
| 2 | 2 | [(2,2)] | Replace 9 with 2 |
| 3 | 5 | [(2,2), (5,3)] | Append |
| 4 | 3 | [(2,2), (3,4)] | Replace 5 with 3 |
| 5 | 7 | [(2,2), (3,4), (7,5)] | Append |
| 6 | 101 | [(2,2), (3,4), (7,5), (101,6)] | Append |
| 7 | 18 | [(2,2), (3,4), (7,5), (18,7)] | Replace 101 |

**LIS Length**: 4  
**LIS**: [2, 3, 7, 18]

### 3.4 Why Binary Search Works

The `tails` array maintains this invariant:
- `tails[i]` is the smallest possible last element of any LIS of length `i+1`
- Therefore, `tails` is always sorted
- We can use binary search to find where to insert/replace

## 4. Complexity Analysis

### 4.1 Time Complexity

| Approach | Complexity |
|----------|------------|
| Naive DP | O(n²) |
| Binary Search | O(n log n) |

**Derivation**: n elements × O(log n) binary search = O(n log n)

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Tails array | O(n) |
| Previous pointers | O(n) |
| **Total** | **O(n)** |

## 5. Implementation Notes

### 5.1 Rust-Specific Features

**Generic Implementation**:
```rust
pub fn longest_increasing_subsequence<T: Ord + Clone>(
    input_array: &[T]
) -> Vec<T>
```

**Binary Search on Tuples**:
```rust
increasing_sequence
    .binary_search(&(value.clone(), 0))
    .unwrap_or_else(|x| x)
```

### 5.2 Edge Cases

| Case | Result |
|------|--------|
| Empty array | [] |
| Single element | [that element] |
| All equal | [first element] |
| Strictly decreasing | [any single element] |
| Already sorted | The entire array |

### 5.3 Important Note

When multiple LIS exist, this implementation returns the **lexicographically first** one that was found.

## 6. Applications

1. **Box Stacking**: Stack boxes where each must be smaller than the one below
2. **Scheduling**: Finding maximum compatible jobs
3. **Patience Sorting**: Card game strategy analysis
4. **Stock Trading**: Finding longest period of rising prices

## 7. Variants

| Variant | Modification |
|---------|--------------|
| Non-strictly increasing | Use ≤ instead of < |
| Longest Decreasing | Negate values or reverse comparison |
| Number of LIS | Count all LIS of maximum length |
| K-increasing | Allow at most k decreases |

## 8. Visualization

```
Array:  10  9  2  5  3  7  101  18
        ○   ○  ●  ○  ●  ●   ○   ●
            
LIS:        2 → 3 → 7 → 18   (length 4)

Evolution of tails array:
Step 0: [10]
Step 1: [9]        (9 < 10, replace)
Step 2: [2]        (2 < 9, replace)
Step 3: [2, 5]     (5 > 2, append)
Step 4: [2, 3]     (3 < 5, replace)
Step 5: [2, 3, 7]  (7 > 3, append)
Step 6: [2, 3, 7, 101]  (append)
Step 7: [2, 3, 7, 18]   (18 < 101, replace)
```

## 9. References

1. [Wikipedia - Longest increasing subsequence](https://en.wikipedia.org/wiki/Longest_increasing_subsequence)
2. Fredman, M. L. (1975). On computing the length of longest increasing subsequences
3. [LeetCode Problem 300](https://leetcode.com/problems/longest-increasing-subsequence/)
