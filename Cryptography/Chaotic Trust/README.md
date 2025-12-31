# Chaotic Trust - BYPASS CTF 
## Challenge Information

| Field | Value |
|-------|--------|
| **CTF** | BYPASS CTF |
| **Category** | Cryptography |
| **Difficulty** | Medium |
| **Points** | 250 |
| **Files** | chall.py, output_cphsRGh.txt |

## Description

> Chaos was used to generate a keystream for encrypting the flag using XOR.  
> A partial keystream leak was left behind during debugging.  
>  
> Chaos on computers isn’t always unpredictable.  
> Can you exploit floating-point precision and recover the flag?

**Flag format:** `BYPASS_CTF{.*}`

---

## TL;DR

A logistic map was used to generate a keystream, but part of the keystream leaked. By abusing float32 precision and brute-forcing missing bits, the full keystream can be recovered and XORed with the ciphertext to get the flag.

**Vulnerability:** Weak chaotic PRNG + keystream leak  
**Approach:** Float32 brute-force + XOR decryption

---

## Initial Analysis

From `chall.py`:
- Encryption uses **XOR stream cipher**
- Keystream generated via **logistic map**
- Each step leaks the **last 2 bytes of a float32**
- A prefix of the keystream is leaked

From the leaked keystream prefix, decrypting part of the ciphertext reveals:

BYPASS_CTF{CH40T...


So the flag structure is confirmed.

---

## Core Weakness

The logistic map uses **float32**, which has limited precision.  
Only part of the float bytes are unknown.

By brute-forcing the missing **16 bits** of the first float:
- Recompute future logistic steps
- Validate against leaked keystream bytes
- Narrow candidates to a few valid states

---

## Exploitation

### Strategy

1. Use leaked keystream bytes to validate logistic map outputs
2. Brute-force missing float32 bits
3. Generate full keystream
4. XOR keystream with ciphertext
5. Filter for printable ASCII + valid flag format

---

## Exploit Code

```python
import struct
import math
import string

def logistic_map(x, r=3.99):
    return r * x * (1 - x)

def last2_bytes(x):
    # force little-endian to match unpacking
    return struct.pack("<f", x)[-2:]

def float32_from_bytes(b0, b1, b2, b3):
    return struct.unpack("<f", bytes((b0, b1, b2, b3)))[0]

def generate_keystream_from_x1(x1, length):
    x = x1
    stream = b""
    while len(stream) < length:
        stream += last2_bytes(x)
        x = logistic_map(x)
    return stream[:length]

def is_printable_ascii(data):
    try:
        return all(c in string.printable for c in data.decode("ascii"))
    except:
        return False

cipher = bytes.fromhex(
    "9f672a7efb6ec57d0379727c360bc968"
    "c07e8b6a256acc0a850f4c608b6a9e0b"
    "5472f11f0d"
)

leak = bytes.fromhex(
    "dd3e7a3fa83d9a3e573f093f7e3ff93c"
)

for low in range(0x10000):
    x1 = float32_from_bytes(
        low & 0xFF,
        (low >> 8) & 0xFF,
        leak[0],
        leak[1]
    )

    if not math.isfinite(x1):
        continue

    ks = generate_keystream_from_x1(x1, len(cipher))
    pt = bytes(c ^ k for c, k in zip(cipher, ks))

    if (
        pt.startswith(b"BYPASS_CTF{")
        and b"}" in pt
        and is_printable_ascii(pt)
    ):
        print(pt.decode())
        break

```

## Flag

BYPASS_CTF{CH40T1C_TRU57_15_4W350M3!}