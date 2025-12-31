# Signal from the Deck - BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Misc |
| **Difficulty** | Easy |
| **Points** | 100 |

---

## Description

> Something aboard the ship is trying to communicate.  
> No words. No explanations.  
> Only patterns.  
>  
> Nothing useful lives on the surface.  
> The answer waits for those who pay attention.

---

## TL;DR

The challenge is a **pattern-mapping puzzle**.  
Each tile corresponds to a specific banana number. Selecting correct pairs progresses the challenge, while wrong inputs give a humorous error. After completing the correct sequence, the flag is revealed.

**Approach:** Pattern recognition & sequence mapping  
**Trick:** Correct tile–banana pairing

---

## Analysis

- Clicking tiles with banana numbers gives feedback:
  - ✔ Correct — keep going!
  - ❌ Wrong: *“Slay banne ki kosis kr rahi h kya 💅🏻💅🏻”*
- This confirms the challenge expects a **precise mapping**
- Repeated correct inputs reveal the full sequence

---

## Correct Mapping

Tile 1 → Banana 7\
Tile 2 → Banana 9\
Tile 3 → Banana 8\
Tile 4 → Banana 5\
Tile 5 → Banana 1\
Tile 6 → Banana 2\
Tile 7 → Banana 4\
Tile 8 → Banana 6\
Tile 9 → Banana 3\
Entering this sequence correctly unlocks the flag.

---

### Flag
BYPASS_CTF{s3rv3r_s1d3_sl4y_th1ngs}