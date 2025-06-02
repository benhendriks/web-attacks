
# 🛡️ Web Technology Fingerprinting Cheat Sheet (for Ethical Hacking)

> Educational purposes only. Use this only on systems you have explicit permission to test.

---

## 🔍 1. Basic Recon with `whatweb`

```bash
whatweb https://target.com
```

## 🔎 2. Detailed Tech Stack with `wappalyzer`

Browser extension (for Chrome/Firefox) or use CLI:

```bash
npx wappalyzer https://target.com
```

## 📂 3. HTTP Headers with `curl`

```bash
curl -I https://target.com
```

## 🔎 4. Banner Grabbing with `netcat` or `telnet`

```bash
nc target.com 80
GET / HTTP/1.1
Host: target.com
```

## 🔍 5. DNS & Subdomain Recon

```bash
dig target.com
nslookup target.com
whois target.com
```

### Subdomains:
```bash
sublist3r -d target.com
```

## 🌐 6. Directory and File Bruteforce with `gobuster`

```bash
gobuster dir -u https://target.com -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

## 🔎 7. CMS Detection

```bash
cmseek
wpscan --url https://target.com    # WordPress
droopescan scan drupal -u https://target.com
```

## 🐍 8. Detect Frameworks with `builtwith`

```bash
python3 -m pip install builtwith
builtwith https://target.com
```

---

## 📋 Notes:
- Always check robots.txt:
```bash
curl https://target.com/robots.txt
```

- Review HTML source and JS files for clues:
```bash
curl -s https://target.com | less
```

- Look at favicon hash:
```bash
curl -s https://target.com/favicon.ico | sha256sum
```

---

## ⚠️ Disclaimer
This cheat sheet is intended for **legal penetration testing** or **security research**. Unauthorized access or attacks are illegal and unethical.
