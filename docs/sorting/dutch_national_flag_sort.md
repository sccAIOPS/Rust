# Dutch National Flag Sort

## 1. Overview

Dutch National Flag Sort (also known as 3-way partitioning) is an algorithm for sorting an array containing only three distinct values (typically 0, 1, 2 representing colors of the Dutch flag: red, white, blue). It's a key component in 3-way QuickSort.

### Key Characteristics
- **Type**: Partition-based sort
- **In-place**: Yes
- **Stable**: No
- **Single pass**: O(n) time
- **Named after**: Edsger Dijkstra

## 2. Mathematical Foundation

### 2.1 Three-Way Partition

Given three categories (low, mid, high), partition array such that:
- All low values are at the beginning
- All mid values are in the middle  
- All high values are at the end

### 2.2 Invariants

Maintain three pointers (low, mid, high):
```
[0, low-1]      → all values < pivot (category 0)
[low, mid-1]    → all values = pivot (category 1)
[mid, high]     → unexamined
[high+1, n-1]   → all values > pivot (category 2)
```

## 3. Algorithm Description

### 3.1 Pseudocode

```
DUTCH_NATIONAL_FLAG(A)
    low ← 0
    mid ← 0
    high ← length(A) - 1
    
    while mid ≤ high do
        if A[mid] = 0 then
            swap(A[low], A[mid])
            low ← low + 1
            mid ← mid + 1
        else if A[mid] = 1 then
            mid ← mid + 1
        else  // A[mid] = 2
            swap(A[mid], A[high])
            high ← high - 1
```

### 3.2 Step-by-Step Example

Sorting `[2, 0, 1, 2, 1, 0]`:

```
Initial: [2, 0, 1, 2, 1, 0]
         low=0, mid=0, high=5

Step 1: A[mid]=2, swap with A[high], high--
        [0, 0, 1, 2, 1, 2]
         low=0, mid=0, high=4

Step 2: A[mid]=0, swap with A[low], low++, mid++
        [0, 0, 1, 2, 1, 2]
         low=1, mid=1, high=4

Step 3: A[mid]=0, swap with A[low], low++, mid++
        [0, 0, 1, 2, 1, 2]
         low=2, mid=2, high=4

Step 4: A[mid]=1, mid++
        [0, 0, 1, 2, 1, 2]
         low=2, mid=3, high=4

Step 5: A[mid]=2, swap with A[high], high--
        [0, 0, 1, 1, 2, 2]
         low=2, mid=3, high=3

Step 6: A[mid]=1, mid++
        [0, 0, 1, 1, 2, 2]
         low=2, mid=4, high=3

mid > high → DONE

Final: [0, 0, 1, 1, 2, 2]
```

## 4. Complexity Analysis

| Case | Complexity |
|------|------------|
| **Time** | O(n) - single pass |
| **Space** | O(1) |
| **Swaps** | At most n |

## 5. Implementation

```rust
pub fn dutch_national_flag_sort(arr: &mut [u8]) {
    if arr.len() <= 1 {
        return;
    }

    let mut low = 0;
    let mut mid = 0;
    let mut high = arr.len() - 1;

    while mid <= high {
        match arr[mid] {
            0 => {
                arr.swap(low, mid);
                low += 1;
                mid += 1;
            }
            1 => {
                mid += 1;
            }
            2 => {
                arr.swap(mid, high);
                if high == 0 {
                    break;
                }
                high -= 1;
            }
            _ => panic!("Invalid value"),
        }
    }
}
```

### 5.1 Generic Three-Way Partition

```rust
pub fn three_way_partition<T: Ord + Clone>(
    arr: &mut [T],
    pivot: T,
) -> (usize, usize) {
    let mut low = 0;
    let mut mid = 0;
    let mut high = arr.len().saturating_sub(1);

    while mid <= high {
        if arr[mid] < pivot {
            arr.swap(low, mid);
            low += 1;
            mid += 1;
        } else if arr[mid] > pivot {
            arr.swap(mid, high);
            if high == 0 {
                break;
            }
            high -= 1;
        } else {
            mid += 1;
        }
    }

    (low, mid) // Returns indices of equal partition
}
```

## 6. Applications

### 6.1 3-Way QuickSort

```rust
fn quicksort_3way<T: Ord + Clone>(arr: &mut [T]) {
    if arr.len() <= 1 {
        return;
    }
    
    let pivot = arr[arr.len() / 2].clone();
    let (lt, gt) = three_way_partition(arr, pivot);
    
    quicksort_3way(&mut arr[..lt]);
    quicksort_3way(&mut arr[gt..]);
}
```

### 6.2 Color Sorting Problems

- Sort array of 0s, 1s, 2s
- Segregate even and odd numbers
- Sort array with only two distinct elements

## 7. Comparison with Other Partitioning Schemes

| Scheme | Values | Passes | Use Case |
|--------|--------|--------|----------|
| Dutch Flag | 3 | 1 | Three distinct values |
| Lomuto | 2 | 1 | Standard QuickSort |
| Hoare | 2 | 1 | Efficient QuickSort |
| American Flag | k | 1 | k distinct values |

## 8. Advantages and Limitations

### Advantages
| Advantage | Description |
|-----------|-------------|
| Linear time | O(n) with single pass |
| In-place | O(1) extra space |
| Simple | Easy to implement |
| Optimal for 3 values | Best possible for this problem |

### Limitations
| Limitation | Description |
|------------|-------------|
| Only 3 values | Not general sorting |
| Not stable | Swaps break stability |
| Fixed categories | Need to know values in advance |

## 9. History

The problem was posed by Edsger Dijkstra as a programming exercise. The Dutch national flag has three horizontal stripes (red, white, blue), which inspired the name.

## 10. References

1. Dijkstra, E. W. (1976). *A Discipline of Programming*.
2. Bentley, J.; McIlroy, D. (1993). "Engineering a Sort Function".
3. Sedgewick, R. (1998). *Algorithms in C*.

## 11. Source Code

**Implementation**: [src/sorting/dutch_national_flag_sort.rs](../../src/sorting/dutch_national_flag_sort.rs)
