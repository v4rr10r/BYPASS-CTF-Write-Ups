# Dead Men Speak NO Plaintext - BYPASS CTF

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Forensics |
| **Difficulty** | Medium |
| **Points** | 250 |

## Description

> A network capture was recovered from a ship sailing suspicious waters of the Caribbean.  
> At first glance, it’s nothing but noisy chatter — routine lookups, failed connections, and meaningless traffic drifting like flotsam.  
> But legends say that pirates never write their secrets down.  

**Flag format:** `BYPASS_CTF{UPPER_CASE}`

---

## TL;DR

The PCAP contained hidden raw audio data disguised as noisy ICMP and UDP traffic. Extracting the payload and converting it into audio revealed a spoken flag.

**Vulnerability:** Hidden data in network payloads  
**Approach:** Raw payload extraction + audio reconstruction

---

## Initial Analysis

### Packet Inspection

- Opened the PCAP file in Wireshark
- Observed large amounts of ICMP and UDP traffic
- Payloads appeared meaningless at first glance

**Key observations:**
- Repeating hex patterns like `7e 7e`
- Payload data looked rhythmic rather than random
- Suggested a continuous raw data stream

---

## Main Analysis

### Payload Extraction

- Extracted raw payload bytes from ICMP and UDP packets
- Saved the combined data as `pirate_secret.bin`

The structure resembled raw audio rather than text or encryption.

---

## Exploitation

### Strategy

1. Extract raw packet payloads into a binary file
2. Convert raw data into audio using SoX
3. Clean the audio using noise reduction
4. Listen to the recovered message

### Implementation
```bash
sox -t raw -r 8000 -b 8 -c 1 -e unsigned-integer pirate_secret.bin pirate.wav
```
### Explanation:

- -t raw → input has no header
- -r 8000 → sample rate (voice-friendly)
- -b 8 → 8-bit depth
- -c 1 → mono channel
- -e unsigned-integer → matches raw byte encoding

### Audio Cleanup

- Opened pirate.wav in Audacity
- Applied Noise Reduction to remove static
- Clear voice spelling out the flag was revealed

## Flag 

BYPASS_CTF{V01P_J4CK_1N_TH3_0P3N}