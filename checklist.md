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

#### NMAP

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

#### CURL

-i or --include = To get the server headers on port 80 of the target
-I or --head = If we only want the headers.

```shell
curl -I http://target
```

Example Output 

```shell
HTTP/1.1 200 OK
Date: Thu, 18 Apr 2024 18:22:37 GMT
Server: Apache/2.4.52 (Ubuntu)
Content-Type: text/html; charset=utf-8
Content-Length: 2546
Vary: Accept-Encoding
```

#### NETCAT

-v = Option to enable verbose output.

```shell
netcat -v target 22
```

```shell
target [192.168.50.108] 22 (ssh) open
SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.6
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

### Discovering Running Services

-Pn = Set -Pn to skip host discover

```shell
nmap -Pn target
```

-Sv = Put the -sV flag for version detection, is often sufficient during web application assessments 

```shell
nmap -sV target
```

--script = We can run scripts as part of our scan by setting the --script option, followed by the script name we want to run.

```shell
nmap -p 80 --script http-methods target
```

---

## Manual HTTP Endpoint Discovery


Web developers use robots.txt to instruct web crawlers on what portions of the website they can crawl. XML sitemaps are an alternative to robots.txt and provide additional information about the site, such as the last time a page was modified.

## Automated HTTP Endpoint Discovery

### hakrawler

```shell
sudo apt install hakrawler
```

```shell
which hakrawler
```

-u = To limit output to unique URLs
-proxy = Option to proxy the requests through another tool
-X = Option appends an extension to each word

```shell
echo "http://target" | hakrawler -u
```

```shell
hakrawler -url https://target.com > output.txt
```

If you want to include subdomains, JavaScript files, etc.:

```shell
hakrawler -url https://target.com -depth 2 -plain > urls.txt
```

Append to an existing file instead of overwriting:

```shell
hakrawler -url https://target.com >> results.txt
```

If you want to sort and remove duplicates in one command:

```shell
hakrawler -url https://target.com | sort -u > clean-urls.txt
```

---

#### Dirb

```shell
dirb http://target
```

Kali Linux includes several wordlists at /usr/share/wordlists

```shell
ls -alh /usr/share/wordlists
```

## Information Disclosure

Oracle database software typically include ORA in error codes.

Create wordlist

```shell
cat users.txt
```

We'll include "foo" as a control element.

```txt
foo
tomjones
tom.jones
t.jones
tom_jones
t_jones
```

```shell
ffuf -w users.txt -u http://target/auth/login -X POST -d 'username=FUZZ&password=bar' -H 'Content-Type: application/x-www-form-urlencoded'
```

## Basic Host Enumeration and OS Detection

```shell
nmap target
```

Response
```shell
Starting Nmap 7.92 ( https://nmap.org ) at 2022-01-18 15:11 EST
Nmap scan report for asio (192.168.150.101)
Host is up (0.059s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
3389/tcp open  ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 5.80 seconds
```

-O = Argument to enable operating system (OS) detection feature. 
-Pn = To disable ping probes because we know the host is up.

```shell
sudo nmap -O -Pn target
```

Response:
```shell
Starting Nmap 7.92 ( https://nmap.org ) at 2022-01-18 15:12 EST
Nmap scan report for asio (192.168.50.131)
Host is up (0.059s latency).
Not shown: 998 filtered tcp ports (no-response)
PORT     STATE SERVICE
80/tcp   open  http
3389/tcp open  ms-wbt-server
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: specialized|general purpose
Running (JUST GUESSING): AVtech embedded (87%), Microsoft Windows XP (85%)
OS CPE: cpe:/o:microsoft:windows_xp::sp3
Aggressive OS guesses: AVtech Room Alert 26W environmental monitor (87%), Microsoft Windows XP SP3 (85%)
No exact OS matches for host (test conditions non-ideal).

OS detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 11.69 seconds
```

Nmap's output warns us that the "results may be unreliable" and the server could be AVtech embedded or Windows. Because we're dealing with a web application, we are more likely dealing with a Microsoft OS than an AVtech Room Alert 26W environmental monitor. However, we'll have to keep multiple operating systems in mind as we develop any exploits until we can definitively determine the correct OS.

## Content Discovery

Our first step will be exploring the web application as a normal user. We'll do this with a browser proxied through Burp Suite so we can capture all requests and responses for later inspection. Let's start Burp Suite, disable Intercept, open the embedded Chromium browser, and browse to the challenge machine.

Before we continue, let's take a moment to review the HTTP History tab in Burp Suite. Web applications will often load resources from multiple domains. These requests can clutter up Burp Suite if we don't filter them out.

...

Now that we've explored all the functionality we can find on the home page, let's use another tool to search for more pages. Let's run gobuster in directory/file enumeration mode with the dir command. We'll specify our target URL with -u and a wordlist with -w.

```shell
gobuster dir -u http://target -w /usr/share/wordlists/dirb/common.txt
```

### Authentication Bypass

If the server is a version of Windows, then C:\windows\win.ini may be present. Some versions of Windows don't include this file, but we should still check for it.

On a Linux server, we would instead first check for /etc/passwd

We'll need to add several instances of "../" to our payload to traverse back to the base directory of the server. In Repeater, let's change "web200.html" to "../../../../../../../windows/win.ini" and click Send.

```shell
nano paths.txt
cat paths.txt
../
../../
../../../
../../../../
../../../../../
../../../../../../
../../../../../../../
```

```shell
nano files.txt
cat files.txt
application.properties
application.yml
config/application.properties
config/application.yml
```

```shell
wfuzz -w paths.txt -w files.txt --hh 0 http://target/specials?menu=FUZZFUZ2Z
```

```shell
curl http://target/specials?menu=../config/application.properties
```



