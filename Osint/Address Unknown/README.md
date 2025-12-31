# Address Unknown - BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | OSINT |
| **Difficulty** | Easy |
| **Points** | 200 |


## Description

> A mysterious cybersecurity blogger has been publishing sensitive reports through the website: decentsecurity.com
>
> The objective was to trace the website’s registration trail and identify the street name associated with it.

**Flag format:** `BYPASS_CTF{STREET_NAME_WITHOUT_SPACES}`

---

## TL;DR

Used domain registration (WHOIS) to identify the street name in the registrant’s address. Removed spaces to construct the flag.  

**Vulnerability:** Exposed WHOIS registrant information  
**Approach:** WHOIS lookup → extract street name → format as flag

---

## Initial Analysis

### Website Reconnaissance

- Visited the site: [https://decentsecurity.com](https://decentsecurity.com)  
- Inspected all accessible pages: Home, About, Footer  
- Checked source code and metadata  

**Observation:** No author info, address, or hidden clues were present on the website itself.

---

### Domain Registration Lookup

Performed WHOIS lookup to trace the domain registration:
whois decentsecurity.com

**Steps:**

Accessed https://www.whois.com

Entered domain: decentsecurity.com

Reviewed the registrant details, focusing on street information

Key observation: The registrant’s address included the street name required for the flag.

Flag Construction

Extracted the street name from WHOIS record: `Kalkofnsvegur 2`

Removed spaces to match flag format

## FLAG
BYPASS_CTF{Kalkofnsvegur2}