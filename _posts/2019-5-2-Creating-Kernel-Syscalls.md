In this post I will briefly describe how to create a Linux syscall, if you never compiled and installed a Linux Kernel before you
can red my post https://brunohadlich.github.io/Build-Linux-Kernel/.

First thing is editing file arch/x86/entry/syscalls/syscall_64.tbl, you will see two comments, one explaining that you should not
create syscalls with numbers between 387 and 423 and another comment saying number 512 ahead is used for x32-specific system
calls so we can start using number 428 to create our syscall, in case the Linux kernel version you are editing has this number in
use you can choose a greater number, in this post I will be using 428.

After syscall 427 you can add the following line.

```text
428     common  teste                   __x64_sys_teste
```

This will create a x64 syscall called teste with entry point __x64_sys_teste. Now its time to define the function that will
handle such syscall, for simplicity sake we will edit file kernel/exit.c that already has a bunch of syscall definitions, inside
this file add:

```C
SYSCALL_DEFINE0(teste)
{
       uint64_t rcs = 0;
       asm ("mov %%cs, %0" : "=r" (rcs));
       printk("Ring Kernel: %d\n", (int) (rcs & 3));
}
```

This is all you have to do to create your own syscall, the one I wrote above will read the ring in which CPU is currently running
and print it to dmesg, now all you have to do is compiling your kernel and reinstall it, after that reboot your machine and write
the following C code to test your syscall:

```C
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <stdint.h>

int main(int argc, char *argv[]) {
        uint64_t rcs = 0;
        asm ("mov %%cs, %0" : "=r" (rcs));
        printf("Ring user space: %d\n", (int) (rcs & 3));
        syscall(428);
        exit(0);
}
```

Compile it with "gcc teste.c -o teste" and run with ./teste. The output should be equals to "Ring user space: 3", now run dmesg
command and you should see the message "Ring Kernel: 0".
