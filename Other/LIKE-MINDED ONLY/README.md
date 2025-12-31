# LIKE-MINDED ONLY - BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Other |
| **Difficulty** | Easy |
| **Points** | 200 |
| **Challenge Link** | https://chatgpt.com/share/694fb36b-aa0c-8013-b10a-498a5848f4ef |

## Description

> Some links aren’t broken.  
> They’re just meant for the right people with the right skills.  
>  
> Follow the name.  
> Find the signal.

**Flag format:** `BYPASS_CTF{.*}`

---

## TL;DR

A seemingly broken ChatGPT shared link was actually slightly malformed. Fixing the link revealed a discussion pointing to a YouTube channel, which led to a Telegram group where the flag was hidden in the discussion bio.

**Vulnerability:** Obfuscated OSINT trail  
**Approach:** Link fuzzing → OSINT pivoting

---

## Initial Analysis

### Link Inspection

- Opening the provided ChatGPT share link resulted in an error
- The challenge hint suggested the link was not truly broken
- Began fuzzing the URL manually

---

## Main Analysis

### Step 1: ChatGPT Link Fuzzing

- Modified the last character of the link
- Changing the final `f` to `e` opened a valid ChatGPT conversation
- The conversation discussed a specific **YouTube channel**

---

### Step 2: YouTube OSINT Pivot

- Visited the referenced YouTube channel
- No visible clues in videos or descriptions
- Found a **Telegram link** associated with the channel

---

### Step 3: Telegram Investigation

- Joined the Telegram group
- Checked:
  - Channel info
  - Discussion section
  - Bio / description

The flag was directly present in the discussion bio.

---

## Flag

BYPASS_CTF{w3lc0m3_t0_th3_cyb3rfy_c0mmun1ty}

