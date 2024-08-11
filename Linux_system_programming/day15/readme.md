
# Day 15: Advanced Memory Management and Debugging Techniques

## 1. Advanced Memory Management

### Memory Allocation Strategies

Key Concepts:
- Stack vs. Heap allocation
- Custom memory allocators
- Memory pools and arenas

Example: Simple memory pool implementation

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define POOL_SIZE 1024
#define MAX_ALLOCS 100

typedef struct {
    char memory[POOL_SIZE];
    size_t used;
    void* allocations[MAX_ALLOCS];
    size_t alloc_count;
} MemoryPool;

void init_pool(MemoryPool* pool) {
    pool->used = 0;
    pool->alloc_count = 0;
}

void* pool_alloc(MemoryPool* pool, size_t size) {
    if (pool->used + size > POOL_SIZE || pool->alloc_count >= MAX_ALLOCS) {
        return NULL;
    }
    void* ptr = &pool->memory[pool->used];
    pool->used += size;
    pool->allocations[pool->alloc_count++] = ptr;
    return ptr;
}

void pool_free(MemoryPool* pool, void* ptr) {
    for (size_t i = 0; i < pool->alloc_count; i++) {
        if (pool->allocations[i] == ptr) {
            memmove(&pool->allocations[i], &pool->allocations[i+1],
                    (pool->alloc_count - i - 1) * sizeof(void*));
            pool->alloc_count--;
            return;
        }
    }
}

int main() {
    MemoryPool pool;
    init_pool(&pool);

    int* numbers = pool_alloc(&pool, 10 * sizeof(int));
    char* string = pool_alloc(&pool, 20 * sizeof(char));

    if (numbers && string) {
        for (int i = 0; i < 10; i++) {
            numbers[i] = i;
        }
        strcpy(string, "Hello, World!");

        printf("Numbers: ");
        for (int i = 0; i < 10; i++) {
            printf("%d ", numbers[i]);
        }
        printf("\nString: %s\n", string);

        pool_free(&pool, numbers);
        pool_free(&pool, string);
    }

    return 0;
}
```

### Memory Mapping

Key Concepts:
- mmap() and munmap() functions
- Shared memory using mmap()
- Memory-mapped files

Example: Using mmap() to read a file

```c
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <unistd.h>
#include <sys/mman.h>
#include <sys/stat.h>

int main(int argc, char *argv[]) {
    if (argc != 2) {
        fprintf(stderr, "Usage: %s <file>\n", argv[0]);
        exit(1);
    }

    int fd = open(argv[1], O_RDONLY);
    if (fd == -1) {
        perror("open");
        exit(1);
    }

    struct stat sb;
    if (fstat(fd, &sb) == -1) {
        perror("fstat");
        exit(1);
    }

    char *addr = mmap(NULL, sb.st_size, PROT_READ, MAP_PRIVATE, fd, 0);
    if (addr == MAP_FAILED) {
        perror("mmap");
        exit(1);
    }

    write(STDOUT_FILENO, addr, sb.st_size);

    if (munmap(addr, sb.st_size) == -1) {
        perror("munmap");
        exit(1);
    }

    close(fd);
    return 0;
}
```

### Garbage Collection in C

Key Concepts:
- Basic concepts of garbage collection
- Implementing a simple mark-and-sweep collector

(Note: Full implementation of a garbage collector is beyond the scope of this outline, but we can discuss the basic concepts and a simplified example.)

## 2. Debugging Techniques

### Using GDB (GNU Debugger)

Key Concepts:
- Setting breakpoints
- Stepping through code
- Examining variables and memory

Example GDB commands:
```
$ gcc -g program.c -o program
$ gdb ./program
(gdb) break main
(gdb) run
(gdb) next
(gdb) print variable_name
(gdb) backtrace
```

### Memory Debugging with Valgrind

Key Concepts:
- Detecting memory leaks
- Finding use of uninitialized memory
- Identifying data races

Example Valgrind usage:
```
$ gcc -g program.c -o program
$ valgrind --leak-check=full ./program
```

### Static Analysis Tools

Key Concepts:
- Using static analyzers like Clang Static Analyzer or Cppcheck
- Identifying potential bugs and code smells

Example Cppcheck usage:
```
$ cppcheck --enable=all program.c
```

### Writing Testable Code

Key Concepts:
- Unit testing in C
- Test-Driven Development (TDD) principles
- Using frameworks like Check or Unity

Example: Simple unit test using Check framework

```c
#include <check.h>
#include <stdlib.h>

int add(int a, int b) {
    return a + b;
}

START_TEST(test_add)
{
    ck_assert_int_eq(add(2, 3), 5);
    ck_assert_int_eq(add(-1, 1), 0);
    ck_assert_int_eq(add(0, 0), 0);
}
END_TEST

Suite * addition_suite(void)
{
    Suite *s;
    TCase *tc_core;

    s = suite_create("Addition");
    tc_core = tcase_create("Core");

    tcase_add_test(tc_core, test_add);
    suite_add_tcase(s, tc_core);

    return s;
}

int main(void)
{
    int number_failed;
    Suite *s;
    SRunner *sr;

    s = addition_suite();
    sr = srunner_create(s);

    srunner_run_all(sr, CK_NORMAL);
    number_failed = srunner_ntests_failed(sr);
    srunner_free(sr);
    return (number_failed == 0) ? EXIT_SUCCESS : EXIT_FAILURE;
}
```

## 3. Performance Profiling

### Using gprof

Key Concepts:
- Compiling with profiling information
- Running the program to generate profile data
- Analyzing the profile output

Example gprof usage:
```
$ gcc -pg program.c -o program
$ ./program
$ gprof program gmon.out > analysis.txt
```

### Cachegrind for Cache Profiling

Key Concepts:
- Understanding cache behavior
- Identifying cache misses and their impact

Example Cachegrind usage:
```
$ valgrind --tool=cachegrind ./program
```

## Practice Exercises

1. Implement a custom memory allocator that uses a free list to manage memory blocks.

2. Write a program with deliberate memory leaks and use Valgrind to detect and fix them.

3. Create a simple data structure (e.g., linked list) and write comprehensive unit tests for it using a testing framework.

4. Profile a computationally intensive program using gprof and optimize it based on the results.

## Important Tools and Utilities

- gdb: GNU Debugger
- valgrind: Memory debugging and profiling tool
- gprof: Performance profiling tool
- Cppcheck: Static analysis tool for C/C++ code
- AddressSanitizer: Memory error detector

## Additional Resources

1. "Advanced Linux Programming" by Mark Mitchell, Jeffrey Oldham, and Alex Samuel
2. "Debugging with GDB: The GNU Source-Level Debugger" by Richard M. Stallman, et al.
3. "Valgrind 3.3 - Advanced Debugging and Profiling for GNU/Linux applications" by J. Seward, et al.
4. "Test-Driven Development for Embedded C" by James W. Grenning
