# Day 10: Advanced Memory Management and Debugging Techniques

## 1. Advanced Memory Management

### Memory Allocation Strategies

Key Concepts:
- Understanding the heap and memory allocation
- Strategies for efficient memory use
- Custom memory allocators

Example: Simple memory pool allocator

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define POOL_SIZE 1024
#define BLOCK_SIZE 32

typedef struct {
    char memory[POOL_SIZE];
    int used[POOL_SIZE / BLOCK_SIZE];
} MemoryPool;

MemoryPool* create_pool() {
    MemoryPool* pool = malloc(sizeof(MemoryPool));
    memset(pool->used, 0, sizeof(pool->used));
    return pool;
}

void* pool_alloc(MemoryPool* pool) {
    for (int i = 0; i < POOL_SIZE / BLOCK_SIZE; i++) {
        if (!pool->used[i]) {
            pool->used[i] = 1;
            return &pool->memory[i * BLOCK_SIZE];
        }
    }
    return NULL;  // Out of memory
}

void pool_free(MemoryPool* pool, void* ptr) {
    long offset = (char*)ptr - pool->memory;
    if (offset >= 0 && offset < POOL_SIZE) {
        pool->used[offset / BLOCK_SIZE] = 0;
    }
}

void destroy_pool(MemoryPool* pool) {
    free(pool);
}

int main() {
    MemoryPool* pool = create_pool();

    char* str1 = pool_alloc(pool);
    char* str2 = pool_alloc(pool);

    strcpy(str1, "Hello");
    strcpy(str2, "World");

    printf("%s %s\n", str1, str2);

    pool_free(pool, str1);
    pool_free(pool, str2);

    destroy_pool(pool);
    return 0;
}
```

### Memory Mapping

Key Concepts:
- Using mmap() for file and anonymous mappings
- Shared memory between processes

Example: Using mmap() for file I/O

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/mman.h>
#include <sys/stat.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <file>\n", argv[0]);
        exit(1);
    }

    int fd = open(argv[1], O_RDONLY);
    if (fd == -1) {
        perror("open");
        exit(1);
    }

    struct stat sb;
    if (fstat(fd, &sb) == -1) {
        perror("fstat");
        exit(1);
    }

    char *addr = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
    if (addr == MAP_FAILED) {
        perror("mmap");
        exit(1);
    }

    // Print file contents
    write(STDOUT_FILENO, addr, sb.st_size);

    if (munmap(addr, sb.st_size) == -1) {
        perror("munmap");
        exit(1);
    }

    close(fd);
    return 0;
}
```

## 2. Memory Debugging Techniques

### Using Valgrind

Key Concepts:
- Detecting memory leaks
- Finding use of uninitialized memory
- Identifying invalid memory accesses

Example: Compiling and running a program with Valgrind

```bash
gcc -g program.c -o program
valgrind --leak-check=full ./program
```

### Static Analysis Tools

Key Concepts:
- Introduction to static analysis
- Using tools like cppcheck and Clang Static Analyzer

Example: Running cppcheck

```bash
cppcheck --enable=all program.c
```

### Address Sanitizer (ASan)

Key Concepts:
- Detecting memory errors at runtime
- Compiling with ASan support

Example: Compiling and running a program with ASan

```bash
gcc -fsanitize=address -g program.c -o program
./program
```

## 3. Advanced Debugging Techniques

### GDB Advanced Features

Key Concepts:
- Setting conditional breakpoints
- Using watchpoints
- Examining core dumps

Example: GDB commands for advanced debugging

```gdb
# Set a conditional breakpoint
break function_name if condition

# Set a watchpoint
watch variable_name

# Examine a core dump
gdb program core

# Print backtrace with local variables
bt full
```

### Debugging Multithreaded Programs

Key Concepts:
- Thread-specific breakpoints
- Examining thread states
- Detecting deadlocks

Example: GDB commands for debugging threads

```gdb
# List all threads
info threads

# Switch to a specific thread
thread thread_number

# Set a breakpoint for a specific thread
break file.c:line_number thread thread_number

# Continue execution of all threads
set scheduler-locking off
```

### Remote Debugging

Key Concepts:
- Setting up a GDB server
- Connecting to a remote debugging session

Example: Setting up remote debugging

```bash
# On the target machine
gdbserver :1234 ./program

# On the development machine
gdb
(gdb) target remote targethost:1234
```

## 4. Performance Profiling

### Using gprof

Key Concepts:
- Compiling with profiling support
- Generating and interpreting profiling data

Example: Using gprof

```bash
gcc -pg program.c -o program
./program
gprof program gmon.out > analysis.txt
```

### Perf Tool

Key Concepts:
- Sampling-based profiling
- Analyzing system performance

Example: Using perf

```bash
perf record ./program
perf report
```

## 5. Best Practices and Considerations

- Writing testable code
- Implementing proper error handling and logging
- Continuous integration and automated testing

## Practice Exercises

1. Implement a custom memory allocator with different allocation strategies (e.g., first-fit, best-fit).

2. Create a program with deliberate memory errors and use Valgrind to detect and fix them.

3. Write a multithreaded program and use GDB to debug race conditions.

4. Profile a computationally intensive program using gprof and optimize it based on the results.

## Important Tools and Utilities

- strace: Trace system calls and signals
- ltrace: Library call tracer
- objdump: Display information from object files

## Additional Resources

1. "Debugging with GDB" (Free Software Foundation)
2. "Valgrind User Manual"
3. "Linux Debugging and Performance Tuning" by Steve Best
4. "Advanced Linux Programming" by Mark Mitchell, Jeffrey Oldham, and Alex Samuel
