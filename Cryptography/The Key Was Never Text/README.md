# The Key Was Never Text - BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|--------|
| **CTF** | BYPASS CTF |
| **Category** | Cryptography |
| **Difficulty** | Medium |
| **Points** | 250 |

## Description

> He never trusted digital things.  
> His favorite key was something with hands but no voice.  
>  
> “The face tells you everything if you know how to read it.”  
>  
> You recovered this message:  
>  
> `18 5 25 11 10 1 22 9 11 9 3 5 12 1 14 4`

**Flag Format:** `BYPASS_CTF{ANSWER_IN_UPPERCASE_NO_SPACES}`

---

## TL;DR

The numbers map directly to letters using the classic **A1Z26** cipher (A=1, B=2, … Z=26). Decoding the sequence reveals the hidden message.

**Vulnerability:** Simple numeric substitution  
**Approach:** A1Z26 decoding

---

## Initial Analysis

Key hints:
- “Something with hands but no voice” → **clock/watch**
- “The face tells you everything” → **numbers on a clock / alphabet positions**

This strongly hints at **alphabet indexing**.

---

## Decoding

Using **A1Z26** mapping:

18 = R
5 = E
25 = Y
11 = K
10 = J
1 = A
22 = V
9 = I
11 = K
9 = I
3 = C
5 = E
12 = L
1 = A
14 = N
4 = D

Decoded text:  REYKJAVIKICELAND


This spells **REYKJAVIK ICELAND** (the clock-face hint refers to geography and time).

Removing spaces and converting to uppercase:

---

## Flag

BYPASS_CTF{REYKJAVIKICELAND}

