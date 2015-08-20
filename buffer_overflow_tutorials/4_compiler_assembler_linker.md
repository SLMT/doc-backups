# A Compiler, Assembler, Linker & Loader

### The Relocation Records

Because the various object files will include references to each others code and/or data, these will need to be combined during the link time. For example in Figure 1, the object file that has main() includes calls to funct() and printf() functions. After linking all of the object files together, the linker uses the **relocation records** to find all of the addresses that need to be filled in.

<img src="images/image018.png" /><br />
Figure 1: The relocation record.

### The Symbol Table

Since assembling to machine code removes all traces of labels from the code, the object file format has to keep these around in a different place. It is accomplished by the **symbol table**, a list of names and their corresponding offsets in the text and data segments. A **disassembler** provides support for translating back from an object file or executable.

### Linking (Example in listing – linking 2 object files)

The linker actually enables separate compilation. As shown in Figure 2, an executable can be made up of a number of source files which can be compiled and assembled into their object files respectively, independently.

<img src="images/image019.png" /><br />
Figure 2:  Linking process of object files

### Shared Objects

In a typical system, a number of programs will be running. Each program relies on a number of functions, some of which will be standard C library functions, like printf(), malloc(), strcpy(), etc. If every program uses the standard C library, it means that each program would normally have a unique copy of this particular library present within it. Unfortunately, this result in wasted resources, degrade the efficiency and performance. Since the C library is common, it is better to have each program reference the common, one instance of that library, instead of having each program contain a copy of the library. This is implemented during the linking process where some of the objects are linked during the link time whereas some done during the run time (deferred/dynamic linking).

## Statically Linked

The term ‘statically linked’ means that the program and the particular library that it’s linked against are combined together by the linker at link time. This means that the binding between the program and the particular library is fixed and known at link time before the program run. It also means that we can't change this binding, unless we re-link the program with a new version of the library.

Programs that are linked statically are linked against archives of objects (libraries) that typically have the extension of .a. An example of such a collection of objects is the standard C library, **libc.a**. You might consider linking a program statically for example, in cases where you weren't sure whether the correct version of a library will be available at runtime, or if you were testing a new version of a library that you don't yet want to install as shared. For gcc, the –static option is used during the compilation/linking of the program.

```
gcc –static filename.c –o filename
```

The drawback of this technique is that the executable is quite big in size.

### Examples

Compile and link (for static linking).

```
[bodo@bakawali testbed5]$ gcc -g -static testbuff.c -o testbuff
/tmp/ccwh4rvU.o(.text+0x1e): In function 'Test':
/home/bodo/testbed5/testbuff.c:8: warning: the 'gets' function is dangerous and should not be used.

[bodo@bakawali testbed5]$ size -d testbuff
  text    data     bss   dec     hex     filename
380157    3368    4476  388001   5eba1   testbuff
```

Compile and link (for dynamic linking).  Note the executable size.

```
[bodo@bakawali testbed5]$ gcc -g testbuff.c -o testbuff
/tmp/ccMunGye.o(.text+0x1e): In function `Test':
/home/bodo/testbed5/testbuff.c:8: warning: the `gets' function is dangerous and should not be used.

[bodo@bakawali testbed5]$ size -d testbuff
text     data      bss  dec      hex filename
1031     264       4    1299     513 testbuff
```

## Dynamically Linked

The term ‘dynamically linked’ means that the program and the particular library it references are not combined together by the linker at link time. Instead, the linker places information into the executable that tells the loader which shared object module the code is in and which runtime linker should be used to find and bind the references. This means that the binding between the program and the shared object is done at runtime that is before the program starts, the appropriate shared objects are found and bound.

This type of program is called a partially bound executable, because it isn't fully resolved.  The linker, at link time, didn't cause all the referenced symbols in the program to be associated with specific code from the library. Instead, the linker simply said something like: “This program calls some functions within a particular shared object, so I'll just make a note of which shared object these functions are in, and continue on”.  Symbols for the shared objects are only verified for their validity to ensure that they do exist somewhere and are not yet combined into the program. The linker stores in the executable program, the locations of the external libraries where it found the missing symbols. Effectively, this defers the binding until runtime.

Programs that are linked dynamically are linked against shared objects that have the extension .so. An example of such an object is the shared object version of the **standard C library, libc.so**. The advantageous to defer some of the objects/modules during the static linking step until they are finally needed (during the run time) includes:

1. Program files (on disk) become much smaller because they need not hold all necessary text and data segments information.  It is very useful for portability.
2. Standard libraries may be upgraded or patched without every one program need to be re-linked.  This clearly requires some agreed module-naming convention that enables the dynamic linker to find the newest, installed module such as some version specification.  Furthermore the distribution of the libraries is in binary form (no source), including dynamically linked libraries (DLLs) and when you change your program you only have to recompile the file that was changed.
3. Software vendors need only provide the related libraries module required.  Additional runtime linking functions allow such programs to programmatically-link the required modules only.
4. In combination with virtual memory, dynamic linking permits two or more processes to share read-only executable modules such as standard C libraries and kernel.  Using this technique, only one copy of a module needs be resident in memory at any time, and multiple processes, each can executes this shared code (read only).  This results in a considerable memory saving, although demands an efficient swapping policy.

## How Shared Objects Are Used

To understand how a program makes use of shared objects, let's first examine the format of an executable and the steps that occur when the program starts.

### Some Detail Of The ELF Format

Executable and Linking Format is binary format, which is used in SVR4 Unix and Linux systems. It is a format for storing programs or fragments of programs on disk, created as a result of compiling and linking. ELF not only simplifies the task of making shared libraries, but also enhances dynamic loading of modules at runtime.

### ELF Sections

The Executable and Linking Format used by GNU/Linux and other operating systems, defines a number of ‘**sections**’ in an executable program. These are to provide order to the binary file and allow inspection. Important function sections include the **Global Offset Table** (GOT), which stores **addresses of system functions**, the **Procedure Linking Table** (PLT), which stores **indirect links** to the GOT, .init/.fini, for **internal initialization** and **shutdown**, .ctors/.dtors, for **constructors** and **deconstructors**. The data sections are .rodata, for read only data, .data for initialized data, and .bss for uninitialized data. The summary of partial list of the ELF sections are organized as follows (from low to high):

1. .init - Startup
2. .text - String
3. .fini - Shutdown
4. .rodata - Read Only
5. .data - Initialized Data
6. .tdata - Initialized Thread Data
7. .tbss - Uninitialized Thread Data
8. .ctors - Constructors
9. .dtors - Destructors
10. .got - Global Offset Table
11. .bss - Uninitialized Data

You can use the readelf or objdump program against the object or executable files in order to view the sections. In the following Figure, two views of an ELF file are shown: the linking view and the execution view.

<img src="images/image020.png" /><br />
Figure 3: Simplified object file format: linking view and execution view.

Keep in mind that the full format of the ELF contains many more items. As explained previously, the linking view, which is used when the program or library is linked, deals with sections within an object file. Sections contain the bulk of the object file information: data, instructions, relocation information, symbols, debugging information, etc. The execution view, which is used when the program runs, deals with segments. Segments are a way of grouping related sections.

For example, the text segment groups executable code, the data segment groups the program data, and the dynamic segment groups information relevant to dynamic loading. Each segment consists of one or more sections. A process image is created by loading and interpreting segments. The operating system logically copies a file’s segment to a virtual memory segment according to the information provided in the program header table. The OS can also use segments to create a shared memory resource. At link time, the program or library is built by merging together sections with similar attributes into segments. Typically, all the executable and read-only data sections are combined into a single text segment, while the data and BSS are combined into the data segment. These segments are normally called load segments, because they need to be loaded in memory at process creation. Other sections such as symbol information and debugging sections are merged into other, non-load segments.
