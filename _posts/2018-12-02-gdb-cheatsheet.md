---
layout: article
title: GNU Debugging Cheatsheet
permalink: /page/gdb-cheatsheet
key: page-aside
aside:
  toc: true
---

A quick-reference for some handy commands.

<!--more-->


## Compiling
### gcc flags
Produce debugging info in OS native format<br>
<code>gcc -g <i>&lt;source&gt;</i> -o <i>&lt;binary&gt;</i></code>

Expressive debugging format for gdb<br>
<code>gcc -ggdb <i>&lt;source&gt;</i> -o <i>&lt;binary&gt;</i></code>

## Basics
### objdump
Disassemble using objdump<br>
<code>objdump -d <i>&lt;binary&gt;</i></code>

Set intel mode<br>
<code>objdump -d <i>&lt;binary&gt;</i> -M intel</code>

### gdb Program Flow
Print program<br>
<code>(gdb) l</code> and press [enter] to continue printing

Run a program<br>
<code>(gdb) run <i>&lt;args&gt;</i></code>

Disassemble function<br>
<code>(gdb) disassemble <i>&lt;function&gt;</i></code>

Set breakpoint at function<br>
<code>(gdb) b <i>&lt;function&gt;</i></code><br>
<code>(gdb) b *<i>&lt;address&gt;</i></code><br>
e.g.<code>(gdb) b *0x08084bd</code><br>

List breakpoints<br>
<code>(gdb) info breakpoints</code>

Enable breakpoint<br>
<code>(gdb) enable <i>&lt;breakpoint #&gt;</i></code>

Disable breakpoint<br>
<code>(gdb) disable <i>&lt;breakpoint #&gt;</i></code>

Find entry point address without symbols using <b>readelf</b><br>
<code>(gdb) shell readelf -h <i>&lt;binary&gt;</i></code> then set breakpoint at entry address

Set conditional breakpoint<br>
<code>(gdb) condition <i>&lt;breakpoint #&gt;</i>  <i>&lt;condition&gt;</i></code><br>
e.g. <code>(gdb) condition 1 user_input == 'yes'</code> stops program only if <code>user_input</code> == 'yes'<br>
e.g. <code>(gdb) condition 1 $eax != 0</code>

Continue<br>
<code>(gdb) c</code>

Step one instruction<br>
<code>(gdb) stepi <i>[N]</i></code> where <i>N</i> is the number of times to step

Step until next source line<br>
<code>(gdb) step <i>[N]</i></code> where <i>N</i> is the number of times to step

Examine help<br>
<code>(gdb) help x</code>

Examine instruction<br>
<code>(gdb) x/i <i>&lt;address&gt;</i></code><br>
e.g. <code>(gdb) x/10i 0x80484bd</code>

Examine and print <i>N</i> hex (x) words at address<br>
<code>(gdb) x/10xw <i>&lt;address&gt;</i></code><br>
e.g. <code>(gdb) x/10xw $esp</code><br>
e.g. <code>(gdb) x/1xw argv</code>

### Hooks
From the gdb manual:<br>
>Defining (`hook-stop') makes the associated commands execute every time execution stops in your program: before breakpoint commands are run, displays are printed, or the stack frame is printed.

Define hook-stop. The following example prints contents of eax, ebx, ecx, 8 hex bytes at address of some_var and disassembles address at eip to eip+10:<br>
<code>(gdb) define hook-stop</code><br>
<code>(gdb) print $eax</code><br>
<code>(gdb) print $ebx</code><br>
<code>(gdb) print $ecx</code><br>
<code>(gdb) print x/8xb &some_var</code><br>
<code>(gdb) disas $eip,+10</code><br>
<code>(gdb) end</code>

### Disassembling
Set disassembly flavor<br>
<code>(gdb) set disassembly-flavor <i>&lt;style&gt;</i></code><br>
e.g. <code>(gdb) set disassembly-flavor att</code><br>
e.g. <code>(gdb) set disassembly-flavor intel</code><br>

<code>(gdb) show disassembly-flavor</code>

Disassemble a function<br>
<code>(gdb) disas <i>&lt;function&gt;</i></code>

### Modifying Registers and Memory
Modifying memory with <b>set</b> at runtime<br>
<code>(gdb) set {<i>type</i>} <i>&lt;address&gt;</i> = '<i>value</i>'</code><br>
e.g. <code>(gdb) set sum = 2000</code> sets variable <code>sum</code> to 2000<br>
e.g. <code>(gdb) set $eax = 10</code><br>
e.g. <code>(gdb) set {char} 0xbffff7e6 = 'B'</code><br>
e.g. <code>(gdb) set {char} (0xbffff7e6 + 1) = 'B'</code>

### gdb Debugging with Symbols
List global/static variables<br>
<code>info variables</code>

List local variables<br>
<code>info scope <i>&lt;function_name&gt;</i></code>

List functions<br>
<code>info functions</code>

Dump symbols<br>
<code>maint print symbols <i>&lt;filename_to_store&gt;</i></code>

## Convenience Variables and Calling Routines
### Convenience Variables
From the GDB manual:<br>
>GDB provides convenience variables that you can use within GDB to hold on to a value and refer to it later. These variables exist entirely within GDB; they are not part of your program, and setting a convenience variable has no direct effect on further execution of your program. That is why you can use them freely.

>Convenience variables are prefixed with '$'. Any name preceded by '$' can be used for a convenience variable, unless it is one of the predefined machine-specific register names.

<code>(gdb) set $<i>var_name</i> = <i>value</i></code><br>
e.g. <code>(gdb) set $foo = 10</code><br>
e.g. <code>(gdb) set $foo = *object_ptr</code> would save in $foo the value contained in the object pointed to by object_ptr.

### Calling Routines
<code>(gdb) call <i>function(args)</i></code><br>
e.g. <code>(gdb) call AddNumbers($i, $j)</code>


## Symbols
### Adding and Removing Debugging Symbols
Copy symbol files from binary into new file<br>
<code>objcopy --only-keep-debug <i>&lt;rip_from_binary&gt;</i> <i>&lt;debug_file&gt;</i></code>

Strip debugging symbols off binary<br>
<code>strip --strip-debug <i>&lt;binary_to_strip&gt;</i></code>

Strip all symbols not needed for relocation processing off binary (better for security)<br>
<code>strip --strip-unneeded <i>&lt;binary_to_strip&gt;</i></code>

Add debug symbols to binary using objdump<br>
<code>objcopy --add-gnu-debuglink=<i>&lt;symbol_file&gt;</i> <i>&lt;binary&gt;</i></code>

Add debug symbols to a binary using symbol-file within gdb<br>
<code>(gdb) symbol-file <i>&lt;file_name&gt;</i></code>

### Inspecting Symbols with nm
Good for a quick overview of an executable.

<a href="https://linux.die.net/man/1/nm" target="_blank">nm</a> lists symbols from object files<br>
<code>nm ./<i>&lt;binary&gt;</i></code>

<b>Symbol Types:</b><br>
Generally, upper case is a global symbol and lower case is a local symbol.<br>

| Symbol type | Meaning |
|:-----------:|:-------|
|      A      | The symbol's value is absolute, and will not be changed by further linking.  |
|      B      | The symbol is in the uninitialized data section (.bss).  |
|      D      | The symbol is in the initialized data section (.data).|
|      N      | Debugging symbol |
|      T      | The symbol is in the text code section (.text). |
|      U      | The symbol is undefined. |
|      u      | The symbol is a unique global symbol. This is a GNU extension to the standard set of ELF symbol bindings. For such a symbol the dynamic linker will make sure that in the entire process there is just one symbol with this name and type in use. |

### Practical nm
Display all symbols, even debugger-only symbols (normally not listed)<br>
<code>nm -a ./<i>&lt;binary&gt;</i></code>

Search for function in symbols<br>
<code>nm ./<i>&lt;binary&gt;</i> | grep <i>&lt;function_name&gt;</i></code>

Search for symbol type in symbols<br>
<code>nm ./<i>&lt;binary&gt;</i> | grep ' <i>&lt;letter&gt;</i></code> '

Sort symbols by address<br>
<code>nm -n ./<i>&lt;binary&gt;</i></code>

Print global/external symbols<br>
<code>nm -g ./<i>&lt;binary&gt;</i></code>

Print value and size of defined symbols<br>
<code>nm -S ./<i>&lt;binary&gt;</i></code>

## System Calls
### strace
<code>strace</code> helps us understand how the program interacts with the OS by tracing all system calls made by the program. It can tell us about arguments passed and has great filtering capabilities among others.

Trace execution<br>
<code>strace ./<i>&lt;executable_to_trace&gt;</i></code><br>
<code>-o <i>&lt;output_file&gt;</i></code><br>
<code>-t</code> for timestamp<br>
<code>-r</code> for relative timestamp

Trace a specific system call<br>
<code>strace -e <i>&lt;expression&gt;</i> ./<i>&lt;executable_to_trace&gt;</i></code><br>
* e.g. <code>strace -e write ./<i>&lt;executable_to_trace&gt;</i></code><br>

Attach strace to a running process<br>
<code>strace -e <i>&lt;pid&gt;</i></code>

Get statistics on system calls<br>
<code>strace -c <i>&lt;executable&gt;</i> <i>&lt;args&gt;</i></code>





