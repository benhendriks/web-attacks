SecLists install

```console
sudo apt-get install -y seclists
```

move to

```console
cd /usr/share/seclists/
```

show all

```console
ls -lsa
ls -lsaht
```

Cewl a wordlist generating tool

```console
which cewl
cewl --help
```

generate wordlist with Cewl

```console
sudo cewl -d 2 -m 5 -w ourWordlist.txt www.target.com
ls -lsa ourWordlist.txt
```

show all

```console
cat ourWordlist.txt
```

make use of sed to remove any extra spaces. Lastly, we can make certain there are no duplicate values by piping all output to sort with a parameter of -u indicating that we only want unique results. Naturally, we would like to save our output, so we use the greater-than symbol (>) to pipe all output to binaries-wordlist.txt.

```console
ls -sa /usr/bin | sed 's/[0-9]*//g' | sed -r 's/\s+//g' |sort -u > $HOME/binaries-wordlist.txt
cat $HOME/binaries-wordlist.txt
```

SecLists contains multiple wordlists that cover discovery, fuzzing, common passwords, and web shells. We can install SecLists by cloning the repo to a directory of our choice or using Kali's package manager.

```console
sudo apt install seclists
ls -lh /usr/share/seclists
```

Payloads All The Things is another useful collection of wordlists focused on exploitation. As with SecLists, we can clone the repo from GitHub or install it with Kali's package manager.

```console
sudo apt install payloadsallthethings
ls -lh /usr/share/payloadsallthethings
```

Create custom wordlists

```console
cewl --write output.txt --lowercase -m 4 http://enum-sandbox/manual
tail output.txt
```

we could use ls to get the contents of /usr/bin, pipe the results to grep, use -v / to exclude directories, and redirect the results to a file.

```console
ls /usr/bin | grep -v "/" > binaries.txt
tail binaries.txt
```
