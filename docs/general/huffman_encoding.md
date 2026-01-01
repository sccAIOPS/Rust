# Huffman Encoding

## 1. Overview

Huffman encoding is a lossless data compression algorithm developed by David A. Huffman in 1952 while he was a Ph.D. student at MIT. It creates an optimal prefix-free binary code by assigning variable-length codes to characters based on their frequencies—more frequent characters receive shorter codes, while less frequent ones get longer codes.

The algorithm is widely used in file compression formats (GZIP, JPEG, MP3) and forms the basis for many modern compression techniques.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a set of characters $C = \{c_1, c_2, ..., c_n\}$ with corresponding frequencies $F = \{f_1, f_2, ..., f_n\}$, find a prefix-free binary encoding that minimizes the total encoded length:

$$L = \sum_{i=1}^{n} f_i \cdot len(code_i)$$

where $len(code_i)$ is the length of the code assigned to character $c_i$.

### 2.2 Mathematical Model

**Input**: 
- Alphabet $C = \{c_1, c_2, ..., c_n\}$
- Frequency distribution $F = \{f_1, f_2, ..., f_n\}$ where $f_i > 0$

**Output**: 
- Prefix-free code $\phi: C \rightarrow \{0,1\}^*$

**Prefix-Free Property**: For any two distinct characters $c_i, c_j$, neither $\phi(c_i)$ is a prefix of $\phi(c_j)$ nor vice versa.

**Optimality Condition**: The encoding minimizes the expected code length $E[L] = \sum_{i=1}^{n} p_i \cdot |\phi(c_i)|$ where $p_i = f_i / \sum_{j=1}^{n} f_j$.

### 2.3 Correctness Proof

**Theorem**: Huffman's algorithm produces an optimal prefix-free code.

**Proof Sketch**:
1. **Greedy Choice**: Combining the two least frequent characters is always part of an optimal solution
2. **Optimal Substructure**: Reducing the problem by merging two nodes preserves optimality
3. **Induction**: The algorithm maintains optimality at each step by the greedy choice property

The proof relies on showing that any deviation from Huffman's greedy choices cannot improve the total code length.

## 3. Algorithm Description

### 3.1 Intuition

Huffman encoding works by building a binary tree bottom-up:

1. Start with all characters as leaf nodes, each with its frequency
2. Repeatedly merge the two nodes with smallest frequencies into a new parent node
3. The parent's frequency is the sum of its children's frequencies
4. Continue until only one root node remains
5. Assign binary codes: left edges = 0, right edges = 1
6. The path from root to each leaf gives that character's code

Frequent characters end up closer to the root (shorter codes), while rare characters are deeper (longer codes).

### 3.2 Pseudocode

```
function HuffmanEncode(characters, frequencies):
    // Build priority queue of nodes
    queue = MinHeap()
    for i = 1 to n:
        node = new Node(characters[i], frequencies[i])
        queue.insert(node)
    
    // Build Huffman tree
    while queue.size() > 1:
        left = queue.extractMin()
        right = queue.extractMin()
        
        parent = new Node(null, left.freq + right.freq)
        parent.left = left
        parent.right = right
        
        queue.insert(parent)
    
    root = queue.extractMin()
    
    // Generate codes by tree traversal
    codes = {}
    function generateCodes(node, code):
        if node.isLeaf():
            codes[node.char] = code
        else:
            generateCodes(node.left, code + "0")
            generateCodes(node.right, code + "1")
    
    generateCodes(root, "")
    return codes

function HuffmanDecode(encodedData, root):
    result = []
    current = root
    
    for bit in encodedData:
        if bit == 0:
            current = current.left
        else:
            current = current.right
        
        if current.isLeaf():
            result.append(current.char)
            current = root
    
    return result
```

### 3.3 Step-by-Step Example

Let's encode the string "BCAADDDCCACACAC" (15 characters).

**Step 1**: Count frequencies
- A: 5, B: 1, C: 6, D: 3

**Step 2**: Build Huffman tree

```
Initial queue: [B:1, D:3, A:5, C:6]

Iteration 1: Merge B:1 and D:3
    [*:4, A:5, C:6]  (* represents internal node)
         *:4
        / \
      B:1 D:3

Iteration 2: Merge *:4 and A:5
    [*:9, C:6]
         *:9
        / \
      *:4 A:5
     / \
   B:1 D:3

Iteration 3: Merge *:9 and C:6
    [*:15]  (root)
          *:15
         /   \
       C:6   *:9
            / \
          *:4 A:5
         / \
       B:1 D:3
```

**Step 3**: Assign codes (left=0, right=1)
- C: 0
- B: 100
- D: 101
- A: 11

**Step 4**: Encode "BCAADDDCCACACAC"
```
B    C    A    A    D    D    D    C    C    A    C    A    C    A    C
100  0    11   11   101  101  101  0    0    11   0    11   0    11   0

Encoded: 100011111011011010011011011
Length: 27 bits
```

**Original**: 15 characters × 8 bits = 120 bits  
**Compressed**: 27 bits (77.5% reduction)

## 4. Complexity Analysis

### 4.1 Time Complexity

**Building the Tree**:
- Creating initial heap: $O(n)$ where $n$ is the number of unique characters
- Each iteration: 2 extract-min operations + 1 insert operation = $O(\log n)$
- Total iterations: $n - 1$
- **Total**: $O(n \log n)$

**Generating Codes**:
- Tree traversal: $O(n)$ (visit each node once)

**Encoding**:
- For message of length $m$: $O(m)$ (lookup each character's code)

**Decoding**:
- For encoded bit stream of length $b$: $O(b)$ (traverse tree for each bit)

**Overall**: $O(n \log n + m)$ where $n$ = unique characters, $m$ = message length

### 4.2 Space Complexity

- **Huffman tree**: $O(n)$ - stores $2n - 1$ nodes (n leaves + n-1 internal nodes)
- **Code table**: $O(n)$ - one entry per character
- **Encoded data**: $O(m \cdot \bar{L})$ where $\bar{L}$ is average code length
- **Total**: $O(n + m \cdot \bar{L})$

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
// Node representation with Rc for shared ownership
use std::rc::Rc;
use std::cell::RefCell;
use std::collections::BinaryHeap;

#[derive(Debug)]
struct Node {
    frequency: usize,
    character: Option<char>,
    left: Option<Rc<RefCell<Node>>>,
    right: Option<Rc<RefCell<Node>>>,
}

// BinaryHeap is max-heap, so implement Ord for min-heap behavior
impl Ord for Node {
    fn cmp(&self, other: &Self) -> Ordering {
        other.frequency.cmp(&self.frequency) // Reverse for min-heap
    }
}

// Ownership considerations:
// - Use Rc<RefCell<Node>> for tree nodes (shared ownership)
// - HashMap<char, String> for code table
// - Vec<bool> or BitVec for compact bit representation
```

**Key Rust Features**:
- `BinaryHeap` for priority queue (reverse ordering for min-heap)
- `Rc<RefCell<T>>` for tree nodes with shared ownership
- `HashMap` for character-to-code mapping
- Generic implementation over `char` or `u8` for flexibility

### 5.2 Edge Cases

1. **Empty input**: Return empty encoding
2. **Single character**: Assign code "0" (need at least one bit)
3. **Two characters**: Balanced tree, both get 1-bit codes
4. **Equal frequencies**: Tree structure may vary, but total length is optimal
5. **Very long codes**: Characters with extremely low frequency may get long codes (limit depth if needed)
6. **Character not in tree**: Error during encoding
7. **Corrupted bit stream**: Invalid path in tree during decoding

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. File Compression**
- GZIP (DEFLATE algorithm combines LZ77 + Huffman)
- ZIP archives
- PNG image format (for compressed data blocks)

**2. Network Protocols**
- HTTP/2 header compression (HPACK uses Huffman coding)
- JPEG image compression (Huffman codes for DCT coefficients)
- MP3 audio compression (part of entropy encoding stage)

**3. Data Transmission**
- Fax machines (Modified Huffman coding)
- Modem protocols
- Satellite communications

**4. Database Systems**
- Column compression in analytics databases
- Index compression
- Log file compression

### 6.2 Related Algorithms

**Variations**:
- **Canonical Huffman Coding**: Standardized code assignment for easier decoding
- **Adaptive Huffman Coding**: Updates tree as data is processed (one-pass)
- **Length-Limited Huffman Coding**: Constrains maximum code length

**Alternative Compression**:
- **Arithmetic Coding**: Achieves better compression but more complex
- **LZ77/LZ78**: Dictionary-based compression
- **Run-Length Encoding**: Simple, works well for repetitive data
- **Shannon-Fano Coding**: Similar greedy approach, but suboptimal

**Complementary Techniques**:
- **Burrows-Wheeler Transform**: Preprocessing for better compression
- **Move-to-Front**: Improves locality for Huffman coding
- **Delta Encoding**: Preprocesses data for better frequency distribution

**When to Use**:
- **Huffman**: Static data, known frequency distribution, fast encoding/decoding
- **Arithmetic**: Maximum compression, computational resources available
- **LZ77**: Repetitive data, unknown frequency distribution
- **Adaptive Huffman**: Streaming data, no preprocessing pass possible

## 7. References

### Academic Papers
1. Huffman, D.A. (1952). "A Method for the Construction of Minimum-Redundancy Codes". *Proceedings of the IRE*, 40(9), 1098-1101.
2. Gallager, R.G. (1978). "Variations on a Theme by Huffman". *IEEE Transactions on Information Theory*, 24(6), 668-674.
3. Moffat, A., & Turpin, A. (1997). "On the Implementation of Minimum Redundancy Prefix Codes". *IEEE Transactions on Communications*, 45(10), 1200-1207.

### Books
1. Cover, T.M., & Thomas, J.A. (2006). *Elements of Information Theory* (2nd ed.). Wiley. Chapter 5: Data Compression.
2. Sayood, K. (2017). *Introduction to Data Compression* (5th ed.). Morgan Kaufmann.
3. Cormen, T.H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Section 16.3: Huffman codes.

### Online Resources
1. [Huffman Coding - Brilliant.org](https://brilliant.org/wiki/huffman-encoding/)
2. [Data Compression Lecture Notes - Stanford CS166](http://web.stanford.edu/class/cs166/)
3. [Huffman Coding Visualization](https://www.cs.usfca.edu/~galles/visualization/Huffman.html)

### Implementation
- Source: `src/general/huffman_encoding.rs`
- Tests: Included in source file under `#[cfg(test)] mod tests`
