# OSINT

## Common OSINT Steps for Web Page Assessment

1. **Identify the Target Domain**
   - Determine the main domain and subdomains.
   - Use tools like `whois`, `nslookup`, or `dig`.

2. **Perform DNS Reconnaissance**
   - Enumerate subdomains (e.g., `amass`, `sublist3r`, `crt.sh`).
   - Check for zone transfers.

3. **Check for Publicly Available Information**
   - Search company names, domains, emails via:
     - Google dorks
     - LinkedIn
     - social media platforms
     - public breach data

4. **Enumerate Directories and Files**
   - Use wordlists (`SecLists`) with tools like:
     - `dirb`
     - `dirbuster`
     - `gobuster`
   - Look for hidden directories (`/admin`, `/backup`, `/old`).

5. **Review Page Source and JavaScript**
   - Look for:
     - API endpoints
     - Comments
     - Hidden fields
     - Keys or credentials accidentally embedded

6. **Identify Technologies and Frameworks**
   - Use tools:
     - `whatweb`
     - `wappalyzer`
   - Check versions for known vulnerabilities.

7. **Gather Metadata**
   - Download documents (PDFs, DOCs) to extract metadata (author, paths).
   - Use `exiftool`.

8. **Review Certificates**
   - Use `crt.sh` to find certificates listing subdomains.

9. **Check Robots.txt and Sitemap.xml**
   - Look for disallowed or hidden paths.

10. **Search Vulnerability Databases**
    - Use `CVE` feeds or `exploit-db` for matching known exploits.

11. **Review Historical Content**
    - Use `archive.org` to see old versions of the website.
    - Look for directories or content no longer linked.

12. **Enumerate Open Ports and Services**
    - Perform Nmap scans to see what is exposed externally.

13. **Correlate Findings**
    - Map all discovered assets and paths.
    - Identify potential attack vectors.

---

## Banner grabbing

```bash
sudo nmap -p22,80 --script banner 192.168.45.211 
```

### Example Output 

```shell
PORT   STATE SERVICE
22/tcp open  ssh
 banner: SSH-2.0-OpenSSH_8.4
80/tcp open  http
 banner: HTTP/1.1 200 OK
 Server: Apache/2.4.41 (Ubuntu)
```

# 📘 Nmap Common Options Table

| Command Option          | Description                                                                 |
|------------------------|-----------------------------------------------------------------------------|
| `-p <ports>`           | Specify ports to scan (e.g., `-p80`, `-p22,80`, `-p1-1000`)                  |
| `-p-`                  | Scan all 65,535 TCP ports                                                   |
| `-sV`                  | Detect service versions on open ports                                       |
| `-A`                   | Aggressive scan: includes OS detection, version, script scanning, traceroute |
| `-T4`                  | Set timing template for faster execution (`T0`=slowest, `T5`=fastest)        |
| `-O`                   | Enable OS detection                                                         |
| `-sS`                  | TCP SYN (stealth) scan (default with sudo)                                  |
| `-sT`                  | TCP connect scan (when not run as root)                                     |
| `-sU`                  | UDP scan                                                                    |
| `--script <name>`      | Run specific NSE script(s) (e.g., `--script banner`)                         |
| `--script "default"`   | Run default set of scripts                                                  |
| `--script http-title`  | Get the `<title>` of a web page                                             |
| `-oN <file>`           | Save normal output to a file                                                |
| `-oX <file>`           | Save XML output to a file                                                   |
| `-iL <list.txt>`       | Read list of target IPs from file                                           |
| `-oA <prefix>`         | Save results in all formats (normal, XML, grepable)                         |
| `-Pn`                  | Disable host discovery (treat all hosts as online)                          |
| `-n`                   | No DNS resolution                                                           |
| `-v`, `-vv`            | Increase verbosity level                                                     |

---

