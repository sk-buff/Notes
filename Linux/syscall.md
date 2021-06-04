# Introduction

This document is about system calls in Linux.

### How to find the implementation in Linux

We use the  socket ``connect`` system call as an example to show how to find it's implementation. It's easy, just search the function name and find the reference of the name warped by **SYSCALL_DEFINE*** macro, where ***** is a number which indicates the number of parameters of the system call. **SYSCALL_DEFINE*** macro will explicitly define system calls. The definition of the `connect` system call in Linux 5.11 locates at line 1859 in `net/socket.c`, which is:

```
SYSCALL_DEFINE3(connect, int, fd, struct sockaddr __user *, uservaddr,
				int, addrlen)
```

Here we don't discuss details about these system call definition macros, but except for the numbers in these macros will indicate the number of parameters of the corresponding system call defined, we do have to now it will add a `sys_` prefix to the system call name. For example, `SYSCALL_DEFINE3(connect, ...)` mentioned above actually defines a function `sys_connect`. This is consistent with the system call table `arch/x86/entry/syscalls/syscall_32.tbl` or `arch/x86/entry/syscalls/syscall_64.tbl` where the actual kernel entry point is sys_connect.

