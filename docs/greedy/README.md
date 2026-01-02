# Greedy Algorithms

## Overview

Greedy algorithms are a class of algorithms that make locally optimal choices at each step with the hope of finding a global optimum. These algorithms follow the problem-solving heuristic of making the choice that seems best at the moment without reconsidering previous choices.

## Key Characteristics

- **Local Optimization**: Makes the best decision at each step without looking ahead
- **Irrevocable Decisions**: Once a choice is made, it's never reconsidered
- **Efficiency**: Often faster than dynamic programming or exhaustive search
- **Not Always Optimal**: May not always produce the globally optimal solution

## Greedy Choice Property

A problem exhibits the **greedy choice property** if:
1. A globally optimal solution can be arrived at by making locally optimal choices
2. The choice made at each step is independent of future choices

## When to Use Greedy Algorithms

Greedy algorithms work well for problems with:
1. **Optimal Substructure**: Optimal solution contains optimal solutions to subproblems
2. **Greedy Choice Property**: Local optimal choices lead to global optimum
3. **Matroid Structure**: Mathematical structure guaranteeing greedy optimality

## Algorithms in This Category

### Stable Matching (Gale-Shapley Algorithm)

**File**: [stable_matching.md](stable_matching.md)

**Problem**: Find stable matchings between two equally sized sets where participants have preferences.

**Complexity**: O(n²) time, O(n²) space

**Key Applications**:
- Medical residency matching (NRMP)
- School choice systems
- Kidney exchange programs
- Job market matching

**Greedy Strategy**: Men propose to women in order of preference; women tentatively accept the best available option and may upgrade later.

**Why It Works**: The algorithm guarantees a stable matching where no blocking pairs exist. The greedy approach of always accepting the currently best proposal ensures stability.

## Classic Greedy Problems

### Activity Selection
- **Problem**: Select maximum number of non-overlapping activities
- **Greedy Choice**: Always pick activity that finishes earliest
- **Complexity**: O(n log n)

### Huffman Coding
- **Problem**: Optimal prefix-free code for data compression
- **Greedy Choice**: Merge two lowest-frequency symbols
- **Complexity**: O(n log n)

### Dijkstra's Shortest Path
- **Problem**: Find shortest paths from source to all vertices
- **Greedy Choice**: Always expand nearest unvisited vertex
- **Complexity**: O((V+E) log V) with priority queue

### Kruskal's MST
- **Problem**: Find minimum spanning tree of graph
- **Greedy Choice**: Always add cheapest edge that doesn't create cycle
- **Complexity**: O(E log E)

### Prim's MST
- **Problem**: Find minimum spanning tree of graph
- **Greedy Choice**: Always add cheapest edge connecting tree to non-tree vertex
- **Complexity**: O((V+E) log V)

### Fractional Knapsack
- **Problem**: Maximize value in knapsack allowing fractional items
- **Greedy Choice**: Take items in order of value-to-weight ratio
- **Complexity**: O(n log n)

## Comparison: Greedy vs Dynamic Programming

| Aspect | Greedy | Dynamic Programming |
|--------|--------|---------------------|
| **Decision Making** | Local optimum | Global optimum via subproblems |
| **Reconsideration** | Never reconsiders | Considers all possibilities |
| **Complexity** | Usually O(n log n) | Usually O(n²) or higher |
| **Optimality** | Not always guaranteed | Guaranteed if formulated correctly |
| **Examples** | Dijkstra, MST, Huffman | 0/1 Knapsack, LCS, Edit Distance |

## Proving Greedy Correctness

To prove a greedy algorithm is correct:

1. **Greedy Choice Property**: Show that making a greedy choice leads to an optimal solution
2. **Optimal Substructure**: Show that combining the greedy choice with an optimal solution to the remaining subproblem yields an optimal overall solution
3. **Exchange Argument**: Show that any optimal solution can be transformed into one that includes the greedy choice without making it worse

### Example: Stable Matching Proof Sketch

**Claim**: Gale-Shapley produces a stable matching.

**Proof by Contradiction**:
- Assume (m, w) is a blocking pair in the output
- Since m proposes in order of preference, m proposed to w before his current partner
- If w rejected or replaced m, she has someone she prefers more
- Contradiction: (m, w) cannot be a blocking pair

## Greedy Algorithm Template

```rust
fn greedy_algorithm<T>(elements: Vec<T>) -> Solution {
    let mut solution = initialize_solution();
    
    // Sort elements by greedy criterion
    let sorted = sort_by_greedy_criterion(elements);
    
    for element in sorted {
        if is_feasible(&solution, &element) {
            solution = add_to_solution(solution, element);
        }
    }
    
    solution
}
```

## Common Pitfalls

1. **Assuming Greedy Always Works**: Many problems (e.g., 0/1 Knapsack) don't have greedy solutions
2. **Wrong Greedy Criterion**: Choosing the wrong local optimization metric
3. **Not Proving Correctness**: Always verify greedy choice property and optimal substructure
4. **Ignoring Constraints**: Greedy choices must respect problem constraints

## When Greedy Fails: 0/1 Knapsack Example

**Problem**: Maximize value in knapsack with integer item quantities

**Why Greedy Fails**:
```
Items: (value, weight)
Item A: (60, 10)
Item B: (100, 20) 
Item C: (120, 30)
Capacity: 50

Greedy by value/weight ratio:
- A: 6.0 (take it)
- B: 5.0 (take it)  
- C: 4.0 (can't fit)
Total: 160

Optimal:
- B + C = 220 (better!)
```

**Solution**: Must use dynamic programming for 0/1 Knapsack.

## Further Reading

- **Introduction to Algorithms** (CLRS), Chapter 16: Greedy Algorithms
- **Algorithm Design** by Kleinberg & Tardos, Chapter 4: Greedy Algorithms
- **The Algorithm Design Manual** by Skiena, Chapter 10: Greedy Heuristics

## Related Topics

- [Graph Algorithms](../graph/README.md) - Dijkstra, MST algorithms
- [Dynamic Programming](../dynamic_programming/README.md) - Alternative to greedy when needed
- [Sorting](../sorting/README.md) - Often used as preprocessing step
