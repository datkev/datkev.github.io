---
layout: article
title: Reversing with Radare2 - Google CTF 2018 - Part 2
permalink: /page/reversing-with-radare2-part-2
key: page-article-header-overlay-background-image-HB
cover: /assets/images/thumbs/reversing-with-radare2-part-1.png
header:
  theme: dark
  background: 'linear-gradient(135deg, rgb(195, 55, 100), rgb(29, 38, 113))'
article_header:
  type: overlay
  theme: dark
  background_color: '#0F2027'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(195, 55, 100, .6), rgba(29, 38, 113, .6))'
    src: /assets/images/posts/reversing-with-radare2-part-1/cover.png
---

Tackling reverse engineering challenges in Google CTF Beginners Quest with Radare2 (Admin UI 2).

<!--more-->

<style>
  .page__header .header__brand path {
    fill: rgba(255, 255, 255, .95);
  }
</style>


## Admin UI 2

<img src="/assets/images/posts/reversing-with-radare2-part-2/node.png" alt="" class="center" alt="" class="center">
<br>
<br>
<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/70.png" alt="" class="center">

This challenge is a continuation of the previous. The prompt tells us that we might be able to find the password somewhere in the binary file. Let's go back and take a look with <code>radare2</code>.

Using the graph view, we can inspect the function calls for <b>secondary_login__</b>.

```bash
root@kali:~/google-ctf-2018/admin-ui# r2 main 
[0x41414150]> aaa
[x] Analyze all flags starting with sym. and entry0 (aa)
[x] Analyze len bytes of instructions for references (aar)
[x] Analyze function calls (aac)
[ ] [*] Use -AA or aaaa to perform additional experimental analysis.
[x] Constructing a function name for fcn.* and sym.func.* functions (aan))
[0x41414150]> s 0x41414446
[0x41414446]> VV
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/71.png" alt="" class="center">

Notice that obj.FLAG at <b>0x41414a40</b> is being loaded into the address stored in <b>local_90h</b>. Naturally, the rest of the contents of the flag are also being loaded into the adjacent memory addresses. 

We can also observe that there is a point in the function where each byte of the flag is being xor'ed with the lower byte of <b>0xffffffc7</b>.

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/72.png" alt="" class="center">

Let's grab the pre-xor'ed flag.

```bash
[0x41414150]> s 0x41414a40
[0x41414a00]> v
```

<img src="/assets/images/ctfs/google-ctf-beginners-quest-2018/73.png" alt="" class="center">

We can paste the flag in a python script and perform the xor byte-by-byte with <b>0xc7</b> to see what the post-xor'ed contents are.

<b>key.py</b>
```python
#!/usr/bin/python

pre_xor = bytearray("849381bc93b0a89897a6b494b0a8b583bd9885a2b3b3a2b598b3aff3a998f698acf8ba".decode('hex'))
post_xor = ""

for byte in pre_xor:
        post_xor += chr(byte^0xc7)

print(post_xor)
```

```bash
root@kali:~/google-ctf-2018/admin-ui# python key.py
CTF{Two_PasSworDz_Better_th4n_1_k?}
```

<br>
<br>
### Solution
<b>CTF{Two_PasSworDz_Better_th4n_1_k?}</b>


In part 3, we'll look at the final quest in this line, Admin UI 3.