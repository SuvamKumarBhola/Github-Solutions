# LeetCode 202: Happy Number

## The Problem

Given a positive integer `n`, repeatedly replace it with the sum of the squares of its digits:

```text
next(n) = (each digit of n)² added together
```

The number is **happy** if this process eventually reaches `1`. If the process repeats a value before reaching `1`, it is trapped in a cycle and is not happy.

Return `true` for a happy number and `false` otherwise.

### Constraints

- `1 <= n <= 2^31 - 1`

### Examples

```text
Input: n = 19
Output: true

19 -> 82 -> 68 -> 100 -> 1
```

```text
Input: n = 2
Output: false

2 -> 4 -> 16 -> 37 -> 58 -> 89 -> 145 -> 42 -> 20 -> 4 -> ...
```

## A Small Helper

Every approach needs a function that produces the next number in the sequence.

```python
def next_number(n: int) -> int:
    total = 0

    while n > 0:
        digit = n % 10
        total += digit * digit
        n //= 10

    return total
```

For example, `next_number(19)` is `1² + 9² = 82`.

---

## Approach 1: Simulate and Search a List (Brute Force)

### Idea

Keep every number already produced in a list. At each step:

1. If the current number is `1`, return `True`.
2. Search the list to see whether the current number has appeared before.
3. If it has, the sequence is cycling, so return `False`.
4. Otherwise, add it to the list and continue with its next value.

This is the most direct implementation of the definition. Its drawback is that testing `n in seen` scans the list from beginning to end.

### Walkthrough for `n = 19`

```text
current = 19, seen = []       -> remember 19; next is 82
current = 82, seen = [19]     -> remember 82; next is 68
current = 68, seen = [19, 82] -> remember 68; next is 100
current = 100                 -> next is 1
current = 1                   -> happy
```

### Python Solution

```python
class Solution:
    def isHappy(self, n: int) -> bool:
        seen = []

        while n != 1:
            if n in seen:
                return False

            seen.append(n)
            n = self.next_number(n)

        return True

    def next_number(self, n: int) -> int:
        total = 0
        while n > 0:
            digit = n % 10
            total += digit * digit
            n //= 10
        return total
```

### Complexity

Let `k` be the number of sequence values examined before reaching `1` or detecting a repeated value, and let `d` be the maximum number of digits in a value examined.

- Time: `O(k² + k * d)` because each list membership check can scan up to `k` values.
- Space: `O(k)` for the list.

---

## Approach 2: Use a Hash Set

### Idea

The repeated-value check is the expensive part of the brute-force approach. A set gives average `O(1)` membership checks instead of a linear scan.

The algorithm is otherwise identical:

1. While `n` is not `1`, check whether it is already in `seen`.
2. A repeated value means all later values will repeat in the same loop, so return `False`.
3. Add `n` to `seen` and calculate the next number.
4. Reaching `1` means return `True`.

### Why a Repeat Means Failure

The next number is determined entirely by the current number. Therefore, if we arrive at a value seen earlier, the following sequence will be exactly the same as it was after that earlier visit. Since we did not reach `1` first, we never will.

### Python Solution

```python
class Solution:
    def isHappy(self, n: int) -> bool:
        seen = set()

        while n != 1 and n not in seen:
            seen.add(n)
            n = self.next_number(n)

        return n == 1

    def next_number(self, n: int) -> int:
        total = 0
        while n > 0:
            digit = n % 10
            total += digit * digit
            n //= 10
        return total
```

### Complexity

- Time: `O(k * d)` on average. Each of the `k` transformations processes at most `d` digits.
- Space: `O(k)` for the set of visited values.

---

## Approach 3: Floyd's Cycle Detection (Optimal Space)

### Key Insight

The sequence is like a linked list:

```text
n -> next(n) -> next(next(n)) -> ...
```

It either reaches `1` or enters a cycle. Floyd's tortoise-and-hare algorithm detects a cycle without storing all prior values:

- `slow` moves one transformation at a time.
- `fast` moves two transformations at a time.
- If the sequence has a cycle, the two pointers must eventually meet inside it.
- If either pointer reaches `1`, the original number is happy.

### Walkthrough for `n = 2`

```text
Start: slow = 2, fast = 2
Move:  slow = 4,  fast = 16
Move:  slow = 16, fast = 58
...
Eventually slow and fast meet inside the cycle:
4 -> 16 -> 37 -> 58 -> 89 -> 145 -> 42 -> 20 -> 4

They met before reaching 1, so return false.
```

### Python Solution

```python
class Solution:
    def isHappy(self, n: int) -> bool:
        slow = n
        fast = self.next_number(n)

        while fast != 1 and slow != fast:
            slow = self.next_number(slow)
            fast = self.next_number(self.next_number(fast))

        return fast == 1

    def next_number(self, n: int) -> int:
        total = 0
        while n > 0:
            digit = n % 10
            total += digit * digit
            n //= 10
        return total
```

### Complexity

- Time: `O(k * d)`. The pointers visit a linear number of sequence states, and each transformation processes digits.
- Space: `O(1)`. Only a fixed number of integer variables is used.

This is the optimal approach when extra space matters: it has the same asymptotic running time as the hash-set solution while using constant auxiliary space.

---

## Edge Cases and Pitfalls

- **`n = 1`:** It is already happy. The Floyd solution handles it because `fast` starts at `next(1)`, which is still `1`.
- **Do not stop at a single digit:** `7` is happy even though it is a single digit: `7 -> 49 -> 97 -> 130 -> 10 -> 1`.
- **Check for a cycle, not for a guessed iteration limit:** A fixed limit may accidentally work for a constraint range but does not prove the algorithm is correct.
- **Use integer division:** In Python, use `n //= 10`, not `/= 10`, so the value remains an integer.
- **Why termination is guaranteed:** After one transformation, even a 10-digit integer maps to at most `10 * 9² = 810`. From then on the sequence stays in a finite set, so it must reach `1` or repeat.

## Approach Comparison

| Approach | Cycle detection | Time | Extra space | When to use |
| --- | --- | --- | --- | --- |
| List simulation | Linear list search | `O(k² + k*d)` | `O(k)` | First-principles learning only |
| Hash set | Average constant-time lookup | `O(k*d)` | `O(k)` | Clearest practical solution |
| Floyd's cycle detection | Slow/fast pointers | `O(k*d)` | `O(1)` | Best when constant extra space is desired |

The hash-set version is often the easiest to explain in an interview. Floyd's algorithm is the space-optimal refinement once you recognize that repeatedly applying `next_number` forms a cycle-detection problem.
