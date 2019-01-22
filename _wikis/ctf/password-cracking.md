---
layout: article
title: Password Cracking
permalink: /wikis/ctf/password-cracking
aside:
    toc: true
sidebar:
    nav: wikis
---

John the Ripper
```bash
john --format=<format> --wordlist=/usr/share/wordlists/rockyou.txt <file>
```

Password-protected zip archive
```bash
zip2john <zip-file> <file.hash>
john <file.hash>
```

Webpage wordlist generation 
```bash
cewl -d <depth #> -m <min word length #> -w <wordlist.txt> <server> 
```
 
SSH bruteforce 
```bash
hydra -e nsr -l <user> -P <wordlist> <target-ip> ssh -t 4 
```
 
Online MD5 cracker<br>
<a href="https://hashkiller.co.uk/md5-decrypter.aspx" target="_blank">https://hashkiller.co.uk/md5-decrypter.aspx<a>