# COMP2017 Systems Programming — Practice Exam
**Same difficulty and format as mock exam + 2024 exam**
Total: 100 marks | 2 hours writing + 10 min reading

---

---

## Question 1 — Simple C (10 marks)

### Q1(a) — 5 marks

Write a C program that reads lines from a file called `scores.txt`. Each line contains a player name and an integer score separated by a space. The program should print the name and score of the player with the highest score.

**Sample Input (scores.txt):**
```
Alice 88
Bob 95
Charlie 72
Diana 95
```

**Sample Output:**
```
Bob 95
```
*(If there are ties, print the first one encountered.)*

You may assume the file exists, contains at least one entry, and names are at most 31 characters.

---

### Q1(b) — 5 marks

Describe how you would modify your program from Q1(a) so that it instead prints the **top 3** scores in descending order. You do not need to write complete code — explain the data structure changes and the algorithm clearly.

---

---

## Question 2 — Data Structures (10 marks)

Given the following struct definition:

```c
struct packet {
    uint8_t  flags;
    uint32_t src_ip;
    union {
        struct {
            uint16_t port;
            uint16_t checksum;
        } tcp;
        struct {
            uint32_t msg_id;
        } udp;
    } header;
    char tag;
};

struct packet packets[2];
```

Assume:
- `uint8_t` is 1 byte, `uint16_t` is 2 bytes, `uint32_t` is 4 bytes
- `char` is 1 byte
- Memory is aligned to each field's natural size (i.e., a 4-byte field starts at an address divisible by 4)

Determine the **memory offset in bytes** of each field from the start of one `struct packet`:

| Field | Offset (bytes) |
|-------|---------------|
| `packets[0].flags` | |
| `packets[0].src_ip` | |
| `packets[0].header.tcp.port` | |
| `packets[0].header.tcp.checksum` | |
| `packets[0].header.udp.msg_id` | |
| `packets[0].tag` | |
| `packets[1].flags` | |

---

---

## Question 3 — Security & Linking (10 marks)

### Q3(a) — 5 marks

Consider the following C macro definition:

```c
#define SQUARE(x) x * x
```

(a) What is the output of the following code, and why?

```c
int a = 3;
int result = SQUARE(a + 1);
printf("%d\n", result);
```

(b) Write a corrected version of the `SQUARE` macro that behaves correctly for all inputs including expressions with side effects like `a++`.

---

### Q3(b) — 5 marks

A developer stores sensitive user passwords in a global `char *password` variable, reads the password using `scanf("%s", buffer)` and then does `password = buffer`.

Identify **three distinct security or memory safety problems** with this approach and briefly explain each.

---

---

## Question 4 — Concurrency (20 marks)

A university library system allows students to borrow and return books. Books are stored in a shared array. Multiple student threads call `borrow_book()` and `return_book()` concurrently.

```c
#define MAX_BOOKS 100

typedef struct {
    int  book_id;
    int  available;   // 1 = on shelf, 0 = borrowed
    char title[64];
} book_t;

book_t library[MAX_BOOKS];
int    num_books = 0;

// You may NOT modify these functions
int find_book(int book_id) {
    for (int i = 0; i < num_books; i++) {
        if (library[i].book_id == book_id)
            return i;
    }
    return -1;
}

// You MAY modify the code below
int borrow_book(int book_id) {
    int idx = find_book(book_id);
    if (idx == -1)        return -1;  // not found
    if (!library[idx].available) return 0;  // already borrowed
    library[idx].available = 0;
    return 1;  // success
}

int return_book(int book_id) {
    int idx = find_book(book_id);
    if (idx == -1) return -1;
    library[idx].available = 1;
    return 1;
}

void library_init(void) {
    // initialise library data here
}
```

**Task:** Rewrite `borrow_book`, `return_book`, and `library_init`, adding all synchronisation necessary to avoid all race conditions. You may add global variables. Pizzas should be prepared in parallel where possible — similarly, **independent books must be borrowable/returnable in parallel** (i.e., do not use a single global lock over all books).

You may add additional global variables and perform additional initialisation in `library_init`.

---

---

## Question 5 — Programming (20 marks)

### Q5(a) — 15 marks

The code below implements a function that counts the frequency of each byte value (0–255) in a buffer:

```c
void count_bytes(const unsigned char *buf, size_t len, int freq[256]) {
    for (size_t i = 0; i < len; i++) {
        freq[buf[i]]++;
    }
}
```

Write a **parallel implementation** of the above using **8 pthreads**. Each thread should process a roughly equal portion of `buf`. The final `freq` array must be correct (all counts summed across threads). There are no requirements on the order of processing.

The function signature must remain:
```c
void count_bytes(const unsigned char *buf, size_t len, int freq[256]);
```

---

### Q5(b) — 5 marks

What are the advantages and disadvantages of using **processes (fork)** instead of **pthreads** for the parallelism in Q5(a)? Discuss differences in address space, communication overhead, and fault isolation.

---

---

## Question 6 — Programming (30 marks)

You are to implement a simple **producer-consumer pipeline** for a log processing system.

The system has:
- **1 reader process** that reads log lines from `stdin` (one line per read) and writes them into a shared buffer
- **N worker threads** (N passed as first command-line argument) that each take a log line, process it (convert to uppercase), and write the result to `stdout`

The following skeleton is provided:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <pthread.h>
#include <ctype.h>

#define MAX_LINE   256
#define BUFFER_SIZE 16

typedef struct {
    char lines[BUFFER_SIZE][MAX_LINE];
    int  head;
    int  tail;
    int  count;
    int  done;    // set to 1 when no more input
    // add synchronisation fields here
} queue_t;

queue_t queue;

// Called once at startup
void queue_init(queue_t *q) {
    q->head  = 0;
    q->tail  = 0;
    q->count = 0;
    q->done  = 0;
    // initialise synchronisation here
}

// Add a line to the queue (blocks if full)
void enqueue(queue_t *q, const char *line) {
    // your code here
}

// Remove a line from the queue (blocks if empty and not done)
// Returns 1 if a line was retrieved, 0 if done and empty
int dequeue(queue_t *q, char *line) {
    // your code here
    return 0;
}

// Worker thread: dequeues lines, uppercases them, prints to stdout
void *worker(void *arg) {
    // your code here
    return NULL;
}

int main(int argc, char **argv) {
    if (argc < 2) { fprintf(stderr, "Usage: %s <num_workers>\n", argv[0]); return 1; }
    int n = atoi(argv[1]);

    queue_init(&queue);

    // TODO: create n worker threads

    // Reader: read lines from stdin, enqueue each one
    char line[MAX_LINE];
    while (fgets(line, MAX_LINE, stdin) != NULL) {
        enqueue(&queue, line);
    }

    // Signal workers that input is done
    // TODO

    // TODO: join all worker threads

    return 0;
}
```

**Task:** Complete the implementation. Specifically:
- Implement `queue_init`, `enqueue`, `dequeue`, `worker`, and the missing parts of `main`
- `enqueue` must block if the buffer is full
- `dequeue` must block if the buffer is empty and `done == 0`; return 0 if empty and `done == 1`
- Workers must process lines in parallel where possible
- Use condition variables (not busy-waiting)
- Output order does not need to be deterministic

---
---
---

# SOLUTIONS

---

## Q1(a) Solution

```c
#include <stdio.h>
#include <string.h>

int main(void) {
    FILE *fp = fopen("scores.txt", "r");
    if (fp == NULL) {
        perror("fopen");
        return 1;
    }

    char best_name[32];
    int  best_score = -1;
    char name[32];
    int  score;

    while (fscanf(fp, "%31s %d", name, &score) == 2) {
        if (score > best_score) {
            best_score = score;
            strncpy(best_name, name, sizeof(best_name));
        }
    }
    fclose(fp);

    if (best_score != -1)
        printf("%s %d\n", best_name, best_score);

    return 0;
}
```

**Key marks:**
- `fopen` / `fclose` with error check
- `fscanf` loop checking return value == 2
- Correct tracking of maximum

---

## Q1(b) Solution

Keep a sorted array of 3 `{name, score}` structs (the "top 3 list").

**Data structure change:**
```c
typedef struct { char name[32]; int score; } entry_t;
entry_t top3[3];
int count = 0;   // how many slots filled so far (0..3)
```

**Algorithm:**
For each line read:
1. If `count < 3`: insert and increment `count`, then sort descending by score.
2. Else if the new score > `top3[2].score` (the smallest of the top 3): replace `top3[2]` with the new entry, then re-sort.
3. After all lines: print `top3[0]`, `top3[1]`, `top3[2]`.

Sorting 3 elements is O(1) (at most 3 comparisons via insertion sort or `qsort`).

---

## Q2 Solution

`struct packet` layout with alignment:

```
offset 0:  flags    (uint8_t, 1 byte)
offset 1:  [3 bytes padding — src_ip needs 4-byte alignment]
offset 4:  src_ip   (uint32_t, 4 bytes)
offset 8:  header   (union — size = max(sizeof tcp, sizeof udp) = 4 bytes)
               tcp.port      → offset 8  (uint16_t)
               tcp.checksum  → offset 10 (uint16_t)
               udp.msg_id    → offset 8  (uint32_t) — union shares same address
offset 12: tag      (char, 1 byte)
offset 13: [3 bytes padding — struct size rounds up to multiple of 4]
sizeof(struct packet) = 16
```

| Field | Offset |
|-------|--------|
| `packets[0].flags` | **0** |
| `packets[0].src_ip` | **4** |
| `packets[0].header.tcp.port` | **8** |
| `packets[0].header.tcp.checksum` | **10** |
| `packets[0].header.udp.msg_id` | **8** |
| `packets[0].tag` | **12** |
| `packets[1].flags` | **16** |

---

## Q3(a) Solution

**Output: 7**

The macro `SQUARE(a + 1)` expands textually to:
```c
a + 1 * a + 1
```
which evaluates as `a + (1 * a) + 1 = 3 + 3 + 1 = 7`, not `(3+1)² = 16`.

**Corrected macro:**
```c
#define SQUARE(x) ((x) * (x))
```
All parameters must be wrapped in parentheses to force correct precedence. However, this still evaluates `x` twice, which is a problem for `SQUARE(a++)` — `a` would be incremented twice. The truly safe solution is an inline function:
```c
static inline int square(int x) { return x * x; }
```

---

## Q3(b) Solution

Three distinct problems:

1. **Buffer overflow via `scanf("%s")`**: `scanf("%s", buffer)` reads until whitespace with no length limit. If the password is longer than `buffer`'s size, adjacent stack memory is overwritten → stack smashing attack. Fix: use `scanf("%255s", buffer)` or `fgets`.

2. **Dangling pointer**: `password = buffer` stores a pointer to a local variable (stack frame). When the function returns, `buffer` is gone, and `password` points to freed stack memory. Any later use of `password` is undefined behaviour. Fix: `password = strdup(buffer)` (allocates heap copy).

3. **Storing plaintext password in a global variable**: A global `char *password` is accessible to any code in the same address space. If the process's memory is read (e.g., via a memory dump, another vulnerable function, or `/proc/self/mem`), the plaintext password is exposed. Passwords should be hashed, not stored in plaintext.

---

## Q4 Solution

The key insight: use **one mutex per book** so independent books can be borrowed/returned in parallel.

```c
#include <pthread.h>
#include <stdlib.h>

#define MAX_BOOKS 100

typedef struct {
    int  book_id;
    int  available;
    char title[64];
    pthread_mutex_t lock;   // per-book lock
} book_t;

book_t library[MAX_BOOKS];
int    num_books = 0;

// Cannot modify:
int find_book(int book_id) {
    for (int i = 0; i < num_books; i++)
        if (library[i].book_id == book_id)
            return i;
    return -1;
}

void library_init(void) {
    for (int i = 0; i < MAX_BOOKS; i++)
        pthread_mutex_init(&library[i].lock, NULL);
}

int borrow_book(int book_id) {
    int idx = find_book(book_id);
    if (idx == -1) return -1;

    pthread_mutex_lock(&library[idx].lock);
    int result = 0;
    if (library[idx].available) {
        library[idx].available = 0;
        result = 1;
    }
    pthread_mutex_unlock(&library[idx].lock);
    return result;
}

int return_book(int book_id) {
    int idx = find_book(book_id);
    if (idx == -1) return -1;

    pthread_mutex_lock(&library[idx].lock);
    library[idx].available = 1;
    pthread_mutex_unlock(&library[idx].lock);
    return 1;
}
```

**Why this is correct:**
- `find_book` is read-only and safe to call without a lock (it only reads `book_id` which is set at init time and never changes)
- The check-and-set of `available` is inside the per-book lock → no race condition on the same book
- Two threads borrowing *different* books never contend → parallel

---

## Q5(a) Solution

```c
#include <pthread.h>
#include <stdlib.h>
#include <string.h>

#define NUM_THREADS 8

typedef struct {
    const unsigned char *buf;
    size_t start;
    size_t end;
    int    local_freq[256];
} thread_arg_t;

static void *count_chunk(void *arg) {
    thread_arg_t *t = (thread_arg_t *)arg;
    memset(t->local_freq, 0, sizeof(t->local_freq));
    for (size_t i = t->start; i < t->end; i++)
        t->local_freq[t->buf[i]]++;
    return NULL;
}

void count_bytes(const unsigned char *buf, size_t len, int freq[256]) {
    pthread_t      threads[NUM_THREADS];
    thread_arg_t   args[NUM_THREADS];

    size_t chunk = len / NUM_THREADS;

    for (int i = 0; i < NUM_THREADS; i++) {
        args[i].buf   = buf;
        args[i].start = i * chunk;
        args[i].end   = (i == NUM_THREADS - 1) ? len : (i + 1) * chunk;
        pthread_create(&threads[i], NULL, count_chunk, &args[i]);
    }

    memset(freq, 0, 256 * sizeof(int));

    for (int i = 0; i < NUM_THREADS; i++) {
        pthread_join(threads[i], NULL);
        for (int b = 0; b < 256; b++)
            freq[b] += args[i].local_freq[b];
    }
}
```

**Why no mutex needed:** Each thread writes into its *own* `local_freq[256]`. The merge loop in the main thread runs only after all threads are joined, so there is no concurrent access to `freq`. This is the "local accumulator" pattern — cheaper than locking a shared array on every increment.

---

## Q5(b) Solution

**Advantages of fork/processes over pthreads here:**

- **Fault isolation**: if one worker process crashes (segfault), the others keep running. With threads, one segfault kills the whole process.
- **No shared memory corruption**: processes have separate address spaces, so a buggy worker cannot corrupt the main `freq` array.

**Disadvantages of fork/processes here:**

- **Communication overhead**: processes do not share memory. To merge `local_freq` arrays back, you need IPC (pipes or shared memory), adding significant complexity and copying cost.
- **Higher creation cost**: `fork()` is slower than `pthread_create()` — it copies the process's page tables.
- **More complex result collection**: you'd need to write 256 ints × 4 bytes = 1024 bytes over a pipe per worker, plus read and merge them, vs. threads where the merge is a simple local array iteration after `join`.

**For this specific problem**, pthreads are clearly better: the shared-read of `buf` is safe (read-only), and the local-accumulator approach avoids all locking overhead entirely.

---

## Q6 Solution

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <pthread.h>
#include <ctype.h>

#define MAX_LINE    256
#define BUFFER_SIZE 16

typedef struct {
    char lines[BUFFER_SIZE][MAX_LINE];
    int  head;
    int  tail;
    int  count;
    int  done;
    pthread_mutex_t mutex;
    pthread_cond_t  not_full;
    pthread_cond_t  not_empty;
} queue_t;

queue_t queue;

void queue_init(queue_t *q) {
    q->head  = 0;
    q->tail  = 0;
    q->count = 0;
    q->done  = 0;
    pthread_mutex_init(&q->mutex, NULL);
    pthread_cond_init(&q->not_full,  NULL);
    pthread_cond_init(&q->not_empty, NULL);
}

void enqueue(queue_t *q, const char *line) {
    pthread_mutex_lock(&q->mutex);
    while (q->count == BUFFER_SIZE)
        pthread_cond_wait(&q->not_full, &q->mutex);

    strncpy(q->lines[q->tail], line, MAX_LINE);
    q->tail  = (q->tail + 1) % BUFFER_SIZE;
    q->count++;

    pthread_cond_signal(&q->not_empty);
    pthread_mutex_unlock(&q->mutex);
}

int dequeue(queue_t *q, char *line) {
    pthread_mutex_lock(&q->mutex);

    // Wait while empty AND not done
    while (q->count == 0 && !q->done)
        pthread_cond_wait(&q->not_empty, &q->mutex);

    if (q->count == 0) {
        // done == 1 and empty → no more work
        pthread_mutex_unlock(&q->mutex);
        return 0;
    }

    strncpy(line, q->lines[q->head], MAX_LINE);
    q->head  = (q->head + 1) % BUFFER_SIZE;
    q->count--;

    pthread_cond_signal(&q->not_full);
    pthread_mutex_unlock(&q->mutex);
    return 1;
}

void *worker(void *arg) {
    (void)arg;
    char line[MAX_LINE];
    char upper[MAX_LINE];

    while (dequeue(&queue, line)) {
        // Convert to uppercase
        size_t len = strlen(line);
        for (size_t i = 0; i < len; i++)
            upper[i] = (char)toupper((unsigned char)line[i]);
        upper[len] = '\0';
        printf("%s", upper);
    }
    return NULL;
}

int main(int argc, char **argv) {
    if (argc < 2) {
        fprintf(stderr, "Usage: %s <num_workers>\n", argv[0]);
        return 1;
    }
    int n = atoi(argv[1]);

    queue_init(&queue);

    pthread_t *threads = malloc(n * sizeof(pthread_t));
    for (int i = 0; i < n; i++)
        pthread_create(&threads[i], NULL, worker, NULL);

    char line[MAX_LINE];
    while (fgets(line, MAX_LINE, stdin) != NULL)
        enqueue(&queue, line);

    // Signal done and wake all sleeping workers
    pthread_mutex_lock(&queue.mutex);
    queue.done = 1;
    pthread_cond_broadcast(&queue.not_empty);
    pthread_mutex_unlock(&queue.mutex);

    for (int i = 0; i < n; i++)
        pthread_join(threads[i], NULL);

    free(threads);
    return 0;
}
```

**Key design decisions explained:**

1. **Two condition variables** (`not_full`, `not_empty`): `enqueue` waits on `not_full`; `dequeue` waits on `not_empty`. This avoids waking threads that cannot proceed.

2. **`done` flag inside the mutex**: setting `done = 1` and broadcasting must be atomic with respect to the queue state. Workers check `done` only after acquiring the mutex, so they cannot miss the signal.

3. **`pthread_cond_broadcast` when done**: all workers are waiting on `not_empty`. Using `signal` would only wake one — the others would sleep forever. `broadcast` wakes all, each checks the condition in a `while` loop, sees `count==0 && done==1`, and exits.

4. **`while` not `if` for condition wait**: protects against spurious wakeups.

5. **Circular buffer** with head/tail indices: avoids shifting elements; O(1) enqueue and dequeue.
