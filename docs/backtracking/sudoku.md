# Sudoku Solver

## 1. Overview

**Sudoku** (数独, "single number") is a logic-based combinatorial number-placement puzzle. The objective is to fill a 9×9 grid with digits so that each column, each row, and each of the nine 3×3 subgrids contain all digits from 1 to 9 exactly once.

This implementation uses a **backtracking algorithm** to solve Sudoku puzzles by systematically trying digits in empty cells and backtracking when conflicts arise.

### Historical Context
- **1979**: First modern Sudoku published by Howard Garns in Dell Magazines as "Number Place"
- **1984**: Introduced to Japan by Nikoli, named "Sudoku"
- **2004**: Global popularity explosion after Wayne Gould's promotion
- **Modern**: Standard benchmark for constraint satisfaction algorithms

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a partially filled 9×9 grid, complete it such that:

$$\forall i \in [1,9]: \text{row}_i \text{ contains } \{1,2,...,9\}$$
$$\forall j \in [1,9]: \text{col}_j \text{ contains } \{1,2,...,9\}$$
$$\forall k \in [1,9]: \text{box}_k \text{ contains } \{1,2,...,9\}$$

Where $\text{box}_k$ refers to the $k$-th 3×3 subgrid.

### 2.2 Mathematical Model

**Input**: 9×9 grid where 0 represents empty cells

**Output**: Completed 9×9 grid or `None` if unsolvable

**Constraints** (for each cell $(r, c)$ with value $v$):
1. **Row uniqueness**: $\forall j \neq c: \text{grid}[r][j] \neq v$
2. **Column uniqueness**: $\forall i \neq r: \text{grid}[i][c] \neq v$
3. **Box uniqueness**: Within the 3×3 box containing $(r, c)$, no other cell has value $v$

### 2.3 Constraint Satisfaction Perspective

Sudoku is an instance of a **Constraint Satisfaction Problem (CSP)**:
- **Variables**: 81 cells
- **Domains**: $\{1, 2, ..., 9\}$ for each cell
- **Constraints**: All-different constraints for rows, columns, and boxes

The total number of valid Sudoku grids is exactly **6,670,903,752,021,072,936,960** (discovered in 2005).

## 3. Algorithm Description

### 3.1 Intuition

The solver uses a simple yet effective strategy:
1. Find the first empty cell (scanning row by row)
2. Try digits 1-9 in that cell
3. For each valid digit (no immediate conflict), recursively solve the rest
4. If no digit works, backtrack and try a different digit in the previous cell
5. If all cells are filled, we have a solution

### 3.2 Pseudocode

```
function sudoku_solver(board):
    if solve(board):
        return board
    else:
        return None

function solve(board):
    cell = find_empty_cell(board)
    if cell is None:
        return true  // All cells filled - solution found
    
    (row, col) = cell
    for value in 1 to 9:
        if is_valid(board, row, col, value):
            board[row][col] = value
            if solve(board):
                return true
            board[row][col] = 0  // Backtrack
    
    return false  // No valid value found

function is_valid(board, row, col, value):
    // Check row
    for j in 0 to 8:
        if board[row][j] == value:
            return false
    
    // Check column
    for i in 0 to 8:
        if board[i][col] == value:
            return false
    
    // Check 3x3 box
    box_row = (row / 3) * 3
    box_col = (col / 3) * 3
    for i in box_row to box_row + 2:
        for j in box_col to box_col + 2:
            if board[i][j] == value:
                return false
    
    return true
```

### 3.3 Step-by-Step Example

```
Initial:                       After placing first values:
+-------+-------+-------+      +-------+-------+-------+
| 3 . 6 | 5 . 8 | 4 . . |      | 3 1 6 | 5 7 8 | 4 9 2 |
| 5 2 . | . . . | . . . |      | 5 2 9 | 1 3 4 | 7 6 8 |
| . 8 7 | . . . | . 3 1 |      | 4 8 7 | 6 2 9 | 5 3 1 |
+-------+-------+-------+      +-------+-------+-------+
| . . 3 | . 1 . | . 8 . |      | 2 6 3 | 4 1 5 | 9 8 7 |
| 9 . . | 8 6 3 | . . 5 |  →   | 9 7 4 | 8 6 3 | 1 2 5 |
| . 5 . | . 9 . | 6 . . |      | 8 5 1 | 7 9 2 | 6 4 3 |
+-------+-------+-------+      +-------+-------+-------+
| 1 3 . | . . . | 2 5 . |      | 1 3 8 | 9 4 7 | 2 5 6 |
| . . . | . . . | . 7 4 |      | 6 9 2 | 3 5 1 | 8 7 4 |
| . . 5 | 2 . 6 | 3 . . |      | 7 4 5 | 2 8 6 | 3 1 9 |
+-------+-------+-------+      +-------+-------+-------+
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Worst Case**: $O(9^{81}) = O(9^{81})$
  - In the absolute worst case, we try all 9 digits for all 81 cells
  - This is $9^{81} \approx 1.97 \times 10^{77}$ operations

- **Practical Case**: Much better due to constraints
  - A well-posed Sudoku has a unique solution
  - Constraint checking prunes most invalid branches early
  - Typical puzzles solve in milliseconds

- **Average Case**: Hard to characterize analytically
  - Depends on the number of given clues
  - More clues = faster solving (fewer empty cells)

### 4.2 Space Complexity

- **Board Storage**: $O(81) = O(1)$ (fixed size)
- **Recursion Stack**: $O(81) = O(1)$ (at most 81 recursive calls)
- **Total**: $O(1)$ - constant space (the board size is fixed)

### 4.3 Complexity with Optimizations

| Technique | Time Improvement | Space Trade-off |
|-----------|------------------|-----------------|
| Basic backtracking | Baseline | O(1) |
| Constraint propagation | ~10-100x | O(81×9) for candidates |
| MRV heuristic | ~10x | O(81) for counts |
| Dancing Links (DLX) | ~1000x | O(constraints) |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
// Fixed-size arrays for efficiency
board: [[u8; 9]; 9]

// Using Option for the return type
pub fn sudoku_solver(board: &[[u8; 9]; 9]) -> Option<[[u8; 9]; 9]>
```

**Key Patterns**:
- **Fixed-size arrays**: `[[u8; 9]; 9]` instead of `Vec<Vec<u8>>` for cache efficiency
- **Copy semantics**: The board is copied on input for immutable solving
- **Option return**: Elegant handling of unsolvable puzzles

### 5.2 Edge Cases

| Case | Description | Expected Output |
|------|-------------|-----------------|
| Valid puzzle | Standard Sudoku with unique solution | `Some(solution)` |
| Invalid puzzle | Conflicts in initial state | `None` |
| Empty puzzle | All zeros | First valid solution (non-unique) |
| Already solved | Complete valid grid | `Some(same grid)` |

### 5.3 Potential Optimizations

1. **Most Constrained Variable (MCV/MRV)**: Choose the cell with fewest legal values
2. **Constraint Propagation**: When placing a digit, eliminate it from peers
3. **Naked/Hidden Singles**: Identify cells where only one value is possible
4. **Bit manipulation**: Use bitsets for tracking available digits

```rust
// Example: Bitset for available digits
let mut available: u16 = 0b1111111110;  // bits 1-9 set
// Clear bit when digit used
available &= !(1 << digit);
// Check if digit available
let is_available = (available >> digit) & 1 == 1;
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Constraint Satisfaction Solvers**: Template for general CSP problems
2. **SAT Solver Testing**: Sudoku can be encoded as SAT problems
3. **Parallel Algorithm Research**: Sudoku parallelization strategies
4. **Puzzle Generation**: Creating puzzles with unique solutions
5. **Educational Tools**: Teaching backtracking and constraint propagation

### 6.2 Related Algorithms

| Algorithm | Relationship |
|-----------|-------------|
| N-Queens | Same backtracking pattern, different constraints |
| Graph Coloring | Sudoku is 9-coloring of a specific graph |
| SAT Solving | Sudoku can be reduced to SAT |
| CSP Solvers | Sudoku is a canonical CSP example |

## 7. Implementation Analysis

### 7.1 Current Implementation Review

```rust
fn is_value_valid(&self, coordinates: (usize, usize), value: u8) -> bool {
    let (row, column) = coordinates;
    
    // Row check - O(9)
    for current_column in 0..9 {
        if self.board[row][current_column] == value {
            return false;
        }
    }
    
    // Column check - O(9)
    for current_row in 0..9 {
        if self.board[current_row][column] == value {
            return false;
        }
    }
    
    // Box check - O(9)
    let start_row = row / 3 * 3;
    let start_column = column / 3 * 3;
    for current_row in start_row..start_row + 3 {
        for current_column in start_column..start_column + 3 {
            if self.board[current_row][current_column] == value {
                return false;
            }
        }
    }
    true
}
```

**Analysis**:
- ✅ Correctly validates all three constraints
- ✅ Uses integer division for box calculation (`row / 3 * 3`)
- ⚠️ Checks 27 cells total; could use bitsets for O(1) checking
- ⚠️ `find_empty_cell` is O(81) per call; could cache next empty

### 7.2 Complexity Verification

| Operation | Claimed | Verified |
|-----------|---------|----------|
| `is_value_valid` | O(27) = O(1) | ✅ Fixed iterations |
| `find_empty_cell` | O(81) = O(1) | ✅ Fixed iterations |
| `solve` | O(9^81) worst | ✅ 9 choices, 81 cells max |

## 8. Testing

### 8.1 Test Cases Covered

```rust
test_sudoku_correct: /* Valid puzzle with solution */
test_sudoku_incorrect: /* Invalid puzzle returning None */
```

### 8.2 Recommended Additional Tests

```rust
#[test]
fn test_empty_puzzle() {
    let empty = [[0u8; 9]; 9];
    let result = sudoku_solver(&empty);
    assert!(result.is_some());
    // Verify solution validity
}

#[test]
fn test_already_solved() {
    let solved = /* complete valid grid */;
    assert_eq!(sudoku_solver(&solved), Some(solved));
}

#[test]
fn test_multiple_solutions_returns_one() {
    let ambiguous = /* puzzle with multiple solutions */;
    let result = sudoku_solver(&ambiguous);
    assert!(result.is_some());
    assert!(is_valid_sudoku(&result.unwrap()));
}
```

## 9. Advanced Topics

### 9.1 Sudoku as Graph Coloring

Sudoku can be modeled as a graph coloring problem:
- **Vertices**: 81 cells
- **Edges**: Connect cells that share row, column, or box
- **Colors**: Digits 1-9

This graph is called the **Sudoku graph** and has 810 edges.

### 9.2 Minimum Clue Sudoku

- The minimum number of clues for a unique solution is **17**
- Proven in 2012 by exhaustive computer search
- No 16-clue Sudoku has a unique solution

### 9.3 NP-Completeness

Generalized n²×n² Sudoku is NP-complete (Yato & Seta, 2003).

## 10. References

1. Yato, T., & Seta, T. (2003). "Complexity and Completeness of Finding Another Solution and Its Application to Puzzles". *IEICE Transactions*.
2. McGuire, G., et al. (2012). "There is no 16-Clue Sudoku: Solving the Sudoku Minimum Number of Clues Problem". arXiv:1201.0749.
3. Simonis, H. (2005). "Sudoku as a Constraint Problem". *CP Workshop on Modeling and Reformulating Constraint Satisfaction Problems*.
4. [GeeksforGeeks: Sudoku Backtracking](https://www.geeksforgeeks.org/sudoku-backtracking-7/)
