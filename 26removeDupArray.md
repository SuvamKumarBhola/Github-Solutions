# Remove Duplicates from Sorted Array

## Problem Statement

You're given an integer array `nums` sorted in **non-decreasing order**. You need to remove the duplicates **in-place** such that each unique element appears only **once**. The relative order of elements should be kept the same.

Since it's not possible to resize an array in some languages, you must instead:
- Modify `nums` in-place so that the first `k` elements contain the unique elements (in order)
- The remaining elements beyond `k` don't matter
- Return `k` (the number of unique elements)

### Example

```
Input:  nums = [1,1,2,2,3,4,4,5]
Output: 5, nums = [1,2,3,4,5,_,_,_]
```

Explanation: The function should return `k = 5`, with the first five elements of `nums` being `1, 2, 3, 4, 5`. It doesn't matter what values are set beyond the returned `k`.

Another example:
```
Input:  nums = [0,0,1,1,1,2,2,3,3,4]
Output: 5, nums = [0,1,2,3,4,_,_,_,_,_]
```

---

## Key Insight

Because the array is **already sorted**, all duplicate values are guaranteed to be **adjacent** to each other. This means we never need to search the whole array to find duplicates — we only ever need to compare an element with the last unique element we've kept so far.

This is the classic setup for the **two-pointer technique**.

---

## Approach 1: Two Pointers (Optimal) ✅

**Idea:** 
Keep one pointer `i` that marks the position where the next unique element should go. Use another pointer `j` to scan through the array. Whenever `nums[j]` is different from `nums[i-1]` (the last unique value placed), copy it into position `i` and advance `i`.

### Walkthrough with `[0,0,1,1,1,2,2,3,3,4]`

| j | nums[j] | Compare with nums[i-1] | Action | i (after) |
|---|---------|--------------------------|--------|-----|
| 0 | 0 | i=0, so just place it | nums[0]=0 | 1 |
| 1 | 0 | equals nums[0]=0 | skip | 1 |
| 2 | 1 | ≠ nums[0]=0 | nums[1]=1 | 2 |
| 3 | 1 | equals nums[1]=1 | skip | 2 |
| 4 | 1 | equals nums[1]=1 | skip | 2 |
| 5 | 2 | ≠ nums[1]=1 | nums[2]=2 | 3 |
| 6 | 2 | equals | skip | 3 |
| 7 | 3 | ≠ nums[2]=2 | nums[3]=3 | 4 |
| 8 | 3 | equals | skip | 4 |
| 9 | 4 | ≠ nums[3]=3 | nums[4]=4 | 5 |

Final: `i = 5` → array becomes `[0,1,2,3,4,...]`

### Code (Python)

```python
def removeDuplicates(nums):
    if not nums:
        return 0
    
    i = 1  # position to place the next unique element
    for j in range(1, len(nums)):
        if nums[j] != nums[i - 1]:
            nums[i] = nums[j]
            i += 1
    
    return i
```

### Code (C++)

```cpp
int removeDuplicates(vector<int>& nums) {
    if (nums.empty()) return 0;
    
    int i = 1;
    for (int j = 1; j < nums.size(); j++) {
        if (nums[j] != nums[i - 1]) {
            nums[i] = nums[j];
            i++;
        }
    }
    return i;
}
```

### Code (Java)

```java
public int removeDuplicates(int[] nums) {
    if (nums.length == 0) return 0;
    
    int i = 1;
    for (int j = 1; j < nums.length; j++) {
        if (nums[j] != nums[i - 1]) {
            nums[i] = nums[j];
            i++;
        }
    }
    return i;
}
```

**Complexity:**
- Time: **O(n)** — single pass through the array
- Space: **O(1)** — in-place, no extra data structures

---

## Approach 2: Using a Set (Not in-place-friendly, but intuitive)

**Idea:** Use a set (or dict) to track seen values, then rebuild the array.

```python
def removeDuplicates(nums):
    unique_vals = sorted(set(nums))
    for i, val in enumerate(unique_vals):
        nums[i] = val
    return len(unique_vals)
```

**Why it's worse:**
- Uses **O(n) extra space** for the set, which violates the spirit of the "in-place" constraint
- Also does unnecessary work — sorting again, even though the array was already sorted
- Time: O(n log n) due to `sorted()`, or O(n) if you just use insertion order carefully — but still needs extra space

This approach works and is easy to understand, but it doesn't respect the O(1) space constraint the problem is really testing.

---

## Approach 3: Brute Force / Shifting Elements

**Idea:** Whenever you find a duplicate, shift all subsequent elements left by one to "delete" it.

```python
def removeDuplicates(nums):
    i = 0
    while i < len(nums) - 1:
        if nums[i] == nums[i + 1]:
            nums.pop(i + 1)  # shifts everything after by one — costly
        else:
            i += 1
    return len(nums)
```

**Why it's worse:**
- Every `pop()` from the middle of a list costs O(n) in the worst case (shifting elements)
- Overall time complexity becomes **O(n²)** in the worst case (e.g., array of all duplicates)
- It does work, and it is technically in-place, but it's inefficient compared to the two-pointer approach

---

## Comparison Table

| Approach | Time | Space | In-place? | Notes |
|---|---|---|---|---|
| **Two Pointers** | O(n) | O(1) | ✅ Yes | Optimal, exploits sorted property |
| **Set/Rebuild** | O(n log n) or O(n) | O(n) | ❌ No | Simple but wastes space |
| **Shift on duplicate** | O(n²) | O(1) | ✅ Yes | Correct but slow due to repeated shifting |

---

## Why Two Pointers Is the "Right" Answer

This problem is a great teaching example for the **two-pointer / slow-fast pointer pattern**, which shows up constantly in array problems (e.g., "Remove Duplicates II" where each value can appear up to twice, "Move Zeroes", "Remove Element"). 

The core trick to remember:
> **Slow pointer (`i`)** = where to write the next "good" value.
> **Fast pointer (`j`)** = scans ahead looking for new values.

Once you internalize this pattern, a whole family of "modify array in-place" problems become much easier to solve.
