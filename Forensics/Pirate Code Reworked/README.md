# Pirate Code Reworked - BYPASS CTF

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Forensics |
| **Difficulty** | Easy |
| **Points** | 250 |


---

## Description

> Captain Cipher's "Magic Vault" blew itself to smithereens when the Navy closed in.  
> No code left. Just a smoking crater on the blockchain.  
>  
> But the ledger never forgets.  
> What’s dead may never die, but it surely leaves a trace.

---

## TL;DR

A smart contract on **Sepolia** was inspected using a blockchain explorer.  
The flag was recovered by decoding the **raw input bytecode** of a transaction, revealing embedded ASCII strings.

**Approach:** Blockchain forensics  
**Trick:** Extracting readable strings from EVM bytecode

---

## Analysis

1. Opened **https://eth-sepolia.blockscout.com/**
2. Searched the target address: 0xaAD59779A3f824a0C09dB329dcA57aFdbD483314

3. Observed **two transactions**
4. Opened one transaction and inspected the **Raw Input / Bytecode**
5. Found multiple hardcoded hex strings inside the contract

---

## Key Bytecode Snippet

```text
7f4259504153535f4354467b
7f626c30636b636834696e
7f5f31735f66756e6e6e7d
```
```text
Decoding
Hex → ASCII decoding:
42 59 50 41 53 53 5f 43 54 46 7b  -> BYPASS_CTF{\
62 6c 30 63 6b 63 68 34 69 6e     -> bl0ckch4in\
5f 31 73 5f 66 75 6e 6e 6e 7d     -> _1s_funnn}
```
---
### Flag
BYPASS_CTF{bl0ckch4in_1s_funnn}