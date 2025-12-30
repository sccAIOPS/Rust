# Knight's Tour

## 1. Overview

The **Knight's Tour** is a classic chess puzzle where a knight must visit every square on a chessboard exactly once. The challenge is to find a sequence of moves that accomplishes this without revisiting any square.

This problem has fascinated mathematicians for centuries and serves as an excellent example of backtracking and heuristic search algorithms.

### Historical Context
- **9th century**: First documented in Arabic literature
- **1759**: Euler's famous paper on knight's tours
- **1823**: Warnsdorff published his rule for efficient solving
- **Modern**: Applications in circuit design and mathematical research

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an $M \times N$ chessboard, find a sequence of knight moves $(x_1, y_1), (x_2, y_2), ..., (x_{MN}, y_{MN})$ such that:

1. Each square $(x_i, y_i)$ is visited exactly once
2. Each move follows knight movement rules (L-shape)
3. All $M \times N$ squares are covered

### 2.2 Mathematical Model

**Input**: 
- Board dimensions: `size_x × size_y`
- Starting position: `(start_x, start_y)`

**Output**: Tour matrix where each cell contains the move number (1 to $M \times N$)

**Knight Moves**: From position $(x, y)$, a knight can move to:
$$\{(x±1, y±2), (x±2, y±1)\}$$

This gives 8 possible moves (fewer at edges).

### 2.3 Existence of Solutions

| Board Size | Open Tour? | Closed Tour? |
|------------|-----------|--------------|
| 5×5 | Yes (from some squares) | No |
| 6×6 | Yes | Yes |
| 7×7 | Yes | No |
| 8×8 | Yes | Yes |
| n×n (n≥5) | Yes | Yes (if n even) |

For rectangular boards: An open tour exists for all $m \times n$ boards where $\min(m,n) \geq 5$.

## 3. Algorithm Description

### 3.1 Intuition

The backtracking approach systematically explores knight moves:
1. Mark the starting square with move number 1
2. From current position, try all 8 possible knight moves
3. For each valid move (on board and unvisited), recurse
4. If all squares are visited, return success
5. If stuck, backtrack by unmarking the current square

### 3.2 Pseudocode

```
function find_knight_tour(size_x, size_y, start_x, start_y):
    board = new Board(size_x, size_y)
    board[start_x][start_y] = 1  // First move
    
    if solve_tour(board, start_x, start_y, 2):
        return board
    else:
        return None

function solve_tour(board, x, y, move_count):
    if move_count > size_x * size_y:
        return true  // All squares visited
    
    for each (dx, dy) in KNIGHT_MOVES:
        next_x = x + dx
        next_y = y + dy
        
        if is_safe(board, next_x, next_y):
            board[next_x][next_y] = move_count
            if solve_tour(board, next_x, next_y, move_count + 1):
                return true
            board[next_x][next_y] = 0  // Backtrack
    
    return false

KNIGHT_MOVES = [(2,1), (1,2), (-1,2), (-2,1), (-2,-1), (-1,-2), (1,-2), (2,-1)]
```

### 3.3 Step-by-Step Example

For a 5×5 board starting at (0,0):

```
Initial:              After several moves:      Solution:
+--+--+--+--+--+     +--+--+--+--+--+         +--+--+--+--+--+
| 1|  |  |  |  |     | 1| 6|15|10|21|         | 1| 6|15|10|21|
+--+--+--+--+--+     +--+--+--+--+--+         +--+--+--+--+--+
|  |  |  |  |  |     |14| 9|20| 5|16|         |14| 9|20| 5|16|
+--+--+--+--+--+     +--+--+--+--+--+         +--+--+--+--+--+
|  |  |  |  |  |     |19| 2| 7|22|11|         |19| 2| 7|22|11|
+--+--+--+--+--+     +--+--+--+--+--+         +--+--+--+--+--+
|  |  |  |  |  |     | 8|13|24|17| 4|         | 8|13|24|17| 4|
+--+--+--+--+--+     +--+--+--+--+--+         +--+--+--+--+--+
|  |  |  |  |  |     |25|18| 3|12|23|         |25|18| 3|12|23|
+--+--+--+--+--+     +--+--+--+--+--+         +--+--+--+--+--+
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Worst Case**: $O(8^{N^2})$ where $N^2$ is the number of squares
  - At each of $N^2$ squares, up to 8 moves possible
  - Forms a search tree of depth $N^2$

- **With Warnsdorff's Heuristic**: Nearly $O(N^2)$ in practice
  - The heuristic dramatically reduces backtracking
  - Almost always finds a solution without backtracking

- **Average Case**: Highly dependent on board size and starting position

### 4.2 Space Complexity

- **Board Storage**: $O(N^2)$ for the tour matrix
- **Recursion Stack**: $O(N^2)$ depth

**Total**: $O(N^2)$

### 4.3 Impact of Heuristics

| Approach | Time (8×8 board) | Backtracking |
|----------|------------------|--------------|
| Naive backtracking | Hours to days | Extensive |
| Warnsdorff's rule | Milliseconds | Minimal |
| Neural network | Milliseconds | None (deterministic) |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
struct KnightTour {
    board: Vec<Vec<usize>>,
}

impl KnightTour {
    const MOVES: [(isize, isize); 8] = [
        (2, 1), (1, 2), (-1, 2), (-2, 1),
        (-2, -1), (-1, -2), (1, -2), (2, -1),
    ];
    
    fn is_safe(&self, x: isize, y: isize) -> bool {
        x >= 0 && y >= 0 
            && x < self.size_x() as isize 
            && y < self.size_y() as isize
            && self.board[x as usize][y as usize] == 0
    }
}
```

**Key Patterns**:
- **Constant moves array**: Static definition of knight moves
- **`isize` for coordinates**: Allows negative intermediate values during bounds checking
- **2D `Vec` for board**: Flexible board sizes
- **Zero as unvisited**: Simple sentinel value

### 5.2 Edge Cases

| Case | Parameters | Result |
|------|------------|--------|
| Standard 8×8 | (8, 8, 0, 0) | Some (tour matrix) |
| Minimum solvable | (5, 5, 0, 0) | Some |
| Some positions unsolvable | (5, 5, 2, 1) | None |
| Invalid start | (8, 8, 10, 10) | None |
| Tiny board | (3, 3, 0, 0) | None (no tour exists) |

### 5.3 Warnsdorff's Heuristic

**Rule**: Always move to the square with the fewest onward moves.

```rust
fn warnsdorff_move(&self, x: isize, y: isize) -> Option<(isize, isize)> {
    Self::MOVES.iter()
        .map(|&(dx, dy)| (x + dx, y + dy))
        .filter(|&(nx, ny)| self.is_safe(nx, ny))
        .min_by_key(|&(nx, ny)| self.count_onward_moves(nx, ny))
}

fn count_onward_moves(&self, x: isize, y: isize) -> usize {
    Self::MOVES.iter()
        .filter(|&&(dx, dy)| self.is_safe(x + dx, y + dy))
        .count()
}
```

**Why it works**: By choosing squares with fewer exits, we avoid "painting ourselves into a corner."

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **VLSI Design**: Routing connections without crossings
2. **Image Processing**: Pixel traversal patterns
3. **Puzzle Games**: Game AI for chess-based puzzles
4. **Memory Access**: Cache-friendly matrix traversal
5. **Cryptography**: Permutation generation

### 6.2 Related Algorithms

| Algorithm | Relationship |
|-----------|-------------|
| Hamiltonian Path | Knight's tour is Hamiltonian path on knight's graph |
| Depth-First Search | Foundation of backtracking approach |
| A* Search | Alternative for finding tours |
| Divide and Conquer | Construct tours by combining sub-tours |

## 7. Implementation Analysis

### 7.1 Current Implementation Review

```rust
fn solve_tour(&mut self, x: isize, y: isize, move_count: usize) -> bool {
    if move_count == self.size_x() * self.size_y() {
        return true;
    }
    
    for &(dx, dy) in &Self::MOVES {
        let next_x = x + dx;
        let next_y = y + dy;

        if self.is_safe(next_x, next_y) {
            self.board[next_x as usize][next_y as usize] = move_count + 1;
            
            if self.solve_tour(next_x, next_y, move_count + 1) {
                return true;
            }
            // Backtrack
            self.board[next_x as usize][next_y as usize] = 0;
        }
    }
    false
}
```

**Analysis**:
- ✅ Clean implementation of basic backtracking
- ✅ Efficient bounds checking with `isize`
- ✅ Proper backtracking (reset to 0)
- ⚠️ No heuristic ordering (Warnsdorff's rule would help)
- ⚠️ Returns first solution found, not all solutions

### 7.2 Move Order Impact

The order of moves in `MOVES` affects performance:
```rust
const MOVES: [(isize, isize); 8] = [
    (2, 1), (1, 2), (-1, 2), (-2, 1),
    (-2, -1), (-1, -2), (1, -2), (2, -1),
];
```

Different orderings can lead to finding solutions faster or slower depending on the starting position.

### 7.3 Complexity Verification

| Operation | Claimed | Verified |
|-----------|---------|----------|
| `is_safe` | O(1) | ✅ Array lookup |
| `solve_tour` | O(8^(N²)) | ✅ 8 choices at N² levels |
| Space | O(N²) | ✅ Board + recursion stack |

## 8. Testing

### 8.1 Test Cases Covered

```rust
test_knight_tour_5x5: (5, 5, 0, 0) → Some (valid tour)
test_knight_tour_6x6: (6, 6, 0, 0) → Some (valid tour)
test_knight_tour_8x8: (8, 8, 0, 0) → Some (valid tour)
test_no_solution: (5, 5, 2, 1) → None
test_invalid_start_position: (8, 8, 10, 10) → None
```

### 8.2 Test Validation

Each test verifies:
1. Starting position is marked with 1
2. Solution is complete (all squares visited)
3. Each move follows knight rules

```rust
if expected.is_some() {
    assert_eq!(expected.clone().unwrap()[start_x][start_y], 1)
}
```

## 9. Open vs Closed Tours

### 9.1 Open Tour
Visits all squares exactly once (Hamiltonian path).

### 9.2 Closed Tour
An open tour where the ending square is a knight's move from the starting square (Hamiltonian cycle).

```rust
fn is_closed_tour(&self, tour: &Vec<Vec<usize>>) -> bool {
    let (start_x, start_y) = self.find_position(tour, 1);
    let (end_x, end_y) = self.find_position(tour, self.size_x() * self.size_y());
    
    Self::MOVES.iter().any(|&(dx, dy)| {
        start_x as isize + dx == end_x as isize 
        && start_y as isize + dy == end_y as isize
    })
}
```

### 9.3 Existence by Board Size

| Size | Open Tours | Closed Tours |
|------|-----------|--------------|
| 5×5 | Yes (some starts) | No |
| 6×6 | Yes | Yes |
| 7×7 | Yes | No |
| 8×8 | Yes | Yes |

For $n \times n$ boards: Closed tours exist for even $n \geq 6$.

## 10. References

1. Euler, L. (1759). "Solution d'une question curieuse que ne paroit soumise à aucune analyse". *Mémoires de l'académie des sciences de Berlin*.
2. Warnsdorff, H. C. (1823). *Des Rösselsprunges einfachste und allgemeinste Lösung*. Schmalkalden.
3. Schwenk, A. J. (1991). "Which Rectangular Chessboards Have a Knight's Tour?". *Mathematics Magazine*.
4. Parberry, I. (1997). "An Efficient Algorithm for the Knight's Tour Problem". *Discrete Applied Mathematics*.
5. [Wikipedia: Knight's Tour](https://en.wikipedia.org/wiki/Knight%27s_tour)
