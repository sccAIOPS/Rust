# Suffix Array (Manber-Myers Algorithm)

## 1. Overview

The Manber-Myers algorithm is an efficient method for constructing suffix arrays in $O(n \log n)$ time. It improves upon the basic doubling technique by using more efficient sorting strategies during the iterative refinement of suffix ranks.

Named after Udi Manber and Gene Myers, this algorithm is widely used in practice due to its good balance between implementation complexity and performance.

## 2. Mathematical Foundation

### 2.1 Core Concept

The algorithm uses **prefix doubling**: at each iteration $k$, suffixes are sorted based on their first $2^k$ characters. After $\lceil \log n \rceil$ iterations, all suffixes are fully sorted.

### 2.2 Key Insight

If suffixes are sorted by their first $2^{k-1}$ characters, sorting by first $2^k$ characters requires comparing:
- First $2^{k-1}$ characters (already ranked)
- Next $2^{k-1}$ characters (rank of suffix starting at position $i + 2^{k-1}$)

This reduces to sorting pairs of integers!

### 2.3 Mathematical Formulation

Let $rank_k[i]$ = rank of suffix $S[i..]$ when sorted by first $2^k$ characters.

**Comparison key at step k:**
$$key_k[i] = (rank_{k-1}[i], rank_{k-1}[i + 2^{k-1}])$$

If $i + 2^{k-1} \geq n$, use sentinel value -1.

## 3. Algorithm Description

### 3.1 Pseudocode

```
function SUFFIX_ARRAY_MANBER_MYERS(input):
    if input is empty:
        return []
    
    n = len(input)
    
    // Step 1: Initialize suffixes with first character
    suffixes = [(i, input[i:]) for i in range(n)]
    
    // Sort initially by full suffix (for small strings) or first char
    suffixes.sort(by=lambda x: x[1])
    
    // Compute initial ranks
    suffix_array = [0] * n
    rank = [0] * n
    
    cur_rank = 0
    prev_suffix = suffixes[0][1]
    
    for i, (idx, suf) in enumerate(suffixes):
        if suf != prev_suffix:
            cur_rank += 1
            prev_suffix = suf
        rank[idx] = cur_rank
        suffix_array[i] = idx
    
    // Step 2: Iteratively double comparison length
    k = 1
    while k < n:
        // Sort by (rank[i], rank[i + k])
        suffix_array.sort(by=lambda x: (rank[x], rank[(x + k) % n]))
        
        // Update ranks
        new_rank = [0] * n
        cur_rank = 0
        prev = suffix_array[0]
        new_rank[prev] = cur_rank
        
        for i in range(1, n):
            curr = suffix_array[i]
            // Check if same key as previous
            if (rank[prev], rank[(prev + k) % n]) != (rank[curr], rank[(curr + k) % n]):
                cur_rank += 1
            new_rank[curr] = cur_rank
            prev = curr
        
        rank = new_rank
        k *= 2
    
    return suffix_array
```

### 3.2 Step-by-Step Example

**String:** "banana"

**Initial Sort by Full Suffix:**
| i | Suffix | Initial Rank |
|---|--------|--------------|
| 5 | a | 0 |
| 3 | ana | 1 |
| 1 | anana | 2 |
| 0 | banana | 3 |
| 4 | na | 4 |
| 2 | nana | 5 |

**k=1: Sort by (rank[i], rank[i+1])**

| i | (rank[i], rank[i+1]) |
|---|----------------------|
| 5 | (0, -1) |
| 3 | (1, 4) |
| 1 | (2, 5) |
| 0 | (3, 1) |
| 4 | (4, 0) |
| 2 | (5, 1) |

Already sorted by initial full suffix comparison.

**k=2, k=4, ...** Continue until k ≥ n.

**Final Suffix Array:** [5, 3, 1, 0, 4, 2]

## 4. Complexity Analysis

### 4.1 Time Complexity

| Component | Complexity |
|-----------|------------|
| Initial sorting | $O(n \log n)$ or $O(n^2 \log n)$ naive |
| Iterations | $O(\log n)$ |
| Each iteration sort | $O(n \log n)$ |
| **Total** | $O(n \log^2 n)$ with comparison sort |
| **With radix sort** | $O(n \log n)$ |

### 4.2 Space Complexity

| Component | Space |
|-----------|-------|
| Suffix array | $O(n)$ |
| Rank array | $O(n)$ |
| New rank array | $O(n)$ |
| **Total** | $O(n)$ |

## 5. Implementation Notes

### 5.1 Rust-Specific Implementation

```rust
pub fn generate_suffix_array_manber_myers(input: &str) -> Vec<usize> {
    if input.is_empty() {
        return Vec::new();
    }
    let n = input.len();
    let mut suffixes: Vec<(usize, &str)> = Vec::with_capacity(n);

    for (i, _suffix) in input.char_indices() {
        suffixes.push((i, &input[i..]));
    }

    suffixes.sort_by_key(|&(_, s)| s);
    // ... continue with rank updates
}
```

**Key Implementation Details:**
- Uses `char_indices()` for proper UTF-8 handling
- Initial sort uses full suffix comparison (simple but O(n² log n))
- Iterative refinement with pair comparison

### 5.2 Comparison with Basic Implementation

| Aspect | Basic SA | Manber-Myers |
|--------|----------|--------------|
| Initial sort | By first 2 chars | By full suffix or single char |
| Rank storage | Tuple (rank0, rank1) | Single rank + lookup |
| Memory swapping | Copy tuples | Swap arrays |
| Practical speed | Good | Slightly better |

### 5.3 Edge Cases

| Input | Suffix Array |
|-------|--------------|
| "" | [] |
| "a" | [0] |
| "zzzzzz" | [5, 4, 3, 2, 1, 0] |
| "abcdefghijklmnopqrstuvwxyz" | [0, 1, 2, ..., 25] |
| "abracadabra!" | [11, 10, 7, 0, 3, 5, 8, 1, 4, 6, 9, 2] |

## 6. Real-World Applications

### 6.1 Use Cases

1. **Text Indexing:**
   - Search engines
   - Document retrieval systems

2. **Bioinformatics:**
   - Genome assembly
   - Sequence alignment (BWA, Bowtie)

3. **Data Compression:**
   - BWT-based compression
   - Dictionary compression

### 6.2 When to Choose Manber-Myers

✅ **Good for:**
- Medium-sized strings (10K - 10M characters)
- When implementation simplicity matters
- When no linear-time algorithm library available

❌ **Consider alternatives:**
- Very large strings: Use SA-IS or DC3 (linear time)
- Real-time constraints: Pre-built libraries
- Memory-constrained: Streaming approaches

## 7. Optimizations

### 7.1 Radix Sort Optimization

Replace comparison sort with radix sort for $O(n \log n)$ total:

```rust
fn radix_sort_by_rank(sa: &mut [usize], rank: &[usize], k: usize) {
    let n = sa.len();
    let max_rank = n;
    
    // Count sort by second key (rank[i + k])
    let mut count = vec![0; max_rank + 2];
    for &i in sa.iter() {
        let key = if i + k < n { rank[i + k] + 1 } else { 0 };
        count[key] += 1;
    }
    // ... counting sort implementation
    
    // Count sort by first key (rank[i])
    // ... similar
}
```

### 7.2 Memory Optimization

Reuse arrays instead of allocating new ones:

```rust
fn update_ranks_in_place(rank: &mut [usize], sa: &[usize], k: usize) {
    // Use suffix_array order to assign new ranks
    let mut new_rank = 0;
    let mut prev_key = (rank[sa[0]], rank.get(sa[0] + k).copied().unwrap_or(0));
    
    for &i in sa.iter().skip(1) {
        let curr_key = (rank[i], rank.get(i + k).copied().unwrap_or(0));
        if curr_key != prev_key {
            new_rank += 1;
            prev_key = curr_key;
        }
        // Store temporarily and update later
    }
}
```

## 8. Comparison with Other Algorithms

| Algorithm | Time | Space | Implementation |
|-----------|------|-------|----------------|
| Naive | $O(n^2 \log n)$ | $O(n^2)$ | Trivial |
| Manber-Myers | $O(n \log n)$ | $O(n)$ | Moderate |
| DC3/Skew | $O(n)$ | $O(n)$ | Complex |
| SA-IS | $O(n)$ | $O(n)$ | Complex |
| libdivsufsort | $O(n)$ | $O(n)$ | Library |

## 9. References

1. Manber, U., & Myers, G. (1993). "Suffix Arrays: A New Method for On-Line String Searches". *SIAM Journal on Computing*, 22(5), 935-948.
2. Puglisi, S. J., Smyth, W. F., & Turpin, A. H. (2007). "A Taxonomy of Suffix Array Construction Algorithms". *ACM Computing Surveys*.
3. Kärkkäinen, J., Sanders, P., & Burkhardt, S. (2006). "Linear Work Suffix Array Construction". *Journal of the ACM*.

## Implementation

See: [src/string/suffix_array_manber_myers.rs](../../src/string/suffix_array_manber_myers.rs)
