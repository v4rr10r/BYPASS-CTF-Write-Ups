# The Cursed Compass - BYPASS CTF 

## Challenge Information
| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Rev|
| **Difficulty** | 	Hard |
| **Points** | 400 |


## Challenge Description

> "The seas are rough, and the Kraken awaits."
>
> We've recovered a strange game from a derelict pirate ship. The captain claimed the game held the coordinates to his greatest treasure.
> But every time we win, the treasure seems... fake.
>
> Can you navigate the treacherous code and find what lies beneath the surface?
>
> The game is built for Linux (x86_64). You might need to install SDL2 to run it (`sudo apt install libsdl2-2.0-0` or similar).
>
> **Hint:** Sometimes, the waves themselves whisper the secrets.
>
> **Note:** The flag format is BYPASS_CTF{...}

Files given: `cursed_compass`
## Introduction

"The Cursed Compass" is a reverse engineering challenge that presents the player with a Linux ELF executable. The binary is a graphical game built using the SDL2 library. The core of the challenge lies in identifying that the obvious "win" condition leads to a decoy flag, while the real flag is obfuscated within the rendering logic of the game. The solution involves static analysis to identify a custom decryption routine disguised as a physics calculation and writing a script to extract the hidden information.

## Initial Reconnaissance

We start by analyzing the provided file `cursed_compass`.

```bash
$ file cursed_compass
cursed_compass: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 4.4.0, with debug_info, not stripped
```

It is a 64-bit unstripped ELF binary. Running `strings` on the binary reveals some interesting text, including what appears to be a flag:

```bash
$ strings cursed_compass | grep "BYPASS"
BYPASS_CTF{Y0u_S4il3d_Th3_S3v3n_S34s_But_M1ss3d_Th3_Tr34sur3}
```

However, the challenge description warns us: *"But every time we win, the treasure seems... fake."* This suggests that `BYPASS_CTF{Y0u_S4il3d_Th3_S3v3n_S34s_But_M1ss3d_Th3_Tr34sur3}` is a decoy.

Running the game (`./cursed_compass`) opens a window titled "The Cursed Compass". The objective is to collect gold coins while avoiding enemies.

## Static Analysis

We proceed with static analysis using `objdump` (or a tool like Ghidra/IDA Pro). Since the binary is not stripped, we can easily identify function names.

### The Main Loop

The `main` function initializes SDL and enters a standard game loop:

1.  **Initialization**: Calls `SDL_Init`, `SDL_CreateWindow`, `SDL_CreateRenderer`.
2.  **Game Loop**:
    *   `SDL_PollEvent`: Handles input.
    *   `update_game`: Updates game logic (movement, collisions).
    *   `render_game`: Draws the game state to the screen.
    *   `SDL_Delay`: Controls frame rate.

### The Decoy

Looking at `update_game`, we can see the logic for checking if the player has won.

```assembly
    if (99 < state->score) {
      state->victory = 1;
      puts("\n[!] VICTORY! The seas are safe.");
      printf("[!] Reward: %s\n",FAKE_FLAG);
    }
  }
  if (lVar1 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
```

When the win condition is met, the code explicitly prints the `FAKE_FLAG`. This confirms that playing the game normally will not yield the correct flag.

### The Hidden Logic

The hint *"Sometimes, the waves themselves whisper the secrets"* directs our attention to the rendering or "wave" related code. We examine `render_game`.

Inside `render_game`, amidst the standard SDL drawing calls, there is a call to a function named `calculate_wave_physics`.

```
cVar2 = calculate_wave_physics((frame_count / 10) % 0x24,g_chaos);
```

This function is called inside a loop that seems to be iterating over pixels or grid cells.

### Analyzing `calculate_wave_physics`

Let's disassemble `calculate_wave_physics`

```
char calculate_wave_physics(int index,uint current_state)

{
  byte bVar1;
  uint current_state_local;
  int index_local;
  uchar s_val;
  char magic;
  uint s;
  int k;
  int shift;
  
  if ((index < 0) || (0x23 < (uint)index)) {
    bVar1 = 0;
  }
  else {
    s = 0xbadf00d;
    for (k = 0; k < index; k = k + 1) {
      s = s * 0x19660d + 0x3c6ef35f;
    }
    bVar1 = *(byte *)((long)&g_tide_data + (long)index) ^
            (byte)(s >> ((char)index + (char)(index / 7) * -7 & 0x1fU));
  }
  return bVar1;
}
```

**Logic Breakdown:**

1.  **Input**: The function takes an integer index `i`.
2.  **State Initialization**: It initializes a variable `state` with `0xbadf00d`.
3.  **State Evolution**: It runs a loop `i` times. In each iteration, it updates `state` using a Linear Congruential Generator (LCG) formula:
    `state = (state * 0x19660d + 0x3c6ef35f) & 0xFFFFFFFF`
4.  **Key Extraction**: It calculates a shift amount based on `i` (specifically `i % 7` logic is implied by the surrounding assembly, though simplified here). It then shifts the `state` right by this amount and takes the lowest byte to form a `key`.
5.  **Decryption**: It fetches the byte at index `i` from a global array `g_tide_data` and XORs it with the `key`.

This function isn't calculating physics at all; it's decrypting data from `g_tide_data` byte by byte.

### Extracting the Data

We need the contents of `g_tide_data`. We can find its location in the `.rodata` section.

```bash
                             g_tide_data                                     XREF[2]:     calculate_wave_physics:004012cd(
                                                                                          calculate_wave_physics:004012d4(
        00402080 4f 5d 21 4e     int        4E215D4Fh                                        cursed_compass.c:28
        00402084 0a              ??         0Ah
        00402085 5e              ??         5Eh    ^
        00402086 98              ??         98h
                            ........
```

The relevant bytes (before the string "[-] The Kraken...") are:
`4f 5d 21 4e 0a 5e 98 0d fe ea b2 b0 c8 57 9e e8 b8 49 84 05 ce 7e 49 ea ef 6f 16 e3 8a 29 70 44 83 a5 39 67`

## Solution Script

We can now write a Python script to replicate the `calculate_wave_physics` logic and decrypt the flag.

```python
# Encrypted bytes extracted from g_tide_data
data = [
    0x4f, 0x5d, 0x21, 0x4e, 0x0a, 0x5e, 0x98, 0x0d, 
    0xfe, 0xea, 0xb2, 0xb0, 0xc8, 0x57, 0x9e, 0xe8, 
    0xb8, 0x49, 0x84, 0x05, 0xce, 0x7e, 0x49, 0xea, 
    0xef, 0x6f, 0x16, 0xe3, 0x8a, 0x29, 0x70, 0x44, 
    0x83, 0xa5, 0x39, 0x67
]

def decrypt():
    flag = ""
    # Iterate through each byte of the encrypted data
    for i in range(len(data)):
        state = 0xbadf00d
        
        # Replicate the inner loop: update state 'i' times
        # state = (state * 0x19660d + 0x3c6ef35f)
        for j in range(i):
            state = (state * 0x19660d + 0x3c6ef35f) & 0xFFFFFFFF
    
        shift = i % 7
        
        # Extract key byte
        key = (state >> shift) & 0xFF
        
        # Decrypt
        decrypted_char = chr(data[i] ^ key)
        flag += decrypted_char
        
    print(f"Flag: {flag}")

if __name__ == "__main__":
    decrypt()
```

### Running the Solution

```bash
$ python3 solve.py
Flag: BYPASS_CTF{Fr4m3_By_Fr4m3_D3c3pt10n}
```

## Conclusion

The challenge successfully misled players by providing a functional game with a fake victory condition. The true flag was hidden within the rendering pipeline, protected by a custom stream cipher dependent on the frame/pixel index. By reverse engineering the `calculate_wave_physics` function, we were able to extract the decryption algorithm and recover the flag.

## Flag:
 `BYPASS_CTF{Fr4m3_By_Fr4m3_D3c3pt10n}`

pwn by **W4RR1OR**