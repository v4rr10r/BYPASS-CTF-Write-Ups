# Pelagic Node-14 - BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | OSINT |
| **Difficulty** | Medium |
| **Points** | 300 |


## Description

> A submerged visual relay—designated Pelagic Node-14—has been continuously transmitting a live feed from an uncharted mid-ocean depth.  
>  
> The camera is fixed, silent, pressure-worn, and linked to a small YouTube livestream that rarely registers more than a handful of viewers.  
>  
> No logs accompany it.  
> No metadata survives the stream.  
>  
> Only a single captured frame has been archived for assessment.  
>  
> The frame contains nothing unusual at first glance—only drifting particulates and the muted gradients of deep water.  
>  
> Yet the relay itself operates under an internal classification, a descriptor assigned long before the livestream was ever configured, and long after its manufacturer vanished from public record.  
>  
> Your objective is simply to determine the descriptor traditionally used for this class of deep-water relay.

**Files provided:**
- `pelagic.jpg` – Captured frame from the submerged relay

**Flag format:** `BYPASS_CTF{city_state_country}`

---

## TL;DR

Reverse image search on the provided image revealed the original livestream source. The camera location was identified and formatted to obtain the flag.

**Vulnerability:** Publicly indexed image source  
**Approach:** Reverse image search → identify location → format flag

---

## Initial Analysis

### Image Identification

The challenge provided only a single image file with no metadata.

Steps taken:
- Performed reverse image search using multiple engines
- Looked for visually identical aquarium feeds
- Matched the image with a known public livestream

**Key observations:**
- Image matches Aquarium of the Pacific livestream footage
- Identical color gradients and coral layout confirmed the source

---

## OSINT Analysis

### Source & Location Discovery

Once the livestream source was identified:

`https://explore.org/livecams/aquarium-of-the-pacific/pacific-aquarium-tropical-reef-habitat-cam`
- Source: Aquarium of the Pacific
- Verified via official livestream listings
- Physical location determined as:
  - City: Long Beach
  - State: California
  - Country: USA

This descriptor matched the challenge requirement.

---

## Flag

BYPASS_CTF{Long_Beach_California_Usa}