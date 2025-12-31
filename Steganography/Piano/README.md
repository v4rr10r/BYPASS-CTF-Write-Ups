# Piano - BYPASS CTF


## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Steganography |
| **Difficulty** | Medium |
| **Points** | 250 |

---
> "They said the melody was cursed... a tune played by a mad pirate who hid his treasure in sound itself."  
>  
> Listen close, sailor. The chords you hear aren’t random — they spell the name of his lost ship.

**File:** `pirate_song.mp3`

---

## Challenge Overview
We are given an MP3 audio file and a hint that the secret is hidden *in the melody itself*, not just metadata or obvious steganography.

---

## Solution

### Step 1: Metadata Analysis
First, I checked the file using tools like `exiftool`.  
Inside the comments, I found the following message: 
[Ship Log Entry 13B]
Coordinates: 24°N, 82°W
Note: The Captain hid the map key in the melody itself... only the true ear shall hear it.
Encoded relic:
`123_107_126_65_111_105_116_150_143_110_122_150_141_127_64_163_111_105_153_147_144_107_150_160_142_155_163_147_145_127_71_61_64_157_103_132_144_155_125_147_142_107_106_165_132_107_126_153_111_107_71_165_111_105_122_150_144_156_153_147_123_155_71_165_132_130_120_151_147_112_153_147_143_62_150_160_143_103_105_147_113_105_132_150_141_62_125_147_122_155_170_150_132_171_153_75`


Decoding this produced a readable message, but **no flag**, which confirmed this was just a hint and not the final answer.

---

### Step 2: Listening to the Melody
Since the description strongly hinted at **sound-based encoding**, I analyzed the audio using an online pitch detector:

🔗 https://singingcarrots.com/

After uploading the MP3, the detected **musical notes** were: B4 A4 D5 F5 A4 C4 E4


---

### Step 3: Interpreting the Notes
Taking the **note letters only** and reading them in sequence:


---

## Flag
BYPASS_CTF{BADFACE}