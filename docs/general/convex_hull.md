# Convex Hull

## 1. Overview

The convex hull of a set of points is the smallest convex polygon that contains all the points. Imagine placing rubber bands around nails on a board—the shape formed is the convex hull. This fundamental problem in computational geometry has applications in computer graphics, pattern recognition, robotics, and geographic information systems.

Common algorithms include Graham's Scan, Jarvis March (Gift Wrapping), QuickHull, and Chan's algorithm, each with different time complexities and use cases.

## 2. Mathematical Foundation

### 2.1 Problem Definition

Given a set of points $P = \{p_1, p_2, ..., p_n\}$ in the plane, where each point $p_i = (x_i, y_i)$, find the minimal convex polygon $H$ such that:

1. All points in $P$ are either inside or on the boundary of $H$
2. $H$ is convex: for any two points in $H$, the line segment connecting them lies entirely within $H$

### 2.2 Mathematical Model

**Input**: Set of points $P \subset \mathbb{R}^2$ with $|P| = n$

**Output**: Ordered sequence of vertices $H = \langle v_1, v_2, ..., v_k \rangle$ where $v_i \in P$ and $k \leq n$

**Convexity Condition**: For any three consecutive points $v_i, v_{i+1}, v_{i+2}$ in $H$, the turn from $v_i \to v_{i+1} \to v_{i+2}$ is counterclockwise (or all clockwise).

**Cross Product Test**: Points $p, q, r$ form a counterclockwise turn if:
$$\text{cross}(p, q, r) = (q_x - p_x)(r_y - p_y) - (q_y - p_y)(r_x - p_x) > 0$$

### 2.3 Correctness Proof

**Theorem** (Graham's Scan): The algorithm produces the correct convex hull.

**Proof Sketch**:
1. **Starting Point**: The lowest point (minimum y-coordinate) must be on the hull
2. **Sorted Order**: Processing points by polar angle ensures we traverse the boundary
3. **Invariant**: At each step, the stack contains a partial hull
4. **Backtracking**: Removing non-left-turning points maintains convexity
5. **Completeness**: All extreme points are included; interior points are correctly excluded

## 3. Algorithm Description

### 3.1 Intuition

**Graham's Scan** (most common):
1. Find the lowest point (anchor)—guaranteed to be on the hull
2. Sort all other points by polar angle relative to anchor
3. Process points in sorted order, maintaining a stack of hull points
4. For each new point, remove points from the stack that would create a right turn (concave)
5. Add the new point to the stack
6. The stack contains the convex hull vertices in order

**Jarvis March** (Gift Wrapping):
- Start from the leftmost point
- At each step, find the point that makes the smallest counterclockwise angle
- Wrap around the point set like wrapping a gift
- Simpler but slower: $O(nh)$ where $h$ is hull size

### 3.2 Pseudocode (Graham's Scan)

```
function ConvexHull(points):
    if len(points) < 3:
        return points  // Degenerate case
    
    // Step 1: Find anchor point (lowest y, then leftmost)
    anchor = findLowestPoint(points)
    
    // Step 2: Sort by polar angle relative to anchor
    sorted_points = sortByPolarAngle(points, anchor)
    
    // Step 3: Initialize stack with first three points
    stack = [anchor, sorted_points[0], sorted_points[1]]
    
    // Step 4: Process remaining points
    for i = 2 to len(sorted_points) - 1:
        // Remove points that create right turn
        while len(stack) > 1 and !isLeftTurn(stack[-2], stack[-1], sorted_points[i]):
            stack.pop()
        
        stack.push(sorted_points[i])
    
    return stack

function isLeftTurn(p, q, r):
    // Cross product test
    cross = (q.x - p.x) * (r.y - p.y) - (q.y - p.y) * (r.x - p.x)
    return cross > 0

function findLowestPoint(points):
    lowest = points[0]
    for point in points:
        if point.y < lowest.y or (point.y == lowest.y and point.x < lowest.x):
            lowest = point
    return lowest

function sortByPolarAngle(points, anchor):
    // Sort by angle, then by distance if angles are equal
    return sorted(points, key=lambda p: (angle(anchor, p), distance(anchor, p)))
```

### 3.3 Step-by-Step Example

Given points: `A(0,0), B(2,2), C(4,0), D(1,1), E(3,1), F(0,3)`

**Step 1**: Find anchor (lowest point)
- Anchor = `A(0,0)`

**Step 2**: Sort by polar angle from A

```
Angles from A(0,0):
B(2,2): 45°
C(4,0): 0°
D(1,1): 45°
E(3,1): 18.4°
F(0,3): 90°

Sorted order: A(0,0), C(4,0), E(3,1), D(1,1), B(2,2), F(0,3)
```

**Step 3**: Process points

```
Stack: [A, C, E]

Check D(1,1):
  Turn C→E→D: cross product = (3-4)*(1-0) - (1-0)*(1-1) = -1 < 0 (right turn)
  Remove E
  Stack: [A, C, D]
  Turn C→D is left turn from A
  
Check B(2,2):
  Turn C→D→B: cross product = (1-4)*(2-0) - (1-0)*(2-1) = -6 - 1 = -7 < 0
  Remove D
  Turn C→B from A: cross product = positive
  Stack: [A, C, B]
  
Check F(0,3):
  Turn C→B→F: cross product positive (left turn)
  Turn B→F from A: cross product positive
  Stack: [A, C, B, F]
```

**Result**: Convex Hull = `[A(0,0), C(4,0), B(2,2), F(0,3)]`

## 4. Complexity Analysis

### 4.1 Time Complexity

**Graham's Scan**:
- Finding anchor: $O(n)$
- Sorting by angle: $O(n \log n)$
- Processing points: $O(n)$ (each point pushed/popped at most once)
- **Total**: $O(n \log n)$

**Jarvis March**:
- For each hull point, scan all points: $O(nh)$ where $h$ is hull size
- **Best case**: $O(n)$ when $h = O(1)$
- **Worst case**: $O(n^2)$ when $h = n$ (all points on hull)

**QuickHull**:
- **Average**: $O(n \log n)$
- **Worst**: $O(n^2)$ (rare in practice)

**Chan's Algorithm**:
- **Optimal**: $O(n \log h)$ where $h$ is output size
- Combines Jarvis march with Graham scan

### 4.2 Space Complexity

- **Graham's Scan**: $O(n)$ for sorted array and stack
- **Jarvis March**: $O(h)$ for hull storage (can be done in-place)
- **All algorithms**: Output requires $O(h)$ space where $h \leq n$

## 5. Implementation Notes

### 5.1 Rust-Specific Considerations

```rust
#[derive(Debug, Clone, Copy, PartialEq)]
struct Point {
    x: f64,
    y: f64,
}

impl Point {
    fn cross_product(&self, p2: &Point, p3: &Point) -> f64 {
        // Cross product: (p2 - self) × (p3 - self)
        (p2.x - self.x) * (p3.y - self.y) - (p2.y - self.y) * (p3.x - self.x)
    }
    
    fn polar_angle_from(&self, anchor: &Point) -> f64 {
        (self.y - anchor.y).atan2(self.x - anchor.x)
    }
    
    fn distance_to(&self, other: &Point) -> f64 {
        ((self.x - other.x).powi(2) + (self.y - other.y).powi(2)).sqrt()
    }
}

// Generic over coordinate types
fn convex_hull<T: Ord + Copy>(points: &mut Vec<Point>) -> Vec<Point> {
    // Implementation
}

// Use Vec for dynamic stack
// Avoid floating-point errors with epsilon comparisons
const EPSILON: f64 = 1e-10;
fn is_zero(x: f64) -> bool {
    x.abs() < EPSILON
}
```

**Key Considerations**:
- Use `f64` for coordinates to handle decimal points
- Handle collinear points carefully (cross product ≈ 0)
- In-place sorting to save memory
- Generic implementation for integer or floating-point coordinates
- Robust cross product computation to avoid overflow

### 5.2 Edge Cases

1. **Fewer than 3 points**: All points form the hull (or line segment for 2)
2. **All collinear**: Hull is a line segment between extremes
3. **Duplicate points**: Remove duplicates or handle in angle sorting
4. **All points on hull**: $h = n$, no interior points
5. **Floating-point precision**: Use epsilon for comparisons
6. **Collinear hull points**: Decision needed—include or exclude intermediate points
7. **Very large coordinates**: Risk of overflow in cross product (use wider types or normalization)

## 6. Real-World Applications

### 6.1 Software Engineering Use Cases

**1. Computer Graphics**
- Collision detection (bounding shapes)
- Mesh simplification
- Shadow boundary computation
- View frustum culling

**2. Robotics & Path Planning**
- Configuration space boundaries
- Obstacle avoidance
- Minimum enclosing polygons for robot workspace

**3. Geographic Information Systems (GIS)**
- Boundary detection of geographic regions
- Spatial indexing
- Map simplification
- Territory analysis

**4. Image Processing**
- Shape analysis and recognition
- Object boundary detection
- Feature extraction
- Pattern matching

**5. Game Development**
- Collision hulls for complex objects
- Navigation mesh generation
- Line-of-sight calculations
- Terrain analysis

**6. Data Analysis**
- Outlier detection (points far from hull)
- Clustering boundary visualization
- Anomaly detection in 2D data

### 6.2 Related Algorithms

**Variants**:
- **3D Convex Hull**: Extends to 3D space (output is polyhedron)
- **Dynamic Convex Hull**: Supports point insertion/deletion
- **Approximate Convex Hull**: Faster but non-exact
- **Convex Hull on Sphere**: For geographic/spherical coordinates

**Related Problems**:
- **Minimum Bounding Box**: Smallest rectangle containing points
- **Diameter of Point Set**: Farthest pair (can use hull)
- **Closest Pair**: Nearest two points (different approach)
- **Voronoi Diagram**: Dual of Delaunay triangulation, related to hulls

**When to Use**:
- **Graham's Scan**: General purpose, balanced performance
- **Jarvis March**: When $h << n$ (few points on hull)
- **QuickHull**: Good average case, cache-friendly
- **Chan's Algorithm**: When $h$ is unknown but small, optimal output-sensitive

## 7. References

### Academic Papers
1. Graham, R.L. (1972). "An Efficient Algorithm for Determining the Convex Hull of a Finite Planar Set". *Information Processing Letters*, 1(4), 132-133.
2. Jarvis, R.A. (1973). "On the Identification of the Convex Hull of a Finite Set of Points in the Plane". *Information Processing Letters*, 2(1), 18-21.
3. Chan, T.M. (1996). "Optimal Output-Sensitive Convex Hull Algorithms in Two and Three Dimensions". *Discrete & Computational Geometry*, 16(4), 361-368.
4. Preparata, F.P., & Hong, S.J. (1977). "Convex Hulls of Finite Sets of Points in Two and Three Dimensions". *Communications of the ACM*, 20(2), 87-93.

### Books
1. de Berg, M., et al. (2008). *Computational Geometry: Algorithms and Applications* (3rd ed.). Springer. Chapter 1: Convex Hulls.
2. Cormen, T.H., et al. (2009). *Introduction to Algorithms* (3rd ed.). MIT Press. Section 33.3: Finding the convex hull.
3. O'Rourke, J. (1998). *Computational Geometry in C* (2nd ed.). Cambridge University Press.

### Online Resources
1. [Convex Hull Algorithms - GeeksforGeeks](https://www.geeksforgeeks.org/convex-hull-set-1-jarviss-algorithm-or-wrapping/)
2. [Convex Hull Visualization](https://www.cs.usfca.edu/~galles/visualization/ConvexHull.html)
3. [Computational Geometry Course - MIT OCW](https://ocw.mit.edu/courses/electrical-engineering-and-computer-science/)

### Implementation
- Source: `src/general/convex_hull.rs`
- Tests: Included in source file under `#[cfg(test)] mod tests`
