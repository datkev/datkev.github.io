---
layout: article
title: Git
permalink: /wikis/misc/security
aside:
    toc: true
sidebar:
    nav: wikis
---


Security

## Cryptography

Generate RSA private key
```bash
openssl genrsa -aes256 -out <privname>.key 2048
```

Generate certificate signing request for certificate authority to sign
```bash
openssl req -new -key <privname>.key -out <name>.csr
```

Check certificate details
```bash
openssl x509 -in <certname>.cer -text
```

## Searchsploit

Examine exploit or script
```bash
searchsploit -x <dir>
```

Copy exploit to clipboard
```bash
searchsploit -m <dir>
```

## SMB Server

Launch SMB server with no password in current directory. Useful for transferring files from remote to local.
```bash
impacket-smbserver -smb2support <share_name> $(pwd)
```

## SSH

Generate public key from private key file
```bash
ssh-keygen -y -f <privname>.pem
```

Fingerprinting, different formats
```bash
ssh-keygen -l -f <pubname>.pub

ssh-keygen -l -E md5 <pubname>.pub
```