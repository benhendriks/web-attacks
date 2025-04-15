Before we install Hakrawler, we will make certain that our apt is up-to-date. To do this, we can issue the apt update -y command.

```console
sudo apt update -y
```

install

```console
sudo apt update -y
sudo apt install hakrawler
which hakrawler
hakrawler --help
```

Now that we have installed Hakrawler, let's run it against our sandbox VM. Instead of passing our target URL as a parameter, we need to pipe it to hakrawler. We'll also set -u to limit output to unique URLs.

```console
echo "http://enum-sandbox" | hakrawler -u
```

By default, Hakrawler will print out the URLs it discovered. We could redirect the output to a file to save the results. We could also run Hakrawler with the -proxy option to proxy the requests through another tool.

We cat the information stored in urls.txt in the same directory where we created it. Then, we pipe those URLs to be handled by Hakrawler and finally review the output below.

```console
echo "https://www.target.com/" > urls.txt
cat urls.txt |./hakrawler
```

