# System calls

### Definition:
   A system call is a programmatic way for an application to request a service from the kernel of the operating system.

### Purpose:
   - Provide an interface between user-space programs and the kernel
   - Allow controlled access to hardware and other protected system resources
   - Ensure security and stability of the system

### Common Types:
   - Process Control (e.g., fork, exec, exit)
   - File Management (e.g., open, read, write, close)
   - Device Management (e.g., ioctl)
   - Information Maintenance (e.g., getpid, alarm, sleep)
   - Communication (e.g., pipe, sockets)
   - Memory Management (e.g., mmap, brk)

### Mechanism:
   - Programs typically use library functions that wrap system calls
   - System calls involve a context switch from user mode to kernel mode

### Examples:
   - Unix/Linux: open(), read(), write(), fork(), exec()
   - Windows: CreateProcess(), ReadFile(), WriteFile()

### Importance:
   - Essential for OS abstraction and portability
   - Critical for system security and resource management

### Performance:
   - System calls are relatively expensive operations due to mode switching
   - Efficient use of system calls is important for performance optimization

### Tracing:
   - Tools like strace (Linux) or dtrace (Solaris) can monitor system calls for debugging and performance analysis

When `ls` command is run with strace,

[Execution Flow Diagram for 'ls' command]
```

+-------------------+
|    Start 'ls'     |
+-------------------+
          |
          v
+-------------------+
|  execve() - Load  |
|   'ls' program    |
+-------------------+
          |
          v
+-------------------+
| Memory Setup      |
| - brk()           |
| - mmap() calls    |
+-------------------+
          |
          v
+-------------------+
| Load Libraries    |
| - openat()        |
| - read()          |
| - mmap()          |
+-------------------+
          |
          v
+-------------------+
| Security & Random |
| - getrandom()     |
+-------------------+
          |
          v
+-------------------+
| Locale Setup      |
| - openat() locale |
+-------------------+
          |
          v
+-------------------+
| Terminal Check    |
| - ioctl() calls   |
+-------------------+
          |
          v
+-------------------+
| Open Directory    |
| - openat(".")     |
+-------------------+
          |
          v
+-------------------+
| Read Directory    |
| - getdents64()    |
+-------------------+
          |
          v
+-------------------+
| Write Output      |
| - write()         |
+-------------------+
          |
          v
+-------------------+
| Cleanup & Exit    |
| - close() calls   |
| - exit_group()    |
+-------------------+
```