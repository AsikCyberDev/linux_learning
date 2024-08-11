
# Day 2: File Systems

## 1. VFS (Virtual File System): Abstraction Layer for File Systems

### Introduction
The Virtual File System (VFS) is a key component of the Linux kernel that provides a common interface for various file systems. It acts as an abstraction layer, allowing different file systems to coexist and be accessed through a unified API.

### Key Concepts

1. **VFS Objects**
   - Superblock: Represents a mounted filesystem
   - Inode: Represents a file or directory
   - Dentry: Represents a directory entry
   - File: Represents an open file

2. **VFS Operations**
   - File operations: read, write, seek, etc.
   - Inode operations: create, link, unlink, etc.
   - Superblock operations: mount, unmount, etc.

3. **Mount Points**
   - Where filesystems are attached to the directory tree

4. **Filesystem Types**
   - Disk-based: ext4, XFS, Btrfs
   - Network: NFS, SMB
   - Special: procfs, sysfs

### Practical Example: Implementing a Simple File System

```c
#include <linux/fs.h>
#include <linux/init.h>
#include <linux/module.h>

static struct super_operations simplefs_ops = {
    .drop_inode = generic_delete_inode,
};

static int simplefs_fill_super(struct super_block *sb, void *data, int silent)
{
    struct inode *root_inode;

    sb->s_magic = 0x12345678;
    sb->s_op = &simplefs_ops;

    root_inode = new_inode(sb);
    if (!root_inode)
        return -ENOMEM;

    root_inode->i_ino = 1;
    root_inode->i_sb = sb;
    root_inode->i_atime = root_inode->i_mtime = root_inode->i_ctime = current_time(root_inode);
    inode_init_owner(root_inode, NULL, S_IFDIR);

    sb->s_root = d_make_root(root_inode);
    if (!sb->s_root)
        return -ENOMEM;

    return 0;
}

static struct dentry *simplefs_mount(struct file_system_type *fs_type,
    int flags, const char *dev_name, void *data)
{
    return mount_nodev(fs_type, flags, data, simplefs_fill_super);
}

static struct file_system_type simplefs_type = {
    .owner      = THIS_MODULE,
    .name       = "simplefs",
    .mount      = simplefs_mount,
    .kill_sb    = kill_anon_super,
};

static int __init simplefs_init(void)
{
    return register_filesystem(&simplefs_type);
}

static void __exit simplefs_exit(void)
{
    unregister_filesystem(&simplefs_type);
}

module_init(simplefs_init);
module_exit(simplefs_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
```

### Gotchas and Best Practices

1. **Caching**: Understand the implications of VFS caching on file operations.
2. **Concurrency**: Be aware of locking mechanisms when implementing filesystem operations.
3. **Error Handling**: Properly handle and report errors in filesystem operations.
4. **Performance**: Consider the performance impact of VFS operations, especially for network filesystems.

### Interview Questions

1. Q: What is the purpose of the VFS in Linux?
   A: The VFS provides a common interface for different filesystems, allowing them to coexist and be accessed through a unified API.

2. Q: Explain the relationship between inodes and dentries.
   A: An inode represents a file or directory in the filesystem, while a dentry (directory entry) represents a component in a pathname and links the filename to the inode.

3. Q: How does the VFS handle different filesystem types?
   A: The VFS defines a set of common operations that each filesystem must implement. When a filesystem is mounted, it registers its specific implementations of these operations with the VFS.

4. Q: What is the difference between a block device and a character device in the context of filesystems?
   A: Block devices (like hard drives) allow random access to data in fixed-size blocks, while character devices (like keyboards) provide a stream of characters and are accessed sequentially.

5. Q: How does the VFS contribute to the Unix philosophy of "everything is a file"?
   A: The VFS allows various resources (including hardware devices, network sockets, and kernel data structures) to be represented as files, providing a uniform interface for interacting with these resources.

## 2. Inodes and File Descriptors: Core File System Concepts

### Introduction
Inodes and file descriptors are fundamental concepts in Unix-like operating systems, including Linux. They play crucial roles in how the system manages and accesses files.

### Key Concepts

1. **Inodes**
   - Contain metadata about a file (size, permissions, timestamps, etc.)
   - Do not contain the filename or file data
   - Unique identifier for a file within a filesystem

2. **File Descriptors**
   - Integer that uniquely identifies an open file for a process
   - Acts as an index into the process's table of open files

3. **Hard Links**
   - Multiple directory entries pointing to the same inode
   - Allow a file to have multiple names

4. **Soft Links (Symbolic Links)**
   - Special file type that contains a path to another file or directory
   - Can span different filesystems

### Practical Example: Working with Inodes and File Descriptors

```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/stat.h>

int main() {
    int fd;
    struct stat file_stat;

    // Open a file and get its file descriptor
    fd = open("example.txt", O_RDONLY);
    if (fd == -1) {
        perror("Error opening file");
        return 1;
    }

    // Get file information
    if (fstat(fd, &file_stat) == -1) {
        perror("Error getting file info");
        close(fd);
        return 1;
    }

    printf("File Descriptor: %d\n", fd);
    printf("Inode number: %lu\n", file_stat.st_ino);
    printf("Number of hard links: %lu\n", file_stat.st_nlink);
    printf("File size: %ld bytes\n", file_stat.st_size);

    close(fd);
    return 0;
}
```

### Inode Structure (simplified)

```c
struct inode {
    umode_t         i_mode;     // File type and permissions
    uid_t           i_uid;      // User ID of owner
    gid_t           i_gid;      // Group ID of owner
    loff_t          i_size;     // File size
    struct timespec i_atime;    // Last access time
    struct timespec i_mtime;    // Last modification time
    struct timespec i_ctime;    // Last status change time
    unsigned long   i_ino;      // Inode number
    union {
        struct pipe_inode_info *i_pipe;
        struct block_device *i_bdev;
        struct cdev *i_cdev;
    };
    // ... many more fields ...
};
```

### Gotchas and Best Practices

1. **File Descriptor Limits**: Be aware of system and per-process limits on open file descriptors.
2. **Inode Exhaustion**: Large numbers of small files can exhaust available inodes before disk space is full.
3. **Symlink Security**: Be cautious with symlinks, especially in security-sensitive contexts.
4. **File Descriptor Leaks**: Always close file descriptors when they're no longer needed.

### Interview Questions

1. Q: What information is stored in an inode?
   A: An inode stores metadata about a file, including file size, owner, permissions, timestamps, and pointers to the file's data blocks. It does not store the filename.

2. Q: How are filenames associated with inodes?
   A: Filenames are stored in directory entries (dentries), which map names to inode numbers.

3. Q: What's the difference between a hard link and a symbolic link?
   A: A hard link is a directory entry that points directly to an inode, while a symbolic link is a special file that contains a path to another file or directory.

4. Q: What happens to a file's data when all hard links to it are deleted?
   A: When the last hard link to a file is deleted, and no process has the file open, the inode and associated data blocks are freed, effectively deleting the file.

5. Q: How does the operating system keep track of open files for a process?
   A: The OS maintains a file descriptor table for each process, mapping small integer file descriptors to entries in the system-wide open file table.

## 3. File Operations: read, write, seek, and their implementations

### Introduction
File operations are fundamental to any operating system. In Linux, these operations are implemented through system calls that interact with the VFS layer, which then communicates with the specific filesystem implementation.

### Key Concepts

1. **Basic File Operations**
   - `open()`: Opens a file and returns a file descriptor
   - `read()`: Reads data from a file
   - `write()`: Writes data to a file
   - `lseek()`: Changes the file offset for read/write operations
   - `close()`: Closes a file descriptor

2. **File Offset**
   - Current position in the file for read/write operations
   - Maintained by the kernel for each open file descriptor

3. **Buffering**
   - Kernel buffers (Page Cache)
   - User-space buffers (e.g., stdio library)

4. **Synchronous vs. Asynchronous I/O**
   - Synchronous: Process waits for I/O to complete
   - Asynchronous: Process continues execution while I/O is performed

### Practical Example: File Operations in C

```c
#include <stdio.h>
#include <fcntl.h>
#include <unistd.h>
#include <string.h>

int main() {
    int fd;
    char buffer[100];
    ssize_t bytes_read, bytes_written;
    off_t offset;

    // Open file
    fd = open("example.txt", O_RDWR | O_CREAT, 0644);
    if (fd == -1) {
        perror("Error opening file");
        return 1;
    }

    // Write to file
    const char *text = "Hello, World!\n";
    bytes_written = write(fd, text, strlen(text));
    if (bytes_written == -1) {
        perror("Error writing to file");
        close(fd);
        return 1;
    }

    // Seek to the beginning of the file
    offset = lseek(fd, 0, SEEK_SET);
    if (offset == -1) {
        perror("Error seeking");
        close(fd);
        return 1;
    }

    // Read from file
    bytes_read = read(fd, buffer, sizeof(buffer) - 1);
    if (bytes_read == -1) {
        perror("Error reading from file");
        close(fd);
        return 1;
    }
    buffer[bytes_read] = '\0';  // Null-terminate the string

    printf("Read from file: %s", buffer);

    // Close file
    if (close(fd) == -1) {
        perror("Error closing file");
        return 1;
    }

    return 0;
}
```

### Implementation Details

1. **Read Operation**
   - Check user permissions
   - Acquire locks if necessary
   - Copy data from kernel space to user space
   - Update file offset

2. **Write Operation**
   - Check user permissions
   - Acquire locks if necessary
   - Copy data from user space to kernel space
   - Update file size if necessary
   - Update file offset

3. **Seek Operation**
   - Validate new offset
   - Update file offset in the file descriptor

### Gotchas and Best Practices

1. **Error Handling**: Always check return values of system calls.
2. **Buffer Sizes**: Use appropriate buffer sizes for efficient I/O.
3. **File Descriptors**: Remember to close file descriptors to avoid leaks.
4. **Large Files**: Be aware of limitations with 32-bit offsets and use appropriate functions for large files (e.g., `pread64`, `pwrite64`).

### Interview Questions

1. Q: What is the difference between buffered and unbuffered I/O?
   A: Buffered I/O (like `fread()`, `fwrite()`) uses user-space buffers to reduce system calls, while unbuffered I/O (like `read()`, `write()`) directly invokes system calls for each operation.

2. Q: How does the kernel optimize read and write operations?
   A: The kernel uses techniques like read-ahead (prefetching data) and write-back (delaying writes) to optimize I/O performance.

3. Q: What is the purpose of the `O_SYNC` flag when opening a file?
   A: `O_SYNC` ensures that each write operation is committed to the storage device before returning, providing data integrity at the cost of performance.

4. Q: Explain the difference between `fseek()` and `lseek()`.
   A: `fseek()` is a C library function that works with buffered I/O and updates the file position indicator in the `FILE` structure. `lseek()` is a system call that directly modifies the file offset in the kernel's file descriptor table.

5. Q: How does memory-mapped I/O differ from regular file I/O?
   A: Memory-mapped I/O (`mmap()`) allows a file to be mapped directly into the process's address space, allowing file access through memory operations rather than explicit read/write calls.
```

This content covers the three main topics for Day 2: VFS, Inodes and File Descriptors, and File Operations. It provides an in-depth look at each topic, including practical examples, key concepts, and interview questions.