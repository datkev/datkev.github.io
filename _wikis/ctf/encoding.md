---
layout: article
title: Encoding
permalink: /wikis/ctf/encoding
aside:
    toc: true
sidebar:
    nav: ctf
---


Base64<br>
-n (no trailing newline), -e (interpret blackslash escapes)  
```bash
echo -ne <to-encode> | base64 
echo -ne <to-decode> | base64 --decode
```