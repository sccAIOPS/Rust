# Tower of Hanoi

## 1. Overview

The Tower of Hanoi is a classic mathematical puzzle invented by French mathematician Édouard Lucas in 1883. The puzzle consists of three rods and a number of disks of different sizes that can slide onto any rod. The objective is to move the entire stack from one rod to another, following simple rules.

Beyond being a puzzle, it's a fundamental example of recursive problem-solving and has applications in computer science, cognitive psychology, and backup rotation schemes.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given:
- Three rods: Source (A), Auxiliary (B), Destination (C)
- $n$ disks of different sizes stacked on rod A in descending order (largest at bottom)

**Goal**: Move all disks from rod A to rod C

**Rules**:
1. Only one disk can be moved at a time
2. Only the top disk from any rod can be moved
3. A disk cannot be placed on top of a smaller disk

### 2.2 Mathematical Model

**Recurrence Relation**:

Let $T(n)$ be the minimum number of moves to transfer $n$ disks.

$$T(n) = \begin{cases} 
1 & \text{if } n = 1 \\
2T(n-1) + 1 & \text{if } n > 1
\end{cases}$$

**Closed Form Solution**:
$$T(n) = 2^n - 1$$

**Proof by Induction**:
- Base case: $T(1) = 1 = 2^1 - 1$ ✓
- Inductive step: Assume $T(k) = 2^k - 1$
  - $T(k+1) = 2T(k) + 1 = 2(2^k - 1) + 1 = 2^{k+1} - 2 + 1 = 2^{k+1} - 1$ ✓

### 2.3 Correctness Proof

**Theorem**: The recursive algorithm solves the Tower of Hanoi problem optimally.

**Proof**:
1. **Base case** ($n=1$): Trivially move the single disk. ✓

2. **Inductive step**: Assume the algorithm works for $n-1$ disks.
   - To move $n$ disks from A to C using B as auxiliary:
     a) Move top $n-1$ disks from A to B (using C as auxiliary) - works by induction
     b) Move disk $n$ from A to C - valid (C is empty or has larger disks)
     c) Move $n-1$ disks from B to C (using A as auxiliary) - works by induction

3. **Optimality**: Any solution must:
   - Move disk $n$ at least once (to destination)
   - Clear disks above it first: $T(n-1)$ moves
   - Move remaining disks on top: $T(n-1)$ moves
   - Total: $2T(n-1) + 1$ moves (matches our formula) ∎

## 3. Algorithm Description

### 3.1 Intuition

The recursive solution elegantly breaks down the problem:

1. **Base case**: Moving 1 disk is trivial—just move it
2. **Recursive case**: To move $n$ disks from A to C:
   - **Step 1**: Move $n-1$ disks from A to B (using C as spare)
   - **Step 2**: Move the largest disk from A to C
   - **Step 3**: Move $n-1$ disks from B to C (using A as spare)

The key insight: we can temporarily use any rod as destination or auxiliary. The problem reduces by one disk at each level of recursion.

### 3.2 Pseudocode

```
function HanoiRecursive(n, source, destination, auxiliary):
    // Base case: only one disk
    if n == 1:
        print "Move disk 1 from", source, "to", destination
        return
    
    // Step 1: Move n-1 disks from source to auxiliary
    HanoiRecursive(n-1, source, auxiliary, destination)
    
    // Step 2: Move the largest disk from source to destination
    print "Move disk", n, "from", source, "to", destination
    
    // Step 3: Move n-1 disks from auxiliary to destination
    HanoiRecursive(n-1, auxiliary, destination, source)

// Iterative approach (using stack to simulate recursion)
function HanoiIterative(n):
    total_moves = 2^n - 1
    source = 'A'
    auxiliary = 'B'
    destination = 'C'
    
    // If n is even, swap auxiliary and destination
    if n % 2 == 0:
        swap(auxiliary, destination)
    
    for move from 1 to total_moves:
        if move % 3 == 1:
            // Move between source and destination
            moveDisk(source, destination)
        else if move % 3 == 2:
            // Move between source and auxiliary
            moveDisk(source, auxiliary)
        else:
            // Move between auxiliary and destination
            moveDisk(auxiliary, destination)

function moveDisk(from_rod, to_rod):
    // Move the smaller top disk between two rods
    // (Implementation needs to track actual disk positions)
```

### 3.3 Step-by-Step Example

Solve Tower of Hanoi with $n=3$ disks.

```
Initial state:
  A: [3, 2, 1]  (3=largest, 1=smallest)
  B: []
  C: []

Goal: Move all to C

Call: Hanoi(3, A, C, B)

├─ Call: Hanoi(2, A, B, C)  [Move 2 disks from A to B using C]
│  ├─ Call: Hanoi(1, A, C, B)
│  │  └─ Move disk 1: A → C
│  │     A:[3,2] B:[] C:[1]
│  │
│  ├─ Move disk 2: A → B
│  │  A:[3] B:[2] C:[1]
│  │
│  └─ Call: Hanoi(1, C, B, A)
│     └─ Move disk 1: C → B
│        A:[3] B:[2,1] C:[]
│
├─ Move disk 3: A → C
│  A:[] B:[2,1] C:[3]
│
└─ Call: Hanoi(2, B, C, A)  [Move 2 disks from B to C using A]
   ├─ Call: Hanoi(1, B, A, C)
   │  └─ Move disk 1: B → A
   │     A:[1] B:[2] C:[3]
   │
   ├─ Move disk 2: B → C
   │  A:[1] B:[] C:[3,2]
   │
   └─ Call: Hanoi(1, A, C, B)
      └─ Move disk 1: A → C
         A:[] B:[] C:[3,2,1]

Final state:
  A: []
  B: []
  C: [3, 2, 1]  ✓

Total moves: 7 = 2³ - 1
```

## 4. Complexity Analysis

### 4.1 Time Complexity

**Recursive Analysis**:
- Recurrence: $T(n) = 2T(n-1) + O(1)$
- Solution: $T(n) = O(2^n)$
- Exact number of moves: $2^n - 1$

**Why exponential?**
- Each call spawns 2 recursive calls
- Recursion tree has $2^n - 1$ nodes
- Each node does $O(1)$ work

**Practical implications**:
- $n=10$: 1,023 moves (~1 second)
- $n=20$: 1,048,575 moves (~17 minutes at 1000 moves/sec)
- $n=64$: $2^{64} - 1$ moves (~585 billion years!)

### 4.2 Space Complexity

- **Recursive**: $O(n)$ stack depth
- **Iterative**: $O(n)$ for storing rod states
- **Output**: $O(2^n)$ if storing all moves

The recursion depth is $n$ because we reduce the problem size by 1 at each level.

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
// Simple recursive implementation
fn hanoi(n: u32, source: char, destination: char, auxiliary: char) {
    if n == 1 {
        println!("Move disk 1 from {} to {}", source, destination);
        return;
    }
    
    hanoi(n - 1, source, auxiliary, destination);
    println!("Move disk {} from {} to {}", n, source, destination);
    hanoi(n - 1, auxiliary, destination, source);
}

// With move collection
fn hanoi_with_moves(n: u32, source: char, dest: char, aux: char) -> Vec<(char, char)> {
    if n == 0 {
        return Vec::new();
    }
    
    if n == 1 {
        return vec![(source, dest)];
    }
    
    let mut moves = Vec::new();
    
    // Move n-1 disks from source to auxiliary
    moves.extend(hanoi_with_moves(n - 1, source, aux, dest));
    
    // Move largest disk from source to destination
    moves.push((source, dest));
    
    // Move n-1 disks from auxiliary to destination
    moves.extend(hanoi_with_moves(n - 1, aux, dest, source));
    
    moves
}

// Iterative approach
fn hanoi_iterative(n: u32) -> Vec<(char, char)> {
    let total_moves = (1 << n) - 1; // 2^n - 1
    let mut moves = Vec::with_capacity(total_moves as usize);
    
    let (mut src, mut aux, mut dst) = ('A', 'B', 'C');
    
    // If even number of disks, swap auxiliary and destination
    if n % 2 == 0 {
        std::mem::swap(&mut aux, &mut dst);
    }
    
    for i in 1..=total_moves {
        let m = match i % 3 {
            1 => (src, dst),
            2 => (src, aux),
            0 => (aux, dst),
            _ => unreachable!(),
        };
        moves.push(m);
    }
    
    moves
}

// With actual state tracking
#[derive(Debug)]
struct TowerState {
    rods: [Vec<u32>; 3],
}

impl TowerState {
    fn new(n: u32) -> Self {
        let mut state = TowerState {
            rods: [Vec::new(), Vec::new(), Vec::new()],
        };
        // Initialize first rod with disks n, n-1, ..., 1
        state.rods[0] = (1..=n).rev().collect();
        state
    }
    
    fn move_disk(&mut self, from: usize, to: usize) -> Result<(), &str> {
        if let Some(disk) = self.rods[from].pop() {
            if let Some(&top) = self.rods[to].last() {
                if disk > top {
                    self.rods[from].push(disk);
                    return Err("Cannot place larger disk on smaller disk");
                }
            }
            self.rods[to].push(disk);
            Ok(())
        } else {
            Err("No disk to move")
        }
    }
}
```

### 5.2 Edge Cases

1. **$n = 0$**: No moves needed (empty puzzle)
2. **$n = 1$**: Single move
3. **Very large $n$**: Exponential time, risk of stack overflow
4. **Invalid moves**: State tracking ensures rules are followed
5. **Stack depth**: Recursion depth of $n$ may cause stack overflow for large $n$

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Backup Rotation (Grandfather-Father-Son)**
- Daily, weekly, monthly backups
- Optimal rotation strategy
- Minimizing tape/disk usage

**2. Recursive Algorithm Teaching**
- Classic example in CS education
- Demonstrates divide-and-conquer
- Illustrates exponential complexity

**3. Cognitive Psychology**
- Problem-solving research
- Working memory studies
- Planning and foresight testing

**4. Compilers & Interpreters**
- Register allocation analogy
- Instruction scheduling
- Temporary variable management

**5. Robot Motion Planning**
- Path planning with constraints
- Object manipulation
- Assembly sequence planning

**6. Puzzle Games**
- Game mechanics inspiration
- Difficulty scaling (vary $n$)
- AI move generation

### 6.2 Related Algorithms

**Variations**:
- **Multi-peg Hanoi**: More than 3 pegs (Frame-Stewart algorithm)
- **Cyclic Hanoi**: Moves only in one direction
- **Colored Hanoi**: Disks with color constraints
- **Tower of Bucharest**: Different movement rules

**Related Problems**:
- **Josephus Problem**: Similar recursive structure
- **Gray Code**: Binary reflected Gray code (related to iterative solution)
- **Binary Tree Traversal**: Similar recursion pattern
- **Merge Sort**: Divide-and-conquer with similar complexity

**Recursive Patterns**:
- Divide and conquer
- Tree recursion
- Tail recursion (not in standard Hanoi)

## 7. References

### Academic Papers
1. Lucas, É. (1883). *Récréations mathématiques*. Gauthier-Villars.
2. Hinz, A.M., et al. (2013). *The Tower of Hanoi – Myths and Maths*. Birkhäuser.
3. Frame, J.S., & Stewart, B.M. (1941). "Solution to Advanced Problem 3918". *American Mathematical Monthly*, 48, 216-219.

### Books
1. Knuth, D.E. (2011). *The Art of Computer Programming, Volume 4A: Combinatorial Algorithms*. Addison-Wesley.
2. Graham, R.L., et al. (1994). *Concrete Mathematics* (2nd ed.). Addison-Wesley. Section 1.1.
3. Roberts, E. (2005). *Thinking Recursively with Java*. Wiley. Chapter 8.

### Online Resources
1. [Tower of Hanoi - Wikipedia](https://en.wikipedia.org/wiki/Tower_of_Hanoi)
2. [Interactive Visualization](https://www.mathsisfun.com/games/towerofhanoi.html)
3. [Hanoi Puzzle Analysis](http://www.cut-the-knot.org/recurrence/hanoi.shtml)
4. [CS50 - Tower of Hanoi](https://cs50.harvard.edu/x/2023/notes/3/)

### Implementation
- Source: `src/general/hanoi.rs`
- Tests: Included in source file under `#[cfg(test)] mod tests`
