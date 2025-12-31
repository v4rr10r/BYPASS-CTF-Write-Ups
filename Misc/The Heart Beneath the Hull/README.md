# The Heart Beneath the Hull - BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Misc|
| **Difficulty** | Easy |
| **Points** | 50 |

---

## Description

> Not all treasures are buried in sand.

A yo.PNG file was provided as part of the challenge.

---

## TL;DR

The flag was hidden as **hexadecimal values** visibly embedded in an image.  
Decoding the hex values directly revealed the flag.

**Approach:** Hex decoding  
**Trick:** Data hidden in plain sight inside an image

---

## Analysis

Upon opening the provided PNG file, the following sequence of numbers was visible:

68 65 61 72 74 5f 69 6e 5f 61 5f 63 68 65 73 68 65 61 72 74 5f 69 6e 5f 61 5f 63 68 65 74

rust
Copy code

These values resemble **hexadecimal ASCII codes**.

---

## Decoding

Converting the hex values to ASCII:

```text
68 -> h
65 -> e
61 -> a
72 -> r
74 -> t
5f -> _
69 -> i
6e -> n
5f -> _
61 -> a
5f -> _
63 -> c
68 -> h
65 -> e
73 -> s
68 -> h
65 -> e
61 -> a
72 -> r
74 -> t
5f -> _
69 -> i
6e -> _
61 -> a
5f -> _
63 -> c
68 -> h
65 -> e
74 -> t
Decoded string:

nginx
Copy code
heart_in_a_chestheart_in_a_chet
```
### Flag

BYPASS_CTF{heart_in_a_chest}