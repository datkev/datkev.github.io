---
layout: article
title: Nix File Transfer
permalink: /wikis/ctf/nix-file-transfer
aside:
    toc: true
sidebar:
    nav: ctf
---

Grab all files from directory
```bash
wget -r -nH --no-parent http://<host-ip>/<dir>/ --reject="index.html*" -q
```
