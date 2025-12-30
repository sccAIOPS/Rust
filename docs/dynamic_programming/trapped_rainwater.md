# Trapping Rain Water

## 1. Overview

The Trapping Rain Water problem calculates how much water can be trapped between bars after raining. This classic interview problem demonstrates the power of precomputation and two-pointer techniques.

**File**: `src/dynamic_programming/trapped_rainwater.rs`

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given an array $height[0..n-1]$ representing bar heights:

Water at position $i$ = $\min(\max_{j \leq i} height[j], \max_{j \geq i} height[j]) - height[i]$

Total water = $\sum_{i=0}^{n-1} \text{water}[i]$

### 2.2 Key Insight

Water above bar $i$ is bounded by:
- Tallest bar to its left
- Tallest bar to its right
- The minimum of these two determines water level

## 3. Algorithm Description

### 3.1 Approach 1: Precomputation (DP)

```
FUNCTION trap_water(height)
    n ← len(height)
    left_max[0..n] ← 0
    right_max[0..n] ← 0
    
    // Precompute left maximums
    left_max[0] ← height[0]
    FOR i ← 1 TO n-1 DO
        left_max[i] ← max(left_max[i-1], height[i])
    END FOR
    
    // Precompute right maximums
    right_max[n-1] ← height[n-1]
    FOR i ← n-2 DOWNTO 0 DO
        right_max[i] ← max(right_max[i+1], height[i])
    END FOR
    
    // Calculate water
    water ← 0
    FOR i ← 0 TO n-1 DO
        water ← water + min(left_max[i], right_max[i]) - height[i]
    END FOR
    
    RETURN water
END FUNCTION
```

### 3.2 Step-by-Step Example

**Input**: height = [0, 1, 0, 2, 1, 0, 1, 3, 2, 1, 2, 1]

```
Index:      0  1  2  3  4  5  6  7  8  9 10 11
Height:     0  1  0  2  1  0  1  3  2  1  2  1
Left Max:   0  1  1  2  2  2  2  3  3  3  3  3
Right Max:  3  3  3  3  3  3  3  3  2  2  2  1

Water at each position:
i=0: min(0,3) - 0 = 0
i=1: min(1,3) - 1 = 0
i=2: min(1,3) - 0 = 1  ✓
i=3: min(2,3) - 2 = 0
i=4: min(2,3) - 1 = 1  ✓
i=5: min(2,3) - 0 = 2  ✓
i=6: min(2,3) - 1 = 1  ✓
i=7: min(3,3) - 3 = 0
i=8: min(3,2) - 2 = 0
i=9: min(3,2) - 1 = 1  ✓
i=10: min(3,2) - 2 = 0
i=11: min(3,1) - 1 = 0

Total: 0+0+1+0+1+2+1+0+0+1+0+0 = 6
```

**Visual**:
```
       █
   █   ██ █
 █ ██ ████ █
 ▓█▓██▓████▓█    ▓ = water
```

## 4. Complexity Analysis

### 4.1 DP Approach
- **Time**: O(n) - Three passes
- **Space**: O(n) - Two auxiliary arrays

### 4.2 Two-Pointer Approach
- **Time**: O(n) - Single pass
- **Space**: O(1) - Only pointers

## 5. Implementation Notes

### 5.1 Rust Implementation (DP)

```rust
pub fn trapped_rainwater(heights: &[u32]) -> u32 {
    if heights.len() < 3 {
        return 0;
    }
    
    let n = heights.len();
    let mut left_max = vec![0u32; n];
    let mut right_max = vec![0u32; n];
    
    // Build left_max
    left_max[0] = heights[0];
    for i in 1..n {
        left_max[i] = left_max[i - 1].max(heights[i]);
    }
    
    // Build right_max
    right_max[n - 1] = heights[n - 1];
    for i in (0..n - 1).rev() {
        right_max[i] = right_max[i + 1].max(heights[i]);
    }
    
    // Calculate water
    (0..n)
        .map(|i| left_max[i].min(right_max[i]) - heights[i])
        .sum()
}
```

### 5.2 Two-Pointer Solution (O(1) Space)

```rust
pub fn trapped_rainwater_optimized(heights: &[u32]) -> u32 {
    if heights.len() < 3 {
        return 0;
    }
    
    let (mut left, mut right) = (0, heights.len() - 1);
    let (mut left_max, mut right_max) = (0u32, 0u32);
    let mut water = 0u32;
    
    while left < right {
        if heights[left] < heights[right] {
            if heights[left] >= left_max {
                left_max = heights[left];
            } else {
                water += left_max - heights[left];
            }
            left += 1;
        } else {
            if heights[right] >= right_max {
                right_max = heights[right];
            } else {
                water += right_max - heights[right];
            }
            right -= 1;
        }
    }
    
    water
}
```

### 5.3 Why Two-Pointer Works

At each step, we move the pointer with smaller height:
- If `left_max < right_max`, water at left is bounded by left_max
- We don't need the exact right_max, just that it's higher
- This guarantees correctness while using O(1) space

### 5.4 Edge Cases

| Case | Result |
|------|--------|
| Empty array | 0 |
| Single element | 0 |
| Two elements | 0 |
| Monotonic increasing | 0 |
| Monotonic decreasing | 0 |
| All same height | 0 |
| V-shape | Fills the V |

## 6. Alternative Approach: Stack

```rust
pub fn trap_stack(height: &[u32]) -> u32 {
    let mut stack: Vec<usize> = Vec::new();
    let mut water = 0u32;
    
    for i in 0..height.len() {
        while !stack.is_empty() && height[i] > height[*stack.last().unwrap()] {
            let top = stack.pop().unwrap();
            if stack.is_empty() { break; }
            
            let distance = (i - stack.last().unwrap() - 1) as u32;
            let bounded_height = height[i].min(height[*stack.last().unwrap()]) 
                                 - height[top];
            water += distance * bounded_height;
        }
        stack.push(i);
    }
    
    water
}
```

## 7. Applications

1. **Civil Engineering**: Dam and reservoir capacity
2. **Geography**: Flood simulation, watershed analysis
3. **Image Processing**: Filling regions in terrain maps
4. **Game Development**: Water physics simulation

## 8. Variants

### 8.1 2D Trapping Rain Water

Extend to 2D height map - use BFS/Priority Queue:
```
Given:     Solution:
1 4 3 1    Trapped water = 4
3 2 1 3    (The '2' and middle '1's can hold water)
2 3 3 2
```

### 8.2 Container With Most Water

Different problem: Find two bars that form container with most water
- Two-pointer approach, O(n)
- Move pointer with shorter height inward

## 9. Comparison of Approaches

| Approach | Time | Space | Difficulty |
|----------|------|-------|------------|
| Brute Force | O(n²) | O(1) | Easy |
| DP Precompute | O(n) | O(n) | Medium |
| Two Pointer | O(n) | O(1) | Medium |
| Stack | O(n) | O(n) | Hard |

## 10. Common Mistakes

1. **Off-by-one errors**: Boundary handling
2. **Negative water**: Ensure non-negative before summing
3. **Integer overflow**: Use appropriate types for large inputs
4. **Empty array**: Always check for edge cases

## 11. References

1. [LeetCode Problem 42](https://leetcode.com/problems/trapping-rain-water/)
2. [LeetCode Problem 407](https://leetcode.com/problems/trapping-rain-water-ii/) (2D version)
3. [LeetCode Problem 11](https://leetcode.com/problems/container-with-most-water/) (related)
