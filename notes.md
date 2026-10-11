# Lab 2 Meet the OS You Will Build

Student: Ciocanu Ilinca  
Group: FAF-241

## 1 Objective

The objective of this laboratory work is to understand the basic structure of xv6 and implement a user program that communicates with the kernel through a system call. My program, `sleep`, pauses for a given number of clock ticks.

## 2 The xv6 operating system

xv6 is a small Unix-like operating system written mainly in C. The laboratory environment uses Ubuntu on UTM, with QEMU to emulate the RISC-V hardware for xv6. The command `make qemu` compiles the source and starts the emulator.

The startup sequence ends by launching the shell:

```text
xv6 kernel is booting
init: starting sh
$
```

The shell provides an interface for entering commands and starting user programs.

## 3 Basic commands and source structure

Three programs included in xv6 are `ls`, `cat`, and `echo`. The `ls` command lists files, `cat README` displays the README file, and `echo` prints its arguments. For example:

```text
$ echo hello xv6
hello xv6
```

The command `wc README` counts the file's lines, words, and bytes. The command `ls | grep c` passes the directory listing to `grep`, which selects lines containing the letter c. For this pipe to work, the OS must support processes and communication between them. The xv6 shell follows the same basic ideas as a Linux shell, but has fewer commands and features.

The source is divided into `kernel/` and `user/`. The kernel manages protected resources such as memory, processes, and files. User programs run with restricted privileges and request kernel services through system calls.

The `cat` program uses five system calls. `open` opens a file, `read` obtains its bytes, `write` sends them to the output, `close` releases the file descriptor, and `exit` ends the program. The function `fprintf` formats messages in userspace and uses `write` to print them.

The implementation of `sys_read` is in `kernel/sysfile.c`. In the MIT source listed in the references, the function name appears on line 66:

```text
66:sys_read(void)
```

The handler checks the request and passes it to the file-reading code. This shows how a user program's request reaches the kernel implementation.

<!-- PAGE -->

## 4 My implementation

I implemented `sleep` as a small C program. It accepts one argument, converts it to an integer with `atoi`, and passes that value to the delay system call. The argument represents clock ticks. It does not represent seconds, as it does in the Linux `sleep` command.

The source file is `user/sleep.c`:

```c
#include "kernel/types.h"
#include "user/user.h"

int
main(int argc, char *argv[])
{
  if(argc != 2){
    fprintf(2, "usage: sleep <ticks>\n");
    exit(1);
  }
  sleep(atoi(argv[1]));
  exit(0);
}
```

The condition `argc != 2` checks for an incorrect number of arguments. The program name counts as the first argument, so one tick value gives a total of two. If the check fails, the program prints a usage message to standard error and exits with status 1. Otherwise, it requests the delay and exits with status 0.

The build entry for this program belongs in the Makefile's `UPROGS` list:

```makefile
	$U/_sleep\
```

This entry makes the executable part of the xv6 filesystem image. The listing follows the handout's `sleep` interface; newer MIT source uses `pause` for the same user-facing delay operation.

## 5 Program behavior

The command `sleep 10` requests a wait of ten clock ticks. It prints no message and returns control to the shell after the wait. Calling `sleep` without an argument reaches the error branch and prints:

```text
usage: sleep <ticks>
```

My implementation checks the argument count. It does not include additional checks for letters or negative values in the argument. During a timed wait, the kernel can schedule other processes instead of making this program continuously use the CPU.

## 6 Conclusion

My program uses an existing kernel service, so it does not require a new system call. The laboratory connects the user-side function call with the kernel's responsibility for managing processes. The separation between `user/` and `kernel/` makes this relationship easy to follow in the xv6 source.

## References

MIT xv6 source: https://github.com/mit-pdos/xv6-riscv
