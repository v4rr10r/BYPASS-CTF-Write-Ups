#  Silas's Last Voyage - Bypass CTF 

## Challenge Information
| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Forensics |
| **Difficulty** | Hard |
| **Points** | 400 |


## Challenge Description

Recovered from the wreck of *The Gilded Eel*, this hard drive belonged to the paranoid navigator Silas Blackwood. Legend says Silas found the route to the "Zero Point" treasure, but he claimed the map itself was alive—shifting and lying to anyone who tried to read it directly.

He left his legacy in this drive. We've tried standard extraction, but all we found were sea shanties and corrupted images. There’s a rumor that Silas split the truth into four winds: Sound, Sight, Logic, and Path.

Beware the "Fools' Gold" (fake flags)—Silas loved to mock the impatient. You must assemble the pieces in the order the *Captain* commands.

**File:** `silas_drive.img`

---

## Introduction

"Silas's Last Voyage" is a multi-stage forensics challenge that requires participants to analyze a disk image, recover data from a damaged or non-standard partition, and solve multiple puzzles hidden within the recovered files. The challenge combines file system analysis (BTRFS), image processing (XOR steganography), reverse engineering of Python scripts, and cryptographic decoding based on contextual clues. The final objective is to assemble a flag from fragments obtained by solving these distinct puzzles.

## Solution Walkthrough

### Step 1: Initial Analysis and File System Identification

The challenge provides a disk image named `silas_drive.img`. The first step is to identify the file type and partition structure.

```bash
$ file silas_drive.img
silas_drive.img: DOS/MBR boot sector; partition 1 : ID=0xc, start-CHS (0x0,32,33), end-CHS (0x1,70,6), startsector 2048, 18433 sectors, extended partition table (last)
```

Running `strings` on the image reveals many fake flags, confirming the "Fools' Gold" warning in the description.

```bash
$ strings silas_drive.img | grep "BYPASS_CTF" | head -n 5
BYPASS_CTF{y0u_f0und_1t_n0t}
BYPASS_CTF{y0u_f0und_1t_n0t}
BYPASS_CTF{y0u_f0und_1t_n0t}
BYPASS_CTF{y0u_f0und_1t_n0t}
BYPASS_CTF{y0u_f0und_1t_n0t}
```

Using `binwalk`, we can see the partition layout more clearly. It reveals a FAT32 partition at the beginning and a BTRFS file system starting at offset `20971520`.

```bash
$ binwalk silas_drive.img

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             DOS Master Boot Record, partition: FAT32, image size: 10486272 bytes
20971520      0x1400000       BTRFS file system, node size: 16384, sector size: 4096...
```

### Step 2: Partition Extraction and Data Recovery

Standard extraction tools like `7z` might fail to extract the BTRFS partition correctly or might only see the first FAT partition (which contains a `readme.txt` with no useful info). We need to carve out the BTRFS partition manually using `dd`.

```bash
$ dd if=silas_drive.img of=btrfs_partition.img bs=1 skip=20971520
105906176+0 records in
105906176+0 records out
105906176 bytes (106 MB, 101 MiB) copied, 0.346 s, 306 MB/s
```

Attempting to mount or extract this image might fail if the filesystem is damaged or if we lack permissions. However, the `btrfs restore` utility is excellent for recovering files from unmounted BTRFS images.

```bash
$ mkdir btrfs_restore_output
$ btrfs restore btrfs_partition.img btrfs_restore_output
```

The restored directory structure contains several interesting files:

```text
btrfs_restore_output/
├── charts/
│   └── north_sea.png
├── games/
│   └── navigate.py
├── logs/
│   └── .captain_log.swp
├── system/
│   └── password_hint.txt
├── treasure/
│   └── coin_scan.bmp
├── flag.txt
├── script.py
└── shanty_of_lies.wav
```

The `flag.txt` contains another fake flag: `BYPASS_CTF{y0u_f0und_1t_n0t}`. We must look deeper into the other files.

### Step 3: Solving the "Four Winds"

The description mentions four components: **Sound, Sight, Logic, and Path**. We need to investigate the files to find these pieces.

#### 1. Logic (`games/navigate.py`)

Analyzing `games/navigate.py`, we find a simple text-based game. However, inside the `start_game` function, there is a suspicious variable `_error_log` containing a base64 string.

```python
def start_game():
    print("Welcome to the Navigator's Challenge.")
    print("Directions: N, E, S, W")
    
    # Hidden secret
    _error_log = "dGVsbF9ub18=" 
    # ...
```

Decoding this base64 string:

```bash
$ echo "dGVsbF9ub18=" | base64 -d
tell_no_
```

**Fragment found:** `tell_no_`

#### 2. Sight (`script.py`, `charts/north_sea.png`, `treasure/coin_scan.bmp`)

The `script.py` file attempts to perform an image operation but has a bug (it looks for `.jpg` instead of `.png`) and requires the images to be the same size.

```python
# Original script.py snippet
img1 = cv2.imread('charts/north_sea.jpg') # Error: File is actually .png
img2 = cv2.imread('treasure/coin_scan.bmp')
# ...
result = cv2.bitwise_xor(img1, img2)
```

We can fix the script to point to the correct files and perform the XOR operation.

**Solver Script (`solve_xor.py`):**

```python
import cv2
import numpy as np
import os

# Load the images with correct paths
img1 = cv2.imread('btrfs_restore_output/charts/north_sea.png')
img2 = cv2.imread('btrfs_restore_output/treasure/coin_scan.bmp')

# Resize img2 to match img1
if img1.shape != img2.shape:
    img2 = cv2.resize(img2, (img1.shape[1], img1.shape[0]))

# Perform XOR
result = cv2.bitwise_xor(img1, img2)

# Save result
cv2.imwrite('xor_result.png', result)
print("Result saved to xor_result.png")
```

Running this script produces `xor_result.png`. The resulting image contains the text **"tales"**.

**Fragment found:** `tales`

#### 3. Path (`logs/.captain_log.swp`)

The `logs` directory contains a hidden Vim swap file `.captain_log.swp`. Running `strings` on it reveals a poem and a block of hex data.

```text
Seven Islands Lined Across Sea
Under Night, Death Eclipses All
Ride East, Ghosts Upon Red
Find Infinity, Never Going Over
HEX_DATA_START
37 30 33 22 0C 38 37 28 0C
HEX_DATA_END
```

The poem is an acrostic. Taking the first letter of each line spells **SURF**:
*   **S**even...
*   **U**nder...
*   **R**ide...
*   **F**ind...

Using "SURF" as a repeating XOR key against the hex data reveals the next fragment.

**Decoding Logic:**

```python
hex_data = [0x37, 0x30, 0x33, 0x22, 0x0C, 0x38, 0x37, 0x28, 0x0C]
key = "SURF" # ASCII: 53 55 52 46

decoded = ""
for i, byte in enumerate(hex_data):
    decoded += chr(byte ^ ord(key[i % len(key)]))

print(decoded)
```

**Output:**
```text
dead_men_
```

**Fragment found:** `dead_men_`

#### 4. Sound (`shanty_of_lies.wav`)

The file `shanty_of_lies.wav` was analyzed. Given the filename ("Lies") and the challenge description ("Silas claimed the map... lying"), this file serves as a red herring. It contains no hidden data in the spectrogram or metadata relevant to the flag. It represents the "Sound" wind, which is a lie.

### Step 4: Assembly

We have the following fragments:
1.  `tell_no_` (Logic)
2.  `tales` (Sight)
3.  `dead_men_` (Path)

The challenge description says: *"You must assemble the pieces in the order the Captain commands."*

The fragments form the famous pirate phrase: **"Dead men tell no tales"**.

Ordering them logically:
1.  Path: `dead_men_`
2.  Logic: `tell_no_`
3.  Sight: `tales`

Combining them with the flag format:

## Flag
`BYPASS_CTF{dead_men_tell_no_tales}`
