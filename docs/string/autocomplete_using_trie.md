# Autocomplete Using Trie

## 1. Overview

**Autocomplete** (or predictive text) suggests completions for a partially typed word based on a dictionary. Using a **Trie** (prefix tree) data structure enables efficient prefix-based lookup, making it ideal for autocomplete systems. This algorithm achieves O(m + k) lookup time where m is the prefix length and k is the number of suggestions.

## 2. Mathematical Foundation

### 2.1 Trie Definition

A **Trie** is a tree where:
- Each edge represents a character
- Each node represents a prefix
- Terminal nodes mark complete words
- The path from root to any node spells out the prefix

### 2.2 Properties

1. **Prefix Sharing:** Common prefixes are stored once
2. **Space:** $O(\sum |w_i|)$ for dictionary of words $w_i$
3. **Lookup:** $O(m)$ to find prefix of length $m$
4. **Insert:** $O(m)$ for word of length $m$
5. **Completions:** $O(k)$ to collect $k$ completions

### 2.3 Trie Structure Example

For words: ["car", "card", "care", "cat", "dog"]

```
        (root)
       /      \
      c        d
      |        |
      a        o
     / \       |
    r   t      g*
   / \
  *   d*
  |
  e*
```

## 3. Algorithm Description

### 3.1 Trie Node Structure

```
struct TrieNode:
    children: map of char → TrieNode
    is_end_of_word: boolean
    word: string (optional, for convenience)
```

### 3.2 Insert Word

```
function INSERT(root, word):
    current = root
    for c in word:
        if c not in current.children:
            current.children[c] = new TrieNode()
        current = current.children[c]
    current.is_end_of_word = true
    current.word = word
```

### 3.3 Autocomplete

```
function AUTOCOMPLETE(root, prefix, limit):
    // Navigate to prefix node
    current = root
    for c in prefix:
        if c not in current.children:
            return []  // No completions
        current = current.children[c]
    
    // Collect all words from this node
    results = []
    DFS_COLLECT(current, results, limit)
    return results

function DFS_COLLECT(node, results, limit):
    if len(results) >= limit:
        return
    
    if node.is_end_of_word:
        results.append(node.word)
    
    for c in sorted(node.children.keys()):
        DFS_COLLECT(node.children[c], results, limit)
```

### 3.4 Step-by-Step Example

**Dictionary:** ["apple", "app", "application", "apply", "apt", "banana"]
**Prefix:** "app"

**Step 1:** Navigate to prefix "app"
```
root → 'a' → 'p' → 'p' (current node)
```

**Step 2:** DFS from current node
```
'p' (is_end → "app") ✓
 ├─ 'l' 
 │   ├─ 'e' (is_end → "apple") ✓
 │   ├─ 'i' → 'c' → 'a' → 't' → 'i' → 'o' → 'n' (is_end → "application") ✓
 │   └─ 'y' (is_end → "apply") ✓
```

**Result:** ["app", "apple", "application", "apply"]

## 4. Complexity Analysis

| Operation | Time | Space |
|-----------|------|-------|
| Insert | $O(m)$ | $O(m)$ |
| Search prefix | $O(m)$ | $O(1)$ |
| Autocomplete | $O(m + k)$ | $O(k)$ |
| Build trie | $O(\sum |w_i|)$ | $O(\sum |w_i|)$ |

Where $m$ is prefix length and $k$ is number of results.

## 5. Implementation Notes

### 5.1 Rust Implementation - Trie Structure

```rust
use std::collections::HashMap;

#[derive(Default, Debug)]
pub struct TrieNode {
    children: HashMap<char, TrieNode>,
    is_end: bool,
}

#[derive(Default, Debug)]
pub struct Trie {
    root: TrieNode,
}

impl Trie {
    pub fn new() -> Self {
        Trie {
            root: TrieNode::default(),
        }
    }
    
    pub fn insert(&mut self, word: &str) {
        let mut current = &mut self.root;
        
        for c in word.chars() {
            current = current.children.entry(c).or_default();
        }
        
        current.is_end = true;
    }
    
    pub fn search(&self, word: &str) -> bool {
        self.find_node(word)
            .map(|node| node.is_end)
            .unwrap_or(false)
    }
    
    pub fn starts_with(&self, prefix: &str) -> bool {
        self.find_node(prefix).is_some()
    }
    
    fn find_node(&self, prefix: &str) -> Option<&TrieNode> {
        let mut current = &self.root;
        
        for c in prefix.chars() {
            current = current.children.get(&c)?;
        }
        
        Some(current)
    }
}
```

### 5.2 Autocomplete Implementation

```rust
impl Trie {
    pub fn autocomplete(&self, prefix: &str, limit: usize) -> Vec<String> {
        let mut results = Vec::new();
        
        if let Some(node) = self.find_node(prefix) {
            self.collect_words(node, prefix.to_string(), &mut results, limit);
        }
        
        results
    }
    
    fn collect_words(
        &self,
        node: &TrieNode,
        current: String,
        results: &mut Vec<String>,
        limit: usize,
    ) {
        if results.len() >= limit {
            return;
        }
        
        if node.is_end {
            results.push(current.clone());
        }
        
        // Sort keys for deterministic order
        let mut keys: Vec<_> = node.children.keys().collect();
        keys.sort();
        
        for &c in keys {
            let child = &node.children[&c];
            let mut next = current.clone();
            next.push(c);
            self.collect_words(child, next, results, limit);
        }
    }
}
```

### 5.3 Frequency-Based Autocomplete

```rust
#[derive(Default, Debug)]
pub struct TrieNodeWithFreq {
    children: HashMap<char, TrieNodeWithFreq>,
    is_end: bool,
    frequency: u32,
}

impl TrieNodeWithFreq {
    pub fn autocomplete_by_frequency(
        &self,
        prefix: &str,
        limit: usize,
    ) -> Vec<(String, u32)> {
        // Collect all completions with frequencies
        let mut all_results: Vec<(String, u32)> = Vec::new();
        
        if let Some(node) = self.find_node(prefix) {
            self.collect_with_freq(node, prefix.to_string(), &mut all_results);
        }
        
        // Sort by frequency (descending) and take top k
        all_results.sort_by(|a, b| b.1.cmp(&a.1));
        all_results.truncate(limit);
        
        all_results
    }
}
```

### 5.4 Compact Trie (Radix Tree)

```rust
#[derive(Default, Debug)]
pub struct RadixNode {
    children: HashMap<String, RadixNode>,
    is_end: bool,
}

impl RadixNode {
    pub fn insert(&mut self, word: &str) {
        if word.is_empty() {
            self.is_end = true;
            return;
        }
        
        // Find longest common prefix with existing edge
        for (edge, child) in &mut self.children {
            let common_len = word
                .chars()
                .zip(edge.chars())
                .take_while(|(a, b)| a == b)
                .count();
            
            if common_len > 0 {
                if common_len == edge.len() {
                    // Edge is prefix of word
                    child.insert(&word[common_len..]);
                    return;
                } else {
                    // Need to split edge
                    // ... (implementation details)
                }
            }
        }
        
        // No common prefix, add new edge
        let mut new_node = RadixNode::default();
        new_node.is_end = true;
        self.children.insert(word.to_string(), new_node);
    }
}
```

### 5.5 Edge Cases

| Prefix | Dictionary | Result |
|--------|------------|--------|
| "" | ["a", "b"] | ["a", "b"] (all words) |
| "xyz" | ["a", "b"] | [] (no match) |
| "a" | ["a", "ab"] | ["a", "ab"] |
| "ab" | ["a", "ab"] | ["ab"] |

## 6. Real-World Applications

### 6.1 Use Cases

1. **Search Engines:**
   - Google search suggestions
   - Query completion

2. **IDEs:**
   - Code completion
   - Variable/function suggestions

3. **Mobile Keyboards:**
   - Predictive text
   - SwiftKey, Gboard

4. **Command Line:**
   - Tab completion
   - Shell autocomplete

### 6.2 Production Considerations

```rust
struct ProductionAutocomplete {
    trie: Trie,
    cache: LruCache<String, Vec<String>>,
    bloom_filter: BloomFilter,  // Fast negative lookup
}

impl ProductionAutocomplete {
    fn suggest(&mut self, prefix: &str) -> Vec<String> {
        // Check cache first
        if let Some(cached) = self.cache.get(prefix) {
            return cached.clone();
        }
        
        // Fast negative check
        if !self.bloom_filter.might_contain(prefix) {
            return Vec::new();
        }
        
        // Compute and cache
        let results = self.trie.autocomplete(prefix, 10);
        self.cache.put(prefix.to_string(), results.clone());
        results
    }
}
```

### 6.3 Fuzzy Matching Extension

```rust
fn fuzzy_autocomplete(
    trie: &Trie,
    prefix: &str,
    max_edits: usize,
) -> Vec<String> {
    let mut results = Vec::new();
    fuzzy_search_helper(
        &trie.root,
        prefix.chars().collect::<Vec<_>>().as_slice(),
        String::new(),
        max_edits,
        &mut results,
    );
    results
}
```

## 7. Optimizations

### 7.1 Space Optimization

| Technique | Benefit | Trade-off |
|-----------|---------|-----------|
| Array children | Faster lookup | More memory for sparse |
| Compressed trie | Less memory | Complex insertion |
| DAG sharing | Less memory | More complex |

### 7.2 Time Optimization

| Technique | Benefit |
|-----------|---------|
| Caching hot prefixes | O(1) for common queries |
| Lazy loading | Faster startup |
| Bloom filter | Fast negative lookups |
| Top-K heap | Better for frequency-based |

## 8. Comparison with Alternatives

| Structure | Insert | Autocomplete | Memory |
|-----------|--------|--------------|--------|
| **Trie** | O(m) | O(m + k) | High |
| **Sorted Array** | O(n) | O(log n + k) | Low |
| **Hash Map** | O(1) | O(n) | Medium |
| **B-Tree** | O(log n) | O(log n + k) | Medium |

## 9. References

1. Fredkin, E. (1960). "Trie Memory". *Communications of the ACM*.
2. Morrison, D. R. (1968). "PATRICIA—Practical Algorithm To Retrieve Information Coded in Alphanumeric". *JACM*.
3. Knuth, D. E. "The Art of Computer Programming, Vol. 3: Sorting and Searching".

## Implementation

See: [src/string/autocomplete_using_trie.rs](../../src/string/autocomplete_using_trie.rs)
