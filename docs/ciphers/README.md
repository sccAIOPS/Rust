# Ciphers & Cryptography

This directory contains comprehensive documentation for all cryptographic algorithm implementations in TheAlgorithms/Rust.

## Overview

Cryptography is the practice of securing information through encoding techniques. This module covers a wide range of cryptographic algorithms, from classical ciphers used historically to modern encryption standards used in production systems.

## Algorithm Categories

### Modern Symmetric Encryption
| Algorithm | File | Description | Security Level |
|-----------|------|-------------|----------------|
| [AES](aes.md) | `aes.rs` | Advanced Encryption Standard (128/192/256-bit) | Production |
| [ChaCha20](chacha.md) | `chacha.rs` | Stream cipher by Daniel J. Bernstein | Production |
| [Salsa20](salsa.md) | `salsa.rs` | Stream cipher, predecessor to ChaCha | Production |
| [TEA](tea.md) | `tea.rs` | Tiny Encryption Algorithm | Educational |

### Asymmetric Encryption & Key Exchange
| Algorithm | File | Description | Security Level |
|-----------|------|-------------|----------------|
| [RSA](rsa.md) | `rsa_cipher.rs` | Rivest-Shamir-Adleman public-key cryptosystem | Educational |
| [Diffie-Hellman](diffie_hellman.md) | `diffie_hellman.rs` | Key exchange protocol | Production |

### Cryptographic Hash Functions
| Algorithm | File | Description | Output Size |
|-----------|------|-------------|-------------|
| [SHA-256](sha256.md) | `sha256.rs` | SHA-2 family, 256-bit digest | 256 bits |
| [SHA-3](sha3.md) | `sha3.rs` | Keccak-based hash (224/256/384/512) | Variable |
| [Blake2b](blake2b.md) | `blake2b.rs` | Modern hash function | Up to 512 bits |

### Classical Substitution Ciphers
| Algorithm | File | Description | Era |
|-----------|------|-------------|-----|
| [Caesar Cipher](caesar.md) | `caesar.rs` | Simple letter shifting | Ancient Rome |
| [ROT13](rot13.md) | `rot13.rs` | Special case of Caesar (shift=13) | Modern |
| [Vigenère Cipher](vigenere.md) | `vigenere.rs` | Polyalphabetic substitution | 16th Century |
| [Baconian Cipher](baconian.md) | `baconian_cipher.rs` | Binary encoding steganography | 1605 |
| [Polybius Square](polybius.md) | `polybius.rs` | Grid-based substitution | Ancient Greece |

### Classical Transposition Ciphers
| Algorithm | File | Description | Era |
|-----------|------|-------------|-----|
| [Transposition](transposition.md) | `transposition.rs` | Columnar transposition | Historical |
| [Rail Fence](rail_fence.md) | `rail_fence.rs` | Zigzag pattern cipher | Civil War Era |

### Encoding Schemes
| Algorithm | File | Description | Use Case |
|-----------|------|-------------|----------|
| [Base64](base64.md) | `base64.rs` | Binary-to-text encoding | Data transport |
| [Morse Code](morse_code.md) | `morse_code.rs` | Telecommunication encoding | Communication |
| [XOR Cipher](xor.md) | `xor.rs` | Bitwise exclusive-or | Simple encryption |

## Complexity Summary

| Algorithm | Encrypt Time | Decrypt Time | Space | Key Size |
|-----------|-------------|--------------|-------|----------|
| AES-128 | O(n) | O(n) | O(1) | 128 bits |
| AES-256 | O(n) | O(n) | O(1) | 256 bits |
| RSA | O(k³) | O(k³) | O(k) | 1024-4096 bits |
| SHA-256 | O(n) | N/A | O(1) | N/A |
| ChaCha20 | O(n) | O(n) | O(1) | 256 bits |
| Caesar | O(n) | O(n) | O(1) | 1-25 |
| Vigenère | O(n) | O(n) | O(k) | Variable |

Where:
- `n` = message length
- `k` = key size (bits or characters)

## Security Classification

### ⚠️ Educational Only (NOT for production)
These implementations are for learning purposes and should **never** be used for real security:
- RSA (small key sizes, no padding)
- Classical ciphers (Caesar, Vigenère, etc.)
- TEA (known vulnerabilities)

### ✅ Algorithm Design is Production-Ready
The algorithm design is sound, but always use established libraries for production:
- AES
- ChaCha20/Salsa20
- SHA-256/SHA-3
- Blake2b
- Diffie-Hellman

## Learning Path

### Beginner
1. [Caesar Cipher](caesar.md) - Understand basic substitution
2. [XOR Cipher](xor.md) - Learn bitwise operations
3. [Base64](base64.md) - Understand encoding vs encryption

### Intermediate
4. [Vigenère Cipher](vigenere.md) - Polyalphabetic concepts
5. [Transposition](transposition.md) - Permutation-based encryption
6. [TEA](tea.md) - Introduction to block ciphers

### Advanced
7. [AES](aes.md) - Modern block cipher standard
8. [SHA-256](sha256.md) - Cryptographic hash functions
9. [RSA](rsa.md) - Public-key cryptography basics
10. [Diffie-Hellman](diffie_hellman.md) - Key exchange protocols
11. [ChaCha20](chacha.md) - Modern stream ciphers

## References

- NIST Cryptographic Standards: https://csrc.nist.gov/publications/fips
- RFC 8439 (ChaCha20-Poly1305): https://tools.ietf.org/html/rfc8439
- RFC 6234 (SHA-256): https://tools.ietf.org/html/rfc6234
- RFC 7693 (Blake2): https://tools.ietf.org/html/rfc7693
- FIPS 197 (AES): https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197.pdf
