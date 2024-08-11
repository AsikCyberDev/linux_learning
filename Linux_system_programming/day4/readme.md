# Day 4: Multithreading and Synchronization

## 1. Introduction to POSIX Threads (pthreads)

Threads allow concurrent execution within a single process, sharing the same memory space.

### Thread Creation and Termination

Key Concepts:
- Creating threads with pthread_create()
- Terminating threads with pthread_exit()
- Joining threads with pthread_join()

Example: Creating and joining threads

```c
#include <stdio.h>
#include <pthread.h>

void *thread_function(void *arg) {
    int thread_id = *(int*)arg;
    printf("Thread %d is running\n", thread_id);
    return NULL;
}

int main() {
    pthread_t threads[5];
    int thread_args[5];

    for (int i = 0; i < 5; i++) {
        thread_args[i] = i;
        if (pthread_create(&threads[i], NULL, thread_function, &thread_args[i]) != 0) {
            perror("pthread_create");
            return 1;
        }
    }

    for (int i = 0; i < 5; i++) {
        if (pthread_join(threads[i], NULL) != 0) {
            perror("pthread_join");
            return 1;
        }
    }

    printf("All threads have completed\n");
    return 0;
}
```

Explanation:
- `pthread_create()` is used to create new threads.
- Each thread executes the `thread_function`.
- `pthread_join()` waits for each thread to complete.

### Thread Attributes

Customizing thread behavior using attributes.

Key Concepts:
- Initializing and destroying thread attributes
- Setting thread stack size
- Configuring thread detach state

Example: Setting thread stack size

```c
#include <stdio.h>
#include <pthread.h>

void *thread_function(void *arg) {
    printf("Thread is running\n");
    return NULL;
}

int main() {
    pthread_t thread;
    pthread_attr_t attr;
    size_t stack_size = 2 * 1024 * 1024; // 2 MB

    pthread_attr_init(&attr);
    pthread_attr_setstacksize(&attr, stack_size);

    if (pthread_create(&thread, &attr, thread_function, NULL) != 0) {
        perror("pthread_create");
        return 1;
    }

    pthread_attr_destroy(&attr);
    pthread_join(thread, NULL);

    return 0;
}
```

Explanation:
- `pthread_attr_init()` initializes the attribute object.
- `pthread_attr_setstacksize()` sets the thread's stack size.
- The attribute is used in `pthread_create()` to create the thread with the specified attributes.

## 2. Thread Synchronization

### Mutexes (Mutual Exclusion)

Mutexes provide exclusive access to shared resources.

Key Concepts:
- Initializing and destroying mutexes
- Locking and unlocking mutexes
- Handling deadlocks

Example: Using a mutex to protect a shared counter

```c
#include <stdio.h>
#include <pthread.h>

#define NUM_THREADS 5
#define COUNT_PER_THREAD 1000000

long long counter = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void *increment_counter(void *arg) {
    for (int i = 0; i < COUNT_PER_THREAD; i++) {
        pthread_mutex_lock(&mutex);
        counter++;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

int main() {
    pthread_t threads[NUM_THREADS];

    for (int i = 0; i < NUM_THREADS; i++) {
        if (pthread_create(&threads[i], NULL, increment_counter, NULL) != 0) {
            perror("pthread_create");
            return 1;
        }
    }

    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }

    printf("Final counter value: %lld\n", counter);
    return 0;
}
```

Explanation:
- A mutex is used to protect access to the shared `counter` variable.
- `pthread_mutex_lock()` and `pthread_mutex_unlock()` ensure exclusive access to the critical section.

### Condition Variables

Condition variables allow threads to synchronize based on the value of data.

Key Concepts:
- Initializing and destroying condition variables
- Waiting on conditions with pthread_cond_wait()
- Signaling conditions with pthread_cond_signal() and pthread_cond_broadcast()

Example: Producer-Consumer problem using condition variables

```c
#include <stdio.h>
#include <pthread.h>
#include <stdlib.h>

#define BUFFER_SIZE 5

int buffer[BUFFER_SIZE];
int count = 0;
int in = 0;
int out = 0;

pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t full = PTHREAD_COND_INITIALIZER;
pthread_cond_t empty = PTHREAD_COND_INITIALIZER;

void *producer(void *arg) {
    int item;
    for (int i = 0; i < 10; i++) {
        item = rand() % 100;
        pthread_mutex_lock(&mutex);
        while (count == BUFFER_SIZE) {
            pthread_cond_wait(&empty, &mutex);
        }
        buffer[in] = item;
        in = (in + 1) % BUFFER_SIZE;
        count++;
        printf("Produced: %d\n", item);
        pthread_cond_signal(&full);
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

void *consumer(void *arg) {
    int item;
    for (int i = 0; i < 10; i++) {
        pthread_mutex_lock(&mutex);
        while (count == 0) {
            pthread_cond_wait(&full, &mutex);
        }
        item = buffer[out];
        out = (out + 1) % BUFFER_SIZE;
        count--;
        printf("Consumed: %d\n", item);
        pthread_cond_signal(&empty);
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

int main() {
    pthread_t prod, cons;
    pthread_create(&prod, NULL, producer, NULL);
    pthread_create(&cons, NULL, consumer, NULL);
    pthread_join(prod, NULL);
    pthread_join(cons, NULL);
    return 0;
}
```

Explanation:
- Mutex protects access to the shared buffer.
- `full` and `empty` condition variables synchronize producer and consumer.
- `pthread_cond_wait()` releases the mutex and waits for the condition.
- `pthread_cond_signal()` wakes up a waiting thread.

### Read-Write Locks

Read-write locks allow multiple readers or a single writer.

Key Concepts:
- Initializing and destroying read-write locks
- Acquiring read and write locks
- Upgrading and downgrading locks

Example: Using read-write locks for a shared data structure

```c
#include <stdio.h>
#include <pthread.h>
#include <unistd.h>

pthread_rwlock_t rwlock = PTHREAD_RWLOCK_INITIALIZER;
int shared_data = 0;

void *reader(void *arg) {
    int id = *(int*)arg;
    for (int i = 0; i < 5; i++) {
        pthread_rwlock_rdlock(&rwlock);
        printf("Reader %d read value: %d\n", id, shared_data);
        pthread_rwlock_unlock(&rwlock);
        sleep(1);
    }
    return NULL;
}

void *writer(void *arg) {
    int id = *(int*)arg;
    for (int i = 0; i < 3; i++) {
        pthread_rwlock_wrlock(&rwlock);
        shared_data++;
        printf("Writer %d updated value to: %d\n", id, shared_data);
        pthread_rwlock_unlock(&rwlock);
        sleep(2);
    }
    return NULL;
}

int main() {
    pthread_t readers[3], writers[2];
    int reader_ids[3] = {1, 2, 3};
    int writer_ids[2] = {1, 2};

    for (int i = 0; i < 3; i++) {
        pthread_create(&readers[i], NULL, reader, &reader_ids[i]);
    }
    for (int i = 0; i < 2; i++) {
        pthread_create(&writers[i], NULL, writer, &writer_ids[i]);
    }

    for (int i = 0; i < 3; i++) {
        pthread_join(readers[i], NULL);
    }
    for (int i = 0; i < 2; i++) {
        pthread_join(writers[i], NULL);
    }

    return 0;
}
```

Explanation:
- `pthread_rwlock_rdlock()` acquires a read lock, allowing multiple readers.
- `pthread_rwlock_wrlock()` acquires a write lock, excluding all other threads.
- Readers can access the data concurrently, while writers have exclusive access.

## 3. Thread-Safe Programming

### Reentrant Functions

Writing functions that can be safely called by multiple threads.

Key Concepts:
- Avoiding global and static variables
- Using thread-local storage
- Identifying and using reentrant library functions

Example: Making a function thread-safe using thread-local storage

```c
#include <stdio.h>
#include <pthread.h>
#include <string.h>

__thread char buffer[256];  // Thread-local storage

char* thread_safe_strtok(char* str, const char* delim) {
    return strtok_r(str, delim, &buffer);
}

void* tokenize(void* arg) {
    char* input = (char*)arg;
    char* token;

    token = thread_safe_strtok(input, " ");
    while (token != NULL) {
        printf("Thread %lu: %s\n", pthread_self(), token);
        token = thread_safe_strtok(NULL, " ");
    }
    return NULL;
}

int main() {
    pthread_t thread1, thread2;
    char input1[] = "Hello world from thread one";
    char input2[] = "Greetings universe from thread two";

    pthread_create(&thread1, NULL, tokenize, input1);
    pthread_create(&thread2, NULL, tokenize, input2);

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    return 0;
}
```

Explanation:
- `__thread` keyword creates thread-local storage for `buffer`.
- `strtok_r()` is used instead of `strtok()` for thread-safety.
- Each thread has its own context for tokenization.

### Thread-Safe Data Structures

Implementing data structures that can be safely used by multiple threads.

Key Concepts:
- Fine-grained locking
- Lock-free algorithms
- Using atomic operations

Example: A simple thread-safe queue

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

typedef struct Node {
    int data;
    struct Node* next;
} Node;

typedef struct {
    Node* head;
    Node* tail;
    pthread_mutex_t mutex;
} Queue;

void queue_init(Queue* q) {
    q->head = q->tail = NULL;
    pthread_mutex_init(&q->mutex, NULL);
}

void queue_enqueue(Queue* q, int value) {
    Node* new_node = malloc(sizeof(Node));
    new_node->data = value;
    new_node->next = NULL;

    pthread_mutex_lock(&q->mutex);
    if (q->tail == NULL) {
        q->head = q->tail = new_node;
    } else {
        q->tail->next = new_node;
        q->tail = new_node;
    }
    pthread_mutex_unlock(&q->mutex);
}

int queue_dequeue(Queue* q, int* value) {
    pthread_mutex_lock(&q->mutex);
    if (q->head == NULL) {
        pthread_mutex_unlock(&q->mutex);
        return 0;
    }

    Node* temp = q->head;
    *value = temp->data;
    q->head = q->head->next;
    if (q->head == NULL) {
        q->tail = NULL;
    }
    pthread_mutex_unlock(&q->mutex);

    free(temp);
    return 1;
}

// Usage example in main() and thread functions...
```

Explanation:
- The queue is protected by a mutex to ensure thread-safety.
- Enqueue and dequeue operations are atomic from the perspective of other threads.

## Practice Exercises

1. Implement a thread pool that can execute a given number of tasks concurrently.

2. Create a multi-threaded web server that can handle multiple client connections simultaneously.

3. Develop a parallel merge sort algorithm using pthreads.

4. Implement a thread-safe hash table with fine-grained locking.

## Important Tools and Utilities

- valgrind --tool=helgrind: Detect synchronization errors in multithreaded programs
- gdb: Debug multithreaded programs
- pstack: Display the call stack of all threads in a process
- htop: View and manage processes and threads in real-time

## Best Practices and Considerations

1. Minimize the use of global variables in multithreaded programs.
2. Keep critical sections as short as possible to improve concurrency.
3. Be aware of potential deadlocks and use techniques like lock ordering to prevent them.
4. Use higher-level synchronization primitives (e.g., from C++11 or boost) for more complex scenarios.
5. Profile your multithreaded programs to identify performance bottlenecks.

## Additional Resources

1. "Programming with POSIX Threads" by David R. Butenhof
2. "C++ Concurrency in Action" by Anthony Williams
3. POSIX Threads Programming: https://computing.llnl.gov/tutorials/pthreads/
4. Linux man pages: man 7 pthreads, man 3 pthread_create, man 3 pthread_mutex_init
