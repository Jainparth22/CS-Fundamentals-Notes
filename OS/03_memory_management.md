# Memory Management

> The OS manages physical RAM and provides each process with a virtual address space. Key concepts: paging, segmentation, TLB, and page replacement algorithms.

---

## 1. Address Space Layout (Virtual Memory)

```
High address ┌──────────────────┐
             │  Kernel space    │  (not accessible to user)
             ├──────────────────┤
             │  Stack           │  grows downward ↓
             │      ↓           │
             │   (gap)          │
             │      ↑           │
             │  Heap            │  grows upward ↑
             ├──────────────────┤
             │  BSS (uninit)    │  zero-initialized globals
             ├──────────────────┤
             │  Data (init)     │  initialized globals
             ├──────────────────┤
Low address  │  Text (code)     │  read-only
             └──────────────────┘
```

---

## 2. Paging

**Concept:** Divide physical memory into fixed-size **frames** and logical memory into same-size **pages**. Eliminate external fragmentation.

```
Virtual Address = [Page Number | Offset]
Physical Address = frame_number * page_size + offset

Page Table: maps page number → frame number
```

**Page Table Entry (PTE) contains:**
- Frame number
- Present/valid bit (is page in RAM?)
- Dirty bit (has page been modified?)
- Reference bit (recently accessed?)
- Protection bits (read/write/execute)

### Translation
```
Virtual Address: 32-bit, page size 4KB (2^12)
  → top 20 bits = page number (1M pages)
  → bottom 12 bits = offset

Physical address = PageTable[page_num] * 4096 + offset
```

---

## 3. TLB — Translation Lookaside Buffer

The page table is in RAM — translating every address would require two memory accesses. The TLB caches recent page-to-frame mappings.

```
TLB lookup:
  HIT  (>95% of accesses): address translation in ~1 ns
  MISS: walk page table in RAM (~100 ns) + update TLB

Effective Access Time = HIT_RATE × TLB_time + MISS_RATE × (TLB_time + PageTable_time)
```

**TLB Shootdown:** On context switch, TLB must be flushed (or use ASIDs to tag entries per process).

---

## 4. Multi-Level Paging

For 64-bit address spaces, a single page table would be enormous. Use a hierarchy:

```
4-Level Paging (x86-64):
  Virtual Address = [PML4 | PDP | PD | PT | Offset]
                       9b     9b   9b   9b    12b

  CR3 register → PML4 table → PDP table → PD table → Page Table → Frame
```

Each level adds one memory access on a TLB miss (5 accesses total on miss).

---

## 5. Page Replacement Algorithms

When a page fault occurs and no free frame is available, the OS must evict a page.

### FIFO (First In First Out)
- Evict the page that has been in memory the longest
- Simple but suffers **Belady's Anomaly** (more frames can cause more faults)

### Optimal (OPT / Belady's Algorithm)
- Evict the page that won't be used for the longest time in the future
- **Not implementable** (requires knowing future); used as benchmark

### LRU (Least Recently Used) ✅
- Evict the page that was least recently accessed
- Requires timestamp or stack; approximated by hardware reference bits
- No Belady's anomaly

### LRU Approximation — Clock Algorithm
```
Pages arranged in a circle; clock hand sweeps:
- If reference bit = 1: clear to 0, advance hand (second chance)
- If reference bit = 0: evict this page, advance hand
```

### Comparison Example
```
Reference string: 1 2 3 4 1 2 5 1 2 3 4 5
Frames = 3

FIFO faults: 9     (suffers Belady's: 4 frames gives 10!)
OPT faults:  6
LRU faults:  8
```

---

## 6. Thrashing

**Definition:** Processes spend more time page-faulting than executing.

**Cause:** Total working set size > available physical memory.

**Solutions:**
1. **Working Set Model** — give each process exactly its working set's frames
2. **Page Fault Frequency** — if fault rate > upper threshold, give more frames; if < lower, take frames back
3. **Swap out entire processes** to reduce multiprogramming degree

---

## 7. Memory Allocation Strategies

| Strategy | Description | Fragmentation |
|----------|-------------|---------------|
| First Fit | Allocate first hole that's big enough | External |
| Best Fit | Smallest hole that fits | External (worst) |
| Worst Fit | Largest hole | External |
| Paging | Fixed-size pages → no external frag | Internal only |
| Segmentation | Variable-size segments → external frag | External |

**Internal fragmentation:** Allocated block larger than needed.
**External fragmentation:** Total free space sufficient but not contiguous.

---

## 8. Interview Quick Reference

| Question | Answer |
|----------|--------|
| What is a page fault? | Accessing a page not currently in physical memory; OS loads it from disk |
| What is demand paging? | Pages loaded only when accessed, not all at program start |
| Stack vs Heap? | Stack: auto-managed, LIFO, fast; Heap: manual/GC, can be fragmented, slower |
| What is `malloc` doing? | Calling `brk`/`mmap` syscall to get pages from OS, then managing in user space |
| Virtual memory advantage? | Isolation, allows overcommit, enables features like CoW and memory-mapped files |
