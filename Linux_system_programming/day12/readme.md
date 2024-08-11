# Day 12: Inter-Process Communication (IPC) and Advanced I/O

## 1. Inter-Process Communication (IPC)

### Overview of IPC Mechanisms

Key Concepts:
- Types of IPC: pipes, FIFOs, message queues, shared memory, semaphores
- Choosing the right IPC mechanism for different scenarios

### Pipes and FIFOs

Key Concepts:
- Anonymous pipes for related processes
- Named pipes (FIFOs) for unrelated processes

Example: Using a pipe for parent-child communication

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>

#define BUFFER_SIZE 256

int main() {
    int pipefd[2];
    pid_t pid;
    char buffer[BUFFER_SIZE];

    if (pipe(pipefd) == -1) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }

    pid = fork();

    if (pid == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    } else if (pid == 0) {  // Child process
        close(pipefd[1]);  // Close unused write end
        read(pipefd[0], buffer, BUFFER_SIZE);
        printf("Child received: %s\n", buffer);
        close(pipefd[0]);
        exit(EXIT_SUCCESS);
    } else {  // Parent process
        close(pipefd[0]);  // Close unused read end
        strcpy(buffer, "Hello from parent!");
        write(pipefd[1], buffer, strlen(buffer) + 1);
        close(pipefd[1]);
        wait(NULL);  // Wait for child to finish
        exit(EXIT_SUCCESS);
    }

    return 0;
}
```

### System V IPC

Key Concepts:
- Message queues
- Shared memory
- Semaphores

Example: Using System V shared memory

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/ipc.h>
#include <sys/shm.h>

#define SHM_SIZE 1024

int main() {
    key_t key = ftok("shmfile", 65);
    int shmid = shmget(key, SHM_SIZE, IPC_CREAT | 0666);

    if (shmid == -1) {
        perror("shmget");
        exit(1);
    }

    char *str = (char*) shmat(shmid, NULL, 0);

    printf("Write Data : ");
    fgets(str, SHM_SIZE, stdin);

    printf("Data written in memory: %s\n", str);

    shmdt(str);

    return 0;
}
```

### POSIX IPC

Key Concepts:
- POSIX message queues
- POSIX shared memory
- POSIX semaphores

Example: Using POSIX shared memory

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <unistd.h>

#define SHM_NAME "/my_shm"
#define SHM_SIZE 1024

int main() {
    int fd = shm_open(SHM_NAME, O_CREAT | O_RDWR, 0666);
    if (fd == -1) {
        perror("shm_open");
        exit(1);
    }

    ftruncate(fd, SHM_SIZE);

    char *ptr = mmap(0, SHM_SIZE, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (ptr == MAP_FAILED) {
        perror("mmap");
        exit(1);
    }

    sprintf(ptr, "Hello, POSIX shared memory!");
    printf("Written to shared memory: %s\n", ptr);

    munmap(ptr, SHM_SIZE);
    close(fd);
    shm_unlink(SHM_NAME);

    return 0;
}
```

## 2. Advanced I/O Operations

### Memory-mapped I/O

Key Concepts:
- Using mmap() for file I/O
- Advantages and disadvantages of memory-mapped I/O

Example: Reading a file using mmap()

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

    write(STDOUT_FILENO, addr, sb.st_size);

    munmap(addr, sb.st_size);
    close(fd);

    return 0;
}
```

### Asynchronous I/O

Key Concepts:
- POSIX AIO operations
- Callback mechanisms

Example: Basic asynchronous read operation

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <fcntl.h>
#include <aio.h>

#define BUFFER_SIZE 1024

int main() {
    char buffer[BUFFER_SIZE];
    struct aiocb cb;
    int fd;

    fd = open("test.txt", O_RDONLY);
    if (fd == -1) {
        perror("open");
        exit(1);
    }

    memset(&cb, 0, sizeof(struct aiocb));
    cb.aio_nbytes = BUFFER_SIZE;
    cb.aio_fildes = fd;
    cb.aio_offset = 0;
    cb.aio_buf = buffer;

    if (aio_read(&cb) == -1) {
        perror("aio_read");
        exit(1);
    }

    while (aio_error(&cb) == EINPROGRESS) {
        // Wait for completion
    }

    int ret = aio_return(&cb);
    if (ret == -1) {
        perror("aio_return");
        exit(1);
    }

    printf("Read %d bytes: %.*s\n", ret, ret, (char*)cb.aio_buf);

    close(fd);
    return 0;
}
```

### Scatter-Gather I/O

Key Concepts:
- Using readv() and writev()
- Efficient handling of non-contiguous buffers

Example: Using writev() to write from multiple buffers

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include /uio.h>
#include <fcntl.h>

int main() {
    struct iovec iov[3];
    char *str0 = "Hello ";
    char *str1 = "scatter-gather ";
    char *str2 = "I/O!\n";

    iov[0].iov_base = str0;
    iov[0].iov_len = strlen(str0);
    iov[1].iov_base = str1;
    iov[1].iov_len = strlen(str1);
    iov[2].iov_base = str2;
    iov[2].iov_len = strlen(str2);

    int fd = open("output.txt", O_WRONLY | O_CREAT | O_TRUNC, 0644);
    if (fd == -1) {
        perror("open");
        exit(1);
    }

    ssize_t nwritten = writev(fd, iov, 3);
    if (nwritten == -1) {
        perror("writev");
        exit(1);
    }

    printf("Wrote %zd bytes\n", nwritten);

    close(fd);
    return 0;
}
```

## 3. Advanced File Operations

### File Locking

Key Concepts:
- Advisory vs. mandatory locking
- Using fcntl() for file locking

Example: Implementing a simple file lock

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    int fd = open("lockfile", O_RDWR | O_CREAT, 0644);
    if (fd == -1) {
        perror("open");
        exit(1);
    }

    struct flock fl = {
        .l_type = F_WRLCK,
        .l_whence = SEEK_SET,
        .l_start = 0,
        .l_len = 0,
    };

    printf("Trying to get lock...\n");
    if (fcntl(fd, F_SETLKW, &fl) == -1) {
        perror("fcntl");
        exit(1);
    }

    printf("Got lock. Press enter to release...\n");
    getchar();

    fl.l_type = F_UNLCK;
    if (fcntl(fd, F_SETLK, &fl) == -1) {
        perror("fcntl");
        exit(1);
    }

    printf("Released lock\n");

    close(fd);
    return 0;
}
```

### Extended Attributes

Key Concepts:
- Setting and getting extended attributes
- Use cases for extended attributes

Example: Setting and getting an extended attribute

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <attr/xattr.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <file>\n", argv[0]);
        exit(1);
    }

    const char *path = argv[1];
    const char *name = "user.comment";
    const char *value = "This is a test comment";

    if (setxattr(path, name, value, strlen(value), 0) == -1) {
        perror("setxattr");
        exit(1);
    }

    char buffer[256];
    ssize_t len = getxattr(path, name, buffer, sizeof(buffer));
    if (len == -1) {
        perror("getxattr");
        exit(1);
    }

    printf("Extended attribute '%s' value: %.*s\n", name, (int)len, buffer);

    return 0;
}
```

## Practice Exercises

1. Implement a simple client-server application using message queues for communication.

2. Create a program that uses shared memory and semaphores to implement a producer-consumer problem.

3. Write a file copy program that uses memory-mapped I/O for efficient copying of large files.

4. Develop a multi-process application that uses various IPC mechanisms to communicate and synchronize.

## Important Tools and Utilities

- ipcs: Examine IPC resources
- ipcrm: Remove IPC resources
- strace: Trace system calls and signals
- lsof: List open files

## Additional Resources

1. "The Linux Programming Interface" by Michael Kerrisk
2. "Advanced Programming in the UNIX Environment" by W. Richard Stevens and Stephen A. Rago
3. "Linux System Programming" by Robert Love
4. "POSIX Programmers Guide" by Donald Lewine
