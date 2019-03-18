---
layout: post
title: How to disassemble glibc functions
---

First write a simple C program that will link to glibc, main.c:

```c
int main() { 
    return 0;
}
```
After that compile it with "gcc main.c -o main".

Now if you run "ldd main" it is possible to see that main program requires libc, in your system it will probably be at /lib/x86_64-linux-gnu/.

My output:

```console
bruno@bruno-laptop:~/git/disassemble_glibc$ ldd main
        linux-vdso.so.1 (0x00007ffd2fbe3000)
        libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007ff00ad26000)
        /lib64/ld-linux-x86-64.so.2 (0x00007ff00b319000)
bruno@bruno-laptop:~/git/disassemble_glibc$
```

Now to finally see the disassembled functions you will need to run gdb debugger passing your program as argument, "gdb main".

After gdb is loaded you have to put a breakpoint at main function of your program with "break main".

Execute "run" and something like "Breakpoint 1, 0x00005555555545fe in main ()" will be displayed.

Now all you need to do is execute "disassemble <function_you_look_for>", examples:

disassemble printf\
disassemble open\
disassemble close\
disassemble free

Here is a step by step:

```console
bruno@bruno-laptop:~/git/disassemble_glibc$ gdb main
GNU gdb (Ubuntu 8.2-0ubuntu1~18.04) 8.2
Copyright (C) 2018 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.
Type "show copying" and "show warranty" for details.
This GDB was configured as "x86_64-linux-gnu".
Type "show configuration" for configuration details.
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>.
Find the GDB manual and other documentation resources online at:
    <http://www.gnu.org/software/gdb/documentation/>.

For help, type "help".
Type "apropos word" to search for commands related to "word"...
Voltron loaded.
Reading symbols from main...(no debugging symbols found)...done.
(gdb) break main
Breakpoint 1 at 0x5fe
(gdb) run
Starting program: /home/bruno/git/disassemble_glibc/main

Breakpoint 1, 0x00005555555545fe in main ()
(gdb) disassemble free
Dump of assembler code for function free:
   0x00007ffff7df0620 <+0>:     mov    0x20daa1(%rip),%rcx        # 0x7ffff7ffe0c8 <alloc_last_block>
   0x00007ffff7df0627 <+7>:     cmp    %rdi,%rcx
   0x00007ffff7df062a <+10>:    je     0x7ffff7df0630 <free+16>
   0x00007ffff7df062c <+12>:    repz retq
   0x00007ffff7df062e <+14>:    xchg   %ax,%ax
   0x00007ffff7df0630 <+16>:    sub    $0x8,%rsp
   0x00007ffff7df0634 <+20>:    mov    0x20da9d(%rip),%rdx        # 0x7ffff7ffe0d8 <alloc_ptr>
   0x00007ffff7df063b <+27>:    xor    %esi,%esi
   0x00007ffff7df063d <+29>:    mov    %rcx,%rdi
   0x00007ffff7df0640 <+32>:    sub    %rcx,%rdx
   0x00007ffff7df0643 <+35>:    callq  0x7ffff7df4a20 <memset>
   0x00007ffff7df0648 <+40>:    mov    %rax,0x20da89(%rip)        # 0x7ffff7ffe0d8 <alloc_ptr>
   0x00007ffff7df064f <+47>:    add    $0x8,%rsp
   0x00007ffff7df0653 <+51>:    retq
End of assembler dump.
(gdb)
```
