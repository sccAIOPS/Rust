# Big Integer Algorithms

## Overview

This category contains algorithms for performing operations on arbitrarily large integers that exceed the capacity of native integer types. These implementations are essential for cryptography, number theory, symbolic mathematics, and any application requiring exact arithmetic with very large numbers.

Big integer algorithms bridge the gap between theoretical mathematics and practical computation, enabling operations on numbers with thousands or even millions of digits. While modern languages often provide big integer libraries (like Rust's `num-bigint`), understanding the underlying algorithms is crucial for optimization, security analysis, and specialized implementations.

## Algorithms in This Category

| Algorithm | File | Complexity | Priority | Use Case |
|-----------|------|------------|----------|----------|
| [Big Integer Multiplication](#multiplication) | `multiply.rs` | O(m × n) | High | Foundation for arbitrary-precision arithmetic |
| [Fast Factorial](#fast-factorial) | `fast_factorial.rs` | O(log log n · M(n log n)) | High | Computing large factorials efficiently |
| [Poly1305 MAC](#poly1305) | `poly1305.rs` | O(n) | Medium | Cryptographic message authentication |

## Algorithm Summaries

### <a name="multiplication"></a> Big Integer Multiplication

**Purpose**: Multiply two arbitrarily large non-negative integers represented as strings.

**Key Concepts**:
- Grade school multiplication algorithm extended to unlimited digits
- Processes digit pairs with immediate carry propagation
- Fundamental building block for all big integer arithmetic

**Complexity**: O(m × n) where m and n are the number of digits

**When to Use**:
- Numbers < 100-1000 digits (simple, low overhead)
- Teaching fundamental multiplication algorithm
- Building blocks for more complex operations

**Real-World Applications**:
- RSA cryptography (multiplying large primes)
- Financial systems (arbitrary-precision currency)
- Computer algebra systems (symbolic mathematics)

**Related Algorithms**: Karatsuba multiplication (O(n^1.585)), Toom-Cook multiplication (O(n^1.465)), FFT-based multiplication (O(n log n))

📄 [Full Documentation](multiply.md)

---

### <a name="fast-factorial"></a> Fast Factorial (Borwein's Algorithm)

**Purpose**: Compute factorials of large numbers much faster than naive iteration.

**Key Concepts**:
- Uses prime factorization and Legendre's formula
- Groups primes by binary representation of their exponents
- Reduces O(n) multiplications to O(log n) exponentiations

**Complexity**: O(log log n · M(n log n)) where M(n) is big integer multiplication complexity

**When to Use**:
- Large factorials (n > 1000)
- Computing binomial coefficients
- Number theory research
- Benchmarking big integer libraries

**Real-World Applications**:
- Combinatorics and probability calculations
- Symbolic mathematics (Taylor series, special functions)
- Scientific computing (Stirling's approximation verification)
- Algorithm benchmarking

**Related Algorithms**: Naive factorial, divide-and-conquer factorial, prime swing factorial (even faster)

📄 [Full Documentation](fast_factorial.md)

---

### <a name="poly1305"></a> Poly1305 Message Authentication Code

**Purpose**: Fast, secure cryptographic message authentication using polynomial evaluation.

**Key Concepts**:
- Polynomial evaluation in prime field p = 2^130 - 5
- One-time key authentication (each key authenticates one message)
- Designed by Daniel J. Bernstein (2005)
- Widely deployed in TLS 1.3, SSH, WireGuard

**Complexity**: O(n) where n is message length, ~3-4 CPU cycles per byte (optimized implementations)

**When to Use**:
- ChaCha20-Poly1305 AEAD (authenticated encryption)
- High-speed message authentication
- Systems without AES hardware acceleration
- Embedded systems and IoT devices

**Real-World Applications**:
- TLS 1.3 cipher suites
- OpenSSH encryption
- WireGuard VPN
- Signal Protocol messaging
- QUIC/HTTP3 protocol

**Security Note**: This implementation uses `num-bigint` which is **not constant-time**. Production cryptographic code requires constant-time implementations to prevent timing attacks.

**Related Algorithms**: HMAC-SHA256, AES-GCM, GHASH, BLAKE3-keyed MAC

📄 [Full Documentation](poly1305.md)

## Common Patterns and Techniques

### 1. String-Based Arithmetic
Many big integer algorithms work with string representations:
- **Input**: Numbers as strings of decimal digits
- **Processing**: Character-by-character operations
- **Output**: Result as string
- **Advantage**: Language-independent, no integer overflow

### 2. Prime Factorization
Several algorithms exploit prime structure:
- **Sieve of Eratosthenes**: Generate primes efficiently
- **Legendre's Formula**: Count prime factors in factorials
- **Chinese Remainder Theorem**: Solve modular equations
- **Applications**: Cryptography, number theory, optimization

### 3. Modular Arithmetic
Working in finite fields and modular systems:
- **Modular exponentiation**: Fast power computation mod p
- **Montgomery multiplication**: Efficient modular reduction
- **Prime field arithmetic**: Cryptographic operations
- **Applications**: RSA, elliptic curves, MACs

### 4. Binary Representation Tricks
Exploiting binary structure for efficiency:
- **Binary exponentiation**: O(log n) exponentiation
- **Bit grouping**: Reduce operations by binary alignment
- **Fast doubling**: Leverage powers of 2
- **Applications**: Fast factorial, modular exponentiation

## Performance Considerations

### Asymptotic Complexity vs. Practical Performance

Big integer algorithms have a hierarchy of complexity:

```
Operation: Multiplication
├── Grade School: O(n²)          [this repository: multiply.rs]
│   └── Best for: n < 100-1000 digits
├── Karatsuba: O(n^1.585)
│   └── Best for: 1,000-10,000 digits
├── Toom-Cook: O(n^1.465)
│   └── Best for: 10,000-100,000 digits
└── FFT-based: O(n log n)
    └── Best for: > 100,000 digits
```

**Key Insight**: Simpler algorithms often win for practical sizes due to lower constant factors and overhead.

### Memory vs. Speed Trade-offs

| Approach | Memory | Speed | Use Case |
|----------|--------|-------|----------|
| Iterative | O(1) aux | Slow | Small results |
| Recursive | O(log n) stack | Medium | Divide-and-conquer |
| Table lookup | O(n) space | Fast | Repeated access |
| Streaming | O(1) aux | Medium | Large inputs |

### Optimization Strategies

1. **Pre-allocation**: Allocate exact result size upfront
2. **Bit operations**: Use bitwise ops instead of arithmetic when possible
3. **Vectorization**: SIMD for parallel digit processing
4. **Constant-time**: Critical for cryptographic applications
5. **Specialized instructions**: Leverage CPU extensions (PCLMULQDQ, AES-NI)

## Security Considerations

When implementing cryptographic algorithms with big integers:

### ⚠️ Timing Attacks
- **Problem**: Operations that take variable time based on secret data
- **Solution**: Use constant-time implementations
- **Example**: This repository's Poly1305 uses `num-bigint` (NOT constant-time)

### ⚠️ Side-Channel Attacks
- **Cache timing**: Memory access patterns leak information
- **Power analysis**: Power consumption reveals computation
- **Electromagnetic**: EM radiation during computation
- **Mitigation**: Constant-time algorithms, masking, hardware countermeasures

### ⚠️ Key Reuse
- **Poly1305**: Each (r, s) key pair must be used only once
- **One-time pads**: Never reuse cryptographic keys
- **Best practice**: Use key derivation functions (KDF) with nonces

### ✅ Production Cryptography Checklist
- [ ] Use audited libraries (libsodium, RustCrypto)
- [ ] Ensure constant-time operations
- [ ] Validate all inputs
- [ ] Use secure random number generation
- [ ] Follow current standards (NIST, IETF RFCs)
- [ ] Regular security audits

## Implementation Quality Indicators

### This Repository's Implementations

| Algorithm | Code Quality | Documentation | Tests | Production-Ready |
|-----------|--------------|---------------|-------|------------------|
| Multiply | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ | ✅ (Non-crypto) |
| Fast Factorial | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ✅ (Non-crypto) |
| Poly1305 | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⚠️ (Educational only) |

**Note**: Poly1305 implementation is **educational only** and should not be used in production due to non-constant-time operations.

## Learning Path

### Beginner
1. Start with [Big Integer Multiplication](multiply.md)
   - Understand digit-by-digit processing
   - Master carry propagation
   - Learn string-to-number conversions

2. Explore modular arithmetic basics
   - Modular addition and multiplication
   - Euclidean algorithm for GCD
   - Extended Euclidean algorithm

### Intermediate
3. Study [Fast Factorial](fast_factorial.md)
   - Prime factorization techniques
   - Legendre's formula
   - Binary grouping optimization

4. Learn advanced multiplication
   - Karatsuba algorithm
   - Toom-Cook algorithm
   - FFT-based methods

### Advanced
5. Cryptographic applications with [Poly1305](poly1305.md)
   - Polynomial evaluation in finite fields
   - Information-theoretic security
   - Constant-time implementation techniques

6. Specialized topics
   - Montgomery multiplication
   - Elliptic curve arithmetic
   - Side-channel resistance

## Dependencies and Tools

### This Repository Uses:
- **num-bigint**: Arbitrary-precision integers for Rust
- **num-traits**: Numeric traits (One, Zero, etc.)
- **std::collections::BTreeMap**: Ordered mappings

### Recommended External Libraries:
- **RustCrypto**: Production cryptography ([https://github.com/RustCrypto](https://github.com/RustCrypto))
- **rug**: Rust bindings to GMP ([https://crates.io/crates/rug](https://crates.io/crates/rug))
- **libsodium**: Modern crypto library ([https://libsodium.org](https://libsodium.org))

### Testing and Benchmarking:
- `cargo test`: Run unit tests
- `cargo bench`: Performance benchmarks
- `cargo clippy`: Linting
- `cargo fmt`: Code formatting

## References and Further Reading

### Books
1. **Knuth, Donald E.** *The Art of Computer Programming, Volume 2: Seminumerical Algorithms*
   - The definitive reference for arithmetic algorithms

2. **Crandall, Richard; Pomerance, Carl.** *Prime Numbers: A Computational Perspective*
   - Advanced number theory and algorithms

3. **Menezes, van Oorschot, Vanstone.** *Handbook of Applied Cryptography*
   - [Free PDF](http://cacr.uwaterloo.ca/hac/)
   - Chapter 14: Efficient Implementation

### Standards
- **RFC 8439**: ChaCha20 and Poly1305 for IETF Protocols
- **FIPS 186-4**: Digital Signature Standard (DSS)
- **NIST SP 800-90A**: Random Number Generation

### Online Resources
- [Rosetta Code: Arbitrary-precision integers](https://rosettacode.org/wiki/Arbitrary-precision_integers)
- [Crypto++ Library Documentation](https://www.cryptopp.com/)
- [GMP Documentation](https://gmplib.org/)

## Contributing

When adding new big integer algorithms to this repository:

1. **Implement the core algorithm** in `src/big_integer/`
2. **Add comprehensive tests** including edge cases
3. **Create detailed documentation** using the template in [PLAN.md](../PLAN.md)
4. **Benchmark** against naive implementations
5. **Document security considerations** for cryptographic algorithms
6. **Add references** to original papers and standards

### Coding Standards
- Follow Rust naming conventions (snake_case)
- Run `cargo fmt` and `cargo clippy`
- Include complexity analysis in comments
- Write clear, self-documenting code

---

## Quick Reference

| Need to... | Use... | Complexity |
|-----------|--------|------------|
| Multiply two large numbers | [multiply.rs](multiply.md) | O(m × n) |
| Compute n! for large n | [fast_factorial.rs](fast_factorial.md) | O(log log n · M(n log n)) |
| Authenticate message (crypto) | [poly1305.rs](poly1305.md) | O(n) |
| Modular exponentiation | See math/modular_exponential.rs | O(log n · M(log n)) |
| GCD of large numbers | See math/gcd.rs | O(log min(a,b) · M(log max(a,b))) |

## Related Categories

- **[Math](../math/README.md)**: Modular arithmetic, GCD, prime numbers
- **[Ciphers](../ciphers/README.md)**: RSA, AES, cryptographic primitives
- **[Number Theory](../number_theory/README.md)**: Euler's totient, Chinese remainder theorem
- **[String](../string/README.md)**: String-based number conversions

---

**Last Updated**: January 2026  
**Maintained By**: TheAlgorithms/Rust Contributors  
**License**: MIT
