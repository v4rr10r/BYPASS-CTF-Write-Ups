# Hidden Hunter - BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | OSINT |
| **Difficulty** | Medium |
| **Points** | 300 |

## Description

> During an inspection of a colonial-era ship registry, investigators found one suspicious entry.  
> The individual listed does not appear in any official naval records, yet his description matches that of a notorious pirate long rumored to sail the seas under an assumed identity.  
>  
> You have recovered the following excerpt from the ship’s manifest:  
>  
> Rig: Ship  
> Age: 47  
> Skin: Dark  
> Hair: Brown  
> Year: 1800's  

**Flag format:** `BYPASS_CTF{Real_Name}`

---

## TL;DR

The challenge required identifying a pirate from the 1800s matching the given description. By correlating the clues with known pirate records, the correct real name was found.

**Vulnerability:** Guessable historical identity  
**Approach:** OSINT using historical pirate records

---

## Initial Analysis

### Historical Correlation

- The timeframe pointed to post–Golden Age piracy (1800s)
- Description matched several lesser-known pirates
- No direct technical artifacts were provided, indicating an OSINT-style challenge

---

## Main Analysis

### Pirate Enumeration

- Referred to the public list of known pirates:
  https://en.wikipedia.org/wiki/List_of_pirates#Post_Golden_Age:_pirates,_privateers,_smugglers,_and_river_pirates:_1730–1885
- Cross-referenced:
  - Age
  - Time period
  - General description

After testing multiple possibilities, one identity matched the challenge constraints.

---

## Flag

BYPASS_CTF{Josiah_Sparrow}

