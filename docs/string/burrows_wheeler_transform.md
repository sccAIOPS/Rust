# Burrows-Wheeler Transform (BWT)

## 1. Overview

The **Burrows-Wheeler Transform (BWT)** is a reversible transformation that rearranges a string into runs of similar characters. Invented by Michael Burrows and David Wheeler in 1994, it's a key component of the bzip2 compression algorithm. The transform doesn't compress data directly but makes it more amenable to compression techniques like run-length encoding and move-to-front encoding.

## 2. Mathematical Foundation

### 2.1 Definition

Given a string $S$ of length $n$ with end marker `$`:
1. Form all $n$ cyclic rotations of $S$
2. Sort rotations lexicographically
3. BWT output is the last column of the sorted matrix
4. Also record the row number of the original string

### 2.2 Properties

1. **Reversible:** Original string can be perfectly reconstructed
2. **Same Length:** $|\text{BWT}(S)| = |S|$
3. **Character Preservation:** Same multiset of characters
4. **Clustering:** Similar characters tend to group together
5. **First Column:** Sorted version of input (not stored)

### 2.3 Key Insight

The transform clusters repeated patterns because:
- Rotations that share a common suffix end up adjacent when sorted
- Adjacent rotations have similar characters in the last column

## 3. Algorithm Description

### 3.1 Forward Transform (Encoding)

```
function BWT_ENCODE(s):
    s = s + "$"  // Add end-of-string marker
    n = len(s)
    
    // Generate all rotations
    rotations = []
    for i = 0 to n - 1:
        rotations.append(s[i:] + s[:i])
    
    // Sort rotations
    sort(rotations)
    
    // Extract last column
    bwt = ""
    original_index = -1
    for i = 0 to n - 1:
        bwt += rotations[i][n-1]
        if rotations[i] = s:
            original_index = i
    
    return (bwt, original_index)
```

### 3.2 Inverse Transform (Decoding)

```
function BWT_DECODE(bwt, original_index):
    n = len(bwt)
    
    // Build first column (sorted bwt)
    first_column = sort(bwt)
    
    // Build transformation vector
    // T[i] = position in first_column that follows position i in last_column
    count = {}
    positions = []
    for i = 0 to n - 1:
        c = bwt[i]
        positions.append(count.get(c, 0))
        count[c] = count.get(c, 0) + 1
    
    // Count occurrences before each char in first column
    start = {}
    total = 0
    for c in sorted(unique(bwt)):
        start[c] = total
        total += count[c]
    
    // Build T vector
    T = []
    for i = 0 to n - 1:
        c = bwt[i]
        T.append(start[c] + positions[i])
    
    // Reconstruct string
    result = ""
    idx = original_index
    for _ = 0 to n - 2:  // n-1 iterations (exclude $)
        idx = T[idx]
        result += bwt[idx]
    
    return result
```

### 3.3 Step-by-Step Example

**String:** "banana$"

**Step 1: Generate Rotations**
```
banana$
anana$b
nana$ba
ana$ban
na$bana
a$banan
$banana
```

**Step 2: Sort Rotations**
```
$banana    (0)
a$banan    (1)
ana$ban    (2)
anana$b    (3)
banana$    (4) ← original
na$bana    (5)
nana$ba    (6)
```

**Step 3: Extract Last Column**
BWT = "annb$aa"
Original Index = 4

**Decoding Process:**
- First column (sorted): "$aaabnn"
- Last column (BWT): "annb$aa"
- Build T vector using LF-mapping

## 4. Complexity Analysis

### 4.1 Naive Implementation

| Operation | Time | Space |
|-----------|------|-------|
| Encode | $O(n^2 \log n)$ | $O(n^2)$ |
| Decode | $O(n)$ | $O(n)$ |

### 4.2 Optimized (Suffix Array)

| Operation | Time | Space |
|-----------|------|-------|
| Encode | $O(n)$ | $O(n)$ |
| Decode | $O(n)$ | $O(n)$ |

## 5. Implementation Notes

### 5.1 Rust Implementation - Basic

```rust
pub fn burrows_wheeler_transform(input: &str) -> (String, usize) {
    if input.is_empty() {
        return (String::new(), 0);
    }
    
    let s = format!("{}$", input);
    let n = s.len();
    
    // Generate all rotations
    let mut rotations: Vec<String> = (0..n)
        .map(|i| format!("{}{}", &s[i..], &s[..i]))
        .collect();
    
    // Sort rotations
    rotations.sort();
    
    // Find original index and build BWT
    let mut bwt = String::with_capacity(n);
    let mut original_index = 0;
    
    for (i, rotation) in rotations.iter().enumerate() {
        bwt.push(rotation.chars().last().unwrap());
        if rotation == &s {
            original_index = i;
        }
    }
    
    (bwt, original_index)
}
```

### 5.2 Rust Implementation - Inverse

```rust
pub fn inverse_burrows_wheeler(bwt: &str, original_index: usize) -> String {
    let n = bwt.len();
    if n == 0 {
        return String::new();
    }
    
    let bwt_chars: Vec<char> = bwt.chars().collect();
    
    // Build first column (sorted)
    let mut first_column: Vec<char> = bwt_chars.clone();
    first_column.sort();
    
    // Count occurrences and positions
    let mut count: std::collections::HashMap<char, usize> = std::collections::HashMap::new();
    let mut positions: Vec<usize> = Vec::with_capacity(n);
    
    for &c in &bwt_chars {
        let pos = *count.get(&c).unwrap_or(&0);
        positions.push(pos);
        count.insert(c, pos + 1);
    }
    
    // Build start positions for each character in first column
    let mut start: std::collections::HashMap<char, usize> = std::collections::HashMap::new();
    let mut total = 0;
    let mut sorted_chars: Vec<char> = count.keys().copied().collect();
    sorted_chars.sort();
    
    for c in sorted_chars {
        start.insert(c, total);
        total += count[&c];
    }
    
    // Build T vector (LF mapping)
    let t: Vec<usize> = (0..n)
        .map(|i| start[&bwt_chars[i]] + positions[i])
        .collect();
    
    // Reconstruct original string
    let mut result = String::with_capacity(n - 1);
    let mut idx = original_index;
    
    for _ in 0..n - 1 {  // Exclude the '$' marker
        idx = t[idx];
        let c = bwt_chars[idx];
        if c != '$' {
            result.push(c);
        }
    }
    
    result
}
```

### 5.3 Suffix Array-Based (Optimized)

```rust
pub fn bwt_suffix_array(input: &str) -> (String, usize) {
    let s = format!("{}$", input);
    let chars: Vec<char> = s.chars().collect();
    let n = chars.len();
    
    // Build suffix array
    let mut suffix_array: Vec<usize> = (0..n).collect();
    suffix_array.sort_by(|&a, &b| {
        // Compare rotations starting at a and b
        for i in 0..n {
            let ca = chars[(a + i) % n];
            let cb = chars[(b + i) % n];
            if ca != cb {
                return ca.cmp(&cb);
            }
        }
        std::cmp::Ordering::Equal
    });
    
    // Build BWT from suffix array
    let bwt: String = suffix_array
        .iter()
        .map(|&i| chars[(i + n - 1) % n])
        .collect();
    
    let original_index = suffix_array.iter().position(|&x| x == 0).unwrap();
    
    (bwt, original_index)
}
```

### 5.4 Edge Cases

| Input | BWT | Index |
|-------|-----|-------|
| "" | "" | 0 |
| "a" | "a$" | 0 |
| "ab" | "b$a" | 1 |
| "aa" | "a$a" | 0 |

## 6. Real-World Applications

### 6.1 Use Cases

1. **Data Compression:**
   - bzip2 compression algorithm
   - Used with MTF and entropy coding

2. **Bioinformatics:**
   - FM-index for genome sequencing
   - DNA sequence alignment
   - BWA (Burrows-Wheeler Aligner)

3. **Full-Text Search:**
   - Compressed full-text indexes
   - Efficient substring queries

4. **Pattern Matching:**
   - Backward search algorithms
   - Count occurrences in O(m) time

### 6.2 Compression Pipeline

```
Original → BWT → MTF → RLE → Huffman → Compressed
```

### 6.3 Example: bzip2 Pipeline

```rust
fn bzip2_like_compress(input: &str) -> Vec<u8> {
    // 1. Burrows-Wheeler Transform
    let (bwt, index) = burrows_wheeler_transform(input);
    
    // 2. Move-to-Front Transform
    let mtf = move_to_front_encode(&bwt);
    
    // 3. Run-Length Encoding
    let rle = run_length_encode(&mtf);
    
    // 4. Huffman Coding (would be applied here)
    // ...
    
    rle // Simplified return
}
```

## 7. FM-Index Application

The FM-index uses BWT for efficient substring search:

```rust
struct FMIndex {
    bwt: String,
    first_column: Vec<char>,
    occ: Vec<std::collections::HashMap<char, usize>>,
    c: std::collections::HashMap<char, usize>,
}

impl FMIndex {
    fn count(&self, pattern: &str) -> usize {
        // O(m) search for pattern occurrences
        let mut sp = 0;
        let mut ep = self.bwt.len();
        
        for c in pattern.chars().rev() {
            // LF-mapping based search
            // ...
        }
        
        ep - sp
    }
}
```

## 8. Related Algorithms

| Algorithm | Purpose |
|-----------|---------|
| **Suffix Array** | Build BWT efficiently |
| **Move-to-Front** | Follow-up transform |
| **Run-Length Encoding** | Compress BWT output |
| **FM-Index** | Searchable compressed index |

## 9. References

1. Burrows, M., & Wheeler, D. J. (1994). "A Block-sorting Lossless Data Compression Algorithm". *Digital Systems Research Center Research Report 124*.
2. Ferragina, P., & Manzini, G. (2000). "Opportunistic Data Structures with Applications". *FOCS 2000*.
3. Li, H., & Durbin, R. (2009). "Fast and Accurate Short Read Alignment with Burrows-Wheeler Transform". *Bioinformatics*.

## Implementation

See: [src/string/burrows_wheeler_transform.rs](../../src/string/burrows_wheeler_transform.rs)
