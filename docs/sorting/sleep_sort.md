# Sleep Sort

## 1. Overview

Sleep Sort is a humorous, impractical sorting algorithm that works by creating a separate thread for each element, where each thread sleeps for a duration proportional to its value before outputting the element. Elements "wake up" in sorted order.

### Key Characteristics
- **Type**: Time-based, parallel sort
- **In-place**: No (requires threads)
- **Stable**: Depends on thread scheduling
- **Practical**: Absolutely not!

## 2. Mathematical Foundation

### 2.1 The Core Idea

For each element $x_i$, create a thread that:
1. Sleeps for $x_i \cdot k$ time units (where k is a constant)
2. Outputs $x_i$

Since larger values sleep longer, they wake up later, producing sorted output.

### 2.2 Issues

1. **Race conditions**: Threads with close values may wake in wrong order
2. **Negative numbers**: Cannot sleep negative time
3. **Resource consumption**: n threads for n elements
4. **Time complexity**: O(max(arr)) wall-clock time

## 3. Algorithm Description

### 3.1 Pseudocode

```
SLEEP_SORT(A)
    result ← []
    threads ← []
    
    for each element x in A do
        t ← CREATE_THREAD:
            SLEEP(x * TIME_UNIT)
            APPEND(result, x)
        threads.append(t)
    
    for each t in threads do
        WAIT(t)
    
    return result
```

### 3.2 Conceptual Example

Sorting `[3, 1, 4, 1, 5]`:

```
Time 0: Start all threads
  Thread 1: sleep(3)
  Thread 2: sleep(1)
  Thread 3: sleep(4)
  Thread 4: sleep(1)
  Thread 5: sleep(5)

Time 1: Threads 2,4 wake → output [1, 1]
Time 3: Thread 1 wakes → output [1, 1, 3]
Time 4: Thread 3 wakes → output [1, 1, 3, 4]
Time 5: Thread 5 wakes → output [1, 1, 3, 4, 5]
```

## 4. Complexity Analysis

| Metric | Complexity |
|--------|------------|
| **Wall-clock Time** | O(max(arr)) |
| **CPU Time** | O(n) for thread creation |
| **Space** | O(n) threads |
| **Comparisons** | 0 (!) |

### 4.1 The "Complexity" Paradox

- Sleep Sort performs **zero comparisons**
- Yet it's not O(1) - it's bounded by the maximum value
- This highlights limitations of traditional complexity analysis

## 5. Implementation

```rust
use std::sync::mpsc;
use std::thread;
use std::time::Duration;

pub fn sleep_sort(arr: &[usize]) -> Vec<usize> {
    let (tx, rx) = mpsc::channel();
    let mut handles = vec![];

    for &num in arr {
        let tx_clone = tx.clone();
        let handle = thread::spawn(move || {
            thread::sleep(Duration::from_millis(num as u64 * 10));
            tx_clone.send(num).unwrap();
        });
        handles.push(handle);
    }

    drop(tx); // Close sender to allow receiver to terminate

    let mut result = vec![];
    for received in rx {
        result.push(received);
    }

    for handle in handles {
        handle.join().unwrap();
    }

    result
}
```

## 6. Problems and Limitations

### 6.1 Race Conditions
```
Values [1, 1, 2]:
- Both 1's sleep for same duration
- Thread scheduling determines order
- Result may be unstable
```

### 6.2 Value Constraints
| Issue | Impact |
|-------|--------|
| Negative numbers | Cannot sleep negative time |
| Zero | May cause race with thread startup |
| Large numbers | Extremely long execution |
| Floating point | Need to scale appropriately |

### 6.3 Resource Issues
- n threads for n elements
- Thread creation overhead dominates for small values
- OS thread limits may be exceeded

## 7. Variants

### 7.1 Scaled Sleep Sort
```rust
// Scale sleep time to handle range
let min_val = arr.iter().min().unwrap();
let scale = 10; // ms per unit
sleep(Duration::from_millis((num - min_val) * scale));
```

### 7.2 Negative-Aware Version
```rust
// Shift all values positive
let offset = arr.iter().min().unwrap().abs();
// Sleep for (value + offset) time
```

## 8. Educational Value

Sleep Sort teaches:
1. **Unconventional thinking**: Different paradigms for sorting
2. **Concurrency issues**: Race conditions, thread synchronization
3. **Complexity limitations**: Wall-clock vs. computational complexity
4. **Practical constraints**: OS resources, scheduling fairness

## 9. When to "Use" (Never)

- **Teaching**: Concurrency concepts
- **Humor**: Programming jokes
- **Interviews**: Discussing algorithm trade-offs
- **Never**: Any real application

## 10. References

1. "Sleep Sort" - 4chan /prog/ (2011)
2. Stack Overflow discussions on novelty sorting algorithms

## 11. Source Code

**Implementation**: [src/sorting/sleep_sort.rs](../../src/sorting/sleep_sort.rs)
