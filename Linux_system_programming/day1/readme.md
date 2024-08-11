
# Day 1: Fundamentals

## 1. System Calls: The Kernel's API

### Introduction
System calls are the fundamental interface between an application and the Linux kernel. They provide a way for programs to request services from the kernel, such as file operations, process management, and network communication.

### Key Concepts

1. **User Space vs. Kernel Space**
   - User space: Where normal programs run
   - Kernel space: Where the kernel operates
   - System calls bridge these two spaces

2. **System Call Mechanism**
   - User program invokes a system call
   - CPU switches from user mode to kernel mode
   - Kernel performs the requested operation
   - CPU switches back to user mode
   - Control returns to the user program

3. **Common System Calls**
   - Process Control: `fork()`, `exec()`, `exit()`
   - File Management: `open()`, `read()`, `write()`, `close()`
   - Device Management: `ioctl()`
   - Information Maintenance: `getpid()`, `alarm()`, `sleep()`
   - Communication: `pipe()`, `shmget()`, `mmap()`

### Practical Example: Using `strace`

```bash
# Trace system calls made by the 'ls' command
strace ls

# Trace only file-related system calls
strace -e trace=file ls

# Count system calls
strace -c ls
```

### System Call Implementation

1. **User Space**
   ```c
   #include <unistd.h>

   int main() {
       write(1, "Hello, World!\n", 14);
       return 0;
   }
   ```

2. **Kernel Space (simplified)**
   ```c
   asmlinkage long sys_write(unsigned int fd, const char __user *buf,
                             size_t count)
   {
       struct file *file;
       ssize_t ret = -EBADF;
       file = fget_light(fd, &fput_needed);
       if (file) {
           ret = vfs_write(file, buf, count, &pos);
           fput_light(file, fput_needed);
       }
       return ret;
   }
   ```

### Gotchas and Best Practices

1. **Error Handling**: Always check the return value of system calls.
2. **Blocking vs. Non-blocking**: Understand when a system call might block.
3. **Security**: Be aware of potential security implications (e.g., buffer overflows).
4. **Performance**: System calls have overhead; use them judiciously.

### Interview Questions

1. Q: What's the difference between a system call and a library function?
   A: A system call directly requests a service from the kernel, requiring a mode switch. A library function runs entirely in user space, though it may ultimately use system calls.

2. Q: How does a program invoke a system call?
   A: Typically through wrapper functions provided by the C library. At a lower level, it involves loading registers with system call number and arguments, then executing a special instruction (e.g., `syscall` on x86-64).

3. Q: What happens if a system call fails?
   A: It typically returns -1 and sets the global variable `errno` to indicate the specific error.

4. Q: Can you name five common system calls and their purposes?
   A:
   - `fork()`: Create a new process
   - `exec()`: Execute a new program
   - `open()`: Open a file or create it
   - `read()`: Read from a file descriptor
   - `write()`: Write to a file descriptor

5. Q: What's the purpose of the `mmap()` system call?
   A: It maps files or devices into memory, allowing direct access to them as if they were arrays in memory.

## 2. Process Management: Creation, Scheduling, and Termination

### Introduction
Process management is a core function of any operating system. In Linux, processes are the basic unit of execution, and the kernel provides mechanisms for creating, scheduling, and terminating them.

### Key Concepts

1. **Process States**
   - Running
   - Waiting (ready to run)
   - Blocked (waiting for I/O or an event)
   - Stopped
   - Zombie

2. **Process Control Block (PCB)**
   - Contains all information about a process
   - Represented by `task_struct` in Linux kernel

3. **Process Creation**
   - `fork()` system call
   - Copy-on-Write (CoW) optimization

4. **Process Scheduling**
   - Completely Fair Scheduler (CFS)
   - Real-time schedulers (FIFO, Round Robin)

5. **Process Termination**
   - `exit()` system call
   - Cleanup and resource deallocation

### Practical Example: Creating a Process

```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();

    if (pid == -1) {
        perror("fork failed");
        return 1;
    } else if (pid == 0) {
        // Child process
        printf("Hello from child (PID: %d)\n", getpid());
        return 0;
    } else {
        // Parent process
        printf("Hello from parent (PID: %d)\n", getpid());
        wait(NULL);  // Wait for child to finish
        return 0;
    }
}
```

### Process Scheduling Internals

1. **Time Slices**: CFS doesn't use fixed time slices but calculates dynamic time slices based on the number of runnable processes.

2. **Virtual Runtime**: CFS uses a virtual runtime to determine which process to run next.

3. **Red-Black Tree**: Runnable processes are stored in a red-black tree, sorted by their virtual runtime.

### Gotchas and Best Practices

1. **Zombie Processes**: Always wait for child processes to prevent zombies.
2. **Resource Leaks**: Ensure proper cleanup when a process exits.
3. **Signal Handling**: Implement proper signal handlers for robust process management.
4. **Priority Inversion**: Be aware of potential priority inversion issues in real-time systems.

### Interview Questions

1. Q: What's the difference between a process and a thread?
   A: A process is an independent program with its own memory space, while threads are lighter-weight units of execution that share the same memory space within a process.

2. Q: Explain the concept of Copy-on-Write (CoW).
   A: CoW is an optimization technique used during `fork()`. Instead of immediately copying all memory, pages are shared between parent and child until one of them modifies a page, at which point a copy is made.

3. Q: What is a zombie process and how can it be prevented?
   A: A zombie process is a process that has finished execution but still has an entry in the process table. It can be prevented by having the parent process wait for the child's termination using `wait()` or `waitpid()`.

4. Q: How does the Completely Fair Scheduler work?
   A: CFS tries to give each process a fair share of CPU time. It uses a red-black tree to organize runnable processes based on their virtual runtime, always selecting the process with the lowest virtual runtime to run next.

5. Q: What happens when you call `fork()` in a multithreaded program?
   A: Only the thread that called `fork()` is replicated in the child process. This can lead to issues if not handled carefully, especially with regards to locked mutexes.

## 3. Memory Management: Virtual Memory and Paging

### Introduction
Memory management is a critical function of the Linux kernel, involving the allocation and deallocation of memory resources to processes and the kernel itself. Virtual memory and paging are key concepts in this domain.

### Key Concepts

1. **Virtual Memory**
   - Abstraction of physical memory
   - Provides each process with its own address space
   - Allows for memory protection and overcommitment

2. **Page Tables**
   - Data structures that map virtual addresses to physical addresses
   - Multi-level page tables in modern systems

3. **Translation Lookaside Buffer (TLB)**
   - Cache for page table entries
   - Speeds up virtual-to-physical address translation

4. **Page Faults**
   - Occurs when a memory access can't be satisfied by current page tables
   - Types: Minor (page is in memory but not mapped) and Major (page needs to be loaded from disk)

5. **Swapping**
   - Moving pages between RAM and disk to free up physical memory

### Virtual Memory Layout

```
High Address
+------------------+
|    Stack         |
|        |         |
|        v         |
+------------------+
|        ^         |
|        |         |
|       Heap       |
+------------------+
|  Uninitialized   |
|      Data        |
+------------------+
| Initialized Data |
+------------------+
|      Text        |
Low Address
```

### Practical Example: Observing Memory Layout

```c
#include <stdio.h>
#include <stdlib.h>

int global_var;  // Uninitialized data
int global_initialized_var = 42;  // Initialized data

int main() {
    int stack_var;  // Stack
    int *heap_var = malloc(sizeof(int));  // Heap

    printf("Addresses:\n");
    printf("global_var: %p\n", (void*)&global_var);
    printf("global_initialized_var: %p\n", (void*)&global_initialized_var);
    printf("main function: %p\n", (void*)main);
    printf("stack_var: %p\n", (void*)&stack_var);
    printf("heap_var: %p\n", (void*)heap_var);

    free(heap_var);
    return 0;
}
```

### Page Table Walk

1. CPU generates virtual address
2. MMU looks up in TLB
3. If TLB miss, walk the page table:
   - Check Page Global Directory (PGD)
   - Check Page Upper Directory (PUD)
   - Check Page Middle Directory (PMD)
   - Check Page Table Entry (PTE)
4. If page not present, generate page fault
5. Otherwise, access physical memory

### Gotchas and Best Practices

1. **Memory Leaks**: Always free dynamically allocated memory.
2. **Page Alignment**: Some operations require page-aligned memory.
3. **Huge Pages**: Use huge pages for large memory allocations to reduce TLB misses.
4. **Copy-on-Write**: Be aware of CoW behavior when forking processes.

### Interview Questions

1. Q: What is the purpose of virtual memory?
   A: Virtual memory provides process isolation, allows for memory overcommitment, simplifies memory allocation, and enables features like demand paging and swapping.

2. Q: Explain the difference between a page fault and a segmentation fault.
   A: A page fault occurs when a valid memory access can't be immediately satisfied by the current page tables. A segmentation fault occurs when a program tries to access memory it's not allowed to access.

3. Q: How does Linux handle memory overcommitment?
   A: Linux allows memory overcommitment by default. It allocates virtual memory liberally but only assigns physical pages when the memory is actually used. If physical memory runs low, the OOM (Out of Memory) killer may terminate processes to free up memory.

4. Q: What is the Translation Lookaside Buffer (TLB) and why is it important?
   A: The TLB is a cache that stores recent virtual-to-physical address translations. It's crucial for performance as it reduces the need for expensive page table walks.

5. Q: How does demand paging work?
   A: Demand paging loads pages into memory only when they are accessed. When a process tries to access a page that's not in memory, a page fault occurs, and the OS loads the required page from disk into memory.
```
