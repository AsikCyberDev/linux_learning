# System calls

When `ls` command is run,

[Execution Flow Diagram for 'ls' command]

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
