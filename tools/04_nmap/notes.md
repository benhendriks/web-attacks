Export our target IP environment variable into the terminal using theexport command. Because we previously edited our /etc/hosts file to match the IP of 172.16.80.1 :

```console
export IP="172.16.80.1"
echo $IP
```

For our first Nmap scan, let's scan port 80 and examine the output. We will provide the -p parameter to Nmap with a value of 80 to indicate that we are interested in only scanning port 80. Next, we have our $IP parameter, which is our exported IP environment variable.

```console
sudo nmap -p80 $IP
```

Next, we'll rerun the same scan, this time adding the -sV flag. This flag is short for Service Version. Essentially, it instructs Nmap to perform simple banner grabs in an attempt to enumerate the service and service version running on a machine.

```console
sudo nmap -p80 -sV $IP
```

We can get a better idea about how many Nmap NSE scripts are present on Kali Linux by locating them in /usr/share/nmap/scripts.

```console
cd /usr/share/nmap/scripts
ls -lsaht |grep -i 'http'
```

With the http-methods script, we can identify the methods supported by the web-server at the URI provided. Additionally, we can provide a specific URI using the --script-args http-methods.url-path='URI-HERE'

```console
nmap -p80 --script=http-methods --script-args http-methods.url-path='/wp-includes/' $IP
```

Since we know our target (http://target:80/) is running WordPress, we can also use the http-wordpress-enum Nmap NSE script. This script will assist us with enumerating plugins and themes present on the target WordPress machine.

```console
nmap -p80 -sV --script http-wordpress-enum targetwp
```

As previously stated, there are far too many Nmap NSE scripts to cover in a single section. However, which one of them might be useful for retrieving the header information from a target server?

```console
nmap -p 80,443 --script=http-methods,http-ls,http-robots.txt,http-cookie-flags,http-cors example.com
```

If we know our target server is up, we can skip the host discovery phase with the -Pn option.

```console
nmap -Pn enum-sandbox
```

Nmap can also perform service and version detection. There are several options we can configure for this, but the -sV flag for version detection is often sufficient during web application assessments.

```console
nmap -sV enum-sandbox
```

Let's try running a scan of port 80 with the http-methods script. This script will identify what HTTP methods the server has enabled. We'll specify -p 80 to limit the scan to port 80 and --script http-methods.

``c̀onsole
nmap -p 80 --script http-methods enum-sandbox
```

