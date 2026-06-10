# Rivision
# COMP2017 – All-Week Concept Reference
**Exam prep cheat sheet | Based on lecture slides W1–W13**

---

## Week 1 — Introduction to C

### Big Picture
C is a low-level, procedural language. No objects, no inheritance, no garbage collection. The programmer controls memory directly. Used for OSes, embedded systems, device drivers.

### Key Concepts

**C vs Java**
- C is closer to the hardware; Java runs on a JVM
- C has no classes, no polymorphism, no templates
- C has pointers and direct memory access; Java does not
- C assumes the programmer knows best

**C Program Structure**
- Source files: `.c` | Header files: `.h`
- Every program needs `int main(int argc, char **argv)`
- Compile with: `gcc hello.c -o hello`
- Compiler flags you must use: `-Wall -Wextra -g -fsanitize=address`

**Functions in C**
- Must be declared before use (forward declaration or prototype)
- `int foo(float, char);` — prototype, no body needed
- `void foo(void)` — no parameters, no return
- One definition per name in the whole program

**Control Structures**
- `if / else`, `while`, `do-while`, `for`, `switch/case`
- `switch` uses `case <const>: ... break;` — always add `break` unless intentional fall-through

**Compilation pipeline**
```
source.c → Preprocessor → Compiler → Assembler → Linker → executable
```
- `gcc -c util.c` produces `util.o` (object file only)
- `gcc myprog.c util.o -o myprog` links them

**Module example**
```c
// foo.h
extern int foo();

// foo.c
#include "foo.h"
int foo() { printf("hello\n"); return 0; }

// main.c
#include "foo.h"
int main(int argc, char **argv) { foo(); return 0; }
```

---

## Week 2 — Pointers

### Big Picture
Pointers store memory addresses. They let you directly manipulate memory, pass data by reference, and build dynamic data structures.

### Key Concepts

**Number Systems (Radix)**

| Format | Base | Literal | printf |
|--------|------|---------|--------|
| Decimal | 10 | `123` | `%d` / `%u` |
| Octal | 8 | `021` | `%o` |
| Hex | 16 | `0x11` | `%x` / `%X` |

**Endianness**
- Big-endian: most significant byte stored first (low address)
- Little-endian: least significant byte first — most modern CPUs (x86) are little-endian
- Matters when you share binary data between machines or cast types

**Pointer Basics**
```c
char initial = 'A';
char *initp = &initial;   // initp holds the ADDRESS of initial
// &initial = "address of initial"
// *initp   = "value at the address stored in initp" = 'A'
```

**Declaration syntax**
```c
int *p;         // p is a pointer to int
int **pp;       // pp is a pointer to a pointer to int
char *str;      // str is a pointer to char (used for strings)
```

**NULL pointer**
- `NULL` = address 0, used to mean "points to nothing"
- Always initialise pointers: `int *p = NULL;`
- Dereferencing NULL causes a segfault

**Pointer Arithmetic**
```c
int x[4];
int *p = x;      // p points to x[0]
*(p + 1)         // same as x[1]
*(p + n)         // same as x[n]
```
- `p + n` moves `n * sizeof(*p)` bytes forward
- Pointer arithmetic is TYPE-AWARE

**Arrays and Pointers**
- `char arr[10]` — `arr` IS a pointer to the first element
- `arr[i]` ≡ `*(arr + i)`
- Strings are `char *` or `char[]` ending with `'\0'`

**Moving through a string (idiom)**
```c
while (*str != '\0')
    str++;
```

**Uninitialised values — danger**
- Always initialise variables before use
- `scanf` may fail → variable stays uninitialised → undefined behaviour

**Enumerated types (intro, covered more in W3)**
```c
enum day_name { Sun, Mon, Tue, Wed, Thu, Fri, Sat };
// Maps to integers 0..6; can do Sun++
```

---

## Week 3 — Enums, Structs, Unions, Files

### Big Picture
C gives you tools to define your own data types (structs, enums, unions) and to read/write files using FILE* or file descriptors.

### Key Concepts

**Enums**
```c
enum month_name { JAN, FEB, MAR, ..., DEC, MONTH_UNDEF };
// JAN=0, FEB=1, ... automatically
// Can assign values: enum { A=1, B=5, C }; // C=6
```

**Structs**
```c
struct point {
    int x;
    int y;
};
struct point p1;
p1.x = 3;

// Pointer to struct — use arrow operator
struct point *pp = &p1;
pp->x = 5;   // same as (*pp).x = 5
```

**Memory alignment in structs**
- Compiler adds padding to align fields to their natural size
- `sizeof(struct)` may be larger than sum of fields
```c
struct a { int x; short s1, s2; float y; char c1,c2,c3,c4; };
// sizeof = 16 (not 14) due to padding
```

**Typedef**
```c
typedef struct point Point;
Point p;  // no need to write "struct point" every time

// Or combined:
typedef struct { int x; int y; } Point;
```

**Unions**
```c
union { int a; char b; } x;
x.a = 0x11223344;
// x.b gives first byte only (endian-dependent)
```
- All members share the SAME memory location
- Size = size of largest member
- Only one member is valid at a time

**Bit Fields**
```c
struct IOdev {
    unsigned R_W: 1;   // 1 bit only
    unsigned Dirn: 8;  // 8 bits
    unsigned mode: 3;  // 3 bits
};
```

**Opaque structs (information hiding)**
```c
// foo.h — only declares the type
struct foo_bar;
int foo_get_length(struct foo_bar *b);

// foo.c — defines the actual internals
struct foo_bar { int length_inches; };
```

**Files (stdio)**
```c
FILE *fp = fopen("data.txt", "r");  // "r", "w", "a", "rb", etc.
fscanf(fp, "%d", &num);
fprintf(fp, "%d\n", num);
feof(fp);   // test for end-of-file
fclose(fp);

// Read until EOF
while (!feof(stdin)) {
    int n;
    int nread = fscanf(stdin, "%d", &n);
    if (nread <= 0) break;
    fprintf(stdout, "num: %d\n", n);
}
```

---

## Week 4a — Memory

### Big Picture
C memory is divided into: Code, Global/static, Stack, Heap. Stack = automatic, Heap = manual (malloc/free).

### Key Concepts

**Memory layout**
```
High address ┌─────────────┐
             │   Stack     │  ← grows downward
             │   (↓)       │
             ├─────────────┤
             │   Heap      │  ← grows upward
             │   (↑)       │
             ├─────────────┤
             │   Global /  │
             │   Static    │
             ├─────────────┤
Low address  │   Code      │
             └─────────────┘
```

**Stack**
- Automatic allocation: local variables, function parameters
- Allocated when function is called, freed when it returns
- Fast, fixed size per call frame
- CPU uses SP (Stack Pointer) and PC (Program Counter) registers
- Stack overflow = too many nested calls / too large local arrays

**Heap**
- Dynamic allocation: controlled by the programmer
- Persists until explicitly freed

**malloc / calloc / realloc / free**
```c
#include <stdlib.h>

int *ptr = (int *)malloc(sizeof(int) * 20);  // allocate 20 ints
// Always check for NULL!
if (ptr == NULL) { /* allocation failed */ }

int *z = (int *)calloc(20, sizeof(int));    // allocate AND zero-initialise

ptr = (int *)realloc(ptr, sizeof(int) * 40); // resize

free(ptr);   // always free when done
ptr = NULL;  // good practice after freeing
```

**Safety rules**
- NEVER free memory that was not malloc'd
- NEVER free the same memory twice (double-free)
- NEVER use memory after freeing it (use-after-free)
- NEVER dereference a pointer after freeing (dangling pointer)
- Always check malloc return value (returns NULL on failure)

**Memory leaks**
- Heap memory not freed → leak
- Use `valgrind` or `-fsanitize=address` to detect

**Global / Static variables**
- `static` local var: retains value across calls, not on stack
- `extern` var: defined in another file
- Global vars: initialised to zero automatically

---

## Week 4b — Linked Lists

### Big Picture
A linked list is a dynamic data structure made of nodes on the heap. Each node contains data and a pointer to the next node. More flexible than arrays for insertion/deletion.

### Key Concepts

**Node definition**
```c
struct list {
    char *data;
    struct list *next;  // self-referential struct
};
```

**Creating a node**
```c
struct list *newp = malloc(sizeof(struct list));
newp->data = "hello";
newp->next = NULL;
```

**Insert at front**
```c
struct list *insert_front(struct list *listp, struct list *newp) {
    newp->next = listp;
    return newp;
}
```

**Traversal**
```c
struct list *current = head;
while (current != NULL) {
    printf("%s\n", current->data);
    current = current->next;
}
```

**Freeing a linked list**
```c
while (head != NULL) {
    struct list *temp = head->next;
    free(head);
    head = temp;
}
```

**Linked list vs Array**

| Operation | Array | Linked List |
|-----------|-------|-------------|
| Access nth element | O(1) | O(n) |
| Insert at front | O(n) | O(1) |
| Insert at back | O(1) amortised | O(n) or O(1) w/ tail ptr |
| Delete from front | O(n) | O(1) |
| Memory | contiguous | scattered (heap) |

**Key exam pattern: double pointer**
```c
void insert(struct list **headp, struct list *newp) {
    newp->next = *headp;
    *headp = newp;     // modifies caller's pointer
}
```

---

## Week 5 — Function Pointers, Signals, Low-level I/O

### Big Picture
Function pointers let you store functions as variables (callbacks, dispatch tables). Signals are software interrupts from OS. Low-level I/O uses file descriptors directly instead of FILE*.

### Key Concepts

**Function Pointers**
```c
// Declare: return_type (*name)(param_types)
int (*fptr)(void) = foo;   // fptr points to function foo
fptr();                     // call through pointer

// As parameter (callback)
void do_sort(int *arr, int n, int (*cmp)(int, int));
```
- Used for: callbacks, plugin systems, `qsort`, signal handlers
- Target address stored in pointer → CPU does `call *%rdx`

**typedef with function pointer**
```c
typedef int (*compare_fn)(int, int);
compare_fn cmp = my_compare;
```

**Signals**
- A signal is a software interrupt sent to a process
- Process execution is interrupted; handler runs; execution resumes
- Common signals: `SIGINT` (Ctrl+C), `SIGTERM`, `SIGSEGV`, `SIGKILL`

**Simple signal handling**
```c
#include <signal.h>
void handler(int signum) { /* ... */ }
signal(SIGINT, handler);   // old API, simpler
```

**sigaction (preferred API)**
```c
struct sigaction sa;
sa.sa_handler = handler;
sigemptyset(&sa.sa_mask);
sa.sa_flags = 0;
sigaction(SIGINT, &sa, NULL);
```

**Low-level I/O (file descriptors)**
```c
#include <fcntl.h>
#include <unistd.h>

int fd = open("file.txt", O_RDONLY);     // returns file descriptor (int)
int fd = open("file.txt", O_WRONLY | O_CREAT, 0644);

ssize_t n = read(fd, buffer, count);    // returns bytes read, 0=EOF, -1=error
ssize_t n = write(fd, buffer, count);   // returns bytes written

close(fd);
```
- Standard FDs: `0`=stdin, `1`=stdout, `2`=stderr
- Error checking: check return value; `errno` is set on error

**errno**
```c
#include <errno.h>
int err = errno;   // capture immediately after failure
if (err != 0) { perror("error"); }
```

---

## Week 6 — Preprocessor, Linking, Data Types, Security

### Big Picture
The C preprocessor handles `#include` / `#define` / `#ifdef` before compilation. Linking combines object files into an executable. Data types have sizes and pitfalls; security issues arise from type misuse.

### Key Concepts

**Preprocessor**
```c
#include <stdio.h>     // system header
#include "myfile.h"    // local header

#define PI 3.14159     // constant macro (no type, no semicolon)
#define MAX(a,b) ((a)>(b)?(a):(b))  // function macro

#ifdef DEBUG
    printf("debug info\n");
#endif

#ifndef MYHEADER_H     // include guard pattern
#define MYHEADER_H
// ... header contents ...
#endif
```

**Macro pitfalls**
```c
// Danger: side effects evaluated multiple times
y = MAX(a++, b);
// expands to: ((a++)>(b)?(a++):(b))  — a++ happens twice!
// Always use parentheses around all macro parameters
```

**Compiling and Linking**
```
gcc -c util.c          → util.o (compile only)
gcc myprog.c util.o    → links and produces a.out
gcc myprog.c util.o -o myprog

gcc -DDEBUG prog.c     → defines DEBUG symbol for preprocessor
```

**Why headers, not code?**
- Including `.c` files would compile the same code twice → linker error (duplicate symbol)
- Headers declare but do not define (except inline or static)

**Conditional compilation**
```c
#ifdef EBUG            // typo in slide — normally DEBUG
#define DEBUG(m) printf("debug: %s in function %s at line %d in file %s\n", \
                         (m), __func__, __LINE__, __FILE__)
#else
#define DEBUG(m) do{}while(0)   // null statement — compiles to nothing
#endif
```

**Data types and sizes**

| Type | Typical Size | Range |
|------|-------------|-------|
| `char` | 1 byte | -128 to 127 (signed) / 0–255 (unsigned) |
| `short` | 2 bytes | ±32,767 |
| `int` | 4 bytes | ±2.1 billion |
| `long` | 4 or 8 bytes | platform-dependent |
| `float` | 4 bytes | ~7 sig. digits |
| `double` | 8 bytes | ~15 sig. digits |

**Floating point errors**
- Not all values exactly representable in binary
- Accumulated error in calculations
- Never test `float == float`; use `fabs(a - b) < epsilon`

**Security: integer overflow / underflow**
- Signed int overflow is undefined behaviour in C
- Unsigned int wraps around: `UINT_MAX + 1 == 0`

---

## Week 7 — Processes: fork and exec

### Big Picture
A process is an instance of a running program with its own virtual memory. `fork()` creates a child process (clone of parent). `exec()` replaces the current process image with a new program.

### Key Concepts

**Process and Memory**
- OS creates a virtual address space for each process
- Virtual memory mapped to physical memory on demand
- Each process has: code, stack, heap, global data segments

**fork()**
```c
#include <unistd.h>
pid_t pid = fork();

if (pid < 0)  { /* error */ }
if (pid == 0) { /* this is the CHILD */ }
if (pid > 0)  { /* this is the PARENT, pid = child's PID */ }
```
- Child is a nearly exact copy of parent
- Both resume from after the `fork()` call
- Child inherits open file descriptors
- Fork bomb: `while(1) fork();` — exhausts process table

**wait()**
```c
#include <sys/wait.h>
int status;
pid_t child = wait(&status);    // wait for ANY child
pid_t child = waitpid(pid, &status, 0);  // wait for specific child

WIFEXITED(status)   // true if child exited normally
WEXITSTATUS(status) // exit code if WIFEXITED
```
- Always `wait()` for children → prevents zombie processes

**exec() family**
```c
#include <unistd.h>
execl("/bin/ls", "ls", "-l", NULL);
execvp("ls", argv);    // most common — searches PATH
```
- `exec` replaces the process's memory with a new program
- If `exec` succeeds → it does NOT return (process is replaced)
- If `exec` returns → it failed (check errno)

**fork + exec pattern**
```c
pid_t pid = fork();
if (pid == 0) {
    execvp(cmd, args);   // child runs new program
    perror("exec");      // only reached if exec fails
    exit(1);
} else {
    wait(NULL);          // parent waits
}
```

**Zombie / Orphan processes**
- Zombie: child has exited but parent hasn't called `wait()` yet
- Orphan: parent has exited before child — adopted by init (PID 1)

**Security**
- Fork bomb: continuously forking can exhaust resources
- Shared file descriptors between parent and child can cause unexpected data sharing

---

## Week 8 — IPC (Inter-Process Communication)

### Big Picture
Processes have separate address spaces — they need special mechanisms to communicate. Options: signals (interrupts), pipes (simple stream), files (persistent, large data), shared memory (fast but needs sync).

### Key Concepts

**IPC mechanisms summary**

| Mechanism | Use case |
|-----------|----------|
| Signals | Interrupt a process (SIGINT, SIGUSR1, etc.) |
| Pipes | Simple byte stream between parent/child |
| Files | Larger or persistent data sharing |
| Shared memory | High-speed large data between processes |

**Pipes**
```c
#include <unistd.h>
int pipefd[2];
pipe(pipefd);
// pipefd[0] = read end, pipefd[1] = write end

// In child:  close(pipefd[1]); read from pipefd[0]
// In parent: close(pipefd[0]); write to pipefd[1]
```
- Unidirectional; data flows one way
- Read blocks until data available; write blocks until space available

**Low-level file I/O for IPC**
```c
int fd = open("shared.txt", O_WRONLY);
write(fd, data, strlen(data));
close(fd);
```

**Blocking I/O**
- `read()` blocks until data arrives
- `write()` blocks until buffer has space
- Processes cannot do other work while blocked → can miss events

**Non-blocking / multiplexing**
```c
// select() — wait on multiple file descriptors
fd_set readfds;
FD_ZERO(&readfds);
FD_SET(fd1, &readfds);
FD_SET(fd2, &readfds);
select(max_fd+1, &readfds, NULL, NULL, &timeout);
if (FD_ISSET(fd1, &readfds)) { /* fd1 is ready */ }
```
- `epoll()` — more efficient version for many FDs

**Shared memory**
- Processes map the same physical memory into their address space
- Fast (no kernel copy), but requires synchronisation (e.g., semaphores)

**IPC decision guide:**
- Need to interrupt? → Signal
- Simple byte stream? → Pipe
- Large data / persistence? → File + select/epoll
- High-speed large data? → Shared memory

---

## Week 9 — Parallelism and Threads

### Big Picture
Threads are lightweight processes that share the same address space. They allow true parallel execution on multi-core CPUs, but require careful synchronisation.

### Key Concepts

**Parallelism concepts**
- Task parallelism: different threads do different tasks simultaneously
- Data parallelism: same task applied to different chunks of data simultaneously

**Why threads instead of processes?**
- Threads share memory → no IPC overhead
- Cheaper to create/switch than processes
- BUT: shared memory → race conditions

**POSIX Threads (pthreads)**
```c
#include <pthread.h>

// Thread function signature
void *thread_func(void *arg) {
    // ... do work ...
    return NULL;
}

// Create
pthread_t tid;
pthread_create(&tid, NULL, thread_func, arg);
//              ^id   ^attr  ^function   ^argument

// Wait for thread to finish
pthread_join(tid, NULL);
//           ^id  ^return_status (can be NULL)
```
- Compile with: `gcc ... -lpthread`

**Execution indeterminism**
- No guarantee on ordering of statements between threads
- Cannot assume Thread A runs before Thread B

**pthread_join()**
- Blocks caller until specified thread terminates
- After join, thread ID is no longer valid
- Cannot join the same thread twice

**Shared data between threads**
```c
// Global var shared between threads
char *message = "Chocolate microscopes?";
int mindex = 0;
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

// In thread function:
pthread_mutex_lock(&lock);
    // critical section — access mindex safely
pthread_mutex_unlock(&lock);
```

---

## Week 10 — Thread Synchronisation

### Big Picture
Multiple threads accessing shared data can cause race conditions. Mutex (mutual exclusion) locks ensure only one thread accesses a critical section at a time. Semaphores extend this to counting.

### Key Concepts

**Race condition**
- Two threads read/modify shared variable without synchronisation
- Result depends on execution order → unpredictable, wrong

**Critical section**
- Code that accesses shared data and must not be interleaved
- Solution: wrap in mutex lock/unlock

**Mutex (pthread_mutex_t)**
```c
// Static initialisation
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

// Dynamic initialisation
pthread_mutex_t *mylock = malloc(sizeof(pthread_mutex_t));
pthread_mutex_init(mylock, NULL);   // NULL = default attributes
// ...
pthread_mutex_destroy(mylock);
free(mylock);

// Locking
pthread_mutex_lock(&lock);
    // critical section
pthread_mutex_unlock(&lock);
```
- Only one thread holds the lock at a time; others block waiting

**Semaphore**
```c
#include <semaphore.h>
sem_t sem;
sem_init(&sem, 0, N);  // 0=thread semaphore, N=initial count

sem_wait(&sem);   // decrement; blocks if 0
sem_post(&sem);   // increment; wakes a blocked thread
sem_destroy(&sem);
```
- Mutex = binary semaphore (count 0 or 1)
- Semaphore = generalised counter (up to N)

**Deadlock**
- Two threads each hold a lock the other needs → both block forever
- Prevention: always acquire multiple locks in the same order (lock hierarchy)
```
Thread 1: acquire A → acquire B
Thread 2: acquire A → acquire B   // same order → no deadlock
// (NOT: T2 acquires B then A)
```

**Livelock**
- Threads keep changing state in response to each other but make no progress
- Like two people in a corridor each stepping aside simultaneously

**Starvation**
- A thread is perpetually denied access to a resource
- Can occur when one thread always acquires a lock before another

**Locking granularity**
- Coarse-grained lock: protects large data → simple but high contention
- Fine-grained lock: one lock per small piece → lower contention but complex

**Amdahl's Law**
- Speedup limited by the serial (non-parallelisable) fraction
- If 20% of code is serial → max speedup = 5× regardless of cores

---

## Week 11 — Further Thread Synchronisation

### Big Picture
Beyond mutexes: barriers synchronise a group of threads at a checkpoint. Condition variables let threads sleep waiting for a state change, avoiding busy-waiting.

### Key Concepts

**Barriers**
```c
pthread_barrier_t barrier;
pthread_barrier_init(&barrier, NULL, N);  // N = number of threads

// In each thread:
pthread_barrier_wait(&barrier);  // blocks until all N threads reach this point

pthread_barrier_destroy(&barrier);
```
- Useful for phases: all threads finish phase 1 before any start phase 2

**Condition Variables**
```c
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

// Thread waiting for a condition:
pthread_mutex_lock(&mutex);
while (!condition_is_true) {           // ALWAYS use while, not if
    pthread_cond_wait(&cond, &mutex);  // atomically: releases mutex + sleeps
}                                      // upon return: mutex is locked again
// ... do work now that condition is true ...
pthread_mutex_unlock(&mutex);

// Thread signalling the condition:
pthread_mutex_lock(&mutex);
condition_is_true = 1;
pthread_cond_signal(&cond);    // wake ONE waiting thread
// OR
pthread_cond_broadcast(&cond); // wake ALL waiting threads
pthread_mutex_unlock(&mutex);
```

**Spurious wakeups**
- `pthread_cond_wait` can return without being signalled
- Always use `while`, not `if`, to recheck the condition

**Thundering herd**
- `broadcast` wakes all blocked threads, but only one gets the resource
- Others go back to sleep → wasted context switches
- Use `signal` (not `broadcast`) when only one thread can proceed

**`pthread_cond_wait` details**
- Must be called with mutex LOCKED
- Atomically releases mutex and blocks → no window for missed signals
- On return, mutex is locked again
- `pthread_cond_signal` must be called with mutex locked

---

## Week 13 — Revision Summary

### Essentials for the Pass
Per the revision lecture, the exam focuses on:
1. C aspects that are conceptually like Java (control flow, functions, types)
2. C pointers basics
3. C pointers with arrays and strings
4. C pointers with lists and other data structures
5. C memory model (stack, heap, malloc/free)
6. Applying synchronisation primitives (mutex, condition variables, semaphores)

### Exam Technique Tips
- Aim to write clear, compilable code
- Write comments to show your thinking
- Draw memory diagrams — they really help
- Map out solution at high level first (pseudo-code), then fill in

### Key postconditions the course expects
- Understand sequential performance limitations and parallel programming concepts
- task-parallelism, data-parallelism, synchronisation, deadlock, livelock, starvation, locking hierarchy, scheduling, Amdahl's Law
- Ability to apply a parallel solution using pthreads
- Ability to measure and identify performance bottlenecks

---

## Master Quick-Reference: All Key Functions

| Function | Header | Purpose |
|----------|--------|---------|
| `malloc(size)` | `<stdlib.h>` | Allocate heap memory |
| `calloc(n, size)` | `<stdlib.h>` | Allocate + zero-initialise |
| `realloc(ptr, size)` | `<stdlib.h>` | Resize heap block |
| `free(ptr)` | `<stdlib.h>` | Release heap memory |
| `fork()` | `<unistd.h>` | Create child process |
| `wait(&status)` | `<sys/wait.h>` | Wait for any child |
| `waitpid(pid,&st,0)` | `<sys/wait.h>` | Wait for specific child |
| `execvp(cmd, argv)` | `<unistd.h>` | Replace process image |
| `open(path, flags)` | `<fcntl.h>` | Open file → file descriptor |
| `read(fd, buf, n)` | `<unistd.h>` | Read from file descriptor |
| `write(fd, buf, n)` | `<unistd.h>` | Write to file descriptor |
| `close(fd)` | `<unistd.h>` | Close file descriptor |
| `pipe(pipefd[2])` | `<unistd.h>` | Create pipe |
| `signal(sig, handler)` | `<signal.h>` | Register signal handler |
| `sigaction(sig,&sa,NULL)` | `<signal.h>` | Advanced signal handling |
| `pthread_create(...)` | `<pthread.h>` | Create thread |
| `pthread_join(tid,NULL)` | `<pthread.h>` | Wait for thread |
| `pthread_mutex_lock(&m)` | `<pthread.h>` | Acquire mutex |
| `pthread_mutex_unlock(&m)` | `<pthread.h>` | Release mutex |
| `pthread_cond_wait(&c,&m)` | `<pthread.h>` | Sleep on condition |
| `pthread_cond_signal(&c)` | `<pthread.h>` | Wake one waiting thread |
| `sem_wait(&s)` | `<semaphore.h>` | Decrement semaphore |
| `sem_post(&s)` | `<semaphore.h>` | Increment semaphore |

---

## Common Exam Traps

1. **Forgetting NULL-terminate strings** — `str[n] = '\0';`
2. **Freeing stack memory** — `free(&local_var)` = crash
3. **Double-free** — free same pointer twice = undefined behaviour
4. **Not checking malloc return** — malloc can return NULL
5. **Using `if` instead of `while` with condition variables** — spurious wakeups!
6. **Forgetting `wait()` for children** — zombie processes
7. **exec() after fork() not checking return** — if exec fails and you don't handle it, child continues
8. **Race condition without mutex** — two threads writing shared var
9. **Deadlock from inconsistent lock ordering** — always acquire in same order
10. **Pointer to local variable returned from function** — local is freed when function returns → dangling pointer
