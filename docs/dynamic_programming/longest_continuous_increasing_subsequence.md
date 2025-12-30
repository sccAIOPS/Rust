# Longest Continuous Increasing Subsequence (LCIS)

## 1. Overview

The Longest Continuous Increasing Subsequence problem finds the length of the longest contiguous subarray where elements are strictly increasing. Unlike LIS, elements must be adjacent in the original array.

**File**: `src/dynamic_programming/longest_continuous_increasing_subsequence.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given array $A[0..n-1]$, find maximum length $L$ such that:

$$\exists i : A[i] < A[i+1] < \ldots < A[i+L-1]$$

### 2.2 Comparison with LIS

| Problem | Contiguous | Time | Example [1,3,2,4] |
|---------|------------|------|-------------------|
| LCIS | Yes | O(n) | 2 (either [1,3] or [2,4]) |
| LIS | No | O(n log n) | 3 ([1,3,4] or [1,2,4]) |

## 3. Algorithm Description

### 3.1 Simple Iteration

```
FUNCTION lcis(arr)
    IF len(arr) = 0 THEN RETURN 0
    
    max_len ← 1
    current_len ← 1
    
    FOR i ← 1 TO len(arr)-1 DO
        IF arr[i] > arr[i-1] THEN
            current_len ← current_len + 1
            max_len ← max(max_len, current_len)
        ELSE
            current_len ← 1
        END IF
    END FOR
    
    RETURN max_len
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input**: [1, 3, 5, 4, 7]

```
i=1: arr[1]=3 > arr[0]=1 → current_len=2, max_len=2
i=2: arr[2]=5 > arr[1]=3 → current_len=3, max_len=3
i=3: arr[3]=4 < arr[2]=5 → current_len=1
i=4: arr[4]=7 > arr[3]=4 → current_len=2, max_len=3

Result: 3 (subarray [1, 3, 5])
```

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(n)** - Single pass through array

### 4.2 Space Complexity
- **O(1)** - Only two variables needed

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn longest_continuous_increasing_subsequence<T: Ord>(input: &[T]) -> usize {
    if input.is_empty() {
        return 0;
    }
    
    let mut max_len = 1;
    let mut current_len = 1;
    
    for i in 1..input.len() {
        if input[i] > input[i - 1] {
            current_len += 1;
            max_len = max_len.max(current_len);
        } else {
            current_len = 1;
        }
    }
    
    max_len
}
```

### 5.2 Generic Implementation

Works with any type implementing `Ord`:
- Integers: `i32`, `u64`, etc.
- Floats: With custom comparator
- Custom types: Implement `Ord` trait

### 5.3 Edge Cases

| Input | Output | Reason |
|-------|--------|--------|
| [] | 0 | Empty array |
| [5] | 1 | Single element |
| [1, 2, 3, 4] | 4 | Fully increasing |
| [4, 3, 2, 1] | 1 | Fully decreasing |
| [1, 1, 1, 1] | 1 | All equal (not strictly increasing) |
| [1, 2, 1, 2] | 2 | Alternating |

## 6. Variants

### 6.1 Non-Strictly Increasing

Allow equal elements:

```rust
if input[i] >= input[i - 1] {
    current_len += 1;
    // ...
}
```

### 6.2 Return the Subsequence

```rust
fn lcis_with_sequence<T: Ord + Clone>(input: &[T]) -> Vec<T> {
    if input.is_empty() { return vec![]; }
    
    let mut max_start = 0;
    let mut max_len = 1;
    let mut start = 0;
    let mut current_len = 1;
    
    for i in 1..input.len() {
        if input[i] > input[i - 1] {
            current_len += 1;
            if current_len > max_len {
                max_len = current_len;
                max_start = start;
            }
        } else {
            start = i;
            current_len = 1;
        }
    }
    
    input[max_start..max_start + max_len].to_vec()
}
```

### 6.3 Count All LCIS of Maximum Length

```rust
fn count_max_lcis<T: Ord>(input: &[T]) -> (usize, usize) {
    // Returns (max_length, count)
    // ...
}
```

### 6.4 Longest Continuous Decreasing

Simply reverse the comparison:

```rust
if input[i] < input[i - 1] {
    // decreasing
}
```

## 7. Applications

1. **Stock Trading**: Longest consecutive price increase
2. **Temperature Data**: Longest warming trend
3. **Performance Metrics**: Longest improvement streak
4. **Signal Processing**: Detecting rising edges

## 8. Related Problems

| Problem | Description | Complexity |
|---------|-------------|------------|
| Longest Increasing Subsequence | Non-contiguous | O(n log n) |
| Maximum Consecutive Ones | Binary version | O(n) |
| Longest Turbulent Subarray | Alternating +/- | O(n) |
| Longest Mountain | Peak finding | O(n) |

## 9. Follow-up: Find All Maximum LCIS

```rust
fn find_all_max_lcis<T: Ord + Clone>(input: &[T]) -> Vec<Vec<T>> {
    if input.is_empty() { return vec![]; }
    
    // First pass: find max length
    let max_len = longest_continuous_increasing_subsequence(input);
    
    // Second pass: collect all of that length
    let mut result = Vec::new();
    let mut start = 0;
    let mut current_len = 1;
    
    for i in 1..=input.len() {
        let continuing = i < input.len() && input[i] > input[i - 1];
        
        if continuing {
            current_len += 1;
        } else {
            if current_len == max_len {
                result.push(input[start..start + current_len].to_vec());
            }
            start = i;
            current_len = 1;
        }
    }
    
    result
}
```

## 10. Comparison Table

| Input | LCIS | LIS | LCDS | LDS |
|-------|------|-----|------|-----|
| [1,3,5,4,7] | 3 | 4 | 2 | 2 |
| [5,4,3,2,1] | 1 | 1 | 5 | 5 |
| [1,2,3,4,5] | 5 | 5 | 1 | 1 |
| [2,1,3,2,4] | 2 | 3 | 2 | 2 |

## 11. Streaming Version

For infinite streams, maintain only current window:

```rust
struct LCISTracker<T: Ord> {
    prev: Option<T>,
    current_len: usize,
    max_len: usize,
}

impl<T: Ord> LCISTracker<T> {
    fn new() -> Self {
        Self { prev: None, current_len: 0, max_len: 0 }
    }
    
    fn push(&mut self, val: T) {
        if let Some(ref prev) = self.prev {
            if val > *prev {
                self.current_len += 1;
            } else {
                self.current_len = 1;
            }
        } else {
            self.current_len = 1;
        }
        self.max_len = self.max_len.max(self.current_len);
        self.prev = Some(val);
    }
}
```

## 12. References

1. [LeetCode Problem 674](https://leetcode.com/problems/longest-continuous-increasing-subsequence/)
2. [GeeksforGeeks - Longest Increasing Subarray](https://www.geeksforgeeks.org/longest-increasing-subarray/)
