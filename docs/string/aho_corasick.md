# Aho-Corasick Algorithm

## 1. Overview

The Aho-Corasick algorithm, invented by Alfred V. Aho and Margaret J. Corasick in 1975, is a multi-pattern string matching algorithm. It constructs a finite state machine (automaton) from a set of patterns and then processes the text in a single pass to find all occurrences of all patterns simultaneously.

This algorithm is particularly powerful when searching for multiple patterns, as its time complexity depends on the text length plus total output, not on the number of patterns.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- A text string $T$ of length $n$
- A set of pattern strings $P = \{P_1, P_2, ..., P_k\}$ with total length $m$

Find: All occurrences of any pattern $P_i$ in $T$.

### 2.2 Mathematical Model

**Automaton Structure:**

The Aho-Corasick automaton consists of:
1. **Goto function** $g(s, c)$: State transitions on input character
2. **Failure function** $f(s)$: Fallback state on mismatch (suffix link)
3. **Output function** $out(s)$: Patterns ending at state $s$

**Formal Definitions:**

For state $s$ representing string $\sigma$:
- $f(s) =$ state representing longest proper suffix of $\sigma$ that is also a prefix of some pattern
- $out(s) =$ set of patterns that are suffixes of $\sigma$

### 2.3 Correctness

**Theorem:** The automaton visits every position in the text exactly once, and for each position, outputs all patterns ending at that position.

*Proof:* The failure function ensures that when we're at state $s$ (representing matched prefix $\sigma$), all patterns that are suffixes of $\sigma$ are reported via the output function.

## 3. Algorithm Description

### 3.1 Intuition

Think of building a combined trie from all patterns, then adding "shortcut" links:
1. **Trie construction:** Insert all patterns into a trie
2. **Failure links:** Add links from each node to the longest suffix that is also in the trie
3. **Output collection:** Propagate pattern matches along failure links
4. **Search:** Walk through text, following failure links on mismatch

### 3.2 Pseudocode

```
structure ACNode:
    transitions: Map<char, ACNode>
    suffix_link: ACNode  // Failure function
    lengths: List<int>   // Lengths of patterns ending here

function BUILD_AUTOMATON(patterns):
    root = new ACNode()
    
    // Phase 1: Build trie
    for pattern in patterns:
        node = root
        for char in pattern:
            if char not in node.transitions:
                node.transitions[char] = new ACNode()
            node = node.transitions[char]
        node.lengths.append(len(pattern))
    
    // Phase 2: Build failure links using BFS
    queue = []
    
    // Initialize depth-1 nodes
    for (char, child) in root.transitions:
        child.suffix_link = root
        queue.append(child)
    
    while queue not empty:
        current = queue.pop_front()
        
        for (char, child) in current.transitions:
            queue.append(child)
            
            // Find failure link for child
            suffix = current.suffix_link
            while suffix ≠ null and char not in suffix.transitions:
                suffix = suffix.suffix_link
            
            if suffix == null:
                child.suffix_link = root
            else:
                child.suffix_link = suffix.transitions[char]
            
            // Collect output from suffix link
            child.lengths.extend(child.suffix_link.lengths)
    
    return root

function SEARCH(root, text):
    matches = []
    current = root
    position = 0
    
    for char in text:
        // Follow failure links until we can transition on char
        while current ≠ root and char not in current.transitions:
            current = current.suffix_link
        
        if char in current.transitions:
            current = current.transitions[char]
        
        // Output all patterns ending at this position
        for length in current.lengths:
            matches.append(text[position - length + 1 : position + 1])
        
        position += 1
    
    return matches
```

### 3.3 Step-by-Step Example

**Patterns:** ["abc", "bc", "c", "xyz"]

**Phase 1: Build Trie**

```
        (root)
       /      \
      a        x
      |        |
      b        y
      |        |
      c*       z*
     /
    [abc,bc,c]

(* = pattern ends here)
```

**Phase 2: Add Failure Links**

```
State "abc" → suffix "bc" → suffix "c" → root
         ↓           ↓           ↓
    failure     failure     failure
```

**Search Text:** "xabcabc"

| Position | Char | State | Matches |
|----------|------|-------|---------|
| 0 | x | x | - |
| 1 | a | xa→a | - |
| 2 | b | ab | - |
| 3 | c | abc | "abc", "bc", "c" |
| 4 | a | a | - |
| 5 | b | ab | - |
| 6 | c | abc | "abc", "bc", "c" |

## 4. Complexity Analysis

### 4.1 Time Complexity

| Phase | Complexity |
|-------|------------|
| Trie construction | $O(m)$ |
| Failure link construction | $O(m)$ |
| Text search | $O(n + z)$ |
| **Total** | $O(m + n + z)$ |

Where:
- $m$ = total length of all patterns
- $n$ = length of text
- $z$ = number of pattern occurrences found

**Key insight:** Each character in the text is processed in amortized O(1) time because the total number of failure link traversals is bounded.

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Trie nodes | $O(m)$ |
| Transitions | $O(m \cdot |\Sigma|)$ worst case |
| Failure links | $O(m)$ |
| Output lists | $O(m)$ |
| **Total** | $O(m \cdot |\Sigma|)$ |

**Note:** Using hash maps for transitions gives $O(m)$ space in practice.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::cell::RefCell;
use std::collections::BTreeMap;
use std::rc::{Rc, Weak};

#[derive(Default)]
struct ACNode {
    trans: BTreeMap<char, Rc<RefCell<ACNode>>>,
    suffix: Weak<RefCell<ACNode>>,
    lengths: Vec<usize>,
}
```

**Implementation Choices:**
- `Rc<RefCell<>>`: Enables shared ownership with interior mutability
- `Weak<>`: Prevents reference cycles in suffix links
- `BTreeMap`: Ordered transitions (vs HashMap for O(1) access)
- `lengths`: Stores pattern lengths instead of patterns (memory efficient)

**Memory Management:**
```rust
// Using Weak references for suffix links prevents cycles
suffix: Weak<RefCell<ACNode>>,

// Upgrade to Rc when needed
if let Some(node) = suffix.upgrade() {
    // Use node
}
```

### 5.2 Edge Cases

| Case | Handling |
|------|----------|
| Empty pattern set | Return empty results |
| Empty text | Return empty results |
| Overlapping patterns | All matches reported |
| Pattern is prefix of another | Both matched |
| Unicode patterns | Handled via `char` iteration |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Intrusion Detection Systems:**
   - Network packet inspection
   - Malware signature matching
   - Snort IDS uses Aho-Corasick

2. **Content Filtering:**
   - Spam detection
   - Profanity filtering
   - URL blacklist matching

3. **Search Engines:**
   - Multi-keyword highlighting
   - Query term matching

4. **Bioinformatics:**
   - DNA motif searching
   - Multiple sequence alignment

5. **Compilers:**
   - Reserved keyword recognition
   - Multi-token lexical analysis

### 6.2 Performance Comparison

| Scenario | Naive | KMP per pattern | Aho-Corasick |
|----------|-------|-----------------|--------------|
| k patterns, n text, m total pattern length | $O(k \cdot n \cdot m)$ | $O(k \cdot n)$ | $O(n + m + z)$ |
| 1000 patterns, 1M text | ~1T ops | ~1B ops | ~1M ops |

### 6.3 Related Algorithms

| Algorithm | Use Case |
|-----------|----------|
| **KMP** | Single pattern matching |
| **Rabin-Karp** | Multiple patterns with hashing |
| **Commentz-Walter** | Boyer-Moore for multiple patterns |
| **Wu-Manber** | Hash-based multi-pattern (good for many short patterns) |

## 7. Variations and Extensions

### 7.1 Streaming Support

The automaton naturally supports streaming input:
```rust
struct StreamMatcher {
    automaton: AhoCorasick,
    current_state: Rc<RefCell<ACNode>>,
}

impl StreamMatcher {
    fn process_char(&mut self, c: char) -> Vec<String> {
        // Update state and return matches
    }
}
```

### 7.2 Case-Insensitive Matching

```rust
fn case_insensitive_search(text: &str) -> Vec<&str> {
    let text_lower = text.to_lowercase();
    // Build automaton with lowercase patterns
    ac.search(&text_lower)
}
```

### 7.3 Approximate Matching Extension

Combine with edit distance for fuzzy multi-pattern matching.

## 8. References

1. Aho, A. V., & Corasick, M. J. (1975). "Efficient String Matching: An Aid to Bibliographic Search". *Communications of the ACM*, 18(6), 333-340.
2. Cormen, T. H., et al. "Introduction to Algorithms", Chapter 32.
3. Navarro, G., & Raffinot, M. "Flexible Pattern Matching in Strings", Chapter 4.

## Implementation

See: [src/string/aho_corasick.rs](../../src/string/aho_corasick.rs)
