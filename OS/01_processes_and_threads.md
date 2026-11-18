# Processes and Threads

> **Process:** Program in execution — has its own address space, file handles, security context. **Thread:** Lightweight unit within a process — shares address space, cheaper to create/switch.

---

## 1. Process vs Thread

| Aspect | Process | Thread |
|--------|---------|--------|
| Memory | Isolated address space | Shared address space |
| Creation cost | High (fork + copy-on-write) | Low (~10x faster) |
| Communication | IPC (pipe, socket, shared mem) | Shared variables (need sync) |
| Crash impact | Isolated (other processes safe) | Crashes whole process |
| Context switch | Slower (save/restore page table) | Faster |

**When to use threads:** CPU-bound parallelism (compute tasks), IO-bound concurrency (web server handling multiple connections).

**Python GIL note:** CPython has a Global Interpreter Lock — only one thread runs Python bytecode at a time. Use `multiprocessing` for CPU-bound work in Python.

---

## 2. Process Control Block (PCB)

The OS maintains a PCB for every process. Contains:
- **PID** — process ID
- **Process state** — new, ready, running, waiting, terminated
- **Program counter** — address of next instruction
- **CPU registers** — context for resuming
- **Memory management info** — page table pointer, segment limits
- **I/O status** — open files, devices
- **Accounting** — CPU time used, priority

---

## 3. Process States

```
NEW ──────────────────────────────► TERMINATED
 |                                       ▲
 ▼                                       |
READY ──► RUNNING ──────────────────────┘
  ▲          |           |
  |──────────┘           ▼
  |                   WAITING
  └───────────────────────┘
     (I/O or event completes)
```

---

## 4. Mutex (Mutual Exclusion Lock)

```cpp
#include <mutex>
mutex mtx;
int shared_counter = 0;

void increment() {
    mtx.lock();
    shared_counter++;
    mtx.unlock();
}

// Preferred: RAII guard (exception-safe)
void increment_safe() {
    lock_guard<mutex> lock(mtx);
    shared_counter++;
} // lock released automatically on scope exit
```

**Deadlock with two mutexes:** Always acquire in the same order across all threads.

---

## 5. Semaphore

A semaphore is a signaling mechanism with a counter.
- `wait()` / `P()`: decrement counter; block if counter == 0
- `signal()` / `V()`: increment counter; wake a waiting thread

```python
import threading

sem = threading.Semaphore(1)  # binary semaphore = mutex

def critical_section():
    sem.acquire()   # P()
    # ... do work ...
    sem.release()   # V()
```

**Counting semaphore:** Allows N concurrent accesses (e.g., connection pool of size N).

---

## 6. Producer-Consumer Problem

```python
import threading
from queue import Queue

buffer = Queue(maxsize=10)  # bounded buffer

def producer():
    while True:
        item = produce_item()
        buffer.put(item)       # blocks if full

def consumer():
    while True:
        item = buffer.get()    # blocks if empty
        consume(item)
        buffer.task_done()
```

**Manual implementation with semaphores:**
- `empty` semaphore = capacity (initially = N)
- `full` semaphore = items in buffer (initially = 0)
- `mutex` for buffer access

---

## 7. Condition Variables

```cpp
mutex mtx;
condition_variable cv;
bool ready = false;

// Thread 1 (waiter)
unique_lock<mutex> lk(mtx);
cv.wait(lk, [](){ return ready; });  // atomically unlock + wait

// Thread 2 (notifier)
{
    lock_guard<mutex> lk(mtx);
    ready = true;
}
cv.notify_one();  // or notify_all()
```

---

## 8. Context Switch

Steps when OS preempts a running thread:
1. Save CPU registers → current thread's PCB/TCB
2. Update process state to READY
3. Run scheduler → select next thread
4. Restore CPU registers from new thread's PCB/TCB
5. Set program counter to new thread's instruction

**Cost:** ~1–10 µs on modern hardware. Excessive context switching = "thrashing."

---

## 9. Interview Quick Reference

| Question | Answer |
|----------|--------|
| What is a race condition? | Two threads access shared data concurrently and at least one writes, without synchronization |
| What is a deadlock? | Circular wait — thread A holds lock 1, waits for lock 2; thread B holds lock 2, waits for lock 1 |
| What is starvation? | A thread never gets CPU time because higher-priority threads always run |
| What is a spinlock? | Thread busy-waits (spins) checking a flag instead of sleeping; good for very short critical sections |
| fork() vs exec()? | `fork()` creates a child process (copy of parent); `exec()` replaces current process image with a new program |
| What is copy-on-write (COW)? | After fork(), parent and child share physical pages; a copy is made only when one writes |
