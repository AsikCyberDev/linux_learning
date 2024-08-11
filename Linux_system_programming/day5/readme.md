# Day 5: Memory Management and Advanced File I/O

## 1. Advanced Memory Management

### Memory Allocation Techniques

Key Concepts:
- Understanding the process memory layout
- Heap vs. stack allocation
- Custom memory allocators

Example: Implementing a simple memory pool allocator

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define POOL_SIZE 1024
#define BLOCK_SIZE 32

typedef struct {
    char memory[POOL_SIZE];
    char used[POOL_SIZE / BLOCK_SIZE];
} MemoryPool;

MemoryPool* create_pool() {
    MemoryPool* pool = malloc(sizeof(MemoryPool));
    memset(pool->used, 0, sizeof(pool->used));
    return pool;
}

void* pool_alloc(MemoryPool* pool, size_t size) {
    if (size > BLOCK_SIZE) return NULL;

    for (int i = 0; i < POOL_SIZE / BLOCK_SIZE; i++) {
        if (!pool->used[i]) {
            pool->used[i] = 1;
            return &pool->memory[i * BLOCK_SIZE];
        }
    }
    return NULL;
}

void pool_free(MemoryPool* pool, void* ptr) {
    ptrdiff_t offset = (char*)ptr - pool->memory;
    if (offset >= 0 && offset < POOL_SIZE) {
        pool->used[offset / BLOCK_SIZE] = 0;
    }
}

void destroy_pool(MemoryPool* pool) {
    free(pool);
}

int main() {
    MemoryPool* pool = create_pool();

    char* str1 = pool_alloc(pool, 10);
    char* str2 = pool_alloc(pool, 20);

    strcpy(str1, "Hello");
    strcpy(str2, "World");

    printf("%s %s\n", str1, str2);

    pool_free(pool, str1);
    pool_free(pool, str2);

    destroy_pool(pool);
    return 0;
}
```

Explanation:
- The memory pool pre-allocates a large chunk of memory.
- `pool_alloc` manages allocation from this pre-allocated memory.
- `pool_free` marks memory as available for reuse.

### Memory Mapping

Key Concepts:
- Using mmap() for file and anonymous mappings
- Shared vs. private mappings
- Managing large datasets with memory mapping

Example: Using mmap() for efficient file I/O

```c
#include <stdio.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>

int main() {
    const char* filepath = "example.txt";
    int fd = open(filepath, O_RDWR | O_CREAT, S_IRUSR | S_IWUSR);
    if (fd == -1) {
        perror("open");
        return 1;
    }

    // Extend the file to 100 bytes
    if (ftruncate(fd, 100) == -1) {
        perror("ftruncate");
        close(fd);
        return 1;
    }

    char* map = mmap(NULL, 100, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (map == MAP_FAILED) {
        perror("mmap");
        close(fd);
        return 1;
    }

    strcpy(map, "Hello, memory-mapped file!");

    // Changes are automatically synced to the file
    msync(map, 100, MS_SYNC);

    munmap(map, 100);
    close(fd);

    printf("File contents written using mmap.\n");
    return 0;
}
```

Explanation:
- `mmap()` creates a memory mapping of the file.
- Writing to the mapped memory directly modifies the file.
- `msync()` ensures changes are written back to the file.

### Memory Debugging Tools

Key Concepts:
- Using Valgrind for memory leak detection
- Address Sanitizer for detecting memory errors
- Custom memory debugging techniques

Example: Using Valgrind to detect memory leaks

```c
#include <stdlib.h>

void leak_memory() {
    int* ptr = malloc(sizeof(int));
    *ptr = 10;
    // Forgot to free ptr
}

int main() {
    leak_memory();
    return 0;
}
```

To compile and run with Valgrind:
```
gcc -g memory_leak.c -o memory_leak
valgrind --leak-check=full ./memory_leak
```

## 2. Advanced File I/O

### File Locking

Key Concepts:
- Advisory vs. mandatory locking
- Read and write locks
- Using fcntl() for file locking

Example: Implementing advisory file locking

```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>

int lock_file(int fd, int type) {
    struct flock fl;
    fl.l_type = type;
    fl.l_whence = SEEK_SET;
    fl.l_start = 0;
    fl.l_len = 0;
    return fcntl(fd, F_SETLKW, &fl);
}

int main() {
    int fd = open("locked_file.txt", O_RDWR | O_CREAT, 0644);
    if (fd == -1) {
        perror("open");
        return 1;
    }

    if (lock_file(fd, F_WRLCK) == -1) {
        perror("fcntl");
        close(fd);
        return 1;
    }

    printf("File locked. Press Enter to unlock...\n");
    getchar();

    if (lock_file(fd, F_UNLCK) == -1) {
        perror("fcntl");
        close(fd);
        return 1;
    }

    printf("File unlocked.\n");
    close(fd);
    return 0;
}
```

Explanation:
- `fcntl()` is used to apply and remove file locks.
- The program acquires a write lock, waits for user input, then releases the lock.

### Asynchronous I/O

Key Concepts:
- AIO control blocks
- Initiating asynchronous operations
- Handling AIO completions

Example: Asynchronous file read operation

```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <aio.h>
#include <errno.h>
#include <string.h>

#define BUFSIZE 1024

int main() {
    char buffer[BUFSIZE];
    struct aiocb cb;

    int fd = open("test.txt", O_RDONLY);
    if (fd == -1) {
        perror("open");
        return 1;
    }

    memset(&cb, 0, sizeof(struct aiocb));
    cb.aio_fildes = fd;
    cb.aio_buf = buffer;
    cb.aio_nbytes = BUFSIZE - 1;
    cb.aio_offset = 0;

    if (aio_read(&cb) == -1) {
        perror("aio_read");
        close(fd);
        return 1;
    }

    while (aio_error(&cb) == EINPROGRESS) {
        printf("Reading...\n");
        usleep(100000);  // Sleep for 100ms
    }

    int ret = aio_return(&cb);
    if (ret == -1) {
        perror("aio_return");
    } else {
        buffer[ret] = '\0';
        printf("Read %d bytes: %s\n", ret, buffer);
    }

    close(fd);
    return 0;
}
```

Explanation:
- `aio_read()` initiates an asynchronous read operation.
- The program polls the status of the operation using `aio_error()`.
- `aio_return()` retrieves the result of the completed operation.

### Memory-Mapped I/O Revisited

Key Concepts:
- Shared memory between processes using mmap
- Copy-on-write semantics
- Large file support with mmap

Example: Inter-process communication using shared memory

```c
#include <stdio.h>
#include <sys/mman.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>
#include <sys/wait.h>

#define SHM_SIZE 1024

int main() {
    int fd = shm_open("/myshm", O_CREAT | O_RDWR, 0666);
    if (fd == -1) {
        perror("shm_open");
        return 1;
    }

    if (ftruncate(fd, SHM_SIZE) == -1) {
        perror("ftruncate");
        return 1;
    }

    char* ptr = mmap(NULL, SHM_SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (ptr == MAP_FAILED) {
        perror("mmap");
        return 1;
    }

    pid_t pid = fork();
    if (pid == 0) {  // Child process
        strcpy(ptr, "Hello from child!");
        exit(0);
    } else if (pid > 0) {  // Parent process
        wait(NULL);
        printf("Received from child: %s\n", ptr);
    } else {
        perror("fork");
        return 1;
    }

    munmap(ptr, SHM_SIZE);
    close(fd);
    shm_unlink("/myshm");
    return 0;
}
```

Explanation:
- `shm_open()` creates a shared memory object.
- `mmap()` maps the shared memory into the process address space.
- Parent and child processes can communicate through the shared memory.

## Practice Exercises

1. Implement a custom memory allocator that uses a free list to manage memory blocks.

2. Create a program that uses file locking to implement a simple database with concurrent read/write access.

3. Develop a file copy program that uses memory mapping for efficient large file copying.

4. Write a multi-threaded program that performs asynchronous I/O operations on multiple files concurrently.

## Important Tools and Utilities

- strace: Trace system calls related to memory management and file I/O
- ltrace: Trace library calls for memory allocation functions
- vmstat: Report virtual memory statistics
- iostat: Report CPU and I/O statistics

## Best Practices and Considerations

1. Always check return values of memory allocation functions and handle out-of-memory conditions.
2. Use appropriate file locking mechanisms to prevent race conditions in multi-process scenarios.
3. Consider using memory mapping for large files to improve I/O performance.
4. Be cautious with asynchronous I/O and ensure proper error handling and resource cleanup.
5. Profile your applications to identify memory bottlenecks and I/O performance issues.

## Additional Resources

1. "Advanced Programming in the UNIX Environment" by W. Richard Stevens and Stephen A. Rago
2. "The Linux Programming Interface" by Michael Kerrisk
3. Linux man pages: man 2 mmap, man 2 fcntl, man 7 aio
4. Valgrind documentation: https://valgrind.org/docs/manual/manual.html
