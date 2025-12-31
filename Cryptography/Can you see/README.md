# Can you see?? — BYPASS CTF

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Cryptography |
| **Difficulty** | Medium |
| **Points** | 100 |
| **File** | dcode-image.png |

## Challenge Description

> “The sea keeps many secrets, but pirates had their own way of hiding the truth.”

**Flag format:** `BYPASS_CTF{.*}`

---

## Solution

1. **Observation:**  
   - The image contains stylized letters that appear scrambled or mirrored.
   - Hint mentions pirates hiding secrets.

2. **Decryption Approach:**  
   - Rotate the image **180°**.  
   - Read the letters **right-to-left**.  
   - Stylized characters resolve into readable text.

3. **Result:**  
   - The hidden message reads:  
     ```
     DEAD MEN TELL NO TALES
     ```

---

## Flag

BYPASS_CTF{DEAD_MEN_TELL_NO_TALES}

**Explanation:**  

- The message was drawn upside-down and mirrored.  
- Rotating the image and reading backwards revealed the correct pirate phrase.  
- Fits perfectly with the theme of secrets at sea.
