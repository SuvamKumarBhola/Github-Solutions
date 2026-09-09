# LeetCode 287: Find the Duplicate Number

## The Problem

You receive an array `nums` with `n + 1` integers. Every value is in the inclusive range `[1, n]`, and exactly one value appears more than once. Return that repeated value.

Two requirements make this problem more interesting:

- Do **not** modify `nums`.
- Use only **constant extra space**.

### Constraints

- `1 <= n <= 10^5`
- `nums.length == n + 1`
- `1 <= nums[i] <= n`
- Every integer appears once except for one integer that appears two or more times.

### Examples

```text
Input:  nums = [1, 3, 4, 2, 2]
Output: 2
```

```text
Input:  nums = [3, 1, 3, 4, 2]
Output: 3
```

```text
Input:  nums = [3, 3, 3, 3, 3]
Output: 3
```

## Why a Duplicate Must Exist

There are `n + 1` array positions, but only `n` possible values (`1` through `n`). By the pigeonhole principle, at least two positions must contain the same value. The constraints also tell us there is exactly one duplicated *value*, even though that value may occur more than twice.

---

## Approach 1: Compare Every Pair (Brute Force)

### Idea

Check each value against every value to its right. As soon as two equal values are found, return that value.

### Steps

1. Choose an index `i`.
2. Compare `nums[i]` with every later element.
3. If any value matches, return it.

### Walkthrough

For `nums = [1, 3, 4, 2, 2]`:

```text
1 compared with 3, 4, 2, 2 -> no match
3 compared with 4, 2, 2    -> no match
4 compared with 2, 2       -> no match
2 compared with 2          -> match: return 2
```

### Python Solution

```python
class Solution:
    def findDuplicate(self, nums: list[int]) -> int:
        for i in range(len(nums)):
            for j in range(i + 1, len(nums)):
                if nums[i] == nums[j]:
                    return nums[i]
```

### Complexity

- Time: `O(n^2)`
- Extra space: `O(1)`

It preserves the array and uses constant space, but it is far too slow for large inputs.

---

## Approach 2: Track Seen Values with a Set

### Idea

Walk through the array once. If a value has already been seen, it is the duplicate; otherwise add it to a set.

### Python Solution

```python
class Solution:
    def findDuplicate(self, nums: list[int]) -> int:
        seen = set()

        for number in nums:
            if number in seen:
                return number
            seen.add(number)
```

### Complexity

- Time: `O(n)` on average
- Extra space: `O(n)`

This is a simple linear-time solution, but the set violates the problem's constant-extra-space requirement.

---

## Approach 3: Binary Search on the Value Range

### Key Insight

Binary search does not have to search array indices. Here, it searches the possible **values** from `1` to `n`.

For a candidate midpoint `mid`, count how many array elements are less than or equal to `mid`.

- There are only `mid` distinct permitted values in `[1, mid]`.
- If more than `mid` numbers are `<= mid`, the duplicate must be in `[1, mid]`.
- Otherwise, it must be in `[mid + 1, n]`.

This is again the pigeonhole principle, applied to a smaller value range.

### Walkthrough

For `nums = [1, 3, 4, 2, 2]`, the possible values are `1..4`.

```text
low = 1, high = 4, mid = 2
count(values <= 2) = 3  (1, 2, 2)
3 > 2, so the duplicate is in [1, 2]

low = 1, high = 2, mid = 1
count(values <= 1) = 1
1 is not greater than 1, so the duplicate is in [2, 2]

return 2
```

### Python Solution

```python
class Solution:
    def findDuplicate(self, nums: list[int]) -> int:
        low, high = 1, len(nums) - 1

        while low < high:
            mid = (low + high) // 2
            count = sum(number <= mid for number in nums)

            if count > mid:
                high = mid
            else:
                low = mid + 1

        return low
```

### Complexity

- Time: `O(n log n)`: there are `O(log n)` value-range steps, and each count scans the array.
- Extra space: `O(1)`

This satisfies both follow-up requirements but is slower than a linear-time solution.

---

## Approach 4: Floyd's Tortoise and Hare (Optimal)

### Turn the Array into a Linked List

Treat each array value as the index of the next node:

```text
next(i) = nums[i]
```

This is valid because every `nums[i]` is in `[1, n]`, which is always a valid index. Start from index `0` and keep following values.

For `nums = [1, 3, 4, 2, 2]`, the path is:

```text
0 -> 1 -> 3 -> 2 -> 4 -> 2 -> 4 -> ...
```

The repeated value `2` is the entrance to the cycle. A duplicate creates this cycle because two different indices point to the same next index.

### Floyd's Two Phases

1. Move `slow` one step and `fast` two steps until they meet. A meeting proves that there is a cycle.
2. Reset one pointer to the start. Move both pointers one step at a time. Their next meeting is the cycle entrance—the duplicate number.

### Walkthrough

For `nums = [1, 3, 4, 2, 2]`:

```text
Path from 0: 0 -> 1 -> 3 -> 2 -> 4 -> 2 -> ...

Phase 1: slow and fast meet somewhere in the 2 <-> 4 cycle.
Phase 2: reset slow to 0; advance both one step at a time.
They meet at 2, the entrance to the cycle and the duplicate value.
```

### Why Phase 2 Finds the Duplicate

Suppose the path from `0` to the cycle entrance has length `a`, the distance from the entrance to the first pointer meeting point is `b`, and the cycle length is `c`. At the meeting, the fast pointer has travelled twice as far as the slow pointer. That means the extra distance travelled by `fast` is a whole number of cycles, which implies that moving one pointer from the start and one from the meeting point by the same number of steps makes them meet at the cycle entrance.

### Python Solution

```python
class Solution:
    def findDuplicate(self, nums: list[int]) -> int:
        # Phase 1: find an intersection inside the cycle.
        slow = nums[0]
        fast = nums[0]

        while True:
            slow = nums[slow]
            fast = nums[nums[fast]]
            if slow == fast:
                break

        # Phase 2: find the cycle entrance.
        slow = nums[0]
        while slow != fast:
            slow = nums[slow]
            fast = nums[fast]

        return slow
```

### Complexity

- Time: `O(n)`
- Extra space: `O(1)`

This is the optimal solution: it runs in linear time, never changes the input, and meets the constant-extra-space constraint.

---

## Edge Cases and Common Pitfalls

- **The duplicate may appear more than twice.** The algorithm still works, for example `[3, 3, 3, 3, 3]` returns `3`.
- **Do not sort `nums` in place.** Sorting would modify the input and breaks the stated requirement.
- **Do not use index `0` as a possible next value.** Values are always in `[1, n]`; index `0` is only the starting point that leads into the linked-list structure.
- **Do not confuse a duplicate index with a duplicate value.** The answer is the repeated value, which is also the cycle entrance in Floyd's interpretation.
- **The hash-set solution is not accepted by the follow-up requirement.** It is correct but uses linear extra memory.

## Approach Comparison

| Approach | Time | Extra space | Modifies `nums`? | Meets all requirements? |
| --- | --- | --- | --- | --- |
| Pair comparison | `O(n^2)` | `O(1)` | No | No—too slow |
| Hash set | `O(n)` average | `O(n)` | No | No—uses extra space |
| Value-range binary search | `O(n log n)` | `O(1)` | No | Yes |
| Floyd's cycle detection | `O(n)` | `O(1)` | No | Yes—optimal |

The value-range binary-search method is a useful application of the pigeonhole principle. Floyd's algorithm is the best final choice because it improves the running time to `O(n)` without giving up constant extra space.
