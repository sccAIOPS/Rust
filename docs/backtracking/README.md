# Backtracking Algorithms

This directory contains comprehensive documentation for backtracking algorithms implemented in the **TheAlgorithms/Rust** repository.

## Overview

**Backtracking** is a general algorithmic technique that considers searching every possible combination to solve computational problems. It's an algorithmic paradigm that incrementally builds candidates to the solutions and abandons a candidate ("backtracks") as soon as it determines that the candidate cannot possibly lead to a valid solution.

### Key Characteristics

- **Systematic exploration**: Explores all possible configurations in a structured way
- **Pruning**: Eliminates configurations that violate constraints early
- **Depth-first search**: Uses DFS-style exploration of the solution space
- **State restoration**: Reverts state changes when backtracking

### General Template

```
function backtrack(state):
    if state is a solution:
        record/return solution
        return
    
    for each choice in available_choices(state):
        if is_valid(choice, state):
            make_choice(choice, state)
            backtrack(state)
            undo_choice(choice, state)  // backtrack
```

## Algorithm Index

| # | Algorithm | File | Description | Complexity |
|---|-----------|------|-------------|------------|
| 1 | [N-Queens](n_queens.md) | `n_queens.rs` | Place N queens on NxN board without conflicts | $O(N!)$ |
| 2 | [Sudoku Solver](sudoku.md) | `sudoku.rs` | Solve 9x9 Sudoku puzzles | $O(9^{81})$ worst |
| 3 | [Permutations](permutations.md) | `permutations.rs` | Generate all distinct permutations | $O(N! \cdot N)$ |
| 4 | [Combinations](all_combination_of_size_k.md) | `all_combination_of_size_k.rs` | Generate C(n,k) combinations | $O(C(n,k) \cdot k)$ |
| 5 | [Subset Sum](subset_sum.md) | `subset_sum.rs` | Find subset with target sum | $O(2^N)$ |
| 6 | [Graph Coloring](graph_coloring.md) | `graph_coloring.rs` | Color graph vertices with k colors | $O(k^V)$ |
| 7 | [Hamiltonian Cycle](hamiltonian_cycle.md) | `hamiltonian_cycle.rs` | Find cycle visiting all vertices once | $O(N!)$ |
| 8 | [Knight's Tour](knight_tour.md) | `knight_tour.rs` | Knight visits all squares exactly once | $O(8^{N^2})$ |
| 9 | [Rat in Maze](rat_in_maze.md) | `rat_in_maze.rs` | Find path through maze | $O(4^{N \times M})$ |
| 10 | [Parentheses Generator](parentheses_generator.md) | `parentheses_generator.rs` | Generate valid parentheses combinations | $O(4^N / \sqrt{N})$ |

## Classification by Problem Type

### Constraint Satisfaction Problems
- **N-Queens**: Place pieces without attacks
- **Sudoku**: Fill grid following rules
- **Graph Coloring**: Color vertices without adjacent same colors

### Path Finding Problems
- **Hamiltonian Cycle**: Visit all vertices exactly once
- **Knight's Tour**: Visit all squares exactly once
- **Rat in Maze**: Navigate from start to end

### Combinatorial Generation
- **Permutations**: All orderings of elements
- **Combinations**: All subsets of size k
- **Subset Sum**: Subsets meeting target
- **Parentheses Generator**: Valid bracket sequences

## Complexity Comparison

| Algorithm | Time Complexity | Space Complexity | Pruning Effectiveness |
|-----------|-----------------|------------------|----------------------|
| N-Queens | $O(N!)$ | $O(N^2)$ | High |
| Sudoku | $O(9^{81})$ worst | $O(81)$ | Very High |
| Permutations | $O(N! \cdot N)$ | $O(N)$ | Medium |
| Combinations | $O(C(n,k) \cdot k)$ | $O(k)$ | High |
| Subset Sum | $O(2^N)$ | $O(N)$ | Low |
| Graph Coloring | $O(k^V)$ | $O(V)$ | High |
| Hamiltonian Cycle | $O(N!)$ | $O(N)$ | Medium |
| Knight's Tour | $O(8^{N^2})$ | $O(N^2)$ | Medium |
| Rat in Maze | $O(4^{N \times M})$ | $O(N \times M)$ | High |
| Parentheses | $O(4^N / \sqrt{N})$ | $O(N)$ | Very High |

## When to Use Backtracking

### Good Use Cases
- Finding all solutions to a constraint satisfaction problem
- Generating all permutations/combinations
- Decision problems where constraints can prune the search space
- Problems with good heuristics for ordering choices

### Avoid When
- The search space is too large without effective pruning
- A polynomial-time algorithm exists (DP, greedy)
- Real-time response is required for large inputs

## Common Optimizations

### 1. Constraint Propagation
Reduce domain of variables after each choice (e.g., Sudoku)

### 2. Variable Ordering (MRV)
Choose the variable with Minimum Remaining Values first

### 3. Value Ordering (LCV)
Choose the Least Constraining Value first

### 4. Symmetry Breaking
Avoid exploring symmetric configurations

### 5. Intelligent Backtracking
Jump back to the source of conflict, not just one level

## Related Algorithms

- **Branch and Bound**: Backtracking with cost bounds
- **Constraint Programming**: General framework with sophisticated propagation
- **SAT Solvers**: DPLL algorithm is specialized backtracking
- **IDA***: Iterative deepening with backtracking

## References

1. Knuth, D. E. (2000). *Dancing Links*. arXiv preprint cs/0011047.
2. Russell, S., & Norvig, P. (2020). *Artificial Intelligence: A Modern Approach*. Chapter 6.
3. Cormen, T. H., et al. (2022). *Introduction to Algorithms*. MIT Press.
4. Skiena, S. S. (2020). *The Algorithm Design Manual*. Springer.
