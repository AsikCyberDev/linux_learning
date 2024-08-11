# Day 5: Networking and I/O

## 1. Networking: TCP/IP Stack and Socket Programming

### Introduction
The Linux networking stack implements the TCP/IP protocol suite, providing a robust foundation for network communication. Understanding the TCP/IP stack and socket programming is crucial for developing networked applications on Linux.

### Key Concepts

1. **TCP/IP Layers**
   - Application Layer (HTTP, FTP, SSH, etc.)
   - Transport Layer (TCP, UDP)
   - Network Layer (IP)
   - Link Layer (Ethernet, Wi-Fi)

2. **Socket Types**
   - Stream Sockets (TCP)
   - Datagram Sockets (UDP)
   - Raw Sockets

3. **Socket API**
   - `socket()`, `bind()`, `listen()`, `accept()`, `connect()`
   - `send()`, `recv()`, `sendto()`, `recvfrom()`

4. **Network Byte Order**
   - Big-endian for network communications
   - `htons()`, `ntohs()`, `htonl()`, `ntohl()`

5. **Non-blocking I/O and Multiplexing**
   - `select()`, `poll()`, `epoll()`

### Practical Example: TCP Server and Client

#### TCP Server

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <arpa/inet.h>

#define PORT 8080
#define BUFFER_SIZE 1024

int main() {
    int server_fd, new_socket;
    struct sockaddr_in address;
    int opt = 1;
    int addrlen = sizeof(address);
    char buffer[BUFFER_SIZE] = {0};

    // Creating socket file descriptor
    if ((server_fd = socket(AF_INET, SOCK_STREAM, 0)) == 0) {
        perror("socket failed");
        exit(EXIT_FAILURE);
    }

    // Forcefully attaching socket to the port 8080
    if (setsockopt(server_fd, SOL_SOCKET, SO_REUSEADDR | SO_REUSEPORT, &opt, sizeof(opt))) {
        perror("setsockopt");
        exit(EXIT_FAILURE);
    }
    address.sin_family = AF_INET;
    address.sin_addr.s_addr = INADDR_ANY;
    address.sin_port = htons(PORT);

    // Forcefully attaching socket to the port 8080
    if (bind(server_fd, (struct sockaddr *)&address, sizeof(address)) < 0) {
        perror("bind failed");
        exit(EXIT_FAILURE);
    }
    if (listen(server_fd, 3) < 0) {
        perror("listen");
        exit(EXIT_FAILURE);
    }
    if ((new_socket = accept(server_fd, (struct sockaddr *)&address, (socklen_t*)&addrlen)) < 0) {
        perror("accept");
        exit(EXIT_FAILURE);
    }
    read(new_socket, buffer, BUFFER_SIZE);
    printf("Message from client: %s\n", buffer);
    send(new_socket, "Hello from server", strlen("Hello from server"), 0);
    printf("Hello message sent\n");
    return 0;
}
```

#### TCP Client

```c
#include <stdio.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>

#define PORT 8080

int main(int argc, char const *argv[]) {
    int sock = 0, valread;
    struct sockaddr_in serv_addr;
    char *hello = "Hello from client";
    char buffer[1024] = {0};

    if ((sock = socket(AF_INET, SOCK_STREAM, 0)) < 0) {
        printf("\n Socket creation error \n");
        return -1;
    }

    serv_addr.sin_family = AF_INET;
    serv_addr.sin_port = htons(PORT);

    // Convert IPv4 and IPv6 addresses from text to binary form
    if(inet_pton(AF_INET, "127.0.0.1", &serv_addr.sin_addr) <= 0) {
        printf("\nInvalid address/ Address not supported \n");
        return -1;
    }

    if (connect(sock, (struct sockaddr *)&serv_addr, sizeof(serv_addr)) < 0) {
        printf("\nConnection Failed \n");
        return -1;
    }
    send(sock, hello, strlen(hello), 0);
    printf("Hello message sent\n");
    valread = read(sock, buffer, 1024);
    printf("Message from server: %s\n", buffer);
    return 0;
}
```

### Gotchas and Best Practices

1. **Error Handling**: Always check return values and handle errors appropriately.
2. **Resource Management**: Close sockets and free resources when they're no longer needed.
3. **Blocking vs Non-Blocking**: Understand the implications of blocking and non-blocking I/O.
4. **Buffer Overflows**: Be cautious with buffer sizes to prevent overflows.
5. **Network Byte Order**: Remember to convert between host and network byte order.

### Interview Questions

1. Q: What is the difference between TCP and UDP?
   A: TCP is a connection-oriented protocol that provides reliable, ordered, and error-checked delivery of data. UDP is a connectionless protocol that provides a best-effort delivery without guarantees of ordering or reliability.

2. Q: Explain the purpose of the `listen()` function in socket programming.
   A: The `listen()` function marks a socket as passive, indicating that it will be used to accept incoming connection requests using `accept()`. It also sets the queue length for pending connections.

3. Q: What is socket multiplexing, and why is it useful?
   A: Socket multiplexing allows a program to monitor multiple file descriptors (including sockets) simultaneously. It's useful for handling multiple connections efficiently without using multiple threads or processes.

4. Q: How does the `select()` system call work, and what are its limitations?
   A: `select()` allows a program to monitor multiple file descriptors, waiting until one or more become ready for I/O operations. Its main limitation is that it's inefficient for a large number of file descriptors, as it requires scanning all monitored descriptors on each call.

5. Q: What is the purpose of the `SO_REUSEADDR` socket option?
   A: `SO_REUSEADDR` allows the socket to bind to an address that is already in use. This is particularly useful for servers that want to bind to the same address after a restart, even if there are still connections in the TIME_WAIT state.

## 2. I/O Models: Blocking, Non-blocking, and Asynchronous I/O

### Introduction
Linux provides several I/O models, each with its own characteristics and use cases. Understanding these models is crucial for developing efficient and scalable applications, especially those dealing with network or file I/O.

### Key Concepts

1. **Blocking I/O**
   - Default model
   - Process blocks until I/O operation completes

2. **Non-blocking I/O**
   - I/O operations return immediately if they can't be completed
   - Requires polling to check for completion

3. **I/O Multiplexing**
   - `select()`, `poll()`, `epoll()`
   - Allows monitoring multiple file descriptors

4. **Signal-driven I/O**
   - Process receives a signal when I/O is possible

5. **Asynchronous I/O**
   - AIO library
   - I/O operations complete in the background

### Practical Examples

#### Blocking I/O

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>

int main() {
    char buffer[1024];
    int fd = open("test.txt", O_RDONLY);
    if (fd == -1) {
        perror("open");
        return 1;
    }

    ssize_t bytes_read = read(fd, buffer, sizeof(buffer));
    if (bytes_read == -1) {
        perror("read");
        close(fd);
        return 1;
    }

    printf("Read %zd bytes\n", bytes_read);
    close(fd);
    return 0;
}
```

#### Non-blocking I/O

```c
#include <stdio.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>

int main() {
    char buffer[1024];
    int fd = open("test.txt", O_RDONLY | O_NONBLOCK);
    if (fd == -1) {
        perror("open");
        return 1;
    }

    while (1) {
        ssize_t bytes_read = read(fd, buffer, sizeof(buffer));
        if (bytes_read == -1) {
            if (errno == EAGAIN || errno == EWOULDBLOCK) {
                printf("No data available right now\n");
                sleep(1);  // Wait a bit before trying again
                continue;
            } else {
                perror("read");
                break;
            }
        } else if (bytes_read == 0) {
            printf("End of file reached\n");
            break;
        } else {
            printf("Read %zd bytes\n", bytes_read);
            break;
        }
    }

    close(fd);
    return 0;
}
```

#### I/O Multiplexing with `select()`

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

int main() {
    fd_set rfds;
    struct timeval tv;
    int retval;

    // Watch stdin (fd 0) to see when it has input.
    FD_ZERO(&rfds);
    FD_SET(0, &rfds);

    // Wait up to five seconds.
    tv.tv_sec = 5;
    tv.tv_usec = 0;

    retval = select(1, &rfds, NULL, NULL, &tv);

    if (retval == -1)
        perror("select()");
    else if (retval)
        printf("Data is available now.\n");
    else
        printf("No data within five seconds.\n");

    return 0;
}
```

### Gotchas and Best Practices

1. **Blocking vs Performance**: Be aware of how blocking I/O can affect application responsiveness.
2. **Error Handling**: Pay attention to error codes, especially with non-blocking I/O.
3. **Scalability**: Consider using `epoll()` for high-performance servers handling many connections.
4. **Timeout Handling**: Implement proper timeout mechanisms to avoid indefinite waits.
5. **Resource Management**: Ensure proper cleanup of file descriptors and other resources.

### Interview Questions

1. Q: What are the main differences between blocking and non-blocking I/O?
   A: Blocking I/O causes the process to wait until the I/O operation completes, while non-blocking I/O returns immediately with an error if the operation would block. Non-blocking I/O allows for more responsive applications but requires more complex programming.

2. Q: Explain the advantages and disadvantages of using `select()` for I/O multiplexing.
   A: Advantages include the ability to monitor multiple file descriptors and set timeouts. Disadvantages include poor scalability for large numbers of file descriptors and the need to rebuild the fd_set after each call.

3. Q: How does `epoll()` improve upon `select()` and `poll()`?
   A: `epoll()` provides better performance for a large number of file descriptors. It uses an edge-triggered interface and doesn't require scanning all monitored file descriptors on each call.

4. Q: What is the key difference between asynchronous I/O and non-blocking I/O?
   A: Non-blocking I/O returns immediately if the operation would block, requiring the application to poll for completion. Asynchronous I/O initiates the operation and notifies the application when it's complete, allowing the application to continue execution without polling.

5. Q: In what scenarios might you choose to use signal-driven I/O?
   A: Signal-driven I/O can be useful in situations where you want to handle I/O events asynchronously without dedicating a thread to polling. It's particularly useful in single-threaded applications that need to handle multiple I/O sources efficiently.

## 3. File Systems: VFS, Inodes, and File Operations

### Introduction
The Linux Virtual File System (VFS) provides a common interface for various file systems. Understanding the VFS, inodes, and file operations is crucial for working with files and developing file system-related software on Linux.

### Key Concepts

1. **Virtual File System (VFS)**
   - Abstraction layer for different file systems
   - Provides a common interface for file operations

2. **Inodes**
   - Data structures that store metadata about files
   - Contains information like permissions, timestamps, and data block pointers

3. **File Descriptors**
   - Integer handles for open files
   - Used by processes to perform file operations

4. **File Operations**
   - open(), read(), write(), close(), lseek(), etc.
   - Manipulate files through the VFS interface

5. **Mount Points**
   - Points in the directory tree where file systems are attached

### Practical Example: File Operations

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/stat.h>

int main() {
    int fd;
    char buffer[100];
    ssize_t bytes_read, bytes_written;
    off_t offset;

    // Open file
    fd = open("test.txt", O_RDWR | O_CREAT, S_IRUSR | S_IWUSR);
    if (fd == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }

    // Write to file
    bytes_written = write(fd, "Hello, VFS!\n", 12);
    if (bytes_written != 12) {
        perror("write");
        close(fd);
        exit(EXIT_FAILURE);
    }

    // Move file pointer to beginning
    offset = lseek(fd, 0, SEEK_SET);
    if (offset == -1) {
        perror("lseek");
        close(fd);
        exit(EXIT_FAILURE);
    }

    // Read from file
    bytes_read = read(fd, buffer, sizeof(buffer) - 1);
    if (bytes_read == -1) {
        perror("read");
        close(fd);
        exit(EXIT_FAILURE);
    }
    buffer[bytes_read] = '\0';

    printf("Read from file: %s", buffer);

    // Get file information
    struct stat file_stat;
    if (fstat(fd, &file_stat) == -1) {
        perror("fstat");
        close(fd);
        exit(EXIT_FAILURE);
    }

    printf("File size: %ld bytes\n", file_stat.st_size);
    printf("Inode number: %ld\n", file_stat.st_ino);

    // Close file
    if (close(fd) == -1) {
        perror("close");
        exit(EXIT_FAILURE);
    }

    return 0;
}
```

### Gotchas and Best Practices

1. **File Descriptor Limits**: Be aware of system limits on open file descriptors.
2. **Error Handling**: Always check return values of file operations and handle errors appropriately.
3. **Resource Management**: Ensure files are properly closed to avoid resource leaks.
4. **Buffering**: Understand the differences between buffered and unbuffered I/O.
5. **Permissions**: Pay attention to file permissions when creating or accessing files.

### Interview Questions

1. Q: What is the purpose of the Virtual File System (VFS) in Linux?
   A: The VFS provides an abstraction layer that allows different file systems to coexist and be accessed through a common interface. It enables applications to interact with various file systems using the same system calls, regardless of the underlying file system type.

2. Q: Explain the relationship between inodes and file names.
   A: An inode is a data structure that stores metadata about a file, including its permissions, timestamps, and data block pointers. File names are stored in directory entries, which map names to inode numbers. This allows multiple file names (hard links) to reference the same inode.

3. Q: What is the difference between a hard link and a symbolic link?
   A: A hard link is a directory entry that points directly to an inode, allowing multiple names for the same file. A symbolic link is a special file that contains a path to another file or directory. Hard links can only be created for files on the same file system, while symbolic links can cross file system boundaries.

4. Q: How does the `lseek()` function work, and why is it useful?
   A: `lseek()` changes the file offset for a file descriptor. It's useful for random access to files, allowing you to read from or write to specific positions within a file. It can also be used to extend a file's size by seeking past the end and writing data.

5. Q: What happens when a process opens a file, and how does the kernel manage this?
   A: When a process opens a file, the kernel creates a file descriptor, which is an integer handle for the open file. The kernel maintains a file descriptor table for each process, mapping file descriptors to open file descriptions. These descriptions contain information like the current file offset and access mode.

## 4. Advanced I/O: Memory-mapped I/O and Direct I/O

### Introduction
Advanced I/O techniques like memory-mapped I/O and direct I/O can provide significant performance improvements for certain types of applications. Understanding these techniques is crucial for optimizing I/O-intensive operations.

### Key Concepts

1. **Memory-mapped I/O**
   - Maps a file or device into memory
   - Allows file access through memory operations

2. **Direct I/O**
   - Bypasses the kernel's page cache
   - Useful for applications managing their own caching

3. **Page Cache**
   - Kernel's cache for file contents
   - Improves read/write performance for frequently accessed data

4. **O_DIRECT Flag**
   - Used to enable direct I/O
   - Requires aligned buffers and block sizes

5. **mmap() System Call**
   - Creates a memory mapping of a file or device

### Practical Example: Memory-mapped I/O

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/mman.h>
#include <string.h>

int main() {
    int fd;
    char *map;
    size_t file_size = 4096;  // 1 page size

    // Open file
    fd = open("mmaptest.txt", O_RDWR | O_CREAT, S_IRUSR | S_IWUSR);
    if (fd == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }

    // Extend file size to 4096 bytes
    if (ftruncate(fd, file_size) == -1) {
        perror("ftruncate");
        close(fd);
        exit(EXIT_FAILURE);
    }

    // Memory-map the file
    map = mmap(NULL, file_size, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (map == MAP_FAILED) {
        perror("mmap");
        close(fd);
        exit(EXIT_FAILURE);
    }

    // Write to the memory-mapped region
    strcpy(map, "Hello, memory-mapped file!");

    // Sync changes to disk
    if (msync(map, file_size, MS_SYNC) == -1) {
        perror("msync");
    }

    // Unmap the file
    if (munmap(map, file_size) == -1) {
        perror("munmap");
    }

    close(fd);
    return 0;
}
```

### Gotchas and Best Practices

1. **Alignment**: Ensure proper alignment when using direct I/O.
2. **Page Size**: Be aware of the system's page size when using memory-mapped I/O.
3. **Synchronization**: Use `msync()` to ensure changes are written to disk with memory-mapped I/O.
4. **File Size Changes**: Be cautious when the file size changes during memory-mapped I/O.
5. **Performance Considerations**: Evaluate whether advanced I/O techniques provide benefits for your specific use case.

### Interview Questions

1. Q: What are the main advantages of using memory-mapped I/O?
   A: Memory-mapped I/O can provide faster access to file data, especially for random access patterns. It allows treating file contents as memory, which can simplify code. It also enables sharing of mapped regions between processes.

2. Q: In what scenarios might direct I/O be preferred over standard buffered I/O?
   A: Direct I/O is useful when an application manages its own caching and wants to avoid double buffering (in both the application and the kernel). It's often used in database systems and other applications that need fine-grained control over I/O operations.

3. Q: What are the potential drawbacks of using memory-mapped I/O?
   A: Drawbacks include the possibility of segmentation faults if accessing unmapped regions, potential issues with files that change size, and the need for careful synchronization in multi-threaded scenarios. Also, for small files, the overhead of creating a mapping might outweigh the benefits.

4. Q: How does the page cache affect file I/O performance, and how does direct I/O interact with it?
   A: The page cache improves I/O performance by keeping frequently accessed data in memory, reducing disk access. Direct I/O bypasses the page cache, which can be beneficial when an application has its own caching strategy but may reduce performance for data that would benefit from caching.

5. Q: Explain the significance of the `MAP_SHARED` and `MAP_PRIVATE` flags in `mmap()`.
   A: `MAP_SHARED` creates a mapping that's shared with other processes mapping the same file, and changes are visible to other processes and are written back to the file. `MAP_PRIVATE` creates a private copy-on-write mapping, where changes are not visible to other processes and are not written back to the file.

This completes the content for Day 5, covering Networking, I/O Models, File Systems, and Advanced I/O techniques. The material provides a comprehensive overview of these topics, including practical examples and interview questions to reinforce understanding.
```

This concludes the content for Day 5, covering all the requested topics in detail.