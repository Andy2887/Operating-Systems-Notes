# Concurrency

### Critical Sections

**Definition:**

The code that accesses a shared resource.

For read access: If all threads only read from shared memory, then it is not a critical section.

If there is at least one write, then it is a critical section.

### Lock Design

**Solution Requirements**

1. No two threads may simultaneously be in their critical sections.

2. Threads outside of critical sections should have no impact.

3. No assumptions should be made about number of cores, speed
   of cores, or scheduler choices.

##### 1. Spinlock

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

##### 2, Ticketlock

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

##### 3, Yielding Lock

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

##### 4, Queueing Lock

- **Mechanism:**
  
  - **Acquire:** If a thread fails to acquire the lock, it adds its ID to a waiting thread queue and put itself to sleep.
  
  - **Release:** The thread releasing the lock dequeues the next waiting thread ID and calls a function to unblock that thread.
  
  - The Linux **Futex (Fast Userspace Mutex)** is a common example, where the queue is managed in the kernel, and system calls are only made when a thread actually needs to wait or wake up.

- **Benefit (Pro):** These sophisticated locks are generally more fair and do not waste processor time on "busy waiting".

- **Drawback (Con):** They can have unnecessary context-switch overhead if the lock is only briefly and rarely held.

### Synchronization

##### 1. Condition Variables (`std::condition_variable`)

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

##### 2. Semaphores

A semaphore is like a bucket of tokens.

- **Acquire (`acquire`)**: Take a token. If bucket is empty, wait.

- **Release (`release`)**: Put a token back.

Unlike a mutex (which has ownership—only the thread that locked it can unlock it), a semaphore can be signaled by *any* thread. This makes it great for **producer-consumer** patterns or limiting concurrent access (e.g., "only allow 3 threads to access the database at once").

```cpp
sem_init(&s, 1)

void thr_exit() {
    sem_post(&s)
}

void thr_join() {
    sem_wait(&s)
}
```

### Synchronization Bugs

##### Atomicity violation

An operation that should have been atomic wasn’t

##### Order violation

Something happens sooner (or later) than expected

##### Deadlock

Two threads wait indefinitely on each other

**Deadlock Prevention**:

The simplest solution is to always acquire locks in the same order.

##### Livelock

Two threads repeatedly block each other from proceeding and retry


