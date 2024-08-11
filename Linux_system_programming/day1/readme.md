# Day 1: Advanced Process Management

## 1. Process Creation and Termination Internals

### Fork() Internals

The `fork()` system call creates a new process by duplicating the calling process. Understanding its internals is crucial for efficient process management.

Key Concepts:
- Copy-on-Write (COW) mechanism: The kernel doesn't immediately copy all resources; instead, it marks pages as read-only and copies them only when modified.
- Page table duplication: The kernel creates a new page table for the child process, initially pointing to the same physical pages as the parent.
- File descriptor inheritance: The child inherits copies of the parent's file descriptors.

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

Explanation:
- `fork()` returns 0 in the child process and the child's PID in the parent process.
- The child process gets a copy of the parent's memory space, but with its own unique PID.
- The parent waits for the child to finish using `wait()` to avoid creating a zombie process.

### Exec() Family Internals

The `exec()` family of functions replaces the current process image with a new process image.

Key Concepts:
- Memory space replacement: The existing process image is replaced, but the process ID remains the same.
- File descriptor preservation: Open file descriptors remain open across an exec call unless the close-on-exec flag is set.
- Signal handling reset: Signal dispositions are reset to their default values, except for ignored signals.

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

Explanation:
- `execvp()` searches for the executable in the PATH and executes it with the given arguments.
- If `execvp()` succeeds, it never returns. If it returns, an error occurred.
- The process image is completely replaced, but the PID remains the same.

### Process Termination and Zombie Processes

Understanding how processes terminate and how to handle zombie processes is crucial for proper resource management.

Key Concepts:
- Resource cleanup: The kernel releases most resources, but the parent must read the exit status.
- Exit status preservation: The kernel keeps minimal information about the terminated process until the parent retrieves it.
- Zombie state: A process that has terminated but whose exit status hasn't been read by its parent.

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

Explanation:
- The child process exits with status 42.
- The parent sleeps for 2 seconds, during which the child becomes a zombie.
- `waitpid()` retrieves the child's exit status, removing the zombie process.
- `WIFEXITED` and `WEXITSTATUS` macros are used to check and extract the exit status.

## 2. Real-time Process Scheduling

Real-time scheduling allows processes to meet strict timing requirements.

### SCHED_FIFO and SCHED_RR Policies

Key Concepts:
- SCHED_FIFO: First-In-First-Out scheduling without time slicing.
- SCHED_RR: Round-Robin scheduling with time slicing for processes of equal priority.
- Priority levels: Real-time priorities range from 1 to 99, with higher numbers indicating higher priority.

Example:
```c
#include <stdio.h>
#include <pthread.h>
#include <sched.h>

void *thread_function(void *arg) {
    int policy;
    struct sched_param param;
    pthread_getschedparam(pthread_self(), &policy, &param);
    printf("Thread running with policy: ");
    switch(policy) {
        case SCHED_FIFO:  printf("SCHED_FIFO\n"); break;
        case SCHED_RR:    printf("SCHED_RR\n"); break;
        case SCHED_OTHER: printf("SCHED_OTHER\n"); break;
        default:          printf("Unknown\n");
    }
    printf("Priority: %d\n", param.sched_priority);
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

Explanation:
- We set the scheduling policy to SCHED_FIFO and the priority to the maximum allowed.
- The thread reports its scheduling policy and priority.
- Note: Running this program may require root privileges or CAP_SYS_NICE capability.

### Changing Process Priorities

Key Concepts:
- nice values: Range from -20 (highest priority) to 19 (lowest priority).
- Real-time priorities: Separate from nice values, used for SCHED_FIFO and SCHED_RR.

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

Explanation:
- `getpriority()` retrieves the current nice value.
- `nice()` changes the nice value, effectively changing the process priority.
- Positive nice values decrease priority, negative values increase it (requires privileges).

## 3. Resource Limits and Control Groups

### Setting Resource Limits

Resource limits allow controlling the resources a process can consume.

Key Concepts:
- Soft limits: Can be increased by the process up to the hard limit.
- Hard limits: Can only be increased by privileged processes.
- Common limits: CPU time, file size, number of open file descriptors, etc.

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

Explanation:
- `getrlimit()` retrieves the current resource limits.
- `setrlimit()` sets new resource limits.
- RLIMIT_NOFILE controls the maximum number of open file descriptors.

### Control Groups (cgroups) Basics

Cgroups allow grouping processes and managing their resource usage collectively.

Key Concepts:
- Hierarchical organization of processes.
- Controllers for different resources (CPU, memory, I/O, etc.).
- Dynamic resource allocation and limitation.

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

Explanation:
- We create a new cgroup named "my_cgroup".
- We add a CPU controller to the cgroup.
- We set the CPU shares to 512, which determines the relative CPU allocation.
- The cgroup is created in the system.

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

1. "Advanced Programming in the UNIX Environment" by W. Richard Stevens and Stephen A. Rago
2. Linux man pages: man 2 fork, man 2 execve, man 2 wait, man 2 nice, man 2 setpriority
3. Linux kernel documentation on scheduling: https://www.kernel.org/doc/html/latest/scheduler/sched-design-CFS.html
4. Control Groups documentation: https://www.kernel.org/doc/Documentation/cgroup-v1/cgroups.txt
