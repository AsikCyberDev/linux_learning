
# Day 14: Multithreading and Synchronization

## 1. Introduction to Multithreading

### Concepts of Multithreading

Key Concepts:
- Processes vs. Threads
- Benefits and challenges of multithreading
- POSIX threads (pthreads) library

### Creating and Managing Threads

Key Concepts:
- pthread_create()
- pthread_join()
- pthread_exit()

Example: Basic thread creation and joining

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

void *thread_function(void *arg) {
    int *id = (int *)arg;
    printf("Thread %d is running\n", *id);
    return NULL;
}

int main() {
    pthread_t thread1, thread2;
    int id1 = 1, id2 = 2;

    pthread_create(&thread1, NULL, thread_function, &id1);
    pthread_create(&thread2, NULL, thread_function, &id2);

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    printf("Both threads have completed\n");

    return 0;
}
```

## 2. Thread Synchronization

### Mutexes

Key Concepts:
- Mutual exclusion
- pthread_mutex_init(), pthread_mutex_lock(), pthread_mutex_unlock()

Example: Using a mutex to protect shared data

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define NUM_THREADS 5
#define NUM_INCREMENTS 1000000

long shared_counter = 0;
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

void *increment_counter(void *arg) {
    for (int i = 0; i < NUM_INCREMENTS; i++) {
        pthread_mutex_lock(&mutex);
        shared_counter++;
        pthread_mutex_unlock(&mutex);
    }
    return NULL;
}

int main() {
    pthread_t threads[NUM_THREADS];

    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_create(&threads[i], NULL, increment_counter, NULL);
    }

    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }

    printf("Final counter value: %ld\n", shared_counter);

    return 0;
}
```

### Condition Variables

Key Concepts:
- Waiting for a condition
- pthread_cond_wait(), pthread_cond_signal()

Example: Producer-Consumer problem using condition variables

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

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

### Read-Write Locks

Key Concepts:
- Multiple readers, single writer
- pthread_rwlock_init(), pthread_rwlock_rdlock(), pthread_rwlock_wrlock()

Example: Using read-write locks

```c
#include <stdio.h>
#include <stdlib.h>
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
        printf("Writer %d wrote value: %d\n", id, shared_data);
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

## 3. Advanced Threading Concepts

### Thread-Local Storage

Key Concepts:
- Per-thread data
- __thread keyword and pthread_key_t

Example: Using thread-local storage

```c
#include <stdio.h>
#include <pthread.h>

__thread int thread_local_var = 0;

void *thread_function(void *arg) {
    int id = *(int*)arg;
    thread_local_var = id;
    printf("Thread %d: thread_local_var = %d\n", id, thread_local_var);
    return NULL;
}

int main() {
    pthread_t thread1, thread2;
    int id1 = 1, id2 = 2;

    pthread_create(&thread1, NULL, thread_function, &id1);
    pthread_create(&thread2, NULL, thread_function, &id2);

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    return 0;
}
```

### Thread Pools

Key Concepts:
- Reusing threads for multiple tasks
- Implementing a basic thread pool

Example: Simple thread pool implementation

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define THREAD_POOL_SIZE 3
#define MAX_QUEUE_SIZE 10

typedef struct {
    void (*function)(void *);
    void *argument;
} Task;

Task task_queue[MAX_QUEUE_SIZE];
int task_count = 0;

pthread_mutex_t queue_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t queue_not_empty = PTHREAD_COND_INITIALIZER;
pthread_cond_t queue_not_full = PTHREAD_COND_INITIALIZER;

void *thread_function(void *arg) {
    while (1) {
        Task task;
        pthread_mutex_lock(&queue_mutex);
        while (task_count == 0) {
            pthread_cond_wait(&queue_not_empty, &queue_mutex);
        }
        task = task_queue[0];
        for (int i = 0; i < task_count - 1; i++) {
            task_queue[i] = task_queue[i + 1];
        }
        task_count--;
        pthread_cond_signal(&queue_not_full);
        pthread_mutex_unlock(&queue_mutex);

        (*(task.function))(task.argument);
    }
    return NULL;
}

void add_task(void (*function)(void *), void *argument) {
    pthread_mutex_lock(&queue_mutex);
    while (task_count == MAX_QUEUE_SIZE) {
        pthread_cond_wait(&queue_not_full, &queue_mutex);
    }
    Task task = {function, argument};
    task_queue[task_count] = task;
    task_count++;
    pthread_cond_signal(&queue_not_empty);
    pthread_mutex_unlock(&queue_mutex);
}

void task(void *arg) {
    int id = *(int*)arg;
    printf("Task %d is running\n", id);
    sleep(1);
}

int main() {
    pthread_t threads[THREAD_POOL_SIZE];

    for (int i = 0; i < THREAD_POOL_SIZE; i++) {
        pthread_create(&threads[i], NULL, thread_function, NULL);
    }

    for (int i = 0; i < 10; i++) {
        int *arg = malloc(sizeof(int));
        *arg = i;
        add_task(task, arg);
    }

    sleep(15);  // Allow time for all tasks to complete

    return 0;
}
```

## Practice Exercises

1. Implement a multithreaded merge sort algorithm.

2. Create a thread-safe queue data structure.

3. Develop a simple web server that uses a thread pool to handle client connections.

4. Implement the dining philosophers problem using pthreads and mutexes.

## Important Tools and Utilities

- gdb: Debugging multithreaded programs
- valgrind: Detecting threading errors and race conditions
- helgrind: Thread error detector

## Additional Resources

1. "Programming with POSIX Threads" by David R. Butenhof
2. "C++ Concurrency in Action" by Anthony Williams
3. "The Art of Multiprocessor Programming" by Maurice Herlihy and Nir Shavit
4. POSIX Threads Programming: https://computing.llnl.gov/tutorials/pthreads/
