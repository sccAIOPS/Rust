# Geometry Algorithms

This module contains implementations of fundamental computational geometry algorithms used for spatial analysis, graphics processing, and geographic information systems.

## Overview

Computational geometry is the branch of computer science devoted to the study of algorithms that can be stated in terms of geometry. It deals with problems involving points, lines, polygons, and other geometric objects.

## Algorithms in this Module

| Algorithm | Description | Time Complexity | Space Complexity |
|-----------|-------------|-----------------|------------------|
| [Graham Scan](graham_scan.md) | Convex hull using polar angle sorting | O(n log n) | O(n) |
| [Jarvis March](jarvis_scan.md) | Convex hull using gift wrapping | O(nh) | O(h) |
| [Closest Pair of Points](closest_points.md) | Find two closest points | O(n log n) | O(n) |
| [Ramer-Douglas-Peucker](ramer_douglas_peucker.md) | Polyline simplification | O(n²) worst, O(n log n) avg | O(n) |
| [Line Segment Operations](segment.md) | Segment intersection and collinearity | O(1) | O(1) |
| [Point Operations](point.md) | 2D point operations and utilities | O(1) | O(1) |
| [Polygon Points](polygon_points.md) | Polygon area and lattice points | O(n) | O(1) |

## Algorithm Categories

### Convex Hull Algorithms
Algorithms for finding the smallest convex polygon containing all given points:
- **Graham Scan**: Efficient O(n log n) algorithm using polar angle sorting
- **Jarvis March (Gift Wrapping)**: Output-sensitive O(nh) algorithm, efficient when h is small

### Distance and Proximity
- **Closest Pair of Points**: Divide-and-conquer approach to find the two points with minimum distance

### Simplification Algorithms
- **Ramer-Douglas-Peucker**: Reduces the number of points in a polyline while preserving its shape

### Fundamental Operations
- **Point**: Basic 2D point representation with distance and orientation calculations
- **Segment**: Line segment operations including intersection detection
- **Polygon Points**: Area calculation using the shoelace formula and Pick's theorem for lattice points

## Common Use Cases

1. **Computer Graphics**: Collision detection, rendering optimizations
2. **Geographic Information Systems (GIS)**: Map simplification, spatial analysis
3. **Robotics**: Path planning, obstacle avoidance
4. **Computer Vision**: Shape recognition, object detection
5. **Game Development**: Physics engines, spatial partitioning

## Mathematical Foundations

### Cross Product (2D)
The 2D cross product of vectors $\vec{a} = (a_x, a_y)$ and $\vec{b} = (b_x, b_y)$:

$$\vec{a} \times \vec{b} = a_x \cdot b_y - a_y \cdot b_x$$

This value represents:
- **Positive**: Counter-clockwise turn from $\vec{a}$ to $\vec{b}$
- **Negative**: Clockwise turn from $\vec{a}$ to $\vec{b}$
- **Zero**: Vectors are collinear

### Euclidean Distance
The distance between points $P_1 = (x_1, y_1)$ and $P_2 = (x_2, y_2)$:

$$d = \sqrt{(x_2 - x_1)^2 + (y_2 - y_1)^2}$$

## Related Modules

- [Graph Algorithms](../graph/README.md) - For graph-based geometric problems
- [Dynamic Programming](../dynamic_programming/README.md) - For optimization problems in geometry
