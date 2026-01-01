# Two Sum

## 1. Overview

The Two Sum problem is a fundamental algorithmic challenge: given an array of numbers and a target sum, find two distinct numbers in the array that add up to the target. This problem appears frequently in coding interviews and serves as an excellent introduction to hash table optimization.

While a brute-force approach requires $O(n^2)$ time, using a hash map reduces this to $O(n)$, demonstrating the power of trading space for time.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A = [a_1, a_2, ..., a_n]$ of integers and a target value $T$, find indices $i$ and $j$ where $i \neq j$ such that:

$$a_i + a_j = T$$

If multiple pairs exist, typically any valid pair is acceptable. If no pair exists, return an indication of failure (e.g., empty result, `None`, or error).

### 2.2 Mathematical Model

**Input**: 
- Array $A[1..n]$ of integers
- Target sum $T \in \mathbb{Z}$

**Output**: 
- Tuple $(i, j)$ where $1 \leq i < j \leq n$ and $A[i] + A[j] = T$
- Or indication that no such pair exists

**Uniqueness**: The problem may have:
- No solution
- Exactly one solution
- Multiple solutions (return any one)

**Constraint**: $i \neq j$ (cannot use the same element twice)

### 2.3 Correctness Proof

**Theorem**: The hash map approach correctly identifies a valid pair or determines none exists.

**Proof**:

**Hash Map Method**:
1. For each element $a_i$, we check if complement $c_i = T - a_i$ exists in the map
2. If $c_i$ exists at index $j < i$, then $a_j + a_i = T$ ✓
3. If we never find a complement, no valid pair exists
4. **Why it works**: For any pair $(a_i, a_j)$ with $i < j$ that sums to $T$:
   - When processing $a_j$, we check for $T - a_j = a_i$
   - Since $i < j$, we've already added $a_i$ to the map
   - Therefore, we will find the pair

**Completeness**: Every valid pair will be detected because we examine all elements and store complements. ∎

## 3. Algorithm Description

### 3.1 Intuition

**Naive Approach** (Don't use):
- Check every pair of elements: for each $i$, check all $j > i$
- Time: $O(n^2)$, Space: $O(1)$

**Optimized Approach** (Use this):
- For each element $x$, we need to find if $T - x$ exists
- Use a hash map to store elements we've seen
- As we scan, check if the complement is in the map
- If yes: found the pair!
- If no: add current element to map and continue
- Time: $O(n)$, Space: $O(n)$

**Key insight**: Instead of checking all pairs, look up complements directly.

### 3.2 Pseudocode

```
function TwoSum(array, target):
    map = new HashMap()  // value -> index
    
    for i from 0 to len(array) - 1:
        complement = target - array[i]
        
        if map.contains(complement):
            // Found the pair!
            j = map.get(complement)
            return (j, i)  // or (i, j) depending on convention
        
        // Store current element for future lookups
        map.put(array[i], i)
    
    // No pair found
    return null

// Variant: Find all pairs (not just first)
function TwoSumAllPairs(array, target):
    map = new HashMap()
    results = []
    
    for i from 0 to len(array) - 1:
        complement = target - array[i]
        
        if map.contains(complement):
            j = map.get(complement)
            results.append((j, i))
        
        map.put(array[i], i)
    
    return results

// Variant: Two pointers (requires sorted array)
function TwoSumSorted(sorted_array, target):
    left = 0
    right = len(sorted_array) - 1
    
    while left < right:
        sum = sorted_array[left] + sorted_array[right]
        
        if sum == target:
            return (left, right)
        else if sum < target:
            left = left + 1  // Need larger sum
        else:
            right = right - 1  // Need smaller sum
    
    return null
```

### 3.3 Step-by-Step Example

Find two numbers that sum to **9** in: `[2, 7, 11, 15]`

**Hash Map Approach**:

```
Array: [2, 7, 11, 15], Target: 9
Map: {}

Step 1: i=0, array[0]=2
  complement = 9 - 2 = 7
  7 not in map
  map.put(2, 0)
  Map: {2: 0}

Step 2: i=1, array[1]=7
  complement = 9 - 7 = 2
  2 IS in map at index 0
  Found pair: (0, 1)
  Return [0, 1]
  
Result: indices [0, 1] → values [2, 7] ✓
```

**Another Example**: Target **6** in `[3, 2, 4]`

```
Array: [3, 2, 4], Target: 6
Map: {}

Step 1: i=0, array[0]=3
  complement = 6 - 3 = 3
  3 not in map (can't use same index)
  Map: {3: 0}

Step 2: i=1, array[1]=2
  complement = 6 - 2 = 4
  4 not in map
  Map: {3: 0, 2: 1}

Step 3: i=2, array[2]=4
  complement = 6 - 4 = 2
  2 IS in map at index 1
  Found pair: (1, 2)
  Return [1, 2]
  
Result: indices [1, 2] → values [2, 4] ✓
```

## 4. Complexity Analysis

### 4.1 Time Complexity

**Hash Map Approach**:
- Single pass through array: $O(n)$
- Hash map operations (insert, lookup): $O(1)$ average case
- **Total**: $O(n)$ average case, $O(n^2)$ worst case (hash collisions)

**Two Pointers Approach** (sorted array):
- Sorting: $O(n \log n)$
- Two pointer scan: $O(n)$
- **Total**: $O(n \log n)$

**Brute Force**:
- Nested loops checking all pairs: $O(n^2)$

**Comparison**: Hash map is optimal for unsorted data.

### 4.2 Space Complexity

**Hash Map Approach**:
- Worst case: store all $n$ elements: $O(n)$
- Best case (early return): $O(1)$ to $O(n)$

**Two Pointers Approach**:
- If sorting in-place: $O(1)$ auxiliary space
- If creating sorted copy: $O(n)$

**Brute Force**:
- $O(1)$ auxiliary space

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::collections::HashMap;

// Basic implementation returning indices
fn two_sum(nums: &[i32], target: i32) -> Option<(usize, usize)> {
    let mut map = HashMap::new();
    
    for (i, &num) in nums.iter().enumerate() {
        let complement = target - num;
        
        if let Some(&j) = map.get(&complement) {
            return Some((j, i));
        }
        
        map.insert(num, i);
    }
    
    None
}

// Return as Vec<usize> (LeetCode style)
fn two_sum_vec(nums: &[i32], target: i32) -> Vec<usize> {
    let mut map = HashMap::new();
    
    for (i, &num) in nums.iter().enumerate() {
        let complement = target - num;
        
        if let Some(&j) = map.get(&complement) {
            return vec![j, i];
        }
        
        map.insert(num, i);
    }
    
    vec![] // No solution
}

// Find all pairs
fn two_sum_all_pairs(nums: &[i32], target: i32) -> Vec<(usize, usize)> {
    let mut map = HashMap::new();
    let mut results = Vec::new();
    
    for (i, &num) in nums.iter().enumerate() {
        let complement = target - num;
        
        if let Some(&j) = map.get(&complement) {
            results.push((j, i));
        }
        
        map.insert(num, i);
    }
    
    results
}

// Two pointers approach (requires sorted array)
fn two_sum_sorted(nums: &[i32], target: i32) -> Option<(usize, usize)> {
    let mut left = 0;
    let mut right = nums.len().saturating_sub(1);
    
    while left < right {
        let sum = nums[left] + nums[right];
        
        match sum.cmp(&target) {
            std::cmp::Ordering::Equal => return Some((left, right)),
            std::cmp::Ordering::Less => left += 1,
            std::cmp::Ordering::Greater => right -= 1,
        }
    }
    
    None
}

// Generic over numeric types
fn two_sum_generic<T>(nums: &[T], target: T) -> Option<(usize, usize)>
where
    T: std::hash::Hash + Eq + Copy + std::ops::Sub<Output = T>,
{
    let mut map = HashMap::new();
    
    for (i, &num) in nums.iter().enumerate() {
        let complement = target - num;
        
        if let Some(&j) = map.get(&complement) {
            return Some((j, i));
        }
        
        map.insert(num, i);
    }
    
    None
}
```

**Key Rust Features**:
- `HashMap` for O(1) lookups
- `Option<T>` for representing no solution
- `if let` for clean pattern matching
- `enumerate()` for index tracking
- Generic implementation with trait bounds
- `saturating_sub()` to prevent underflow

### 5.2 Edge Cases

1. **Empty array**: No solution possible
2. **Single element**: No solution (need two distinct elements)
3. **Two elements**: Check if they sum to target
4. **Duplicate values**: Handle correctly (same value at different indices can form pair)
5. **No solution exists**: Return `None` or empty vector
6. **Multiple solutions**: Return first found (or all, depending on requirements)
7. **Same element twice**: If `array[i] * 2 == target`, need two instances
8. **Overflow**: In languages without checked arithmetic, `target - num` might overflow

**Example edge cases**:
```rust
// Same element used twice (should fail)
two_sum(&[1, 2, 3], 2); // Should not use 1+1 from index 0

// Same value at different indices (should succeed)
two_sum(&[3, 3], 6); // Should return (0, 1) ✓

// Negative numbers
two_sum(&[-1, -2, -3, -4], -5); // Should work: indices (1, 2) → -2 + -3
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Financial Applications**
- Finding transactions that match a total
- Reconciliation of accounting records
- Detecting pairs of credits/debits
- Portfolio optimization (finding complementary assets)

**2. E-commerce**
- Finding product pairs that fit a budget
- Combo deals and bundling recommendations
- Coupon application optimization
- Gift card balance matching

**3. Data Analysis**
- Correlation analysis (finding related data points)
- Anomaly detection (unexpected pairs)
- Pattern matching in datasets
- Duplicate detection with tolerances

**4. Cryptography**
- Hash collision detection
- Finding additive inverses
- Subset sum components
- Partition problems

**5. Game Development**
- Resource combination mechanics
- Crafting systems (combining items)
- Puzzle solving (matching pairs)
- Achievement systems

**6. Networking**
- Packet pairing in protocols
- Load balancing (distributing requests)
- Connection matching
- Resource allocation

### 6.2 Related Algorithms

**Direct Extensions**:
- **Three Sum**: Find three numbers that sum to target ($O(n^2)$)
- **Four Sum**: Find four numbers that sum to target ($O(n^3)$ or $O(n^2)$ with optimization)
- **K Sum**: Generalization to K numbers
- **Two Sum (All Pairs)**: Find all unique pairs
- **Two Sum (Closest)**: Find pair with sum closest to target

**Related Problems**:
- **Two Difference**: Find two numbers with specific difference
- **Two Product**: Find two numbers with specific product
- **Subarray Sum**: Contiguous subarray summing to target
- **Subset Sum**: Any subset summing to target (NP-complete)

**Techniques**:
- **Hash Map**: O(n) time, O(n) space
- **Two Pointers**: O(n log n) time (with sort), O(1) space
- **Binary Search**: For each element, binary search for complement
- **Meet in the Middle**: For exponential problems

**When to Use**:
- **Hash Map**: Unsorted data, need O(n) time
- **Two Pointers**: Sorted data, minimize space
- **Brute Force**: Tiny datasets, simplicity preferred

## 7. References

### Academic Papers
1. Cormode, G., & Muthukrishnan, S. (2005). "An Improved Data Stream Summary: The Count-Min Sketch and its Applications". *Journal of Algorithms*, 55(1), 58-75.
2. Fredman, M.L., et al. (1984). "The Complexity of Maintaining an Array and Computing its Partial Sums". *Journal of the ACM*, 31(3), 638-648.

### Books
1. Cormen, T.H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Chapter 11: Hash Tables.
2. Skiena, S.S. (2008). *The Algorithm Design Manual* (2nd ed.). Springer. Section 12.1: Hashing.
3. Aziz, A., et al. (2015). *Elements of Programming Interviews in Java*. CreateSpace. Problem 12.1: Test for palindromic permutations.

### Online Resources
1. [Two Sum - LeetCode](https://leetcode.com/problems/two-sum/)
2. [Two Sum Explained - GeeksforGeeks](https://www.geeksforgeeks.org/given-an-array-a-and-a-number-x-check-for-pair-in-a-with-sum-as-x/)
3. [Hash Table Applications](https://en.wikipedia.org/wiki/Hash_table)
4. [Two Pointers Technique](https://leetcode.com/articles/two-pointer-technique/)

### Implementation
- Source: `src/general/two_sum.rs`
- Tests: Included in source file under `#[cfg(test)] mod tests`
