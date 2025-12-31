# Level Devil 💀 - BYPASS CTF

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Misc |
| **Difficulty** | Medium |
| **Points** | 250 |
| **Challenge Link** | https://level-devil-dcmi.onrender.com/ |

---

## Description

> Looks simple. Plays dirty.  
>  
> Welcome to Level Devil, a platformer that refuses to play fair.  
> The path looks obvious, the goal seems close—but this level thrives on deception.  
>  
> Hidden traps, unreliable platforms, and misleading progress will test your patience and awareness.  
> Not everything you see is safe, and not every solution lies in plain sight.  
>  
> Your mission is simple:  
> Reach the end and claim what’s hidden.  
>  
> But remember—  
> in this level, trust is your biggest weakness.  
>  
> Good luck. You’ll need it.

**Flag format:** `BYPASS_CTF{.*}`

---

## TL;DR

The game enforces timing checks to prevent fast completion.  
By directly calling backend API endpoints from the browser console with proper delays, these checks can be bypassed and the flag can be retrieved.

**Vulnerability:** Client-side trust & timing-based validation  
**Approach:** Direct API interaction via browser console

---

## Initial Analysis

### Web Inspection

- Opened browser DevTools
- Observed network requests during gameplay
- Identified backend API endpoints controlling game flow

**Key observations:**
- Game logic depends on backend APIs
- Client events trigger server-side timing validation
- A reusable `session_id` is generated via an API call

---

## Web Analysis

### API Flow Understanding

The game backend follows this sequence:

1. Start a session using `/api/start`
2. Prepare flag data using `/api/collect_flag`
3. Validate completion using `/api/win`

These endpoints are callable directly from the browser console.

---

## Exploitation

### Strategy

1. Initialize a valid session to obtain a `session_id`
2. Wait to bypass the “Too fast” timing check
3. Call the flag collection endpoint
4. Finalize the session and retrieve the flag

---

### Implementation

```javascript
fetch('/api/start', { method: 'POST' })
  .then(r => r.json())
  .then(d => {
    const session_id = d.session_id;
    console.log('🎯 SESSION ID:', session_id);

    setTimeout(() => {
      fetch('/api/collect_flag', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ session_id })
      }).then(() => {
        setTimeout(() => {
          fetch('/api/win', {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({ session_id })
          })
          .then(r => r.json())
          .then(console.log);
        }, 10000);
      });
    }, 5000);
  });

```



## Flag
BYPASS_CTF{l3v3l_d3v1l_n0t_s0_1nn0c3nt}

---
pwn by **W4RR1OR**