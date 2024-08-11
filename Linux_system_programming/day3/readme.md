# Day 3: Signal Handling and Advanced I/O Operations

## 1. Signal Handling

Signals are software interrupts used for inter-process communication and handling asynchronous events.

### Basic Signal Concepts

Key Concepts:
- Signal types and their meanings
- Default signal actions
- Signal masks and blocking

Example: Handling SIGINT (Ctrl+C)

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void sigint_handler(int signo) {
    printf("Caught SIGINT! Exiting gracefully...\n");
    exit(0);
}

int main() {
    if (signal(SIGINT, sigint_handler) == SIG_ERR) {
        perror("signal");
        return 1;
    }

    while(1) {
        printf("Running...\n");
        sleep(1);
    }

    return 0;
}
```

Explanation:
- `signal()` sets up a handler for SIGINT.
- The program continues running until SIGINT is received.
- When SIGINT is caught, the handler is invoked, printing a message and exiting.

### Advanced Signal Handling with sigaction()

`sigaction()` provides more control over signal handling compared to `signal()`.

Key Concepts:
- Signal handler installation
- Specifying flags for signal behavior
- Examining and modifying signal masks

Example: Using sigaction() for more robust signal handling

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>
#include <string.h>

void sighandler(int signo, siginfo_t *info, void *context) {
    printf("Received signal %d\n", signo);
    if (signo == SIGINT) {
        printf("SIGINT received. Exiting...\n");
        exit(0);
    }
}

int main() {
    struct sigaction sa;
    memset(&sa, 0, sizeof(sa));
    sa.sa_sigaction = sighandler;
    sa.sa_flags = SA_SIGINFO;

    if (sigaction(SIGINT, &sa, NULL) == -1) {
        perror("sigaction");
        return 1;
    }

    while(1) {
        printf("Running...\n");
        sleep(1);
    }

    return 0;
}
```

Explanation:
- `sigaction()` is used to install the signal handler.
- SA_SIGINFO flag allows the handler to receive additional information about the signal.
- The handler can distinguish between different signals and act accordingly.

### Signal Sets and Blocking

Managing sets of signals and controlling which signals are blocked.

Key Concepts:
- Creating and manipulating signal sets
- Blocking and unblocking signals
- Examining pending signals

Example: Blocking and unblocking signals

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void sighandler(int signo) {
    printf("Received signal %d\n", signo);
}

int main() {
    sigset_t set, oldset;
    signal(SIGINT, sighandler);

    sigemptyset(&set);
    sigaddset(&set, SIGINT);

    // Block SIGINT
    sigprocmask(SIG_BLOCK, &set, &oldset);
    printf("SIGINT blocked. Sleeping for 5 seconds...\n");
    sleep(5);

    // Unblock SIGINT
    sigprocmask(SIG_UNBLOCK, &set, NULL);
    printf("SIGINT unblocked.\n");

    while(1) {
        pause();
    }

    return 0;
}
```

Explanation:
- `sigemptyset()` and `sigaddset()` are used to create a signal set.
- `sigprocmask()` is used to block and unblock signals.
- Blocked signals are held pending and delivered when unblocked.

## 2. Advanced I/O Operations

### Non-blocking I/O

Performing I/O operations without blocking the process.

Key Concepts:
- Setting non-blocking flag on file descriptors
- Handling EAGAIN/EWOULDBLOCK errors
- Use cases for non-blocking I/O

Example: Non-blocking read from stdin

```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>

int main() {
    char buffer[1024];
    int flags = fcntl(STDIN_FILENO, F_GETFL, 0);
    fcntl(STDIN_FILENO, F_SETFL, flags | O_NONBLOCK);

    while(1) {
        ssize_t bytes_read = read(STDIN_FILENO, buffer, sizeof(buffer) - 1);
        if (bytes_read >= 0) {
            buffer[bytes_read] = '\0';
            printf("Read: %s", buffer);
        } else if (errno == EAGAIN || errno == EWOULDBLOCK) {
            printf("No data available\n");
        } else {
            perror("read");
            break;
        }
        sleep(1);
    }

    return 0;
}
```

Explanation:
- `fcntl()` is used to set the non-blocking flag on stdin.
- The read operation doesn't block when no data is available.
- EAGAIN or EWOULDBLOCK indicate that the operation would block in blocking mode.

### Asynchronous I/O (AIO)

Performing I/O operations asynchronously, allowing the program to continue execution.

Key Concepts:
- AIO control block (aiocb) structure
- Initiating asynchronous reads and writes
- Checking for completion of AIO operations

Example: Asynchronous read operation

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
    int fd = open("testfile.txt", O_RDONLY);
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

    while(aio_error(&cb) == EINPROGRESS) {
        printf("Reading...\n");
        sleep(1);
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
- An aiocb structure is set up to describe the asynchronous operation.
- `aio_read()` initiates the asynchronous read operation.
- `aio_error()` is used to check the status of the operation.
- `aio_return()` retrieves the return status of the completed operation.

### Memory-mapped I/O

Using memory mapping for file I/O operations.

Key Concepts:
- Creating memory mappings with mmap()
- Synchronizing mapped memory with msync()
- Advantages and use cases of memory-mapped I/O

Example: Using memory-mapped I/O to read and modify a file

```c
#include <stdio.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>

int main() {
    char *addr;
    int fd = open("testfile.txt", O_RDWR);
    if (fd == -1) {
        perror("open");
        return 1;
    }

    struct stat sb;
    if (fstat(fd, &sb) == -1) {
        perror("fstat");
        close(fd);
        return 1;
    }

    addr = mmap(NULL, sb.st_size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (addr == MAP_FAILED) {
        perror("mmap");
        close(fd);
        return 1;
    }

    printf("File contents: %s\n", addr);

    // Modify the mapped memory
    memcpy(addr, "Hello, memory-mapped I/O!", 25);

    // Synchronize the mapping with the file
    if (msync(addr, sb.st_size, MS_SYNC) == -1) {
        perror("msync");
    }

    munmap(addr, sb.st_size);
    close(fd);
    return 0;
}
```

Explanation:
- `mmap()` creates a memory mapping of the file.
- The file can be read and modified through the memory mapping.
- `msync()` ensures changes are written back to the file.
- `munmap()` removes the mapping when it's no longer needed.

## Practice Exercises

1. Implement a program that uses signals to communicate between a parent and child process.

2. Create a simple server that uses non-blocking I/O to handle multiple client connections without using threads or forking.

3. Develop a file copy program that uses memory-mapped I/O for efficient copying of large files.

4. Write a program that performs multiple asynchronous I/O operations concurrently and waits for their completion.

## Important Tools and Utilities

- strace: Trace system calls and signals
- gdb: Debug programs and examine signal handling
- ltrace: Library call tracer
- valgrind: Memory debugging and profiling tool

## Best Practices and Considerations

1. Always restore signal handlers if modifying them temporarily.
2. Be aware of race conditions in signal handlers and use async-signal-safe functions.
3. Use appropriate error checking for all I/O operations, especially with non-blocking and asynchronous I/O.
4. Consider the trade-offs between different I/O methods based on the specific use case and performance requirements.
5. Be cautious with memory-mapped I/O when dealing with files that can change size.

## Additional Resources

1. "Advanced Programming in the UNIX Environment" by W. Richard Stevens and Stephen A. Rago
2. Linux man pages: man 7 signal, man 2 sigaction, man 2 aio_read, man 2 mmap
3. "The Linux Programming Interface" by Michael Kerrisk
4. POSIX Asynchronous I/O documentation: https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/aio.h.html
