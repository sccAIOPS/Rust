# Lee's BFS Algorithm (Maze Shortest Path)

## 1. Overview

Lee's algorithm is a BFS-based pathfinding algorithm specifically designed for maze/grid routing. It finds the shortest path in a 2D grid with obstacles, commonly used in VLSI circuit routing and robotics path planning.

The algorithm was developed by C. Y. Lee in 1961 for printed circuit board routing.

## 2. Mathematical Foundation

### 2.1 Grid Model

The maze is modeled as a 2D grid where:
- Each cell can be passable (0) or blocked (1)
- Movement is typically 4-directional (up, down, left, right)
- All moves have equal cost (unweighted)

### 2.2 BFS Optimality

For unweighted graphs/grids, BFS guarantees the shortest path because:
- All edges have cost 1
- BFS explores nodes in order of distance from source
- First time we reach a node is via shortest path

## 3. Algorithm

### 3.1 Wave Expansion

1. **Initialization:** Mark source cell with distance 0
2. **Expansion:** For each cell at distance d, mark unvisited neighbors as d+1
3. **Termination:** Stop when destination is reached or all reachable cells visited
4. **Backtracking:** Trace path from destination to source following decreasing distances

### 3.2 Pseudocode

```
LEE-BFS(maze, source, destination):
    if source = destination:
        return [source]
    
    visited[source] ← true
    distance[source] ← 0
    queue ← [source]
    parent ← {}
    
    while queue not empty:
        cell ← queue.dequeue()
        
        for each neighbor in get_neighbors(cell):
            if valid(neighbor) and not visited[neighbor]:
                visited[neighbor] ← true
                distance[neighbor] ← distance[cell] + 1
                parent[neighbor] ← cell
                
                if neighbor = destination:
                    return reconstruct_path(parent, destination)
                
                queue.enqueue(neighbor)
    
    return None  // No path exists

get_neighbors(cell):
    return [(cell.row-1, cell.col),   // up
            (cell.row+1, cell.col),   // down
            (cell.row, cell.col-1),   // left
            (cell.row, cell.col+1)]   // right
```

## 4. Example

```
Maze (0=passable, 1=blocked):
0 0 0 1 0
0 1 0 1 0
0 1 0 0 0
0 0 1 1 0
1 0 0 0 0

Source: (0,0), Destination: (4,4)

BFS Wave:
0 1 2 . .
1 . 3 . .
2 . 4 5 6
3 4 . . 7
. 5 6 7 8

Path: (0,0) → (1,0) → (2,0) → (3,0) → (3,1) → (4,1) → (4,2) → (4,3) → (4,4)
Length: 8
```

## 5. Complexity

| Metric | Complexity |
|--------|------------|
| Time | O(R × C) |
| Space | O(R × C) |

Where R = rows, C = columns.

## 6. Implementation

```rust
use std::collections::VecDeque;

#[derive(Clone, Copy, PartialEq, Eq)]
pub struct Cell {
    row: usize,
    col: usize,
}

pub fn lee_bfs(
    maze: &[Vec<i32>],
    source: Cell,
    destination: Cell,
) -> Option<Vec<Cell>> {
    let rows = maze.len();
    if rows == 0 { return None; }
    let cols = maze[0].len();

    // Validate source and destination
    if !is_valid(&maze, source) || !is_valid(&maze, destination) {
        return None;
    }

    if source == destination {
        return Some(vec![source]);
    }

    // Direction vectors: up, down, left, right
    let dr: [i32; 4] = [-1, 1, 0, 0];
    let dc: [i32; 4] = [0, 0, -1, 1];

    let mut visited = vec![vec![false; cols]; rows];
    let mut parent = vec![vec![None; cols]; rows];
    let mut queue = VecDeque::new();

    visited[source.row][source.col] = true;
    queue.push_back(source);

    while let Some(cell) = queue.pop_front() {
        for i in 0..4 {
            let new_row = cell.row as i32 + dr[i];
            let new_col = cell.col as i32 + dc[i];

            if new_row >= 0 && new_row < rows as i32 
               && new_col >= 0 && new_col < cols as i32 {
                let nr = new_row as usize;
                let nc = new_col as usize;

                if !visited[nr][nc] && maze[nr][nc] == 0 {
                    visited[nr][nc] = true;
                    parent[nr][nc] = Some(cell);

                    if nr == destination.row && nc == destination.col {
                        return Some(reconstruct_path(&parent, destination));
                    }

                    queue.push_back(Cell { row: nr, col: nc });
                }
            }
        }
    }

    None  // No path found
}

fn is_valid(maze: &[Vec<i32>], cell: Cell) -> bool {
    cell.row < maze.len() 
        && cell.col < maze[0].len() 
        && maze[cell.row][cell.col] == 0
}

fn reconstruct_path(
    parent: &[Vec<Option<Cell>>],
    destination: Cell,
) -> Vec<Cell> {
    let mut path = vec![destination];
    let mut current = destination;

    while let Some(prev) = parent[current.row][current.col] {
        path.push(prev);
        current = prev;
    }

    path.reverse();
    path
}
```

## 7. Variations

### 7.1 Eight-Directional Movement

```rust
let dr: [i32; 8] = [-1, -1, -1, 0, 0, 1, 1, 1];
let dc: [i32; 8] = [-1, 0, 1, -1, 1, -1, 0, 1];
```

### 7.2 Weighted Grid (Use Dijkstra/A*)

For cells with different traversal costs, use priority queue instead of regular queue.

### 7.3 Multi-Source BFS

Start BFS from multiple sources simultaneously for problems like "nearest exit".

## 8. Comparison with Other Pathfinding

| Algorithm | Best For | Time | Guarantees |
|-----------|----------|------|------------|
| Lee's BFS | Unweighted grids | O(RC) | Shortest path |
| A* | Heuristic available | O(RC) avg | Shortest path |
| Dijkstra | Weighted grids | O(RC log(RC)) | Shortest path |
| DFS | Any path needed | O(RC) | Not shortest |

## 9. Applications

1. **VLSI Routing:** Connecting pins on circuit boards
2. **Robotics:** Path planning in known environments
3. **Game AI:** NPC movement in tile-based games
4. **Image Processing:** Connected component labeling
5. **Navigation:** Simple grid-based GPS routing

## 10. Optimizations

### 10.1 Bidirectional BFS

Search from both source and destination, meet in middle:
- Time: O(2 × R×C / 2) in best case
- Helpful for large grids

### 10.2 Early Termination

Stop as soon as destination is reached (already implemented above).

### 10.3 Memory Optimization

For very large grids:
- Use bit arrays for visited
- Store only frontier, not all distances

## 11. Common Pitfalls

1. **Off-by-one errors:** Grid bounds checking
2. **Revisiting cells:** Always check visited before queuing
3. **Queue vs Stack:** Using stack gives DFS, not shortest path
4. **Path reconstruction:** Store parent pointers or use BFS levels

## 12. References

- Lee, C. Y. (1961). "An algorithm for path connections and its applications"
- Moore, E. F. (1959). "The shortest path through a maze"
- Cormen, T. H., et al. "Introduction to Algorithms"
