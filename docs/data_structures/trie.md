# Trie (Prefix Tree)

## 1. Overview

A Trie (from "retrieval", pronounced "try" or "tree") is a tree-like data structure used to store a dynamic set of strings or sequences. Each node represents a single character/element, and paths from root to nodes represent prefixes. Tries excel at prefix-based operations like autocomplete and spell checking.

First described by René de la Briandais in 1959, with the name "trie" coined by Edward Fredkin.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a set of strings (or sequences) $S$, support:
- **Insert**: Add string $s$ to $S$
- **Search**: Check if string $s \in S$
- **Prefix Search**: Find all strings with prefix $p$

### 2.2 Mathematical Model

**Trie Structure**:
- Each edge is labeled with a character
- Each node represents the prefix formed by the path from root
- Some nodes are marked as "end of word"

**Alphabet**: Let $\Sigma$ be the alphabet (e.g., lowercase letters, $|\Sigma| = 26$)

**Height**: Maximum string length $m$

**Node Structure**:
$$\text{Node} = (\text{children}: \Sigma \rightarrow \text{Node}, \text{is\_end}: \text{bool}, \text{value}: T)$$

## 3. Algorithm Description

### 3.1 Intuition

A trie shares common prefixes among strings. If you store "car", "card", and "care", they share the path "c-a-r", diverging only at the end. This prefix sharing provides both space efficiency and fast prefix queries.

### 3.2 Structure Visualization

```
Storing: ["car", "card", "care", "cat", "dog"]

        (root)
        /    \
       c      d
       |      |
       a      o
      / \     |
     r   t*   g*
    /|\
   d* e*
   
* = end of word
```

### 3.3 Pseudocode

```
INSERT(trie, key, value):
    node = trie.root
    for char in key:
        if char not in node.children:
            node.children[char] = new Node()
        node = node.children[char]
    node.value = value
    node.is_end = true

GET(trie, key):
    node = trie.root
    for char in key:
        if char not in node.children:
            return NOT_FOUND
        node = node.children[char]
    if node.is_end:
        return node.value
    return NOT_FOUND

PREFIX_SEARCH(trie, prefix):
    node = trie.root
    for char in prefix:
        if char not in node.children:
            return []
        node = node.children[char]
    return COLLECT_ALL_WORDS(node, prefix)

COLLECT_ALL_WORDS(node, prefix):
    results = []
    if node.is_end:
        results.append(prefix)
    for (char, child) in node.children:
        results.extend(COLLECT_ALL_WORDS(child, prefix + char))
    return results
```

### 3.4 Step-by-Step Example

Insert "to", "tea", "ten", "in", "inn":

```
Insert "to":     Insert "tea":    Insert "ten":
    (root)           (root)           (root)
      |                |                |
      t                t                t
      |               / \              / \
      o*             o*  e            o*  e
                         |               / \
                         a*             a*  n*

Insert "in":     Insert "inn":
    (root)           (root)
    /   \            /   \
   t     i          t     i
  / \    |         / \    |
 o*  e   n*       o*  e   n*
    / \              / \   |
   a*  n*           a*  n* n*
```

Search "tea": t → e → a → found (marked as end)
Search "te": t → e → not end of word → NOT_FOUND
Prefix "te": Returns ["tea", "ten"]

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Time Complexity |
|-----------|-----------------|
| Insert    | O(m)            |
| Search    | O(m)            |
| Delete    | O(m)            |
| Prefix Search | O(m + k)    |

Where:
- $m$ = length of the key/prefix
- $k$ = number of matches for prefix search

**Note**: Operations are independent of the number of strings stored!

### 4.2 Space Complexity

- **Worst Case**: O(n × m × |Σ|) for n strings of length m
- **Best Case (shared prefixes)**: Much less due to prefix sharing
- **Per Node**: O(|Σ|) for children pointers (or O(k) with hash map)

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::collections::HashMap;
use std::hash::Hash;

struct Node<Key: Default, Type: Default> {
    children: HashMap<Key, Node<Key, Type>>,
    value: Option<Type>,
}

pub struct Trie<Key, Type>
where
    Key: Default + Eq + Hash,
    Type: Default,
{
    root: Node<Key, Type>,
}
```

**Key Design Patterns**:
- `HashMap` for sparse children (memory efficient for large alphabets)
- Generic over `Key` type (works with chars, integers, etc.)
- `Option<Type>` for value storage at word endings
- Trait bounds: `Default + Eq + Hash` for keys

**Implementation**:
```rust
pub fn insert(&mut self, key: impl IntoIterator<Item = Key>, value: Type) {
    let mut node = &mut self.root;
    for c in key {
        node = node.children.entry(c).or_default();
    }
    node.value = Some(value);
}

pub fn get(&self, key: impl IntoIterator<Item = Key>) -> Option<&Type> {
    let mut node = &self.root;
    for c in key {
        node = node.children.get(&c)?;
    }
    node.value.as_ref()
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty string | Store value at root |
| Empty trie | get returns None |
| Prefix of existing | Returns None unless marked as end |
| Extending existing | Creates new path from divergence |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Autocomplete**: Search engines, IDEs, text editors
2. **Spell Checkers**: Dictionary lookup with suggestions
3. **IP Routing**: Longest prefix matching (CIDR)
4. **T9 Predictive Text**: Phone keyboards
5. **DNA Sequencing**: Suffix tries for genome analysis
6. **File Systems**: Directory path lookup

### 6.2 Variants

| Variant | Description |
|---------|-------------|
| **Compressed Trie** | Merge single-child chains |
| **Radix Tree** | Compressed trie with edge labels |
| **Patricia Trie** | Practical radix tree |
| **Suffix Trie** | All suffixes of a string |
| **Ternary Search Tree** | BST-like, less memory |
| **DAWG** | Directed Acyclic Word Graph |

### 6.3 Related Algorithms

| Structure | Use Case |
|-----------|----------|
| **Hash Table** | Exact match only, O(1) average |
| **BST** | Ordered keys, O(log n) operations |
| **Suffix Array** | Substring matching, less space |
| **Bloom Filter** | Membership test, probabilistic |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Memory Explosion**: Large alphabet × deep trie = many nodes
2. **Pointer Overhead**: HashMap per node adds overhead
3. **Cache Misses**: Pointer chasing is cache-unfriendly
4. **Unicode**: Characters may be multi-byte

### 7.2 Optimization Opportunities

**Array-Based Children** (for small alphabets):
```rust
struct Node {
    children: [Option<Box<Node>>; 26],  // a-z only
    is_end: bool,
}
```

**Compressed Trie**:
```
Before:  a → b → c → d*
After:   "abcd"*
```

**Double-Array Trie**: Cache-friendly, used in NLP libraries

**Succinct Tries**: Bit-level encoding for minimal space

### 7.3 Memory Comparison

| Implementation | Space per Node |
|----------------|----------------|
| HashMap children | ~48 bytes + HashMap overhead |
| Array [26] | 26 × 8 = 208 bytes |
| Compressed radix | Amortized much less |
| Double-array | ~4 bytes (after construction) |

## 8. References

- de la Briandais, R. (1959). "File Searching Using Variable Length Keys". *Proceedings of the Western Joint Computer Conference*.
- Fredkin, E. (1960). "Trie Memory". *Communications of the ACM*.
- Knuth, D. E. (1997). *The Art of Computer Programming, Volume 3: Sorting and Searching*. Section 6.3.
- Morrison, D. R. (1968). "PATRICIA—Practical Algorithm To Retrieve Information Coded in Alphanumeric". *JACM*.
