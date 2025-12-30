# Radix Sort

## 1. Overview

Radix Sort is a non-comparison integer sorting algorithm that sorts numbers by processing individual digits. It processes digits either from least significant digit (LSD) or most significant digit (MSD), using a stable sort (usually counting sort) as a subroutine.

### Key Characteristics
- **Type**: Non-comparison, distribution sort
- **In-place**: No (requires O(n + k) auxiliary space)
- **Stable**: Yes (when using stable subroutine)
- **Adaptive**: No

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$ of $n$ non-negative integers with maximum value $k$, sort using digit-by-digit processing.

### 2.2 Mathematical Model

For numbers with $d$ digits in base $b$:
- Number of passes: $d = \lceil \log_b(k) \rceil$
- Each pass: O(n + b) with counting sort
- Total: O(d × (n + b)) = O((n + b) × log_b(k))

### 2.3 Optimal Radix Selection

Choose radix $b \approx n$ for optimal performance:
- If $b = n$: Complexity = O(n × log_n(k))
- For $k = O(n^c)$: Complexity = O(cn)

## 3. Algorithm Description

### 3.1 Intuition

Radix Sort sorts numbers digit by digit, starting from the least significant digit:
1. Group numbers by their last digit
2. Concatenate groups in order
3. Repeat for each digit position
4. After processing all digits, array is sorted

### 3.2 Pseudocode

```
RADIX_SORT(A)
    max_val ← maximum value in A
    radix ← next_power_of_2(length(A))
    place ← 1
    
    while place ≤ max_val do
        // Count occurrences of each digit
        count ← array of size radix, initialized to 0
        for x in A do
            digit ← (x / place) mod radix
            count[digit] ← count[digit] + 1
        
        // Compute cumulative counts (positions)
        for i ← 1 to radix - 1 do
            count[i] ← count[i] + count[i-1]
        
        // Build sorted array (stable sort)
        output ← array of size length(A)
        for x in reverse(A) do
            digit ← (x / place) mod radix
            count[digit] ← count[digit] - 1
            output[count[digit]] ← x
        
        A ← output
        place ← place × radix
```

### 3.3 Step-by-Step Example

Sorting array `[170, 45, 75, 90, 802, 24, 2, 66]` with radix 10:

```
Original: [170, 45, 75, 90, 802, 24, 2, 66]

Pass 1 (ones digit):
170 → 0    Sort by last digit:
45  → 5    0: [170, 90]
75  → 5    2: [802, 2]
90  → 0    4: [24]
802 → 2    5: [45, 75]
24  → 4    6: [66]
2   → 2
66  → 6
Result: [170, 90, 802, 2, 24, 45, 75, 66]

Pass 2 (tens digit):
170 → 7    Sort by tens digit:
90  → 9    0: [802, 2]
802 → 0    2: [24]
2   → 0    4: [45]
24  → 2    6: [66]
45  → 4    7: [170, 75]
75  → 7    9: [90]
66  → 6
Result: [802, 2, 24, 45, 66, 170, 75, 90]

Pass 3 (hundreds digit):
802 → 8    Sort by hundreds digit:
2   → 0    0: [2, 24, 45, 66, 75, 90]
24  → 0    1: [170]
45  → 0    8: [802]
66  → 0
170 → 1
75  → 0
90  → 0
Result: [2, 24, 45, 66, 75, 90, 170, 802]

Final: [2, 24, 45, 66, 75, 90, 170, 802]
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Notes |
|------|------------|-------|
| **All cases** | O((n + b) × d) | d = digits, b = radix |

For optimal radix (b ≈ n):
- O(n × d) where d = log_n(k)
- For bounded k: O(n)

### 4.2 Space Complexity

| Type | Complexity | Notes |
|------|------------|-------|
| **Auxiliary** | O(n + b) | Output array + count array |

### 4.3 When Radix Sort is Linear

Radix Sort achieves O(n) when:
- Maximum value k is bounded by O(n^c) for constant c
- Numbers have fixed number of digits

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
pub fn radix_sort(arr: &mut [u64]) {
    let max: usize = match arr.iter().max() {
        Some(&x) => x as usize,
        None => return,
    };
    
    // Make radix a power of 2 close to arr.len() for optimal runtime
    let radix = arr.len().next_power_of_two();
    
    let mut place = 1;
    while place <= max {
        let digit_of = |x| x as usize / place % radix;
        
        // Count digit occurrences
        let mut counter = vec![0; radix];
        for &x in arr.iter() {
            counter[digit_of(x)] += 1;
        }
        
        // Compute last index of each digit
        for i in 1..radix {
            counter[i] += counter[i - 1];
        }
        
        // Write elements to their new indices
        for &x in arr.to_owned().iter().rev() {
            counter[digit_of(x)] -= 1;
            arr[counter[digit_of(x)]] = x;
        }
        
        place *= radix;
    }
}
```

**Key Implementation Details**:

1. **Power-of-2 radix**: Enables bit operations for digit extraction
2. **Adaptive radix**: `radix = arr.len().next_power_of_two()`
3. **Stable counting sort**: Preserves order of equal elements

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty array | Returns immediately |
| Single element | Returns immediately |
| All zeros | Single pass |
| Large numbers | More passes needed |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Integer sorting**
   - Sorting database records by ID
   - Index construction

2. **String sorting (MSD Radix)**
   - Suffix array construction
   - URL sorting

3. **Parallel computing**
   - GPU-based sorting
   - MapReduce sorting

4. **Network packet processing**
   - IP address sorting
   - Routing table lookups

### 6.2 Related Algorithms

| Algorithm | Relationship | When to Prefer |
|-----------|--------------|----------------|
| [Counting Sort](counting_sort.md) | Subroutine | Single digit range |
| [Bucket Sort](bucket_sort.md) | Distribution sort | Uniform distribution |
| MSD Radix Sort | Variant | Variable-length strings |

## 7. LSD vs MSD Radix Sort

### LSD (Least Significant Digit)

```
Process: units → tens → hundreds
Pro: Naturally stable
Con: Must process all digits
```

### MSD (Most Significant Digit)

```
Process: hundreds → tens → units (recursive)
Pro: Can short-circuit for unequal prefixes
Con: Requires careful stability handling
```

## 8. Comparison with Comparison Sorts

| Criterion | Radix Sort | Quick Sort |
|-----------|------------|------------|
| Time (best) | O(n) | O(n log n) |
| Time (worst) | O(dn) | O(n²) |
| Space | O(n + k) | O(log n) |
| Comparison-based | No | Yes |
| Works on floats | With encoding | Yes |

**When Radix Sort wins**: Large arrays of bounded integers

## 9. Handling Negative Numbers

```rust
fn radix_sort_signed(arr: &mut [i64]) {
    // Separate negative and positive
    let negatives: Vec<i64> = arr.iter().filter(|&&x| x < 0).copied().collect();
    let positives: Vec<i64> = arr.iter().filter(|&&x| x >= 0).copied().collect();
    
    // Sort positives ascending, negatives by absolute value (then reverse)
    // Combine: reversed negatives + positives
}
```

## 10. References

1. Cormen, T. H., et al. "Introduction to Algorithms", Chapter 8.
2. Knuth, D. E. "The Art of Computer Programming, Vol. 3".
3. McIlroy, P. M., et al. (1993). "Engineering Radix Sort".

## 11. Source Code

**Implementation**: [src/sorting/radix_sort.rs](../../src/sorting/radix_sort.rs)

```rust
pub fn radix_sort(arr: &mut [u64]) {
    let max: usize = match arr.iter().max() {
        Some(&x) => x as usize,
        None => return,
    };
    let radix = arr.len().next_power_of_two();
    let mut place = 1;
    while place <= max {
        let digit_of = |x| x as usize / place % radix;
        let mut counter = vec![0; radix];
        for &x in arr.iter() {
            counter[digit_of(x)] += 1;
        }
        for i in 1..radix {
            counter[i] += counter[i - 1];
        }
        for &x in arr.to_owned().iter().rev() {
            counter[digit_of(x)] -= 1;
            arr[counter[digit_of(x)]] = x;
        }
        place *= radix;
    }
}
```
