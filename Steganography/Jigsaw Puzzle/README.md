# Jigsaw Puzzle  - BYPASS CTF 

## Challenge Information
| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Steganography |
| **Difficulty** | Medium |
| **Points** | 250 |


**Challenge Description:**  
A rival pirate ransacked Captain Jack Sparrow's cabin and, in a fit of rage, tore his portrait to shreds. But this was no ordinary portrait. The Captain, in his infinite cunning, had scrawled his latest secret orders across the back of it before it was framed. The 25 pieces were scattered across the deck. If you can piece the Captain's portrait back together, you might just be able to read his hidden message.

**Files Given:**  pieces.rar
## Introduction

This challenge presents a data reconstruction and visual cryptography problem. The participant is provided with a set of 25 image files, which are fragments of a larger original image containing a hidden message. The core task involves algorithmic image processing to reassemble the scattered pieces based on edge continuity. Once the image is reconstructed, the hidden text must be extracted and deciphered. The challenge tests skills in scripting, image manipulation (using libraries like Pillow or OpenCV), and basic cryptography.

## Vulnerability Analysis

The "vulnerability" in this context is **Information Fragmentation**. The secret data (the flag) was not encrypted but merely obfuscated by splitting the medium (the image) into unordered parts. The security of this method relies entirely on the computational complexity of reassembling the parts. However, since the edges of the image fragments contain redundant visual information (continuity of pixel colors), this complexity can be drastically reduced using edge-matching algorithms, rendering the obfuscation ineffective against automated analysis.

## Solution Walkthrough

### Step 1: Initial Analysis

We start by inspecting the provided files in the `pieces/` directory.

```bash
ls pieces/ | head -n 5
```

Output:
```text
piece_1804d586.png
piece_225265b9.png
piece_31f06ef3.png
piece_38b73580.png
piece_3f42f763.png
...
```

We verify the dimensions and format of the images using a Python script.

```python
import os
from PIL import Image

pieces_dir = 'pieces'
files = os.listdir(pieces_dir)

for f in files:
    if f.endswith('.png'):
        path = os.path.join(pieces_dir, f)
        img = Image.open(path)
        print(f"{f}: {img.size}, mode={img.mode}")
```

All 25 images are found to be **384x216** pixels in RGB mode. Since there are 25 pieces, we can infer they form a **5x5 grid**.

### Step 2: Reassembling the Puzzle

To automate the reassembly, we treat this as an edge-matching problem. For any two pieces, we calculate a "cost" representing how well they fit together.

*   **Horizontal Cost:** Difference between the rightmost column of pixels of Piece A and the leftmost column of Piece B.
*   **Vertical Cost:** Difference between the bottommost row of pixels of Piece A and the topmost row of Piece B.

We use the **Sum of Squared Differences (SSD)** as our metric. A lower SSD indicates a better visual match.

#### Solver Script (`solve_puzzle.py`)

The following script calculates these costs, identifies the best neighbors for each piece, and reconstructs the grid.

```python
import os
import numpy as np
from PIL import Image

pieces_dir = 'pieces'
files = sorted([f for f in os.listdir(pieces_dir) if f.endswith('.png')])
images = {}
for f in files:
    img = Image.open(os.path.join(pieces_dir, f))
    images[f] = np.array(img)

print(f"Loaded {len(images)} images.")

def get_diff(edge1, edge2):
    # Sum of squared differences
    diff = np.sum((edge1.astype(int) - edge2.astype(int)) ** 2)
    return diff

filenames = list(images.keys())
n = len(filenames)
H_cost = np.zeros((n, n))
V_cost = np.zeros((n, n))

# Calculate pairwise costs
for i in range(n):
    for j in range(n):
        if i == j:
            H_cost[i, j] = np.inf
            V_cost[i, j] = np.inf
            continue
        
        img_i = images[filenames[i]]
        img_j = images[filenames[j]]
        
        # Right of i vs Left of j
        right_i = img_i[:, -1, :]
        left_j = img_j[:, 0, :]
        H_cost[i, j] = get_diff(right_i, left_j)
        
        # Bottom of i vs Top of j
        bottom_i = img_i[-1, :, :]
        top_j = img_j[0, :, :]
        V_cost[i, j] = get_diff(bottom_i, top_j)

# Greedy Grid Construction
best_grid = None
min_total_cost = np.inf

# Try starting with every piece as the top-left corner
for start_idx in range(n):
    current_grid = [[None for _ in range(5)] for _ in range(5)]
    used = set()
    
    start_node = filenames[start_idx]
    current_grid[0][0] = start_node
    used.add(start_idx)
    
    current_total_cost = 0
    valid = True
    
    for y in range(5):
        for x in range(5):
            if x == 0 and y == 0: continue
            
            best_next = -1
            best_cost = np.inf
            
            # Find best candidate for position (x, y)
            for candidate_idx in range(n):
                if candidate_idx in used: continue
                
                cost = 0
                
                # Check match with Left neighbor
                if x > 0:
                    left_node = current_grid[y][x-1]
                    left_idx = filenames.index(left_node)
                    cost += H_cost[left_idx, candidate_idx]
                
                # Check match with Top neighbor
                if y > 0:
                    top_node = current_grid[y-1][x]
                    top_idx = filenames.index(top_node)
                    cost += V_cost[top_idx, candidate_idx]
                
                if cost < best_cost:
                    best_cost = cost
                    best_next = candidate_idx
            
            if best_next != -1:
                current_grid[y][x] = filenames[best_next]
                used.add(best_next)
                current_total_cost += best_cost
            else:
                valid = False
                break
        if not valid: break
    
    if valid and current_total_cost < min_total_cost:
        min_total_cost = current_total_cost
        best_grid = current_grid

# Save the result
if best_grid:
    full_img = Image.new('RGB', (384*5, 216*5))
    for y in range(5):
        for x in range(5):
            fname = best_grid[y][x]
            img = Image.open(os.path.join(pieces_dir, fname))
            full_img.paste(img, (x*384, y*216))
    full_img.save('reassembled.png')
    print("Saved reassembled.png")
```

Running this script successfully generates `reassembled.png`, which reveals the complete image with text overlaid on it.

### Step 3: Extracting the Ciphertext

The reassembled image contains the following text, which can be read manually or extracted via OCR tools like Tesseract:

```text
Gu r cn ff jb eq v f: O LC NF F_ PG S{ RV TU G_ CV RP RF _B S_ RV TU G}
```

The spacing is irregular due to the font style, but the characters are clear.

### Step 4: Decoding the Flag

The text `Gu r cn ff jb eq v f` strongly resembles the phrase "The password is" but shifted. This indicates a **ROT13** (Caesar cipher with shift 13) substitution.

We verify this with a Python script:

```python
import codecs

ciphertext = "Gu r cn ff jb eq v f: O LC NF F_ PG S{ RV TU G_ CV RP RF _B S_ RV TU G}"
# Remove extra spaces for cleaner decoding if necessary, 
# but here we decode directly to preserve structure.
plaintext = codecs.decode(ciphertext, 'rot_13')
print(plaintext)
```

**Output:**
```text
Th e pa ss wo rd i s: B YP AS S_ CT F{ EI GH T_ PI EC ES _O F_ EI GH T}
```

Cleaning up the spaces, we get the final flag.

## Conclusion

By algorithmically reassembling the image fragments based on edge pixel continuity and applying a standard ROT13 decryption to the revealed text, we successfully recovered the secret orders.

## Flag
 `BYPASS_CTF{EIGHT_PIECES_OF_EIGHT}`
