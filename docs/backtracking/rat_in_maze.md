# Rat in Maze

## 1. Overview

The **Rat in Maze** problem is a classic backtracking puzzle where we need to find a path for a rat to travel from a starting position to an exit in a maze. The rat can only move through open cells (no walls) and typically can move in four directions: up, down, left, right.

This problem is fundamental in pathfinding algorithms and serves as an introduction to maze-solving techniques.

### Historical Context
- **Ancient**: Maze solving appears in Greek mythology (Theseus and the Minotaur)
- **1800s**: Formal study of maze-solving algorithms began
- **1959**: Claude Shannon built a maze-solving mouse "Theseus"
- **Modern**: Applications in robotics, game AI, and GPS navigation

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an $M \times N$ maze represented as a grid of booleans, find a path from position $(s_x, s_y)$ to the exit at $(M-1, N-1)$ moving only through cells marked as `true` (open).

### 2.2 Mathematical Model

**Input**: 
- Maze: 2D boolean matrix where `true` = open, `false` = wall
- Start position: $(s_x, s_y)$

**Output**: Solution matrix where `true` marks the path, or `None` if no path exists

**Movement Rules**:
- From $(x, y)$, can move to: $(x, y+1)$, $(x+1, y)$, $(x, y-1)$, $(x-1, y)$
- Cannot move to walls or outside boundaries
- Cannot revisit cells in the current path

### 2.3 Path Properties

| Property | Description |
|----------|-------------|
| Simple path | No cell visited twice |
| Shortest path | Minimum number of moves (requires BFS) |
| Any path | First valid path found (this implementation) |

## 3. Algorithm Description

### 3.1 Intuition

The backtracking approach explores the maze systematically:
1. Start at the initial position, mark it as part of the path
2. Try each of the four directions in order
3. For each valid move (open and unvisited), recurse
4. If we reach the exit, return success
5. If all directions fail, unmark current cell and backtrack

### 3.2 Pseudocode

```
function find_path_in_maze(maze, start_x, start_y):
    validate inputs
    solution = new Matrix(same size as maze, all false)
    
    if solve(maze, start_x, start_y, solution):
        return solution
    else:
        return None

function solve(maze, x, y, solution):
    // Base case: reached exit
    if x == height-1 and y == width-1:
        solution[x][y] = true
        return true
    
    if is_valid(maze, x, y, solution):
        solution[x][y] = true  // Mark as part of path
        
        // Try all four directions
        for each (dx, dy) in [(0,1), (1,0), (0,-1), (-1,0)]:
            if solve(maze, x + dx, y + dy, solution):
                return true
        
        // Backtrack
        solution[x][y] = false
    
    return false

function is_valid(maze, x, y, solution):
    return x >= 0 and y >= 0 
        and x < height and y < width
        and maze[x][y] == true      // Open cell
        and solution[x][y] == false  // Not yet in path
```

### 3.3 Step-by-Step Example

For maze (T=open, F=wall):
```
Start (0,0)      After exploring:       Solution:
+---+---+---+    +---+---+---+          +---+---+---+
| T | F | T |    | ✓ | F | T |          | T | F | F |
+---+---+---+    +---+---+---+          +---+---+---+
| T | T | F | →  | ✓ | ✓ | F |    →    | T | T | F |
+---+---+---+    +---+---+---+          +---+---+---+
| F | T | T |    | F | ✓ | ✓ |          | F | T | T |
+---+---+---+    +---+---+---+          +---+---+---+
                 Exit (2,2)
```

Path: (0,0) → (1,0) → (1,1) → (2,1) → (2,2)

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Worst Case**: $O(4^{M \times N})$
  - At each cell, up to 4 directions
  - Maximum $M \times N$ cells in path
  - Upper bound: $4^{M \times N}$

- **Practical Case**: Much better due to:
  - Walls blocking many paths
  - Solution marking prevents revisits
  - Early termination when exit found

- **Dense Maze**: Closer to $O(M \times N)$ with pruning

### 4.2 Space Complexity

- **Solution Matrix**: $O(M \times N)$
- **Recursion Stack**: $O(M \times N)$ depth

**Total**: $O(M \times N)$

### 4.3 Comparison with Other Approaches

| Algorithm | Time | Space | Finds Shortest? |
|-----------|------|-------|-----------------|
| Backtracking (this) | $O(4^{MN})$ worst | $O(MN)$ | No |
| BFS | $O(MN)$ | $O(MN)$ | Yes |
| DFS (iterative) | $O(MN)$ | $O(MN)$ | No |
| A* | $O(MN \log MN)$ | $O(MN)$ | Yes (with heuristic) |

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
#[derive(Debug, PartialEq, Eq)]
pub enum MazeError {
    EmptyMaze,
    OutOfBoundPos,
    ImproperMazeRepr,
}

struct Maze {
    maze: Vec<Vec<bool>>,
}

impl Maze {
    const MOVES: [(isize, isize); 4] = [(0, 1), (1, 0), (0, -1), (-1, 0)];
}
```

**Key Patterns**:
- **Custom error enum**: Comprehensive error handling
- **Boolean matrix**: Simple wall/open representation
- **Constant moves array**: Direction vectors
- **`isize` arithmetic**: Safe bounds checking with negative values

### 5.2 Edge Cases

| Case | Maze | Start | Result |
|------|------|-------|--------|
| Empty maze | `[]` | (0,0) | `Err(EmptyMaze)` |
| Start out of bounds | valid | (10,10) | `Err(OutOfBoundPos)` |
| Non-rectangular | irregular | (0,0) | `Err(ImproperMazeRepr)` |
| Single cell (exit) | `[[true]]` | (0,0) | `Ok(Some([[true]]))` |
| No path exists | blocked | (0,0) | `Ok(None)` |
| Start on wall | wall at (0,0) | (0,0) | `Ok(None)` |

### 5.3 Direction Order Impact

The move order `[(0,1), (1,0), (0,-1), (-1,0)]` (Right, Down, Left, Up) affects:
- Which path is found first
- Performance for different maze layouts

```rust
// Current order: Right → Down → Left → Up
const MOVES: [(isize, isize); 4] = [(0, 1), (1, 0), (0, -1), (-1, 0)];

// Alternative: prioritize toward exit
// For exit at bottom-right: Down → Right → ... would be faster
```

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Robot Navigation**: Finding paths in physical spaces
2. **Game AI**: NPC pathfinding in tile-based games
3. **Network Routing**: Finding valid network paths
4. **Puzzle Games**: Maze-solving games
5. **VLSI Design**: Wire routing on circuit boards

### 6.2 Related Algorithms

| Algorithm | When to Use |
|-----------|-------------|
| BFS | Need shortest path |
| A* | Large mazes with heuristic available |
| Dijkstra | Weighted mazes |
| Wall Follower | Simple mazes (always solvable for simply connected) |
| Tremaux's Algorithm | Guaranteed to find exit in any maze |

## 7. Implementation Analysis

### 7.1 Current Implementation Review

```rust
fn solve(&self, x: isize, y: isize, solution: &mut [Vec<bool>]) -> bool {
    // Exit reached
    if x == (self.height() as isize - 1) && y == (self.width() as isize - 1) {
        solution[x as usize][y as usize] = true;
        return true;
    }

    if self.is_valid(x, y, solution) {
        solution[x as usize][y as usize] = true;

        for &(dx, dy) in &Self::MOVES {
            if self.solve(x + dx, y + dy, solution) {
                return true;
            }
        }

        // Backtrack
        solution[x as usize][y as usize] = false;
        return false;
    }
    false
}

fn is_valid(&self, x: isize, y: isize, solution: &[Vec<bool>]) -> bool {
    x >= 0
        && y >= 0
        && x < self.height() as isize
        && y < self.width() as isize
        && self.maze[x as usize][y as usize]      // Open cell
        && !solution[x as usize][y as usize]       // Not in path
}
```

**Analysis**:
- ✅ Correctly handles rectangular (non-square) mazes
- ✅ Proper bounds checking with `isize`
- ✅ Clean backtracking with state restoration
- ✅ Comprehensive input validation
- ⚠️ Doesn't find shortest path (use BFS for that)
- ⚠️ Returns first path found, not all paths

### 7.2 Complexity Verification

| Operation | Claimed | Verified |
|-----------|---------|----------|
| `is_valid` | O(1) | ✅ Simple checks |
| `solve` | O(4^(MN)) worst | ✅ 4 choices at each cell |
| Space | O(MN) | ✅ Solution matrix + stack |

## 8. Testing

### 8.1 Test Cases Covered

```rust
maze_with_solution_5x5: Standard maze with solution
maze_with_solution_6x6: Larger maze with solution  
maze_with_solution_8x8: Complex maze with solution
maze_without_solution_4x4: No valid path exists
maze_with_solution_3x4: Non-square maze
maze_without_solution_3x4: Non-square, no path
improper_maze_representation: Non-rectangular → Error
out_of_bound_start: Invalid start → Error
empty_maze: Empty input → Error
maze_with_single_cell: 1×1 maze → trivial solution
maze_with_one_row_and_multiple_columns: 1×5 maze
maze_with_multiple_rows_and_one_column: 4×1 maze
maze_with_walls_surrounding_border: No entry
maze_with_no_walls: All open (finds first path)
maze_with_going_back: Requires backtracking
```

### 8.2 Solution Validation

Tests verify:
1. Start position is marked in solution
2. Path is continuous (each cell adjacent to next)
3. Path ends at exit
4. Path only uses open cells

```rust
if let Ok(Some(expected_solution)) = &solution {
    assert_eq!(expected_solution[$start_x][$start_y], true);
}
```

## 9. Variations and Extensions

### 9.1 Diagonal Movement

```rust
const MOVES_8: [(isize, isize); 8] = [
    (0, 1), (1, 0), (0, -1), (-1, 0),     // Cardinal
    (1, 1), (1, -1), (-1, 1), (-1, -1),   // Diagonal
];
```

### 9.2 Weighted Cells

Transform boolean maze to cost matrix and use Dijkstra/A*.

### 9.3 Multiple Exits

```rust
fn solve_multiple_exits(&self, x: isize, y: isize, exits: &[(usize, usize)], ...) -> bool {
    if exits.contains(&(x as usize, y as usize)) {
        // Found an exit
        return true;
    }
    // ... rest of backtracking
}
```

### 9.4 Find All Paths

```rust
fn find_all_paths(&self, ..., all_paths: &mut Vec<Vec<Vec<bool>>>) {
    if at_exit {
        all_paths.push(solution.clone());
        return;  // Don't return true - continue searching
    }
    // ... continue exploring all possibilities
}
```

## 10. References

1. Cormen, T. H., et al. (2009). *Introduction to Algorithms*. Chapter 22 (Graph algorithms).
2. Sedgewick, R., & Wayne, K. (2011). *Algorithms*. Chapter 4 (Graphs).
3. Even, S. (2011). *Graph Algorithms*. Cambridge University Press.
4. [Wikipedia: Maze Solving Algorithm](https://en.wikipedia.org/wiki/Maze_solving_algorithm)
5. LaValle, S. M. (2006). *Planning Algorithms*. Cambridge University Press.
