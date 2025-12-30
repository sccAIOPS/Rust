# Bucket Sort

## 1. Overview

Bucket Sort is a distribution sorting algorithm that works by distributing elements into a number of buckets, sorting each bucket individually, and then concatenating the results. It is particularly efficient for uniformly distributed data.

### Key Characteristics
- **Type**: Non-comparison (distribution), uses comparison sort for buckets
- **In-place**: No
- **Stable**: Yes (if bucket sort is stable)
- **Adaptive**: No

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given $n$ elements uniformly distributed over range $[0, M)$, distribute elements into $n$ buckets and sort.

### 2.2 Expected Analysis

With $n$ buckets and $n$ uniformly distributed elements:
- Expected elements per bucket: 1
- Expected time for sorting each bucket: O(1)
- Total expected time: O(n)

### 2.3 Bucket Assignment

For element $x$ in range $[0, M)$ with $n$ buckets:
$$\text{bucket}(x) = \lfloor \frac{n \times x}{M} \rfloor$$

## 3. Algorithm Description

### 3.1 Pseudocode

```
BUCKET_SORT(A)
    n ← length(A)
    max_val ← maximum(A)
    buckets ← array of n+1 empty lists
    
    // Distribute elements into buckets
    for x in A do
        bucket_idx ← n × x / max_val
        append x to buckets[bucket_idx]
    
    // Sort each bucket
    for bucket in buckets do
        INSERTION_SORT(bucket)
    
    // Concatenate buckets
    result ← empty array
    for bucket in buckets do
        append bucket to result
    
    return result
```

### 3.2 Step-by-Step Example

Sorting array `[0.42, 0.32, 0.23, 0.52, 0.25, 0.47, 0.51]` with 5 buckets:

```
Range: [0, 1), 5 buckets

Step 1: Distribute into buckets
bucket[0] (0.0-0.2): []
bucket[1] (0.2-0.4): [0.32, 0.23, 0.25]
bucket[2] (0.4-0.6): [0.42, 0.52, 0.47, 0.51]
bucket[3] (0.6-0.8): []
bucket[4] (0.8-1.0): []

Step 2: Sort each bucket
bucket[1]: [0.23, 0.25, 0.32]
bucket[2]: [0.42, 0.47, 0.51, 0.52]

Step 3: Concatenate
Result: [0.23, 0.25, 0.32, 0.42, 0.47, 0.51, 0.52]
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Condition |
|------|------------|-----------|
| **Best** | O(n) | Uniform distribution |
| **Average** | O(n + k) | k = number of buckets |
| **Worst** | O(n²) | All elements in one bucket |

### 4.2 Space Complexity

| Type | Complexity |
|------|------------|
| **Auxiliary** | O(n + k) |

## 5. Implementation

### 5.1 Rust Implementation

```rust
pub fn bucket_sort(arr: &[usize]) -> Vec<usize> {
    if arr.is_empty() {
        return vec![];
    }

    let max = *arr.iter().max().unwrap();
    let len = arr.len();
    let mut buckets = vec![vec![]; len + 1];

    for x in arr {
        buckets[len * *x / max].push(*x);
    }

    for bucket in buckets.iter_mut() {
        insertion_sort(bucket);
    }

    let mut result = vec![];
    for bucket in buckets {
        for x in bucket {
            result.push(x);
        }
    }

    result
}
```

## 6. Real-World Applications

1. **Floating-point sorting**: When values are uniformly distributed
2. **External sorting**: Distributing data across storage
3. **Parallel sorting**: Buckets can be sorted independently

## 7. References

1. Cormen, T. H., et al. "Introduction to Algorithms", Chapter 8.

## 8. Source Code

**Implementation**: [src/sorting/bucket_sort.rs](../../src/sorting/bucket_sort.rs)
