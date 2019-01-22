---
layout: article
title: Shell Manipulation
permalink: /wikis/ctf/shell-manipulation
aside:
    toc: true
sidebar:
    nav: wikis
---


In <b>rbash</b>, you can only execute what is in <b>$PATH</b>. <b>$PATH</b> will be read only.<br>
Restricted shell? check <b>$PATH</b>
```bash
echo $PATH 
export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin 
echo os.system("/bin/bash") 
```
<br>
<br>
Python tty and confirmation
```bash
python -c 'import pty; pty.spawn("/bin/sh")' 
tty 
```
<br>
<br>
Spawning tty within a shell 
Background the process using ctrl+z 
```bash
stty raw -echo 
```
Background the process using <code>fg</code> 
```bash
reset 
```
In a different window, check the row and columns <code>stty -a </code> 
In shell, set tty using:
```bash
stty rows <#> cols <#> 
```
<br>
<br>
<b>TERM</b> environment variable not set? 
```bash
export TERM=xterm-color
```
<br>
<br>
<a href="http://www.dest-unreach.org/socat/doc/socat.html#EXAMPLES" target="_blank">socat</a> as a netcat alternative