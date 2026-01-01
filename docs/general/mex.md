# MEX (Minimum Excludant)

## 1. Overview

The Minimum Excludant (MEX) is a fundamental concept in combinatorial game theory. Given a set of non-negative integers, MEX returns the smallest non-negative integer not present in the set. This operation is crucial for computing Grundy numbers (nimbers) in impartial games and appears in various algorithmic problems.

The MEX function is central to the Sprague-Grundy theorem, which provides a complete analysis of impartial combinatorial games.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a set $S \subseteq \mathbb{N}_0$ (non-negative integers), find the minimum excludant:

$$\text{mex}(S) = \min\{n \in \mathbb{N}_0 : n \notin S\}$$

In other words, find the smallest non-negative integer not in $S$.

### 2.2 Mathematical Model

**Input**: Set $S = \{s_1, s_2, ..., s_n\}$ where $s_i \in \mathbb{N}_0$

**Output**: Integer $m \in \mathbb{N}_0$ such that:
1. $m \notin S$
2. $\forall k < m: k \in S$

**Properties**:
- $\text{mex}(\emptyset) = 0$
- $\text{mex}(\{0\}) = 1$
- $\text{mex}(\{1, 2, 3, ...\}) = 0$
- $\text{mex}(S) \leq |S|$ (pigeonhole principle)
- If $S = \{0, 1, 2, ..., n\}$, then $\text{mex}(S) = n + 1$

### 2.3 Sprague-Grundy Theorem Context

In game theory, the Grundy number (nimber) of a position is:

$$g(P) = \text{mex}\{g(P') : P' \text{ is a position reachable from } P\}$$

The MEX operation determines whether a position is winning or losing:
- $\text{mex}(S) = 0$: Losing position (all moves lead to winning positions)
- $\text{mex}(S) > 0$: Winning position

## 3. Algorithm Description

### 3.1 Intuition

**Naive approach**: Start from 0 and check each integer sequentially until finding one not in the set.

**Optimized approach**: 
1. Use a hash set for O(1) membership testing
2. Iterate from 0 upward checking presence
3. Return the first missing value

**Key insight**: The MEX value is at most $|S|$ (if $S = \{0,1,...,n-1\}$, MEX is $n$), so we only need to check values up to the set size.

### 3.2 Pseudocode

```
function MEX(set):
    // Convert to hash set for O(1) lookup
    hash_set = new HashSet(set)
    
    // Start from 0 and find first missing value
    mex = 0
    while mex in hash_set:
        mex = mex + 1
    
    return mex

// Optimized: Since MEX ≤ |S|, we can limit search
function MEXOptimized(array):
    n = length(array)
    present = new boolean array of size n+1, initialized to false
    
    // Mark present values (ignore values ≥ n+1)
    for value in array:
        if 0 <= value <= n:
            present[value] = true
    
    // Find first false
    for i from 0 to n:
        if not present[i]:
            return i
    
    return n + 1  // All values 0..n are present

// In-place using array (for specific range)
function MEXInPlace(array):
    n = length(array)
    
    // Place each element at its index position
    for i from 0 to n-1:
        while 0 <= array[i] < n and array[i] != i:
            swap(array[i], array[array[i]])
    
    // Find first mismatch
    for i from 0 to n-1:
        if array[i] != i:
            return i
    
    return n
```

### 3.3 Step-by-Step Example

**Example 1**: Find MEX of `{1, 2, 4, 7, 9}`

```
Set: {1, 2, 4, 7, 9}

Check 0: 0 ∉ set → MEX = 0

Result: 0
```

**Example 2**: Find MEX of `{0, 1, 2, 4, 5}`

```
Set: {0, 1, 2, 4, 5}

Check 0: 0 ∈ set
Check 1: 1 ∈ set
Check 2: 2 ∈ set
Check 3: 3 ∉ set → MEX = 3

Result: 3
```

**Example 3**: Find MEX of `{0, 1, 2, 3, 4}`

```
Set: {0, 1, 2, 3, 4}

Check 0: 0 ∈ set
Check 1: 1 ∈ set
Check 2: 2 ∈ set
Check 3: 3 ∈ set
Check 4: 4 ∈ set
Check 5: 5 ∉ set → MEX = 5

Result: 5
```

**Example 4**: Game theory application (Nim-like game)

```
Game positions reachable from current state have Grundy numbers: {0, 1, 3}

MEX({0, 1, 3}) = 2

Grundy number of current position = 2 (winning position)
```

## 4. Complexity Analysis

### 4.1 Time Complexity

**Hash Set Approach**:
- Building hash set: $O(n)$
- Finding MEX: $O(n)$ worst case (if MEX = $n$)
- **Total**: $O(n)$

**Array Marking Approach**:
- Marking present elements: $O(n)$
- Finding first missing: $O(n)$
- **Total**: $O(n)$

**In-Place Approach**:
- Cyclic swapping: $O(n)$ amortized (each element swapped at most once)
- Finding mismatch: $O(n)$
- **Total**: $O(n)$

### 4.2 Space Complexity

**Hash Set**: $O(n)$ for storing set

**Array Marking**: $O(n)$ for boolean array

**In-Place**: $O(1)$ auxiliary space (modifies input)

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::collections::HashSet;

// Basic hash set approach
fn mex(values: &[usize]) -> usize {
    let set: HashSet<usize> = values.iter().copied().collect();
    
    (0..)
        .find(|&i| !set.contains(&i))
        .unwrap()
}

// Optimized with size bound
fn mex_optimized(values: &[usize]) -> usize {
    let n = values.len();
    let mut present = vec![false; n + 1];
    
    for &value in values {
        if value <= n {
            present[value] = true;
        }
    }
    
    present.iter()
        .position(|&p| !p)
        .unwrap_or(n + 1)
}

// Functional style
fn mex_functional(values: &[usize]) -> usize {
    let set: HashSet<_> = values.iter().collect();
    (0..).find(|i| !set.contains(i)).unwrap()
}

// For game theory: MEX of iterator
fn mex_iter<I>(values: I) -> usize 
where
    I: IntoIterator<Item = usize>,
{
    let set: HashSet<_> = values.into_iter().collect();
    (0..).find(|i| !set.contains(i)).unwrap()
}

// Bounded version (when you know upper limit)
fn mex_bounded(values: &[usize], max_value: usize) -> usize {
    let mut present = vec![false; max_value + 1];
    
    for &value in values {
        if value <= max_value {
            present[value] = true;
        }
    }
    
    (0..=max_value)
        .find(|&i| !present[i])
        .unwrap_or(max_value + 1)
}

// XOR-based MEX for special cases (when values form certain patterns)
fn mex_xor_special(values: &[usize]) -> usize {
    // For certain game theory applications
    values.iter().fold(0, |acc, &x| acc ^ x)
}
```

**Key Features**:
- Iterator-based for elegance
- `HashSet` for O(1) lookups
- `Vec<bool>` for memory efficiency
- Generic implementations
- Functional style with `find()`

### 5.2 Edge Cases

1. **Empty set**: MEX = 0
2. **Set containing only 0**: MEX = 1
3. **No gaps**: MEX = size + 1
4. **Large gap**: MEX = first gap
5. **All large values**: MEX = 0
6. **Duplicates**: Should be handled (use set or ignore)
7. **Negative numbers**: Typically undefined, or ignore negatives
8. **Very large values**: Optimization needed to avoid huge arrays

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Combinatorial Game Theory**
- Nim and Nim-like games
- Grundy number calculation
- Game position analysis
- Optimal strategy determination

**2. Resource Allocation**
- Finding smallest available ID
- Port number allocation
- Process ID assignment
- Database key generation

**3. Scheduling**
- Finding first available time slot
- Resource assignment
- Task prioritization
- Conflict resolution

**4. Network Protocols**
- Sequence number gaps detection
- Packet loss identification
- Missing frame detection
- Protocol state management

**5. Database Systems**
- Finding gaps in sequences
- Auto-increment key management
- Sparse index optimization
- Data consistency checks

**6. Competitive Programming**
- Game theory problems
- Array manipulation
- Range queries
- Dynamic programming states

### 6.2 Related Algorithms

**Game Theory**:
- **Sprague-Grundy Theorem**: Uses MEX for computing Grundy numbers
- **Nim Game**: XOR of pile sizes = Grundy number
- **Partisan Games**: More complex than MEX-solvable games

**Related Problems**:
- **First Missing Positive**: Find smallest missing positive integer
- **Find Disappeared Numbers**: All missing numbers in range
- **Missing Number**: Single missing number (simpler)
- **Gap Problems**: Finding gaps in sequences

**Optimization Techniques**:
- **Cyclic Sort**: For in-place MEX finding
- **Bucket Sort**: For range-limited values
- **Bit Manipulation**: For certain special cases

**When to Use**:
- **Hash Set**: General case, unknown range
- **Array Marking**: Known small range
- **In-Place**: Memory constrained
- **XOR**: Special nim-like games

## 7. References

### Academic Papers
1. Sprague, R. (1935). "Über mathematische Kampfspiele". *Tôhoku Mathematical Journal*, 41, 438-444.
2. Grundy, P.M. (1939). "Mathematics and Games". *Eureka*, 2, 6-8.
3. Conway, J.H. (1976). *On Numbers and Games*. Academic Press.
4. Berlekamp, E.R., et al. (2001). *Winning Ways for Your Mathematical Plays* (2nd ed.). A K Peters.

### Books
1. Ferguson, T.S. (2014). *Game Theory* (2nd ed.). UCLA.
2. Albert, M.H., et al. (2007). *Lessons in Play: An Introduction to Combinatorial Game Theory*. A K Peters.
3. Siegel, A.N. (2013). *Combinatorial Game Theory*. American Mathematical Society.

### Online Resources
1. [MEX Function - Wikipedia](https://en.wikipedia.org/wiki/Mex_(mathematics))
2. [Sprague-Grundy Theorem](https://cp-algorithms.com/game_theory/sprague-grundy-nim.html)
3. [Game Theory in Programming](https://www.geeksforgeeks.org/combinatorial-game-theory-set-1-introduction/)
4. [First Missing Positive - LeetCode](https://leetcode.com/problems/first-missing-positive/)

### Implementation
- Source: `src/general/mex.rs`
- Tests: Included in source file under `#[cfg(test)] mod tests`
