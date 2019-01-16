---
layout: article
title: Port Scanning
permalink: /wikis/ctf/port-scanning
aside:
    toc: true
sidebar:
    nav: ctf
---


<b>nmap aggressive, normal output</b> 
```bash
nmap -A <target-ip> -oN <output-file> 
```
 
<b>nmap default scripts and service scan, normal, xml, grepable output</b>
```bash
nmap -sC -sv <target-ip> -oA  <output-file> 
```

<b>nmap UDP scan with SYN</b>
```bash
nmap -sU -sS <target-ip> 
```
 
<b>unicornscan, log to file</b> 
```bash
unicornscan -Iv <target-ip>:a -w <output-file> 
```

<b>port probing with netcat/b>
```bash
nc -nvv -w 1 -z <target-ip> 1-4000 &> nc-port-scan 
```
