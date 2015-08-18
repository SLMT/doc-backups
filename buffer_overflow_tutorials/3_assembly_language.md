# Assembly Language

Some knowledge of assembly is necessary in order to understand the operation of the buffer overflow exploits. There are essentially three kinds of languages:

## Kind of languages

### Machine Language

This is what the computer actually sees and deals with. Every command the computer sees is given as a number or sequence of numbers.  It is in binary, normally presented in hex to simplify and be more readable.

```
83 ec 08 -> sub $0x8,%esp
83 e4 f0 -> and $0xfffffff0,%esp
b8 00 00 00 00 -> mov $0x0,%eax
83 c0 0f -> add $0xf,%eax
```

### Assembly Language

This is the same as machine language, except the command numbers have been replaced by letter sequences which are more readable and easier to memorize.

AT&T:

```asm
push   %ebp
sub    $0x8,%esp
movb   $0x41,0xffffffff(%ebp)
```

Intel:

```asm
push ebp
mov  ebp, esp
sub  esp, 0C0h
```

HLA (High Level Assembly):

```asm
program HelloWorld;
#include( "stdlib.hhf" )
begin HelloWorld;
stdout.put( "Hello, World of Assembly Language", nl );
end HelloWorld;
```

### High-Level Language

High-level languages are there to make programming easier. Assembly language requires you to work with the machine itself. High-level languages allow you to describe the program in a more natural language. A single command in a high-level language usually is equivalent to several commands in an assembly language.  Readability is the best.

C/C++:

```c
#include <stdio.h>

int main()
{
  char name[20];
  …
  return 0;
}
```
