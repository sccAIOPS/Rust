# Counting Sort

## 1. Overview

Counting Sort is a non-comparison integer sorting algorithm that operates by counting the occurrences of each distinct element. It is particularly efficient when the range of input values (k) is not significantly larger than the number of elements (n).

### Key Characteristics
- **Type**: Non-comparison, distribution sort
- **In-place**: No (requires O(k) auxiliary space)
- **Stable**: Yes (with proper implementation)
- **Adaptive**: No

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$ of $n$ non-negative integers in range $[0, k]$, sort the array using counting.

### 2.2 Key Insight

If we know exactly how many elements are smaller than element $x$, we know exactly where $x$ should go in the sorted output.

### 2.3 Mathematical Model

For each value $v$ in range $[0, k]$:
$$\text{count}[v] = |\{i : A[i] = v\}|$$

Position of value $v$ in sorted output:
$$\text{position}[v] = \sum_{i=0}^{v-1} \text{count}[i]$$

## 3. Algorithm Description

### 3.1 Intuition

1. Count occurrences of each value
2. Calculate cumulative counts (determines final positions)
3. Place each element in its correct position

### 3.2 Pseudocode

```
COUNTING_SORT(A, k)
    n ← length(A)
    count ← array of size k+1, initialized to 0
    
    // Step 1: Count occurrences
    for i ← 0 to n - 1 do
        count[A[i]] ← count[A[i]] + 1
    
    // Step 2: In-place reconstruction
    index ← 0
    for value ← 0 to k do
        while count[value] > 0 do
            A[index] ← value
            index ← index + 1
            count[value] ← count[value] - 1
```

### 3.3 Step-by-Step Example

Sorting array `[4, 2, 2, 8, 3, 3, 1]` with max value 8:

```
Original: [4, 2, 2, 8, 3, 3, 1]

Step 1: Count occurrences
value:  0  1  2  3  4  5  6  7  8
count: [0, 1, 2, 2, 1, 0, 0, 0, 1]
        ^  ^  ^  ^  ^           ^
        0  1  2  2  1           1 occurrence

Step 2: Reconstruct array
value=0: count=0, skip
value=1: count=1, output [1]
value=2: count=2, output [1, 2, 2]
value=3: count=2, output [1, 2, 2, 3, 3]
value=4: count=1, output [1, 2, 2, 3, 3, 4]
value=5-7: count=0, skip
value=8: count=1, output [1, 2, 2, 3, 3, 4, 8]

Final: [1, 2, 2, 3, 3, 4, 8]
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Notes |
|------|------------|-------|
| **All cases** | O(n + k) | n = elements, k = value range |

- Counting phase: O(n)
- Output phase: O(n + k)

### 4.2 Space Complexity

| Type | Complexity | Notes |
|------|------------|-------|
| **Auxiliary** | O(k) | Count array only |
| **With stability** | O(n + k) | Output array + count array |

### 4.3 When Counting Sort is Efficient

Efficient when $k = O(n)$:
- If $k \leq n$: Truly linear O(n)
- If $k \gg n$: Better to use comparison sort

## 5. Implementation Notes

### 5.1 Rust Implementation (In-place)

```rust
pub fn counting_sort(arr: &mut [u32], maxval: usize) {
    let mut occurences: Vec<usize> = vec![0; maxval + 1];

    for &data in arr.iter() {
        occurences[data as usize] += 1;
    }

    let mut i = 0;
    for (data, &number) in occurences.iter().enumerate() {
        for _ in 0..number {
            arr[i] = data as u32;
            i += 1;
        }
    }
}
```

### 5.2 Generic Implementation

```rust
pub fn generic_counting_sort<T: Into<u64> + From<u8> + AddAssign + Copy>(
    arr: &mut [T],
    maxval: usize,
) {
    let mut occurences: Vec<usize> = vec![0; maxval + 1];

    for &data in arr.iter() {
        occurences[data.into() as usize] += 1;
    }

    let mut i = 0;
    let mut data = T::from(0);

    for &number in occurences.iter() {
        for _ in 0..number {
            arr[i] = data;
            i += 1;
        }
        data += T::from(1);
    }
}
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Empty array | No iterations |
| Single element | Works correctly |
| All same values | Single count entry |
| Sparse values | Many zero counts |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Sorting small integers**
   - Age distribution sorting
   - Grade sorting (0-100)
   - Status code sorting

2. **As subroutine**
   - Radix Sort digit sorting
   - Bucket Sort bucket sorting

3. **Histogram computation**
   - Value distribution analysis
   - Frequency counting

4. **Suffix array construction**
   - String algorithms

### 6.2 Related Algorithms

| Algorithm | Relationship | When to Prefer |
|-----------|--------------|----------------|
| [Radix Sort](radix_sort.md) | Uses counting sort | Large integers |
| [Bucket Sort](bucket_sort.md) | Distribution sort | Floating point |
| [Pigeonhole Sort](pigeonhole_sort.md) | Similar concept | When n ≈ k |

## 7. Stability Considerations

### 7.1 Non-Stable Version (Current Implementation)

The in-place reconstruction loses stability:
```
[(3,'a'), (1,'b'), (3,'c')] → [1, 3, 3]
The original objects are not preserved!
```

### 7.2 Stable Version

```rust
fn stable_counting_sort<T: Clone>(arr: &[T], key: impl Fn(&T) -> usize, k: usize) -> Vec<T> {
    let mut count = vec![0; k + 1];
    
    // Count occurrences
    for item in arr {
        count[key(item)] += 1;
    }
    
    // Cumulative count
    for i in 1..=k {
        count[i] += count[i - 1];
    }
    
    // Build output (iterate in reverse for stability)
    let mut output = vec![arr[0].clone(); arr.len()];
    for item in arr.iter().rev() {
        let key_val = key(item);
        count[key_val] -= 1;
        output[count[key_val]] = item.clone();
    }
    
    output
}
```

## 8. Comparison with Other Sorts

| Criterion | Counting Sort | Quick Sort | Radix Sort |
|-----------|---------------|------------|------------|
| Time | O(n + k) | O(n log n) | O(d(n + k)) |
| Space | O(k) | O(log n) | O(n + k) |
| Comparison-based | No | Yes | No |
| Works for any type | No | Yes | No |
| Range requirement | k = O(n) | None | None |

## 9. Limitations

1. **Only non-negative integers**: No direct support for floats or negatives
2. **Range-dependent space**: Large k means large auxiliary space
3. **Not general purpose**: Works only for specific value types

### Handling Negatives

```rust
fn counting_sort_signed(arr: &mut [i32]) {
    let min = *arr.iter().min().unwrap();
    let max = *arr.iter().max().unwrap();
    let range = (max - min + 1) as usize;
    
    let mut count = vec![0; range];
    
    for &x in arr.iter() {
        count[(x - min) as usize] += 1;
    }
    
    let mut i = 0;
    for (offset, &cnt) in count.iter().enumerate() {
        for _ in 0..cnt {
            arr[i] = min + offset as i32;
            i += 1;
        }
    }
}
```

## 10. References

1. Cormen, T. H., et al. "Introduction to Algorithms", Chapter 8.
2. Knuth, D. E. "The Art of Computer Programming, Vol. 3".
3. Seward, H. H. (1954). "Information sorting in the application of electronic digital computers to business operations". MIT Master's thesis.

## 11. Source Code

**Implementation**: [src/sorting/counting_sort.rs](../../src/sorting/counting_sort.rs)

```rust
pub fn counting_sort(arr: &mut [u32], maxval: usize) {
    let mut occurences: Vec<usize> = vec![0; maxval + 1];

    for &data in arr.iter() {
        occurences[data as usize] += 1;
    }

    let mut i = 0;
    for (data, &number) in occurences.iter().enumerate() {
        for _ in 0..number {
            arr[i] = data as u32;
            i += 1;
        }
    }
}
```
