
# Day 4: Synchronization and IPC

## 1. Synchronization Primitives: Mutexes, Semaphores, and Spinlocks

### Introduction
Synchronization primitives are essential tools for managing concurrent access to shared resources in multi-threaded and multi-process environments. Linux provides various synchronization mechanisms, each with its own use cases and trade-offs.

### Key Concepts

1. **Mutex (Mutual Exclusion)**
   - Provides exclusive access to a shared resource
   - Only one thread can hold a mutex at a time

2. **Semaphore**
   - Allows a fixed number of threads to access a resource
   - Can be used for signaling between threads

3. **Spinlock**
   - Busy-waiting synchronization mechanism
   - Efficient for short-term locking in kernel space

4. **Read-Write Lock**
   - Allows multiple readers or one writer
   - Useful for read-heavy workloads

5. **Atomic Operations**
   - Indivisible operations that don't require explicit locking

### Practical Examples

#### Mutex Example

```c
#include <stdio.h>
#include <pthread.h>

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
int shared_variable = 0;

void* increment(void* arg) {
    for (int i = 0; i < 1000000; i++) {
        pthread_mutex_lock(&mutex);
        shared_variable++;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

int main() {
    pthread_t thread1, thread2;
    pthread_create(&thread1, NULL, increment, NULL);
    pthread_create(&thread2, NULL, increment, NULL);
    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);
    printf("Final value: %d\n", shared_variable);
    return 0;
}
```

#### Semaphore Example

```c
#include <stdio.h>
#include <pthread.h>
#include <semaphore.h>

#define BUFFER_SIZE 5
#define NUM_ITEMS 20

sem_t empty, full;
pthread_mutex_t mutex;
int buffer[BUFFER_SIZE];
int in = 0, out = 0;

void* producer(void* arg) {
    for (int i = 0; i < NUM_ITEMS; i++) {
        sem_wait(&empty);
        pthread_mutex_lock(&mutex);
        buffer[in] = i;
        in = (in + 1) % BUFFER_SIZE;
        pthread_mutex_unlock(&mutex);
        sem_post(&full);
    }
    return NULL;
}

void* consumer(void* arg) {
    for (int i = 0; i < NUM_ITEMS; i++) {
        sem_wait(&full);
        pthread_mutex_lock(&mutex);
        int item = buffer[out];
        out = (out + 1) % BUFFER_SIZE;
        pthread_mutex_unlock(&mutex);
        sem_post(&empty);
        printf("Consumed: %d\n", item);
    }
    return NULL;
}

int main() {
    pthread_t prod, cons;
    sem_init(&empty, 0, BUFFER_SIZE);
    sem_init(&full, 0, 0);
    pthread_mutex_init(&mutex, NULL);
    pthread_create(&prod, NULL, producer, NULL);
    pthread_create(&cons, NULL, consumer, NULL);
    pthread_join(prod, NULL);
    pthread_join(cons, NULL);
    sem_destroy(&empty);
    sem_destroy(&full);
    pthread_mutex_destroy(&mutex);
    return 0;
}
```

### Gotchas and Best Practices

1. **Deadlocks**: Always acquire locks in the same order to prevent deadlocks.
2. **Priority Inversion**: Be aware of priority inversion issues, especially in real-time systems.
3. **Granularity**: Choose the right granularity for locks to balance concurrency and overhead.
4. **Spinning vs. Sleeping**: Use spinlocks for short-term locking, mutexes for potentially longer waits.
5. **Reader-Writer Fairness**: Be cautious of writer starvation in read-heavy workloads.

### Interview Questions

1. Q: What's the difference between a mutex and a semaphore?
   A: A mutex is used for exclusive access and can only be locked once, while a semaphore can allow a specified number of threads to access a resource simultaneously.

2. Q: When would you use a spinlock instead of a mutex?
   A: Spinlocks are typically used in scenarios where the expected wait time is very short, especially in kernel space where sleeping is not desirable.

3. Q: What is the "thundering herd" problem and how can it be mitigated?
   A: The thundering herd problem occurs when many threads are awakened to compete for a resource, but only one can proceed. It can be mitigated by using mechanisms like Linux's futex (fast userspace mutex) which wake only one or a few waiters at a time.

4. Q: Explain the concept of a read-copy-update (RCU) and its advantages.
   A: RCU is a synchronization mechanism that allows extremely low overhead reading of shared data, even in the presence of concurrent updates. It's particularly useful for read-heavy workloads in the Linux kernel.

5. Q: How do atomic operations differ from using locks, and when would you use them?
   A: Atomic operations provide lightweight, lock-free synchronization for simple operations. They're useful for things like reference counting or simple flags where the overhead of a full lock would be excessive.

## 2. Inter-Process Communication (IPC): Pipes, Shared Memory, and Message Queues

### Introduction
Inter-Process Communication (IPC) mechanisms allow processes to exchange data and synchronize their actions. Linux provides various IPC methods, each suited to different communication patterns and requirements.

### Key Concepts

1. **Pipes**
   - Unidirectional data channel
   - Used for communication between related processes

2. **Named Pipes (FIFOs)**
   - Similar to pipes, but with a name in the filesystem
   - Can be used between unrelated processes

3. **Shared Memory**
   - Fastest IPC method
   - Allows multiple processes to access the same memory region

4. **Message Queues**
   - Allow processes to exchange messages
   - Provide more structure than pipes

5. **Sockets**
   - Can be used for IPC on the same machine or over a network
   - Versatile, but with more overhead than other local IPC methods

### Practical Examples

#### Pipe Example

```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>

int main() {
    int pipefd[2];
    char buf[30];
    pid_t pid;

    if (pipe(pipefd) == -1) {
        perror("pipe");
        return 1;
    }

    pid = fork();
    if (pid == -1) {
        perror("fork");
        return 1;
    }

    if (pid == 0) {  // Child process
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

#### Shared Memory Example

```c
#include <stdio.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <sys/wait.h>
#include <unistd.h>
#include <string.h>

int main() {
    const char *message = "Hello, shared memory!";
    void *shared_memory = mmap(NULL, 4096, PROT_READ | PROT_WRITE,
                               MAP_SHARED | MAP_ANONYMOUS, -1, 0);
    if (shared_memory == MAP_FAILED) {
        perror("mmap");
        exit(1);
    }

    pid_t pid = fork();
    if (pid == -1) {
        perror("fork");
        exit(1);
    }

    if (pid == 0) {  // Child process
        sleep(1);  // Ensure parent writes first
        printf("Child read: %s\n", (char *)shared_memory);
    } else {  // Parent process
        strcpy(shared_memory, message);
        wait(NULL);  // Wait for child to finish
    }

    munmap(shared_memory, 4096);
    return 0;
}
```

### Gotchas and Best Practices

1. **Deadlocks**: Be cautious of deadlocks when using multiple IPC mechanisms.
2. **Resource Cleanup**: Always clean up IPC resources (close pipes, unlink shared memory, etc.).
3. **Security**: Be aware of security implications, especially with shared memory and message queues.
4. **Buffering**: Understand the buffering behavior of different IPC mechanisms.
5. **Synchronization**: Use appropriate synchronization methods when accessing shared resources.

### Interview Questions

1. Q: What are the advantages and disadvantages of using shared memory for IPC?
   A: Advantages include very fast data transfer and easy sharing of complex data structures. Disadvantages include the need for explicit synchronization and potential for data corruption if not properly managed.

2. Q: How do named pipes differ from anonymous pipes?
   A: Named pipes have a presence in the filesystem and can be used between unrelated processes, while anonymous pipes are typically used between parent and child processes.

3. Q: In what scenarios would you choose message queues over pipes?
   A: Message queues are preferable when you need to send structured messages, support multiple readers or writers, or require message prioritization.

4. Q: How can you ensure proper synchronization when using shared memory?
   A: Synchronization for shared memory can be achieved using mechanisms like semaphores, mutexes, or atomic operations, depending on the specific requirements of the application.

5. Q: What is the difference between System V IPC and POSIX IPC?
   A: System V IPC is older and uses integer identifiers, while POSIX IPC uses file descriptors and provides a more consistent API. POSIX IPC is generally considered more modern and portable.

## 3. Signals: Handling and Masking

### Introduction
Signals are software interrupts used in Unix-like operating systems for inter-process communication and handling asynchronous events. Understanding signal handling is crucial for developing robust Linux applications.

### Key Concepts

1. **Signal Types**
   - SIGINT, SIGTERM, SIGKILL, SIGSEGV, etc.
   - Each signal has a default action (terminate, ignore, or stop)

2. **Signal Handlers**
   - Custom functions to handle specific signals
   - Registered using `signal()` or `sigaction()`

3. **Signal Masking**
   - Blocking signals temporarily
   - Useful for protecting critical sections

4. **Real-time Signals**
   - Additional signals with more flexibility
   - Guaranteed delivery order

5. **Async-Signal-Safe Functions**
   - Functions that can be safely called from signal handlers

### Practical Example: Signal Handling

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

volatile sig_atomic_t keep_running = 1;

void sigint_handler(int sig) {
    keep_running = 0;
}

int main() {
    struct sigaction sa;
    sa.sa_handler = sigint_handler;
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;

    if (sigaction(SIGINT, &sa, NULL) == -1) {
        perror("sigaction");
        return 1;
    }

    printf("Running... Press Ctrl-C to stop.\n");
    while (keep_running) {
        printf("Working...\n");
        sleep(1);
    }

    printf("Stopped by SIGINT\n");
    return 0;
}
```

### Signal Masking Example

```c
#include <stdio.h>
#include <signal.h>
#include <unistd.h>

void sigint_handler(int sig) {
    printf("SIGINT received\n");
}

int main() {
    struct sigaction sa;
    sigset_t mask, oldmask;

    sa.sa_handler = sigint_handler;
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;
    sigaction(SIGINT, &sa, NULL);

    sigemptyset(&mask);
    sigaddset(&mask, SIGINT);

    printf("Blocking SIGINT for 5 seconds...\n");
    sigprocmask(SIG_BLOCK, &mask, &oldmask);

    sleep(5);  // SIGINT will be blocked during this time

    printf("Unblocking SIGINT...\n");
    sigprocmask(SIG_SETMASK, &oldmask, NULL);

    printf("SIGINT unblocked. Sleeping for 5 more seconds...\n");
    sleep(5);  // SIGINT can be received now

    return 0;
}
```

### Gotchas and Best Practices

1. **Reentrancy**: Use only async-signal-safe functions in signal handlers.
2. **Global Variables**: Use `volatile sig_atomic_t` for variables accessed in signal handlers.
3. **Errno Handling**: Save and restore `errno` in signal handlers if necessary.
4. **Signal Disposition**: Be aware of signal disposition inheritance across `fork()` and `exec()`.
5. **SIGKILL and SIGSTOP**: These signals cannot be caught, blocked, or ignored.

### Interview Questions

1. Q: What is the purpose of `sigaction()` and how does it differ from `signal()`?
   A: `sigaction()` provides more control over signal handling compared to `signal()`. It allows specifying flags for signal behavior and is more portable across Unix-like systems.

2. Q: Why is it important to use `volatile sig_atomic_t` for variables modified in signal handlers?
   A: `volatile sig_atomic_t` ensures that the variable can be accessed atomically and prevents compiler optimizations that might interfere with signal handling.

3. Q: What are some examples of async-signal-safe functions, and why is it important to use only these in signal handlers?
   A: Examples include `write()`, `_exit()`, and `signal()`. It's important because non-async-signal-safe functions may use non-reentrant data structures, potentially causing corruption if interrupted by a signal.

4. Q: How can you temporarily block signals, and why might you want to do this?
   A: Signals can be blocked using `sigprocmask()`. This is useful for protecting critical sections of code from being interrupted by signal handlers.

5. Q: Explain the concept of "signal disposition" and how it's affected by `fork()` and `exec()`.
   A: Signal disposition refers to how a process responds to a signal. After `fork()`, the child inherits the parent's signal dispositions. After `exec()`, handled signals revert to their default disposition, but ignored signals remain ignored.
```

This content covers the three main topics for Day 4: Synchronization Primitives, Inter-Process Communication (IPC), and Signals. It provides an in-depth look at each topic, including practical examples, key concepts, and interview questions.