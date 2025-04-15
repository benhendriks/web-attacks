Next, we'll use DIRB, which uses a wordlist to discover files. Most Kali Linux images typically include DIRB. If our particular Kali VM doesn't include it, we can install it with apt.

Let's try running it against our sandbox VM. We'll run dirb followed by the URL to target.

```console
dirb http://enum-sandbox
```

When using a brute forcing tool, selecting the right wordlist can directly impact our results. DIRB's default wordlist is often useful, but we may want to use different wordlists if we don't get many results. Kali Linux includes several wordlists at /usr/share/wordlists.

```console
ls -alh /usr/share/wordlists
```

It's always a good idea to review the options a tool provides to become more familiar with its capabilities.

```console
dirb
```

The -X option appends an extension to each word. In our example, we could append "php" to search for PHP files. We can also provide a list of extensions with -x. This type of functionality allows us to customize our scanning activities to match our target application.

