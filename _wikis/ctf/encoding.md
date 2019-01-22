---
layout: article
title: Encoding
permalink: /wikis/ctf/encoding
aside:
    toc: true
sidebar:
    nav: wikis
---


<a href="https://gchq.github.io/CyberChef/" target="_blank">CyberChef</a> - All-in-one tool

Base64<br>
-n (no trailing newline), -e (interpret blackslash escapes)  
```bash
echo -ne <to-encode> | base64 
echo -ne <to-decode> | base64 --decode
```