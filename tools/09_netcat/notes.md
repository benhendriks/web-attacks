We can use netcat in a similar way when dealing with non-HTTP protocols, such as SSH. We can use the -v option to enable verbose output. When used this way, netcat opens a connection to the destination, so we'll need to actually complete the request or close it using C+c.

Let's run netcat with -v, followed by the host and port values, separately. We'll check port 22 on the enumeration sandbox.

```console
netcat -v enum-sandbox 22
```

Let's connect to it with Netcat by specifying the host and port. After we connect, we'll run whoami and hostname to verify our connection.

```console
netcat enum-sandbox 9999
hostname
hostname
```

to close

```console
ctlr C
```

Next, let's try a reverse shell. Before we create the reverse shell, we need to create a listener to receive it. We'll use Netcat on our Kali VM, set -l to start in listener mode, set -v for verbose, and use -p to specify our port.

```console
netcat -vlp 9090
```



