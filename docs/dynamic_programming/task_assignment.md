# Task Assignment (Bitmask DP)

## 1. Overview

The Task Assignment problem assigns tasks to workers to minimize total cost using bitmask dynamic programming. This is also known as the Assignment Problem and demonstrates state compression DP techniques.

**File**: `src/dynamic_programming/task_assignment.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- $n$ tasks and $n$ workers
- Cost matrix $C$ where $C[i][j]$ = cost of worker $i$ doing task $j$
- Each worker does exactly one task
- Each task assigned to exactly one worker

Find: Permutation $\sigma$ minimizing $\sum_{i=1}^{n} C[i][\sigma(i)]$

### 2.2 State Representation

Use bitmask to represent which tasks are assigned:
- Bit $j$ set → task $j$ is assigned
- State with $k$ bits set → first $k$ workers assigned

### 2.3 Recurrence

Let $dp[mask]$ = minimum cost to assign tasks indicated by $mask$ to first $popcount(mask)$ workers.

Let $k = popcount(mask)$ (current worker index).

$$dp[mask] = \min_{j \in mask} (dp[mask \setminus \{j\}] + C[k-1][j])$$

Base case: $dp[0] = 0$

## 3. Algorithm Description

### 3.1 Pseudocode

```
FUNCTION min_cost_assignment(cost)
    n ← len(cost)
    dp[0..2^n] ← ∞
    dp[0] ← 0
    
    FOR mask ← 1 TO 2^n - 1 DO
        worker ← popcount(mask) - 1
        FOR task ← 0 TO n-1 DO
            IF bit task is set in mask THEN
                prev_mask ← mask XOR (1 << task)
                dp[mask] ← min(dp[mask], dp[prev_mask] + cost[worker][task])
            END IF
        END FOR
    END FOR
    
    RETURN dp[2^n - 1]
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input** (3 workers, 3 tasks):
```
cost = [[9, 2, 7],
        [6, 4, 3],
        [5, 8, 1]]
```

**Processing**:
```
mask=001 (task 0): worker 0
  dp[001] = dp[000] + cost[0][0] = 0 + 9 = 9

mask=010 (task 1): worker 0
  dp[010] = dp[000] + cost[0][1] = 0 + 2 = 2

mask=011 (tasks 0,1): worker 1
  dp[011] = min(dp[001] + cost[1][1], dp[010] + cost[1][0])
         = min(9 + 4, 2 + 6) = min(13, 8) = 8

mask=100 (task 2): worker 0
  dp[100] = dp[000] + cost[0][2] = 0 + 7 = 7

mask=101 (tasks 0,2): worker 1
  dp[101] = min(dp[001] + cost[1][2], dp[100] + cost[1][0])
         = min(9 + 3, 7 + 6) = min(12, 13) = 12

mask=110 (tasks 1,2): worker 1
  dp[110] = min(dp[010] + cost[1][2], dp[100] + cost[1][1])
         = min(2 + 3, 7 + 4) = min(5, 11) = 5

mask=111 (all tasks): worker 2
  dp[111] = min(dp[011] + cost[2][2], dp[101] + cost[2][1], dp[110] + cost[2][0])
         = min(8 + 1, 12 + 8, 5 + 5) = min(9, 20, 10) = 9
```

**Result**: 9

**Optimal Assignment**: 
- Worker 0 → Task 1 (cost 2)
- Worker 1 → Task 2 (cost 3)
- Worker 2 → Task 0 (cost 5) [Wait, let's verify]

Actually: Worker 0→1, Worker 1→2, Worker 2→0: 2+3+5=10
Let's check 8+1=9: mask=011 means tasks 0,1 done.
- dp[011]=8 came from worker 0→task 1 (2) + worker 1→task 0 (6) = 8
- Then worker 2→task 2 (1)
- Total: 2+6+1 = 9 ✓

## 4. Complexity Analysis

### 4.1 Time Complexity
- **O(n · 2^n)** - Iterate all masks, check n bits each

### 4.2 Space Complexity
- **O(2^n)** for DP table

### 4.3 Comparison with Hungarian Algorithm

| Method | Time | Space | n limit |
|--------|------|-------|---------|
| Bitmask DP | O(n · 2^n) | O(2^n) | ~20 |
| Hungarian | O(n³) | O(n²) | ~10,000 |

Bitmask DP is simpler but limited to small n.

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
use std::collections::HashMap;

pub fn min_cost_task_assignment(cost: &[Vec<i32>]) -> i32 {
    let n = cost.len();
    let full_mask = (1 << n) - 1;
    
    // Using HashMap for sparse states (alternative: Vec)
    let mut dp: HashMap<usize, i32> = HashMap::new();
    dp.insert(0, 0);
    
    for mask in 1..=full_mask {
        let worker = mask.count_ones() as usize - 1;
        let mut min_cost = i32::MAX;
        
        for task in 0..n {
            if (mask & (1 << task)) != 0 {
                let prev_mask = mask ^ (1 << task);
                if let Some(&prev_cost) = dp.get(&prev_mask) {
                    min_cost = min_cost.min(prev_cost + cost[worker][task]);
                }
            }
        }
        
        dp.insert(mask, min_cost);
    }
    
    dp[&full_mask]
}
```

### 5.2 Memory Optimization

For very limited memory, use meet-in-the-middle:
- Split tasks into two halves
- Compute all 2^(n/2) states for each half
- Combine compatible halves

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| n = 0 | Return 0 |
| n = 1 | Return cost[0][0] |
| Negative costs | Works correctly |
| n > 20 | Use Hungarian algorithm instead |

## 6. Bit Manipulation Tricks

```rust
// Count set bits
let worker = mask.count_ones() as usize - 1;

// Check if bit j is set
let is_set = (mask & (1 << j)) != 0;

// Clear bit j
let prev_mask = mask ^ (1 << j);

// Iterate over set bits efficiently
let mut m = mask;
while m != 0 {
    let j = m.trailing_zeros() as usize;
    // process task j
    m &= m - 1;  // clear lowest set bit
}
```

## 7. Applications

1. **Job Scheduling**: Assign jobs to machines
2. **Resource Allocation**: Distribute resources to projects
3. **Transportation**: Vehicle routing
4. **Sports**: Team matching in tournaments

## 8. Variants

### 8.1 Weighted Bipartite Matching

Same problem, different formulation:
- Workers and tasks are nodes in bipartite graph
- Edges have weights (costs)
- Find minimum weight perfect matching

### 8.2 Maximum Assignment

Change min to max for maximum profit assignment.

### 8.3 Partial Assignment

Not all workers/tasks must be matched:
- Modify to allow unassigned states
- Add dummy costs for unassigned

## 9. Path Reconstruction

Track which task was assigned:

```rust
fn reconstruct(cost: &[Vec<i32>], dp: &HashMap<usize, i32>) -> Vec<usize> {
    let n = cost.len();
    let mut assignment = vec![0; n];
    let mut mask = (1 << n) - 1;
    
    for worker in (0..n).rev() {
        for task in 0..n {
            if (mask & (1 << task)) != 0 {
                let prev_mask = mask ^ (1 << task);
                if dp.get(&mask) == Some(&(dp.get(&prev_mask).unwrap_or(&i32::MAX) 
                                           + cost[worker][task])) {
                    assignment[worker] = task;
                    mask = prev_mask;
                    break;
                }
            }
        }
    }
    
    assignment
}
```

## 10. Related Problems

| Problem | Complexity |
|---------|------------|
| Traveling Salesman | O(n² · 2^n) bitmask DP |
| Hamiltonian Path | O(n · 2^n) bitmask DP |
| Set Cover | O(3^n) subset DP |
| Graph Coloring | O(n · 2^n) bitmask DP |

## 11. References

1. [Hungarian Algorithm](https://en.wikipedia.org/wiki/Hungarian_algorithm)
2. [LeetCode - Minimum XOR Sum of Two Arrays](https://leetcode.com/problems/minimum-xor-sum-of-two-arrays/)
3. CLRS - "Introduction to Algorithms" - Bipartite Matching
