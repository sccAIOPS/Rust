# Tarjan's Strongly Connected Components Algorithm

## 1. Overview

Tarjan's algorithm finds all Strongly Connected Components (SCCs) in a directed graph in a single DFS pass. An SCC is a maximal set of vertices where every vertex is reachable from every other vertex. The algorithm was discovered by Robert Tarjan in 1972.

## 2. Mathematical Foundation

### 2.1 Definitions

**Strongly Connected Component:** A maximal subgraph where for every pair of vertices $u, v$, there exists a path from $u$ to $v$ and from $v$ to $u$.

**Properties:**
- The SCC graph (condensation) is a DAG
- Every directed graph can be uniquely partitioned into SCCs

### 2.2 Key Concepts

- **index**: Discovery time of vertex in DFS
- **lowlink**: Smallest index reachable from vertex (including through back edges)
- **Stack**: Vertices in current DFS path that might belong to current SCC

## 3. Algorithm

### 3.1 Pseudocode

```
TARJAN-SCC(G):
    index ← 0
    stack ← []
    on_stack ← [false, ..., false]
    index_of ← [-1, ..., -1]
    lowlink ← [-1, ..., -1]
    components ← []
    
    for each vertex v:
        if index_of[v] = -1:
            STRONG-CONNECT(v)
    
    return components

STRONG-CONNECT(v):
    index_of[v] ← index
    lowlink[v] ← index
    index += 1
    stack.push(v)
    on_stack[v] ← true
    
    for each neighbor w of v:
        if index_of[w] = -1:
            STRONG-CONNECT(w)
            lowlink[v] ← min(lowlink[v], lowlink[w])
        else if on_stack[w]:
            lowlink[v] ← min(lowlink[v], index_of[w])
    
    // v is root of SCC
    if lowlink[v] = index_of[v]:
        component ← []
        repeat:
            w ← stack.pop()
            on_stack[w] ← false
            component.add(w)
        until w = v
        components.add(component)
```

### 3.2 Example

```
    0 → 1 → 2
    ↑   ↓   ↓
    3 ← 4   6
        ↓
        5
```

**DFS Execution:**
1. Start at 0: index_of[0]=0, lowlink[0]=0
2. Visit 1: index_of[1]=1, lowlink[1]=1
3. Visit 4: index_of[4]=2, lowlink[4]=2
4. Visit 3: index_of[3]=3, lowlink[3]=3
5. 3→0 (on stack): lowlink[3]=min(3,0)=0
6. Backtrack: lowlink[4]=0, lowlink[1]=0, lowlink[0]=0
7. At 0: lowlink[0]=index[0], pop SCC: {0,1,4,3}

**SCCs:** {0,1,4,3}, {2}, {5}, {6}

## 4. Complexity

| Metric | Complexity |
|--------|------------|
| Time | O(V + E) |
| Space | O(V) |

Single DFS pass makes it very efficient.

## 5. Implementation

```rust
pub struct Graph {
    n: usize,
    adj_list: Vec<Vec<usize>>,
}

pub fn tarjan_scc(graph: &Graph) -> Vec<Vec<usize>> {
    struct TarjanState {
        index: i32,
        stack: Vec<usize>,
        on_stack: Vec<bool>,
        index_of: Vec<i32>,
        lowlink_of: Vec<i32>,
        components: Vec<Vec<usize>>,
    }

    fn strong_connect(v: usize, graph: &Graph, state: &mut TarjanState) {
        state.index_of[v] = state.index;
        state.lowlink_of[v] = state.index;
        state.index += 1;
        state.stack.push(v);
        state.on_stack[v] = true;

        for &w in &graph.adj_list[v] {
            if state.index_of[w] == -1 {
                strong_connect(w, graph, state);
                state.lowlink_of[v] = state.lowlink_of[v].min(state.lowlink_of[w]);
            } else if state.on_stack[w] {
                state.lowlink_of[v] = state.lowlink_of[v].min(state.index_of[w]);
            }
        }

        if state.lowlink_of[v] == state.index_of[v] {
            let mut component = Vec::new();
            while let Some(w) = state.stack.pop() {
                state.on_stack[w] = false;
                component.push(w);
                if w == v { break; }
            }
            state.components.push(component);
        }
    }

    let mut state = TarjanState {
        index: 0,
        stack: Vec::new(),
        on_stack: vec![false; graph.n],
        index_of: vec![-1; graph.n],
        lowlink_of: vec![-1; graph.n],
        components: Vec::new(),
    };

    for v in 0..graph.n {
        if state.index_of[v] == -1 {
            strong_connect(v, graph, &mut state);
        }
    }

    state.components
}
```

## 6. Applications

1. **Compiler Optimization:** Finding strongly connected regions
2. **Social Networks:** Identifying tightly-knit groups
3. **Web Analysis:** Finding clusters of linked pages
4. **Circuit Analysis:** Finding feedback loops
5. **2-SAT Solver:** Solving boolean satisfiability
6. **Condensation Graph:** Reducing graph complexity

## 7. Tarjan's vs Kosaraju's

| Aspect | Tarjan's | Kosaraju's |
|--------|----------|------------|
| DFS passes | 1 | 2 |
| Space | O(V) extra | O(V + E) for transpose |
| Complexity | O(V + E) | O(V + E) |
| Implementation | More complex | Simpler concept |
| Output order | Reverse topological | Topological |

## 8. Key Insights

1. **Root Detection:** A vertex is an SCC root iff `lowlink[v] == index[v]`
2. **Stack Purpose:** Keeps vertices that might be in current SCC
3. **Lowlink Update:** Only from on-stack vertices (not cross edges)
4. **Output Order:** SCCs in reverse topological order of condensation

## 9. References

- Tarjan, R. E. (1972). "Depth-first search and linear graph algorithms"
- Cormen, T. H., et al. "Introduction to Algorithms", Chapter 22
