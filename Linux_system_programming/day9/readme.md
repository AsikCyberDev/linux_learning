# Day 9: Threads and Synchronization

## 1. Introduction to Threads

### Overview of Threads

Key Concepts:
- Processes vs. Threads
- Benefits and challenges of multithreading
- POSIX threads (pthreads) library

### Creating and Managing Threads

Key Concepts:
- Creating threads with pthread_create()
- Terminating threads with pthread_exit()
- Joining threads with pthread_join()

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
- Protecting shared resources with mutexes
- Initializing and destroying mutexes
- Locking and unlocking mutexes

Example: Using mutexes to protect a shared counter

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

#define NUM_THREADS 5
#define COUNT_LIMIT 1000000

long counter = 0;
pthread_mutex_t counter_mutex = PTHREAD_MUTEX_INITIALIZER;

void *count_function(void *arg) {
    for (int i = 0; i < COUNT_LIMIT; i++) {
        pthread_mutex_lock(&counter_mutex);
        counter++;
        pthread_mutex_unlock(&counter_mutex);
    }
    return NULL;
}

int main() {
    pthread_t threads[NUM_THREADS];

    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_create(&threads[i], NULL, count_function, NULL);
    }

    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
    }

    printf("Final counter value: %ld\n", counter);
    return 0;
}
```

### Condition Variables

Key Concepts:
- Waiting for conditions with pthread_cond_wait()
- Signaling conditions with pthread_cond_signal() and pthread_cond_broadcast()

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

## 3. Advanced Thread Concepts

### Thread-Specific Data

Key Concepts:
- Creating and using thread-specific data
- Key creation and destruction

Example: Using thread-specific data

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>

pthread_key_t thread_log_key;

void *thread_function(void *arg) {
    char *thread_log = malloc(100);
    pthread_setspecific(thread_log_key, thread_log);

    sprintf(thread_log, "Log for thread %ld", (long)pthread_self());
    printf("%s\n", (char *)pthread_getspecific(thread_log_key));

    return NULL;
}

void cleanup_thread_log(void *log) {
    free(log);
}

int main() {
    pthread_t thread1, thread2;

    pthread_key_create(&thread_log_key, cleanup_thread_log);

    pthread_create(&thread1, NULL, thread_function, NULL);
    pthread_create(&thread2, NULL, thread_function, NULL);

    pthread_join(thread1, NULL);
    pthread_join(thread2, NULL);

    pthread_key_delete(thread_log_key);

    return 0;
}
```

### Thread Cancellation

Key Concepts:
- Cancelling threads
- Setting cancellation states and types
- Cleanup handlers

Example: Thread cancellation and cleanup

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

void cleanup_handler(void *arg) {
    printf("Cleanup: freeing resources\n");
    free(arg);
}

void *thread_function(void *arg) {
    char *resource = malloc(100);
    pthread_cleanup_push(cleanup_handler, resource);

    while (1) {
        printf("Thread is running...\n");
        sleep(1);
        pthread_testcancel();
    }

    pthread_cleanup_pop(1);
    return NULL;
}

int main() {
    pthread_t thread;
    pthread_create(&thread, NULL, thread_function, NULL);

    sleep(3);  // Let the thread run for a while
    pthread_cancel(thread);

    pthread_join(thread, NULL);
    printf("Thread has been cancelled\n");

    return 0;
}
```

## 4. Thread Pools

Key Concepts:
- Implementing a basic thread pool
- Task queue management
- Worker thread implementation

Example: Simple thread pool implementation

```c
#include <stdio.h>
#include <stdlib.h>
#include <pthread.h>
#include <unistd.h>

#define NUM_THREADS 3
#define QUEUE_SIZE 10

typedef struct {
    void (*function)(void *);
    void *argument;
} Task;

Task task_queue[QUEUE_SIZE];
int task_count = 0;

pthread_mutex_t queue_mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t queue_not_empty = PTHREAD_COND_INITIALIZER;
pthread_cond_t queue_not_full = PTHREAD_COND_INITIALIZER;

void *worker(void *arg) {
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
    while (task_count == QUEUE_SIZE) {
        pthread_cond_wait(&queue_not_full, &queue_mutex);
    }
    Task task = {function, argument};
    task_queue[task_count] = task;
    task_count++;
    pthread_cond_signal(&queue_not_empty);
    pthread_mutex_unlock(&queue_mutex);
}

void task_function(void *arg) {
    int num = *(int *)arg;
    printf("Executing task %d\n", num);
    sleep(1);
}

int main() {
    pthread_t threads[NUM_THREADS];

    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_create(&threads[i], NULL, worker, NULL);
    }

    for (int i = 0; i < 20; i++) {
        int *num = malloc(sizeof(int));
        *num = i;
        add_task(task_function, num);
    }

    sleep(22);  // Allow time for all tasks to complete

    return 0;
}
```

## 5. Best Practices and Considerations

- Avoiding race conditions and deadlocks
- Scalability considerations
- Debugging multithreaded programs

## Practice Exercises

1. Implement a multithreaded version of the merge sort algorithm.

2. Create a thread-safe queue data structure.

3. Develop a simple multithreaded web server.

4. Implement the readers-writers problem using pthreads.

## Important Tools and Utilities

- gdb: Debugging multithreaded programs
- valgrind: Detecting memory leaks and race conditions
- helgrind: Thread error detector

## Additional Resources

1. "POSIX Threads Programming" tutorial by Blaise Barney (Lawrence Livermore National Laboratory)
2. "Programming with POSIX Threads" by David R. Butenhof
3. "The Art of Multiprocessor Programming" by Maurice Herlihy and Nir Shavit
4. Linux man pages: man 7 pthreads, man 3 pthread_create, man 3 pthread_mutex_init
