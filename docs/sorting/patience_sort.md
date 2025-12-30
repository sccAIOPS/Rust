# Patience Sort

## 1. Overview

Patience Sort is a sorting algorithm inspired by the card game "Patience" (Solitaire). It works by dealing elements into piles according to specific rules, then merging the piles. It's notable for its connection to finding the Longest Increasing Subsequence (LIS).

### Key Characteristics
- **Type**: Comparison-based, pile-based sort
- **In-place**: No (requires O(n) space)
- **Stable**: Yes (with proper implementation)
- **Special property**: Can find LIS in O(n log n)

## 2. Mathematical Foundation

### 2.1 Card Game Rules

1. Cards are dealt one at a time
2. Each card goes on the leftmost pile where it's ≤ the top card
3. If no such pile exists, create a new pile
4. Final step: merge all piles (like merge sort)

### 2.2 LIS Connection

**Theorem**: The number of piles equals the length of the Longest Increasing Subsequence.

This makes Patience Sort useful for LIS computation in O(n log n) time.

### 2.3 Pile Property

At any point, the top cards of all piles form a decreasing sequence from left to right.

## 3. Algorithm Description

### 3.1 Pseudocode

```
PATIENCE_SORT(A)
    piles ← []
    
    // Deal cards into piles
    for each element x in A do
        // Binary search for leftmost pile with top ≥ x
        pile_index ← BINARY_SEARCH_PILE(piles, x)
        
        if pile_index = length(piles) then
            piles.append(new pile with x)
        else
            piles[pile_index].push(x)
    
    // Merge piles using min-heap
    result ← []
    heap ← MIN_HEAP()
    
    for each pile in piles do
        heap.insert((pile.top(), pile_index))
    
    while heap is not empty do
        (min_val, pile_idx) ← heap.extract_min()
        result.append(min_val)
        
        piles[pile_idx].pop()
        if piles[pile_idx] is not empty then
            heap.insert((piles[pile_idx].top(), pile_idx))
    
    return result
```

### 3.2 Step-by-Step Example

Sorting `[4, 3, 2, 5, 1]`:

```
Deal cards into piles:

Card 4: No piles → create pile 1
  Piles: [4]

Card 3: 3 ≤ 4 → add to pile 1
  Piles: [4,3]  (3 on top)

Card 2: 2 ≤ 3 → add to pile 1
  Piles: [4,3,2]  (2 on top)

Card 5: 5 > 2 → create pile 2
  Piles: [4,3,2], [5]

Card 1: 1 ≤ 2 → add to pile 1
  Piles: [4,3,2,1], [5]

Final piles (showing top to bottom):
  Pile 1: 1 → 2 → 3 → 4
  Pile 2: 5

Number of piles = 2 = LIS length (e.g., [4,5] or [3,5])

Merge phase:
  Extract 1 from pile 1
  Extract 2 from pile 1
  Extract 3 from pile 1
  Extract 4 from pile 1
  Extract 5 from pile 2

Result: [1, 2, 3, 4, 5]
```

## 4. Complexity Analysis

| Phase | Time | Space |
|-------|------|-------|
| Pile Building | O(n log n) | O(n) |
| Merging | O(n log p) | O(p) |
| **Total** | **O(n log n)** | **O(n)** |

Where p = number of piles ≤ n.

## 5. Implementation

```rust
use std::collections::BinaryHeap;
use std::cmp::Reverse;

pub fn patience_sort<T: Ord + Clone>(arr: &mut [T]) {
    if arr.len() <= 1 {
        return;
    }

    // Build piles
    let mut piles: Vec<Vec<T>> = Vec::new();
    
    for item in arr.iter() {
        // Binary search for pile
        let pile_idx = piles
            .iter()
            .position(|pile| pile.last().unwrap() >= item)
            .unwrap_or(piles.len());
        
        if pile_idx == piles.len() {
            piles.push(vec![item.clone()]);
        } else {
            piles[pile_idx].push(item.clone());
        }
    }

    // Merge piles using min-heap
    let mut heap: BinaryHeap<Reverse<(T, usize)>> = BinaryHeap::new();
    
    for (i, pile) in piles.iter_mut().enumerate() {
        if let Some(top) = pile.pop() {
            heap.push(Reverse((top, i)));
        }
    }

    let mut idx = 0;
    while let Some(Reverse((val, pile_idx))) = heap.pop() {
        arr[idx] = val;
        idx += 1;
        
        if let Some(next) = piles[pile_idx].pop() {
            heap.push(Reverse((next, pile_idx)));
        }
    }
}
```

### 5.1 LIS Length Computation

```rust
pub fn lis_length<T: Ord>(arr: &[T]) -> usize {
    let mut pile_tops: Vec<&T> = Vec::new();
    
    for item in arr {
        match pile_tops.binary_search(&item) {
            Ok(pos) | Err(pos) => {
                if pos == pile_tops.len() {
                    pile_tops.push(item);
                } else {
                    pile_tops[pos] = item;
                }
            }
        }
    }
    
    pile_tops.len()
}
```

## 6. Connection to LIS

### 6.1 Finding LIS

To reconstruct the actual LIS (not just length):
- Maintain back-pointers when dealing cards
- Trace back from last pile to first

### 6.2 LIS Variants

| Problem | Solution Using Patience |
|---------|------------------------|
| LIS Length | Count piles |
| LIS Elements | Back-pointers |
| Number of LIS | Count paths |
| LDS (Decreasing) | Reverse, then LIS |

## 7. Advantages and Limitations

### Advantages
| Advantage | Description |
|-----------|-------------|
| LIS computation | O(n log n) for LIS |
| Stable | Can preserve order |
| Intuitive | Easy to visualize |

### Limitations
| Limitation | Description |
|------------|-------------|
| O(n) space | Piles need memory |
| Complex merge | Requires heap |
| Not in-place | Unlike HeapSort |

## 8. Applications

1. **Longest Increasing Subsequence**: Primary application
2. **Sequence analysis**: Bioinformatics, text comparison
3. **Scheduling**: Task ordering problems
4. **Data compression**: Finding patterns

## 9. References

1. Mallows, C. L. (1963). "Problem 62-2, A Patience Sorting Problem".
2. Aldous, D.; Diaconis, P. (1999). "Longest Increasing Subsequences".
3. Fredman, M. L. (1975). "On Computing the Length of Longest Increasing Subsequences".

## 10. Source Code

**Implementation**: [src/sorting/patience_sort.rs](../../src/sorting/patience_sort.rs)
