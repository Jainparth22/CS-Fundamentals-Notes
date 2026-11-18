# CPU Scheduling

> The OS scheduler decides which ready process runs next. Key metrics: **Turnaround Time (TAT)** = Completion - Arrival, **Waiting Time (WT)** = TAT - Burst, **Response Time** = First Run - Arrival.

---

## 1. Scheduling Criteria

| Metric | Meaning | Optimize For |
|--------|---------|--------------|
| CPU Utilization | % of time CPU is busy | Maximize |
| Throughput | Processes/second completed | Maximize |
| Turnaround Time | Total time from arrival to completion | Minimize |
| Waiting Time | Time spent in ready queue | Minimize |
| Response Time | Time until first response (interactive) | Minimize |

---

## 2. FCFS — First Come First Served

**Non-preemptive.** Run in arrival order. Simple but can cause the **convoy effect**.

**Example:**
```
Process  Arrival  Burst
  P1       0       24
  P2       1        3
  P3       2        3

Gantt: | P1(0-24) | P2(24-27) | P3(27-30) |
WT:    P1=0, P2=23, P3=25  → Avg WT = 16
TAT:   P1=24, P2=26, P3=28 → Avg TAT = 26
```

**Convoy effect:** Short processes wait for one long process.

---

## 3. SJF — Shortest Job First (Non-preemptive)

Run the process with the smallest burst time next. **Optimal for minimizing average WT** (provable by exchange argument). But requires knowing burst times in advance.

**Example:**
```
Process  Arrival  Burst
  P1       0        6
  P2       0        8
  P3       0        7
  P4       0        3

Sorted by burst: P4(3), P1(6), P3(7), P2(8)
Gantt: | P4(0-3) | P1(3-9) | P3(9-16) | P2(16-24) |
Avg WT = (0 + 3 + 9 + 16) / 4 = 7
```

---

## 4. SRTF — Shortest Remaining Time First (Preemptive SJF)

Preempt if a new process arrives with smaller remaining burst.

```
Process  Arrival  Burst
  P1       0        8
  P2       1        4
  P3       2        9
  P4       3        5

t=0: P1 starts (RT=8)
t=1: P2 arrives (RT=4 < P1 RT=7) → preempt, run P2
t=5: P2 done; P4 (RT=5) < P1 (RT=7) → run P4
t=8: P4 done; P1 (RT=7) < P3 (RT=9) → run P1
t=15: P1 done; run P3
t=24: P3 done.

WT: P1=15-8+1=9? Recompute: 
  P1 runs t=0-1 and t=8-15 → WT = 8-1=7
  Properly: WT = TAT - Burst
```

---

## 5. Round Robin (RR)

**Preemptive.** Each process gets a fixed **time quantum** Q. Preempt after Q; go to back of ready queue.

**Rule of thumb:** Q should be > 80% of burst times (too small = too much context switch overhead; too large = degrades to FCFS).

**Example (Q=4):**
```
Process  Burst
  P1      24
  P2       3
  P3       3

Gantt: |P1(0-4)|P2(4-7)|P3(7-10)|P1(10-14)|P1(14-18)|P1(18-22)|P1(22-26)|
WT:    P1=10-4=10+... compute properly
Avg WT = (6 + 4 + 7) / 3 = 5.67
```

**Best for:** Interactive/time-sharing systems. Fairness over throughput.

---

## 6. Priority Scheduling

Each process has a priority number. CPU given to highest priority (smallest number = highest priority convention).

**Problem: Starvation** — low-priority processes may never run.
**Solution: Aging** — gradually increase priority of waiting processes.

**Preemptive variant:** Preempt if a higher-priority process arrives.

---

## 7. Multilevel Queue & Multilevel Feedback Queue (MLFQ)

**Multilevel Queue:** Multiple queues, each with own algorithm (e.g., foreground = RR, background = FCFS).

**MLFQ (adaptive):**
- New processes start at highest priority queue
- If they use full quantum, demote to lower queue
- If they voluntarily yield (I/O wait), stay at same level or promote
- **Result:** CPU-bound processes sink, I/O-bound processes rise

---

## 8. Formulas

```
Turnaround Time (TAT) = Completion Time - Arrival Time
Waiting Time (WT)     = TAT - Burst Time
Response Time (RT)    = First CPU Time - Arrival Time
CPU Utilization       = (Busy Time / Total Time) × 100%
Throughput            = Completed Processes / Total Time
```

---

## 9. Quick Comparison

| Algorithm | Preemptive | Optimal For | Starvation? |
|-----------|------------|-------------|-------------|
| FCFS | No | Simplicity | No |
| SJF | No | Min Avg WT | Yes (long jobs) |
| SRTF | Yes | Min Avg WT (preemptive) | Yes |
| RR | Yes | Response time / fairness | No |
| Priority | Both | Prioritized workloads | Yes |
| MLFQ | Yes | General-purpose OS | No (with aging) |
