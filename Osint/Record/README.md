# Record - BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | OSINT |
| **Difficulty** | Medium |
| **Points** | 300 |

## Description

> An event without a name is a ghost.  
>  
> Bind this account to its rightful record, and in doing so, expose both the number it bears and the waters it disturbed.

**Flag format:** `BYPASS_CTF{portCity_capitalCity_incidentNumber}`

---

## TL;DR

Decoded a Morse message describing a maritime intrusion incident, matched it with an official IMB piracy report, identified the port city, governing capital city, and extracted the official IMB incident number to construct the flag.

**Vulnerability:** Publicly available maritime incident records  
**Approach:** Morse decode → piracy report correlation → IMB map lookup

---

## Initial Analysis

### Morse Code Decoding

The challenge initially provided Morse-encoded audio/text.

After decoding, the message described:
- Unauthorized persons boarding an anchored tanker
- Crew raised alarm and intruders fled
- Incident reported to VTS
- Patrol boat dispatched

This strongly indicated a **real-world maritime piracy / armed robbery incident**.

---

## OSINT Analysis

### Correlation with Official Piracy Reports

The decoded incident description exactly matched an entry in the official:

**IMB Piracy and Armed Robbery Against Ships – Jan–Sep 2025 Report**

Key details from the report:
- **Location:** Belawan Anchorage, Indonesia
- **Vessel:** Golden Curl
- **Type:** Chemical Tanker
- **IMO Number:** 9348522
- **Incident Date:** 17 January 2025

---

## Capital City Identification

Belawan is a major port but administratively falls under Medan.

Thus:
- **Port City:** Belawan
- **Capital / Administrative City:** Medan

---

## Incident Number Discovery

The IMB PDF does not list incident numbers directly.

Steps taken:
1. Visited the IMB Piracy Reporting Centre (PRC) live map  
   https://icc-ccs.org/map/
2. Filtered using:
   - Date: 17/01/2025
   - Location: Belawan Anchorage
   - Vessel IMO: 9348522
3. Matched the incident to its official identifier.

**IMB Incident Number:** `011-25`

(Admin hint confirmed IMB format with hyphen.)

---

## Flag
BYPASS_CTF{Belawan_Medan_011-25}
