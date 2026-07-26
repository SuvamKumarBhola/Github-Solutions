# Merge Sorted Array

## Problem Statement

You're given two integer arrays `nums1` and `nums2`, both sorted in non-decreasing order, and two integers `m` and `n` representing the number of actual elements in `nums1` and `nums2` respectively.

`nums1` has a size of `m + n` — the first `m` elements are the real values, and the last `n` elements are just placeholder zeros to leave room for merging `nums2` in.

**Merge `nums2` into `nums1`** so that `nums1` becomes one sorted array of length `m + n`. You must do this **in-place** (modifying `nums1` directly, not returning a new array).

### Example 1
```
Input:  nums1 = [1,2,3,0,0,0], m = 3
        nums2 = [2,5,6],       n = 3
Output: nums1 = [1,2,2,3,5,6]
```

### Example 2
```
Input:  nums1 = [1], m = 1
        nums2 = [],  n = 0
Output: nums1 = [1]
```

### Example 3
```
Input:  nums1 = [0], m = 0
        nums2 = [1], n = 1
Output: nums1 = [1]
```

---

## Key Insight — Why Not Just Merge From the Front?

If you try to merge from the **front** (like classic merge-sort merging), you'd need to insert `nums2` elements into the middle of `nums1`, which means **shifting existing elements to the right** every time — expensive, O(n) per insertion.

But notice: `nums1` conveniently has **empty space at the end** already reserved. So instead of merging front-to-back, we merge **back-to-front**. We compare the *largest* remaining elements of each array and place the bigger one at the *last* empty slot, working backwards. This way we're always writing into space that's guaranteed to be empty (no shifting needed) — same trick as "Squares of a Sorted Array."

---

## Approach 1: Three Pointers, Fill From the Back (Optimal) 

**Idea:**
- `p1 = m - 1` → points to the last real element in `nums1`
- `p2 = n - 1` → points to the last element in `nums2`
- `p = m + n - 1` → points to the last slot in `nums1` (where we write next)

Compare `nums1[p1]` and `nums2[p2]`. Place the bigger one at `nums1[p]`, move that pointer back, and move `p` back. Repeat until `nums2` is fully placed (once `nums1`'s real elements are exhausted, any remaining `nums2` elements are just copied over — they're already smaller than everything placed so far).

### Walkthrough with `nums1 = [1,2,3,0,0,0], m=3` and `nums2 = [2,5,6], n=3`

| p1 | p2 | p | nums1[p1] | nums2[p2] | Larger | Action | nums1 (after) |
|---|---|---|---|---|---|---|---|
| 2 | 2 | 5 | 3 | 6 | 6 | place 6 at idx 5, p2-- | [1,2,3,0,0,6] |
| 2 | 1 | 4 | 3 | 5 | 5 | place 5 at idx 4, p2-- | [1,2,3,0,5,6] |
| 2 | 0 | 3 | 3 | 2 | 3 | place 3 at idx 3, p1-- | [1,2,3,3,5,6] |
| 1 | 0 | 2 | 2 | 2 | tie→nums2 | place 2 at idx 2, p2-- | [1,2,2,3,5,6] |
| 1 | -1 | 1 | — | — | p2 < 0 | loop ends (nums2 exhausted) | [1,2,2,3,5,6] |

Result: `[1,2,2,3,5,6]`  — and since `p2` hit `-1`, we stop; any remaining elements in `nums1`'s real portion are already in their correct place.

### Code (Python)

```python
def merge(nums1, m, nums2, n):
    p1 = m - 1        # last real element in nums1
    p2 = n - 1        # last element in nums2
    p = m + n - 1      # last slot to fill in nums1
    
    while p2 >= 0:
        if p1 >= 0 and nums1[p1] > nums2[p2]:
            nums1[p] = nums1[p1]
            p1 -= 1
        else:
            nums1[p] = nums2[p2]
            p2 -= 1
        p -= 1
    # If p2 < 0 first, nums1's remaining elements are already in place.
    # If p1 < 0 first, the while loop keeps copying leftover nums2 elements.
```

### Code (Java)

```java
public void merge(int[] nums1, int m, int[] nums2, int n) {
    int p1 = m - 1, p2 = n - 1, p = m + n - 1;
    
    while (p2 >= 0) {
        if (p1 >= 0 && nums1[p1] > nums2[p2]) {
            nums1[p--] = nums1[p1--];
        } else {
            nums1[p--] = nums2[p2--];
        }
    }
}
```

### Code (C++)

```cpp
void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
    int p1 = m - 1, p2 = n - 1, p = m + n - 1;
    
    while (p2 >= 0) {
        if (p1 >= 0 && nums1[p1] > nums2[p2]) {
            nums1[p--] = nums1[p1--];
        } else {
            nums1[p--] = nums2[p2--];
        }
    }
}
```

**Why the loop only checks `p2 >= 0`:** 
Once `nums2` is fully placed, whatever's left in `nums1` (from index 0 to `p1`) is *already sitting in the correct sorted position* — it never needed to move, since we've only been filling in from empty slots at the back. So there's nothing left to do. But if `nums1` runs out first (`p1 < 0`), the loop must keep running to copy the rest of `nums2` over — that's why the loop condition is on `p2`, not both.

**Complexity:**
- Time: **O(m + n)** — each element visited once
- Space: **O(1)** — no extra array; writes directly into `nums1`'s existing storage

---

## Approach 2: Merge Into a New Array, Then Copy Back

**Idea:** Copy the real elements of both arrays into a fresh list, merge them the "normal" front-to-back way (like classic merge sort's merge step), then copy the result back into `nums1`.

```python
def merge(nums1, m, nums2, n):
    merged = []
    i, j = 0, 0
    while i < m and j < n:
        if nums1[i] <= nums2[j]:
            merged.append(nums1[i])
            i += 1
        else:
            merged.append(nums2[j])
            j += 1
    merged.extend(nums1[i:m])
    merged.extend(nums2[j:n])
    
    for k in range(m + n):
        nums1[k] = merged[k]
```

**Why it's worse:**
- Uses **O(m + n) extra space** for the temporary `merged` list — defeats the purpose of the in-place constraint the problem is testing
- Time complexity is the same, O(m+n), but it's considered a weaker solution in interviews since it ignores the free space already available in `nums1`

---

## Approach 3: Naive — Append, Then Sort

**Idea:** Just append all of `nums2`'s real elements onto `nums1`'s real elements, then sort the whole thing.

```python
def merge(nums1, m, nums2, n):
    nums1[m:] = nums2[:n]
    nums1.sort()
```

**Why it's worse:**
- Time: **O((m+n) log(m+n))** — completely ignores that *both* arrays were already individually sorted, which is exactly the structure that makes this problem solvable in linear time
- It's the simplest code to write, and totally correct, but it's the "brute force" answer an interviewer would expect you to improve on

---

## Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| **Three Pointers (back-to-front)** | O(m+n) | O(1) | Optimal — exploits the pre-allocated space in nums1 |
| **Merge into new array** | O(m+n) | O(m+n) | Same time, wastes the built-in space |
| **Append + Sort** | O((m+n) log(m+n)) | O(1) or O(log n) for sort | Simplest, but ignores sorted structure entirely |

---

## The Common Thread Across All Four Problems

You've now seen this same family of two-pointer tricks four times:

1. **Remove Duplicates from Sorted Array** — slow/fast pointers, same direction, overwriting in place
2. **Remove Duplicates from Sorted List** — same idea, but with pointer *rewiring* instead of array indices
3. **Squares of a Sorted Array** — two pointers converging from *both ends*, writing the result **backwards**
4. **Merge Sorted Array** — two pointers converging from *both ends* of two *separate* arrays, again writing **backwards** into pre-allocated space

The unifying principle: **whenever you have sorted input and are told there's "extra space" or a need for in-place modification, look at whether processing from the back (largest to smallest) lets you avoid shifting elements** — because writing into an already-empty/junk slot is free, whereas writing into an occupied slot in the middle requires displacing something else.
