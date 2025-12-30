# A* Search Algorithm

## 1. Overview

A* (pronounced "A-star") is a best-first search algorithm that finds the shortest path between a start and goal node using a heuristic function to guide its search. It was developed by Peter Hart, Nils Nilsson, and Bertram Raphael in 1968 at Stanford Research Institute.

A* is widely used in pathfinding and graph traversal due to its completeness, optimality (with admissible heuristics), and efficiency.

## 2. Mathematical Foundation

### 2.1 Problem Definition

**Informed Single-Source Single-Target Shortest Path:**

Given a weighted graph $G = (V, E, w)$, a source $s$, a target $t$, and a heuristic function $h$, find the shortest path from $s$ to $t$.

**Input:**
- Graph $G$ with non-negative edge weights
- Source vertex $s$ and target vertex $t$
- Heuristic function $h: V \rightarrow \mathbb{R}^+$

**Output:**
- Shortest path from $s$ to $t$ and its cost
- Or indication that no path exists

### 2.2 Mathematical Model

**Evaluation Function:**
$$f(n) = g(n) + h(n)$$

Where:
- $g(n)$ = actual cost from start to node $n$
- $h(n)$ = estimated cost from $n$ to goal (heuristic)
- $f(n)$ = estimated total cost of path through $n$

**Heuristic Properties:**

1. **Admissible:** $h(n) \leq h^*(n)$ (never overestimates true cost)
2. **Consistent (Monotonic):** $h(n) \leq c(n, n') + h(n')$ for all successors $n'$

### 2.3 Optimality Proof

**Theorem:** A* with an admissible heuristic finds the optimal path.

**Proof Sketch:**
Let $C^*$ be the optimal path cost. At termination, suppose A* returns a non-optimal path with cost $C > C^*$.

- When A* terminated, the goal was dequeued with $f(\text{goal}) = g(\text{goal}) = C$
- Some node $n$ on the optimal path must be in the open set
- Since $h$ is admissible: $f(n) = g(n) + h(n) \leq g(n) + h^*(n) = C^*$
- But $f(n) \leq C^* < C$, so $n$ should have been expanded first
- Contradiction. Therefore A* is optimal.

## 3. Algorithm Description

### 3.1 Intuition

Imagine you're navigating a city with a GPS that shows:
1. How far you've actually traveled (g-score)
2. Estimated remaining distance (h-score via heuristic)

You always explore the location with the lowest total estimated distance. This combines the "certainty" of actual distance traveled with the "estimate" of remaining work.

### 3.2 Pseudocode

```
A-STAR(G, start, goal, h):
    // Priority queue ordered by f = g + h
    open_set ← priority queue with (h(start), start)
    came_from ← empty map
    g_score ← map with default value ∞
    g_score[start] ← 0
    
    while open_set is not empty:
        current ← open_set.pop_min()
        
        if current = goal:
            return reconstruct_path(came_from, current), g_score[goal]
        
        for each neighbor of current:
            tentative_g ← g_score[current] + w(current, neighbor)
            
            if tentative_g < g_score[neighbor]:
                came_from[neighbor] ← current
                g_score[neighbor] ← tentative_g
                f_score ← tentative_g + h(neighbor)
                
                if neighbor not in open_set:
                    open_set.add((f_score, neighbor))
                else:
                    open_set.decrease_key(neighbor, f_score)
    
    return failure  // No path exists
```

### 3.3 Step-by-Step Example

Grid pathfinding with Manhattan distance heuristic:
```
Start (S) at (0,0), Goal (G) at (3,2)
. . . G
. X X .
S . . .
```

Using Manhattan distance: $h(n) = |x_n - x_G| + |y_n - y_G|$

| Step | Current | g | h | f | Open Set |
|------|---------|---|---|---|----------|
| 0 | (0,0) | 0 | 5 | 5 | {(0,0): f=5} |
| 1 | (0,0) → neighbors | - | - | - | {(1,0): f=5, (0,1): f=5} |
| 2 | (1,0) | 1 | 4 | 5 | {(0,1): f=5, (2,0): f=5} |
| 3 | (2,0) | 2 | 3 | 5 | {(0,1): f=5, (3,0): f=5} |
| 4 | (3,0) | 3 | 2 | 5 | {(0,1): f=5, (3,1): f=5} |
| 5 | (3,1) | 4 | 1 | 5 | {(0,1): f=5, (3,2): f=5} |
| 6 | (3,2) | 5 | 0 | 5 | **Goal reached!** |

**Path:** (0,0) → (1,0) → (2,0) → (3,0) → (3,1) → (3,2), Cost = 5

## 4. Complexity Analysis

### 4.1 Time Complexity

| Case | Complexity | Explanation |
|------|------------|-------------|
| Best | $O(d)$ | Perfect heuristic, straight path |
| Average | $O(b^d)$ | Exponential in effective depth |
| With admissible h | $O(b^{d^*})$ | $d^*$ = optimal solution depth |

Where:
- $b$ = branching factor
- $d$ = solution depth
- The heuristic quality dramatically affects practical performance

**Comparison with Dijkstra:**
A* explores fewer nodes when the heuristic is informative, as it focuses the search toward the goal.

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Open set | $O(b^d)$ | All frontier nodes |
| Closed set / g_score | $O(b^d)$ | All expanded nodes |
| came_from | $O(b^d)$ | Path reconstruction |
| **Total** | $O(b^d)$ | |

Space is often the limiting factor for A* in practice.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::collections::{BTreeMap, BinaryHeap};
use num_traits::Zero;

pub fn astar<V: Ord + Copy, E: Ord + Copy + Add<Output = E> + Zero>(
    graph: &Graph<V, E>,
    start: V,
    target: V,
    heuristic: impl Fn(V) -> E,
) -> Option<(E, Vec<V>)> {
    let mut queue = BinaryHeap::new();
    let mut previous = BTreeMap::new();
    let mut weights = BTreeMap::new();
    
    weights.insert(start, E::zero());
    queue.push(Candidate {
        estimated_weight: heuristic(start),
        real_weight: E::zero(),
        state: start,
    });
    
    while let Some(Candidate { real_weight, state: current, .. }) = queue.pop() {
        if current == target {
            break;
        }
        
        for (&next, &weight) in &graph[&current] {
            let new_weight = real_weight + weight;
            if weights.get(&next).is_none_or(|&w| new_weight < w) {
                weights.insert(next, new_weight);
                queue.push(Candidate {
                    estimated_weight: new_weight + heuristic(next),
                    real_weight: new_weight,
                    state: next,
                });
                previous.insert(next, current);
            }
        }
    }
    
    // Reconstruct path
    let weight = weights.get(&target)?;
    let mut path = vec![target];
    let mut current = target;
    while current != start {
        current = *previous.get(&current)?;
        path.push(current);
    }
    path.reverse();
    Some((*weight, path))
}
```

**Implementation Notes:**
- Uses `BinaryHeap` (max-heap) with inverted comparison for min-heap behavior
- Heuristic passed as closure for flexibility
- Returns both path cost and actual path
- Generic over vertex and edge weight types

### 5.2 Common Heuristics

| Domain | Heuristic | Formula |
|--------|-----------|---------|
| Grid (4-way) | Manhattan | $|x_1-x_2| + |y_1-y_2|$ |
| Grid (8-way) | Chebyshev | $\max(|x_1-x_2|, |y_1-y_2|)$ |
| Grid (any angle) | Euclidean | $\sqrt{(x_1-x_2)^2 + (y_1-y_2)^2}$ |
| General | Dijkstra | $h(n) = 0$ |
| Road networks | Great-circle | Haversine formula |

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Start = Goal | Returns path [start], cost 0 |
| No path exists | Returns `None` |
| Zero-cost edges | Handled correctly |
| Null heuristic | Degenerates to Dijkstra |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Video Game Pathfinding**
   - NPC navigation
   - Real-time strategy unit movement
   - Puzzle solving (15-puzzle, Sokoban)

2. **Robotics**
   - Motion planning
   - Autonomous vehicle navigation
   - Drone path planning

3. **Network Routing**
   - When heuristic information available
   - Geographic routing protocols

4. **Logistics**
   - Route optimization with distance heuristics
   - Warehouse robot navigation

5. **Natural Language Processing**
   - Parsing (as search problem)
   - Speech recognition lattice search

### 6.2 Related Algorithms

| Algorithm | Difference |
|-----------|------------|
| Dijkstra | A* with h(n) = 0 |
| Greedy Best-First | Uses only h(n), not g(n) |
| IDA* | Iterative deepening A* (less memory) |
| SMA* | Memory-bounded A* |
| D* | Dynamic A* for changing environments |
| Jump Point Search | A* optimization for uniform grids |

## 7. Heuristic Design

### 7.1 Guidelines

1. **Admissibility Check:** Ensure $h(n) \leq h^*(n)$ always
2. **Consistency Check:** $h(n) \leq c(n, n') + h(n')$ for neighbors
3. **Informedness:** Higher $h$ values (while admissible) = fewer expansions
4. **Computation Cost:** Heuristic should be cheap to compute

### 7.2 Combining Heuristics

If $h_1$ and $h_2$ are both admissible:
- $h(n) = \max(h_1(n), h_2(n))$ is admissible and more informed
- $h(n) = h_1(n) + h_2(n)$ may not be admissible!

### 7.3 Common Mistakes

| Mistake | Consequence |
|---------|-------------|
| Inadmissible heuristic | May return suboptimal path |
| Inconsistent heuristic | May re-expand nodes, slower |
| Expensive heuristic | Computation overhead dominates |
| Zero heuristic | Wasted A* advantages |

## 8. Performance Optimizations

1. **Binary Heap Improvements**
   - Use array-based heap
   - Lazy deletion instead of decrease-key

2. **Preprocessing**
   - Landmark-based heuristics (ALT)
   - Contraction hierarchies
   - Precomputed lookup tables

3. **Pruning Techniques**
   - Jump Point Search for grids
   - Hierarchical path-finding (HPA*)

4. **Memory Optimizations**
   - IDA* for limited memory
   - Beam search for approximate solutions

## 9. References

- Hart, P. E., Nilsson, N. J., & Raphael, B. (1968). "A Formal Basis for the Heuristic Determination of Minimum Cost Paths"
- Hart, P. E., Nilsson, N. J., & Raphael, B. (1972). "Correction to 'A Formal Basis...'"
- Russell, S. & Norvig, P. "Artificial Intelligence: A Modern Approach", Chapter 3
- Cormen, T. H., et al. "Introduction to Algorithms" (CLRS) - for underlying shortest path concepts
