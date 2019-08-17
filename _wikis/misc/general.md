---
layout: article
title: General
permalink: /wikis/misc/general
aside:
    toc: true
sidebar:
    nav: wikis
---


General

## All

Zip and exclude file extensions
```bash
zip -r <zip-folder-name> dir1/ dir2/ dir3/ -x "*.ext1" -x "*.ext2" -x "*.ext3"
```

## Python

strftime for formatting time in Python
<a href="http://strftime.org/" target="_blank">http://strftime.org/</a>


## Windows CMD
Query name server
```cmd
nslookup www.example.com
nslookup 8.8.8.8
```

Get authoritative name servers
```cmd
nslookup -type=NS example.com
```

Get IPv6 address
```cmd
nslookup -type=AAAA www.example.com
```


## Windows Powershell
Query name server
```
Resolve-DNSName www.example.com
Resolve-DNSName 8.8.8.8
```

Get IPv6 Address
```
Resolve-DNSName -Type AAAA www.example.com
```
