# Saddleback Search

## 1. Overview

Saddleback Search (also known as Row-Column Search or Staircase Search) is an elegant algorithm for searching in a **row-wise and column-wise sorted 2D matrix**. It achieves $O(m + n)$ time complexity by exploiting the sorted structure, starting from a "saddle point" corner where one direction increases and another decreases.

The algorithm gets its name from the saddle-shaped surface formed when viewing the matrix values as heights—each corner forms a saddle point in terms of search direction options.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an $m \times n$ matrix $M$ where:
- Each row is sorted in ascending order: $M[i][j] \leq M[i][j+1]$
- Each column is sorted in ascending order: $M[i][j] \leq M[i+1][j]$

Find whether target value $x$ exists in $M$.

### 2.2 Matrix Properties

In a row-column sorted matrix:
- **Minimum element:** Top-left corner $M[0][0]$
- **Maximum element:** Bottom-right corner $M[m-1][n-1]$
- **Saddle points:** Top-right $M[0][n-1]$ and Bottom-left $M[m-1][0]$

At saddle points, we have a **decision point**:
- Top-right $M[0][n-1]$: Row's maximum, Column's minimum
- Bottom-left $M[m-1][0]$: Row's minimum, Column's maximum

### 2.3 Key Insight

From top-right corner $(0, n-1)$:
- If $x < M[0][n-1]$: $x$ cannot be in current column → move left
- If $x > M[0][n-1]$: $x$ cannot be in current row → move down
- If $x = M[0][n-1]$: Found!

Each comparison eliminates either a row or a column.

## 3. Algorithm Description

### 3.1 Intuition

Imagine standing at the top-right corner of a staircase:
- Looking left, values decrease
- Looking down, values increase

To find a target value:
- If target is smaller, step left (toward smaller values)
- If target is larger, step down (toward larger values)
- Each step eliminates an entire row or column

### 3.2 Pseudocode

```
SADDLEBACK-SEARCH(M, target):
    m ← number of rows
    n ← number of columns
    
    // Start from top-right corner
    row ← 0
    col ← n - 1
    
    while row < m AND col ≥ 0:
        if M[row][col] = target:
            return (row, col)
        else if M[row][col] > target:
            col ← col - 1    // Move left
        else:
            row ← row + 1    // Move down
    
    return NOT_FOUND
```

### 3.3 Step-by-Step Example

**Matrix:**
```
     col 0  1  2  3  4
row
 0   [  1, 4, 7, 11, 15 ]
 1   [  2, 5, 8, 12, 19 ]
 2   [  3, 6, 9, 16, 22 ]
 3   [ 10,13,14, 17, 24 ]
 4   [ 18,21,23, 26, 30 ]
```

**Target:** 14

**Search Path:**

| Step | Position | Value | Compare | Action |
|------|----------|-------|---------|--------|
| 1 | (0, 4) | 15 | 15 > 14 | Move left |
| 2 | (0, 3) | 11 | 11 < 14 | Move down |
| 3 | (1, 3) | 12 | 12 < 14 | Move down |
| 4 | (2, 3) | 16 | 16 > 14 | Move left |
| 5 | (2, 2) | 9 | 9 < 14 | Move down |
| 6 | (3, 2) | **14** | Found! | Return (3, 2) |

**Visual Path:**
```
[ 1, 4, 7, 11, 15←]  Start here
[ 2, 5, 8, 12, 19 ]
[ 3, 6, 9, 16, 22 ]
[10,13,14, 17, 24 ]  Found!
[18,21,23, 26, 30 ]
```

### 3.4 Why It Works

```
At position (row, col):
┌─────────────────────────────┐
│    All values ≤ M[row][col] │ ← Already eliminated (left)
│    in this region           │
├─────────────────────────────┤
│ (row, col) = current        │ ← Decision point
├─────────────────────────────┤
│    All values ≥ M[row][col] │ ← Already eliminated (above)
│    in this region           │
└─────────────────────────────┘
```

Each step either:
- Moves left: Eliminates current column (all below are larger)
- Moves down: Eliminates current row (all to left are smaller)

## 4. Complexity Analysis

### 4.1 Time Complexity

| Scenario | Operations | Complexity |
|----------|------------|------------|
| Best case | 1 comparison | O(1) |
| Worst case | m + n - 1 moves | O(m + n) |
| Average case | (m + n) / 2 | O(m + n) |

**Why O(m + n):**
- Start at (0, n-1)
- Maximum row increments: m - 1
- Maximum column decrements: n - 1
- Total moves ≤ m + n - 2

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Position variables | O(1) |
| **Total** | **O(1)** |

### 4.3 Comparison with Alternatives

| Approach | Time | Space | Notes |
|----------|------|-------|-------|
| Linear scan | O(mn) | O(1) | Ignores sorted property |
| Binary search each row | O(m log n) | O(1) | Doesn't use column order |
| Binary search each column | O(n log m) | O(1) | Doesn't use row order |
| **Saddleback** | **O(m + n)** | **O(1)** | **Optimal for this structure** |
| Divide & conquer | O(m + n) | O(log(mn)) | Stack space for recursion |

### 4.4 When Saddleback Wins

For $m = n = 1000$:

| Method | Operations |
|--------|------------|
| Linear | 1,000,000 |
| Binary per row | 10,000 (1000 × 10) |
| **Saddleback** | **1,998** (at most) |

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
/// Searches for `target` in a row-wise and column-wise sorted matrix.
/// Returns true if target exists, false otherwise.
/// Time Complexity: O(m + n) where m is rows, n is columns.
pub fn saddleback_search(matrix: &[Vec<i32>], target: i32) -> bool {
    if matrix.is_empty() || matrix[0].is_empty() {
        return false;
    }

    let (rows, cols) = (matrix.len(), matrix[0].len());
    let (mut row, mut col) = (0, cols - 1);

    while row < rows && col < cols {
        match matrix[row][col].cmp(&target) {
            std::cmp::Ordering::Equal => return true,
            std::cmp::Ordering::Greater => {
                if col == 0 {
                    break;
                }
                col -= 1;
            }
            std::cmp::Ordering::Less => {
                row += 1;
            }
        }
    }

    false
}
```

**Design Decisions:**

1. **Column Bounds Check:** `col < cols` instead of `col >= 0` since `col` is `usize`
2. **Underflow Prevention:** Check `col == 0` before decrementing
3. **Match Expression:** Clean pattern matching on `Ordering`
4. **Return Type:** Simple `bool` for existence check

### 5.2 Alternative: Return Position

```rust
pub fn saddleback_search_pos(
    matrix: &[Vec<i32>], 
    target: i32
) -> Option<(usize, usize)> {
    if matrix.is_empty() || matrix[0].is_empty() {
        return None;
    }

    let (rows, cols) = (matrix.len(), matrix[0].len());
    let (mut row, mut col) = (0, cols - 1);

    while row < rows {
        match matrix[row][col].cmp(&target) {
            Ordering::Equal => return Some((row, col)),
            Ordering::Greater => {
                if col == 0 { break; }
                col -= 1;
            }
            Ordering::Less => row += 1,
        }
    }

    None
}
```

### 5.3 Edge Cases

| Case | Input | Output | Notes |
|------|-------|--------|-------|
| Empty matrix | `[]` | `false` | Check bounds first |
| Empty rows | `[[]]` | `false` | Check column count |
| Single element (found) | `[[5]]`, target=5 | `true` | Immediate match |
| Single element (not found) | `[[5]]`, target=3 | `false` | Single comparison |
| Target < minimum | Any matrix, target < M[0][0] | `false` | Exit at col < 0 |
| Target > maximum | Any matrix, target > M[m-1][n-1] | `false` | Exit at row ≥ m |
| Multiple occurrences | Matrix with duplicates | First found | May not be consistent position |

### 5.4 Common Pitfalls

1. **Unsigned Integer Underflow:**
   ```rust
   // WRONG: col becomes MAX when decrementing from 0
   col -= 1;  // Panic or wrap-around!
   
   // CORRECT:
   if col == 0 { break; }
   col -= 1;
   ```

2. **Wrong Starting Corner:**
   ```rust
   // WRONG: Starting from top-left
   let (mut row, mut col) = (0, 0);  // Both directions increase!
   ```

3. **Assuming Square Matrix:**
   ```rust
   // WRONG: Using same bound for both
   let n = matrix.len();
   while row < n && col < n { ... }  // Fails for m ≠ n
   ```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Database Index Structures:**
   - 2D range queries on sorted dimensions
   - Time-series data indexed by (timestamp, sensor_id)

2. **Image Processing:**
   - Searching in integral images (summed area tables)
   - Finding features in sorted response matrices

3. **Geographic Information Systems:**
   - Location lookup in grid-based maps
   - Finding points in sorted spatial indices

4. **Spreadsheet Applications:**
   - Lookup in sorted pivot tables
   - Finding values in sorted cross-tabulations

### 6.2 Specific Applications

| Domain | Application |
|--------|-------------|
| Finance | Finding transactions by (date, amount) |
| E-commerce | Product search by (price, rating) |
| Gaming | Tile lookup in sorted game maps |
| Scheduling | Finding slots in sorted calendars |

## 7. Variants and Extensions

### 7.1 Bottom-Left Start

Equivalent algorithm starting from bottom-left:

```rust
pub fn saddleback_from_bottom_left(
    matrix: &[Vec<i32>], 
    target: i32
) -> bool {
    if matrix.is_empty() || matrix[0].is_empty() {
        return false;
    }

    let cols = matrix[0].len();
    let mut row = matrix.len() - 1;
    let mut col = 0;

    loop {
        match matrix[row][col].cmp(&target) {
            Ordering::Equal => return true,
            Ordering::Less => {
                col += 1;
                if col >= cols { break; }
            }
            Ordering::Greater => {
                if row == 0 { break; }
                row -= 1;
            }
        }
    }

    false
}
```

### 7.2 Find All Occurrences

For matrices with duplicates, find all positions:

```rust
pub fn find_all_positions(
    matrix: &[Vec<i32>],
    target: i32
) -> Vec<(usize, usize)> {
    let mut results = Vec::new();
    
    if matrix.is_empty() || matrix[0].is_empty() {
        return results;
    }

    let (rows, cols) = (matrix.len(), matrix[0].len());
    let (mut row, mut col) = (0, cols - 1);

    while row < rows && col < cols {
        match matrix[row][col].cmp(&target) {
            Ordering::Equal => {
                results.push((row, col));
                // Continue search in both directions
                row += 1;
                if col > 0 { col -= 1; }
            }
            Ordering::Greater => {
                if col == 0 { break; }
                col -= 1;
            }
            Ordering::Less => row += 1,
        }
    }

    results
}
```

### 7.3 Count Occurrences

Count elements less than or equal to target:

```rust
pub fn count_less_equal(matrix: &[Vec<i32>], target: i32) -> usize {
    if matrix.is_empty() { return 0; }
    
    let (rows, cols) = (matrix.len(), matrix[0].len());
    let mut count = 0;
    let mut col = cols - 1;

    for row in 0..rows {
        while col < cols && matrix[row][col] > target {
            if col == 0 { 
                return count + row * cols; // No elements ≤ target in remaining rows
            }
            col -= 1;
        }
        count += col + 1;  // All elements in row up to col are ≤ target
    }

    count
}
```

### 7.4 Generic Version

```rust
pub fn saddleback_search_generic<T: Ord>(
    matrix: &[Vec<T>],
    target: &T
) -> Option<(usize, usize)> {
    if matrix.is_empty() || matrix[0].is_empty() {
        return None;
    }

    let (rows, cols) = (matrix.len(), matrix[0].len());
    let (mut row, mut col) = (0, cols - 1);

    while row < rows {
        match matrix[row][col].cmp(target) {
            Ordering::Equal => return Some((row, col)),
            Ordering::Greater => {
                if col == 0 { return None; }
                col -= 1;
            }
            Ordering::Less => row += 1,
        }
    }

    None
}
```

## 8. Theoretical Analysis

### 8.1 Optimality

**Theorem:** Any comparison-based algorithm for searching in a row-column sorted matrix requires $\Omega(m + n)$ comparisons in the worst case.

**Proof Sketch:**
Consider the anti-diagonal elements. Without comparing, we cannot determine which side of the anti-diagonal the target lies. There are $m + n - 1$ anti-diagonal elements, requiring $\Omega(m + n)$ comparisons.

Saddleback search matches this lower bound: **optimal**.

### 8.2 Relationship to Binary Search

Saddleback search can be viewed as a 2D generalization of binary search:
- 1D binary search: Eliminates half the array each step
- 2D saddleback: Eliminates a row OR column each step

For square matrices ($n \times n$), both approach $O(n)$ convergence.

## 9. References

1. Saddleback, D. E. (1982). "Two-Dimensional Searching." *Information Processing Letters*.
2. Cormen, T. H., et al. (2009). "Introduction to Algorithms" (3rd ed.). MIT Press. Problem 6-1.
3. Bird, R. (2006). "Saddleback Search: A Lesson in How to Make Progress." *Mathematics of Program Construction*.

---

**Implementation:** [`src/searching/saddleback_search.rs`](../../src/searching/saddleback_search.rs)  
**See Also:** [Binary Search](binary_search.md), [Linear Search](linear_search.md)
