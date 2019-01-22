---
layout: article
title: Services
permalink: /wikis/ctf/services
aside:
    toc: true
sidebar:
    nav: wikis
---


## Sniffing
<b>tcpdump</b><br>
 - Can edit <b>/etc/hosts</b> for name resolution:<br>
Contents of <b>/etc/hosts</b>:
```bash
10.10.0.100 MYBOX 
10.10.2.20 VULNBOX 
```

```bash
tcpdump -i eth0 -vvv -s 65535 not host 10.11.0.100 -w dump.cap 
```

While sniffing, using netcat we can send SYN packets:
```bash
nc 10.10.2.20 <port> 
```
If we receive R packets in response, port is closed. If S packets are looping to destination and no response from port, filtering is likely the cause. 



## FTP
View hidden files 
```bash
dir –a 
```

Windows NT/2K/XP shortname -- 6 letters followed by ~ and # 
```
docume~1 
progra~1 
```
 


## SSH 
Login using private key file 
```bash
ssh -i <private-key-file> <user>@<server> 
```

Local port forwarding<br>
Binds &lt;local port&gt; to &lt;host port&gt; accessible via &lt;server&gt; 
```bash
ssh -L <local port>:<host>:<host port> <user>@<server> 
```
 

Remote port forwarding<br>
Sends &lt;host&gt;:&lt;host port&gt; to &lt;bind port&gt; on &lt;server&gt; 
```bash
ssh -R <bind port>:<host>:<host port> <user>@<server> 
```
 

## SMTP 
Connect to server 
```bash
telnet <server> 25 
```

Initiate conversation 
```bash
HELO <client-domain> 
MAIL FROM: <user>@<domain.com> 
RCPT TO: <user>@<domain.com> 
```
 


## HTTP 
Brute forcing directories 
```bash
gobuster -u <target-ip> -w /usr/share/seclists/Discovery/Web_Content/common.txt -s '200,204,301,302,307,403,500' 
```

Vulnerability scanning 
```bash
nikto -h <target-ip> 
```

WordPress 
```bash
wpscan --url <target-ip> -e ap --log wpscan.log 
```

WebDAV PUT test 
```bash
davtest -url <target-server> 
```
 
WebDAV Client 
```bash
cadaver <target-server> 
```

Drupal<br>
<a href="https://github.com/droope/droopescan" target="_blank">droopescan<a>
```bash
./droopescan scan drupal -u  <target-server> 
```
 
JSON formatting with jq
```bash
curl -s http://<target-ip>/<file> -d '<param>=<val>' | jq -r ."<param>" 
```
 


## MYSQL 
sqsh login 
```bash
sqsh -S <target-ip> -U sa -P password 
```
 


## SMB 
```bash
smbclient //server/service -U "guest"
```