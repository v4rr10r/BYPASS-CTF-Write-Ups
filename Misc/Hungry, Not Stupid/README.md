# Hungry, Not Stupid - BYPASS CTF

## Challenge Information

| Field | Value |
|-------|--------|
| **CTF** | BYPASS CTF |
| **Category** | Misc |
| **Difficulty** | Medium |
| **Points** | 300 |
| **Challenge Link** | https://snack-mxc1.onrender.com/ |

## Description

>  
> The snake is hungry — not desperate.  
> Most food is a lie.  
> Only those who observe, experiment, and learn will survive long enough to reach the flag.


**Flag format:** `^BYPASS_CTF{.*}$`

---

## TL;DR

The Snake game does not require playing at all.  
The correct food position for each step is leaked inside a Flask session cookie.  
By decoding the cookie and always submitting the correct coordinates, the flag can be extracted one character per request.

**Vulnerability:** Sensitive game state stored in client-side Flask session cookie  
**Approach:** Decode session cookie → extract correct food → automate `/api/eat`

---

## Initial Analysis

### Web Reconnaissance

The challenge presents a Snake game where:
- Multiple food blocks appear
- Eating the correct food reveals the next flag character
- Eating the wrong food resets the game

Using browser DevTools and Burp Suite, the following API endpoints were identified:

```http
POST /api/start
POST /api/eat
```
### Observations:

- Game logic heavily relies on server responses
- Each request includes a session cookie
- No JavaScript obfuscation or anti-debugging present

### Key observations:

- The game resets immediately on wrong food
- The snake’s movement is not validated server-side
- A Flask session cookie is used to track progress

## Web Analysis
Session Cookie Inspection\
After starting a game, the server sets a cookie similar to:
session=.eJyrVkrMyYlPy89PiS_IL1ayiq5WqlCy...

This format strongly indicates a Flask signed session cookie.

- Flask session cookies: Are base64 + zlib compressed

Are signed, not encrypted
Can be decoded but not modified

Decoding the cookie revealed the following structure:

{
  "all_food_pos": [
    {"x":24,"y":13},
    {"x":2,"y":18},
    {"x":24,"y":27}
  ],
  "collected_flag": "B",
  "correct_food_pos": {"x":24,"y":27},
  "level": 1
}


💡 Critical discovery:
The correct food position is directly leaked inside the cookie.

### Exploitation Strategy

1.  Start a new game to obtain a valid session cookie
2.  Decode the Flask session cookie
3. Extract correct_food_pos
4. Submit it to /api/eat
5. Receive updated cookie with next flag character
6. Repeat until full flag is revealed

**This avoids:**

- Playing the Snake game 
- Guessing food positions 
- Brute force or threading

## Implementation


Automatically solves the Hungry, Not Stupid challenge by:
- Decoding the Flask session cookie
- Extracting the correct food position
- Submitting it to the server
- Repeating until the full flag is recovered


``` python
import requests
import base64
import zlib
import json

BASE_URL = "https://snack-mxc1.onrender.com"

session = requests.Session()

def decode_flask_cookie(cookie_value):
    """
    Decode Flask session cookie (no secret key needed for reading)
    """
    try:
        compressed = cookie_value.split(".")[1]
        padded = compressed + "=" * (-len(compressed) % 4)
        data = base64.urlsafe_b64decode(padded)
        decompressed = zlib.decompress(data)
        return json.loads(decompressed.decode())
    except Exception as e:
        print("[-] Cookie decode failed:", e)
        return None

def get_session_cookie():
    return session.cookies.get("session")

def start_game():
    r = session.post(f"{BASE_URL}/api/start")
    r.raise_for_status()

def eat(correct_pos, snake):
    payload = {
        "eaten_food_pos": correct_pos,
        "snake_body": snake
    }
    r = session.post(f"{BASE_URL}/api/eat", json=payload)
    r.raise_for_status()
    return r.json()

def solve():
    print("[*] Starting game")
    start_game()

    flag = ""
    snake = [{"x": 0, "y": 0}]  # minimal snake, server doesn’t validate

    while True:
        cookie = get_session_cookie()
        decoded = decode_flask_cookie(cookie)

        if not decoded:
            print("[-] Failed to decode cookie")
            break

        # Extract correct food
        correct_food = decoded.get("correct_food_pos")
        collected = decoded.get("collected_flag", "")
        level = decoded.get("level")

        if collected and collected not in flag:
            flag = collected
            print(f"[+] Flag so far: {flag}")

        if not correct_food:
            print("[-] No correct_food_pos found")
            break

        res = eat(correct_food, snake)

        if res.get("status") == "win":
            print("\n🎉 FINAL FLAG:", res.get("full_flag"))
            break

if __name__ == "__main__":
    solve()
```

## Flag
`BYPASS_CTF{5n4k3_1s_v3ry_l0ng}`

pwn by **W4RR1OR**