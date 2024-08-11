# Day 13: Signal Handling and Real-Time Programming

## 1. Signal Handling

### Introduction to Signals

Key Concepts:
- What are signals?
- Common signals and their meanings
- Signal delivery and handling

### Basic Signal Handling

Key Concepts:
- Using signal() function (deprecated)
- Using sigaction() for robust signal handling

Example: Basic signal handling with sigaction()

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

void sigint_handler(int signo) {
    printf("Caught SIGINT!\n");
}

int main() {
    struct sigaction sa;
    sa.sa_handler = sigint_handler;
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;

    if (sigaction(SIGINT, &sa, NULL) == -1) {
        perror("sigaction");
        exit(1);
    }

    printf("Waiting for SIGINT...\n");
    while(1) {
        sleep(1);
    }

    return 0;
}
```

### Advanced Signal Handling

Key Concepts:
- Blocking and unblocking signals
- Handling multiple signals
- Reentrant functions and async-signal-safe functions

Example: Blocking signals and using sigprocmask()

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

void sigint_handler(int signo) {
    printf("Caught SIGINT!\n");
}

int main() {
    struct sigaction sa;
    sigset_t mask, oldmask;

    sa.sa_handler = sigint_handler;
    sigemptyset(&sa.sa_mask);
    sa.sa_flags = 0;

    if (sigaction(SIGINT, &sa, NULL) == -1) {
        perror("sigaction");
        exit(1);
    }

    sigemptyset(&mask);
    sigaddset(&mask, SIGINT);

    if (sigprocmask(SIG_BLOCK, &mask, &oldmask) == -1) {
        perror("sigprocmask");
        exit(1);
    }

    printf("SIGINT is now blocked. Sleeping for 5 seconds...\n");
    sleep(5);

    printf("Unblocking SIGINT...\n");
    if (sigprocmask(SIG_SETMASK, &oldmask, NULL) == -1) {
        perror("sigprocmask");
        exit(1);
    }

    printf("SIGINT unblocked. Waiting for signals...\n");
    while(1) {
        sleep(1);
    }

    return 0;
}
```

### Signal Sets and Masks

Key Concepts:
- Creating and manipulating signal sets
- Process signal masks

### Sending Signals

Key Concepts:
- Using kill() and raise()
- Sending signals between processes

Example: Sending a signal to another process

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <PID>\n", argv[0]);
        exit(1);
    }

    pid_t pid = atoi(argv[1]);

    if (kill(pid, SIGINT) == -1) {
        perror("kill");
        exit(1);
    }

    printf("Sent SIGINT to process %d\n", pid);

    return 0;
}
```

## 2. Real-Time Programming

### Introduction to Real-Time Systems

Key Concepts:
- Characteristics of real-time systems
- Hard vs. soft real-time systems
- Real-time requirements in Linux

### Real-Time Scheduling

Key Concepts:
- SCHED_FIFO and SCHED_RR scheduling policies
- Setting and getting scheduling parameters

Example: Setting real-time scheduling policy

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

    printf("Set to SCHED_FIFO with maximum priority\n");

    // Your real-time task here
    sleep(10);

    return 0;
}
```

### Real-Time Signals

Key Concepts:
- Real-time signals vs. standard signals
- Using real-time signals for inter-process communication

Example: Using real-time signals

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>

void rt_sighandler(int signo, siginfo_t *info, void *context) {
    printf("Received real-time signal %d\n", signo - SIGRTMIN);
    printf("Value passed: %d\n", info->si_value.sival_int);
}

int main() {
    struct sigaction sa;

    sa.sa_sigaction = rt_sighandler;
    sa.sa_flags = SA_SIGINFO;
    sigemptyset(&sa.sa_mask);

    if (sigaction(SIGRTMIN, &sa, NULL) == -1) {
        perror("sigaction");
        exit(1);
    }

    // Send a real-time signal to self
    union sigval value;
    value.sival_int = 42;

    if (sigqueue(getpid(), SIGRTMIN, value) == -1) {
        perror("sigqueue");
        exit(1);
    }

    sleep(1);  // Wait for signal to be delivered

    return 0;
}
```

### Timer and Clock Functions

Key Concepts:
- High-resolution timers
- POSIX clocks
- Timer creation and management

Example: Using high-resolution timer

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <signal.h>
#include <time.h>

void timer_handler(int signo, siginfo_t *info, void *context) {
    printf("Timer expired!\n");
}

int main() {
    struct sigaction sa;
    struct sigevent sev;
    struct itimerspec its;
    timer_t timerid;

    sa.sa_flags = SA_SIGINFO;
    sa.sa_sigaction = timer_handler;
    sigemptyset(&sa.sa_mask);

    if (sigaction(SIGRTMIN, &sa, NULL) == -1) {
        perror("sigaction");
        exit(1);
    }

    sev.sigev_notify = SIGEV_SIGNAL;
    sev.sigev_signo = SIGRTMIN;
    sev.sigev_value.sival_ptr = &timerid;

    if (timer_create(CLOCK_REALTIME, &sev, &timerid) == -1) {
        perror("timer_create");
        exit(1);
    }

    its.it_value.tv_sec = 1;
    its.it_value.tv_nsec = 0;
    its.it_interval.tv_sec = 1;
    its.it_interval.tv_nsec = 0;

    if (timer_settime(timerid, 0, &its, NULL) == -1) {
        perror("timer_settime");
        exit(1);
    }

    while(1) {
        sleep(1);
    }

    return 0;
}
```

### Memory Locking

Key Concepts:
- Preventing memory from being paged out
- Using mlockall() and mlock()

Example: Locking memory for a real-time process

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/mman.h>

int main() {
    if (mlockall(MCL_CURRENT | MCL_FUTURE) == -1) {
        perror("mlockall");
        exit(1);
    }

    printf("Memory locked. Running real-time task...\n");

    // Your real-time task here
    sleep(10);

    if (munlockall() == -1) {
        perror("munlockall");
        exit(1);
    }

    return 0;
}
```

## Practice Exercises

1. Implement a simple shell that can handle job control using signals.

2. Create a real-time program that responds to external events using real-time signals.

3. Develop a periodic task scheduler using high-resolution timers.

4. Write a program that demonstrates the use of different scheduling policies and priorities.

## Important Tools and Utilities

- chrt: Manipulate real-time attributes of a process
- nice and renice: Run a program with modified scheduling priority
- top: View and manage process priorities
- stress-ng: Stress test a computer system in various selectable ways

## Additional Resources

1. "Linux System Programming" by Robert Love
2. "Real-Time Linux Programming" by John Lombardo
3. "The Linux Programming Interface" by Michael Kerrisk
4. "Programming for the Real World: POSIX.4" by Bill O. Gallmeister
