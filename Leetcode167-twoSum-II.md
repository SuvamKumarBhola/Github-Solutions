# LeetCode 167: Two Sum II - Input Array Is Sorted

## The Problem

You're given an array of integers that's **already sorted in ascending order**, plus a `target` number. You need to find two numbers in the array that add up to `target`, and return their **1-indexed** positions (not 0-indexed, which is a common trick/gotcha in this problem).

**Example:**
```
numbers = [2, 7, 11, 15], target = 9
```
Here, `2 + 7 = 9`, and since `2` is at position 1 and `7` is at position 2 (1-indexed), the answer is `[1, 2]`.

**Constraints worth noting:**
- Exactly one solution exists.
- You can't use the same element twice.
- You must use only constant extra space, O(1) — this is the hint that tells you not to use a hashmap here (even though a hashmap would work for the general "Two Sum" problem).

## The Key Insight: Why Sorted Order Matters

Because the array is sorted, you can use the **two-pointer technique** instead of a hashmap. This is what makes this version of the problem special.

Think of it like this:
- Put one pointer (`left`) at the very start of the array (smallest number).
- Put another pointer (`right`) at the very end (largest number).
- Add the two numbers at these pointers.

Now reason about the sum:
- **If the sum is too small** (less than target), you need a *bigger* number. Since the array is sorted, moving `left` forward gives you a bigger number. So move `left` one step to the right.
- **If the sum is too big** (greater than target), you need a *smaller* number. Moving `right` backward gives you a smaller number. So move `right` one step to the left.
- **If the sum equals target**, you found it! Return the positions.

You keep doing this, and the pointers move toward each other, shrinking the search space each time — until they meet or cross, or (per the problem's guarantee) until you find the answer.

### Walkthrough Example

```
numbers = [2, 7, 11, 15], target = 9

left = 0 (value 2), right = 3 (value 15)
sum = 2 + 15 = 17 → too big → move right left
      
left = 0 (value 2), right = 2 (value 11)
sum = 2 + 11 = 13 → too big → move right left

left = 0 (value 2), right = 1 (value 7)
sum = 2 + 7 = 9 → match! → return [1, 2] (1-indexed)
```

### Why This Works (and is Efficient)

- **Time complexity:** O(n) — each step moves one pointer inward, so in the worst case you scan the array once.
- **Space complexity:** O(1) — you only use two integer variables, no extra data structure.

This is strictly better than the hashmap approach (which is O(n) time but O(n) space) *because* the array being sorted gives you this shortcut.

---

## Python Code

```python
def twoSum(numbers, target):
    left, right = 0, len(numbers) - 1
    
    while left < right:
        current_sum = numbers[left] + numbers[right]
        
        if current_sum == target:
            return [left + 1, right + 1]  # +1 for 1-indexed result
        elif current_sum < target:
            left += 1   # need a bigger sum, move left pointer up
        else:
            right -= 1  # need a smaller sum, move right pointer down
    
    return [left+1, right+1]  # no solution found (won't happen per problem constraints)
```

## Java Code

```java
class Solution {
    public int[] twoSum(int[] numbers, int target) {
        int left = 0, right = numbers.length - 1;
        
        while (left < right) {
            int currentSum = numbers[left] + numbers[right];
            
            if (currentSum == target) {
                return new int[] {left + 1, right + 1}; // 1-indexed
            } else if (currentSum < target) {
                left++;   // need a bigger sum
            } else {
                right--;  // need a smaller sum
            }
        }
        
        return new int[] {-1, -1}; // no solution found (shouldn't happen)
    }
}
```

---

## Quick Recap of the Logic

| Condition | Action | Why |
|---|---|---|
| `sum == target` | Return indices | Found it |
| `sum < target` | `left++` | Sum too small, need a bigger number |
| `sum > target` | `right--` | Sum too big, need a smaller number |

The two-pointer trick is a pattern you'll see often in sorted-array problems — anytime you see "sorted array" plus "find a pair/triplet with some sum property," think of pointers converging from both ends instead of nested loops or extra memory.

## My Solution

```python
def twoSum(self, numbers, target):
        left, right = 0, len(numbers)-1
        while left < right :
            sum = numbers[left] + numbers[right]
            if sum==target:
                return[left+1,right+1]
            elif sum<target:
                left+=1
            else:
                right-=1
        return[left+1,right+1]
```