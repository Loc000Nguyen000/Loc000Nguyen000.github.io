---
title: "Sample Crypto Challenge - Classic RSA"
date: 2024-01-20T14:30:00+07:00
draft: false
description: "RSA challenge with small public exponent vulnerability"
categories: ["Crypto"]
tags: ["RSA", "Low Exponent Attack", "Python", "Number Theory"]
ctfs: ["Example CTF 2024"]
author: "Zicc0Nguyen"
ShowToc: true
TocOpen: true
---

## Challenge Information

- **CTF:** Example CTF 2024
- **Category:** Cryptography
- **Difficulty:** Medium
- **Points:** 250
- **Solves:** 89

## Description

You intercepted an RSA encrypted message. Can you decrypt it?

**Challenge Files:**
- `public_key.pem`
- `encrypted_flag.txt`

## Reconnaissance

Let's examine what we have:

```bash
$ cat public_key.pem
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC...
-----END PUBLIC KEY-----

$ cat encrypted_flag.txt
67c8a3e02f0a0e5d3c8f9b2a1e4d5c6b7a8f9e0d1c2b3a4f5e6d7c8b9a0e1d2c3
```

First, let's extract the RSA parameters from the public key:

```python
from Crypto.PublicKey import RSA

with open('public_key.pem', 'r') as f:
    key = RSA.import_key(f.read())

n = key.n  # Modulus
e = key.e  # Public exponent

print(f"n = {n}")
print(f"e = {e}")
print(f"n bit length = {n.bit_length()}")
```

Output:
```
n = 1234567890123456789...
e = 3
n bit length = 1024
```

## Vulnerability Analysis

The public exponent `e = 3` is very small. This makes the encryption vulnerable to several attacks:

1. **Low Exponent Attack**: If the plaintext is small enough that `m^3 < n`, we can simply take the cube root
2. **Håstad's Broadcast Attack**: If the same message is encrypted with different moduli but same exponent

Let's try the simple cube root attack first:

```python
import gmpy2

# Read the ciphertext
with open('encrypted_flag.txt', 'r') as f:
    c = int(f.read().strip(), 16)

# If m^3 < n, then c = m^3 (in integers, not modulo n)
# We can just take the cube root
m, exact = gmpy2.iroot(c, 3)

if exact:
    print("[+] Found exact cube root!")
    print(f"Plaintext (hex): {hex(int(m))}")
    print(f"Plaintext (bytes): {bytes.fromhex(hex(int(m))[2:])}")
else:
    print("[-] Not a perfect cube, trying other methods...")
```

## Exploitation

### Method 1: Direct Cube Root

Since `e = 3` and the message is small, we can directly compute the cube root:

```python
#!/usr/bin/env python3
import gmpy2
from Crypto.PublicKey import RSA

# Load public key
with open('public_key.pem', 'r') as f:
    key = RSA.import_key(f.read())
    n = key.n
    e = key.e

# Load ciphertext
with open('encrypted_flag.txt', 'r') as f:
    c = int(f.read().strip(), 16)

# Try direct cube root
m, exact = gmpy2.iroot(c, e)

if exact:
    # Convert to bytes
    plaintext = bytes.fromhex(hex(int(m))[2:])
    print(f"[+] Flag: {plaintext.decode()}")
else:
    # If not exact, the message might wrap around modulo n
    # Try adding n until we get a perfect cube
    for k in range(100):
        test_c = c + k * n
        m, exact = gmpy2.iroot(test_c, e)
        if exact:
            plaintext = bytes.fromhex(hex(int(m))[2:])
            print(f"[+] Flag found with k={k}: {plaintext.decode()}")
            break
```

Running the exploit:

```bash
$ python3 solve.py
[+] Flag: flag{l0w_3xp0n3nt_4tt4ck_ftw}
```

### Method 2: Using SageMath

Alternatively, we could use SageMath for more complex attacks:

```python
# sage
n = 1234567890123456789...
e = 3
c = 67c8a3e02f0a0e5d3c8f9b2a1e4d5c6b7a8f9e0d1c2b3a4f5e6d7c8b9a0e1d2c3

# Simple cube root
m = c.nth_root(e)
print(bytes.fromhex(hex(int(m))[2:]))
```

## Flag

```
flag{l0w_3xp0n3nt_4tt4ck_ftw}
```

## Key Takeaways

1. **Never use small public exponents** without proper padding (use OAEP)
2. **Small e is dangerous**: e=3 or e=65537 should always be used with padding schemes
3. **Cube root attacks**: When m^e < n, encryption is reversible without factoring
4. **Use modern standards**: Always use RSA-OAEP instead of textbook RSA

## Tools Used

- Python 3
- PyCryptodome library
- gmpy2 for fast integer arithmetic
- SageMath (alternative solution)

## References

- [RSA (cryptosystem) - Wikipedia](https://en.wikipedia.org/wiki/RSA_(cryptosystem))
- [Twenty Years of Attacks on the RSA Cryptosystem](https://crypto.stanford.edu/~dabo/pubs/papers/RSA-survey.pdf)
- [gmpy2 Documentation](https://gmpy2.readthedocs.io/)
