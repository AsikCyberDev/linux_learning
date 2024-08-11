# Day 2: Inter-Process Communication (IPC) Mechanisms

## 1. Pipes and Named Pipes (FIFOs)

### Anonymous Pipes

Anonymous pipes provide a way for related processes to communicate.

Key Concepts:
- Unidirectional communication
- Used between parent and child processes
- File descriptor passing

Example:
```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>

int main() {
    int pipefd[2];
    char buf[30];

    if (pipe(pipefd) == -1) {
        perror("pipe");
        return 1;
    }

    if (fork() == 0) {  // Child process
        close(pipefd[1]);  // Close unused write end
        read(pipefd[0], buf, sizeof(buf));
        printf("Child received: %s\n", buf);
        close(pipefd[0]);
    } else {  // Parent process
        close(pipefd[0]);  // Close unused read end
        write(pipefd[1], "Hello from parent!", 18);
        close(pipefd[1]);
    }

    return 0;
}
```

Explanation:
- `pipe()` creates a pipe with two file descriptors: pipefd[0] for reading and pipefd[1] for writing.
- The parent writes to the pipe, and the child reads from it.
- Unused ends of the pipe are closed to prevent resource leaks.

### Named Pipes (FIFOs)

FIFOs allow communication between unrelated processes.

Key Concepts:
- Persistent in the filesystem
- Can be used by any process with appropriate permissions
- Blocking I/O operations

Example:
```c
#include <stdio.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <unistd.h>

#define FIFO_NAME "/tmp/myfifo"

int main() {
    mkfifo(FIFO_NAME, 0666);

    if (fork() == 0) {  // Child process (reader)
        int fd = open(FIFO_NAME, O_RDONLY);
        char buf[30];
        read(fd, buf, sizeof(buf));
        printf("Received: %s\n", buf);
        close(fd);
    } else {  // Parent process (writer)
        int fd = open(FIFO_NAME, O_WRONLY);
        write(fd, "Hello through FIFO!", 19);
        close(fd);
    }

    unlink(FIFO_NAME);
    return 0;
}
```

Explanation:
- `mkfifo()` creates a named pipe in the filesystem.
- The child opens the FIFO for reading, while the parent opens it for writing.
- `unlink()` removes the FIFO from the filesystem after use.

## 2. System V IPC Mechanisms

### Message Queues

Message queues allow processes to exchange formatted data packets.

Key Concepts:
- Message types for selective receiving
- Persistent until explicitly removed
- Can hold multiple messages

Example:
```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/msg.h>

struct msg_buffer {
    long msg_type;
    char msg_text[100];
} message;

int main() {
    key_t key = ftok("progfile", 65);
    int msgid = msgget(key, 0666 | IPC_CREAT);

    message.msg_type = 1;
    strcpy(message.msg_text, "Hello from sender");
    msgsnd(msgid, &message, sizeof(message), 0);

    msgrcv(msgid, &message, sizeof(message), 1, 0);
    printf("Received: %s\n", message.msg_text);

    msgctl(msgid, IPC_RMID, NULL);

    return 0;
}
```

Explanation:
- `ftok()` generates a unique key for the message queue.
- `msgget()` creates or accesses a message queue.
- `msgsnd()` sends a message to the queue.
- `msgrcv()` receives a message from the queue.
- `msgctl()` with IPC_RMID removes the message queue.

### Shared Memory

Shared memory allows multiple processes to access a common memory segment.

Key Concepts:
- Fastest IPC mechanism
- Requires synchronization between processes
- Persistent until explicitly removed

Example:
```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <string.h>

int main() {
    key_t key = ftok("shmfile", 65);
    int shmid = shmget(key, 1024, 0666|IPC_CREAT);
    char *str = (char*) shmat(shmid, (void*)0, 0);

    printf("Write Data : ");
    gets(str);

    printf("Data written in memory: %s\n", str);

    shmdt(str);
    shmctl(shmid, IPC_RMID, NULL);

    return 0;
}
```

Explanation:
- `shmget()` creates or accesses a shared memory segment.
- `shmat()` attaches the shared memory segment to the process's address space.
- `shmdt()` detaches the shared memory segment.
- `shmctl()` with IPC_RMID removes the shared memory segment.

### Semaphores

Semaphores provide a synchronization mechanism for processes and threads.

Key Concepts:
- Can be binary (mutex) or counting semaphores
- Used for process synchronization and mutual exclusion
- Operations are atomic

Example:
```c
#include <stdio.h>
#include <sys/ipc.h>
#include <sys/sem.h>

union semun {
    int val;
    struct semid_ds *buf;
    unsigned short *array;
};

int main() {
    key_t key = ftok("semfile", 65);
    int semid = semget(key, 1, 0666 | IPC_CREAT);

    union semun arg;
    arg.val = 1;
    semctl(semid, 0, SETVAL, arg);

    struct sembuf sb = {0, -1, SEM_UNDO};
    semop(semid, &sb, 1);  // Wait

    printf("Critical section\n");

    sb.sem_op = 1;
    semop(semid, &sb, 1);  // Signal

    semctl(semid, 0, IPC_RMID);

    return 0;
}
```

Explanation:
- `semget()` creates or accesses a semaphore set.
- `semctl()` initializes the semaphore value.
- `semop()` performs wait (P) and signal (V) operations on the semaphore.
- IPC_RMID removes the semaphore set.

## 3. POSIX IPC Mechanisms

### POSIX Message Queues

Similar to System V message queues but with a different API.

Key Concepts:
- Named queues accessible via filesystem-like names
- Support for message priorities
- Asynchronous notification of message arrival

Example:
```c
#include <fcntl.h>
#include <sys/stat.h>
#include <mqueue.h>
#include <stdio.h>
#include <string.h>

#define MQ_NAME "/test_queue"

int main() {
    mqd_t mq;
    struct mq_attr attr;
    char buffer[1024];

    attr.mq_flags = 0;
    attr.mq_maxmsg = 10;
    attr.mq_msgsize = 1024;
    attr.mq_curmsgs = 0;

    mq = mq_open(MQ_NAME, O_CREAT | O_RDWR, 0644, &attr);

    mq_send(mq, "Hello, POSIX MQ!", 16, 0);

    mq_receive(mq, buffer, 1024, NULL);
    printf("Received: %s\n", buffer);

    mq_close(mq);
    mq_unlink(MQ_NAME);

    return 0;
}
```

Explanation:
- `mq_open()` creates or opens a message queue.
- `mq_send()` sends a message to the queue.
- `mq_receive()` receives a message from the queue.
- `mq_close()` closes the queue descriptor.
- `mq_unlink()` removes the queue from the system.

### POSIX Shared Memory

Provides memory-mapped files for IPC.

Key Concepts:
- Uses file descriptors and memory mapping
- Can be used with file locking for synchronization
- Persistent until explicitly unlinked

Example:
```c
#include <fcntl.h>
#include <sys/mman.h>
#include <stdio.h>
#include <string.h>
#include <unistd.h>

#define SHM_NAME "/test_shm"

int main() {
    int fd = shm_open(SHM_NAME, O_CREAT | O_RDWR, 0644);
    ftruncate(fd, 1024);

    void *ptr = mmap(NULL, 1024, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);

    sprintf(ptr, "Hello, POSIX shared memory!");
    printf("Written to shared memory: %s\n", (char*)ptr);

    munmap(ptr, 1024);
    close(fd);
    shm_unlink(SHM_NAME);

    return 0;
}
```

Explanation:
- `shm_open()` creates or opens a shared memory object.
- `ftruncate()` sets the size of the shared memory object.
- `mmap()` maps the shared memory into the process's address space.
- `munmap()` unmaps the shared memory.
- `shm_unlink()` removes the shared memory object.

### POSIX Semaphores

Provides named and unnamed semaphores.

Key Concepts:
- Can be shared between processes or thread-local
- Simpler API compared to System V semaphores
- Named semaphores persist until explicitly unlinked

Example:
```c
#include <fcntl.h>
#include <sys/stat.h>
#include <semaphore.h>
#include <stdio.h>

#define SEM_NAME "/test_sem"

int main() {
    sem_t *sem = sem_open(SEM_NAME, O_CREAT, 0644, 1);

    sem_wait(sem);
    printf("Critical section\n");
    sem_post(sem);

    sem_close(sem);
    sem_unlink(SEM_NAME);

    return 0;
}
```

Explanation:
- `sem_open()` creates or opens a named semaphore.
- `sem_wait()` decrements the semaphore (P operation).
- `sem_post()` increments the semaphore (V operation).
- `sem_close()` closes the semaphore.
- `sem_unlink()` removes the named semaphore.

## Practice Exercises

1. Implement a simple chat program using named pipes, allowing two processes to exchange messages.

2. Create a producer-consumer system using System V message queues, where multiple producers send data to multiple consumers.

3. Develop a shared memory-based circular buffer for high-speed data transfer between processes, using semaphores for synchronization.

4. Write a program that uses POSIX message queues to implement a simple task distribution system, where a manager process sends tasks to worker processes.

## Important Tools and Utilities

- ipcs: View information about IPC resources
- ipcrm: Remove IPC resources
- ftok: Generate IPC keys
- lsof: List open files and IPC resources

## Best Practices and Considerations

1. Always clean up IPC resources to prevent resource leaks.
2. Use appropriate permissions for IPC objects to maintain security.
3. Be aware of the persistence of different IPC mechanisms and their cleanup requirements.
4. Consider using POSIX IPC mechanisms for better portability across UNIX-like systems.
5. Implement proper error handling and resource cleanup in case of failures.

## Additional Resources

1. "The Linux Programming Interface" by Michael Kerrisk
2. POSIX IPC documentation: https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/mqueue.h.html
3. Linux man pages: man 7 svipc, man 7 mq_overview, man 7 shm_overview, man 7 sem_overview
4. "Unix Network Programming, Volume 2: Interprocess Communications" by W. Richard Stevens
