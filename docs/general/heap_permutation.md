# Heap's Permutation Algorithm

## 1. Overview

Heap's algorithm is an elegant method for generating all permutations of $n$ objects, discovered by B.R. Heap in 1963. The algorithm is notable for minimizing the number of element swaps and generating permutations in a systematic order, making it one of the most efficient permutation generation algorithms.

The algorithm generates each permutation from the previous one by swapping only two elements, requiring exactly one swap per permutation (except the first).

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a sequence of $n$ distinct elements, generate all $n!$ permutations.

**Goal**: Visit each of the $n!$ permutations exactly once, using minimal operations.

### 2.2 Mathematical Model

**Input**: Array $A = [a_1, a_2, ..., a_n]$ of $n$ distinct elements

**Output**: Set of all permutations $P = \{\pi_1, \pi_2, ..., \pi_{n!}\}$ where each $\pi_i$ is a permutation of $A$

**Minimality Property**: Heap's algorithm performs exactly $n! - 1$ swaps to generate all $n!$ permutations.

**Recurrence Relation**:
- For size $n$: Fix last element, generate all $(n-1)!$ permutations of first $n-1$ elements
- Swap the last element with each of the others in turn
- Repeat for each position of the last element

### 2.3 Correctness Proof

**Theorem**: Heap's algorithm generates all $n!$ permutations exactly once.

**Proof by Induction**:

**Base case** ($n = 1$): Single element has 1 permutation. ✓

**Inductive step**: Assume algorithm works for $n-1$ elements.

For $n$ elements:
1. Fix element at position $n-1$
2. Generate all $(n-1)!$ permutations of first $n-1$ elements (correct by induction)
3. Swap element at position $n-1$ with another element
4. Repeat step 2

This process happens $n$ times (once for each element in the last position), generating $n \times (n-1)! = n!$ permutations.

**Uniqueness**: The swapping pattern ensures no permutation is repeated because we systematically move each element through all positions. ∎

## 3. Algorithm Description

### 3.1 Intuition

Heap's algorithm works recursively:

1. To permute $n$ elements:
   - Recursively permute the first $n-1$ elements
   - After each complete sub-permutation, swap one element into position $n$
   - The swapping pattern depends on whether $n$ is odd or even

2. **Swapping rule**:
   - If $n$ is odd: Always swap first element with last
   - If $n$ is even: Swap $i$-th element with last (where $i$ is iteration counter)

This clever swapping ensures each element gets its turn in each position with minimal work.

### 3.2 Pseudocode

```
function HeapPermute(array, size):
    if size == 1:
        output(array)  // Found a permutation
        return
    
    for i from 0 to size-1:
        // Generate permutations with current element at the end
        HeapPermute(array, size-1)
        
        // Swap to prepare for next iteration
        if size is even:
            swap(array[i], array[size-1])
        else:
            swap(array[0], array[size-1])

// Iterative version (more complex but no recursion)
function HeapPermuteIterative(array):
    n = length(array)
    c = array of n zeros  // Control array
    
    output(array)
    
    i = 0
    while i < n:
        if c[i] < i:
            if i is even:
                swap(array[0], array[i])
            else:
                swap(array[c[i]], array[i])
            
            output(array)
            c[i] += 1
            i = 0
        else:
            c[i] = 0
            i += 1

// With callback for each permutation
function HeapPermuteWithCallback(array, size, callback):
    if size == 1:
        callback(array)
        return
    
    for i from 0 to size-1:
        HeapPermuteWithCallback(array, size-1, callback)
        
        if size is even:
            swap(array[i], array[size-1])
        else:
            swap(array[0], array[size-1])
```

### 3.3 Step-by-Step Example

Generate all permutations of `[1, 2, 3]`:

```
Initial: [1, 2, 3]

Call HeapPermute([1,2,3], 3):

  i=0:
    Call HeapPermute([1,2,3], 2):
      i=0:
        Call HeapPermute([1,2,3], 1):
          Output: [1, 2, 3]  ← Permutation 1
        Swap array[0] and array[1] (size 2 is even)
        → [2, 1, 3]
      
      i=1:
        Call HeapPermute([2,1,3], 1):
          Output: [2, 1, 3]  ← Permutation 2
        (No swap needed, loop ends)
    
    Swap array[0] and array[2] (size 3 is odd)
    → [3, 1, 2]
  
  i=1:
    Call HeapPermute([3,1,2], 2):
      i=0:
        Call HeapPermute([3,1,2], 1):
          Output: [3, 1, 2]  ← Permutation 3
        Swap array[0] and array[1]
        → [1, 3, 2]
      
      i=1:
        Call HeapPermute([1,3,2], 1):
          Output: [1, 3, 2]  ← Permutation 4
    
    Swap array[0] and array[2] (size 3 is odd)
    → [2, 3, 1]
  
  i=2:
    Call HeapPermute([2,3,1], 2):
      i=0:
        Call HeapPermute([2,3,1], 1):
          Output: [2, 3, 1]  ← Permutation 5
        Swap array[0] and array[1]
        → [3, 2, 1]
      
      i=1:
        Call HeapPermute([3,2,1], 1):
          Output: [3, 2, 1]  ← Permutation 6

All 6 permutations generated!
Swaps performed: 5 (one less than number of permutations)
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Total permutations**: $n!$
- **Per permutation**: $O(1)$ swap + $O(n)$ to output
- **Total time**: $O(n \cdot n!)$

The $O(n)$ factor comes from outputting/copying each permutation. The algorithm itself performs minimal swaps.

**Swap count**: Exactly $n! - 1$ swaps (optimal)

### 4.2 Space Complexity

**Recursive**:
- **Recursion depth**: $O(n)$
- **Total space**: $O(n)$

**Iterative**:
- **Control array**: $O(n)$
- **Total space**: $O(n)$

The algorithm operates in-place on the input array.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
// Recursive implementation
fn heap_permute<T: Clone>(arr: &mut [T], size: usize, callback: &mut impl FnMut(&[T])) {
    if size == 1 {
        callback(arr);
        return;
    }
    
    for i in 0..size {
        heap_permute(arr, size - 1, callback);
        
        if size % 2 == 0 {
            arr.swap(i, size - 1);
        } else {
            arr.swap(0, size - 1);
        }
    }
}

// Wrapper function
fn generate_permutations<T: Clone>(arr: &mut [T]) -> Vec<Vec<T>> {
    let mut result = Vec::new();
    heap_permute(arr, arr.len(), &mut |perm| {
        result.push(perm.to_vec());
    });
    result
}

// Iterative implementation
fn heap_permute_iterative<T: Clone>(arr: &mut [T]) -> Vec<Vec<T>> {
    let n = arr.len();
    let mut result = vec![arr.to_vec()];
    let mut c = vec![0; n];
    
    let mut i = 0;
    while i < n {
        if c[i] < i {
            if i % 2 == 0 {
                arr.swap(0, i);
            } else {
                arr.swap(c[i], i);
            }
            
            result.push(arr.to_vec());
            c[i] += 1;
            i = 0;
        } else {
            c[i] = 0;
            i += 1;
        }
    }
    
    result
}

// Generic with trait bounds
fn heap_permute_generic<T>(arr: &mut [T], size: usize, callback: &mut impl FnMut(&[T]))
where
    T: Clone,
{
    if size == 1 {
        callback(arr);
        return;
    }
    
    for i in 0..size {
        heap_permute_generic(arr, size - 1, callback);
        
        let swap_idx = if size % 2 == 0 { i } else { 0 };
        arr.swap(swap_idx, size - 1);
    }
}

// Iterator-based (advanced)
struct HeapPermutations<T> {
    arr: Vec<T>,
    c: Vec<usize>,
    i: usize,
    first: bool,
}

impl<T: Clone> Iterator for HeapPermutations<T> {
    type Item = Vec<T>;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.first {
            self.first = false;
            return Some(self.arr.clone());
        }
        
        while self.i < self.arr.len() {
            if self.c[self.i] < self.i {
                if self.i % 2 == 0 {
                    self.arr.swap(0, self.i);
                } else {
                    self.arr.swap(self.c[self.i], self.i);
                }
                
                self.c[self.i] += 1;
                self.i = 0;
                return Some(self.arr.clone());
            } else {
                self.c[self.i] = 0;
                self.i += 1;
            }
        }
        
        None
    }
}

fn permutations<T: Clone>(arr: Vec<T>) -> HeapPermutations<T> {
    let n = arr.len();
    HeapPermutations {
        arr,
        c: vec![0; n],
        i: 0,
        first: true,
    }
}
```

**Key Features**:
- Closure/callback for handling each permutation
- Generic over any cloneable type
- Iterator implementation for lazy generation
- Both recursive and iterative versions
- In-place swapping

### 5.2 Edge Cases

1. **Empty array**: 1 permutation (empty set)
2. **Single element**: 1 permutation
3. **Two elements**: 2 permutations
4. **Large $n$**: $n!$ grows very fast, memory may be issue
5. **Duplicate elements**: Standard algorithm doesn't handle (will generate duplicates)
6. **Non-comparable types**: Generic implementation handles any type

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Combinatorial Testing**
- Testing all orderings of operations
- Input permutation testing
- Exhaustive state exploration
- Race condition detection

**2. Optimization Problems**
- Traveling Salesman (small instances)
- Task scheduling
- Resource allocation
- Anagram generation

**3. Game Development**
- Puzzle generation
- Move sequence generation
- AI decision tree exploration
- Card shuffling variations

**4. Cryptography**
- Key space exploration (small keys)
- Permutation cipher analysis
- Brute force attacks
- Security testing

**5. Data Analysis**
- Permutation tests (statistics)
- Bootstrap resampling
- Sensitivity analysis
- Monte Carlo methods

**6. String Algorithms**
- Anagram detection and generation
- Pattern matching
- String comparison

### 6.2 Related Algorithms

**Other Permutation Algorithms**:
- **Steinhaus-Johnson-Trotter**: Adjacent swaps only (minimal change)
- **Lexicographic Permutation**: Generates in sorted order
- **Plain Changes**: Bell ringing algorithm
- **Recursive Backtracking**: General but slower

**Related Problems**:
- **Combinations**: Choose k from n ($\binom{n}{k}$)
- **Power Set**: All subsets ($2^n$)
- **Partitions**: Divide set into groups
- **Derangements**: Permutations with no fixed points

**Comparison**:
| Algorithm | Swaps | Order | Space |
|-----------|-------|-------|-------|
| Heap's | $n!-1$ | Non-lexicographic | $O(n)$ |
| SJT | $(n+1)!/2$ | Adjacent only | $O(n)$ |
| Lexicographic | Varies | Sorted | $O(1)$ |
| Backtracking | High | Varies | $O(n)$ |

**When to Use**:
- **Heap's**: Minimal swaps, general permutations
- **SJT**: Need adjacent-only swaps
- **Lexicographic**: Need sorted order
- **Fisher-Yates**: Need random permutation

## 7. References

### Academic Papers
1. Heap, B.R. (1963). "Permutations by Interchanges". *The Computer Journal*, 6(3), 293-298.
2. Sedgewick, R. (1977). "Permutation Generation Methods". *Computing Surveys*, 9(2), 137-164.
3. Knuth, D.E. (2005). *The Art of Computer Programming, Volume 4A: Combinatorial Algorithms, Part 1*. Addison-Wesley. Section 7.2.1.2.

### Books
1. Knuth, D.E. (2011). *The Art of Computer Programming, Volume 4A*. Addison-Wesley.
2. Ruskey, F. (2003). *Combinatorial Generation*. Manuscript.
3. Kreher, D.L., & Stinson, D.R. (1999). *Combinatorial Algorithms: Generation, Enumeration, and Search*. CRC Press.

### Online Resources
1. [Heap's Algorithm - Wikipedia](https://en.wikipedia.org/wiki/Heap%27s_algorithm)
2. [Permutation Generation - GeeksforGeeks](https://www.geeksforgeeks.org/heaps-algorithm-for-generating-permutations/)
3. [Visualization of Heap's Algorithm](https://www.cs.princeton.edu/~rs/talks/perms.pdf)
4. [Permutation Algorithms Comparison](http://www.quickperm.org/)

### Implementation
- Source: `src/general/permutations/heap.rs`
- Tests: Included in source file under `#[cfg(test)] mod tests`
