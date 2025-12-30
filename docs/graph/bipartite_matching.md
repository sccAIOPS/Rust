# Bipartite Matching (Hopcroft-Karp Algorithm)

## 1. Overview

The Hopcroft-Karp algorithm finds maximum cardinality matching in bipartite graphs in O(E√V) time. It improves upon the naive O(VE) Hungarian algorithm by finding multiple augmenting paths in each phase using BFS and DFS.

A **matching** is a set of edges with no common vertices. A **maximum matching** has the largest possible number of edges.

## 2. Mathematical Foundation

### 2.1 Bipartite Graph

A graph $G = (V, E)$ is **bipartite** if vertices can be partitioned into two disjoint sets $U$ and $V$ such that every edge connects a vertex in $U$ to one in $V$.

### 2.2 Augmenting Path

An **augmenting path** alternates between unmatched and matched edges, starting and ending at unmatched vertices.

### 2.3 Berge's Lemma

A matching $M$ is maximum if and only if there is no augmenting path with respect to $M$.

### 2.4 König's Theorem

In bipartite graphs:
$$\text{maximum matching} = \text{minimum vertex cover}$$

## 3. Algorithm

### 3.1 Overview

1. **BFS Phase:** Find shortest augmenting paths, construct level graph
2. **DFS Phase:** Find vertex-disjoint augmenting paths in level graph
3. **Repeat** until no augmenting path exists

### 3.2 Pseudocode

```
HOPCROFT-KARP(G):
    match_u[all] ← NIL
    match_v[all] ← NIL
    matching ← 0
    
    while BFS():  // Find augmenting paths
        for each u in U:
            if match_u[u] = NIL:
                if DFS(u):
                    matching += 1
    
    return matching

BFS():
    queue ← []
    for each u in U:
        if match_u[u] = NIL:
            dist[u] ← 0
            queue.enqueue(u)
        else:
            dist[u] ← ∞
    
    dist[NIL] ← ∞
    while queue not empty:
        u ← queue.dequeue()
        if dist[u] < dist[NIL]:
            for each v adjacent to u:
                if dist[match_v[v]] = ∞:
                    dist[match_v[v]] ← dist[u] + 1
                    queue.enqueue(match_v[v])
    
    return dist[NIL] ≠ ∞

DFS(u):
    if u ≠ NIL:
        for each v adjacent to u:
            if dist[match_v[v]] = dist[u] + 1:
                if DFS(match_v[v]):
                    match_v[v] ← u
                    match_u[u] ← v
                    return true
        dist[u] ← ∞
        return false
    return true
```

## 4. Example

```
U: {u1, u2, u3}     V: {v1, v2, v3}

Edges:
u1 — v1, v2
u2 — v1, v3
u3 — v2, v3
```

**Phase 1:** 
- BFS finds paths of length 1
- Match: (u1, v1), (u2, v3), (u3, v2)

**Maximum Matching = 3**

## 5. Complexity

| Metric | Complexity |
|--------|------------|
| Time | O(E√V) |
| Space | O(V + E) |

The √V bound comes from the fact that at most O(√V) phases are needed.

## 6. Implementation

```rust
use std::collections::VecDeque;

const NIL: usize = 0;

pub struct HopcroftKarp {
    adj: Vec<Vec<usize>>,    // Adjacency list for U vertices (1-indexed)
    match_u: Vec<usize>,     // match_u[u] = v if (u,v) matched
    match_v: Vec<usize>,     // match_v[v] = u if (u,v) matched
    dist: Vec<usize>,        // Distance labels for BFS
    u_size: usize,           // Number of U vertices
    v_size: usize,           // Number of V vertices
}

impl HopcroftKarp {
    pub fn new(u_size: usize, v_size: usize) -> Self {
        HopcroftKarp {
            adj: vec![vec![]; u_size + 1],
            match_u: vec![NIL; u_size + 1],
            match_v: vec![NIL; v_size + 1],
            dist: vec![0; u_size + 1],
            u_size,
            v_size,
        }
    }

    pub fn add_edge(&mut self, u: usize, v: usize) {
        self.adj[u].push(v);
    }

    fn bfs(&mut self) -> bool {
        let mut queue = VecDeque::new();
        
        for u in 1..=self.u_size {
            if self.match_u[u] == NIL {
                self.dist[u] = 0;
                queue.push_back(u);
            } else {
                self.dist[u] = usize::MAX;
            }
        }
        
        self.dist[NIL] = usize::MAX;
        
        while let Some(u) = queue.pop_front() {
            if self.dist[u] < self.dist[NIL] {
                for &v in &self.adj[u] {
                    let matched = self.match_v[v];
                    if self.dist[matched] == usize::MAX {
                        self.dist[matched] = self.dist[u] + 1;
                        queue.push_back(matched);
                    }
                }
            }
        }
        
        self.dist[NIL] != usize::MAX
    }

    fn dfs(&mut self, u: usize) -> bool {
        if u == NIL {
            return true;
        }
        
        // Use index iteration to avoid borrow issues
        for i in 0..self.adj[u].len() {
            let v = self.adj[u][i];
            let matched = self.match_v[v];
            
            if self.dist[matched] == self.dist[u] + 1 && self.dfs(matched) {
                self.match_v[v] = u;
                self.match_u[u] = v;
                return true;
            }
        }
        
        self.dist[u] = usize::MAX;
        false
    }

    pub fn max_matching(&mut self) -> usize {
        let mut matching = 0;
        
        while self.bfs() {
            for u in 1..=self.u_size {
                if self.match_u[u] == NIL && self.dfs(u) {
                    matching += 1;
                }
            }
        }
        
        matching
    }

    pub fn get_matching(&self) -> Vec<(usize, usize)> {
        let mut result = vec![];
        for u in 1..=self.u_size {
            if self.match_u[u] != NIL {
                result.push((u, self.match_u[u]));
            }
        }
        result
    }
}
```

## 7. Alternative: Hungarian Algorithm

Simple O(VE) algorithm using repeated DFS:

```rust
fn augment(u: usize, adj: &[Vec<usize>], visited: &mut [bool], 
           match_v: &mut [usize]) -> bool {
    for &v in &adj[u] {
        if !visited[v] {
            visited[v] = true;
            if match_v[v] == 0 || augment(match_v[v], adj, visited, match_v) {
                match_v[v] = u;
                return true;
            }
        }
    }
    false
}

fn hungarian(adj: &[Vec<usize>], u_size: usize, v_size: usize) -> usize {
    let mut match_v = vec![0; v_size + 1];
    let mut matching = 0;
    
    for u in 1..=u_size {
        let mut visited = vec![false; v_size + 1];
        if augment(u, adj, &mut visited, &mut match_v) {
            matching += 1;
        }
    }
    
    matching
}
```

## 8. Applications

1. **Job assignment:** Workers to tasks
2. **Course scheduling:** Students to courses
3. **Resource allocation:** Resources to requests
4. **Network flows:** Unit capacity bipartite flows
5. **Stable matching:** Related to Gale-Shapley algorithm

## 9. Related Problems

| Problem | Solution |
|---------|----------|
| Maximum matching | Hopcroft-Karp |
| Minimum vertex cover | König's theorem |
| Maximum independent set | V - min vertex cover |
| Weighted matching | Hungarian algorithm (O(V³)) |
| Stable matching | Gale-Shapley |

## 10. Comparison

| Algorithm | Time | Use Case |
|-----------|------|----------|
| Naive DFS | O(VE) | Small graphs |
| Hopcroft-Karp | O(E√V) | Large bipartite |
| Dinic's Flow | O(E√V) | Via max flow reduction |
| Push-Relabel | O(V√V·E) | Very dense graphs |

## 11. Common Pitfalls

1. **NIL handling:** Use sentinel value (0 or usize::MAX)
2. **1-indexed vs 0-indexed:** Be consistent
3. **Distance reset:** dist[u] = MAX when DFS fails
4. **Visited array:** Reset between BFS phases

## 12. References

- Hopcroft, J. E.; Karp, R. M. (1973). "An n^(5/2) Algorithm for Maximum Matchings in Bipartite Graphs"
- Cormen, T. H., et al. "Introduction to Algorithms"
- König, D. (1931). "Graphs and Matrices"
