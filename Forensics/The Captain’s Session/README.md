# The Captain’s Session — BYPASS CTF 

## Challenge Information

| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Forensics |
| **Difficulty** | Medium |
| **Points** | 350 |

## Challenge Description

> The Captain left no open tabs and no saved files.  
> What remains is scattered within the browser itself.

**Flag format:** `BYPASS_CTF{.*}`

---

## Solution

### Step 1: Inspecting the Tar File
- Extract the given tar archive.
- Found a **bookmarks file** inside.
- URL revealed from the bookmarks:   `isdf.dev`

- This domain is considered part of the challenge answer.

---

### Step 2: Inspecting Browser Login Data
- The **Login Data** file is an SQLite database.
- Used Python + `sqlite3` to extract entries:

```python
import sqlite3, os, sys

def open_sqlite_db(path):
  if not os.path.exists(path):
      print(f"[!] File not found: {path}")
      sys.exit(1)
  conn = sqlite3.connect(path)
  return conn

def list_tables(conn):
  cur = conn.cursor()
  cur.execute("SELECT name FROM sqlite_master WHERE type='table';")
  return [r[0] for r in cur.fetchall()]

def dump_table(conn, table):
  cur = conn.cursor()
  cur.execute(f"SELECT * FROM {table}")
  rows = cur.fetchall()
  colnames = [d[0] for d in cur.description]
  print(f"\n=== {table} ===")
  print(colnames)
  for r in rows:
      print(r)

if __name__ == "__main__":
  db_path = "Login Data"
  conn = open_sqlite_db(db_path)
  tables = list_tables(conn)
  print("Found tables:", tables)
  for t in tables:
      dump_table(conn, t)
  conn.close()
  ```
Output revealed the key (note) value:  `stepped_on`

This is the password for the final step.

### Step 3: Inspecting Browser History

Opened the History SQLite database.

Searched for the last occurrence of }.

Extracted the final text forming the flag.

## Flag
`BYPASS_CTF{My_d0g_0st3pp3d_0n_4_b33_shh}`

pwn by **W4RR1OR**