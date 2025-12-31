# Jellies - BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Osint |
| **Difficulty** | Medium |
| **Points** | 300 |

## Description

> A strange image has been recovered from an oceanic research buoy after it briefly connected to an unknown network.  
> No metadata survived — only a single frame showing ethereal, floating creatures suspended in blue water.  
>  
> But something isn’t right.  
> The currents in the background flow too smoothly.  
> The illumination is too perfect.  
> And deep within the fluid shadows, a faint pattern seems to flicker… almost as if the ocean itself is whispering coordinates.  
>  
> They also suspect there is a hidden pattern hiding in the undulating shapes of the drifting creatures.  
> Find out the species of image and the secret code hidden.

**Flag format:** `BYPASS_CTF{species_code}`

---

## TL;DR

The image contained hidden text revealed through steganographic analysis. Identifying the jellyfish species and combining it with the extracted hidden string produced the final flag.

**Vulnerability:** Hidden data in image channels  
**Approach:** Steganography analysis + OSINT species identification

---

## Initial Analysis

### Image Inspection

- Loaded the image into **AperiSolve**
- Checked common steganography outputs:
  - zsteg
  - strings
  - metadata (none present)

**Key observation:**
- zsteg revealed a readable hidden message

---

## Main Analysis

### Hidden Message Extraction

From the zsteg output, the following string was recovered:

this_is_a_jelly_H01&01$<stop>


This indicated:
- A confirmation that the image subject is a jellyfish
- A secret code segment to be used in the flag

---

## Species Identification

- Performed Google Image Search / visual comparison
- Compared known jellyfish species
- The closest and correct match was **Sea Nettles**

---

## Flag Construction

- Species name used: `Sea_Nettles`
- Appended the hidden message exactly as extracted

---

## Flag

BYPASS_CTF{Sea_Nettles_this_is_a_jelly_H01&01$}
