# Find Missing Number

## 1. Overview

The **Find Missing Number** algorithm identifies the single missing element in a sequence of consecutive integers using XOR bit manipulation. This elegant solution achieves O(n) time complexity with O(1) space, avoiding the need for sorting or hash sets.

### Historical Context

This problem is a classic interview question that demonstrates the power of XOR properties. It showcases how bit manipulation can provide solutions that are both space-efficient and elegant.

### Key Insight

XOR has a self-canceling property: `a ^ a = 0`. When we XOR all expected numbers with all present numbers, the duplicates cancel out, leaving only the missing number.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Input**: An array of $n$ integers representing a sequence of $n+1$ consecutive integers with one missing  
**Output**: The missing integer from the sequence

**Constraints**:
- The sequence is consecutive (no gaps except the missing number)
- Exactly one number is missing

### 2.2 Mathematical Model

#### XOR Properties

| Property | Expression | Explanation |
|----------|------------|-------------|
| Self-inverse | $a \oplus a = 0$ | XOR of identical values is 0 |
| Identity | $a \oplus 0 = a$ | XOR with 0 returns original |
| Commutative | $a \oplus b = b \oplus a$ | Order doesn't matter |
| Associative | $(a \oplus b) \oplus c = a \oplus (b \oplus c)$ | Grouping doesn't matter |

#### Cancellation Principle

Given array $A = [a_1, a_2, ..., a_n]$ which is a permutation of $[low, low+1, ..., high]$ with one element $m$ missing:

$$\bigoplus_{i=low}^{high} i \oplus \bigoplus_{j=1}^{n} a_j = m$$

All numbers except $m$ appear exactly twice (once in the range, once in the array) and cancel out.

### 2.3 Correctness Proof

**Theorem**: XORing all expected values with all present values yields the missing number.

**Proof**:

Let $S = \{low, low+1, ..., high\}$ be the complete sequence.
Let $A \subset S$ be the array with $|A| = |S| - 1$.
Let $m = S \setminus A$ be the missing element.

Compute:
$$X = \left(\bigoplus_{s \in S} s\right) \oplus \left(\bigoplus_{a \in A} a\right)$$

Since every element in $A$ also appears in $S$:
$$X = \left(\bigoplus_{a \in A} a \oplus a\right) \oplus m = 0 \oplus m = m$$ $\square$

## 3. Algorithm Description

### 3.1 Intuition

Imagine you have a checklist of all expected numbers and a bag of received numbers. Instead of checking off each one, you XOR everything together. The duplicate XORs cancel to zero, and you're left with the unique (missing) number.

### 3.2 Pseudocode

```
function find_missing_number(nums):
    if nums is empty:
        return ERROR("input array must not be empty")
    if length(nums) == 1:
        return ERROR("array must have at least 2 elements")
    
    low = min(nums)
    high = max(nums)
    
    result = high  // Initialize with high
    
    for i from low to high-1:
        index = i - low
        result = result XOR i XOR nums[index]
    
    return result
```

### 3.3 Step-by-Step Example

**Example**: Find missing number in `[0, 1, 3, 4]`

```
Array: [0, 1, 3, 4]
Expected sequence: [0, 1, 2, 3, 4]
Missing: ?

Step 1: Find range
low = 0, high = 4

Step 2: Initialize result
result = high = 4 (binary: 100)

Step 3: XOR loop
i=0: result = 4 ^ 0 ^ nums[0] = 4 ^ 0 ^ 0 = 100 ^ 000 ^ 000 = 100 (4)
i=1: result = 4 ^ 1 ^ nums[1] = 4 ^ 1 ^ 1 = 100 ^ 001 ^ 001 = 100 (4)
i=2: result = 4 ^ 2 ^ nums[2] = 4 ^ 2 ^ 3 = 100 ^ 010 ^ 011 = 101 (5)
i=3: result = 5 ^ 3 ^ nums[3] = 5 ^ 3 ^ 4 = 101 ^ 011 ^ 100 = 010 (2)

Result: 2 ✓
```

**Alternative View - Full XOR Expansion**:
```
Expected XOR: 0 ^ 1 ^ 2 ^ 3 ^ 4 = 4
Present XOR:  0 ^ 1 ^ 3 ^ 4 = 6
Combined:     4 ^ 6 = 100 ^ 110 = 010 = 2 ✓
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Explanation |
|------|------------|-------------|
| All cases | $O(n)$ | Single pass through the array |

Breakdown:
- Finding min/max: $O(n)$
- XOR loop: $O(n)$
- Total: $O(n)$

### 4.2 Space Complexity

| Metric | Complexity |
|--------|------------|
| Auxiliary space | $O(1)$ |
| Stack space | $O(1)$ |

This is a significant advantage over:
- Sorting approach: O(n log n) time or O(n) space
- Hash set approach: O(n) space

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
pub fn find_missing_number(nums: &[i32]) -> Result<i32, String> {
    if nums.is_empty() {
        return Err("input array must not be empty".to_string());
    }
    if nums.len() == 1 {
        return Err("array must have at least 2 elements to find a missing number".to_string());
    }

    let low = *nums.iter().min().unwrap();
    let high = *nums.iter().max().unwrap();

    let mut missing_number = high;

    for i in low..high {
        let index = (i - low) as usize;
        missing_number ^= i ^ nums[index];
    }

    Ok(missing_number)
}
```

**Key Implementation Details**:
- Uses iterators for finding min/max
- Returns `Result` for proper error handling
- Handles negative numbers correctly

### 5.2 Edge Cases

| Input | Expected Output | Reasoning |
|-------|-----------------|-----------|
| `[]` | Error | Empty array |
| `[5]` | Error | Single element, can't determine sequence |
| `[0, 2]` | 1 | Simple case |
| `[4, 3, 1, 0]` | 2 | Unordered input |
| `[-4, -3, -1, 0]` | -2 | Negative numbers |
| `[-2, 2, 1, 3, 0]` | -1 | Mixed positive/negative |
| `[100, 101, 103, 104]` | 102 | Large range |

### 5.3 Alternative Approaches

**Mathematical Sum Formula**:
```rust
fn find_missing_sum(nums: &[i32]) -> i32 {
    let n = nums.len() as i32 + 1;
    let expected_sum = n * (n - 1) / 2;  // For 0..n-1
    let actual_sum: i32 = nums.iter().sum();
    expected_sum - actual_sum
}
```
*Caveat*: Can overflow for large numbers.

**Sorting Approach**:
```rust
fn find_missing_sort(nums: &mut [i32]) -> Option<i32> {
    nums.sort();
    for i in 0..nums.len() {
        if nums[i] != i as i32 {
            return Some(i as i32);
        }
    }
    Some(nums.len() as i32)
}
```
*Time: O(n log n), Space: O(1) for in-place sort*

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Database Integrity Checks**
   - Detecting missing sequence numbers in auto-increment IDs
   - Finding gaps in transaction logs

2. **Network Protocols**
   - Identifying missing packet sequence numbers
   - Detecting dropped frames in video streaming

3. **File System Recovery**
   - Finding missing block numbers
   - Detecting orphaned inode entries

4. **Distributed Systems**
   - Detecting missing messages in ordered delivery
   - Gap detection in event sourcing

5. **Data Validation**
   - Validating complete uploads (chunks)
   - Verifying lottery ticket sequences

### 6.2 Related Algorithms

| Algorithm | Relationship |
|-----------|--------------|
| [Find Unique Number](find_unique_number.md) | Same XOR technique, different problem |
| [Two Missing Numbers](N/A) | Extension using XOR and bit partitioning |
| Cyclic Sort | Alternative O(n) approach |

## 7. Comparison of Approaches

| Method | Time | Space | Overflow Risk | Handles Negatives |
|--------|------|-------|---------------|-------------------|
| XOR | O(n) | O(1) | No | Yes |
| Sum Formula | O(n) | O(1) | Yes | With care |
| Sorting | O(n log n) | O(1) | No | Yes |
| Hash Set | O(n) | O(n) | No | Yes |
| Binary Search | O(n log n) | O(1) | No | Yes |

## 8. Extensions

### Finding Two Missing Numbers

When two numbers are missing, we can:
1. Compute XOR of all (gives `m1 ^ m2`)
2. Find any set bit in the result
3. Partition numbers by that bit
4. Apply single-missing algorithm to each partition

```rust
fn find_two_missing(nums: &[i32], n: i32) -> (i32, i32) {
    // XOR all expected with all present
    let mut xor_all = 0;
    for i in 1..=n {
        xor_all ^= i;
    }
    for &num in nums {
        xor_all ^= num;
    }
    // xor_all = m1 ^ m2
    
    // Find rightmost set bit
    let set_bit = xor_all & (-xor_all);
    
    let mut m1 = 0;
    let mut m2 = 0;
    
    // Partition by the set bit
    for i in 1..=n {
        if i & set_bit != 0 { m1 ^= i; } else { m2 ^= i; }
    }
    for &num in nums {
        if num & set_bit != 0 { m1 ^= num; } else { m2 ^= num; }
    }
    
    (m1, m2)
}
```

## 9. Common Pitfalls

### Pitfall 1: Assuming 0-Based Sequence
```rust
// Bug: Assumes sequence starts at 0
fn buggy(nums: &[i32]) -> i32 {
    let n = nums.len() as i32;
    let expected = n * (n + 1) / 2;  // Sum 0 to n
    expected - nums.iter().sum::<i32>()
}

// Fix: Handle arbitrary ranges
fn correct(nums: &[i32]) -> i32 {
    let low = *nums.iter().min().unwrap();
    let high = *nums.iter().max().unwrap();
    // Use XOR approach with actual range
}
```

### Pitfall 2: Integer Overflow with Sum
```rust
// Bug: Can overflow for large sequences
fn sum_approach(nums: &[i32]) -> i32 {
    let n = nums.len() as i64 + 1;
    let expected = n * (n - 1) / 2;  // Can overflow
    // ...
}

// Fix: Use XOR (never overflows) or checked arithmetic
```

### Pitfall 3: Not Validating Input
```rust
// Bug: Crashes on empty input
fn buggy(nums: &[i32]) -> i32 {
    let low = *nums.iter().min().unwrap();  // Panics if empty!
}

// Fix: Validate first
fn correct(nums: &[i32]) -> Result<i32, String> {
    if nums.is_empty() {
        return Err("empty input".to_string());
    }
    // ...
}
```

## 10. References

1. Cormen, T. H., et al. (2022). *Introduction to Algorithms* (4th ed.). MIT Press.
2. Warren, H. S. (2012). *Hacker's Delight* (2nd ed.). Addison-Wesley.
3. Knuth, D. E. (2011). *The Art of Computer Programming, Vol. 4A*. Addison-Wesley.
