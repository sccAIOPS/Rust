# Bingo Sort

## 1. Overview

Bingo Sort is a variant of Selection Sort optimized for arrays with many duplicate values. Instead of finding one minimum at a time, it identifies the minimum value and moves ALL elements equal to that value to their final positions in one pass.

### Key Characteristics
- **Type**: Comparison-based, selection sort variant
- **In-place**: Yes
- **Stable**: No
- **Best for**: Arrays with many duplicates

## 2. Mathematical Foundation

### 2.1 Optimization Insight

In standard Selection Sort:
- Find minimum in each pass: O(n) comparisons
- Place one element: O(1) swap
- Total: O(n²) comparisons

In Bingo Sort with k distinct values:
- Each distinct value handled in one "round"
- All equal values placed together
- Better performance when k << n

### 2.2 Complexity Relation

| Scenario | Selection Sort | Bingo Sort |
|----------|---------------|------------|
| All distinct | O(n²) | O(n²) |
| k distinct (k << n) | O(n²) | O(n·k) |
| All equal | O(n²) | O(n) |

## 3. Algorithm Description

### 3.1 Pseudocode

```
BINGO_SORT(A)
    n ← length(A)
    if n ≤ 1 then return
    
    // Find maximum and minimum
    max_val ← maximum(A)
    next_val ← minimum(A)
    
    next_pos ← 0
    
    while next_val ≠ max_val do
        current_val ← next_val
        next_val ← max_val
        
        for i ← next_pos to n - 1 do
            if A[i] = current_val then
                swap(A[i], A[next_pos])
                next_pos ← next_pos + 1
            else if A[i] < next_val then
                next_val ← A[i]
    
    // Handle remaining max values
    // (they're already at the end)
```

### 3.2 Step-by-Step Example

Sorting `[3, 1, 3, 2, 1, 2, 3, 1]`:

```
Initial: [3, 1, 3, 2, 1, 2, 3, 1]
max_val = 3, next_val = 1, next_pos = 0

Round 1: current_val = 1, next_val = 3
  Scan and move all 1s:
  i=1: A[1]=1 → swap → [1, 3, 3, 2, 1, 2, 3, 1], next_pos=1
  i=4: A[4]=1 → swap → [1, 1, 3, 2, 3, 2, 3, 1], next_pos=2
  i=7: A[7]=1 → swap → [1, 1, 1, 2, 3, 2, 3, 3], next_pos=3
  Also track: next_val = 2 (smallest > 1)

After Round 1: [1, 1, 1, 2, 3, 2, 3, 3]
              └───────┘ sorted

Round 2: current_val = 2, next_val = 3
  Scan from next_pos=3:
  i=3: A[3]=2 → already in place, next_pos=4
  i=5: A[5]=2 → swap → [1, 1, 1, 2, 2, 3, 3, 3], next_pos=5
  next_val = 3

After Round 2: [1, 1, 1, 2, 2, 3, 3, 3]
              └───────────┘ sorted

Round 3: current_val = 3 = max_val → DONE

Final: [1, 1, 1, 2, 2, 3, 3, 3]
```

## 4. Complexity Analysis

| Case | Time | Condition |
|------|------|-----------|
| **Best** | O(n) | All elements equal |
| **Average** | O(n·k) | k distinct values |
| **Worst** | O(n²) | All elements distinct |
| **Space** | O(1) | In-place |

## 5. Implementation

```rust
pub fn bingo_sort<T: Ord + Clone>(arr: &mut [T]) {
    let n = arr.len();
    if n <= 1 {
        return;
    }

    // Find max and min
    let max_val = arr.iter().max().unwrap().clone();
    let mut next_val = arr.iter().min().unwrap().clone();
    
    let mut next_pos = 0;

    while next_val != max_val {
        let current_val = next_val.clone();
        next_val = max_val.clone();

        for i in next_pos..n {
            if arr[i] == current_val {
                arr.swap(i, next_pos);
                next_pos += 1;
            } else if arr[i] < next_val {
                next_val = arr[i].clone();
            }
        }
    }
}
```

### 5.1 Optimized Version

```rust
pub fn bingo_sort_optimized<T: Ord + Clone>(arr: &mut [T]) {
    if arr.len() <= 1 {
        return;
    }

    let (mut min_idx, mut max_idx) = (0, 0);
    for i in 1..arr.len() {
        if arr[i] < arr[min_idx] {
            min_idx = i;
        }
        if arr[i] > arr[max_idx] {
            max_idx = i;
        }
    }

    let max_val = arr[max_idx].clone();
    let mut bingo = arr[min_idx].clone();
    
    let mut next_pos = 0;
    let mut end_pos = arr.len() - 1;

    while bingo != max_val {
        let current_bingo = bingo.clone();
        bingo = max_val.clone();

        let mut i = next_pos;
        while i <= end_pos {
            if arr[i] == current_bingo {
                arr.swap(i, next_pos);
                next_pos += 1;
            } else if arr[i] < bingo {
                bingo = arr[i].clone();
            }
            i += 1;
        }
    }
}
```

## 6. Comparison with Selection Sort

| Aspect | Selection Sort | Bingo Sort |
|--------|---------------|------------|
| Passes | n | k (distinct values) |
| Best case | O(n²) | O(n) |
| With duplicates | Same work | Less work |
| Elements moved/pass | 1 | All of one value |

## 7. Advantages and Limitations

### Advantages
| Advantage | Description |
|-----------|-------------|
| Efficient for duplicates | O(n·k) when k << n |
| In-place | O(1) extra space |
| Simple | Easy to understand |
| Cache-friendly | Sequential access |

### Limitations
| Limitation | Description |
|------------|-------------|
| O(n²) worst case | When all elements distinct |
| Not stable | Order of equal elements may change |
| Not adaptive | Doesn't benefit from partial sorting |

## 8. Applications

1. **Sorting with few distinct keys**: Student grades, categories
2. **Color sorting**: Limited color palette
3. **Priority buckets**: Limited priority levels
4. **Age grouping**: Demographic data

## 9. References

1. Selection Sort variants and optimizations
2. Sorting algorithms for data with duplicates

## 10. Source Code

**Implementation**: [src/sorting/bingo_sort.rs](../../src/sorting/bingo_sort.rs)
