# Count-Min Sketch

## 1. Overview

A Count-Min Sketch is a probabilistic data structure for frequency estimation in data streams. It uses sub-linear space to answer "how many times has element x appeared?" with bounded error. Unlike hash maps, it handles massive streams without storing individual elements.

Introduced by Graham Cormode and S. Muthukrishnan in 2005.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a stream of elements, support:
- **Update(x, c)**: Add count $c$ to element $x$
- **Query(x)**: Estimate frequency of $x$

**Guarantees**:
- **Never underestimates**: $\hat{f}(x) \geq f(x)$ (actual frequency)
- **Bounded overestimate**: $\hat{f}(x) \leq f(x) + \epsilon \cdot N$ with probability $1 - \delta$

Where $N$ is total count of all elements.

### 2.2 Mathematical Model

**Structure**:
- 2D array of counters: $d$ rows × $w$ columns
- $d$ independent hash functions $h_1, h_2, ..., h_d$, each mapping to $[0, w-1]$

**Update(x, c)**:
For each row $i$: `count[i][h_i(x)] += c`

**Query(x)**:
$$\hat{f}(x) = \min_{i \in [1,d]} \text{count}[i][h_i(x)]$$

### 2.3 Parameter Selection

For error $\epsilon$ and failure probability $\delta$:

$$w = \lceil e / \epsilon \rceil \approx 2.72 / \epsilon$$
$$d = \lceil \ln(1/\delta) \rceil$$

**Space**: $O(d \times w) = O(\frac{1}{\epsilon} \log \frac{1}{\delta})$ counters

### 2.4 Error Bound Proof (Sketch)

For a single row, collision probability ≤ 1/w by hash uniformity.
Expected overcount from collisions ≤ N/w = εN.
Taking min over d rows, probability all rows overcounted by > εN is ≤ (1/e)^d = δ.

## 3. Algorithm Description

### 3.1 Intuition

Each row is like a compressed frequency table. When you increment an element, its counter goes up, but so might other elements' counters (collisions). By using multiple rows with different hash functions and taking the minimum, collision noise is reduced.

### 3.2 Structure Visualization

```
Count-Min Sketch (w=5, d=3):

Element stream: A, B, A, C, A, B, D, A

After processing:
             0    1    2    3    4
Row 0 (h₀): [2]  [4]  [0]  [1]  [1]   ← A→1, B→0, C→3, D→4
Row 1 (h₁): [1]  [2]  [4]  [1]  [0]   ← A→2, B→1, C→1, D→0
Row 2 (h₂): [0]  [1]  [1]  [4]  [2]   ← A→3, B→4, C→2, D→1

Query(A):
  h₀(A)=1 → count[0][1] = 4
  h₁(A)=2 → count[1][2] = 4
  h₂(A)=3 → count[2][3] = 4
  Estimate: min(4, 4, 4) = 4  (actual = 4) ✓

Query(B):
  h₀(B)=0 → count[0][0] = 2
  h₁(B)=1 → count[1][1] = 2
  h₂(B)=4 → count[2][4] = 2
  Estimate: min(2, 2, 2) = 2  (actual = 2) ✓

Query(E) (never seen):
  h₀(E)=0 → count[0][0] = 2  (collision with B!)
  h₁(E)=0 → count[1][0] = 1
  h₂(E)=0 → count[2][0] = 0
  Estimate: min(2, 1, 0) = 0  (actual = 0) ✓
```

### 3.3 Pseudocode

```
CREATE(epsilon, delta):
    w = ceil(e / epsilon)
    d = ceil(ln(1 / delta))
    count = 2D array [d][w] initialized to 0
    hash_functions = generate d hash functions
    return CountMinSketch(count, hash_functions, w, d)

UPDATE(sketch, element, count=1):
    for i from 0 to d-1:
        j = hash_i(element) mod w
        sketch.count[i][j] += count

QUERY(sketch, element):
    result = INFINITY
    for i from 0 to d-1:
        j = hash_i(element) mod w
        result = min(result, sketch.count[i][j])
    return result
```

### 3.4 Step-by-Step Example

**Track word frequencies with ε=0.1, δ=0.01**:

```
Parameters:
  w = ceil(e/0.1) = 28
  d = ceil(ln(100)) = 5

Stream: "the", "quick", "brown", "fox", "the", "lazy", "the"

Update "the" (3 times total):
  For each of 5 hash functions, increment count[i][h_i("the")]

Query "the":
  Return min of 5 counts = 3 (correct!)

Query "cat" (never seen):
  Some counters might be non-zero due to collisions
  Return min = 0 (likely, if no collisions in ALL rows)
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Time |
|-----------|------|
| Update | O(d) = O(log 1/δ) |
| Query | O(d) = O(log 1/δ) |

Typically d is small (5-10), so effectively O(1).

### 4.2 Space Complexity

- **Counters**: O(w × d) = O((1/ε) × log(1/δ))
- **Per counter**: Typically 4-8 bytes
- **Total**: Much smaller than exact counting for large domains

### 4.3 Example Space Savings

For 1 billion unique elements:
- Hash map: ~16 GB (key + count per element)
- Count-Min (ε=0.01, δ=0.01): ~1.4 KB (28×5×4 bytes)

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::collections::hash_map::DefaultHasher;
use std::hash::{Hash, Hasher};

pub struct CountMinSketch {
    counters: Vec<Vec<u64>>,
    width: usize,
    depth: usize,
}

impl CountMinSketch {
    pub fn new(epsilon: f64, delta: f64) -> Self {
        let width = (std::f64::consts::E / epsilon).ceil() as usize;
        let depth = (1.0 / delta).ln().ceil() as usize;
        
        CountMinSketch {
            counters: vec![vec![0; width]; depth],
            width,
            depth,
        }
    }
    
    fn hash_with_seed<T: Hash>(item: &T, seed: usize) -> u64 {
        let mut hasher = DefaultHasher::new();
        seed.hash(&mut hasher);
        item.hash(&mut hasher);
        hasher.finish()
    }
    
    pub fn increment<T: Hash>(&mut self, item: &T) {
        self.add(item, 1);
    }
    
    pub fn add<T: Hash>(&mut self, item: &T, count: u64) {
        for row in 0..self.depth {
            let col = (Self::hash_with_seed(item, row) as usize) % self.width;
            self.counters[row][col] = self.counters[row][col].saturating_add(count);
        }
    }
    
    pub fn estimate<T: Hash>(&self, item: &T) -> u64 {
        (0..self.depth)
            .map(|row| {
                let col = (Self::hash_with_seed(item, row) as usize) % self.width;
                self.counters[row][col]
            })
            .min()
            .unwrap_or(0)
    }
}
```

**Key Design Patterns**:
- Seeded hashing for multiple hash functions
- `saturating_add` to prevent overflow
- `Vec<Vec<u64>>` for 2D counter array
- Generic over `T: Hash`

### 5.2 Conservative Update

Improves accuracy by only incrementing up to current minimum:

```rust
pub fn conservative_add<T: Hash>(&mut self, item: &T, count: u64) {
    let current_min = self.estimate(item);
    let new_val = current_min + count;
    
    for row in 0..self.depth {
        let col = (Self::hash_with_seed(item, row) as usize) % self.width;
        if self.counters[row][col] < new_val {
            self.counters[row][col] = new_val;
        }
    }
}
```

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Zero count | Valid, don't increment |
| Negative count | Not supported (use Count-Mean-Min) |
| Counter overflow | Use saturating arithmetic |
| Query unseen element | May return > 0 (false positive) |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Network Traffic**: Track flow frequencies
2. **Database Query Optimization**: Frequency estimation for joins
3. **Click Streams**: Popular page tracking
4. **NLP**: Word frequency estimation
5. **Advertising**: Click fraud detection
6. **Anomaly Detection**: Find heavy hitters

### 6.2 Heavy Hitters Problem

Find elements appearing more than θ·N times:

```rust
fn find_heavy_hitters(
    sketch: &CountMinSketch,
    candidates: &[Item],
    threshold: f64,
    total_count: u64,
) -> Vec<Item> {
    let min_count = (threshold * total_count as f64) as u64;
    candidates
        .iter()
        .filter(|item| sketch.estimate(item) >= min_count)
        .cloned()
        .collect()
}
```

### 6.3 Related Structures

| Structure | Use Case |
|-----------|----------|
| **Count-Min Sketch** | Frequency estimation |
| **Bloom Filter** | Set membership |
| **HyperLogLog** | Cardinality estimation |
| **AMS Sketch** | Second moment estimation |
| **Misra-Gries** | Heavy hitters |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **Always Overestimates**: Never returns less than true count
2. **No Deletion**: Standard version doesn't support decrements
3. **Hash Quality**: Poor hashes increase collisions
4. **Parameter Tuning**: Must choose ε, δ based on use case

### 7.2 Count-Mean-Min Sketch

Reduces bias by subtracting estimated noise:

```rust
fn estimate_with_mean_adjustment<T: Hash>(&self, item: &T, total_count: u64) -> i64 {
    let estimates: Vec<u64> = (0..self.depth)
        .map(|row| {
            let col = (Self::hash_with_seed(item, row) as usize) % self.width;
            self.counters[row][col]
        })
        .collect();
    
    let noise_estimate = total_count / self.width as u64;
    estimates.iter()
        .map(|&e| e as i64 - noise_estimate as i64)
        .max()
        .unwrap_or(0)
        .max(0)
}
```

### 7.3 Merging Sketches

Count-Min Sketches are mergeable (great for distributed systems):

```rust
fn merge(a: &CountMinSketch, b: &CountMinSketch) -> CountMinSketch {
    assert_eq!(a.width, b.width);
    assert_eq!(a.depth, b.depth);
    
    let counters = a.counters.iter()
        .zip(&b.counters)
        .map(|(row_a, row_b)| {
            row_a.iter().zip(row_b).map(|(x, y)| x + y).collect()
        })
        .collect();
    
    CountMinSketch {
        counters,
        width: a.width,
        depth: a.depth,
    }
}
```

### 7.4 When to Use

| Scenario | Use Count-Min Sketch? |
|----------|----------------------|
| Memory constrained | ✓ Yes |
| Need exact counts | ✗ No |
| Stream processing | ✓ Yes |
| Small domain | ✗ Use hash map |
| Need decrements | ✗ Use Count-Min-Log |

## 8. References

- Cormode, G., & Muthukrishnan, S. (2005). "An Improved Data Stream Summary: The Count-Min Sketch and its Applications". *Journal of Algorithms*.
- Cormode, G. (2011). "Sketch Techniques for Approximate Query Processing". *Foundations and Trends in Databases*.
- Deng, F., & Rafiei, D. (2007). "New Estimation Algorithms for Streaming Data: Count-min Can Do More". *WebDB*.
- Goyal, A., Daumé III, H., & Cormode, G. (2012). "Sketch Algorithms for Estimating Point Queries in NLP". *EMNLP*.
