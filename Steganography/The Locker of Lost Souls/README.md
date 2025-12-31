# The Locker of Lost Souls — BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Steganography |
| **Difficulty** | Medium |
| **Points** | 300 |

---

## Challenge Description

> They say that to be locked away in Davy Jones' Locker is to be erased from the world of the living, a fate worse than death.  
> One of our divers recovered this image from the wreck of the *Sea Serpent*.  
>  
> The ship's log spoke of a curse, a vision that could only be understood by those who could **"see beyond the veil"**.  
>  
> The image seems to be just a picture of an old locker on the seabed, covered in barnacles.  
> Standard instruments find nothing.  
>  
> Maybe the old captain was just mad from the pressure…  
> Or maybe you're just not looking at it the right way.

**File provided:**  
- `davy_jones_locker.png`

---

## TL;DR

The image is a **single-image stereogram (autostereogram)**.  
By detecting the horizontal repetition pattern and subtracting a shifted copy of the image, the hidden text becomes visible.

**Technique:** Stereogram / Depth illusion decoding  
**Key Insight:** Hidden information appears only after shifting the image by the correct horizontal period.

---

## Initial Analysis

- Visually, the image looks completely normal.
- No metadata, no obvious LSB patterns.
- AperiSolve, zsteg, binwalk, strings → **nothing useful**.
- Challenge hint: *“see beyond the veil”* strongly suggests a **visual illusion** rather than classic stego.

This matches how **autostereograms** work:  
> Information is hidden in repeating horizontal patterns, not pixel values.

---

## Main Analysis: Stereogram Decoding

### Step 1: Find the Repetition Period

Autostereograms hide depth by repeating horizontal patterns.  
The correct **horizontal shift** is the key.

Approach:
- Shift the image left by varying pixel amounts.
- Compute the **mean absolute difference** between original and shifted image.
- The shift with the **lowest difference** reveals the repeat width.

✔️ The correct shift was found at:

SHIFT = 90 pixels


---

### Step 2: Difference Image

Once the correct shift is known:
- Subtract the shifted image from the original.
- This removes the repeating texture and highlights hidden depth.

This immediately revealed **faint text**.

---

### Step 3: Enhance Visibility

To make the text readable:
- Convert to grayscale
- Increase contrast
- Binarize
- Apply small morphological cleanup
- Crop the central horizontal band

At this stage, the hidden message becomes clearly readable.

---

## Exploitation Script

```python
import cv2
import numpy as np

# Load image
img = cv2.imread("davy_jones_locker.png", cv2.IMREAD_GRAYSCALE)

# Horizontal shift (found experimentally)
SHIFT = 90

# Roll image horizontally
shifted = np.roll(img, -SHIFT, axis=1)

# Compute difference
diff = cv2.absdiff(img, shifted)

# Enhance contrast
diff = cv2.normalize(diff, None, 0, 255, cv2.NORM_MINMAX)

# Threshold to binary
_, binary = cv2.threshold(diff, 40, 255, cv2.THRESH_BINARY)

# Morphological cleanup
kernel = np.ones((3,3), np.uint8)
binary = cv2.morphologyEx(binary, cv2.MORPH_CLOSE, kernel)

# Save output
cv2.imwrite("revealed_flag.png", binary)

print("[+] Hidden message revealed in revealed_flag.png")
```

Result

Opening the difference image (shift = 90) clearly reveals the hidden flag text.

## Flag
BYPASS_CTF{D34D_M4N5_CH35T}