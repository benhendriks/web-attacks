


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


