# Moore's Voting Algorithm

## 1. Overview

Moore's Voting Algorithm is an elegant linear-time, constant-space algorithm for finding the **majority element** in an array—the element that appears more than $\lfloor n/2 \rfloor$ times. Invented by Robert S. Boyer and J Strother Moore in 1981, it's remarkably simple yet mathematically beautiful.

The algorithm exploits a key insight: if we pair up different elements and "cancel" them out, the majority element will always survive due to its numerical advantage.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $A[0..n-1]$, find element $m$ such that:

$$\text{count}(m) > \lfloor n/2 \rfloor$$

The majority element, if it exists, must appear **strictly more than half** the time.

### 2.2 Core Theorem

**Boyer-Moore Majority Vote Theorem:**

If a majority element $m$ exists, and we maintain a counter that:
- Increments when seeing $m$
- Decrements when seeing any other element

Then after processing all elements, the counter will be positive.

**Proof Sketch:**
- Let $m$ appear $k$ times where $k > n/2$
- Non-majority elements appear at most $n - k < n/2$ times
- Net count for $m$: $k - (n-k) = 2k - n > 0$

### 2.3 Algorithm Principle

The algorithm uses a "voting" metaphor:
1. **Candidate Selection (Phase 1):** Elements "vote" - same element increases count, different decreases
2. **Verification (Phase 2):** Confirm candidate is actually majority

**Key Insight:** When counter reaches 0, we can discard all elements seen so far because:
- If majority was in discarded prefix, at least as many non-majority were too
- Majority must still be majority in remaining suffix

## 3. Algorithm Description

### 3.1 Intuition

Imagine a battle where each element fights opposing elements:
- If two different elements meet, they destroy each other
- The last element standing (with soldiers remaining) is the candidate
- The majority element, having the largest army, will survive

### 3.2 Pseudocode

```
MOORE-VOTING(A):
    // Phase 1: Find candidate
    candidate ← A[0]
    count ← 1
    
    for i ← 1 to n-1:
        if count = 0:
            candidate ← A[i]
            count ← 1
        else if A[i] = candidate:
            count ← count + 1
        else:
            count ← count - 1
    
    // Phase 2: Verify candidate (optional but recommended)
    count ← 0
    for i ← 0 to n-1:
        if A[i] = candidate:
            count ← count + 1
    
    if count > n/2:
        return candidate
    else:
        return NO_MAJORITY
```

### 3.3 Step-by-Step Example

**Array:** `[2, 2, 1, 1, 1, 2, 2]`  
**n = 7**, majority threshold = 3 (need > 3.5)

**Phase 1: Find Candidate**

| Index | Element | Candidate | Count | Action |
|-------|---------|-----------|-------|--------|
| 0 | 2 | 2 | 1 | Initialize |
| 1 | 2 | 2 | 2 | Match: increment |
| 2 | 1 | 2 | 1 | Different: decrement |
| 3 | 1 | 2 | 0 | Different: decrement |
| 4 | 1 | 1 | 1 | Count=0: new candidate |
| 5 | 2 | 1 | 0 | Different: decrement |
| 6 | 2 | 2 | 1 | Count=0: new candidate |

**Candidate = 2**, Count = 1

**Phase 2: Verify**

Count occurrences of 2: **4 times** out of 7  
4 > 3.5 ✓

**Result:** Majority element is **2**

### 3.4 Visual Representation

```
Array: [2, 2, 1, 1, 1, 2, 2]
        ^  ^  ^  ^  ^  ^  ^
        +1 +1 -1 -1  0 +1 +1  (relative to candidate)
        
Cancellation pairs:
  2 ←→ 1 (indices 2 vs 0)
  2 ←→ 1 (indices 3 vs 1)
  1 ←→ 2 (indices 4 vs 5)
  
Surviving: 2 (index 6) → candidate
Verification: 2 appears 4 times > 3 ✓
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Phase | Operations | Complexity |
|-------|------------|------------|
| Phase 1: Find candidate | n iterations, O(1) each | O(n) |
| Phase 2: Verify | n iterations, O(1) each | O(n) |
| **Total** | - | **O(n)** |

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Candidate variable | O(1) |
| Counter | O(1) |
| **Total** | **O(1)** |

### 4.3 Comparison with Other Approaches

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Hash map counting | O(n) | O(n) | Requires extra memory |
| Sorting | O(n log n) | O(1) or O(n) | Check middle element |
| Moore's Voting | O(n) | O(1) | Optimal |

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
/// Find the majority element in the array using Moore's Voting Algorithm.
/// Returns the majority element if it exists, None otherwise.
/// Majority element appears more than ⌊n/2⌋ times.
pub fn moore_voting<T: Eq + Copy + Ord>(arr: &[T]) -> Option<T> {
    if arr.is_empty() {
        return None;
    }

    // Phase 1: Find candidate
    let mut candidate = arr[0];
    let mut count: isize = 1;

    for &item in arr.iter().skip(1) {
        if count == 0 {
            candidate = item;
            count = 1;
        } else if item == candidate {
            count += 1;
        } else {
            count -= 1;
        }
    }

    // Phase 2: Verify candidate
    let actual_count = arr.iter().filter(|&&x| x == candidate).count();

    if actual_count > arr.len() / 2 {
        Some(candidate)
    } else {
        None
    }
}
```

**Design Decisions:**

1. **Trait Bounds:**
   - `Eq`: For equality comparison
   - `Copy`: For efficient value copying
   - `Ord`: Appears unnecessary in this algorithm (could be removed)

2. **Return Type:** `Option<T>` properly handles no-majority case

3. **Two-Pass Design:** Correct implementation with verification phase

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty array | Returns `None` |
| Single element | Returns that element (100% majority) |
| Two elements (same) | Returns that element |
| Two elements (different) | Returns `None` (no majority) |
| No majority exists | Returns `None` |
| All same elements | Returns that element |

### 5.3 Common Pitfalls

1. **Forgetting Verification:**
   ```rust
   // WRONG: Returns candidate without verification
   fn moore_wrong<T: Eq + Copy>(arr: &[T]) -> Option<T> {
       // ... Phase 1 only ...
       Some(candidate)  // May not be majority!
   }
   ```

2. **Off-by-One in Threshold:**
   ```rust
   // WRONG: >= instead of >
   if count >= arr.len() / 2 { ... }  // Should be >
   ```

3. **Integer Division Edge Case:**
   For odd n, `n/2` is floor division in Rust (correct behavior)

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Distributed Systems - Leader Election:**
   ```rust
   fn find_leader(votes: &[NodeId]) -> Option<NodeId> {
       moore_voting(votes)
   }
   ```
   In Byzantine fault tolerance, finding a leader that majority agrees on.

2. **Data Streaming - Heavy Hitter Detection:**
   Finding elements that appear frequently in network traffic.

3. **Database Systems:**
   - Mode calculation in analytics queries
   - Identifying dominant value in a column

4. **Machine Learning:**
   - Ensemble voting classifiers
   - Finding consensus label in crowdsourced annotations

### 6.2 Specific Applications

| Domain | Application |
|--------|-------------|
| Networking | Detecting DDoS source IPs |
| Social Media | Finding trending topics |
| Genomics | Finding dominant allele |
| Elections | Verifying majority winner |

## 7. Variants and Extensions

### 7.1 Finding Elements Appearing > n/k Times

**Boyer-Moore-Hunt Variant** for k candidates:

```rust
use std::collections::HashMap;

fn find_frequent<T: Eq + std::hash::Hash + Copy>(
    arr: &[T], 
    k: usize
) -> Vec<T> {
    let mut candidates: HashMap<T, isize> = HashMap::new();
    
    // Phase 1: Find up to k-1 candidates
    for &item in arr {
        if candidates.contains_key(&item) {
            *candidates.get_mut(&item).unwrap() += 1;
        } else if candidates.len() < k - 1 {
            candidates.insert(item, 1);
        } else {
            // Decrement all counters
            candidates.retain(|_, v| {
                *v -= 1;
                *v > 0
            });
        }
    }
    
    // Phase 2: Verify candidates
    let threshold = arr.len() / k;
    candidates.keys()
        .filter(|&&c| arr.iter().filter(|&&x| x == c).count() > threshold)
        .copied()
        .collect()
}
```

### 7.2 Streaming Variant

For data streams where we can't store all elements:

```rust
struct StreamingMajority<T> {
    candidate: Option<T>,
    count: isize,
}

impl<T: Eq + Copy> StreamingMajority<T> {
    fn new() -> Self {
        Self { candidate: None, count: 0 }
    }
    
    fn process(&mut self, item: T) {
        match &self.candidate {
            None => {
                self.candidate = Some(item);
                self.count = 1;
            }
            Some(c) if *c == item => {
                self.count += 1;
            }
            _ => {
                self.count -= 1;
                if self.count == 0 {
                    self.candidate = None;
                }
            }
        }
    }
    
    fn get_candidate(&self) -> Option<T> {
        self.candidate
    }
}
```

Note: Streaming variant cannot verify without second pass over data.

### 7.3 Parallel Version

For multi-core systems:

```rust
fn parallel_moore<T: Eq + Copy + Send + Sync>(arr: &[T]) -> Option<T> {
    // Split array into chunks
    // Find candidate in each chunk
    // Merge candidates: majority in whole must be majority in at least one half
    // Verify final candidate
    todo!()
}
```

## 8. Correctness Proof

### 8.1 Formal Proof

**Claim:** If majority element $m$ exists, Phase 1 outputs $m$ as candidate.

**Proof by contradiction:**

Assume Phase 1 outputs candidate $c \neq m$.

Let:
- $n$ = total elements
- $k$ = count of $m$ where $k > n/2$
- Final count for $c$ = positive value $p$

Each occurrence of $m$ either:
1. Incremented count when $m$ was candidate, or
2. Decremented count when $m$ was not candidate

Since $c$ is final candidate with positive count, elements "supporting" $c$ exceeded elements "opposing" it. But $m$ appears more than half the time, so $m$ must have more support than any other element, including $c$.

This contradicts $c$ being the final candidate with positive count.

Therefore, Phase 1 must output $m$ as candidate. ∎

## 9. References

1. Boyer, R. S., & Moore, J. S. (1981). "MJRTY—A Fast Majority Vote Algorithm." *Automated Reasoning: Essays in Honor of Woody Bledsoe*.
2. Cormen, T. H., et al. (2009). "Introduction to Algorithms" (3rd ed.). MIT Press. Problem 4-6.
3. Misra, J., & Gries, D. (1982). "Finding Repeated Elements." *Science of Computer Programming*.

---

**Implementation:** [`src/searching/moore_voting.rs`](../../src/searching/moore_voting.rs)  
**See Also:** [Linear Search](linear_search.md)
