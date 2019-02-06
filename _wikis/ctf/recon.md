---
layout: article
title: Recon
permalink: /wikis/ctf/recon
aside:
    toc: true
sidebar:
    nav: wikis
---

## Port Scanning

<b>nmap aggressive, normal output</b> 
```bash
nmap -A <target-ip> -oN <output-file> 
```
 
<b>nmap default scripts and service scan, normal, xml, grepable output</b>
```bash
nmap -sC -sV <target-ip> -oA  <output-file> 
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


## Host Discovery

<b>ICMP Ping Sweep</b> 
```bash
nmap -sn 192.168.1.1-254 
nmap -sn 192.168.1.0/24 
```
 
<b>ARP Scan</b><br>
Whereas ICMP ping scans can be blocked, all IPv4 devices communicating in on your LAN must respond to Address Resolution Protocol. If the target network resides in a different subnet or WAN, ICMP packets are the only option since ARP Scans do not work outside your subnet.

```bash
arp-scan --interface=tap0 192.168.1.0/24 
netdiscover -i tap0 -r 192.168.1.0/24 
```
 
<b>Reverse DNS Lookup</b>
```bash
host <target-ip> <DNS server>
```
 
<b>OS Discovery</b><br>
OS discovery can be performed with with ping.
- Default Windows TTL = 127
- Default Linux TTL = 64
- Default Cisco TTL = 254 
```bash
ping <target-ip>
```