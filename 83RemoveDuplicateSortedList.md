# Remove Duplicates from Sorted List

## Problem Statement

You're given the `head` of a **sorted linked list** (sorted in non-decreasing order). Delete all duplicate nodes such that each unique value appears **only once**. Return the linked list, still sorted.

### Example 1
```
Input:  1 -> 1 -> 2
Output: 1 -> 2
```

### Example 2
```
Input:  1 -> 1 -> 2 -> 3 -> 3
Output: 1 -> 2 -> 3
```

---

## Key Insight

Just like the array version, the list is **already sorted**, so duplicates are guaranteed to be **adjacent nodes**. This means at any node, you only ever need to compare it with the **next node** — no need to search further ahead or use extra data structures to track "seen" values.

---

## Approach 1: Single Pointer, Iterative (Optimal) ✅

**Idea:** 
Walk through the list with one pointer `curr`. If `curr.val == curr.next.val`, it means `curr.next` is a duplicate — skip over it by relinking `curr.next = curr.next.next`. If they're different, just move `curr` forward.

The key subtlety: when you find a duplicate and skip it, you **don't move `curr` forward** — you stay put, because the node after the deleted one might *also* be a duplicate of `curr`.

### Walkthrough with `1 -> 1 -> 1 -> 2 -> 3 -> 3`

| curr | curr.next | Equal? | Action |
|---|---|---|---|
| 1 (node A) | 1 (node B) | Yes | Skip B: curr.next = C |
| 1 (node A) | 1 (node C) | Yes | Skip C: curr.next = D(2) |
| 1 (node A) | 2 | No | Move curr → 2 |
| 2 | 3 | No | Move curr → 3 (first) |
| 3 | 3 | Yes | Skip second 3 |
| 3 | null | — | Done |

Result: `1 -> 2 -> 3`

### Code (Python)

```python
# Definition for singly-linked list.
class ListNode:
    def __init__(self, val=0, next=None):
        self.val = val
        self.next = next

def deleteDuplicates(head):
    curr = head
    while curr and curr.next:
        if curr.val == curr.next.val:
            curr.next = curr.next.next  # skip the duplicate
        else:
            curr = curr.next  # move forward only when values differ
    return head
```

### Code (Java)

```java
public ListNode deleteDuplicates(ListNode head) {
    ListNode curr = head;
    while (curr != null && curr.next != null) {
        if (curr.val == curr.next.val) {
            curr.next = curr.next.next;
        } else {
            curr = curr.next;
        }
    }
    return head;
}
```

### Code (C++)

```cpp
ListNode* deleteDuplicates(ListNode* head) {
    ListNode* curr = head;
    while (curr && curr->next) {
        if (curr->val == curr->next->val) {
            curr->next = curr->next->next;
        } else {
            curr = curr->next;
        }
    }
    return head;
}
```

**Complexity:**
- Time: **O(n)** — single pass through the list
- Space: **O(1)** — no extra structures, just pointer manipulation (in-place)

Note: unlike the array problem, we don't need to worry about "freeing" the skipped nodes manually in Python/Java (garbage collected); in C++ you'd typically `delete` the skipped node to avoid a memory leak if that matters for the context.

---

## Approach 2: Recursive

**Idea:** Recursively process the rest of the list first, then decide whether to keep the current node.

```python
def deleteDuplicates(head):
    if not head or not head.next:
        return head
    
    head.next = deleteDuplicates(head.next)
    
    return head.next if head.val == head.next.val else head
```

**How it works:** This recurses all the way to the end of the list first, then unwinds. At each step, after the rest of the list has been de-duplicated, it checks if the current node's value matches the (now-deduplicated) next node's value. If so, it discards the current node by returning `head.next` instead.

**Complexity:**
- Time: **O(n)**
- Space: **O(n)** — due to the recursion call stack (this is the main downside vs. the iterative approach)

---

## Approach 3: Using a Set/Hash to Track Seen Values

**Idea:** Even though it's sorted (so this is overkill here), you could track seen values in a set and skip nodes whose value you've already seen.

```python
def deleteDuplicates(head):
    if not head:
        return head
    
    seen = {head.val}
    curr = head
    while curr.next:
        if curr.next.val in seen:
            curr.next = curr.next.next
        else:
            seen.add(curr.next.val)
            curr = curr.next
    return head
```

**Why it's worse here:**
- Uses **O(n) extra space** for the set — completely unnecessary since the list is sorted and duplicates are always adjacent
- This approach is really meant for the *unsorted* version of this problem, where duplicates could be scattered anywhere in the list. On a sorted list, it's strictly more expensive than Approach 1 for no benefit.

---

## Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| **Iterative (single pointer)** | O(n) | O(1) | Optimal — exploits sorted + adjacency |
| **Recursive** | O(n) | O(n) | Elegant, but stack space cost |
| **Set/Hash tracking** | O(n) | O(n) | Overkill for sorted list; useful for *unsorted* variant |

---

## Connection to the Array Version

This is essentially the **linked-list twin** of "Remove Duplicates from Sorted Array" you looked at earlier. The core idea is identical — **exploit sortedness so duplicates are adjacent, and only ever compare a node/index with its immediate neighbor.** The mechanics differ because:
- Arrays need **index bookkeeping** (a slow pointer marking where to overwrite)
- Linked lists need **pointer rewiring** (skipping over nodes by changing `.next`)

There's also a well-known follow-up: **"Remove Duplicates from Sorted List II"**, where instead of keeping one copy of each duplicate value, you **delete all nodes** that have duplicates, leaving only values that appeared exactly once. Want me to walk through that one too? It requires a dummy head node and a slightly trickier pointer logic since the head itself might get deleted.