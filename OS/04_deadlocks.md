# Deadlocks

> A deadlock is a state where a set of processes are each waiting for a resource held by another in the set — a circular wait with no progress possible.

---

## 1. Coffman's Four Necessary Conditions

**All four must hold simultaneously for a deadlock:**

| Condition | Description |
|-----------|-------------|
| **Mutual Exclusion** | At least one resource must be non-shareable (held by only one process) |
| **Hold and Wait** | A process holding a resource is waiting to acquire additional resources |
| **No Preemption** | Resources cannot be forcibly taken; must be released voluntarily |
| **Circular Wait** | P1 waits for P2, P2 waits for P3, ..., Pn waits for P1 |

**Prevention = eliminate at least one condition.**

---

## 2. Resource Allocation Graph (RAG)

- **Circle** = process, **Rectangle** = resource, **Dot inside rectangle** = instance
- **Request edge:** Process → Resource (wants it)
- **Assignment edge:** Resource → Process (holds it)
- **Deadlock:** Cycle in the RAG (for single-instance resources, cycle = deadlock; for multi-instance, cycle is necessary but not sufficient)

---

## 3. Deadlock Prevention

| Condition to Break | Strategy |
|-------------------|----------|
| Mutual Exclusion | Make resources shareable (e.g., read-only files) — not always possible |
| Hold and Wait | Process must request ALL resources before starting, or release all before requesting new ones |
| No Preemption | If blocked, forcibly take away held resources; reacquire later |
| Circular Wait | Impose total ordering on resources; always acquire in order |

**Most practical:** Circular wait elimination — number all locks globally and always acquire lower-numbered locks first.

---

## 4. Banker's Algorithm — Deadlock Avoidance

Checks if granting a resource request leads to a **safe state** (there exists a sequence where all processes can eventually complete).

**Data structures:**
```
n = number of processes, m = number of resource types

Available[m]       — available instances of each resource
Max[n][m]          — maximum demand of each process
Allocation[n][m]   — current allocation
Need[n][m]         — remaining need = Max - Allocation
```

**Safety Algorithm:**
```
Work = Available.copy()
Finish = [False] * n

Repeat until no process found:
    Find process i where:
        Finish[i] == False AND Need[i] <= Work
    If found:
        Work += Allocation[i]
        Finish[i] = True

If all Finish[i] == True → safe state → safe sequence found
```

**Resource Request Algorithm:**
```
On Request[i] (process i requesting):
1. If Request[i] > Need[i]: ERROR (exceeds max claim)
2. If Request[i] > Available: WAIT (not enough resources)
3. Pretend to allocate:
   Available -= Request[i]
   Allocation[i] += Request[i]
   Need[i] -= Request[i]
4. Run Safety Algorithm:
   - If safe: grant the request
   - If unsafe: rollback and block process i
```

### Worked Example

```
Processes: P0, P1, P2, P3, P4   Resources: A(10), B(5), C(7)

           Allocation    Max        Need       Available
           A  B  C    A  B  C    A  B  C     A  B  C
P0         0  1  0    7  5  3    7  4  3     3  3  2
P1         2  0  0    3  2  2    1  2  2
P2         3  0  2    9  0  2    6  0  0
P3         2  1  1    2  2  2    0  1  1
P4         0  0  2    4  3  3    4  3  1

Safety check:
Work = [3,3,2]
P1: Need[1]=[1,2,2] <= [3,3,2] ✓ → Work=[5,3,2], Finish[1]=T
P3: Need[3]=[0,1,1] <= [5,3,2] ✓ → Work=[7,4,3], Finish[3]=T
P4: Need[4]=[4,3,1] <= [7,4,3] ✓ → Work=[7,4,5], Finish[4]=T
P2: Need[2]=[6,0,0] <= [7,4,5] ✓ → Work=[10,4,7], Finish[2]=T
P0: Need[0]=[7,4,3] <= [10,4,7] ✓ → Finish[0]=T

Safe sequence: P1 → P3 → P4 → P2 → P0 ✓
```

---

## 5. Deadlock Detection & Recovery

**Detection:** Run the safety algorithm periodically. If no safe sequence, deadlock detected.

**Recovery options:**
1. **Process termination** — kill one or all deadlocked processes (expensive)
2. **Resource preemption** — forcibly take resources from a victim process, roll it back
   - Victim selection: minimize cost (CPU time used, resources held)
   - Rollback: restart from checkpoint
   - Starvation risk: same process always chosen as victim → add rollback count to cost

---

## 6. Deadlock vs Livelock vs Starvation

| Term | Description | Example |
|------|-------------|---------|
| Deadlock | Circular wait, no progress | Thread A waits for B's lock, B waits for A's lock |
| Livelock | Threads actively try but keep giving way to each other | Two people in a corridor both stepping aside infinitely |
| Starvation | Process never gets CPU/resource due to priority | Low-priority process never scheduled |

---

## 7. Real-World Deadlock Prevention

```python
# Python: always acquire locks in the same order
import threading

lock_a = threading.Lock()
lock_b = threading.Lock()

def thread_1():
    with lock_a:         # always acquire A first
        with lock_b:
            pass

def thread_2():
    with lock_a:         # always acquire A first (not B first!)
        with lock_b:
            pass
```

**Other techniques:** Lock timeouts (`try_lock`), lock-free data structures (CAS/atomic), transactional memory.
