
# Day 1: Advanced Process Management

## 1. Process Creation and Termination Internals

### Fork() Internals
- Copy-on-write (COW) mechanism
- Page table duplication
- File descriptor inheritance

Example:
```c
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();
    if (pid == 0) {
        printf("Child process: PID = %d\n", getpid());
        exit(0);
    } else if (pid > 0) {
        printf("Parent process: Child's PID = %d\n", pid);
        wait(NULL);
    } else {
        perror("fork failed");
        return 1;
    }
    return 0;
}
```

### Exec() Family Internals
- Memory space replacement
- File descriptor preservation
- Signal handling reset

Example:
```c
#include <stdio.h>
#include <unistd.h>

int main() {
    char *args[] = {"ls", "-l", NULL};
    execvp(args[0], args);
    perror("execvp failed");
    return 1;
}
```

### Process Termination and Zombie Processes
- Resource cleanup
- Exit status preservation
- Zombie state and reaping

Example:
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>

int main() {
    pid_t pid = fork();
    if (pid == 0) {
        printf("Child process exiting\n");
        exit(42);
    } else if (pid > 0) {
        int status;
        sleep(2);  // Simulate work, child becomes zombie
        waitpid(pid, &status, 0);
        if (WIFEXITED(status)) {
            printf("Child exited with status %d\n", WEXITSTATUS(status));
        }
    }
    return 0;
}
```

## 2. Real-time Process Scheduling

### SCHED_FIFO and SCHED_RR Policies
- Priority-based preemptive scheduling
- Time slice allocation in SCHED_RR

Example:
```c
#include <stdio.h>
#include <pthread.h>
#include <sched.h>

void *thread_function(void *arg) {
    printf("Thread running with policy: ");
    switch(sched_getscheduler(0)) {
        case SCHED_FIFO:  printf("SCHED_FIFO\n"); break;
        case SCHED_RR:    printf("SCHED_RR\n"); break;
        case SCHED_OTHER: printf("SCHED_OTHER\n"); break;
        default:          printf("Unknown\n");
    }
    return NULL;
}

int main() {
    pthread_t thread;
    pthread_attr_t attr;
    struct sched_param param;

    pthread_attr_init(&attr);
    pthread_attr_setschedpolicy(&attr, SCHED_FIFO);
    param.sched_priority = sched_get_priority_max(SCHED_FIFO);
    pthread_attr_setschedparam(&attr, &param);

    pthread_create(&thread, &attr, thread_function, NULL);
    pthread_join(thread, NULL);

    return 0;
}
```

### Changing Process Priorities
- nice() and setpriority() system calls
- Real-time priorities vs. nice values

Example:
```c
#include <stdio.h>
#include <sys/resource.h>
#include <unistd.h>

int main() {
    int current_priority = getpriority(PRIO_PROCESS, 0);
    printf("Current priority: %d\n", current_priority);

    nice(10);  // Decrease priority

    current_priority = getpriority(PRIO_PROCESS, 0);
    printf("New priority: %d\n", current_priority);

    return 0;
}
```

## 3. Resource Limits and Control Groups

### Setting Resource Limits
- Using setrlimit() to set process limits
- Common limits: CPU time, file size, number of processes

Example:
```c
#include <stdio.h>
#include <sys/time.h>
#include <sys/resource.h>

int main() {
    struct rlimit limit;

    getrlimit(RLIMIT_NOFILE, &limit);
    printf("Current NOFILE limit: soft=%lu hard=%lu\n",
           limit.rlim_cur, limit.rlim_max);

    limit.rlim_cur = 1024;
    setrlimit(RLIMIT_NOFILE, &limit);

    getrlimit(RLIMIT_NOFILE, &limit);
    printf("New NOFILE limit: soft=%lu hard=%lu\n",
           limit.rlim_cur, limit.rlim_max);

    return 0;
}
```

### Control Groups (cgroups) Basics
- Creating and managing cgroups
- Limiting CPU, memory, and I/O for process groups

Example (using libcgroup):
```c
#include <stdio.h>
#include <libcgroup.h>

int main() {
    struct cgroup *cg;
    struct cgroup_controller *cg_controller;

    cgroup_init();

    cg = cgroup_new_cgroup("my_cgroup");
    cg_controller = cgroup_add_controller(cg, "cpu");

    cgroup_add_value_string(cg_controller, "cpu.shares", "512");

    if (cgroup_create_cgroup(cg, 0) != 0) {
        printf("Failed to create cgroup\n");
        return 1;
    }

    printf("Created cgroup 'my_cgroup' with CPU shares set to 512\n");

    cgroup_free(&cg);
    return 0;
}
```

## Practice Exercises

1. Write a program that creates a child process, sets its scheduling policy to SCHED_FIFO, and measures the execution time difference compared to the parent process running under SCHED_OTHER.

2. Implement a simple shell that supports job control, including background processes and bringing processes to the foreground.

3. Create a program that sets various resource limits for itself, then tries to exceed these limits and handles the resulting signals.

4. Write a C program that creates a cgroup, sets memory limits, and runs a child process within that cgroup. Monitor the memory usage and enforce the limits.

## Important Tools and Utilities

- strace: For tracing system calls and signals
- ps, top: For monitoring process states and resource usage
- cgcreate, cgexec: Command-line tools for cgroup management
- chrt: For manipulating real-time attributes of processes

## Best Practices and Considerations

1. Always check return values of system calls and handle errors appropriately.
2. Be cautious when using real-time scheduling, as it can starve other processes.
3. When working with cgroups, ensure proper cleanup to avoid resource leaks.
4. Use appropriate privileges (e.g., CAP_SYS_NICE) when modifying process priorities.
5. Consider the impact on system stability when adjusting resource limits.

## Additional Resources

1. "Understanding the Linux Kernel" by Daniel P. Bovet and Marco Cesati
2. Linux man pages: man 2 fork, man 2 execve, man 2 wait, man 2 nice, man 2 setpriority
3. Linux kernel documentation on scheduling: https://www.kernel.org/doc/html/latest/scheduler/sched-design-CFS.html
4. Control Groups documentation: https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt
