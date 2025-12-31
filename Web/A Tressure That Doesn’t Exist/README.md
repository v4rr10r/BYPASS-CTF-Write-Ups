# A Treasure That Doesn’t Exist - BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Web |
| **Difficulty** | Hard |
| **Points** | 350 |
| **Challenge Link** | https://tressurefound.vercel.app/ |

---

## Description

> The page insists it's not here — a digital dead end.  
> Yet, something about this absence feels… intentional.  
>  
> Pages may lie, but the browser doesn’t.

---

## TL;DR

The website contains no useful source code or common hidden paths.  
The flag was hidden inside the website’s `favicon.ico` file and revealed using `strings`.

**Approach:** Static file inspection  
**Trick:** Hidden data inside favicon

---

## Initial Analysis

- Page shows nothing useful
- No hints in HTML source
- No secrets in common paths:
  - `/robots.txt`
  - `/.env`
  - `/sitemap.xml`

This suggested the challenge was based on **misdirection**.

---

## Discovery

Browsers always request `/favicon.ico`.  
Manually visiting the file revealed a downloadable icon.

```bash
strings favicon.ico
Output:

yq[+
flag.txtUT
BYPASS_CTF{404_Err0r_N0t_F0und_v}
yq[+
flag.txtUT

The flag was embedded directly inside the favicon file.
```
### Flag

BYPASS_CTF{404_Err0r_N0t_F0und_v}

pwn by **W4RR1OR**