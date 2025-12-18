# Concurrency

## Critical Sections

### Definition

The code that accesses a shared resource.

**Read access:** If all threads only read from shared memory, then it is not a critical section.

**Write access:** If there is at least one write, then it is a critical section.

## Lock Design

### Solution Requirements

1. No two threads may simultaneously be in their critical sections.
2. Threads outside of critical sections should have no impact.
3. No assumptions should be made about number of cores, speed of cores, or scheduler choices.

### Lock Types Comparison

| Lock Type | Overhead | CPU Waste | Best For | Worst Case |
|-----------|----------|-----------|----------|-----------|
| Spinlock | Very low (~10-50 cycles) | High (busy-waiting) | Very short critical sections (<1ms) | Long held locks waste CPU |
| Ticketlock | Low (fair access) | High | Many competing threads, fairness needed | Long held locks, low contention |
| Yielding Lock | Medium (syscall overhead) | Low | Moderate contention, unknown lock duration | Frequent yields if lock held briefly |
| Queueing Lock | Medium-high (syscalls, queue management) | Very low | High contention, longer lock duration | Rarely held locks (context switch overhead) |

### Real-World Lock Implementations

- **Linux:** Uses **Futex (Fast Userspace Mutex)**, which is a hybrid approach. Threads first try to acquire a lock with atomic operations (similar to spinlock). If the lock is already held, the kernel is called to put the thread to sleep and manage a queue. This combines low overhead for uncontended locks with fairness for contended locks.

- **Windows:** Uses **Critical Sections** and **Slim Reader/Writer (SRW) Locks**. Critical sections are similar to futexes—userspace operations with kernel involvement only when blocking is needed.

- **macOS/BSD:** Uses **OSSpinLock** (spinlock) and **os_lock** (efficient userspace lock). Modern versions prefer futex-like approaches for better performance.

- **Java/Python:** Standard `Mutex` or `Lock` typically use OS-provided primitives (futex on Linux). Languages often provide higher-level abstractions built on these primitives.

- **Pthreads (POSIX):** `pthread_mutex_t` is implementation-defined but commonly uses futex on Linux, making it efficient for both uncontended and heavily-contended scenarios.

### 1. Spinlock

A spinlock is a lock where the thread simply waits in a loop ("spins") repeatedly checking if the lock is available.

- **Pros:** Very fast for short tasks because it avoids the overhead of putting a thread to sleep and waking it up (context switching).

- **Cons:** Wastes CPU cycles if the lock is held for a long time.

- **Implementation:** In C++, we use `std::atomic_flag` which is guaranteed to be lock-free.

```cpp
typedef struct {
    int flag; // 0 indicates that mutex is available, 1 that it is held
} lock_t;

void mutex_init(lock_t* mutex) {
    mutex->flag = 0; // lock starts available
}

void mutex_acquire(lock_t* mutex) {
    // atomic_exchange(destptr, newval): Write a new value to memory, and return the old one
    while (atomic_exchange(&(mutex->flag), 1) == 1); // spin-wait until available
}

void mutex_release(lock_t* mutex) {
    atomic_store(&(mutex->flag), 0); // make lock available
}
```

### 2. Ticketlock

Similar Pros and Cons with Spinlock.

```cpp
typedef struct {
    unsigned int ticket; // current available ticket
    unsigned int turn; // which ticket gets to proceed
} lock_t;

void mutex_init(lock_t* mutex) {
    mutex->ticket = 0;
    mutex->turn = 0;
}

void mutex_lock(lock_t* mutex) {
    // increment the value, and return the old value
    int myturn = atomic_fetch_and_add(&(mutex->ticket), 1); // take a ticket
    while (mutex->turn != myturn); // spin-wait until available
}

void mutex_unlock(lock_t* mutex) {
    atomic_fetch_and_add(&(mutex->turn), 1); // next turn
}
```

### 3. Yielding Lock

This approach is an optimization of a spinlock that addresses the problem of busy-waiting.

**Mechanism:** When a thread attempts to acquire a lock and finds it is already held, instead of continuously "spinning" (repeatedly checking the lock state), the thread calls the `sched_yield()` syscall.

- The `sched_yield()` syscall unschedules the current thread, giving up its CPU timeslice.

- This allows another thread—potentially the one holding the lock—to run and release the lock.

**Benefit (Pro):** It reduces busy-waiting.

**Drawback (Con):** It still results in "a lot of unnecessary context switches". This overhead is more expensive than spinning if the lock is held for a very short time.

```cpp
void mutex_lock(lock_t* mutex) {
    int myturn = atomic_fetch_and_add(&(mutex->ticket), 1); // take a ticket
    while (mutex->turn != myturn) {
        sched_yield(); // not ready yet
    }
}
```

### 4. Queueing Lock

- **Mechanism:**
  
  - **Acquire:** If a thread fails to acquire the lock, it adds its ID to a waiting thread queue and put itself to sleep.
  
  - **Release:** The thread releasing the lock dequeues the next waiting thread ID and calls a function to unblock that thread.
  
  - The Linux **Futex (Fast Userspace Mutex)** is a common example, where the queue is managed in the kernel, and system calls are only made when a thread actually needs to wait or wake up.

- **Benefit (Pro):** These sophisticated locks are generally more fair and do not waste processor time on "busy waiting".

- **Drawback (Con):** They can have unnecessary context-switch overhead if the lock is only briefly and rarely held.

## Synchronization

### 1. Condition Variables (`std::condition_variable`)

Condition variables are used not just to protect data, but to wait for a specific *state* change (e.g., "Wait until the queue is not empty"). They allow threads to sleep until notified.

**Key Pattern:**

1. Acquire a lock.

2. Check a condition.

3. If condition is false, `wait()` (this atomically unlocks the mutex and puts the thread to sleep).

4. When notified, the thread wakes up, re-locks the mutex, and checks the condition again.

```cpp
void thr_exit() {
    Pthread_mutex_lock(&m);
    done = 1;
    Pthread_cond_signal(&c);
    Pthread_mutex_unlock(&m);
}

void thr_join() {
    Pthread_mutex_lock(&m);
    while (done == 0)
        Pthread_cond_wait(&c, &m);
    Pthread_mutex_unlock(&m);
}
```

**Producer-Consumer Example with Bounded Buffer:**

```cpp
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
pthread_cond_t cond_full = PTHREAD_COND_INITIALIZER;
pthread_cond_t cond_empty = PTHREAD_COND_INITIALIZER;

int buffer[10];
int count = 0;  // Number of items in buffer
int in = 0;     // Position to insert
const int BUFFER_SIZE = 10;

void producer(int item) {
    pthread_mutex_lock(&lock);

    // Wait if buffer is full
    while (count == BUFFER_SIZE)
        pthread_cond_wait(&cond_empty, &lock);

    // Insert item
    buffer[in] = item;
    in = (in + 1) % BUFFER_SIZE;
    count++;

    // Signal that buffer has an item
    pthread_cond_signal(&cond_full);

    pthread_mutex_unlock(&lock);
}

int consumer() {
    pthread_mutex_lock(&lock);

    // Wait if buffer is empty
    while (count == 0)
        pthread_cond_wait(&cond_full, &lock);

    // Extract item
    int item = buffer[(in - count) % BUFFER_SIZE];
    count--;

    // Signal that buffer has space
    pthread_cond_signal(&cond_empty);

    pthread_mutex_unlock(&lock);
    return item;
}
```

### 2. Semaphores

A semaphore is like a bucket of tokens.

- **Acquire (`acquire`)**: Take a token. If bucket is empty, wait.

- **Release (`release`)**: Put a token back.

Unlike a mutex (which has ownership—only the thread that locked it can unlock it), a semaphore can be signaled by *any* thread. This makes it great for **producer-consumer** patterns or limiting concurrent access (e.g., "only allow 3 threads to access the database at once").

**Simple Thread Join Example:**

```cpp
sem_init(&s, 0)  // Initialize with 0 tokens

void thr_exit() {
    sem_post(&s)  // Signal that thread is done
}

void thr_join() {
    sem_wait(&s)  // Wait until thread signals
}
```

**Producer-Consumer Pattern (with bounded buffer):**

```cpp
sem_t empty;   // Tracks empty slots in buffer (initialized to BUFFER_SIZE)
sem_t full;    // Tracks filled slots in buffer (initialized to 0)
int buffer[BUFFER_SIZE];
int in = 0, out = 0;

void producer(int item) {
    sem_wait(&empty);         // Wait if buffer is full
    // Acquire lock to protect buffer access
    buffer[in] = item;
    in = (in + 1) % BUFFER_SIZE;
    // Release lock
    sem_post(&full);          // Signal that buffer has an item
}

void consumer() {
    sem_wait(&full);          // Wait if buffer is empty
    // Acquire lock to protect buffer access
    int item = buffer[out];
    out = (out + 1) % BUFFER_SIZE;
    // Release lock
    sem_post(&empty);         // Signal that buffer has space
    return item;
}
```

**Real-World Example: Database Connection Pool**

```cpp
sem_t pool;  // Initialized to MAX_CONNECTIONS (e.g., 5)

// A thread needing database access
void process_request() {
    sem_wait(&pool);          // Get a connection from the pool
    // Use database connection
    execute_query();
    // Release connection back to the pool
    sem_post(&pool);
}
// This ensures only 5 threads use the database simultaneously
```

## Common Pitfalls

**1. Forgetting to Re-check Conditions After Waking**

After a condition variable wakes a thread, always re-check the condition in a loop, not an `if` statement. The condition might have become false again due to other threads.

```cpp
// WRONG
if (done == 0)
    pthread_cond_wait(&cond, &lock);

// CORRECT
while (done == 0)
    pthread_cond_wait(&cond, &lock);
```

**2. Mixing Different Synchronization Primitives Incorrectly**

Don't use a spinlock where a condition variable is needed, or vice versa. Spinlocks waste CPU for waiting; condition variables have context-switch overhead.

**3. Lock Ordering Mistakes**

If you acquire multiple locks, always acquire them in the same order in every thread. Inconsistent ordering leads to deadlock.

**4. Holding Locks Too Long**

Critical sections should be as small as possible. Holding a lock while doing I/O, computation, or other long operations reduces concurrency.

```cpp
// WRONG: Lock held for entire operation
pthread_mutex_lock(&lock);
int value = database_query();  // Slow!
data = process(value);
pthread_mutex_unlock(&lock);

// CORRECT: Lock held only for shared data access
int value = database_query();
pthread_mutex_lock(&lock);
data = process(value);
pthread_mutex_unlock(&lock);
```

**5. Not Signaling When Waking All Waiting Threads is Needed**

Use `pthread_cond_broadcast(&cond)` when multiple threads might be waiting, not `pthread_cond_signal(&cond)` (which wakes only one thread).

**6. Assuming Specific Thread Scheduling**

Never assume threads execute in a particular order. Always synchronize explicitly with locks and condition variables.

## Synchronization Bugs

### Atomicity Violation

An operation that should have been atomic wasn't.

**Example of Broken Code:**

```cpp
// Shared variable
int balance = 100;

// Thread 1: Withdraw $50
int temp = balance;      // Read: temp = 100
temp = temp - 50;        // Compute: temp = 50
balance = temp;          // Write: balance = 50

// Thread 2: Withdraw $30 (interleaves with Thread 1)
int temp2 = balance;     // Read: temp2 = 100 (balance not updated yet!)
temp2 = temp2 - 30;      // Compute: temp2 = 70
balance = temp2;         // Write: balance = 70

// Result: balance = 70, but should be 20!
// Money was lost due to non-atomic reads/writes
```

**Fix:** Protect the entire operation with a lock:

```cpp
pthread_mutex_lock(&lock);
int temp = balance;
temp = temp - amount;
balance = temp;
pthread_mutex_unlock(&lock);
```

### Order Violation

Something happens sooner (or later) than expected.

**Example of Broken Code:**

```cpp
// Thread 1
void init() {
    value = 42;           // Initialize value
    initialized = 1;      // Signal initialization complete
}

// Thread 2
void use_value() {
    while (initialized == 0)  // Wait for initialization
        ;  // Spin-wait
    printf("%d\n", value);    // Print value
}

// Problem: Without synchronization, Thread 2 might read an
// outdated or partially-written 'value', or the compiler might
// reorder instructions, causing initialized to be set before value
```

**Fix:** Use a condition variable:

```cpp
// Thread 1
pthread_mutex_lock(&lock);
value = 42;
initialized = 1;
pthread_cond_signal(&cond);
pthread_mutex_unlock(&lock);

// Thread 2
pthread_mutex_lock(&lock);
while (initialized == 0)
    pthread_cond_wait(&cond, &lock);
printf("%d\n", value);
pthread_mutex_unlock(&lock);
```

### Deadlock

Two threads wait indefinitely on each other.

**Example of Broken Code:**

```cpp
// Thread 1
pthread_mutex_lock(&lock1);        // Acquire lock1
sleep(1);                           // ... do work ...
pthread_mutex_lock(&lock2);        // Try to acquire lock2 (BLOCKED!)
pthread_mutex_unlock(&lock2);
pthread_mutex_unlock(&lock1);

// Thread 2
pthread_mutex_lock(&lock2);        // Acquire lock2
sleep(1);                           // ... do work ...
pthread_mutex_lock(&lock1);        // Try to acquire lock1 (BLOCKED!)
pthread_mutex_unlock(&lock1);
pthread_mutex_unlock(&lock2);

// Deadlock: Thread 1 waits for lock2 (held by Thread 2)
//           Thread 2 waits for lock1 (held by Thread 1)
// Both threads hang forever
```

**Deadlock Prevention:** Always acquire locks in the same order across all threads:

```cpp
// All threads follow this order: lock1, then lock2
pthread_mutex_lock(&lock1);
pthread_mutex_lock(&lock2);
// ... critical section ...
pthread_mutex_unlock(&lock2);
pthread_mutex_unlock(&lock1);
```

### Livelock

Two threads repeatedly block each other from proceeding and retry.

**Example:** Two threads try to acquire locks, but each time one succeeds in acquiring one lock, the other thread also tries and they keep interfering with each other's attempts, causing repeated retries instead of progress.
