# Suffix Tree

## 1. Overview

A suffix tree is a compressed trie (radix tree) containing all suffixes of a given text string. It enables many fast operations on strings, including pattern matching in linear time, finding the longest repeated substring, and solving many other string problems efficiently.

Invented by Peter Weiner in 1973, suffix trees are one of the most important data structures in stringology.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a string $S$ of length $n$, construct a rooted tree where:
- Each edge is labeled with a non-empty substring of $S$
- Each internal node (except root) has at least two children
- All suffixes of $S$ appear as paths from root to leaves

### 2.2 Mathematical Properties

**Key Properties:**
1. Exactly $n$ leaves (one per suffix)
2. At most $n - 1$ internal nodes
3. At most $2n - 1$ nodes total
4. Total edge label length: $O(n^2)$ naive, $O(n)$ with edge compression

**Edge Compression:**
Instead of storing entire substrings on edges, store $(start\_index, end\_index)$ pairs representing positions in the original string.

### 2.3 Tree Structure

For string "banana$" ($ is sentinel):

```
                    (root)
                   /  |  \  \
                  /   |   \  \
                 a    b    n  $
                 |    |    |
               (na)  ana  (a)
              /    \   $    \
             na$   $        na$
             |              |
             $              $

Leaves represent suffixes:
- banana$
- anana$
- nana$
- ana$
- na$
- a$
- $
```

## 3. Algorithm Description

### 3.1 Intuition (Simple Construction)

The implementation uses a straightforward approach:
1. Start with an empty tree (root node)
2. For each suffix $S[i..]$, insert it into the tree
3. When inserting, follow existing edges as long as they match
4. Split edges or create new nodes as needed

### 3.2 Pseudocode

```
structure Node:
    sub: String       // Edge label (substring)
    children: [Node]  // Child nodes

function BUILD_SUFFIX_TREE(s):
    root = new Node("", [])
    
    for i = 0 to len(s) - 1:
        suffix = s[i:]
        add_suffix(root, suffix)
    
    return root

function ADD_SUFFIX(node, suffix):
    n = node  // Current node
    i = 0     // Position in suffix
    
    while i < len(suffix):
        // Find child that matches current character
        found = false
        for x, child in enumerate(n.children):
            if child.sub[0] == suffix[i]:
                // Found matching child, follow edge
                j = 0
                while j < len(child.sub) and i + j < len(suffix):
                    if suffix[i + j] != child.sub[j]:
                        // Mismatch within edge - split
                        split_node(n, child, x, suffix[i:], j)
                        return
                    j += 1
                
                if j == len(child.sub):
                    // Exhausted edge, continue to child
                    i += j
                    n = child
                    found = true
                    break
        
        if not found:
            // No matching edge, create new leaf
            new_child = Node(suffix[i:], [])
            n.children.append(new_child)
            return

function SPLIT_NODE(parent, child, child_idx, suffix, split_pos):
    // Create internal node at split point
    internal = Node(child.sub[:split_pos], [])
    
    // Update original child
    child.sub = child.sub[split_pos:]
    internal.children.append(child)
    
    // Add new suffix as sibling
    new_leaf = Node(suffix[split_pos:], [])
    internal.children.append(new_leaf)
    
    // Replace child with internal node
    parent.children[child_idx] = internal
```

### 3.3 Step-by-Step Example

**String:** "banana$"

**Insert "banana$":**
```
root → "banana$"
```

**Insert "anana$":**
```
root → "banana$"
     → "anana$"
```

**Insert "nana$":**
```
root → "banana$"
     → "anana$"
     → "nana$"
```

**Insert "ana$":**
Need to split "anana$" at position 3:
```
root → "banana$"
     → "a" → "nana$"
           → "na$"
     → "nana$"
```

**Continue for remaining suffixes...**

## 4. Complexity Analysis

### 4.1 Time Complexity

| Algorithm | Construction | Pattern Search |
|-----------|-------------|----------------|
| Simple (current) | $O(n^2)$ | $O(m)$ |
| Ukkonen's | $O(n)$ | $O(m)$ |
| Weiner's | $O(n)$ | $O(m)$ |
| McCreight's | $O(n)$ | $O(m)$ |

**Current Implementation:** $O(n^2)$ due to naive suffix insertion.

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Nodes | $O(n)$ |
| Edge labels (as strings) | $O(n^2)$ |
| Edge labels (as indices) | $O(n)$ |
| Children vectors | $O(n)$ |

**Current Implementation:** $O(n^2)$ due to explicit string storage.

## 5. Implementation Notes

### 5.1 Rust Implementation

```rust
#[derive(Debug, PartialEq, Eq, Clone)]
pub struct Node {
    pub sub: String,      // Edge label substring
    pub ch: Vec<usize>,   // Indices of children in nodes array
}

pub struct SuffixTree {
    pub nodes: Vec<Node>,
}

impl SuffixTree {
    pub fn new(s: &str) -> Self {
        let mut suf_tree = SuffixTree {
            nodes: vec![Node::empty()],
        };
        for i in 0..s.len() {
            suf_tree.add_suffix(&s[i..]);
        }
        suf_tree
    }
}
```

**Design Choices:**
- Nodes stored in flat `Vec` (cache-friendly)
- Children are indices into node array (avoids pointer complexity)
- Edge labels stored as `String` (simple but space-inefficient)

### 5.2 Potential Optimizations

```rust
// Space-efficient edge representation
struct EdgeLabel {
    start: usize,
    end: usize,  // or use special value for "to end of string"
}

struct OptimizedNode {
    label: EdgeLabel,
    children: SmallVec<[usize; 4]>,  // Usually few children
    suffix_link: Option<usize>,       // For Ukkonen's algorithm
}
```

### 5.3 Edge Cases

| Input | Behavior |
|-------|----------|
| Empty string | Single root node |
| Single character | Root with one child |
| All same chars "aaa" | Linear chain |
| Unique chars "abc" | Flat tree from root |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Pattern Matching:**
   - O(m) pattern search after O(n) construction
   - Find all occurrences in O(m + k) where k = occurrences

2. **Longest Repeated Substring:**
   - Find deepest internal node
   - O(n) after construction

3. **Longest Common Substring:**
   - Generalized suffix tree for multiple strings
   - Find deepest internal node with leaves from all strings

4. **Bioinformatics:**
   - DNA sequence analysis
   - Genome alignment
   - Finding repetitive elements

5. **Data Compression:**
   - LZ77/LZ78 compression
   - Finding repeated patterns

### 6.2 Common Operations

**Pattern Search:**
```rust
fn search(&self, pattern: &str) -> bool {
    let mut node_idx = 0;
    let mut pattern_pos = 0;
    
    while pattern_pos < pattern.len() {
        let node = &self.nodes[node_idx];
        let mut found = false;
        
        for &child_idx in &node.ch {
            let child = &self.nodes[child_idx];
            if pattern[pattern_pos..].starts_with(&child.sub) {
                pattern_pos += child.sub.len();
                node_idx = child_idx;
                found = true;
                break;
            } else if child.sub.starts_with(&pattern[pattern_pos..]) {
                return true;  // Pattern ends mid-edge
            }
        }
        
        if !found { return false; }
    }
    true
}
```

**Longest Repeated Substring:**
```rust
fn longest_repeated(&self) -> String {
    let mut best = String::new();
    let mut best_depth = 0;
    
    fn dfs(tree: &SuffixTree, node_idx: usize, depth: usize, 
           best: &mut String, best_depth: &mut usize, path: &mut String) {
        let node = &tree.nodes[node_idx];
        
        if node.ch.len() >= 2 && depth > *best_depth {
            *best_depth = depth;
            *best = path.clone();
        }
        
        for &child_idx in &node.ch {
            let child = &tree.nodes[child_idx];
            path.push_str(&child.sub);
            dfs(tree, child_idx, depth + child.sub.len(), best, best_depth, path);
            path.truncate(path.len() - child.sub.len());
        }
    }
    
    dfs(self, 0, 0, &mut best, &mut best_depth, &mut String::new());
    best
}
```

## 7. Comparison with Suffix Array

| Aspect | Suffix Tree | Suffix Array |
|--------|-------------|--------------|
| Space | O(n) with compression, O(n²) naive | O(n) |
| Construction | O(n) with Ukkonen | O(n) with SA-IS |
| Pattern search | O(m) | O(m log n) or O(m) with LCP |
| Cache efficiency | Poor (pointer chasing) | Good (contiguous) |
| Implementation | Complex | Simpler |

## 8. References

1. Weiner, P. (1973). "Linear Pattern Matching Algorithms". *FOCS*.
2. McCreight, E. M. (1976). "A Space-Economical Suffix Tree Construction Algorithm". *JACM*.
3. Ukkonen, E. (1995). "On-line Construction of Suffix Trees". *Algorithmica*.
4. Gusfield, D. "Algorithms on Strings, Trees, and Sequences", Chapters 5-9.

## Implementation

See: [src/string/suffix_tree.rs](../../src/string/suffix_tree.rs)
