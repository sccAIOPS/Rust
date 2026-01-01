# Fisher-Yates Shuffle

## 1. Overview

The Fisher-Yates shuffle (also known as the Knuth shuffle) is an algorithm for generating a random permutation of a finite sequence. Developed by Ronald Fisher and Frank Yates in 1938 and later popularized by Donald Knuth, it produces an unbiased permutation where every possible ordering has equal probability.

The modern algorithm runs in linear time and requires only $O(1)$ extra space, making it the standard method for shuffling in programming.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a sequence $A = [a_1, a_2, ..., a_n]$ of $n$ elements, generate a random permutation where each of the $n!$ possible permutations has equal probability $\frac{1}{n!}$.

### 2.2 Mathematical Model

**Input**: 
- Array $A$ of length $n$
- Random number generator producing uniformly distributed integers

**Output**: 
- Permutation of $A$ where each permutation occurs with probability $\frac{1}{n!}$

**Uniformity Property**: For any fixed permutation $\pi$ of $A$:
$$P(\text{output} = \pi) = \frac{1}{n!}$$

**In-Place Property**: The algorithm modifies the input array without requiring additional memory proportional to input size.

### 2.3 Correctness Proof

**Theorem**: The Fisher-Yates shuffle produces a uniformly random permutation.

**Proof by Induction**:

**Base case** ($n = 1$): Trivially uniform—only one permutation.

**Inductive step**: Assume the algorithm produces uniform permutations for arrays of size $n-1$.

For an array of size $n$:
1. The algorithm chooses position $i$ uniformly from $[0, n-1]$ with probability $\frac{1}{n}$
2. Element at position $n-1$ is swapped with element at position $i$
3. Any specific element becomes the last element with probability $\frac{1}{n}$
4. The remaining $n-1$ elements are shuffled uniformly (by induction hypothesis)
5. For any permutation $\pi$: 
   $$P(\pi) = \frac{1}{n} \cdot \frac{1}{(n-1)!} = \frac{1}{n!}$$

Therefore, all $n!$ permutations are equally likely. ∎

## 3. Algorithm Description

### 3.1 Intuition

The Fisher-Yates shuffle works by iterating through the array and randomly swapping each element with any element that comes before it (including itself):

1. Start from the last element
2. Pick a random element from positions 0 to current position
3. Swap the current element with the randomly chosen element
4. Move to the previous element and repeat
5. Continue until you reach the first element

This ensures that:
- Each element has an equal chance of ending up in any position
- The algorithm makes exactly $n-1$ swaps (or $n$ if you include the first element)
- No element is moved more than once in each iteration

**Why it works**: At each step, we're deciding which element should go in the current position with equal probability, then never touching that position again.

### 3.2 Pseudocode

```
function FisherYatesShuffle(array):
    n = length(array)
    
    // Iterate from last element to second element
    for i from n-1 down to 1:
        // Pick random index from 0 to i (inclusive)
        j = random_integer(0, i)
        
        // Swap array[i] with array[j]
        swap(array[i], array[j])
    
    return array

// Alternative: Forward iteration
function FisherYatesShuffleForward(array):
    n = length(array)
    
    for i from 0 to n-2:
        // Pick random index from i to n-1 (inclusive)
        j = random_integer(i, n-1)
        
        // Swap array[i] with array[j]
        swap(array[i], array[j])
    
    return array
```

Both versions are equivalent and produce uniformly random permutations.

### 3.3 Step-by-Step Example

Shuffle array: `[1, 2, 3, 4, 5]`

**Backward iteration**:

```
Initial: [1, 2, 3, 4, 5]

Step 1: i=4 (index of '5')
  j = random(0, 4) → suppose j=1
  Swap array[4] with array[1]
  Result: [1, 5, 3, 4, 2]

Step 2: i=3 (index of '4')
  j = random(0, 3) → suppose j=3
  Swap array[3] with array[3] (no change)
  Result: [1, 5, 3, 4, 2]

Step 3: i=2 (index of '3')
  j = random(0, 2) → suppose j=0
  Swap array[2] with array[0]
  Result: [3, 5, 1, 4, 2]

Step 4: i=1 (index of '5')
  j = random(0, 1) → suppose j=1
  Swap array[1] with array[1] (no change)
  Result: [3, 5, 1, 4, 2]

Final: [3, 5, 1, 4, 2]
```

**Probability analysis for first element**:
- After step 1: P(element is at position 4) = 1/5
- After step 2: P(element is at position 3) = 1/5
- After step 3: P(element is at position 2) = 1/5
- After step 4: P(element is at position 1) = 1/5
- Remaining probability for position 0: 1/5

Each element has exactly 1/5 probability of being in any position.

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Iterations**: $n - 1$ (or $n$ depending on implementation)
- **Per iteration**: 
  - Random number generation: $O(1)$
  - Swap operation: $O(1)$
- **Total**: $O(n)$

The algorithm makes exactly one pass through the array.

### 4.2 Space Complexity

- **In-place**: $O(1)$ auxiliary space
- Only uses a few variables for indices and temporary swap storage
- Does not allocate any additional arrays or data structures

**Note**: The random number generator may require internal state, but this is typically not counted as part of the algorithm's space complexity.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use rand::Rng;

// Generic implementation
fn fisher_yates_shuffle<T>(arr: &mut [T]) {
    let mut rng = rand::thread_rng();
    let n = arr.len();
    
    for i in (1..n).rev() {
        // Generate random index from 0 to i (inclusive)
        let j = rng.gen_range(0..=i);
        arr.swap(i, j);
    }
}

// Alternative with custom RNG
fn fisher_yates_shuffle_with_rng<T, R: Rng>(arr: &mut [T], rng: &mut R) {
    let n = arr.len();
    
    for i in (1..n).rev() {
        let j = rng.gen_range(0..=i);
        arr.swap(i, j);
    }
}

// Trait-based approach for testing
trait Shuffler {
    fn shuffle<T>(&mut self, arr: &mut [T]);
}

struct FisherYatesShuffler<R: Rng> {
    rng: R,
}

impl<R: Rng> Shuffler for FisherYatesShuffler<R> {
    fn shuffle<T>(&mut self, arr: &mut [T]) {
        fisher_yates_shuffle_with_rng(arr, &mut self.rng);
    }
}
```

**Key Rust Features**:
- Generic over element type `T`
- Uses `rand` crate for RNG
- `arr.swap(i, j)` is safe and idiomatic
- Mutable slice `&mut [T]` allows in-place modification
- Can accept custom RNG for deterministic testing

**Testing Considerations**:
```rust
#[cfg(test)]
mod tests {
    use super::*;
    use rand::SeedableRng;
    use rand::rngs::StdRng;
    
    #[test]
    fn test_deterministic_shuffle() {
        let mut arr = vec![1, 2, 3, 4, 5];
        let mut rng = StdRng::seed_from_u64(42);
        fisher_yates_shuffle_with_rng(&mut arr, &mut rng);
        
        // Result is deterministic with fixed seed
        assert_eq!(arr, vec![3, 5, 1, 4, 2]);
    }
    
    #[test]
    fn test_all_elements_present() {
        let mut arr = vec![1, 2, 3, 4, 5];
        fisher_yates_shuffle(&mut arr);
        arr.sort();
        assert_eq!(arr, vec![1, 2, 3, 4, 5]);
    }
}
```

### 5.2 Edge Cases

1. **Empty array**: No shuffle needed, return immediately
2. **Single element**: Already shuffled (no swaps needed)
3. **Two elements**: One swap determines the permutation
4. **Duplicate elements**: Algorithm works correctly (permutes positions, not values)
5. **Large arrays**: Ensure RNG can generate numbers up to $n-1$ without bias
6. **RNG quality**: Poor RNG can introduce bias; use cryptographically secure RNG if needed

**Common Mistake** (Incorrect Algorithm):
```rust
// WRONG: Introduces bias!
for i in 0..n {
    let j = rng.gen_range(0..n);  // Should be 0..=i or i..n
    arr.swap(i, j);
}
```
This produces biased results because elements can be moved multiple times in unpredictable ways.

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Gaming**
- Card shuffling (poker, solitaire, blackjack)
- Randomizing enemy spawns or loot drops
- Shuffling playlist or level order
- Randomizing quiz questions or survey items

**2. Machine Learning**
- Shuffling training data before each epoch
- Creating random mini-batches for SGD
- Cross-validation fold generation
- Data augmentation pipelines

**3. Cryptography**
- Generating random permutations for cryptographic protocols
- Shuffling data blocks for encryption
- Creating random padding sequences

**4. Simulations & Sampling**
- Monte Carlo simulations
- Random sampling without replacement
- A/B testing group assignment
- Randomized algorithms (e.g., quicksort with random pivot)

**5. User Experience**
- Shuffling advertisement displays
- Randomizing testimonial order
- Creating unpredictable UI element layouts
- Shuffle play in media players

**6. Testing**
- Generating random test cases
- Fuzzing input generation
- Property-based testing with random permutations
- Stress testing with varied operation orders

### 6.2 Related Algorithms

**Variations**:
- **Partial Shuffle**: Shuffle only first $k$ elements for sampling $k$ items
- **Reservoir Sampling**: Online algorithm for sampling from stream
- **Sattolo's Algorithm**: Generates random cyclic permutation (no fixed points)
- **Inside-Out Shuffle**: Generates new shuffled array without modifying input

**Alternative Approaches**:
- **Sort with Random Keys**: Assign random key to each element, then sort ($O(n \log n)$)
- **Recursive Shuffle**: Select random element, recurse on remainder (less efficient)

**Related Problems**:
- **Random Selection**: Select $k$ random elements from $n$
- **Weighted Shuffle**: Shuffle with non-uniform probabilities
- **Derangement Generation**: Permutations with no fixed points
- **Random Subset**: Select random subset of size $k$

**When to Use**:
- **Fisher-Yates**: Standard shuffling, best performance
- **Reservoir Sampling**: Streaming data, unknown size
- **Partial Shuffle**: Only need first $k$ random elements
- **Sort with Random Keys**: Stable shuffle required (rare)

## 7. References

### Academic Papers
1. Fisher, R.A., & Yates, F. (1938). *Statistical Tables for Biological, Agricultural and Medical Research*. Oliver & Boyd.
2. Knuth, D.E. (1969). *The Art of Computer Programming, Volume 2: Seminumerical Algorithms*. Addison-Wesley. Section 3.4.2: Random Sampling and Shuffling.
3. Durstenfeld, R. (1964). "Algorithm 235: Random Permutation". *Communications of the ACM*, 7(7), 420.

### Books
1. Knuth, D.E. (1997). *The Art of Computer Programming, Volume 2: Seminumerical Algorithms* (3rd ed.). Addison-Wesley.
2. Cormen, T.H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Section 5.3: Randomized algorithms.
3. McConnell, J.J. (2008). *Analysis of Algorithms: An Active Learning Approach* (2nd ed.). Jones & Bartlett.

### Online Resources
1. [Fisher-Yates Shuffle - Wikipedia](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle)
2. [Visualizing the Shuffle - Mike Bostock](https://bost.ocks.org/mike/shuffle/)
3. [How to Shuffle - Jeff Atwood](https://blog.codinghorror.com/the-danger-of-naivete/)
4. [Why Fisher-Yates is Unbiased - Stack Overflow](https://stackoverflow.com/questions/859253/why-is-fisher-yates-shuffle-unbiased)

### Implementation
- Source: `src/general/fisher_yates_shuffle.rs`
- Tests: Included in source file under `#[cfg(test)] mod tests`
