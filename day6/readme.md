# Day 6: System Calls and Kernel Modules

## 1. System Calls: Implementation and Handling

### Introduction
System calls are the interface between user space applications and the kernel. They provide a way for programs to request services from the operating system kernel. Understanding how system calls are implemented and handled is crucial for both application and kernel developers.

### Key Concepts

1. **System Call Interface**
   - User space to kernel space transition
   - Architecture-specific implementation

2. **System Call Table**
   - Array of function pointers to system call handlers
   - Each system call has a unique number

3. **System Call Wrapper Functions**
   - Provided by C library (e.g., glibc)
   - Handle the details of making the actual system call

4. **Context Switch**
   - Switching from user mode to kernel mode
   - Saving and restoring process state

5. **System Call Arguments**
   - Passing arguments from user space to kernel space
   - Validation and copying of data

### System Call Flow

1. Application calls a system call wrapper function
2. Wrapper function sets up registers with system call number and arguments
3. Trap instruction is executed (e.g., `syscall` on x86-64)
4. CPU switches to kernel mode
5. Kernel saves user space context
6. Kernel executes the system call handler
7. Results are passed back to user space
8. CPU switches back to user mode
9. Execution returns to the application

### Practical Example: Simple System Call

Here's a simple example of how to use the `write` system call in C:

```c
#include <unistd.h>
#include <string.h>

int main() {
    const char* message = "Hello, System Call!\n";
    ssize_t bytes_written = write(STDOUT_FILENO, message, strlen(message));
    return 0;
}
```

### Gotchas and Best Practices

1. **Error Handling**: Always check the return value of system calls and handle errors appropriately.
2. **Performance**: System calls have overhead; use them judiciously in performance-critical code.
3. **Security**: Validate all user input before passing it to system calls to prevent security vulnerabilities.
4. **Blocking vs Non-blocking**: Be aware of which system calls may block and how this affects your application.
5. **Portability**: Some system calls are not available on all platforms; consider using library functions for better portability.

### Interview Questions

1. Q: What is the purpose of the system call table?
   A: The system call table is an array of function pointers that maps system call numbers to their corresponding handler functions in the kernel. It allows the kernel to dispatch system calls to the appropriate handler based on the system call number provided by the user space application.

2. Q: Explain the difference between a system call and a library function.
   A: A system call is a request for service from the kernel and involves a transition from user mode to kernel mode. A library function is a user space function that may or may not use system calls. Library functions often provide a higher-level abstraction and may combine multiple system calls for convenience.

3. Q: What happens during a context switch from user mode to kernel mode?
   A: During a context switch from user mode to kernel mode, the CPU saves the current user space context (including register values and the instruction pointer), switches to the kernel's address space, and begins executing kernel code. The kernel then handles the system call and eventually switches back to user mode, restoring the saved context.

4. Q: How does the kernel protect itself from malicious or erroneous system call arguments?
   A: The kernel validates all arguments passed from user space before using them. This includes checking for valid memory addresses, verifying permissions, and copying data from user space to kernel space using safe functions like `copy_from_user()`. The kernel never directly dereferences user space pointers.

5. Q: What is the role of the C library in system calls?
   A: The C library provides wrapper functions for system calls, which handle the details of setting up registers and executing the appropriate trap instruction. These wrappers make it easier for programmers to use system calls and can provide additional functionality, such as setting `errno` on errors.

## 2. Kernel Modules: Writing and Loading

### Introduction
Kernel modules are pieces of code that can be loaded and unloaded into the kernel upon demand. They extend the functionality of the kernel without the need to reboot the system. Understanding how to write and load kernel modules is essential for kernel developers and system administrators.

### Key Concepts

1. **Module Structure**
   - init and exit functions
   - Module license and author information

2. **Module Compilation**
   - Using `Makefile` with kernel build system
   - Compiling against the running kernel

3. **Module Loading/Unloading**
   - `insmod`, `rmmod`, and `modprobe` commands
   - Automatic dependency resolution with `modprobe`

4. **Module Parameters**
   - Passing parameters to modules at load time
   - Using `module_param` macro

5. **Kernel Symbol Table**
   - Exporting symbols for use by other modules
   - Using `EXPORT_SYMBOL` macro

### Practical Example: Simple Kernel Module

Here's a basic "Hello World" kernel module:

```c
#include <linux/init.h>
#include <linux/module.h>
#include <linux/kernel.h>

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
MODULE_DESCRIPTION("A simple Hello World module");
MODULE_VERSION("0.1");

static int __init hello_init(void) {
    printk(KERN_INFO "Hello, World!\n");
    return 0;
}

static void __exit hello_exit(void) {
    printk(KERN_INFO "Goodbye, World!\n");
}

module_init(hello_init);
module_exit(hello_exit);
```

Makefile for the module:

```makefile
obj-m += hello.o

all:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) modules

clean:
    make -C /lib/modules/$(shell uname -r)/build M=$(PWD) clean
```

### Loading and Unloading the Module

To compile and load the module:

```bash
make
sudo insmod hello.ko
```

To unload the module:

```bash
sudo rmmod hello
```

### Gotchas and Best Practices

1. **Kernel Version Compatibility**: Ensure your module is compatible with the target kernel version.
2. **Error Handling**: Properly handle errors and clean up resources in the init function.
3. **Concurrency**: Be aware of concurrency issues; use appropriate locking mechanisms.
4. **Memory Management**: Use kernel memory allocation functions (e.g., `kmalloc`) instead of user space functions.
5. **Debugging**: Use `printk` for debugging, but be cautious about performance impact in production.

### Interview Questions

1. Q: What is the difference between a loadable kernel module and built-in kernel code?
   A: A loadable kernel module can be dynamically loaded and unloaded without rebooting the system, while built-in kernel code is compiled directly into the kernel image and is always present. Modules offer flexibility and ease of maintenance, but may have a slight performance overhead compared to built-in code.

2. Q: Explain the purpose of the `module_init` and `module_exit` macros.
   A: `module_init` specifies the function to be called when the module is loaded, typically used for initialization. `module_exit` specifies the function to be called when the module is unloaded, used for cleanup and resource freeing.

3. Q: How can you pass parameters to a kernel module at load time?
   A: Parameters can be passed to a kernel module using the `module_param` macro to declare them, and then providing values when loading the module with `insmod` or `modprobe`. For example: `insmod mymodule.ko myparam=42`.

4. Q: What is the purpose of the `MODULE_LICENSE` macro?
   A: The `MODULE_LICENSE` macro declares the license under which the module is distributed. This is important for determining which kernel symbols the module can access, as some symbols are only available to modules with GPL-compatible licenses.

5. Q: How does `modprobe` differ from `insmod` when loading kernel modules?
   A: `modprobe` is more intelligent than `insmod`. It can handle module dependencies automatically, loading any required modules before loading the requested module. It also searches for modules in standard locations, while `insmod` requires the full path to the module file.

This content covers the basics of system calls and kernel modules, providing both theoretical knowledge and practical examples. It should give a solid foundation for understanding these important aspects of Linux kernel development.
```
