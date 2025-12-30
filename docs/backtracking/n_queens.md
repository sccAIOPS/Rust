# N-Queens Problem

## 1. Overview

The **N-Queens problem** is a classic combinatorial puzzle that asks: *How can N chess queens be placed on an N×N chessboard so that no two queens threaten each other?*

This means no two queens can share the same row, column, or diagonal. The problem was first posed in 1848 by chess composer Max Bezzel and has since become a benchmark problem for constraint satisfaction and backtracking algorithms.

### Historical Context
- **1848**: First proposed by Max Bezzel for 8 queens
- **1850**: First solutions published by Franz Nauck
- **1874**: Complete enumeration of 8-queens solutions by Gunther and Glaisher
- **Modern**: Used as a standard benchmark for CSP solvers

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an $N \times N$ chessboard, find all arrangements of $N$ queens such that:

$$\forall i, j \in [1, N], i \neq j: \text{queen}_i \text{ does not attack queen}_j$$

Two queens attack each other if they are on the same:
- **Row**: $\text{row}_i = \text{row}_j$
- **Column**: $\text{col}_i = \text{col}_j$
- **Diagonal**: $|\text{row}_i - \text{row}_j| = |\text{col}_i - \text{col}_j|$

### 2.2 Mathematical Model

**Input**: Integer $N \geq 0$

**Output**: All valid placements $(c_1, c_2, ..., c_N)$ where $c_i$ is the column position of the queen in row $i$

**Constraints**:
1. $c_i \in [0, N-1]$ for all $i$
2. $c_i \neq c_j$ for all $i \neq j$ (no two queens in same column)
3. $|i - j| \neq |c_i - c_j|$ for all $i \neq j$ (no two queens on same diagonal)

### 2.3 Solution Count

The number of solutions for small $N$:

| N | Solutions | Distinct (up to symmetry) |
|---|-----------|--------------------------|
| 1 | 1 | 1 |
| 2 | 0 | 0 |
| 3 | 0 | 0 |
| 4 | 2 | 1 |
| 5 | 10 | 2 |
| 6 | 4 | 1 |
| 7 | 40 | 6 |
| 8 | 92 | 12 |

No closed-form formula exists for the number of solutions.

## 3. Algorithm Description

### 3.1 Intuition

The backtracking approach places queens row by row:
1. Start with an empty board
2. For each row, try placing a queen in each column
3. If placement is safe (no conflicts), proceed to the next row
4. If all columns fail, backtrack to the previous row and try the next column
5. When all N queens are placed successfully, record the solution

### 3.2 Pseudocode

```
function n_queens_solver(n):
    solutions = []
    board = empty n×n grid
    solve(board, 0, n, solutions)
    return solutions

function solve(board, row, n, solutions):
    if row == n:
        add board configuration to solutions
        return
    
    for col in 0 to n-1:
        if is_safe(board, row, col):
            place_queen(board, row, col)
            solve(board, row + 1, n, solutions)
            remove_queen(board, row, col)  // backtrack

function is_safe(board, row, col):
    // Check column above current row
    for i in 0 to row-1:
        if board[i][col] == 'Q':
            return false
    
    // Check upper-left diagonal
    for i, j = row-1, col-1 while i >= 0 and j >= 0:
        if board[i][j] == 'Q':
            return false
        i--, j--
    
    // Check upper-right diagonal
    for i, j = row-1, col+1 while i >= 0 and j < n:
        if board[i][j] == 'Q':
            return false
        i--, j++
    
    return true
```

### 3.3 Step-by-Step Example (N=4)

```
Step 1: Place Q at (0,0)        Step 2: Row 1, try col 0,1 - fail
+---+---+---+---+              Place Q at (1,2)
| Q |   |   |   |              +---+---+---+---+
+---+---+---+---+              | Q |   |   |   |
|   |   |   |   |              +---+---+---+---+
+---+---+---+---+              |   |   | Q |   |
|   |   |   |   |              +---+---+---+---+
+---+---+---+---+              |   |   |   |   |
|   |   |   |   |              +---+---+---+---+
+---+---+---+---+              |   |   |   |   |
                               +---+---+---+---+

Step 3: Row 2 - no valid       Step 4: Backtrack, try (0,1)
position, backtrack            +---+---+---+---+
                               |   | Q |   |   |
                               +---+---+---+---+
                               |   |   |   | Q |
                               +---+---+---+---+
                               | Q |   |   |   |
                               +---+---+---+---+
                               |   |   | Q |   |  ← Solution!
                               +---+---+---+---+
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Worst Case**: $O(N!)$
  - At row 0: N choices
  - At row 1: at most N-1 choices (column constraint)
  - At row k: at most N-k choices
  - Total: $N \times (N-1) \times (N-2) \times ... \times 1 = N!$

- **Average Case**: Much better due to diagonal pruning
  - Empirically closer to $O(N!)$ but with small constant

- **Best Case**: $O(N^2)$ when finding first solution quickly

**Derivation**: Each node at depth $d$ has at most $N-d$ children. The branching factor decreases due to column exclusion. Diagonal constraints further prune the tree.

### 4.2 Space Complexity

- **Board Storage**: $O(N^2)$ for the 2D board
- **Recursion Stack**: $O(N)$ depth
- **Solutions Storage**: $O(S \times N^2)$ where $S$ is the number of solutions

**Total**: $O(N^2 + S \times N^2)$ or $O(N^2)$ if only counting solutions

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
// Uses Vec<Vec<char>> for the board representation
board: Vec<Vec<char>>

// Efficient memory management with std::mem::take
fn solve(&mut self) -> Vec<Vec<String>> {
    self.solve_helper(0);
    std::mem::take(&mut self.solutions)  // Move without clone
}
```

**Key Patterns**:
- **Encapsulation**: `NQueensSolver` struct contains all state
- **Builder pattern**: `new()` → `solve()` workflow
- **Zero-copy return**: Uses `std::mem::take` instead of cloning

### 5.2 Edge Cases

| Case | N | Expected Output |
|------|---|-----------------|
| Empty board | 0 | One empty solution `[[]]` |
| Single queen | 1 | `[["Q"]]` |
| Impossible | 2, 3 | Empty vector `[]` |
| Standard | 4+ | Multiple solutions |

### 5.3 Potential Optimizations

1. **Bit manipulation**: Use bitmasks for column/diagonal tracking
2. **Symmetry reduction**: Only search half, mirror for remaining
3. **Constraint propagation**: Track available positions more efficiently

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **VLSI Circuit Design**: Placing components without interference
2. **Parallel Memory Access**: Non-conflicting memory bank assignments
3. **Task Scheduling**: Assigning tasks to time slots without conflicts
4. **Network Traffic Management**: Non-interfering packet routing

### 6.2 Related Algorithms

| Algorithm | When to Use |
|-----------|------------|
| N-Queens | Constraint satisfaction with spatial constraints |
| Sudoku | Grid-based constraint satisfaction |
| Graph Coloring | General vertex constraint satisfaction |
| SAT Solver | Boolean constraint satisfaction |

## 7. Implementation Analysis

### 7.1 Current Implementation Review

```rust
fn is_safe(&self, row: usize, col: usize) -> bool {
    for i in 0..row {
        if self.board[i][col] == 'Q'
            || (col >= row - i && self.board[i][col - (row - i)] == 'Q')
            || (col + row - i < self.size && self.board[i][col + (row - i)] == 'Q')
        {
            return false;
        }
    }
    true
}
```

**Analysis**:
- ✅ Correctly checks column and both diagonals
- ✅ Only checks rows above current (since we place row by row)
- ✅ Efficient single-loop check combining all three conditions
- ⚠️ Could use bit manipulation for O(1) conflict checking

### 7.2 Complexity Verification

| Operation | Claimed | Verified |
|-----------|---------|----------|
| `is_safe` | O(N) | ✅ Single loop to row |
| `solve_helper` | O(N!) | ✅ N choices at each of N levels |
| `solve` | O(N!) | ✅ Calls solve_helper once |

## 8. Testing

### 8.1 Test Cases Covered

```rust
test_0_queens: (0, vec![Vec::<String>::new()]),  // Edge case
test_1_queen: (1, vec![vec!["Q"]]),              // Trivial
test_2_queens: (2, Vec::new()),                  // No solution
test_3_queens: (3, Vec::new()),                  // No solution
test_4_queens: (4, /* 2 solutions */),           // First non-trivial
test_5_queens: (5, /* 10 solutions */),          // Larger
test_6_queens: (6, /* 4 solutions */),           // Larger still
```

### 8.2 Recommended Additional Tests

```rust
#[test]
fn test_solution_validity() {
    // Verify no two queens attack each other in any solution
    for solution in n_queens_solver(8) {
        assert!(is_valid_solution(&solution));
    }
}

#[test]
fn test_solution_count() {
    assert_eq!(n_queens_solver(8).len(), 92);
}
```

## 9. References

1. Dijkstra, E. W. (1972). "The Eight Queens Problem". EWD316.
2. Knuth, D. E. (2000). "Dancing Links". arXiv:cs/0011047.
3. Bell, J., & Stevens, B. (2009). "A survey of known results and research areas for n-queens". *Discrete Mathematics*, 309(1), 1-31.
4. [OEIS A000170](https://oeis.org/A000170) - Number of solutions to N-queens problem
