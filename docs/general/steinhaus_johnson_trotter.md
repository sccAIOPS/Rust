# Steinhaus-Johnson-Trotter Algorithm

## 1. Overview

The Steinhaus-Johnson-Trotter algorithm generates all permutations of a sequence by repeatedly swapping adjacent elements. Also known as the "Plain Changes" algorithm or the "Johnson-Trotter algorithm," it was independently discovered by Hugo Steinhaus, Selmer M. Johnson, and Hale F. Trotter in the 1960s.

The algorithm's key property is that each permutation differs from the previous one by a single adjacent transposition, making it ideal for applications requiring minimal change between consecutive permutations (like bell ringing or mechanical devices).

## 2. Mathematical Foundation

### 2.1 Problem Definition

Generate all $n!$ permutations of $n$ elements such that consecutive permutations differ by swapping exactly two adjacent elements.

### 2.2 Mathematical Model

**Input**: Sequence of $n$ distinct elements

**Output**: Sequence of $n!$ permutations where each permutation is obtained from the previous by one adjacent swap

**Direction Concept**: Each element has a direction (left ← or right →):
- An element is **mobile** if it points to a smaller adjacent element
- At each step, swap the largest mobile element with its neighbor in its direction

**Properties**:
1. Generates all $n!$ permutations
2. Each permutation differs from previous by one adjacent swap
3. Total swaps: $\frac{(n+1)!}{2} - 1$
4. Forms a Hamiltonian path in the permutation graph

### 2.3 Correctness Proof

**Theorem**: The SJT algorithm generates all permutations exactly once, using only adjacent swaps.

**Proof Sketch**:
1. **Mobile element always exists**: Until we reach the last permutation (elements in decreasing order, all pointing left), there's always a mobile element.

2. **No cycles**: The sequence of moves forms a Hamiltonian path in the permutation graph where vertices are permutations and edges connect permutations differing by one adjacent swap.

3. **All permutations reached**: The algorithm systematically explores all $n!$ states of the permutation graph.

4. **Adjacent-only property**: By construction, we only swap adjacent elements. ∎

## 3. Algorithm Description

### 3.1 Intuition

The algorithm maintains a direction for each element:

1. Start with elements in ascending order, all pointing left
2. Find the largest mobile element (points to a smaller neighbor)
3. Swap it with its neighbor in its direction
4. Reverse the direction of all elements larger than the swapped element
5. Repeat until no mobile elements remain

**Why it works**: 
- Swapping the largest mobile element ensures we explore systematically
- Reversing directions of larger elements helps us backtrack correctly
- The pattern naturally generates all permutations with adjacent swaps

### 3.2 Pseudocode

```
function SteinhausJohnsonTrotter(n):
    // Initialize
    elements = [1, 2, ..., n]
    directions = [LEFT, LEFT, ..., LEFT]  // All point left initially
    
    output(elements)
    
    while true:
        // Find largest mobile element
        mobile = -1
        mobile_value = -1
        mobile_index = -1
        
        for i from 0 to n-1:
            if isMobile(elements, directions, i):
                if elements[i] > mobile_value:
                    mobile_value = elements[i]
                    mobile_index = i
        
        if mobile_index == -1:
            break  // No mobile elements, done
        
        // Swap mobile element with neighbor in its direction
        if directions[mobile_index] == LEFT:
            swap(elements[mobile_index], elements[mobile_index - 1])
            swap(directions[mobile_index], directions[mobile_index - 1])
            mobile_index = mobile_index - 1
        else:  // RIGHT
            swap(elements[mobile_index], elements[mobile_index + 1])
            swap(directions[mobile_index], directions[mobile_index + 1])
            mobile_index = mobile_index + 1
        
        // Reverse direction of all elements larger than moved element
        for i from 0 to n-1:
            if elements[i] > mobile_value:
                directions[i] = reverse(directions[i])
        
        output(elements)

function isMobile(elements, directions, index):
    n = length(elements)
    value = elements[index]
    direction = directions[index]
    
    if direction == LEFT:
        if index == 0:
            return false  // At left boundary
        return elements[index - 1] < value
    else:  // RIGHT
        if index == n - 1:
            return false  // At right boundary
        return elements[index + 1] < value

// Alternative: Recursive formulation
function SJTRecursive(n):
    if n == 1:
        return [[1]]
    
    permutations = []
    sub_perms = SJTRecursive(n - 1)
    
    for i from 0 to length(sub_perms) - 1:
        perm = sub_perms[i]
        
        // Insert n at all positions (left to right or right to left)
        if i % 2 == 0:
            // Even: insert from right to left
            for pos from n down to 0:
                permutations.append(insertAt(perm, n, pos))
        else:
            // Odd: insert from left to right
            for pos from 0 to n:
                permutations.append(insertAt(perm, n, pos))
    
    return permutations
```

### 3.3 Step-by-Step Example

Generate all permutations of `[1, 2, 3]` using SJT:

```
Notation: ←n means element n points left, n→ means points right

Initial: ←1 ←2 ←3

Step 1: Largest mobile = 3 (points left, 2 < 3)
  Swap 3 with 2
  Result: ←1 ←3 ←2
  Reverse direction of none (no elements > 3)

Step 2: Largest mobile = 3 (points left, 1 < 3)
  Swap 3 with 1
  Result: ←3 ←1 ←2
  Reverse direction of none

Step 3: Largest mobile = 2 (points left, 1 < 2)
  Swap 2 with 1
  Result: ←3 ←2 ←1
  Reverse direction of 3 (3 > 2)
  Result: 3→ ←2 ←1

Step 4: Largest mobile = 3 (points right, 2 < 3)
  Swap 3 with 2
  Result: ←2 3→ ←1
  Reverse direction of none

Step 5: Largest mobile = 3 (points right, 1 < 3)
  Swap 3 with 1
  Result: ←2 ←1 3→
  Reverse direction of none

Step 6: Largest mobile = 2 (points left, 1 < 2)
  Swap 2 with 1
  Result: ←1 ←2 3→
  Reverse direction of 3 (3 > 2)
  Result: ←1 ←2 ←3

Now ←1 ←2 ←3 has no mobile elements (all point left, at boundary)
Algorithm terminates.

All 6 permutations:
1. [1, 2, 3]
2. [1, 3, 2]
3. [3, 1, 2]
4. [3, 2, 1]
5. [2, 3, 1]
6. [2, 1, 3]

Total swaps: 5 (one per transition)
```

## 4. Complexity Analysis

### 4.1 Time Complexity

- **Total permutations**: $n!$
- **Per permutation**:
  - Finding mobile element: $O(n)$
  - Swapping: $O(1)$
  - Reversing directions: $O(n)$ worst case
  - Total per permutation: $O(n)$
- **Total time**: $O(n \cdot n!)$

**Swap count**: $\sum_{k=2}^{n} k! = \frac{(n+1)!}{2} - 1$

### 4.2 Space Complexity

- **Element array**: $O(n)$
- **Direction array**: $O(n)$
- **Total**: $O(n)$

The algorithm operates in-place with linear auxiliary space for directions.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
#[derive(Clone, Copy, PartialEq, Eq)]
enum Direction {
    Left,
    Right,
}

impl Direction {
    fn reverse(&self) -> Self {
        match self {
            Direction::Left => Direction::Right,
            Direction::Right => Direction::Left,
        }
    }
}

fn steinhaus_johnson_trotter<T: Clone + Ord>(mut elements: Vec<T>) -> Vec<Vec<T>> {
    let n = elements.len();
    if n == 0 {
        return vec![vec![]];
    }
    
    let mut directions = vec![Direction::Left; n];
    let mut result = vec![elements.clone()];
    
    loop {
        // Find largest mobile element
        let mobile = find_largest_mobile(&elements, &directions);
        
        if mobile.is_none() {
            break;
        }
        
        let (mobile_idx, mobile_val) = mobile.unwrap();
        
        // Swap with neighbor in direction
        let swap_idx = match directions[mobile_idx] {
            Direction::Left => mobile_idx - 1,
            Direction::Right => mobile_idx + 1,
        };
        
        elements.swap(mobile_idx, swap_idx);
        directions.swap(mobile_idx, swap_idx);
        
        // Reverse direction of larger elements
        for i in 0..n {
            if elements[i] > mobile_val {
                directions[i] = directions[i].reverse();
            }
        }
        
        result.push(elements.clone());
    }
    
    result
}

fn find_largest_mobile<T: Ord>(
    elements: &[T],
    directions: &[Direction],
) -> Option<(usize, T)> 
where
    T: Clone,
{
    let n = elements.len();
    let mut largest_mobile = None;
    
    for i in 0..n {
        if is_mobile(elements, directions, i) {
            let value = elements[i].clone();
            match largest_mobile {
                None => largest_mobile = Some((i, value)),
                Some((_, ref max_val)) if value > *max_val => {
                    largest_mobile = Some((i, value));
                }
                _ => {}
            }
        }
    }
    
    largest_mobile
}

fn is_mobile<T: Ord>(elements: &[T], directions: &[Direction], index: usize) -> bool {
    let n = elements.len();
    let value = &elements[index];
    
    match directions[index] {
        Direction::Left => {
            index > 0 && elements[index - 1] < *value
        }
        Direction::Right => {
            index < n - 1 && elements[index + 1] < *value
        }
    }
}

// Iterator-based implementation
struct SJTIterator<T> {
    elements: Vec<T>,
    directions: Vec<Direction>,
    finished: bool,
}

impl<T: Clone + Ord> Iterator for SJTIterator<T> {
    type Item = Vec<T>;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.finished {
            return None;
        }
        
        let current = self.elements.clone();
        
        // Find and move largest mobile element
        let mobile = find_largest_mobile(&self.elements, &self.directions);
        
        match mobile {
            None => {
                self.finished = true;
            }
            Some((mobile_idx, mobile_val)) => {
                let swap_idx = match self.directions[mobile_idx] {
                    Direction::Left => mobile_idx - 1,
                    Direction::Right => mobile_idx + 1,
                };
                
                self.elements.swap(mobile_idx, swap_idx);
                self.directions.swap(mobile_idx, swap_idx);
                
                for i in 0..self.elements.len() {
                    if self.elements[i] > mobile_val {
                        self.directions[i] = self.directions[i].reverse();
                    }
                }
            }
        }
        
        Some(current)
    }
}

fn sjt_permutations<T: Clone + Ord>(elements: Vec<T>) -> SJTIterator<T> {
    let n = elements.len();
    SJTIterator {
        elements,
        directions: vec![Direction::Left; n],
        finished: false,
    }
}
```

**Key Features**:
- Direction enum for clarity
- Generic over ordered types
- Iterator for lazy generation
- Efficient in-place operations

### 5.2 Edge Cases

1. **Empty sequence**: 1 permutation (empty)
2. **Single element**: 1 permutation
3. **Two elements**: 2 permutations, 1 swap
4. **Duplicate elements**: Algorithm works but generates duplicate permutations
5. **Large $n$**: Exponential growth, memory issues

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Bell Ringing (Change Ringing)**
- Original application
- Minimizing bell ringer movement
- Traditional campanology patterns

**2. Mechanical Permutation Devices**
- Minimal movement required
- Cyclic permutation machines
- Physical constraint satisfaction

**3. Animation & Graphics**
- Smooth transitions between states
- Morphing between configurations
- Gradual transformation sequences

**4. Hardware Testing**
- Testing all connection orders
- Minimal reconfiguration
- Sequential state testing

**5. Sorting Networks**
- Design of comparison networks
- Parallel sorting analysis
- Permutation network design

### 6.2 Related Algorithms

**Comparison with Heap's Algorithm**:
| Feature | SJT | Heap's |
|---------|-----|--------|
| Swaps per perm | 1 (adjacent) | 1 (any) |
| Total swaps | $\frac{(n+1)!}{2}-1$ | $n!-1$ |
| Swap type | Adjacent only | Any positions |
| Use case | Minimal change | Minimal swaps |

**Related Permutation Algorithms**:
- **Heap's Algorithm**: Fewer total swaps
- **Lexicographic**: Sorted order
- **Plain Changes**: Bell ringing specific
- **Star Transposition**: Different swap pattern

**When to Use SJT**:
- Need adjacent-only swaps
- Minimizing physical movement
- Bell ringing applications
- Teaching adjacent transpositions

## 7. References

### Academic Papers
1. Johnson, S.M. (1963). "Generation of Permutations by Adjacent Transpositions". *Mathematics of Computation*, 17(83), 282-285.
2. Trotter, H.F. (1962). "Algorithm 115: Perm". *Communications of the ACM*, 5(8), 434-435.
3. Even, S. (1973). *Algorithmic Combinatorics*. Macmillan.

### Books
1. Knuth, D.E. (2011). *The Art of Computer Programming, Volume 4A: Combinatorial Algorithms*. Addison-Wesley. Section 7.2.1.2.
2. Sedgewick, R. (1977). "Permutation Generation Methods". *ACM Computing Surveys*, 9(2), 137-164.
3. Ruskey, F. (2003). *Combinatorial Generation*. Manuscript.

### Online Resources
1. [Steinhaus-Johnson-Trotter - Wikipedia](https://en.wikipedia.org/wiki/Steinhaus%E2%80%93Johnson%E2%80%93Trotter_algorithm)
2. [Permutation Algorithms](https://www.cut-the-knot.org/do_you_know/AllPerm.shtml)
3. [Change Ringing](http://www.ringing.info/)
4. [Algorithm Visualization](https://www.cs.usfca.edu/~galles/visualization/Permutations.html)

### Implementation
- Source: `src/general/permutations/steinhaus_johnson_trotter.rs`
- Tests: Included in source file under `#[cfg(test)] mod tests`
