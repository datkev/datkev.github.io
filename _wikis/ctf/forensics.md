---
layout: article
title: Forensics
permalink: /wikis/ctf/forensics
aside:
    toc: true
sidebar:
    nav: wikis
---

Check executable properties with <a href = "https://github.com/slimm609/checksec.sh" target="_blank">checksec</a>
```bash
checksec --file <file>
```

Extract metadata from file
```bash
exiftool <file>
```

Dumping text from pdf
```bash
pdftotext <file>
```

Find and extract embedded files
```bash
binwalk <file>
binwalk -e <file>
```

Mounting a filesystem
```bash
mount <block_device> </mount/point>
```

