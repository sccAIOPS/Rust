# The Algorithms - Rust

## Project Purpose
This is an **educational project** that showcases common algorithms implemented in Rust. It's part of "The Algorithms" organization (https://github.com/TheAlgorithms/) and focuses on idiomatic Rust code and generic implementations.

## Repository Information
- **Owner**: TheAlgorithms
- **Repository**: Rust
- **Branch**: master
- **Primary Language**: Rust (Edition 2021)

## Tech Stack
- **Language**: Rust (Edition 2021)
- **Build System**: Cargo
- **Testing Framework**: Built-in Rust testing + quickcheck for property-based testing
- **Linting**: Clippy with extensive configuration
- **Formatting**: rustfmt (cargo fmt)

## Dependencies
### Runtime
- `rand = "0.9"` - Random number generation
- `nalgebra = "0.34.0"` - Linear algebra
- `num-bigint` (optional, feature: big-math) - Big integer arithmetic
- `num-traits` (optional, feature: big-math) - Numeric traits

### Development
- `quickcheck = "1.0"` - Property-based testing
- `quickcheck_macros = "1.0"` - Macros for quickcheck

## Features
- `default = ["big-math"]` - By default, big-math feature is enabled
- `big-math` - Enables num-bigint and num-traits dependencies

## Algorithm Categories (21 modules)
1. **backtracking** - N-Queens, Sudoku, Hamiltonian Cycle, etc.
2. **big_integer** - Fast factorial, multiplication, Poly1305
3. **bit_manipulation** - Binary operations, two's complement, etc.
4. **ciphers** - AES, RSA, SHA256, SHA3, ChaCha, etc.
5. **compression** - Burrows-Wheeler, Run-length encoding
6. **conversions** - Number base conversions, RGB/CMYK
7. **data_structures** - Trees, Graphs, Heaps, Queues, etc.
8. **dynamic_programming** - DP algorithms
9. **financial** - Financial calculations
10. **general** - General algorithms
11. **geometry** - Geometric algorithms
12. **graph** - Graph algorithms
13. **greedy** - Greedy algorithms
14. **machine_learning** - ML algorithms
15. **math** - Mathematical algorithms
16. **navigation** - Navigation algorithms
17. **number_theory** - Number theory algorithms
18. **searching** - Search algorithms
19. **signal_analysis** - Signal processing
20. **sorting** - 30+ sorting algorithms
21. **string** - String algorithms
