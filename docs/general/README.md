# General Algorithms

This category contains miscellaneous algorithms that don't fit neatly into other categories. These are fundamental algorithms used across various domains in computer science and software engineering.

## Contents

### High Priority Algorithms

1. **[Huffman Encoding](huffman_encoding.md)** - Optimal prefix-free encoding for data compression
2. **[Convex Hull](convex_hull.md)** - Finding the smallest convex boundary containing a set of points
3. **[Fisher-Yates Shuffle](fisher_yates_shuffle.md)** - Unbiased random permutation generation
4. **[Kadane's Algorithm](kadane_algorithm.md)** - Maximum subarray sum in linear time
5. **[Two Sum](two_sum.md)** - Finding pairs that sum to a target value
6. **[K-Means Clustering](kmeans.md)** - Unsupervised clustering algorithm

### Medium Priority Algorithms

7. **[Genetic Algorithm](genetic.md)** - Evolutionary optimization technique
8. **[Tower of Hanoi](hanoi.md)** - Classic recursive puzzle
9. **[Heap Permutation](heap_permutation.md)** - Efficient permutation generation
10. **[Steinhaus-Johnson-Trotter](steinhaus_johnson_trotter.md)** - Generating permutations with minimal changes

### Low Priority Algorithms

11. **[MEX (Minimum Excludant)](mex.md)** - Finding smallest non-negative integer not in set

## Overview

General algorithms encompass a diverse set of computational techniques that solve common problems across multiple domains:

- **Data Compression**: Huffman encoding for optimal lossless compression
- **Computational Geometry**: Convex hull for spatial analysis
- **Randomization**: Fisher-Yates for unbiased shuffling
- **Optimization**: Dynamic programming (Kadane's), evolutionary algorithms (genetic)
- **Search Problems**: Two Sum and related techniques
- **Machine Learning**: K-Means clustering
- **Combinatorics**: Permutation generation algorithms
- **Classic Problems**: Tower of Hanoi, game theory (MEX)

## Common Patterns

### Greedy Approaches
- Huffman encoding builds optimal trees greedily
- Kadane's algorithm makes locally optimal choices

### Divide and Conquer
- Convex hull algorithms (Graham scan, Jarvis march)
- Efficient two sum variants

### Dynamic Programming
- Kadane's algorithm for maximum subarray
- Tower of Hanoi with memoization

### Randomization
- Fisher-Yates shuffle
- Genetic algorithm selection

### Iterative Optimization
- K-Means clustering
- Genetic algorithm evolution

## Applications

These algorithms find applications in:

- **Data Compression**: File compression, network protocols
- **Graphics & Gaming**: Convex hulls, random generation, shuffling
- **Data Analysis**: Clustering, pattern recognition
- **Optimization Problems**: Scheduling, resource allocation
- **Algorithm Design**: Teaching fundamental concepts
- **Competitive Programming**: Two Sum, MEX for game theory

## Implementation Notes

All algorithms in this category follow TheAlgorithms/Rust standards:
- Generic implementations where applicable
- Comprehensive test coverage
- Edge case handling (empty inputs, single elements)
- Performance optimization for production use
- Clear documentation and examples
