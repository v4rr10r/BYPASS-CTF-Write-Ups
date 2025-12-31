# Pieces of Four - BYPASS CTF 
## Challenge Information
| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Forensics |
| **Difficulty** | Medium |
| **Points** | 300 |

## Description
> Legends speak of a cursed chest recovered from the depths of the Caribbean.
> Every pirate who opens it swears they see something different —
> a torn map fragment, a broken sigil, or a piece of a greater truth.

**Files provided:** `piece_of_four`

**Flag format:** `BYPASS_CTF{.*}`
## TL;DR
An embedded SVG was found using strings, which contained a base64-encoded PNG. Decoding the PNG revealed a QR code that directly contained the flag.
**Vulnerability:** Hidden base64 image inside SVG data
**Approach:** Strings → Base64 decode → QR scan
## Initial Analysis
### File Inspection
```bash
...
strings piece_of_four
xlink:href="data:image/png;base64,...."
...
```
While reviewing output, an SVG snippet appeared containing an embedded PNG:
This indicated image-in-image steganography.
## Forensic Analysis
### Extracting Embedded Image

The base64 PNG was extracted and decoded using the following Python script:
``` python
import base64
data = """data:image/png;base64,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"""
base64_data = data.split(",")[1]
image_bytes = base64.b64decode(base64_data)
with open("hidden.png","wb") as f:
    f.write(image_bytes)
print("hidden.png created successfully")
```
The hidden.PNG contains a QR code. Scanning the QR reveals the final flag.

## Flag

BYPASS_CTF{JPEG_PNG_GIF_TIFF_c0mm0n}