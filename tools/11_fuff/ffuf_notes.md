# 🚀 FFUF Notes

## 📌 Command

```bash
ffuf -w exercise.txt -u http://enum-sandbox/auth/login -X POST -d 'username=r.jones&password=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded'
```

---

## 🧰 Tool Description

**FFUF** (Fuzz Faster U Fool) is a fast web fuzzer written in Go. It is commonly used for:

- Web content discovery
- Parameter fuzzing
- Authentication and form fuzzing
- API endpoint testing

---

## 🔍 Command Breakdown

| Option / Argument                                   | Description |
|----------------------------------------------------|-------------|
| `ffuf`                                             | Launches the FFUF tool. |
| `-w exercise.txt`                                  | Wordlist file containing words to fuzz (from `cewl` in this case). |
| `-u http://enum-sandbox/auth/login`                | Target URL for the fuzzing attack. |
| `-X POST`                                          | Use the POST HTTP method instead of the default GET. |
| `-d 'username=r.jones&password=FUZZ'`              | POST data being sent. `FUZZ` is the placeholder that gets replaced with each word from the wordlist. |
| `-H 'Content-Type: application/x-www-form-urlencoded'` | Sets the appropriate content-type for form submissions. |

---

## 💡 Purpose

This command attempts to brute-force the password field for the user `r.jones` on a login page:

- It sends a POST request to the login URL.
- Each request replaces `FUZZ` with a word from `exercise.txt`.
- Useful for password spraying or login fuzzing.

---

## 📝 Example Wordlist (`exercise.txt`)

```text
password123
welcome
admin2024
r.jones123
securelogin
```

FFUF will try each of these as the password in the POST data.

---

## 🧠 What Happens

For each word in `exercise.txt`, FFUF sends:

```
POST http://enum-sandbox/auth/login
Content-Type: application/x-www-form-urlencoded

username=r.jones&password=<word>
```

---

## ⚠️ Tips & Considerations

- Use `-fc` or `-mc` flags to filter or match based on HTTP response codes.
- Use `-fs` or `-ml` to filter based on response size or length.
- Combine with proxy tools like Burp Suite for better visibility.
- Make sure you have permission to fuzz the target server.

---

## ✅ Usage Scenario

This command is ideal when:

- You have a known username but unknown password.
- You generated a context-specific wordlist (e.g., using `cewl`).
- You are testing authentication resilience against brute-force attempts.

