# 2-SAT (Two Satisfiability)

## 1. Overview

2-SAT (2-Satisfiability) is the problem of determining whether a boolean formula in Conjunctive Normal Form (CNF) with exactly 2 literals per clause can be satisfied. Unlike general SAT which is NP-complete, 2-SAT can be solved in linear time using implication graphs and strongly connected components.

## 2. Mathematical Foundation

### 2.1 2-CNF Formula

A 2-CNF formula is a conjunction of clauses, each containing exactly 2 literals:
$$(x_1 \lor x_2) \land (\neg x_3 \lor x_4) \land (x_1 \lor \neg x_2) \land \ldots$$

### 2.2 Implication Form

Each clause $(a \lor b)$ is equivalent to two implications:
$$(\neg a \Rightarrow b) \land (\neg b \Rightarrow a)$$

### 2.3 Implication Graph

Construct directed graph where:
- Vertices: Each variable $x_i$ and its negation $\neg x_i$
- Edges: $a \rightarrow b$ for each implication $a \Rightarrow b$

### 2.4 Key Theorem

A 2-SAT formula is **satisfiable** if and only if for no variable $x$, both $x$ and $\neg x$ are in the same strongly connected component.

## 3. Algorithm

### 3.1 Steps

1. Build implication graph from clauses
2. Find all SCCs (using Kosaraju or Tarjan)
3. Check if any variable and its negation are in same SCC
4. If satisfiable, assign values based on topological order of SCCs

### 3.2 Pseudocode

```
2-SAT(clauses):
    // Build implication graph
    for each clause (a ∨ b):
        add_edge(¬a, b)
        add_edge(¬b, a)
    
    // Find SCCs
    sccs = TARJAN-SCC(graph)
    
    // Check satisfiability
    for each variable x:
        if scc[x] = scc[¬x]:
            return UNSATISFIABLE
    
    // Assign values (SCCs in reverse topological order)
    for each variable x:
        if scc[x] > scc[¬x]:  // ¬x comes before x in reverse topo order
            assignment[x] = true
        else:
            assignment[x] = false
    
    return assignment
```

### 3.3 Value Assignment Intuition

- SCCs form a DAG
- If $x \Rightarrow y$ and both satisfiable, choose $y = true$ before $x = true$
- Process SCCs in reverse topological order
- Set variable to true if its SCC comes after its negation's SCC

## 4. Example

```
Formula: (x₁ ∨ x₂) ∧ (¬x₁ ∨ x₃) ∧ (¬x₂ ∨ ¬x₃) ∧ (x₁ ∨ ¬x₂)

Implications:
¬x₁ → x₂,  ¬x₂ → x₁
x₁ → x₃,   ¬x₃ → ¬x₁  
x₂ → ¬x₃,  x₃ → ¬x₂
¬x₁ → ¬x₂, x₂ → x₁

Implication Graph:
¬x₁ → x₂ → x₁ → x₃ → ¬x₂ → ¬x₃ → ¬x₁
         ↑               ↓
         └───────────────┘

SCCs: {¬x₁, x₂}, {x₁, ¬x₂}, {x₃}, {¬x₃}

Check: x₁ and ¬x₁ in different SCCs ✓
       x₂ and ¬x₂ in different SCCs ✓
       x₃ and ¬x₃ in different SCCs ✓

SATISFIABLE

Assignment (reverse topo order):
scc[x₁] > scc[¬x₁] → x₁ = true
scc[x₂] < scc[¬x₂] → x₂ = false
scc[x₃] > scc[¬x₃] → x₃ = true

Verify: (T ∨ F) ∧ (F ∨ T) ∧ (T ∨ F) ∧ (T ∨ T) = T ✓
```

## 5. Complexity

| Operation | Time | Space |
|-----------|------|-------|
| Build graph | O(m) | O(n + m) |
| Find SCCs | O(n + m) | O(n) |
| Total | O(n + m) | O(n + m) |

Where n = number of variables, m = number of clauses.

## 6. Implementation

```rust
pub struct TwoSat {
    n: usize,  // Number of variables
    adj: Vec<Vec<usize>>,     // Implication graph
    adj_rev: Vec<Vec<usize>>, // Reverse graph for Kosaraju
}

impl TwoSat {
    pub fn new(n: usize) -> Self {
        TwoSat {
            n,
            adj: vec![vec![]; 2 * n],
            adj_rev: vec![vec![]; 2 * n],
        }
    }

    // Encode variable: x_i (true) -> 2*i, x_i (false) -> 2*i + 1
    fn var(&self, x: usize, is_true: bool) -> usize {
        if is_true { 2 * x } else { 2 * x + 1 }
    }

    fn neg(&self, v: usize) -> usize {
        v ^ 1
    }

    /// Add clause (a ∨ b) where a = (var_a, sign_a), b = (var_b, sign_b)
    /// sign = true means positive literal, false means negated
    pub fn add_clause(&mut self, var_a: usize, sign_a: bool, var_b: usize, sign_b: bool) {
        // (a ∨ b) ≡ (¬a → b) ∧ (¬b → a)
        let a = self.var(var_a, sign_a);
        let b = self.var(var_b, sign_b);
        
        // ¬a → b
        self.adj[self.neg(a)].push(b);
        self.adj_rev[b].push(self.neg(a));
        
        // ¬b → a
        self.adj[self.neg(b)].push(a);
        self.adj_rev[a].push(self.neg(b));
    }

    /// Add implication a → b
    pub fn add_implication(&mut self, var_a: usize, sign_a: bool, var_b: usize, sign_b: bool) {
        let a = self.var(var_a, sign_a);
        let b = self.var(var_b, sign_b);
        
        self.adj[a].push(b);
        self.adj_rev[b].push(a);
        
        // Contrapositive: ¬b → ¬a
        self.adj[self.neg(b)].push(self.neg(a));
        self.adj_rev[self.neg(a)].push(self.neg(b));
    }

    /// Solve 2-SAT, return assignment if satisfiable
    pub fn solve(&self) -> Option<Vec<bool>> {
        let num_vertices = 2 * self.n;
        
        // Kosaraju's algorithm
        let mut order = Vec::new();
        let mut visited = vec![false; num_vertices];
        
        // First DFS: get finish order
        for v in 0..num_vertices {
            if !visited[v] {
                self.dfs1(v, &mut visited, &mut order);
            }
        }
        
        // Second DFS: find SCCs
        let mut scc_id = vec![0; num_vertices];
        let mut current_scc = 0;
        visited.fill(false);
        
        for &v in order.iter().rev() {
            if !visited[v] {
                self.dfs2(v, &mut visited, &mut scc_id, current_scc);
                current_scc += 1;
            }
        }
        
        // Check satisfiability
        let mut assignment = vec![false; self.n];
        for i in 0..self.n {
            let pos = self.var(i, true);
            let neg = self.var(i, false);
            
            if scc_id[pos] == scc_id[neg] {
                return None;  // Unsatisfiable
            }
            
            // Assign based on SCC order (later SCC = higher id)
            assignment[i] = scc_id[pos] > scc_id[neg];
        }
        
        Some(assignment)
    }

    fn dfs1(&self, v: usize, visited: &mut [bool], order: &mut Vec<usize>) {
        visited[v] = true;
        for &u in &self.adj[v] {
            if !visited[u] {
                self.dfs1(u, visited, order);
            }
        }
        order.push(v);
    }

    fn dfs2(&self, v: usize, visited: &mut [bool], scc_id: &mut [usize], id: usize) {
        visited[v] = true;
        scc_id[v] = id;
        for &u in &self.adj_rev[v] {
            if !visited[u] {
                self.dfs2(u, visited, scc_id, id);
            }
        }
    }
}
```

## 7. Common Clause Patterns

| Constraint | Clause Form | Implications |
|------------|-------------|--------------|
| At least one true | $(x \lor y)$ | $\neg x \rightarrow y$, $\neg y \rightarrow x$ |
| At most one true | $(\neg x \lor \neg y)$ | $x \rightarrow \neg y$, $y \rightarrow \neg x$ |
| Exactly one true | Both above | Combined |
| x must be true | $(x \lor x)$ | $\neg x \rightarrow x$ |
| If x then y | $(\neg x \lor y)$ | $x \rightarrow y$, $\neg y \rightarrow \neg x$ |

## 8. Applications

1. **Circuit satisfiability:** Check if circuit can produce output 1
2. **Scheduling:** Constraints with pairs of tasks
3. **Graph coloring:** 2-colorability (bipartiteness)
4. **Type inference:** Simple type constraints
5. **Game solving:** Puzzle satisfiability

## 9. Extensions

### 9.1 Weighted 2-SAT

Maximize number of satisfied clauses (MAX-2-SAT is NP-hard).

### 9.2 Quantified 2-SAT

With quantifiers, remains polynomial.

### 9.3 Counting Solutions

#2-SAT is #P-complete.

## 10. Comparison with General SAT

| Aspect | 2-SAT | 3-SAT | k-SAT (k≥3) |
|--------|-------|-------|-------------|
| Complexity | P | NP-complete | NP-complete |
| Algorithm | Linear time | Exponential | Exponential |
| Practical solver | SCC-based | DPLL/CDCL | DPLL/CDCL |

## 11. Edge Cases

| Case | Result |
|------|--------|
| Empty formula | Satisfiable (any assignment) |
| $(x \lor \neg x)$ | Always satisfiable (tautology) |
| $(x) \land (\neg x)$ | Unsatisfiable |
| No clauses | Satisfiable |

## 12. Common Pitfalls

1. **Variable encoding:** Keep track of positive/negative literal indices
2. **Both implications:** Each clause creates TWO edges
3. **SCC comparison:** Compare SCC IDs, not SCCs directly
4. **Self-loops:** $(x \lor x)$ forces $x = true$

## 13. References

- Aspvall, B.; Plass, M. F.; Tarjan, R. E. (1979). "A linear-time algorithm for testing the truth of certain quantified boolean formulas"
- Papadimitriou, C. H. "Computational Complexity", Chapter 9
- Even, S.; Itai, A.; Shamir, A. (1976). "On the complexity of timetable and multicommodity flow problems"
