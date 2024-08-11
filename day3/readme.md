Here's the content for Day 3 in Markdown format:

```markdown
# Day 3: Process Scheduling and Memory Management

## 1. Process Scheduling: CFS (Completely Fair Scheduler)

### Introduction
The Completely Fair Scheduler (CFS) is the default process scheduler in the Linux kernel. It aims to maximize overall CPU utilization while providing fair CPU time to all processes.

### Key Concepts

1. **Virtual Runtime (vruntime)**
   - Tracks how long a process has been running
   - Used to determine which process should run next

2. **Red-Black Tree**
   - Self-balancing binary search tree
   - Stores runnable processes, sorted by vruntime

3. **Time Slice**
   - Dynamic, based on the number of runnable processes
   - Calculated to give each process a fair share of CPU time

4. **Nice Values**
   - Range from -20 (highest priority) to 19 (lowest priority)
   - Affect the rate at which vruntime increases

5. **Load Balancing**
   - Distributes processes across multiple CPU cores

### CFS Algorithm (simplified)

1. When a CPU needs to schedule a new process:
   - Pick the leftmost node in the red-black tree (lowest vruntime)
2. Run the selected process for its calculated time slice
3. Update the process's vruntime
4. Re-insert the process into the tree
5. Repeat

### Practical Example: Observing Scheduler Behavior

```bash
# Run a CPU-intensive task
stress --cpu 1 &

# Observe process priorities and CPU usage
top

# Change process priority
renice -n 10 -p PID

# View detailed scheduler statistics
cat /proc/schedstat

# Trace scheduler events
trace-cmd record -p function_graph -g sched_switch sleep 1
trace-cmd report
```

### Gotchas and Best Practices

1. **Priority Inversion**: Be aware of potential priority inversion issues in real-time systems.
2. **CPU Affinity**: Use CPU affinity to bind processes to specific cores when necessary.
3. **Nice Values**: Use nice values judiciously; extreme values can lead to starvation.
4. **Real-Time Priorities**: For time-critical tasks, consider using real-time scheduling policies (SCHED_FIFO or SCHED_RR).

### Interview Questions

1. Q: How does CFS achieve fairness in CPU allocation?
   A: CFS uses the concept of virtual runtime (vruntime) to track how long each process has been running. It always selects the process with the lowest vruntime to run next, ensuring that all processes get a fair share of CPU time.

2. Q: What is the role of the red-black tree in CFS?
   A: The red-black tree is used to efficiently store and retrieve runnable processes, sorted by their vruntime. This allows CFS to quickly select the process with the lowest vruntime.

3. Q: How do nice values affect scheduling in CFS?
   A: Nice values affect the rate at which a process's vruntime increases. Processes with lower nice values (higher priority) have their vruntime increase more slowly, giving them more CPU time.

4. Q: What is the difference between CFS and real-time schedulers like SCHED_FIFO?
   A: CFS aims for overall fairness and is suitable for general-purpose computing. Real-time schedulers like SCHED_FIFO provide strict prioritization and are used for time-critical tasks where predictable response times are crucial.

5. Q: How does CFS handle multicore systems?
   A: CFS includes load balancing mechanisms to distribute processes across multiple CPU cores, aiming to maximize overall system utilization while maintaining fairness.

## 2. Memory Management: Page Tables and TLB

### Introduction
Memory management is a crucial aspect of operating systems. Linux uses virtual memory and paging to provide each process with its own address space and to efficiently manage physical memory.

### Key Concepts

1. **Virtual Memory**
   - Provides an abstraction of physical memory
   - Allows each process to have its own address space

2. **Page Tables**
   - Data structures that map virtual addresses to physical addresses
   - Multi-level page tables in modern systems (e.g., 4-level on x86-64)

3. **Translation Lookaside Buffer (TLB)**
   - Cache for recent virtual-to-physical address translations
   - Speeds up memory access by reducing page table walks

4. **Page Faults**
   - Occur when a memory access can't be satisfied by current page tables
   - Types: Minor (page is in memory but not mapped) and Major (page needs to be loaded from disk)

5. **Huge Pages**
   - Larger page sizes (e.g., 2MB or 1GB instead of 4KB)
   - Reduce TLB misses for large memory allocations

### Page Table Walk (x86-64, 4-level paging)

1. CR3 register points to the Page Global Directory (PGD)
2. PGD entry points to Page Upper Directory (PUD)
3. PUD entry points to Page Middle Directory (PMD)
4. PMD entry points to Page Table Entry (PTE)
5. PTE contains the physical page frame number

### Practical Example: Observing Page Faults

```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/resource.h>

int main() {
    struct rusage usage;
    int *array;
    size_t size = 1024 * 1024 * 100;  // 100 MB

    // Allocate memory
    array = malloc(size * sizeof(int));
    if (array == NULL) {
        perror("malloc failed");
        return 1;
    }

    // Get initial page fault count
    getrusage(RUSAGE_SELF, &usage);
    long initial_faults = usage.ru_majflt + usage.ru_minflt;

    // Access memory, causing page faults
    for (size_t i = 0; i < size; i += 1024) {
        array[i] = i;
    }

    // Get final page fault count
    getrusage(RUSAGE_SELF, &usage);
    long final_faults = usage.ru_majflt + usage.ru_minflt;

    printf("Page faults: %ld\n", final_faults - initial_faults);

    free(array);
    return 0;
}
```

### Gotchas and Best Practices

1. **TLB Thrashing**: Avoid accessing many widely spread memory locations in short time.
2. **Huge Pages**: Use huge pages for large memory allocations to reduce TLB misses.
3. **Memory Alignment**: Align data structures to page boundaries for better performance.
4. **NUMA Awareness**: On NUMA systems, be aware of memory node allocation for optimal performance.

### Interview Questions

1. Q: What is the purpose of virtual memory?
   A: Virtual memory provides process isolation, allows for memory overcommitment, simplifies memory allocation, and enables features like demand paging and swapping.

2. Q: Explain the difference between a page fault and a segmentation fault.
   A: A page fault occurs when a valid memory access can't be immediately satisfied by the current page tables. A segmentation fault occurs when a program tries to access memory it's not allowed to access.

3. Q: How does the TLB improve memory access performance?
   A: The TLB caches recent virtual-to-physical address translations, reducing the need for expensive page table walks for frequently accessed memory locations.

4. Q: What are the advantages and disadvantages of using huge pages?
   A: Advantages include reduced TLB misses and potentially improved performance for large memory allocations. Disadvantages include increased memory fragmentation and potentially wasted memory for small allocations.

5. Q: How does Linux handle memory overcommitment?
   A: Linux allows memory overcommitment by default. It allocates virtual memory liberally but only assigns physical pages when the memory is actually used. If physical memory runs low, the OOM (Out of Memory) killer may terminate processes to free up memory.

## 3. Memory Allocation: Slab Allocator and Buddy System

### Introduction
Efficient memory allocation is crucial for system performance. Linux uses two main allocation systems: the Buddy System for large allocations and the Slab Allocator for smaller, more frequent allocations.

### Key Concepts

1. **Buddy System**
   - Used for allocating larger chunks of memory (pages)
   - Splits memory into power-of-two sized blocks
   - Efficient for coalescing free memory

2. **Slab Allocator**
   - Used for allocating smaller objects
   - Maintains caches of commonly-sized objects
   - Reduces fragmentation and improves allocation speed

3. **SLUB Allocator**
   - Modern version of the Slab allocator
   - Simplified design with better performance

4. **Fragmentation**
   - Internal: Wasted space within allocated blocks
   - External: Free memory split into small, unusable chunks

### Buddy System Algorithm

1. Maintain free lists for each order (power of 2) of pages
2. For allocation:
   - Find the smallest order that can satisfy the request
   - If not available, split a larger block and add unused part to appropriate free list
3. For deallocation:
   - Free the block
   - Check if buddy is also free; if so, coalesce and move to higher order

### Slab Allocator Concept

1. Create caches for commonly-sized objects
2. Allocate large chunks of memory (slabs) from the buddy system
3. Divide slabs into object-sized pieces
4. Maintain lists of full, partial, and empty slabs
5. Allocate from partial slabs when possible

### Practical Example: Kernel Module to Observe Slab Allocator

```c
#include <linux/module.h>
#include <linux/kernel.h>
#include <linux/init.h>
#include <linux/slab.h>

static struct kmem_cache *my_cache;

static int __init my_init(void)
{
    int i;
    void *obj;

    // Create a cache for objects of size 256 bytes
    my_cache = kmem_cache_create("my_cache", 256, 0, 0, NULL);
    if (!my_cache)
        return -ENOMEM;

    // Allocate and free objects
    for (i = 0; i < 1000; i++) {
        obj = kmem_cache_alloc(my_cache, GFP_KERNEL);
        if (obj)
            kmem_cache_free(my_cache, obj);
    }

    printk(KERN_INFO "My cache: %s\n", my_cache->name);
    printk(KERN_INFO "Object size: %d\n", my_cache->size);
    printk(KERN_INFO "Slab size: %d\n", my_cache->size * my_cache->num);

    return 0;
}

static void __exit my_exit(void)
{
    kmem_cache_destroy(my_cache);
}

module_init(my_init);
module_exit(my_exit);

MODULE_LICENSE("GPL");
MODULE_AUTHOR("Your Name");
```

### Gotchas and Best Practices

1. **Right-Sizing**: Choose appropriate object sizes to minimize internal fragmentation.
2. **Cache Sharing**: Share caches between similar-sized objects when possible.
3. **Memory Pressure**: Be aware of memory pressure and implement proper reclaim mechanisms.
4. **Debugging**: Use tools like `slabinfo` and `/proc/slabinfo` for debugging slab-related issues.

### Interview Questions

1. Q: What is the main advantage of the Buddy System for memory allocation?
   A: The Buddy System allows for efficient coalescing of free memory blocks, reducing external fragmentation. It's particularly good for managing larger chunks of memory.

2. Q: How does the Slab Allocator improve upon the Buddy System for small allocations?
   A: The Slab Allocator reduces internal fragmentation and improves allocation speed for small objects by maintaining caches of commonly-sized objects.

3. Q: What is the difference between SLAB and SLUB allocators?
   A: SLUB is a more modern version of the SLAB allocator. It has a simpler design, better memory efficiency, and improved performance, especially on multi-core systems.

4. Q: How does the kernel decide whether to use the Buddy System or the Slab Allocator?
   A: Generally, the Buddy System is used for page-sized and larger allocations, while the Slab Allocator is used for smaller, sub-page allocations. The exact threshold can vary.

5. Q: What is the purpose of kmalloc() and when would you use it instead of a slab cache?
   A: `kmalloc()` is a general-purpose allocator for small memory allocations in the kernel. It's simpler to use than creating a custom slab cache and is suitable for one-off or infrequent allocations of varying sizes.
```

This content covers the three main topics for Day 3: Process Scheduling (CFS), Memory Management (Page Tables and TLB), and Memory Allocation (Slab Allocator and Buddy System). It provides an in-depth look at each topic, including practical examples, key concepts, and interview questions.