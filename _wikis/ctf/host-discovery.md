---
layout: article
title: Host Discovery
permalink: /wikis/ctf/host-discovery
aside:
    toc: true
sidebar:
    nav: ctf
---

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