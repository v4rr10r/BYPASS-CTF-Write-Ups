# Dead Man's Riddle - BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Rev |
| **Difficulty** | Hard |
| **Points** | 400 |


## Description

> Ye who seek the treasure must pay the price... Navigate the chaos, roll the dice.
>
> A spectral chest sits before you, guarded by a cursed lock that shifts with the tides.  
> The lock remembers every mistake. There are no keys, only a passphrase.
>
> Can you break the curse and claim the flag?

**Files provided:**
- `dead_mans_riddle` – Linux ELF binary implementing a stateful password check

**Flag format:** `BYPASS_CTF{.*}`

---

## TL;DR

The binary validates a 30-character passphrase using a global state (`g_state`) that mutates per character.  
By reversing the initialization logic, character transformation, and validation table, the exact passphrase can be computed programmatically.

**Vulnerability:** Predictable stateful verification logic  
**Approach:** Static reverse engineering + mathematical inversion

---

## Initial Analysis

### Binary Reconnaissance

```bash
└─$ file dead_mans_riddle  
dead_mans_riddle: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=c06e914df52cd0afa53dc5fc5bf2e323bcd253d4, for GNU/Linux 4.4.0, with debug_info, not stripped

└─$strings dead_mans_riddle
....
BYPASS_CTF{D3ad_M3n_T3ll_N0_Tal3s_But_Th1s_1s_F4k3}
...
```
``` bash

int main(int argc,char **argv)

{
  int iVar1;
  char *pcVar2;
  size_t sVar3;
  char **argv_local;
  int argc_local;
  navigator_t wheel [4];
  char input [64];
  int val;
  int wheel_idx;
  size_t len;
  int i;
  
  puts("Welcome to the Captain\'s Quarters.");
  puts("A heavy iron chest sits before you, bound by a spectral lock.");
  printf("Whisper the passphrase to break the curse: ");
  pcVar2 = fgets(input,0x40,stdin);
  if (pcVar2 == (char *)0x0) {
    iVar1 = 0;
  }
  else {
    sVar3 = strcspn(input,"\n");
    input[sVar3] = '\0';
    sVar3 = strlen(input);
    if (sVar3 == 0x1e) {
      wheel[0] = consult_compass;
      wheel[1] = consult_compass;
      wheel[2] = consult_compass;
      wheel[3] = consult_compass;
      puts("Turning the tumblers...");
      for (i = 0; (uint)i < 0x1e; i = i + 1) {
        iVar1 = (*wheel[(i * 7) % 4])((int)input[i],i);
        iVar1 = check_course(iVar1,i);
        if (iVar1 == 0) {
          walk_the_plank();
        }
        if (i % 5 == 0) {
          usleep(100);
        }
      }
      if (g_state == 0x7fe43150) {
        walk_the_plank();
      }
      open_chest();
      iVar1 = 0;
    }
    else {
      puts("The lock does not budge. The length seems wrong.");
      iVar1 = 1;
    }
  }
  return iVar1;
}


 ```
**Key observations:**

- 64-bit ELF, not stripped
- Fake flag present in strings (red herring)
- Uses a global variable g_state defined as `0xdeadbeef` stored in .data 
- `if (sVar3 == 0x1e) ` Input length must be exactly 30 characters

## Reverse Engineering
### Global State Initialization (init_map)

The function init_map is executed before main() via .init_array.

``` code 
void init_map(void)
{
  g_state = ((g_state ^ 0x1337c0de) << 3 | g_state >> 0x1d) ^ 0x1337c0de;
  return;
}
```
`g_state = ((g_state ^ 0x1337c0de) <<< 3) ^ 0x1337c0de`

Initial value from .data: g_state = `0xdeadbeef`

After initialization:
g_state = `0x7fe43150`

**The function changes (scrambles) the bits, but does it in a reversible way, so you can recover the original value if you know the steps.**

#### Example 
Start value (4 bits)
g_state = 1011
We will rotate left by 1 (same idea as your code, just smaller).
```
Step 1: 1011 << 1  = 0110 (Left shift)

Step 2: 1011 >> 3  = 0001 (Right Shift)

(This grabs the bit that fell off)

Step 3: OR them together  
0110
0001
----
0111

Final result
1011  →  0111
The lost bit wrapped around to the right side.
```
### Character Transformation (consult_compass)
``` code 
int consult_compass(int c,int pos)
{
  uint uVar1;
  int pos_local;
  int c_local;
  int transformed;
  int key;
  int shift;
  
  uVar1 = g_state >> ((char)pos + (char)(pos / 5) * -5 & 0x1fU);
  g_state = c + g_state * 0x7a69;
  return pos + c ^ uVar1 & 0xff;
}
```
Each character is processed as follows:
```
shift  = index % 5
wheel  = (g_state >> shift) & 0xff
result = (char + index) ^ wheel
g_state = g_state * 0x7a69 + char
```
The transformed value is then validated.

### Validation (check_course)

check_course is a large switch statement that checks the transformed value for each index (0–29).\
Expected values table:
```
[0x12,0x53,0x3c,0x44,0x20,0x77,0xa8,0xe8,0x52,0x31,
 0xeb,0x93,0x38,0x28,0x6f,0x67,0x5f,0x2e,0xc8,0xde,
 0x74,0xe0,0x79,0xb9,0x48,0x54,0xf1,0x80,0xcb,0x58]
```

## Exploitation

- Recreate init_map to get correct starting g_state
- For each index:
- Bruteforce the character that satisfies
(char + index) ^ wheel == expected[index]
- Update g_state after each character
- Assemble the final passphrase

### Script
``` python
#!/usr/bin/env python3

def rol32(val, n):
    return ((val << n) | (val >> (32 - n))) & 0xffffffff

def init_map(g):
    g ^= 0x1337c0de
    g = rol32(g, 3)
    g ^= 0x1337c0de
    return g

def consult_compass(c, i, g):
    shift = i % 5
    wheel = (g >> shift) & 0xff
    res = (c + i) ^ wheel
    g = (g * 0x7a69 + c) & 0xffffffff
    return res, g

EXPECTED = [
    0x12,0x53,0x3c,0x44,0x20,0x77,0xa8,0xe8,0x52,0x31,
    0xeb,0x93,0x38,0x28,0x6f,0x67,0x5f,0x2e,0xc8,0xde,
    0x74,0xe0,0x79,0xb9,0x48,0x54,0xf1,0x80,0xcb,0x58
]

g = init_map(0xdeadbeef)
flag = ""

for i in range(30):
    for c in range(256):
        r, _ = consult_compass(c, i, g)
        if r == EXPECTED[i]:
            flag += chr(c)
            _, g = consult_compass(c, i, g)
            break

print(flag)
```
$ python3 solve.py
BYPASS_CTF{T1d3s_0f_D3c3pt10n}

## Flag 
BYPASS_CTF{T1d3s_0f_D3c3pt10n}

pwn by **W4RR1OR**
