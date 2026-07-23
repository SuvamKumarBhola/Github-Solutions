# LeetCode 1: Two Sum

## The Problem

You're given an array of integers `nums` (this time **not necessarily sorted**) and a `target`. Find the indices of the two numbers that add up to `target`, and return them as an array (this time **0-indexed**, unlike Two Sum II).

**Example:**
```
nums = [2, 7, 11, 15], target = 9
```
`nums[0] + nums[1] = 2 + 7 = 9` → return `[0, 1]`

**Constraints:**
- Exactly one solution exists.
- You can't use the same element twice.
- You can return the answer in any order.

## Why the Two-Pointer Trick Doesn't Work Here

In problem 167, we relied on the array being **sorted** to know which direction to move our pointers. Here, the array is **unsorted**, so there's no guarantee that moving left or right gives you a bigger or smaller number. The two-pointer approach breaks down.

So instead, we use a different tool: a **hash map (dictionary)**.

## The Key Insight: "What Do I Still Need?"

As you walk through the array one number at a time, ask yourself: *"What number would I need to see, combined with the number I'm looking at right now, to hit the target?"*

That missing number is simply:
```
complement = target - current_number
```

So the strategy becomes:
1. Walk through the array once, number by number.
2. For each number, calculate its `complement` (the number that would complete the pair).
3. Check: *"Have I already seen this complement earlier in the array?"*
   - If **yes** — great, you've found your pair! Return the complement's stored index and the current index.
   - If **no** — store the current number and its index in a hashmap, so future numbers can check against it.

This way, you only need to scan the array **once**, and checking "have I seen this before" is instant (O(1)) thanks to the hashmap.

### Walkthrough Example

```
nums = [2, 7, 11, 15], target = 9
seen = {}  (empty hashmap: number → index)

i=0, num=2:  complement = 9 - 2 = 7 → is 7 in seen? No → store seen[2] = 0
i=1, num=7:  complement = 9 - 7 = 2 → is 2 in seen? Yes! (seen[2] = 0)
             → return [0, 1]
```

Notice we never had to look ahead or sort anything — by the time we reach `7`, we've already remembered that `2` showed up at index `0`.

### A Trickier Example (to show why we check *before* inserting)

```
nums = [3, 3], target = 6

i=0, num=3: complement = 6 - 3 = 3 → is 3 in seen? No (seen is empty) → store seen[3] = 0
i=1, num=3: complement = 6 - 3 = 3 → is 3 in seen? Yes! (seen[3] = 0) → return [0, 1]
```

This is why the order matters: **check for the complement first, then insert the current number** — otherwise you might accidentally pair a number with itself in a single step incorrectly (e.g. if you inserted before checking, you could still get lucky here, but the check-first order is the safe, correct habit, especially in edge cases with more repeats).

### Complexity

- **Time complexity:** O(n) — one pass through the array, O(1) hashmap lookups/inserts.
- **Space complexity:** O(n) — in the worst case, you store almost all elements in the hashmap before finding a match.

This trades space for the ability to handle **unsorted** input in linear time — a classic tradeoff compared to Two Sum II's O(1) space but O(n log n) or O(n) time (sorted) approach.

---

## Python Code

```python
def twoSum(nums, target):
    seen = {}  # value -> index
    
    for i, num in enumerate(nums):
        complement = target - num
        
        if complement in seen:
            return [seen[complement], i]
        
        seen[num] = i
    
    return []  # no solution found (won't happen per problem constraints)
```

## Java Code

```java
import java.util.HashMap;
import java.util.Map;

class Solution {
    public int[] twoSum(int[] nums, int target) {
        Map<Integer, Integer> seen = new HashMap<>(); // value -> index
        
        for (int i = 0; i < nums.length; i++) {
            int complement = target - nums[i];
            
            if (seen.containsKey(complement)) {
                return new int[] {seen.get(complement), i};
            }
            
            seen.put(nums[i], i);
        }
        
        return new int[] {-1, -1}; // no solution found (shouldn't happen)
    }
}
```

---

## Comparing to Two Sum II

| | Two Sum (this one) | Two Sum II (sorted) |
|---|---|---|
| Input | Unsorted | Sorted |
| Technique | Hashmap | Two pointers |
| Time | O(n) | O(n) |
| Space | O(n) | O(1) |
| Index base | 0-indexed | 1-indexed |

The big lesson here: **the same underlying problem can demand completely different techniques depending on one property of the input** (sorted vs. unsorted). Whenever you see "sorted array" in a problem, it's often a strong hint that a hashmap-heavy solution can be replaced by a leaner two-pointer one.