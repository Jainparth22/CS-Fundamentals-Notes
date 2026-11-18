# Linked Lists

> **Core idea:** Non-contiguous nodes connected by pointers. O(1) insert/delete at known position, O(n) access.

---

## 1. Node Structure

```cpp
// Singly Linked List
struct ListNode {
    int val;
    ListNode* next;
    ListNode(int x) : val(x), next(nullptr) {}
};

// Doubly Linked List
struct DNode {
    int val;
    DNode *prev, *next;
    DNode(int x) : val(x), prev(nullptr), next(nullptr) {}
};
```

```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next
```

---

## 2. Essential Operations

### Reverse a Singly Linked List
```cpp
ListNode* reverse(ListNode* head) {
    ListNode* prev = nullptr;
    ListNode* cur  = head;
    while (cur) {
        ListNode* nxt = cur->next;
        cur->next = prev;
        prev = cur;
        cur = nxt;
    }
    return prev;  // new head
}
```

**Trace:** `1->2->3->nullptr` becomes `nullptr<-1<-2<-3`, return `3`.

### Merge Two Sorted Lists
```cpp
ListNode* merge(ListNode* l1, ListNode* l2) {
    ListNode dummy(0);
    ListNode* tail = &dummy;
    while (l1 && l2) {
        if (l1->val <= l2->val) { tail->next = l1; l1 = l1->next; }
        else                    { tail->next = l2; l2 = l2->next; }
        tail = tail->next;
    }
    tail->next = l1 ? l1 : l2;
    return dummy.next;
}
```

> **Dummy head trick** — avoids special-casing the first node. Use whenever you build a new list.

### Find Middle (Slow/Fast Pointers)
```cpp
ListNode* findMiddle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
    }
    return slow;  // middle (for even length: second middle)
}
```

---

## 3. Floyd's Cycle Detection

**Problem:** Detect if a linked list has a cycle; find the start of the cycle.

```cpp
bool hasCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) return true;
    }
    return false;
}

// Find cycle start
ListNode* detectCycle(ListNode* head) {
    ListNode* slow = head;
    ListNode* fast = head;
    // Phase 1: detect
    while (fast && fast->next) {
        slow = slow->next;
        fast = fast->next->next;
        if (slow == fast) {
            // Phase 2: find entry
            ListNode* entry = head;
            while (entry != slow) {
                entry = entry->next;
                slow  = slow->next;
            }
            return entry;
        }
    }
    return nullptr;
}
```

**Why Phase 2 works:** Let `F` = distance from head to cycle start, `C` = cycle length, `K` = distance from cycle start to meeting point. After meeting: `F = C - K`. Moving one pointer from head and one from meeting point at same speed, they meet at cycle start.

---

## 4. Common Patterns

### K-th Node from End (Two Pointers)
```cpp
ListNode* removeNthFromEnd(ListNode* head, int n) {
    ListNode dummy(0);
    dummy.next = head;
    ListNode* fast = &dummy;
    ListNode* slow = &dummy;
    // advance fast by n+1
    for (int i = 0; i <= n; i++) fast = fast->next;
    while (fast) { slow = slow->next; fast = fast->next; }
    slow->next = slow->next->next;  // remove
    return dummy.next;
}
```

### Palindrome Check
1. Find middle (slow/fast)
2. Reverse second half
3. Compare both halves
4. (Optional) Restore: reverse second half again

### Reorder List (LC 143)
1. Find middle
2. Reverse second half
3. Merge alternately: `1->4->2->3` from `1->2->3` and `4->3`

---

## 5. Complexity

| Operation | Singly LL | Doubly LL | Array |
|-----------|-----------|-----------|-------|
| Access by index | O(n) | O(n) | O(1) |
| Insert at head | O(1) | O(1) | O(n) |
| Insert at tail | O(n)* | O(1) with tail ptr | O(1) amortized |
| Insert at known node | O(1) | O(1) | O(n) |
| Delete at head | O(1) | O(1) | O(n) |
| Search | O(n) | O(n) | O(n) |

*O(1) if tail pointer is maintained.

---

## 6. Practice Problems

| Problem | Key Technique | LC # |
|---------|---------------|------|
| Reverse Linked List | Iterative/recursive | 206 |
| Merge Two Sorted Lists | Dummy head, two pointers | 21 |
| Reorder List | Middle + reverse + merge | 143 |
| Remove Nth from End | Two pointers (gap n+1) | 19 |
| Linked List Cycle | Floyd's detect | 141 |
| Find Duplicate Number | Floyd's on array as graph | 287 |
| LRU Cache | Doubly LL + hash map | 146 |
| Merge K Sorted Lists | Min-heap or divide-and-conquer | 23 |
| Reverse Nodes in K-Group | Reverse sublist iteratively | 25 |
