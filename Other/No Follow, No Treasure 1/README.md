# No Follow, No Treasure 1 - BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Other |
| **Difficulty** | Easy |
| **Points** | 100 |
| **Author** | Rohit Yadav |

**Tags:** Beginner  
**Flag format:** `BYPASS_CTF{.*}`

---

## Description

> Access denied.  
>  
> Reason: No social footprint detected.  
>  
> Authenticate yourself at the official Bypass CTF page  
> and leave a visible trace.  
>  
> Flag unlocks post-verification.  
>  
> Just look around the ship; sometimes the pirates carry the flag.

---

## TL;DR

The challenge required exploring the **Bypass CTF Discord server**.  
Different parts of the flag were hidden across Discord channels, emojis, and a moderator’s banner.

**Category:** Discord OSINT  
**Approach:** Manual inspection of Discord server elements

---

## Solution

The flag was split across multiple locations inside the official **ByPASS CTF Discord server**:

### Part 1 – Browse Channel
- One fragment of the flag was visible in the **browse channel**
- This hinted that the challenge required active participation on Discord

### Part 2 – Channel Emoji
- Another fragment was hidden inside a **custom emoji**
- Inspecting emoji names revealed meaningful text

### Part 3 – Moderator Banner Deva
- The final fragment was found in the **banner of a Discord moderator**
- Viewing the profile banner completed the flag

Combining all parts revealed the full flag.

---

## Flag

BYPASS_CTF{w3lc0m3_t00_byp4ss_ctf}


---

pwn by **W4RR1OR**