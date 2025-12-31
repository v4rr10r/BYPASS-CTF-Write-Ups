# Whispers of the Cursed Scroll - BYPASS CTF

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Cryptography  |
| **Difficulty** | Medium |
| **Points** | 200 |

---

## Description

> An old pirate scroll has resurfaced from the depths of the sea.  
> It looks ordinary… maybe even some code.  
>  
> Legends say the message was hidden not in what is written.  
> The sea doesn’t shout its secrets.  
> Only those who read between the lines survive. ☠️⚓  
>  
> *Don't trust me I am Lier...*

A file named `They_call_me_Cutie.txt` was provided.

---

## TL;DR

The file used a **Whitespace-style binary encoding** with only three characters.  
By mapping characters to binary, splitting on delimiters, and decoding ASCII, the hidden flag was recovered.

**Approach:** Custom whitespace decoding  
**Trick:** “Read between the lines” hint

---

## Encoding Scheme
S → 0\
T → 1\
L → end of one binary character

Each sequence between `L` represents one ASCII character.

---

## Useful Python Script

```python
with open("They_call_me_Cutie.txt", "r") as f:
    data = f.read().strip()

decoded = ""
binary = ""

for c in data:
    if c == "S":
        binary += "0"
    elif c == "T":
        binary += "1"
    elif c == "L":
        if binary:
            decoded += chr(int(binary, 2))
            binary = ""

print(decoded)
```
```bash
└─$ python3 py.py 
BYPASS_CTF{Wh1tsp4c3_cut13_1t_w4s}
```
### FLAG
BYPASS_CTF{Wh1tsp4c3_cut13_1t_w4s}