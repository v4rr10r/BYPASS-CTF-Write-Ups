# Gold Challenge - BYPASS CTF 
## Challenge Information
| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Steganography |
| **Difficulty** | Medium |
| **Points** | 300 |

## Description
> The challenge is contained within the Medallion_of_Cortez.bmp file.
> This cursed coin holds more than just gold.
> They say the greed of those who plundered it left a stain upon its very soul — a fractured image, visible only to those who can peel back the layers of light.
> To lift the curse, you must first reassemble the key. Once the key is whole, its message will grant you the power to unlock the true treasure within.
> Beware, for the final step is guarded, and only the words revealed by the light will let you pass.

**Files provided:** `Medallion_of_Cortez.bmp`

**Flag format:** `BYPASS_CTF{.*}`
## TL;DR
Inspect RGB bit-planes to reveal a hidden QR code, scan it to obtain a password, then use that password with steghide to extract the final flag.
**Vulnerability:** Data hidden in RGB bit planes + steghide payload
**Approach:** Bit-plane analysis → QR decode → steghide extraction
## Initial Analysis
### File Identification
```bash
└─$ file Medallion_of_Cortez.bmp 
Medallion_of_Cortez.bmp: PC bitmap, Windows 3.x format, 1260 x 1260 x 24, image size 4762800, cbSize 4762854, bits offset 54
```
### Steganographic Analysis
Bit-Plane Inspection

Using tools like Stegsolve/GIMP, inspect the 0-bit planes of R, G, and B channels. A QR code appears in one of the 0-bit RGB planes.

QR Decode: 
Scanning the QR code reveals the string:
`SunlightRevealsAll`
This string is used as a password for further extraction.

```
steghide extract -sf Medallion_of_Cortez.bmp -p SunlightRevealsAll
Wrote extracted data to "treasure.txt".
```
### Flag

BYPASS_CTF{Aztec_Gold_Curse_Lifted}
