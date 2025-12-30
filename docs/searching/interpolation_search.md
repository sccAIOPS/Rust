# Interpolation Search

## 1. Overview

Interpolation Search is an improved variant of binary search designed for uniformly distributed sorted data. Instead of always dividing the search space in half, it estimates the position of the target based on its value relative to the boundary values—similar to how humans search in a phone book by guessing the approximate location of a name.

Published by W. W. Peterson in 1957, this algorithm achieves $O(\log \log n)$ average-case complexity for uniformly distributed data, making it significantly faster than binary search in such scenarios.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a sorted array $A[0..n-1]$ of integers and target $x$, find index $i$ such that $A[i] = x$, using value-based position estimation.

### 2.2 Mathematical Model

**Position Estimation Formula:**

Assuming linear interpolation between boundary values:

$$\text{pos} = \text{low} + \frac{(x - A[\text{low}]) \times (\text{high} - \text{low})}{A[\text{high}] - A[\text{low}]}$$

**Intuition:** If values are uniformly distributed, the target's position should be proportional to its value relative to the range.

**Example:** Array `[10, 20, 30, 40, 50]`, searching for `40`:
- low = 0, high = 4
- $\text{pos} = 0 + \frac{(40 - 10) \times (4 - 0)}{50 - 10} = 0 + \frac{30 \times 4}{40} = 3$ ✓

### 2.3 Complexity Analysis

**For uniformly distributed data:**
- Each probe reduces search space by a factor proportional to $\sqrt{n}$
- Expected probes: $O(\log \log n)$

**For non-uniform data:**
- Worst case degrades to $O(n)$ (e.g., exponentially distributed values)

### 2.4 Correctness Proof

The algorithm maintains the invariant that if $x$ exists in the array, then $x \in A[\text{low}..\text{high}]$.

**Early termination:** The condition `*item < nums[low] || *item > nums[high]` ensures we don't search outside valid bounds.

## 3. Algorithm Description

### 3.1 Intuition

Imagine searching for "Smith" in a phone book:
- You wouldn't open to the middle (like binary search)
- You'd estimate that "S" is about 70-80% through the alphabet
- Open to approximately that position
- Adjust based on what you find

### 3.2 Pseudocode

```
INTERPOLATION-SEARCH(A, x):
    low ← 0
    high ← n - 1
    
    while low ≤ high:
        if x < A[low] OR x > A[high]:
            return NOT_FOUND
        
        // Calculate probe position
        offset ← low + ((high - low) / (A[high] - A[low])) × (x - A[low])
        
        if A[offset] = x:
            return offset
        else if A[offset] < x:
            low ← offset + 1
        else:
            high ← offset - 1
    
    return NOT_FOUND
```

### 3.3 Step-by-Step Example

**Array:** `[10, 20, 30, 40, 50, 60, 70, 80, 90, 100]` (n = 10)  
**Target:** `70`

| Step | low | high | A[low] | A[high] | Calculated pos | A[pos] | Action |
|------|-----|------|--------|---------|----------------|--------|--------|
| 1 | 0 | 9 | 10 | 100 | $0 + \frac{(70-10)(9-0)}{100-10} = 6$ | 70 | **Found!** |

**Target:** `25`

| Step | low | high | A[low] | A[high] | Calculated pos | A[pos] | Action |
|------|-----|------|--------|---------|----------------|--------|--------|
| 1 | 0 | 9 | 10 | 100 | $0 + \frac{(25-10)(9-0)}{100-10} = 1$ | 20 | 20 < 25, low=2 |
| 2 | 2 | 9 | 30 | 100 | $2 + \frac{(25-30)(7)}{70}$ | - | 25 < 30, exit loop |
| - | - | - | - | - | - | - | **Not found** |

**Comparison:**
- Binary search: 3-4 comparisons
- Interpolation search: 1 comparison for found case!

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Condition |
|------|------------|-----------|
| Best | $O(1)$ | Target at estimated position |
| Average | $O(\log \log n)$ | Uniformly distributed data |
| Worst | $O(n)$ | Highly skewed distribution |

**Average Case Derivation:**

For uniform distribution, after one probe:
- Expected remaining search space: $O(\sqrt{n})$
- After $k$ probes: $n^{1/2^k}$
- Terminates when $n^{1/2^k} = 1$, i.e., $k = \log_2 \log_2 n$

### 4.2 Space Complexity

| Aspect | Complexity |
|--------|------------|
| Auxiliary | $O(1)$ |
| Total | $O(1)$ |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn interpolation_search<Ordering>(nums: &[i32], item: &i32) -> Result<usize, usize> {
    if nums.is_empty() {
        return Err(0);
    }
    let mut low: usize = 0;
    let mut high: usize = nums.len() - 1;
    
    while low <= high {
        if *item < nums[low] || *item > nums[high] {
            break;
        }
        let offset: usize = low
            + (((high - low) / (nums[high] - nums[low]) as usize) 
               * (*item - nums[low]) as usize);
        match nums[offset].cmp(item) {
            std::cmp::Ordering::Equal => return Ok(offset),
            std::cmp::Ordering::Less => low = offset + 1,
            std::cmp::Ordering::Greater => high = offset - 1,
        }
    }
    Err(0)
}
```

**Implementation Issues:**

1. **Type restriction:** Currently only works with `i32`, not generic
2. **Division ordering:** `(high - low) / (nums[high] - nums[low])` may cause integer division truncation issues
3. **Phantom type parameter:** `<Ordering>` is unused (likely a mistake)
4. **Return type:** Uses `Result<usize, usize>` instead of `Option<usize>`

**Improved Version:**
```rust
pub fn interpolation_search_improved(nums: &[i32], item: &i32) -> Option<usize> {
    if nums.is_empty() {
        return None;
    }
    let mut low = 0usize;
    let mut high = nums.len() - 1;
    
    while low <= high && *item >= nums[low] && *item <= nums[high] {
        if low == high {
            return if nums[low] == *item { Some(low) } else { None };
        }
        
        // Use larger type to prevent overflow
        let pos = low + ((*item - nums[low]) as usize * (high - low)) 
                      / ((nums[high] - nums[low]) as usize);
        
        match nums[pos].cmp(item) {
            std::cmp::Ordering::Equal => return Some(pos),
            std::cmp::Ordering::Less => low = pos + 1,
            std::cmp::Ordering::Greater => high = pos.saturating_sub(1),
        }
    }
    None
}
```

### 5.2 Edge Cases

| Case | Current Behavior | Expected |
|------|------------------|----------|
| Empty array | Returns `Err(0)` | ✓ |
| Single element (found) | Works correctly | ✓ |
| Target out of range | Exits via bounds check | ✓ |
| All elements equal | Division by zero! | ✗ Bug |
| Target at boundaries | Works correctly | ✓ |

**Critical Bug:** When `nums[high] == nums[low]`, division by zero occurs.

### 5.3 Numerical Stability

**Overflow Risk:**
```rust
// Risky if (high - low) and (item - nums[low]) are large
((high - low) / (nums[high] - nums[low])) * (item - nums[low])
```

**Safer Calculation:**
```rust
// Reorder to prevent overflow
(item - nums[low]) * (high - low) / (nums[high] - nums[low])
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Phone Book Search:** Names are roughly uniformly distributed
2. **Dictionary Lookup:** Word frequencies follow predictable patterns
3. **Log File Analysis:** Timestamp-based searches in sequential logs
4. **Database Indexing:** When key distribution is known
5. **Large Sorted Datasets:** When $O(\log \log n)$ matters

### 6.2 When to Use Interpolation Search

| Scenario | Recommendation |
|----------|---------------|
| Uniform distribution, large n | ✓ Interpolation Search |
| Unknown distribution | Binary Search |
| Small arrays (<100) | Binary Search |
| Non-numeric keys | Binary Search |
| Guaranteed O(log n) needed | Binary Search |

### 6.3 Distribution Impact

| Distribution | Expected Complexity | Notes |
|--------------|---------------------|-------|
| Uniform | $O(\log \log n)$ | Optimal case |
| Normal | ~$O(\log \log n)$ | Near-optimal for middle values |
| Exponential | $O(n)$ | Degrades badly |
| Clustered | $O(n)$ | Many probes needed |

## 7. Variants

### 7.1 Quadratic Interpolation

Uses quadratic fit instead of linear:

$$\text{pos} = \text{low} + \frac{(x - A[\text{low}])^2 \times (\text{high} - \text{low})}{(A[\text{high}] - A[\text{low}])^2}$$

### 7.2 Interpolation-Binary Hybrid

Falls back to binary search after a few failed interpolation probes:

```rust
const MAX_INTERPOLATION_PROBES: usize = 3;
let mut probe_count = 0;

while low <= high {
    let pos = if probe_count < MAX_INTERPOLATION_PROBES {
        interpolate(low, high, item, &nums)
    } else {
        (low + high) / 2  // Fall back to binary
    };
    probe_count += 1;
    // ...
}
```

## 8. References

1. Peterson, W. W. (1957). "Addressing for Random-Access Storage." IBM Journal of Research and Development.
2. Perl, Y., Itai, A., & Avni, H. (1978). "Interpolation Search—A Log Log N Search." Communications of the ACM.
3. Gonnet, G. H., Rogers, L. D., & George, J. A. (1980). "An Algorithmic and Complexity Analysis of Interpolation Search." Acta Informatica.

---

**Implementation:** [`src/searching/interpolation_search.rs`](../../src/searching/interpolation_search.rs)  
**See Also:** [Binary Search](binary_search.md), [Exponential Search](exponential_search.md)
