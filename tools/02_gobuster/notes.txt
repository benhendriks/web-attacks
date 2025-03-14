install

sudo apt install -y gobuster
gobuster --help

export a URL environment variable to target with Gobuster

export URL="http://target:80/"
echo$URL

using Gobuster to discover file and directory endpoints. We'll set dir as our mode for directory and file enumeration, -w for our wordlist, and -t to run the scan with only 5 threads. We'll also provide the parameter -b 301 for a status-code blocklist so our attempt is not interrupted by redirects. For this example, we'll use the /usr/share/wordlists/dirb/common.txt wordlist on Kali Linux.

gobuster dir -u $URL -w /usr/share/wordlists/dirb/common.txt -t 5 -b 301

example subdomain for the targetcorpone.com domain might be mail.targetcorpone.com. During our testing, we can specify this subdomain for using the -d parameter.

gobuster dns -d targetcorpone.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 30


