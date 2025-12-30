# Maximum Subarray (Kadane's Algorithm)

## 1. Overview

The Maximum Subarray problem finds the contiguous subarray with the largest sum. This implementation uses Kadane's algorithm, achieving optimal **O(n)** time complexity.

**File**: `src/dynamic_programming/maximum_subarray.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[1..n]$, find indices $i, j$ (where $1 \leq i \leq j \leq n$) that maximize:
$$\sum_{k=i}^{j} A[k]$$

### 2.2 Kadane's Insight

At each position, we decide: extend the current subarray or start fresh.

$$dp[i] = \max(A[i], dp[i-1] + A[i])$$

The answer is $\max(dp[1], dp[2], \ldots, dp[n])$.

## 3. Algorithm Description

### 3.1 Pseudocode

```
FUNCTION maximum_subarray(array)
    IF array is empty THEN
        RETURN Error(EmptyArray)
    END IF
    
    cur_sum ← array[0]
    max_sum ← array[0]
    
    FOR i ← 1 TO n-1 DO
        cur_sum ← MAX(cur_sum + array[i], array[i])
        max_sum ← MAX(max_sum, cur_sum)
    END FOR
    
    RETURN Ok(max_sum)
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input**: [-2, 1, -3, 4, -1, 2, 1, -5, 4]

| Index | Value | cur_sum | max_sum | Action |
|-------|-------|---------|---------|--------|
| 0 | -2 | -2 | -2 | Start |
| 1 | 1 | 1 | 1 | Reset (1 > -2+1=-1) |
| 2 | -3 | -2 | 1 | Extend |
| 3 | 4 | 4 | 4 | Reset (4 > -2+4=2) |
| 4 | -1 | 3 | 4 | Extend |
| 5 | 2 | 5 | 5 | Extend |
| 6 | 1 | 6 | **6** | Extend |
| 7 | -5 | 1 | 6 | Extend |
| 8 | 4 | 5 | 6 | Extend |

**Result**: 6 (subarray [4, -1, 2, 1])

## 4. Complexity Analysis

- **Time**: O(n) - single pass
- **Space**: O(1) - only two variables

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn maximum_subarray(array: &[isize]) -> Result<isize, MaximumSubarrayError> {
    if array.is_empty() {
        return Err(MaximumSubarrayError::EmptyArray);
    }

    let mut cur_sum = array[0];
    let mut max_sum = cur_sum;

    for &x in &array[1..] {
        cur_sum = (cur_sum + x).max(x);
        max_sum = max_sum.max(cur_sum);
    }

    Ok(max_sum)
}
```

### 5.2 Edge Cases

| Case | Result |
|------|--------|
| Empty array | Error |
| All negative | Largest negative number |
| All positive | Sum of all elements |
| Single element | That element |

## 6. Applications

1. **Stock Trading**: Maximum profit from buy-sell
2. **Image Processing**: Maximum brightness region
3. **Genomics**: Gene expression analysis

## 7. Variants

| Variant | Modification |
|---------|--------------|
| Return indices | Track start/end positions |
| Circular array | Allow wrap-around |
| 2D maximum | Maximum sum rectangle |

## 8. Visualization

```
Array: [-2, 1, -3, 4, -1, 2, 1, -5, 4]
        ○  ○   ○  ●   ●  ●  ●   ○  ○
        
        └──────────────────────────┘
              Maximum Subarray
                [4, -1, 2, 1]
                 Sum = 6
```

## 9. References

1. Kadane, J. (1984). Maximum Sum Subarray Problem
2. [Wikipedia - Maximum subarray problem](https://en.wikipedia.org/wiki/Maximum_subarray_problem)
