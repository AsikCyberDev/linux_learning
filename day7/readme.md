
# Day 7: Memory Management and Virtual Memory

## 1. Memory Management in Linux

### Introduction
Memory management is a critical component of the Linux kernel, responsible for allocating and deallocating memory resources to processes and the kernel itself. Understanding how Linux manages memory is crucial for both system administrators and developers working on performance-critical applications.

### Key Concepts

1. **Physical vs. Virtual Memory**
   - Physical memory: Actual RAM in the system
   - Virtual memory: Abstraction that provides each process with its own address space

2. **Page Tables**
   - Data structures that map virtual addresses to physical addresses
   - Multi-level page tables for efficiency

3. **Memory Zones**
   - ZONE_DMA, ZONE_NORMAL, ZONE_HIGHMEM
   - Used to manage different types of memory

4. **Slab Allocator**
   - Efficient allocation of kernel objects
   - Reduces fragmentation and improves performance

5. **OOM Killer**
   - Out-of-Memory Killer
   - Terminates processes when system runs out of memory

### Memory Allocation in User Space

1. **Heap Allocation**
   - `malloc()`, `free()`, `realloc()`
   - Managed by the C library (e.g., glibc)

2. **Stack Allocation**
   - Automatic allocation for local variables
   - Managed by the compiler

3. **mmap()**
   - Allows direct mapping of files or anonymous memory

### Kernel Space Memory Allocation

1. **kmalloc()**
   - Allocates physically contiguous memory
   - Suitable for small allocations

2. **vmalloc()**
   - Allocates virtually contiguous memory
   - Can allocate larger chunks but with higher overhead

3. **slab allocator functions**
   - `kmem_cache_alloc()`, `kmem_cache_free()`
   - Efficient for frequently used object types

### Practical Example: Analyzing System Memory

Here's a simple shell script to display basic memory information:

```bash
#!/bin/bash

echo "Total Memory:"
free -h | grep Mem | awk '{print $2}'

echo "Used Memory:"
free -h | grep Mem | awk '{print $3}'

echo "Free Memory:"
free -h | grep Mem | awk '{print $4}'

echo "Cached Memory:"
free -h | grep Mem | awk '{print $6}'

echo "Swap Usage:"
free -h | grep Swap | awk '{print $3}'
```

### Gotchas and Best Practices

1. **Memory Leaks**: Always free allocated memory in user space programs.
2. **Fragmentation**: Be aware of memory fragmentation, especially in long-running processes.
3. **Swapping**: Excessive swapping can severely impact system performance.
4. **OOM Killer**: Configure OOM killer settings appropriately for critical systems.
5. **Memory Overcommit**: Understand the implications of memory overcommit settings.

### Interview Questions

1. Q: What is the purpose of virtual memory in Linux?
   A: Virtual memory provides an abstraction layer between the physical memory and processes. It allows each process to have its own address space, enables memory protection between processes, facilitates efficient use of physical memory through paging, and allows the system to use more memory than physically available through swapping.

2. Q: Explain the difference between `kmalloc()` and `vmalloc()`.
   A: `kmalloc()` allocates physically contiguous memory and is faster but limited in size. `vmalloc()` allocates virtually contiguous memory, which can be physically discontinuous. `vmalloc()` can allocate larger chunks but has higher overhead due to page table modifications.

3. Q: What is the slab allocator, and why is it used?
   A: The slab allocator is a memory management mechanism used for efficient allocation of kernel objects. It maintains caches of commonly-used objects, reducing fragmentation and improving performance by reusing memory and avoiding frequent allocations and deallocations.

4. Q: How does the OOM killer decide which process to terminate?
   A: The OOM killer uses a scoring system based on various factors including:
      - Memory usage of the process
      - CPU usage
      - Process priority (nice value)
      - Time the process has been running
      - Whether the process is privileged or not
   It generally tries to kill the process that will free up the most memory with the least impact on the system.

5. Q: What is memory overcommit, and how can it be configured in Linux?
   A: Memory overcommit is a feature where the kernel allows processes to allocate more memory than physically available, betting on the fact that not all allocated memory will be used simultaneously. It can be configured using the `/proc/sys/vm/overcommit_memory` sysctl:
      - 0: Heuristic overcommit (default)
      - 1: Always overcommit
      - 2: Never overcommit
   The overcommit ratio can be adjusted with `/proc/sys/vm/overcommit_ratio`.

## 2. Virtual Memory and Paging

### Introduction
Virtual memory is a memory management technique that provides an abstraction of the storage resources available to a process. It allows processes to use more memory than physically available and provides memory protection between processes. Paging is the mechanism used to implement virtual memory.

### Key Concepts

1. **Page**
   - Fixed-size block of memory (typically 4KB on x86 systems)
   - Basic unit of memory management for virtual memory

2. **Page Table**
   - Data structure that maps virtual page numbers to physical frame numbers
   - Multi-level page tables for efficiency (e.g., 4-level on x86-64)

3. **Translation Lookaside Buffer (TLB)**
   - Cache for page table entries
   - Speeds up virtual-to-physical address translation

4. **Page Fault**
   - Occurs when a process accesses a page that is not currently in physical memory
   - Handled by the kernel's page fault handler

5. **Swapping**
   - Process of moving pages between RAM and disk storage
   - Allows system to use more memory than physically available

### Page Replacement Algorithms

1. **Least Recently Used (LRU)**
   - Replaces the page that hasn't been used for the longest time
2. **Clock Algorithm**
   - Approximation of LRU, more efficient to implement
3. **Not Recently Used (NRU)**
   - Considers both reference and modify bits

### Practical Example: Analyzing Page Faults

Here's a simple C program that deliberately causes page faults:

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/resource.h>

#define ARRAY_SIZE (1024 * 1024 * 100)  // 100 MB

int main() {
    struct rusage usage_start, usage_end;
    int *large_array = malloc(ARRAY_SIZE * sizeof(int));

    getrusage(RUSAGE_SELF, &usage_start);

    // Access the array in a way that causes page faults
    for (int i = 0; i < ARRAY_SIZE; i += 1024) {
        large_array[i] = i;
    }

    getrusage(RUSAGE_SELF, &usage_end);

    long page_faults = usage_end.ru_majflt - usage_start.ru_majflt;
    printf("Major Page Faults: %ld\n", page_faults);

    free(large_array);
    return 0;
}
```

### Gotchas and Best Practices

1. **Page Size**: Be aware of the system's page size when optimizing memory access patterns.
2. **Huge Pages**: Consider using huge pages for large memory allocations to reduce TLB misses.
3. **Memory Access Patterns**: Design algorithms with cache-friendly memory access patterns.
4. **Swappiness**: Adjust the swappiness parameter to control how aggressively the system swaps.
5. **Memory Locking**: Use `mlock()` judiciously to prevent critical pages from being swapped out.

### Interview Questions

1. Q: What is the purpose of the Translation Lookaside Buffer (TLB)?
   A: The TLB is a cache that stores recent virtual-to-physical address translations. It significantly speeds up memory access by reducing the number of times the CPU needs to walk the page table hierarchy for address translation.

2. Q: Explain the difference between a major and minor page fault.
   A: A minor page fault occurs when the required page is in memory but not mapped into the process's page table. A major page fault occurs when the required page is not in memory and needs to be loaded from disk. Major page faults are more expensive in terms of performance.

3. Q: How does the Linux kernel handle a page fault?
   A: When a page fault occurs:
      1. The CPU generates an exception and transfers control to the kernel.
      2. The kernel identifies the type of fault and the faulting address.
      3. If it's a valid access, the kernel allocates a physical frame if necessary.
      4. The kernel updates the page table to map the virtual page to the physical frame.
      5. If the page was swapped out, it's read back into memory.
      6. The process is resumed at the instruction that caused the fault.

4. Q: What is thrashing, and how can it be mitigated?
   A: Thrashing occurs when a system spends more time paging data in and out of memory than executing actual processes. It can be mitigated by:
      - Adding more physical memory
      - Reducing the number of running processes
      - Adjusting the swappiness parameter
      - Optimizing application memory usage

5. Q: Explain the concept of demand paging.
   A: Demand paging is a method of virtual memory management where pages are only brought into physical memory when they are actually needed (demanded) by a process. This allows programs to start quickly and use memory efficiently, as only the parts of the program that are actively used need to be in memory.

This content covers the essentials of Memory Management and Virtual Memory in Linux, providing both theoretical knowledge and practical examples. It should give a comprehensive understanding of these critical aspects of operating system design and implementation.
```
