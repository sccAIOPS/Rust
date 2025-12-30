# Rust Algorithm Implementation Expertise

## Context
Use this prompt when implementing or analyzing algorithms in Rust. It contains idiomatic patterns, common pitfalls, and best practices specific to algorithm development in Rust.

## Language-Specific Patterns

### Generic Programming
```rust
// Pattern 1: Minimal trait bounds
pub fn sort<T: Ord>(arr: &mut [T]) { }

// Pattern 2: Multiple bounds
pub fn algorithm<T: Ord + Clone + Default>(data: &[T]) -> T { }

// Pattern 3: Where clauses for complex bounds
pub fn complex<T, U>(a: T, b: U) -> T
where
    T: Ord + From<U>,
    U: Copy,
{ }
```

### Slice Patterns
```rust
// In-place mutation
pub fn modify<T>(arr: &mut [T]) { }

// Read-only access
pub fn analyze<T>(arr: &[T]) -> usize { }

// Return new collection
pub fn transform<T: Clone>(arr: &[T]) -> Vec<T> { }
```

### Option/Result Patterns
```rust
// Search returning Option
pub fn find<T: Eq>(arr: &[T], target: &T) -> Option<usize> {
    arr.iter().position(|x| x == target)
}

// Fallible operation returning Result
pub fn parse(s: &str) -> Result<i32, ParseError> {
    s.parse().map_err(|_| ParseError::Invalid)
}
```

## Algorithmic Idioms

### Recursion to Iteration
```rust
// Recursive (may stack overflow)
fn recursive_dfs(node: &Node, visited: &mut HashSet<usize>) {
    if visited.insert(node.id) {
        for neighbor in &node.neighbors {
            recursive_dfs(neighbor, visited);
        }
    }
}

// Iterative (safer)
fn iterative_dfs(start: &Node) {
    let mut stack = vec![start];
    let mut visited = HashSet::new();
    
    while let Some(node) = stack.pop() {
        if visited.insert(node.id) {
            stack.extend(&node.neighbors);
        }
    }
}
```

### Efficient Swapping
```rust
// Use std::mem::swap or slice::swap
arr.swap(i, j);

// For pivot-based algorithms
let pivot = arr.len() - 1;
let mut store_idx = 0;
for i in 0..pivot {
    if arr[i] < arr[pivot] {
        arr.swap(i, store_idx);
        store_idx += 1;
    }
}
```

### Pre-allocation
```rust
// Bad: Multiple reallocations
let mut result = Vec::new();
for item in items {
    result.push(process(item));
}

// Good: Pre-allocate
let mut result = Vec::with_capacity(items.len());
for item in items {
    result.push(process(item));
}

// Best: Use iterators
let result: Vec<_> = items.iter().map(process).collect();
```

## Data Structure Patterns

### Tree Node Pattern
```rust
pub struct TreeNode<T> {
    value: T,
    left: Option<Box<TreeNode<T>>>,
    right: Option<Box<TreeNode<T>>>,
}

impl<T> TreeNode<T> {
    pub fn new(value: T) -> Self {
        Self {
            value,
            left: None,
            right: None,
        }
    }
    
    pub fn with_children(
        value: T,
        left: Option<Box<TreeNode<T>>>,
        right: Option<Box<TreeNode<T>>>,
    ) -> Self {
        Self { value, left, right }
    }
}
```

### Graph Representation
```rust
// Adjacency list (most common)
pub struct Graph {
    adj: Vec<Vec<usize>>,
}

// With edge weights
pub struct WeightedGraph<W> {
    adj: Vec<Vec<(usize, W)>>,
}

// Using HashMap for sparse graphs
pub struct SparseGraph<T> {
    nodes: HashMap<T, Vec<T>>,
}
```

### Heap Implementation Pattern
```rust
pub struct Heap<T: Ord> {
    data: Vec<T>,
}

impl<T: Ord> Heap<T> {
    fn parent(i: usize) -> usize { (i - 1) / 2 }
    fn left(i: usize) -> usize { 2 * i + 1 }
    fn right(i: usize) -> usize { 2 * i + 2 }
    
    fn sift_up(&mut self, mut i: usize) {
        while i > 0 && self.data[i] > self.data[Self::parent(i)] {
            let p = Self::parent(i);
            self.data.swap(i, p);
            i = p;
        }
    }
    
    fn sift_down(&mut self, mut i: usize) {
        loop {
            let mut largest = i;
            let left = Self::left(i);
            let right = Self::right(i);
            
            if left < self.data.len() && self.data[left] > self.data[largest] {
                largest = left;
            }
            if right < self.data.len() && self.data[right] > self.data[largest] {
                largest = right;
            }
            
            if largest == i {
                break;
            }
            
            self.data.swap(i, largest);
            i = largest;
        }
    }
}
```

## Common Pitfalls

### Integer Overflow
```rust
// Pitfall: mid calculation can overflow
let mid = (left + right) / 2;  // WRONG for large values

// Fix: Use subtraction
let mid = left + (right - left) / 2;
```

### Off-by-One Errors
```rust
// Pitfall: Wrong loop bound
for i in 0..arr.len() {
    if arr[i] > arr[i + 1] { }  // WRONG: out of bounds on last iteration
}

// Fix: Use windows or adjust bound
for window in arr.windows(2) {
    if window[0] > window[1] { }
}
```

### Unnecessary Cloning
```rust
// Pitfall: Cloning when borrowing works
let result = expensive_data.clone();
process(&result);

// Fix: Just borrow
process(&expensive_data);
```

### Index vs Iterator
```rust
// Pitfall: Index-based iteration
for i in 0..arr.len() {
    process(arr[i]);  // Bounds check on every access
}

// Fix: Use iterators
for item in arr.iter() {
    process(item);  // No bounds checks
}
```

## Performance Tips

### 1. Use `std::mem::swap` for swapping
```rust
use std::mem;
mem::swap(&mut a, &mut b);
```

### 2. Pre-size collections
```rust
let mut vec = Vec::with_capacity(n);
let mut map = HashMap::with_capacity(n);
```

### 3. Avoid unnecessary bounds checks
```rust
// When you've already validated bounds
unsafe { *arr.get_unchecked(i) }
```

### 4. Use iterators for fusion
```rust
// Single pass, no intermediate allocations
arr.iter()
   .filter(|x| condition(x))
   .map(|x| transform(x))
   .collect()
```

### 5. Consider stack allocation for small data
```rust
// Instead of Vec for small, fixed-size data
let arr: [i32; 8] = [0; 8];
```

## Testing Patterns

### Standard Test Suite
```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test] fn test_empty() { }
    #[test] fn test_single() { }
    #[test] fn test_two_elements() { }
    #[test] fn test_basic() { }
    #[test] fn test_sorted() { }
    #[test] fn test_reverse() { }
    #[test] fn test_duplicates() { }
    #[test] fn test_negative() { }
    
    #[test]
    #[ignore]
    fn test_large_input() { }
}
```

### Property-Based Tests
```rust
use quickcheck_macros::quickcheck;

#[quickcheck]
fn prop_sorted(mut arr: Vec<i32>) -> bool {
    sort(&mut arr);
    arr.windows(2).all(|w| w[0] <= w[1])
}
```
