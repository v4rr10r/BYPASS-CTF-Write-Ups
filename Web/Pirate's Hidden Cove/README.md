# Pirate's Hidden Cove - BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Web |
| **Difficulty** | Easy |
| **Points** | 200|
| **Challenge Link** | http://sjvsa2qdemto3sgcf2s76fbxup5fxqkt6tmgslzgj2qsuxeafbqnicyd.onion/ |

---

## Description

> You've discovered a secret pirate cove, hidden deep within the Tor network —  
> a place where digital buccaneers stash their treasures.  
>  
> Somewhere on these sites lies the captain's flag.  
> Can you find the 📄.

---

## TL;DR

Nothing useful was found in the page source or common paths.  
The flag was hidden inside an **file placed at `/.env`**, not a real environment file.

**Trick:** Non-standard use of `.env`

---

## Initial Analysis

- Checked page source → nothing interesting
- Tried common paths:
  - `/robots.txt`
  - `/sitemap.xml`
  - `/admin`
- No visible clues or comments

This indicated the challenge relied on **path misdirection**.

---

## Discovery

Accessing `/.env` revealed it was **a text file**.

Steps:
1. Download the file
2. Flag was visible inside the content

---

## Flag

BYPASS_CTF{T0r_r0ut314}



---

pwn by **W4RR1OR**