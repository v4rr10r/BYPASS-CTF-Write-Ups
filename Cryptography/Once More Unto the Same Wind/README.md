# Once More Unto the Same Wind - BYPASS CTF

## Challenge Information

| Field | Value |
|-------|--------|
| **CTF** | BYPASS CTF |
| **Category** | Cryptography |
| **Difficulty** | Medium |
| **Points** | 200 |

## Description

> The crew of the Black Horizon believed their cipher unbreakable.  
> Captain Blackwind swore by the Galois Seal — “no blade can cut it, no storm can bend it.”  
>  
> Yet in his haste, the navigator trusted the same wind to carry more than one message.

Files Given :
```
output_RIIqaYT.txt
nn.txt
enq_EQq1fJa.py
 ```

**Flag format:** `BYPASS_CTF{.*}`

---

## TL;DR

AES-GCM was used with the same key and nonce to encrypt two different messages. One plaintext was known, allowing recovery of the keystream and decryption of the flag.

**Vulnerability:** AES-GCM nonce reuse  
**Approach:** XOR keystream recovery

---

## Initial Analysis

From `enq_EQq1fJa.py`, we observe:
- AES in GCM mode is used
- Same **key**, **nonce**, and **AAD**
- Two plaintexts encrypted:
  - `P1 = "A" * len(FLAG)` (known)
  - `P2 = FLAG` (secret)

The outputs (`output_RIIqaYT.txt`, `nn.txt`) contain the ciphertexts.

---

## Cryptographic Flaw

AES-GCM uses CTR mode internally for encryption:


If **key + nonce are reused**, the **same keystream** is generated again.

This breaks confidentiality.

---

## Exploitation

### Strategy

1. Use the known plaintext to recover the keystream
2. XOR the keystream with the second ciphertext
3. Recover the flag directly

Authentication tags do **not** protect against this misuse.

### Math
Keystream = C1 ⊕ P1\
FLAG = C2 ⊕ Keystream


---

## Exploit Code

```python
from binascii import unhexlify

C1 = unhexlify("7713283f5e9979693d337dc27b7f5575350591c530d1d4c9070607c898be0588e5cf437aef")
C2 = unhexlify("740b393f4c8b676b283447f14f534b5d071bb2e105e4f0fa19332ee8b7a027a0d4e66749d3")

P1 = b"A" * len(C1)

keystream = bytes(a ^ b for a, b in zip(C1, P1))
flag = bytes(a ^ b for a, b in zip(C2, keystream))

print(flag.decode(errors="replace"))

```
## Flag

BYPASS_CTF{rum_is_better_than_cipher}