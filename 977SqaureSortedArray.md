# Squares of a Sorted Array

## Problem Statement

Given an integer array `nums` sorted in **non-decreasing order**, return an array of the **squares of each number**, also sorted in non-decreasing order.

### Example 1
```
Input:  nums = [-4,-1,0,3,10]
Output: [0,1,9,16,100]
```
Explanation: After squaring, the array becomes `[16,1,0,9,100]`. After sorting, it becomes `[0,1,9,16,100]`.

### Example 2
```
Input:  nums = [-7,-3,2,3,11]
Output: [4,9,9,49,121]
```

---

## Key Insight

The tricky part: the input is sorted, but it can contain **negative numbers**. Squaring a negative number can make it "jump" far ahead in sorted order (e.g., `-10` squares to `100`, which is huge). So you **can't just square in place** and expect it to stay sorted — you need to re-sort.

The naive fix is to square everything and sort again. But there's a smarter observation:

> In the original sorted array, the **largest squared values** always come from the **elements furthest from zero** — i.e., either the most negative numbers (far left) or the most positive numbers (far right).

This means if you look at the array from **both ends inward**, at every step the larger of the two "extreme" elements produces the larger square. That's a perfect setup for the **two-pointer technique**, filling the result array **from the back (largest) to the front (smallest)**.

---

## Approach 1: Two Pointers (Optimal) ✅

**Idea:** 
Place one pointer `left` at the start, one pointer `right` at the end. Compare `abs(nums[left])` vs `abs(nums[right])`. Whichever is bigger, its square is the **largest remaining value** — place it at the end of the result array, and move that pointer inward. Repeat until the pointers meet.

### Walkthrough with `[-4,-1,0,3,10]`

Result array size 5, fill from index 4 down to 0.

| left | right | nums[left] | nums[right] | nums[left]| vs |nums[right]| Placed | pos |
|------|-------|------------|-------------|-----------|----|-----------|--------|-----|
| 0 | 4 | -4 | 10 | 4 < 10 | 100 at index 4 | right-- |
| 0 | 3 | -4 | 3 | 4 > 3 | 16 at index 3 | left++ |
| 1 | 3 | -1 | 3 | 1 < 3 | 9 at index 2 | right-- |
| 1 | 2 | -1 | 0 | 1 > 0 | 1 at index 1 | left++ |
| 2 | 2 | 0 | 0 | equal | 0 at index 0 | done |

Result: `[0, 1, 9, 16, 100]` ✅

### Code (Python)

```python
def sortedSquares(nums):
    n = len(nums)
    result = [0] * n
    left, right = 0, n - 1
    pos = n - 1  # fill from the back
    
    while left <= right:
        left_sq = nums[left] ** 2
        right_sq = nums[right] ** 2
        
        if left_sq > right_sq:
            result[pos] = left_sq
            left += 1
        else:
            result[pos] = right_sq
            right -= 1
        pos -= 1
    
    return result
```

### Code (Java)

```java
public int[] sortedSquares(int[] nums) {
    int n = nums.length;
    int[] result = new int[n];
    int left = 0, right = n - 1, pos = n - 1;
    
    while (left <= right) {
        int leftSq = nums[left] * nums[left];
        int rightSq = nums[right] * nums[right];
        
        if (leftSq > rightSq) {
            result[pos] = leftSq;
            left++;
        } else {
            result[pos] = rightSq;
            right--;
        }
        pos--;
    }
    return result;
}
```

### Code (C++)

```cpp
vector<int> sortedSquares(vector<int>& nums) {
    int n = nums.size();
    vector<int> result(n);
    int left = 0, right = n - 1, pos = n - 1;
    
    while (left <= right) {
        int leftSq = nums[left] * nums[left];
        int rightSq = nums[right] * nums[right];
        
        if (leftSq > rightSq) {
            result[pos--] = leftSq;
            left++;
        } else {
            result[pos--] = rightSq;
            right--;
        }
    }
    return result;
}
```

**Complexity:**
- Time: **O(n)** — single pass with two pointers
- Space: **O(n)** — for the output array (required anyway since we must return a new sorted array; O(1) *extra* space beyond that)

---

## Approach 2: Brute Force (Square + Sort)

**Idea:** Just square every element, then sort.

```python
def sortedSquares(nums):
    return sorted(x * x for x in nums)
```

**Why it's worse:**
- Time: **O(n log n)** due to the sort — ignores the fact that the input was already sorted, which is valuable structure we're throwing away
- It's correct and very simple, but doesn't exploit the problem's key property

---

## Approach 3: Find the Pivot (Split at Zero), Then Merge

**Idea:** Since the array is sorted, all negative numbers are on the left and non-negative on the right. Find the boundary (pivot) where numbers cross from negative to non-negative. Square both halves — the negative half, once squared, becomes **reverse-sorted**; the non-negative half is already sorted. Then **merge** the two sorted sequences (like in merge sort).

```python
def sortedSquares(nums):
    n = len(nums)
    # find first non-negative index
    pivot = 0
    while pivot < n and nums[pivot] < 0:
        pivot += 1
    
    left = pivot - 1   # last negative index, walks left (increasing square)
    right = pivot       # first non-negative index, walks right (increasing square)
    result = []
    
    while left >= 0 and right < n:
        if nums[left] ** 2 < nums[right] ** 2:
            result.append(nums[left] ** 2)
            left -= 1
        else:
            result.append(nums[right] ** 2)
            right += 1
    
    while left >= 0:
        result.append(nums[left] ** 2)
        left -= 1
    while right < n:
        result.append(nums[right] ** 2)
        right += 1
    
    return result
```

**Why it's essentially equivalent:** 
This is really the **same idea** as Approach 1, just framed differently (splitting at the negative/positive boundary instead of starting from both absolute ends). It's O(n) time too, but slightly more code and edge-case handling (all-negative or all-positive arrays) compared to the cleaner two-pointer-from-both-ends version.

---

## Comparison Table

| Approach | Time | Space | Notes |
|---|---|---|---|
| **Two Pointers (from ends)** | O(n) | O(n) output | Optimal, simplest to implement |
| **Square + Sort** | O(n log n) | O(n) | Simple but ignores sorted structure |
| **Pivot + Merge** | O(n) | O(n) | Same complexity as two pointers, more bookkeeping |

---

## Why This Problem Matters

This is a great example of a subtly different **two-pointer variant**: instead of pointers converging to *find* something (like a pair sum), here they converge while **writing the answer from the outside in**. The key generalizable lesson:

> When an operation (like squaring) can reverse the relative order of elements based on sign or magnitude, but the *extremes* of the sorted array still map predictably to the *extremes* of the transformed array, two pointers from both ends is often the right tool — filling the result from one end to the other.

## uniique approach

``` python
class Solution(object):
    def sortedSquares(self, nums):
        size = len(nums)
        pos = []
        neg = []

        '''first divide the entire 
        sorted array into two 
        positve and negative'''
        for num in nums :
            if num < 0 :
                neg.append(num)
            else :
                pos.append(num)

        '''if there is no negative
        number inside the array then
        square the array and return'''
        if len(neg) == 0:
            return [x * x for x in pos]
        
        '''if there is no positive elements 
        inside the array then sqaure the list
         and then reverse it and return '''
        if len(pos) == 0 :
            res = [x * x for x in neg][::-1]
            return res

        '''if both exist then first sqaure 
        them respectively postive as it
        is and then negative with reversing the list'''
        pos = [x*x for x in pos] 
        neg = [x*x for x in neg][::-1]

        '''store the length of the list inside the 
        varibales and then use two pointers for 
        merging two sorted arrays'''
        n, m = len(neg), len(pos)
        res = []
        i = j = 0

        while(i<n and j < m):
            if neg[i] < pos[j]:
                res.append(neg[i])
                i+=1
            else :
                res.append(pos[j])
                j+=1
        
        while i<n:
            res.append(neg[i])
            i+=1
        while j<m:
            res.append(pos[j])
            j+=1
        return res 
```

## My Code
```python
class Solution(object):
    def sortedSquares(self, nums):
        'first take the size of the list creates and result list with the size and assign the ponters left and right as 0 and n-1(at the last) create another pointer that will be placed at the last for insertion of the element from the last'
        size = len(nums)
        result = [0] * size
        left, right = 0, size-1
        pos = size -1 

        'start loop until both pointers meet'
        while(left <= right):
            'sqaure both side'
            left_sqrt = nums[left] ** 2
            right_sqrt = nums[right] ** 2

            'compare both sides square place the biggest one inside the result and then decrease right pointer or increase left pointer after the placement'
            if left_sqrt < right_sqrt:
                result[pos] = right_sqrt
                right-=1
            else:
                result[pos] = left_sqrt
                left+=1
            'decrease the position from the last to fill more'
            pos -=1
        'return the result'
        return result
```
