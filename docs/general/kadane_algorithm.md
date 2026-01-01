# Kadane's Algorithm

## 1. Overview

Kadane's algorithm solves the maximum subarray problem: finding the contiguous subarray within a one-dimensional array of numbers that has the largest sum. Developed by Jay Kadane in 1984, this elegant algorithm runs in linear time using dynamic programming principles.

The algorithm is a classic example of the power of dynamic programming and greedy optimization, transforming a problem that appears to require $O(n^2)$ time into an $O(n)$ solution.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A = [a_1, a_2, ..., a_n]$ of real numbers (which may include negative values), find indices $i$ and $j$ where $1 \leq i \leq j \leq n$ such that the sum $\sum_{k=i}^{j} a_k$ is maximized.

### 2.2 Mathematical Model

**Input**: Array $A[1..n]$ of real numbers

**Output**: Maximum sum $S_{max}$ and optionally the subarray $A[i..j]$ achieving this sum

**Objective**: 
$$S_{max} = \max_{1 \leq i \leq j \leq n} \sum_{k=i}^{j} a_k$$

**Dynamic Programming Recurrence**:

Define $M[i]$ as the maximum sum of any subarray ending at position $i$:

$$M[i] = \max(A[i], M[i-1] + A[i])$$

with base case $M[1] = A[1]$.

Then: $S_{max} = \max_{1 \leq i \leq n} M[i]$

**Intuition**: At each position, either extend the previous subarray or start a new one.

### 2.3 Correctness Proof

**Theorem**: Kadane's algorithm correctly computes the maximum subarray sum.

**Proof**:

1. **Optimal Substructure**: If subarray $A[i..j]$ has maximum sum, then subarray $A[i..j-1]$ either:
   - Has maximum sum among all subarrays ending at $j-1$, or
   - Has negative sum (in which case $A[j..j]$ is better)

2. **Greedy Choice**: At position $i$, we choose:
   - If $M[i-1] > 0$: Extend by adding $A[i]$ to previous sum
   - If $M[i-1] \leq 0$: Start fresh with $A[i]$

3. **Invariant**: After processing position $i$, $M[i]$ correctly stores the maximum sum of any subarray ending at $i$.

4. **Completeness**: Since every subarray ends somewhere, checking all $M[i]$ values ensures we find the global maximum. ∎

## 3. Algorithm Description

### 3.1 Intuition

The key insight is that at any position, a subarray ending there is either:
1. The current element alone (start a new subarray)
2. The current element plus the best subarray ending at the previous position (extend)

We keep track of:
- `current_sum`: Best sum ending at current position
- `max_sum`: Best sum seen so far overall

As we scan left to right:
- If `current_sum` becomes negative, we reset it (no point carrying negative weight forward)
- Otherwise, we add the next element to `current_sum`
- Update `max_sum` whenever we see a better sum

### 3.2 Pseudocode

```
function KadaneAlgorithm(array):
    n = length(array)
    
    if n == 0:
        return 0  // Or error/special value
    
    max_sum = array[0]
    current_sum = array[0]
    
    for i from 1 to n-1:
        // Either extend the subarray or start new one
        current_sum = max(array[i], current_sum + array[i])
        
        // Update global maximum
        max_sum = max(max_sum, current_sum)
    
    return max_sum

// Variant: Also return the subarray indices
function KadaneWithIndices(array):
    n = length(array)
    
    max_sum = array[0]
    current_sum = array[0]
    
    start = 0
    end = 0
    temp_start = 0
    
    for i from 1 to n-1:
        if current_sum < 0:
            current_sum = array[i]
            temp_start = i
        else:
            current_sum = current_sum + array[i]
        
        if current_sum > max_sum:
            max_sum = current_sum
            start = temp_start
            end = i
    
    return (max_sum, start, end)
```

### 3.3 Step-by-Step Example

Find maximum subarray sum in: `[-2, 1, -3, 4, -1, 2, 1, -5, 4]`

```
Index:        0   1   2   3   4   5   6   7   8
Array:       -2   1  -3   4  -1   2   1  -5   4

i=0: current_sum = -2, max_sum = -2

i=1: current_sum = max(1, -2+1) = max(1, -1) = 1
     max_sum = max(-2, 1) = 1

i=2: current_sum = max(-3, 1+(-3)) = max(-3, -2) = -2
     max_sum = max(1, -2) = 1

i=3: current_sum = max(4, -2+4) = max(4, 2) = 4
     max_sum = max(1, 4) = 4

i=4: current_sum = max(-1, 4+(-1)) = max(-1, 3) = 3
     max_sum = max(4, 3) = 4

i=5: current_sum = max(2, 3+2) = max(2, 5) = 5
     max_sum = max(4, 5) = 5

i=6: current_sum = max(1, 5+1) = max(1, 6) = 6
     max_sum = max(5, 6) = 6

i=7: current_sum = max(-5, 6+(-5)) = max(-5, 1) = 1
     max_sum = max(6, 1) = 6

i=8: current_sum = max(4, 1+4) = max(4, 5) = 5
     max_sum = max(6, 5) = 6

Result: max_sum = 6
Subarray: [4, -1, 2, 1] (indices 3 to 6)
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Single pass**: $O(n)$
- **Per iteration**: $O(1)$ (simple comparisons and arithmetic)
- **Total**: $O(n)$

This is optimal since we must examine each element at least once.

**Comparison with Naive Approach**:
- Brute force (check all subarrays): $O(n^2)$ or $O(n^3)$
- Divide and conquer: $O(n \log n)$
- Kadane's algorithm: $O(n)$ ✓

### 4.2 Space Complexity

- **Variables**: $O(1)$ for `max_sum`, `current_sum`, and optional indices
- **Input**: $O(n)$ (not counted as auxiliary space)
- **Total auxiliary space**: $O(1)$

The algorithm is in-place and doesn't require any additional data structures.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
// Basic implementation
fn kadane(arr: &[i32]) -> i32 {
    if arr.is_empty() {
        return 0; // Or panic!/return Option/Result
    }
    
    let mut max_sum = arr[0];
    let mut current_sum = arr[0];
    
    for &num in arr.iter().skip(1) {
        current_sum = num.max(current_sum + num);
        max_sum = max_sum.max(current_sum);
    }
    
    max_sum
}

// Generic over numeric types
fn kadane_generic<T>(arr: &[T]) -> T 
where
    T: Copy + Ord + std::ops::Add<Output = T>,
{
    assert!(!arr.is_empty(), "Array must not be empty");
    
    let mut max_sum = arr[0];
    let mut current_sum = arr[0];
    
    for &num in arr.iter().skip(1) {
        current_sum = num.max(current_sum + num);
        max_sum = max_sum.max(current_sum);
    }
    
    max_sum
}

// Return subarray indices
fn kadane_with_indices(arr: &[i32]) -> (i32, usize, usize) {
    assert!(!arr.is_empty(), "Array must not be empty");
    
    let mut max_sum = arr[0];
    let mut current_sum = arr[0];
    let mut start = 0;
    let mut end = 0;
    let mut temp_start = 0;
    
    for (i, &num) in arr.iter().enumerate().skip(1) {
        if current_sum < 0 {
            current_sum = num;
            temp_start = i;
        } else {
            current_sum += num;
        }
        
        if current_sum > max_sum {
            max_sum = current_sum;
            start = temp_start;
            end = i;
        }
    }
    
    (max_sum, start, end)
}

// Using functional style
fn kadane_functional(arr: &[i32]) -> i32 {
    arr.iter()
        .skip(1)
        .fold((arr[0], arr[0]), |(max_sum, current_sum), &num| {
            let new_current = num.max(current_sum + num);
            (max_sum.max(new_current), new_current)
        })
        .0
}
```

**Key Rust Features**:
- Iterator methods (`iter()`, `skip()`, `fold()`)
- Pattern matching for tuple destructuring
- Generic implementation with trait bounds
- Safe handling with `Option` or `assert!` for empty arrays
- `Copy` trait for numeric types

### 5.2 Edge Cases

1. **Empty array**: Undefined; return error, `None`, or 0
2. **Single element**: Return that element
3. **All negative**: Return the least negative element
4. **All positive**: Return sum of entire array
5. **All zeros**: Return 0
6. **Mixed signs**: Standard case
7. **Very large numbers**: Risk of overflow (use checked arithmetic or wider types)
8. **Floating-point**: Handle NaN and infinity appropriately

**Overflow Protection** (Rust):
```rust
fn kadane_checked(arr: &[i32]) -> Option<i32> {
    if arr.is_empty() {
        return None;
    }
    
    let mut max_sum = arr[0];
    let mut current_sum = arr[0];
    
    for &num in arr.iter().skip(1) {
        current_sum = match current_sum.checked_add(num) {
            Some(sum) => num.max(sum),
            None => return None, // Overflow
        };
        max_sum = max_sum.max(current_sum);
    }
    
    Some(max_sum)
}
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Financial Analysis**
- Maximum profit period in stock prices
- Best consecutive days for investment
- Optimal trading window identification
- Revenue trend analysis

**2. Data Analysis**
- Finding periods of maximum growth
- Identifying trends in time series data
- Anomaly detection (unusual high-sum periods)
- Signal processing (maximum energy intervals)

**3. Genomics & Bioinformatics**
- Finding GC-rich regions in DNA sequences
- Identifying protein domains with specific properties
- Maximum scoring segments in sequence alignment

**4. Image Processing**
- Finding brightest rectangular regions
- Histogram analysis
- Feature extraction

**5. Resource Allocation**
- Optimal task scheduling
- Maximum utilization periods
- Load balancing windows

**6. Gaming & Simulation**
- Maximum score sequences
- Optimal move chains
- Performance analysis

### 6.2 Related Algorithms

**Extensions**:
- **2D Kadane**: Maximum sum rectangle in 2D array ($O(n^3)$)
- **Circular Array**: Maximum subarray in circular array (two Kadane runs)
- **Maximum Product Subarray**: Track both max and min products
- **K-Kadane**: Find K disjoint subarrays with maximum total sum

**Related Problems**:
- **Maximum Product Subarray**: Track min/max due to negative numbers
- **Best Time to Buy/Sell Stock**: Variant with constraints
- **Longest Subarray with Sum ≤ K**: Sliding window technique
- **Maximum Average Subarray**: Kadane with averaging
- **Minimum Subarray Sum**: Same algorithm with negation

**Alternative Approaches**:
- **Divide and Conquer**: $O(n \log n)$, useful for parallel processing
- **Prefix Sum + Min Tracker**: Alternative $O(n)$ approach
- **Segment Tree**: For dynamic updates ($O(\log n)$ per update)

**When to Use**:
- **Kadane's**: Standard maximum subarray, best performance
- **2D Kadane**: Maximum sum rectangle problems
- **Divide & Conquer**: Parallel computing environments
- **Segment Tree**: Frequent array modifications

## 7. References

### Academic Papers
1. Bentley, J. (1984). "Programming Pearls: Algorithm Design Techniques". *Communications of the ACM*, 27(9), 865-873.
2. Gries, D. (1982). "A Note on a Standard Strategy for Developing Loop Invariants and Loops". *Science of Computer Programming*, 2(3), 207-214.
3. Takaoka, T. (2002). "Efficient Algorithms for the Maximum Subarray Problem by Distance Matrix Multiplication". *Electronic Notes in Theoretical Computer Science*, 61, 191-200.

### Books
1. Bentley, J. (1999). *Programming Pearls* (2nd ed.). Addison-Wesley. Column 8: Algorithm Design Techniques.
2. Cormen, T.H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Section 4.1: The maximum-subarray problem.
3. Kleinberg, J., & Tardos, É. (2005). *Algorithm Design*. Addison-Wesley. Chapter 6: Dynamic Programming.

### Online Resources
1. [Maximum Subarray Problem - Wikipedia](https://en.wikipedia.org/wiki/Maximum_subarray_problem)
2. [Kadane's Algorithm - GeeksforGeeks](https://www.geeksforgeeks.org/largest-sum-contiguous-subarray/)
3. [Maximum Subarray - LeetCode](https://leetcode.com/problems/maximum-subarray/)
4. [Programming Pearls - Kadane's Algorithm](http://www.cs.cmu.edu/~15451-f17/lectures/lec19-dp1.pdf)

### Implementation
- Source: `src/general/kadane_algorithm.rs`
- Tests: Included in source file under `#[cfg(test)] mod tests`
