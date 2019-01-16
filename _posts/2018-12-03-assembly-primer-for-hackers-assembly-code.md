---
layout: article
title: Assembly Primer for Hackers Assembly Code Repo
permalink: /page/assembly-primer-for-hackers-source-code
key: page-aside
aside:
  toc: true
---

Code repository for the Assembly Primer for Hackers YouTube series.

<!--more-->

<a href="https://www.youtube.com/playlist?list=PLue5IPmkmZ-P1pDbF3vSQtuNquX0SZHpB" target="_blank">Assembly Primer for Hackers</a> is a short YouTube series put out by security researcher Vivek Ramachandran. It aims to teach basic assembly to those interested in exploring exploit development and reverse engineering.

Below you'll find the source code he uses in his gdb walkthroughs. The code should allow you to follow along with the videos without interruption. It is recommended to go back and manually create a copy of each source file on your machine before compiling and running through gdb.

If you will be spending time in gdb, <a href="https://github.com/longld/peda" target="_blank">PEDA - Python Exploit Development Assistance for GDB</a> may be a tool worth investigating.

## Quick Reference Register Board
<img src="/assets/images/general/register-table.png" alt="x86-32-register-board" class="center">

## Part 1 - System Organization
### CPU Registers
* **EIP**: Instruction Pointer Register
* **EAX**: Accumulator Register - stores operands and result data
* **EBX**: Base Register - pointer to data
* **ECX**: Counter Register - loop and string operations
* **EDX**: Data Register - I/O Pointer
* **ESI**: Data Pointer Register for memory operations, Source Index
* **EDI**: Data Pointer Register for memory operations, Destination Index
* **ESP**: Stack Pointer Register
* **EBP**: Stack Data Pointer Register
* Segment Registers include:
  - **CS**: Code Segment
  - **DS** Data Segment
  - **SS** Stack Segment
  - **ES**, **FS**, **GS** are generally pointers used for other segments

### Program Memory
* **Stack**: Used for storing function arguments and local variables
* **Unused Memory**
* **Heap**: Dynamic Memory - malloc()
* **.bss**: Uninitialized Data
* **.data**: Initialized Data
* **.text**: Program Code


## Part 2 - Virtual Memory Organization
Shows assignments of virtual memory<br>
`cat /proc/<PID>/maps`

Check ASLR settings - 0 (disabled), 1 (conservative randomization), or 2 (full randomization)<br>
`cat /proc/sys/kernel/randomize_va_space` 

<a href="https://askubuntu.com/questions/318315/how-can-i-temporarily-disable-aslr-address-space-layout-randomization" target="_blank">Disabling ASLR</a><br>
Linux >= 2.6.12 enables ASLR by default

### SimpleDemo.c
```c
#include <stdio.h>
#include <stdlib.h>

int add(int x, int y)
{
	int z = 10;
	z = x + y;
	return z;
}

main(int argc, char **argv)
{
	int a = atoi(argv[1]);
	int b = atoi(argv[2]);
	int c;
	char buffer[100];

	gets(buffer);
	puts(buffer);

	c = add(a,b);

	printf("Sum of %d+%d = %d\n", a, b, c);

	exit(0);
}
```

## Part 4 - Hello World
### Linux System Calls
* The "library" which the kernel exposes to get various tasks done (e.g. exit(), read(), write())
* List of system calls available in /usr/include/asm/\<header\> 
* Ask the compiler directly with <code>echo -n "#include \<sys/syscall.h\>\nSYS_read" | gcc -E -</code>
* System calls invoked by processes using a software interrupt: `int 0x80`
* Invoking Linux syscalls requires passing appropriate arguments
  * EAX - system call number, EBX - first argument, ECX - second argument EDX - third argument, ESI - fourth argument, EDI - fifth argument

### Example of exit() in Assembly
* Calling exit(0) to exit a program
* <code>man 2 exit</code>
* Function definition: <code>void _exit(int <i>status</i>)</code>
1. syscall number for exit() is 1, so load EAX with 1<br>
  `movl $1, %eax`
2. If "status" is "0" - EBX must be loaded with "0"<br>
  `movl $0, %ebx`
3. software interrupt<br>
  `int 0x80`

### HelloWorld in Assembly
* Use write() syscall to print "Hello World"
* exit() to gracefully exit program
* <code>man 2 write</code>
* write() syscall description:<br>
  <code>ssize_t write(int <i>fd</i>, const void <i>buf</i>, size_t <i>count</i>);</code>
* syscall number for write() is 4 (store in EAX), find this in /usr/include/asm/\<header\>
* fd = 1 for STDOUT (in EBX)
* buf = pointer to memory location containing "Hello World" string (in ECX)
* count = string length (in EDX)

About as complex as it gets:
### JustExit.s
```nasm
.text

.global _start

_start:
	movl $1, %eax
	movl $0, %ebx
	int $0x80
```

Named `HelloWorld.s` instead of `HelloWorldProgram.s`.

### HelloWorld.s
```nasm
.data

HelloWorldString: 
    .ascii "Hello World\n"

.text

.globl _start

_start:
    # ssize_T write(int fd, const void *buf, size_t count)
    # load all the arguments for write()
    # syscall number for write() is 4 (store in EAX)
    # fd = 1 for STDOUT (in EBX)
    # buf = pointer to memory location containing "Hello World" string (in ECX)
    # count = string length (in EDX)
    
    movl $4, %eax
    movl $1, %ebx
    movl $HelloWorldString, %ecx
    movl $12, %edx
    int $0x80

    # exit the program with exit syscall()
    movl $1, %eax
    movl $0, %ebx
    int $0x80
```

## Part 5 - Data Types
### Data Types in .data
<i>Space is reserved at compile time.</i>
* **.byte** = 1 byte
* **.ascii** = string
* **.asciz** = null terminated string
* **.int** = 32 bit integer
* **.short** = 16 bit integer
* **.float** = single precision floating point number
* **.double** = double precision floating point number

### Data Types in .bss
<i>Space is created at runtime.</i>
* **.comm** - declares common memory area
* **.lcomm** - declares local common memory area
<br>

### VariableDemo.s
```nasm
# Demo program to show how to use Data types and MOVx instructions

.data
	HelloWorld:
		.ascii "Hello World!"

	ByteLocation:
		.byte 10

	Int32:
		.int 2

	Int16:
		.short 3

	Float:
		.float 10.23

	IntegerArray:
		.int 10,20,30,40,50

.bss
	.comm LargeBuffer, 10000

.text
	.globl _start

	_start:
		nop
		# Exit syscall to exit the program

		movl $1, %eax
		movl $0, %ebx
		int $0x80
```

## Part 6 - Moving Data
### Moving Data
1. Between registers:<br>
  <code>movl %eax, %ebx</code>
2. Between registers and memory:<br>
  <code>movl %eax, <i>location</i></code><br>
  <code>movl <i>location</i>, %ebx</code>
3. Immediate value into register:<br>
  <code>movl $10, %ebx</code>
4. Immediate value into memory location:<br>
  <code>movb $10, <i>location</i></code>
5. Into an Index Memory Location:<br>
  IntegerArray: .int 10, 20, 30, 40, 50<br>
  Selecting the 3rd integer "30":<br>
  <code>BaseAddress(Offset, Index, Size) = IntegerArray(0, 2, 4)</code><br>
  <code>movl %eax, IntegerArray(0, 2, 4)</code>

### Indirect Addressing using Registers
Placing the "$" sign before a label name takes the memory address of the variable and not the value<br>
<code>movl $location, %edi</code><br>
Place value "9" in memory location pointed to by EDI:<br>
<code>movl $9, (%edi)</code><br>
Place value "9" in memory location pointed to by EDI+4:<br>
<code>movb $10, 4(%edi)</code><br>
Place value "9" in memory location pointed to by EDI-2:<br>
<code>movb $10, -2(%edi)</code>


### VariableDemo.s
```nasm
# Demo program to show how to use Data types and MOVx instructions

.data
	HelloWorld:
		.ascii "Hello World!"

	ByteLocation:
		.byte 10

	Int32:
		.int 2

	Int16:
		.short 3

	Float:
		.float 10.23

	IntegerArray:
		.int 10,20,30,40,50


.bss
	.comm LargeBuffer, 10000

.text
	.globl _start

	_start:
		nop
		
		# 1. MOV immediate value into register
		movl $10, %eax

		# 2. MOV immediate value into memory location
		movw $50, Int16
		
		# 3. MOV immediate data between registers
		movl %eax, %ebx
		
		# 4. MOV data from memory into register
		movl Int32, %eax	
	
		# 5. MOV data from register to memory
		movb $3, %al
		movb %al, ByteLocation
			
		# 6. MOV data into an indexed memory location
		# Location is decided by BaseAddress(offset, index, data size)
		# Offset and Index must be registers, Data Size can be a numerical value
		
		movl $0, %ecx
		movl $2, %edi
		movl $22, IntegerArray(%ecx, %edi, 4)


		#7. Indirect addressing using registers
		
		movl $Int32, %eax
		movl (%eax), %ebx

		movl $9, (%eax)

		# Exit syscall to exit the program

		movl $1, %eax
		movl $2, %ebx
		int $0x80
```

## Part 7 - Working with Strings

### Working with Strings in Assembly
Moving strings from one memory location to another with <b>movsx</b>:<br>
<code>movsb</code> - move a byte (8 bits)<br>
<code>movsw</code> - move a word (16 bits)<br>
<code>movsl</code> - move a double word (32 bits)<br>
Source - ESI points to memory location<br>
Destination - EDI points to memory location

### The DF Flag
* Direction Flag (DF) is part of the <a href="(https://www.youtube.com/playlist?list=PLue5IPmkmZ-P1pDbF3vSQtuNquX0SZHpB" target="_blank">EFLAGS</a> registers
* Decides whether to increment/decrement ESI, EDI registers after a movsx instruction
* If DF is set (1), then ESI, EDI are decremented
* If DF is cleared (0), then ESI, EDI are incremented
* You can set DF using the <code>std</code> instruction
* You can clear DF using the <code>cld</code> instruction
 
### The REP Instruction
* Used to repeat a string instruction using ECX as a counter
* Simple use:
  * Load the ECX register with the string length
  * Use the <code>rep movsx</code> instruction to copy the string from source to destination

### Loading Strings from Memory into Registers
* Loading Strings
  * Loads into the EAX register
  * String source pointed to by ESI
* <b>lodsx</b>
  * <code>lodsb</code> - load a byte from memory location into al<br>
  * <code>lodsb</code> - load a word from memory location into ax<br>
  * <code>lodsb</code> - load a double word from memory location into eax<br>
* ESI is automatically incremented/decremented based on DF flag after the lodsx instruction executes

### Storing Strings from Memory into Registers
* Storing Strings
  * Stores into memory from EAX
  * EDI point to destination memory
* <b>stosx</b>
  * <code>stosb</code> - store al to memory<br>
  * <code>stosw</code> - store ax to memory<br>
  * <code>stosl</code> - store eax to memory<br>

### Comparing Strings
* Comparing Strings
  * ESI contains source string, EDI the destination string
  * DF flag decides whether ESI/EDI are incremented/decremented
* <b>cmpsx</b>
  * <code>compsb</code> - compares byte value<br>
  * <code>compsw</code> - compares word value<br>
  * <code>compsl</code> - compares double word value<br>
* cmpsx subtracst the destination string from the source string and sets the EFLAGS register appropriately

### REPZ and REPNZ
* REPZ - repeat instruction while zero flag is set
* REPZ - repeat instruction while zero flag not set


### StringBasics.s
```nasm
.data
	HelloWorldString:
		.asciz "Hello World of Assembly!"
	H3110:
		.asciz "H3110"

.bss
	.lcomm Destination, 100
	.lcomm DestinationUsingRep, 100
	.lcomm DestinationUsingStos, 100

.text
	.globl _start

	_start:
		nop

		# 1. Simple copying using movsb, movsw, movsl

		movl $HelloWorldString, %esi
		movl $Destination, %edi

		movsb
		movsw
		movsl

		# 2. Getting/Clearing the DF flag

		std # set the DF flag
		cld # clear the DF flag

		# 3. Using Rep
		
		movl $HelloWorldString, %esi
		movl $DestinationUsingRep, %edi
		movl $25, %ecx    # set the string length in ECX
		cld    # clear the DF
		rep movsb
		std

		# 4. Loading string from meory into EAX register

		cld
		leal HelloWorldString, %esi
		lodsb
		movb $0, %al

		dec %esi
		lodsw
		movw $0, %ax

		subl $2, %esi    # Make ESI point back to the original string
		lodsl

		# 5. Storing strings from EAX to memory

		leal DestinationUsingStos, %edi
		stosb
		stosw
		stosl

		# 6. Comparing Strings

		cld
		leal HelloWorldString, %esi
		leal H3110, %edi
		cmpsb

		dec %esi
		dec %edi
		cmpsw

		subl $2, %esi
		subl $2, %edi
		cmpsl

		# The exit() routine
		
		movl $1, %eax
		movl $10, %ebx
		int $0x80
```


## Part 8 - Unconditional Branching
### Program Execution Flow
* jmp: 
  * Compare it with <code>goto</code> statement in C
  * Syntax - jmp *label*
  * Short, near, and far jmp possible
* call:
  * Just like calling a function in C
  * Syntax - call *location*
  * There is an associated <code>ret</code> statement with every <code>call</code>
    * Compare with <code>return</code> in C
  * Using <code>call</code> pushes next instruction address onto the stack
    * This instruction is popped back into EIP on hitting the <code>ret</code> instruction

### UnconditionalBranching.s
```nasm
.data
    HelloWorld:
        .asciz "Hello World!"
    
    CallDemo:
        .asciz "Call works!"

.text
    .globl _start

    _start:
        # nop
        # call CallMe
        # jmp ExitProgram
        
        # Write HelloWorld
        movl $4, %eax
        movl $1, %ebx
        movl $HelloWorld, %ecx
        movl $12, %edx
        int $0x80

    ExitProgram:
        # Exit the program
        movl $1, %eax
        movl $10, %ebx
        int $0x80

    CallMe:
        movl $4, %eax
        movl $1, %ebx
        movl $CallDemo, %ecx
        movl $11, %edx
        int $0x80
        ret
```

## Part 9 - Conditional Branching
### Conditional Branching
* JXX - JA, JAE, JE, JG, JZ, JNZ ... etc
* Dictated by the state of the
  - Zero flag (ZF)
  - Parity flag (PF)
  - Overflow flag (OF)
  - Sign Flag (SF)
  - Carry Flag (CF)
* In order to use conditional Jumps you must have an operation which sets the EFLAGs register appropriately
* In conditional jumps - only Short and Near jumps are supported. Far jumps are not supported

### Loop Instruction
* Loops through set of instruction a predetermined number of times
* ECX used as counter and is decremented
* Usage:
  ```nasm
  <code>

  movl $10, %ecx    # counter set to 10
  LoopThis:
    <code>
    loop LoopThis
  ```

### Conditional Loops
* <code>loopz</code> - loop until ECX is not zero or the zero flag (ZF) is not set
* <code>loopnz</code> - loop until ECX is not zero or the zero flag (ZF) is set

### ConditionalBranching.s
```nasm
.data
    HelloWorld:
        .asciz "Hello World!\n"
    ZeroFlagSet:
        .asciz "Zero Flag was set!"
    ZeroFlagNotSet:
        .asciz "Zero Flag not set!"

.text
    .globl _start
    
    _start:
        nop
        movl $10, %eax
        # xorl %eax, %eax
        jz FlagSetPrint
        # jz PrintHelloWorld

    FlagNotSetPrint:
        movl $4, %eax
        movl $1, %ebx
        leal ZeroFlagNotSet, %ecx
        movl $19, %edx
        int $0x80
        jmp ExitCall

    FlagSetPrint:
        movl $4, %eax
        movl $1, %ebx
        leal ZeroFlagSet, %ecx
        movl $19, %edx
        int $0x80
        jmp ExitCall

    ExitCall:
        movl $1, %eax
        movl $0, %ebx
        int $0x80

    PrintHelloWorld:
        movl $10, %ecx
        PrintTenTimes:
            pushl %ecx
            movl $4, %eax
            movl $1, %ebx
            leal HelloWorld, %ecx
            movl $14, %edx
            int $0x80
            popl %ecx
        loop PrintTenTimes
        jmp ExitCall
```

## Part 10 - Functions
### Functions in Assembly
* Defining a function in Assembly:
  ```nasm
  .type MyFunction, @function

  MyFunction:
    <code>
    <code>
    ret
  ```
* Function invokes using <code>call MyFunction</code>

### Passing Arguments and Returning Values
* Passing arguments to functions
  * Could use registers and load arguments before calling function
  * Use global memory location, e.g. define memory in .bss or .data segment
  * Pass argument over the stack
* Returning values from functions
  * Return values in registers
  * Return values in global memory locations

### Function.s
```nasm
.data
    HelloWorld:
        .asciz "Hello World!\n"
    HelloFunction:
        .asciz "Hello Function!\n"

.text
    .global _start

    .type MyFunction, @function

    MyFunction:    # String pointer and len to be added by caller
        movl $4, %eax
        movl $1, %ebx
        int $0x80
        ret

    _start:
        nop
        
        # Print Hello World
        leal HelloWorld, %ecx
        movl $14, %edx
        call MyFunction

        # Print Hello World Function
        leal HelloFunction, %ecx
        movl $17, %edx
        call MyFunction

        # Exit Routine

    ExitCall:
        movl $1, %eax
        movl $0, %ebx
        int $0x80
```

### Function2.s
```nasm
.data
    HelloWorld:
        .asciz "Hello World!\n"
    HelloFunction:
        .asciz "Hello Function!\n"

.bss
    .lcomm StringPointer, 4
    .lcomm StringLength, 4

.text
    .global _start

    .type MyFunction, @function

    MyFunction:    # String pointer and len to be added by caller
        movl $4, %eax
        movl $1, %ebx
        movl StringPointer, %ecx
        movl StringLength, %edx
        int $0x80
        ret

    _start:
        nop
        
        # Print Hello World
        movl $HelloWorld, StringPointer
        movl $14, StringLength
        call MyFunction

        # Print Hello World Function
        movl $HelloFunction, StringPointer
        movl $17, StringLength
        call MyFunction

        # Exit Routine

    ExitCall:
        movl $1, %eax
        movl $0, %ebx
        int $0x80
```

### Function3.s
```nasm
.data
    HelloWorld:
        .asciz "Hello World!\n"

.text
    .globl _start
    .type PrintFunction, @function

    PrintFunction:
        pushl %ebp    # store current value of EBP onto stack
        movl %esp, %ebp    # Make EBP point to top of stack
        
        # The write function
        movl $4, %eax
        movl $1, %ebx
        movl 8(%ebp), %ecx    # copy pointer to HelloWorld str to ECX
        movl 12(%ebp), %edx    # copy strlen to EDX
        int $0x80

        movl %ebp, %esp    # Restore the old value of ESP
        popl %ebp    # Restore the old value of EBP
        ret    # change EIP to start of next instruction

    _start:
        nop
        
        # push the strlen onto the stack
        pushl $13
        
        # push string pointer onto the stack
        pushl $HelloWorld

        # call the function
        call PrintFunction

        # adjust the stack pointer
        addl $8, %esp

        # Exit Routine
        ExitCall:
        movl $1, %eax
        int $0x80
```
