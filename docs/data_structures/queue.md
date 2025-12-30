# Queue

## 1. Overview

A Queue is a linear data structure following the First-In-First-Out (FIFO) principle: elements are added at the rear and removed from the front. Think of it like a line at a store—the first person in line is the first served.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Design a collection supporting:
- **Enqueue**: Add element to the rear
- **Dequeue**: Remove and return element from the front
- **Peek/Front**: View front element without removal
- **isEmpty**: Check if queue is empty

### 2.2 Mathematical Model

**Queue as Sequence**:
$$Q = \langle a_0, a_1, ..., a_{n-1} \rangle$$

**Operations**:
- $\text{enqueue}(Q, x) = \langle a_0, ..., a_{n-1}, x \rangle$
- $\text{dequeue}(Q) = \langle a_1, ..., a_{n-1} \rangle$ (returns $a_0$)

**FIFO Property**: $\text{dequeue}(\text{enqueue}^n(Q, x_1, ..., x_n)) = x_1$

### 2.3 Invariants

1. Elements exit in the same order they entered
2. Only the front element is accessible
3. Size = number of enqueues - number of dequeues

## 3. Algorithm Description

### 3.1 Intuition

A queue is like a pipe: items enter at one end and exit at the other. The item that has been waiting longest is served next.

### 3.2 Implementations

**Array-Based (Circular Buffer)**:
```
front →  0   1   2   3   4   5   6   7  ← rear
        [·] [A] [B] [C] [D] [·] [·] [·]
            ↑               ↑
          front           rear

After dequeue A:
        [·] [·] [B] [C] [D] [·] [·] [·]
                ↑           ↑
              front       rear

After enqueue E, F:
        [·] [·] [B] [C] [D] [E] [F] [·]
                ↑                   ↑
              front               rear
```

**Linked List Based**:
```
Front                    Rear
  ↓                       ↓
[A|→]──→[B|→]──→[C|→]──→[D|/]

Dequeue: Remove A, front = B
Enqueue E: D.next = E, rear = E
```

### 3.3 Pseudocode

```
// Array-based circular queue
ENQUEUE(queue, value):
    if (rear + 1) % capacity == front:
        error "Queue full" or resize
    queue[rear] = value
    rear = (rear + 1) % capacity
    size++

DEQUEUE(queue):
    if front == rear:
        error "Queue empty"
    value = queue[front]
    front = (front + 1) % capacity
    size--
    return value

PEEK(queue):
    if front == rear:
        error "Queue empty"
    return queue[front]
```

### 3.4 Step-by-Step Example

**Operations**: enqueue(1), enqueue(2), dequeue(), enqueue(3), dequeue()

```
Initial:     []              front=0, rear=0

enqueue(1):  [1]             front=0, rear=1
              ↑
            front/rear

enqueue(2):  [1, 2]          front=0, rear=2
              ↑  ↑
            front rear

dequeue():   [_, 2]          returns 1
                 ↑           front=1, rear=2
               front
               rear→

enqueue(3):  [_, 2, 3]       front=1, rear=3
                 ↑  ↑
               front rear

dequeue():   [_, _, 3]       returns 2
                    ↑        front=2, rear=3
                  front
                  rear→
```

## 4. Complexity Analysis

### 4.1 Time Complexity

| Operation | Array (circular) | Linked List |
|-----------|-----------------|-------------|
| Enqueue   | O(1) amortized  | O(1)        |
| Dequeue   | O(1)            | O(1)        |
| Peek      | O(1)            | O(1)        |
| isEmpty   | O(1)            | O(1)        |

### 4.2 Space Complexity

- **Array**: O(n) where n is capacity (may waste space)
- **Linked List**: O(n) where n is current size (extra pointer overhead)

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
use std::collections::VecDeque;

// Using standard library
let mut queue: VecDeque<i32> = VecDeque::new();
queue.push_back(1);   // enqueue
queue.pop_front();    // dequeue

// Custom implementation
pub struct Queue<T> {
    elements: Vec<T>,
}

impl<T> Queue<T> {
    pub fn new() -> Self {
        Queue { elements: vec![] }
    }
    
    pub fn enqueue(&mut self, item: T) {
        self.elements.push(item);
    }
    
    pub fn dequeue(&mut self) -> Option<T> {
        if self.elements.is_empty() {
            None
        } else {
            Some(self.elements.remove(0))
        }
    }
}
```

**Note**: The above `dequeue` using `remove(0)` is O(n). For O(1):
- Use `VecDeque` (double-ended queue)
- Use circular buffer implementation
- Use two-stack implementation

### 5.2 Two-Stack Queue

```rust
pub struct Queue<T> {
    inbox: Vec<T>,
    outbox: Vec<T>,
}

impl<T> Queue<T> {
    pub fn enqueue(&mut self, item: T) {
        self.inbox.push(item);
    }
    
    pub fn dequeue(&mut self) -> Option<T> {
        if self.outbox.is_empty() {
            while let Some(item) = self.inbox.pop() {
                self.outbox.push(item);
            }
        }
        self.outbox.pop()
    }
}
```

This gives O(1) amortized for both operations!

### 5.3 Edge Cases

| Case | Handling |
|------|----------|
| Dequeue from empty | Return None or panic |
| Circular buffer wrap | Use modulo arithmetic |
| Full circular buffer | Resize or error |
| Single element | After dequeue, front == rear |

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

1. **Task Scheduling**: CPU scheduling, print spooler
2. **BFS (Breadth-First Search)**: Level-order traversal
3. **Message Queues**: RabbitMQ, Kafka
4. **Buffering**: I/O buffers, video streaming
5. **Rate Limiting**: Request queues
6. **Event Systems**: Event loops, GUI event handling

### 6.2 Variants

| Variant | Description |
|---------|-------------|
| **Deque** | Double-ended queue (insert/remove both ends) |
| **Priority Queue** | Dequeue by priority, not arrival order |
| **Circular Queue** | Fixed-size, wraps around |
| **Blocking Queue** | Waits when empty/full (concurrency) |
| **Concurrent Queue** | Thread-safe operations |

### 6.3 Queue vs Stack

| Aspect | Queue (FIFO) | Stack (LIFO) |
|--------|--------------|--------------|
| Order | First in, first out | Last in, first out |
| Use case | Scheduling, BFS | Recursion, DFS, undo |
| Analogy | Line at store | Stack of plates |

## 7. Pitfalls and Optimizations

### 7.1 Common Pitfalls

1. **O(n) Dequeue**: Using array remove(0) instead of circular/deque
2. **Memory Waste**: Fixed-size array too large
3. **Off-by-One**: Circular buffer index calculations
4. **Full vs Empty**: Both have front == rear in naive implementation

### 7.2 Detecting Full vs Empty (Circular Buffer)

**Solution 1**: Track size separately
```rust
is_empty = (size == 0)
is_full = (size == capacity)
```

**Solution 2**: Waste one slot
```rust
is_empty = (front == rear)
is_full = ((rear + 1) % capacity == front)
```

**Solution 3**: Boolean flag
```rust
is_full: bool  // Set when last enqueue fills queue
```

### 7.3 Standard Library

```rust
use std::collections::VecDeque;

let mut q = VecDeque::new();
q.push_back(1);      // enqueue
q.push_back(2);
let front = q.pop_front();  // dequeue -> Some(1)
let peek = q.front();       // peek -> Some(&2)
```

`VecDeque` is a growable ring buffer—the best of both worlds!

## 8. References

- Knuth, D. E. (1997). *The Art of Computer Programming, Volume 1: Fundamental Algorithms* (3rd ed.). Section 2.2.1.
- Sedgewick, R. (1988). *Algorithms* (2nd ed.). Addison-Wesley. Chapter 4.
- Cormen, T. H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Section 10.1.
- Rust std::collections::VecDeque documentation.
