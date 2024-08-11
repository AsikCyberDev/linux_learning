# Day 8: Signals and Real-Time Programming

## 1. Introduction to Signals

### Overview of Signals

Key Concepts:
- What are signals?
- Standard signals vs. real-time signals
- Signal numbers and their meanings

### Signal Handling

Key Concepts:
- Default actions for signals
- Ignoring signals
- Catching and handling signals

Example: Basic signal handling

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

void signal_handler(int signum) {
    printf("Caught signal %d\n", signum);
}

int main() {
    signal(SIGINT, signal_handler);

    printf("Process will sleep for 10 seconds. Try pressing Ctrl+C...\n");
    sleep(10);

    printf("Sleep completed.\n");
    return 0;
}
```

## 2. Advanced Signal Handling

### Sigaction

Key Concepts:
- Using sigaction() for more control over signal handling
- Signal masks and blocking signals

Example: Using sigaction()

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <string.h>

void sigaction_handler(int signum, siginfo_t *info, void *context) {
    printf("Caught signal %d\n", signum);
    if (signum == SIGINT) {
        printf("SIGINT received. Sender's PID: %d\n", info->si_pid);
    }
}

int main() {
    struct sigaction sa;
    memset(&sa, 0, sizeof(sa));
    sa.sa_sigaction = sigaction_handler;
    sa.sa_flags = SA_SIGINFO;

    if (sigaction(SIGINT, &sa, NULL) == -1) {
        perror("sigaction");
        exit(1);
    }

    printf("Process will sleep for 10 seconds. Try pressing Ctrl+C...\n");
    sleep(10);

    printf("Sleep completed.\n");
    return 0;
}
```

### Signal Sets and Masks

Key Concepts:
- Creating and manipulating signal sets
- Blocking and unblocking signals

Example: Blocking signals

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

int main() {
    sigset_t set;

    sigemptyset(&set);
    sigaddset(&set, SIGINT);

    // Block SIGINT
    if (sigprocmask(SIG_BLOCK, &set, NULL) == -1) {
        perror("sigprocmask");
        exit(1);
    }

    printf("SIGINT is now blocked. Try pressing Ctrl+C...\n");
    sleep(5);

    // Unblock SIGINT
    if (sigprocmask(SIG_UNBLOCK, &set, NULL) == -1) {
        perror("sigprocmask");
        exit(1);
    }

    printf("SIGINT is now unblocked.\n");
    sleep(5);

    return 0;
}
```

## 3. Real-Time Signals

### Introduction to Real-Time Signals

Key Concepts:
- Differences between standard and real-time signals
- Queuing of real-time signals

### Using Real-Time Signals

Example: Sending and handling real-time signals

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <string.h>

#define SIGRTMIN_VALUE (__libc_current_sigrtmin())
#define SIGRTMAX_VALUE (__libc_current_sigrtmax())

void rt_sighandler(int signo, siginfo_t *info, void *context) {
    printf("Caught real-time signal %d\n", signo);
    printf("Value passed: %d\n", info->si_value.sival_int);
}

int main() {
    struct sigaction sa;
    memset(&sa, 0, sizeof(sa));
    sa.sa_sigaction = rt_sighandler;
    sa.sa_flags = SA_SIGINFO;

    // Set up handler for all real-time signals
    for (int i = SIGRTMIN_VALUE; i <= SIGRTMAX_VALUE; i++) {
        if (sigaction(i, &sa, NULL) == -1) {
            perror("sigaction");
            exit(1);
        }
    }

    // Send a real-time signal to self
    union sigval value;
    value.sival_int = 42;
    if (sigqueue(getpid(), SIGRTMIN_VALUE, value) == -1) {
        perror("sigqueue");
        exit(1);
    }

    sleep(1);  // Allow time for signal handling

    return 0;
}
```

## 4. Timer and Alarm Functions

### Using alarm()

Key Concepts:
- Setting up alarms
- Handling SIGALRM

Example: Using alarm()

```c
#include <stdio.h>
#include <unistd.h>
#include <signal.h>

void alarm_handler(int signum) {
    printf("Alarm fired!\n");
}

int main() {
    signal(SIGALRM, alarm_handler);

    printf("Setting alarm for 5 seconds...\n");
    alarm(5);

    pause();  // Wait for a signal to be delivered

    return 0;
}
```

### POSIX Timers

Key Concepts:
- Creating and using POSIX timers
- Timer expiration notifications

Example: Using POSIX timers

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <time.h>

#define CLOCKID CLOCK_REALTIME
#define SIG SIGRTMIN

static void handler(int sig, siginfo_t *si, void *uc) {
    timer_t *tidp;
    tidp = si->si_value.sival_ptr;

    printf("Caught signal %d\n", sig);
}

int main() {
    timer_t timerid;
    struct sigevent sev;
    struct itimerspec its;
    struct sigaction sa;

    sa.sa_flags = SA_SIGINFO;
    sa.sa_sigaction = handler;
    sigemptyset(&sa.sa_mask);
    if (sigaction(SIG, &sa, NULL) == -1) {
        perror("sigaction");
        exit(1);
    }

    sev.sigev_notify = SIGEV_SIGNAL;
    sev.sigev_signo = SIG;
    sev.sigev_value.sival_ptr = &timerid;
    if (timer_create(CLOCKID, &sev, &timerid) == -1) {
        perror("timer_create");
        exit(1);
    }

    its.it_value.tv_sec = 5;
    its.it_value.tv_nsec = 0;
    its.it_interval.tv_sec = 0;
    its.it_interval.tv_nsec = 0;

    if (timer_settime(timerid, 0, &its, NULL) == -1) {
        perror("timer_settime");
        exit(1);
    }

    sleep(6);  // Wait for the timer to expire

    timer_delete(timerid);
    return 0;
}
```

## 5. Real-Time Scheduling

### Introduction to Real-Time Scheduling

Key Concepts:
- Real-time priorities
- SCHED_FIFO and SCHED_RR scheduling policies

### Setting Real-Time Priorities

Example: Setting real-time priority

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sched.h>

int main() {
    struct sched_param param;
    int max_priority;

    max_priority = sched_get_priority_max(SCHED_FIFO);
    if (max_priority == -1) {
        perror("sched_get_priority_max");
        exit(1);
    }

    param.sched_priority = max_priority;
    if (sched_setscheduler(0, SCHED_FIFO, &param) == -1) {
        perror("sched_setscheduler");
        exit(1);
    }

    printf("Set to SCHED_FIFO with maximum priority.\n");

    // Your real-time task here
    sleep(10);

    return 0;
}
```

## 6. Best Practices and Considerations

- Signal-safe functions
- Avoiding race conditions with signals
- Proper use of real-time features

## Practice Exercises

1. Implement a simple shell that can handle Ctrl+C (SIGINT) and Ctrl+Z (SIGTSTP).

2. Create a program that uses multiple real-time signals to communicate between processes.

3. Develop a real-time task scheduler using POSIX timers and real-time priorities.

4. Implement a signal-based IPC mechanism for synchronizing multiple processes.

## Important Tools and Utilities

- kill: Send signals to processes
- top: View and manage process priorities
- chrt: Manipulate real-time attributes of a process

## Additional Resources

1. "The Linux Programming Interface" by Michael Kerrisk (chapters on signals and timers)
2. "Programming with POSIX Threads" by David R. Butenhof (for understanding real-time concepts)
3. Linux man pages: man 7 signal, man 7 time, man 7 sched
4. "Real-Time Linux Programming" by John Shapley Gray
