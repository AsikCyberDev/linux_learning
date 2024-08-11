# Day 7: Inter-Process Communication (IPC) and System V IPC

## 1. Introduction to IPC

### Overview of IPC Mechanisms

Key Concepts:
- Importance of IPC in system programming
- Different IPC mechanisms available in Linux
- Choosing the right IPC method for specific use cases

### Comparison of IPC Methods

- Pipes and FIFOs
- System V IPC (Message Queues, Semaphores, Shared Memory)
- POSIX IPC
- Sockets
- Signals (as a basic form of IPC)

## 2. Pipes and FIFOs

### Anonymous Pipes

Key Concepts:
- Creating and using pipes
- Parent-child communication using pipes

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

        ssize_t num_read = read(pipefd[0], buffer, BUFFER_SIZE);
        if (num_read > 0) {
            printf("Child received: %s\n", buffer);
        }

        close(pipefd[0]);
        exit(EXIT_SUCCESS);
    } else {  // Parent process
        close(pipefd[0]);  // Close unused read end

        const char *message = "Hello from parent!";
        write(pipefd[1], message, strlen(message) + 1);

        close(pipefd[1]);
        wait(NULL);  // Wait for child to finish
        exit(EXIT_SUCCESS);
    }

    return 0;
}
```

### Named Pipes (FIFOs)

Key Concepts:
- Creating and using FIFOs
- Communication between unrelated processes

Example: Creating and using a FIFO

```c
// writer.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <string.h>

#define FIFO_NAME "/tmp/myfifo"

int main() {
    mkfifo(FIFO_NAME, 0666);

    int fd = open(FIFO_NAME, O_WRONLY);
    if (fd == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }

    const char *message = "Hello through FIFO!";
    write(fd, message, strlen(message) + 1);

    close(fd);
    return 0;
}

// reader.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>

#define FIFO_NAME "/tmp/myfifo"
#define BUFFER_SIZE 256

int main() {
    char buffer[BUFFER_SIZE];

    int fd = open(FIFO_NAME, O_RDONLY);
    if (fd == -1) {
        perror("open");
        exit(EXIT_FAILURE);
    }

    ssize_t num_read = read(fd, buffer, BUFFER_SIZE);
    if (num_read > 0) {
        printf("Received: %s\n", buffer);
    }

    close(fd);
    unlink(FIFO_NAME);
    return 0;
}
```

## 3. System V IPC

### Message Queues

Key Concepts:
- Creating and accessing message queues
- Sending and receiving messages

Example: Using System V message queues

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/msg.h>

#define MAX_TEXT 512

struct msg_buffer {
    long msg_type;
    char msg_text[MAX_TEXT];
};

int main() {
    key_t key;
    int msgid;
    struct msg_buffer message;

    // Generate a key
    key = ftok("progfile", 65);

    // Create a message queue
    msgid = msgget(key, 0666 | IPC_CREAT);

    // Sending a message
    message.msg_type = 1;
    strcpy(message.msg_text, "Hello from message queue!");
    msgsnd(msgid, &message, sizeof(message), 0);

    // Receiving a message
    msgrcv(msgid, &message, sizeof(message), 1, 0);
    printf("Received: %s\n", message.msg_text);

    // Destroy the message queue
    msgctl(msgid, IPC_RMID, NULL);

    return 0;
}
```

### Semaphores

Key Concepts:
- Creating and initializing semaphores
- Using semaphores for synchronization

Example: Using System V semaphores

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/sem.h>

#define KEY 1234

union semun {
    int val;
    struct semid_ds *buf;
    unsigned short *array;
};

int main() {
    int semid;
    struct sembuf sb = {0, -1, 0};  // semaphore operation
    union semun arg;

    // Create the semaphore
    semid = semget(KEY, 1, 0666 | IPC_CREAT);

    // Initialize the semaphore
    arg.val = 1;
    if (semctl(semid, 0, SETVAL, arg) == -1) {
        perror("semctl");
        exit(1);
    }

    printf("Semaphore created and initialized\n");

    // Use the semaphore
    printf("Trying to acquire the semaphore...\n");
    if (semop(semid, &sb, 1) == -1) {
        perror("semop");
        exit(1);
    }

    printf("Semaphore acquired. Critical section starts.\n");
    sleep(5);  // Simulate some work
    printf("Critical section ends.\n");

    // Release the semaphore
    sb.sem_op = 1;  // Releasing the semaphore
    if (semop(semid, &sb, 1) == -1) {
        perror("semop");
        exit(1);
    }

    printf("Semaphore released\n");

    // Remove the semaphore
    if (semctl(semid, 0, IPC_RMID, arg) == -1) {
        perror("semctl");
        exit(1);
    }

    return 0;
}
```

### Shared Memory

Key Concepts:
- Creating and attaching shared memory segments
- Synchronizing access to shared memory

Example: Using System V shared memory

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/ipc.h>
#include <sys/shm.h>

#define SHM_SIZE 1024

int main() {
    key_t key;
    int shmid;
    char *data;

    // Generate a key
    key = ftok("shmfile", 65);

    // Create the shared memory segment
    shmid = shmget(key, SHM_SIZE, 0666 | IPC_CREAT);
    if (shmid == -1) {
        perror("shmget");
        exit(1);
    }

    // Attach the shared memory segment
    data = shmat(shmid, (void *)0, 0);
    if (data == (char *)(-1)) {
        perror("shmat");
        exit(1);
    }

    // Write to shared memory
    strcpy(data, "Hello, shared memory!");

    printf("Written to shared memory: %s\n", data);

    // Detach from shared memory
    shmdt(data);

    // Reattach to read
    data = shmat(shmid, (void *)0, 0);
    printf("Read from shared memory: %s\n", data);

    // Detach from shared memory
    shmdt(data);

    // Delete the shared memory segment
    shmctl(shmid, IPC_RMID, NULL);

    return 0;
}
```

## 4. POSIX IPC (Overview)

- POSIX Message Queues
- POSIX Semaphores
- POSIX Shared Memory

## 5. Comparison of System V IPC and POSIX IPC

- Advantages and disadvantages of each
- When to use System V IPC vs POSIX IPC

## 6. Best Practices and Considerations

- Proper cleanup and resource management
- Avoiding deadlocks and race conditions
- Security considerations in IPC

## Practice Exercises

1. Implement a simple producer-consumer problem using pipes.

2. Create a chat application using System V message queues.

3. Develop a multi-process application that uses semaphores for synchronization and shared memory for data sharing.

4. Implement a simple distributed computing system using a combination of IPC mechanisms.

## Important Tools and Utilities

- ipcs: Provide information on IPC facilities
- ipcrm: Remove IPC facilities
- strace: Trace system calls related to IPC

## Additional Resources

1. "The Linux Programming Interface" by Michael Kerrisk (chapters on IPC)
2. "Advanced Programming in the UNIX Environment" by W. Richard Stevens and Stephen A. Rago
3. Linux man pages: man 7 svipc, man 7 mq_overview, man 7 sem_overview, man 7 shm_overview
4. "Unix Network Programming, Volume 2: Interprocess Communications" by W. Richard Stevens
