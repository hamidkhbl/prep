# 🟢 Top 10 Easy LeetCode Questions
> Full Python Solutions + Test Cases + Complexity Analysis

---

## TABLE OF CONTENTS

| # | Problem | Pattern |
|---|---|---|
| 1 | [Reverse a String](#1-reverse-a-string) | Two Pointers |
| 2 | [Palindrome Check](#2-palindrome-check) | Two Pointers |
| 3 | [FizzBuzz](#3-fizzbuzz) | Math / Modulo |
| 4 | [Single Number](#4-single-number) | Bit Manipulation |
| 5 | [Climbing Stairs](#5-climbing-stairs) | Dynamic Programming |
| 6 | [Best Time to Buy and Sell Stock](#6-best-time-to-buy-and-sell-stock) | Greedy / Sliding Window |
| 7 | [Contains Duplicate](#7-contains-duplicate) | Hash Set |
| 8 | [Missing Number](#8-missing-number) | Math / XOR |
| 9 | [Reverse Linked List](#9-reverse-linked-list) | Linked List |
| 10 | [Maximum Depth of Binary Tree](#10-maximum-depth-of-binary-tree) | Tree / Recursion |

---

## 1. Reverse a String

### Problem
> Write a function that reverses a string. The input is given as an array of characters `s`. Reverse it **in-place** with O(1) extra memory.

**Example:**
```
Input:  ["h","e","l","l","o"]
Output: ["o","l","l","e","h"]

Input:  ["H","a","n","n","a","h"]
Output: ["h","a","n","n","a","H"]
```

### Approach: Two Pointers
Swap characters from both ends, moving inward until pointers meet.

```python
def reverse_string(s: list[str]) -> None:
    left, right = 0, len(s) - 1

    while left < right:
        s[left], s[right] = s[right], s[left]
        left  += 1
        right -= 1


# Bonus: reverse a plain string (returns new string)
def reverse_str(s: str) -> str:
    return s[::-1]          # slice — Pythonic one-liner


# ── Tests ──────────────────────────────────────────────────
def test_odd_length():
    s = ["h","e","l","l","o"]
    reverse_string(s)
    assert s == ["o","l","l","e","h"]

def test_even_length():
    s = ["H","a","n","n","a","h"]
    reverse_string(s)
    assert s == ["h","a","n","n","a","H"]

def test_single_char():
    s = ["a"]
    reverse_string(s)
    assert s == ["a"]

def test_two_chars():
    s = ["a","b"]
    reverse_string(s)
    assert s == ["b","a"]

def test_empty():
    s = []
    reverse_string(s)
    assert s == []

def test_slice_shorthand():
    assert reverse_str("Python") == "nohtyP"
```

### Complexity
| | Value |
|---|---|
| Time | O(n) — visits each element once |
| Space | O(1) — in-place swap, no extra memory |

### Trace Through
```
s = ["h","e","l","l","o"]
     L=0              R=4

Step 1: swap s[0]↔s[4] → ["o","e","l","l","h"]  L=1, R=3
Step 2: swap s[1]↔s[3] → ["o","l","l","e","h"]  L=2, R=2
Step 3: L >= R → stop ✓
```

---

## 2. Palindrome Check

### Problem
> Given an integer `x`, return `True` if `x` is a palindrome, and `False` otherwise. A palindrome reads the same forward and backward. Negative numbers are **not** palindromes.

**Example:**
```
Input:  121   → True
Input:  -121  → False
Input:  10    → False
```

### Approach 1: String Conversion (Simple)
```python
def is_palindrome_str(x: int) -> bool:
    if x < 0:
        return False
    s = str(x)
    return s == s[::-1]
```

### Approach 2: Reverse Half the Number (No String, Follow-up)
Reverse only the second half of the number and compare to the first half. Avoids integer overflow in other languages and demonstrates mathematical thinking.

```python
def is_palindrome(x: int) -> bool:
    # Negatives and numbers ending in 0 (except 0 itself) are not palindromes
    if x < 0 or (x % 10 == 0 and x != 0):
        return False

    reversed_half = 0
    while x > reversed_half:
        reversed_half = reversed_half * 10 + x % 10
        x //= 10

    # Even length:  x == reversed_half
    # Odd length:   x == reversed_half // 10  (middle digit discarded)
    return x == reversed_half or x == reversed_half // 10


# ── Tests ──────────────────────────────────────────────────
def test_palindrome():
    assert is_palindrome(121) is True

def test_negative():
    assert is_palindrome(-121) is False

def test_trailing_zero():
    assert is_palindrome(10) is False

def test_zero():
    assert is_palindrome(0) is True

def test_single_digit():
    assert is_palindrome(7) is True

def test_even_palindrome():
    assert is_palindrome(1221) is True

def test_not_palindrome():
    assert is_palindrome(123) is False
```

### Complexity
| | String | Half-Reverse |
|---|---|---|
| Time | O(n digits) | O(n/2 digits) |
| Space | O(n digits) | O(1) |

### Trace Through
```
x = 1221

Loop 1: reversed_half = 0*10 + 1221%10 = 1,  x = 122
Loop 2: reversed_half = 1*10 + 122%10  = 12, x = 12
Now x(12) == reversed_half(12) → True ✓

x = 121  (odd length)
Loop 1: reversed_half = 1,  x = 12
Loop 2: reversed_half = 12, x = 1   ← x < reversed_half, stop
x(1) == reversed_half//10(1) → True ✓
```

---

## 3. FizzBuzz

### Problem
> Given an integer `n`, return a string array where:
> - `"FizzBuzz"` if divisible by both 3 and 5
> - `"Fizz"` if divisible by 3
> - `"Buzz"` if divisible by 5
> - The number as a string otherwise

**Example:**
```
Input:  n = 15
Output: ["1","2","Fizz","4","Buzz","Fizz","7","8","Fizz",
         "Buzz","11","Fizz","13","14","FizzBuzz"]
```

### Approach: String Concatenation (Scalable)
Build the output by concatenating tokens. This scales cleanly if new rules are added (e.g., "Jazz" for 7) without a growing chain of `elif`.

```python
def fizz_buzz(n: int) -> list[str]:
    result = []

    for i in range(1, n + 1):
        token = ""
        if i % 3 == 0: token += "Fizz"
        if i % 5 == 0: token += "Buzz"
        result.append(token or str(i))

    return result


# Extensible version with a rules dict
def fizz_buzz_extended(n: int, rules: dict[int, str] = None) -> list[str]:
    if rules is None:
        rules = {3: "Fizz", 5: "Buzz"}

    result = []
    for i in range(1, n + 1):
        token = "".join(label for divisor, label in rules.items() if i % divisor == 0)
        result.append(token or str(i))
    return result


# ── Tests ──────────────────────────────────────────────────
def test_standard():
    out = fizz_buzz(15)
    assert out[2]  == "Fizz"     # 3
    assert out[4]  == "Buzz"     # 5
    assert out[14] == "FizzBuzz" # 15
    assert out[0]  == "1"
    assert out[6]  == "7"

def test_length():
    assert len(fizz_buzz(20)) == 20

def test_n_equals_1():
    assert fizz_buzz(1) == ["1"]

def test_extensible_with_7():
    out = fizz_buzz_extended(21, {3:"Fizz", 5:"Buzz", 7:"Jazz"})
    assert out[6]  == "Jazz"      # 7
    assert out[20] == "FizzJazz"  # 21 = 3×7
```

### Complexity
| | Value |
|---|---|
| Time | O(n) |
| Space | O(n) — output list |

---

## 4. Single Number

### Problem
> Given a non-empty array of integers `nums`, every element appears **twice** except for one. Find that single element. Must run in O(n) time and O(1) space.

**Example:**
```
Input:  [2, 2, 1]       → 1
Input:  [4, 1, 2, 1, 2] → 4
Input:  [1]             → 1
```

### Approach: XOR Bit Trick
XOR properties:
- `a ^ a = 0` — any number XOR itself is 0
- `a ^ 0 = a` — any number XOR 0 is itself
- XOR is commutative and associative

So XOR-ing all numbers cancels every duplicate, leaving the unique one.

```python
def single_number(nums: list[int]) -> int:
    result = 0
    for num in nums:
        result ^= num
    return result


# One-liner using functools.reduce
from functools import reduce
from operator import xor

def single_number_v2(nums: list[int]) -> int:
    return reduce(xor, nums)


# ── Tests ──────────────────────────────────────────────────
def test_standard():
    assert single_number([2, 2, 1]) == 1

def test_larger():
    assert single_number([4, 1, 2, 1, 2]) == 4

def test_single_element():
    assert single_number([1]) == 1

def test_negative():
    assert single_number([-1, -1, -2]) == -2

def test_unordered():
    assert single_number([3, 1, 3, 2, 2]) == 1
```

### Complexity
| | Value |
|---|---|
| Time | O(n) |
| Space | O(1) — just one integer variable |

### Trace Through
```
nums = [4, 1, 2, 1, 2]

result = 0
0 ^ 4 = 4
4 ^ 1 = 5
5 ^ 2 = 7
7 ^ 1 = 6   ← 1 cancels
6 ^ 2 = 4   ← 2 cancels

return 4 ✓
```

---

## 5. Climbing Stairs

### Problem
> You are climbing a staircase with `n` steps. Each time you can climb **1 or 2 steps**. In how many distinct ways can you reach the top?

**Example:**
```
Input:  n = 2  → 2   (1+1 or 2)
Input:  n = 3  → 3   (1+1+1, 1+2, 2+1)
Input:  n = 5  → 8
```

### Key Insight
The number of ways to reach step `n` = ways to reach `n-1` (then take 1 step) + ways to reach `n-2` (then take 2 steps). This is the **Fibonacci sequence**.

### Approach: Dynamic Programming (Space-Optimized)
```python
def climb_stairs(n: int) -> int:
    if n <= 2:
        return n

    prev2, prev1 = 1, 2   # ways to reach step 1 and step 2

    for _ in range(3, n + 1):
        curr  = prev1 + prev2
        prev2 = prev1
        prev1 = curr

    return prev1


# Naive recursion (for understanding — O(2^n), don't use in interviews)
def climb_stairs_recursive(n: int) -> int:
    if n <= 2: return n
    return climb_stairs_recursive(n-1) + climb_stairs_recursive(n-2)


# Memoized recursion
from functools import lru_cache

@lru_cache(maxsize=None)
def climb_stairs_memo(n: int) -> int:
    if n <= 2: return n
    return climb_stairs_memo(n-1) + climb_stairs_memo(n-2)


# ── Tests ──────────────────────────────────────────────────
def test_one_step():
    assert climb_stairs(1) == 1

def test_two_steps():
    assert climb_stairs(2) == 2

def test_three_steps():
    assert climb_stairs(3) == 3

def test_five_steps():
    assert climb_stairs(5) == 8

def test_ten_steps():
    assert climb_stairs(10) == 89

def test_large():
    assert climb_stairs(45) == 1836311903
```

### Complexity
| | Naive Recursion | DP (optimized) |
|---|---|---|
| Time | O(2ⁿ) | O(n) |
| Space | O(n) call stack | O(1) |

### Trace Through
```
n = 5

prev2=1, prev1=2

i=3: curr=1+2=3,  prev2=2, prev1=3
i=4: curr=2+3=5,  prev2=3, prev1=5
i=5: curr=3+5=8,  prev2=5, prev1=8

return 8 ✓
```

---

## 6. Best Time to Buy and Sell Stock

### Problem
> You are given an array `prices` where `prices[i]` is the price of a stock on day `i`. Maximize your profit by choosing a **single day to buy** and a **different day in the future to sell**. Return the maximum profit (or `0` if no profit is possible).

**Example:**
```
Input:  [7, 1, 5, 3, 6, 4]  → 5   (buy day 2 @ 1, sell day 5 @ 6)
Input:  [7, 6, 4, 3, 1]     → 0   (always decreasing)
```

### Approach: Greedy (One Pass)
Track the minimum price seen so far. At each day, compute the profit if we sold today. Update the maximum profit.

```python
def max_profit(prices: list[int]) -> int:
    min_price  = float('inf')
    max_profit = 0

    for price in prices:
        if price < min_price:
            min_price = price               # found a cheaper buy day
        else:
            max_profit = max(max_profit, price - min_price)  # best sell today

    return max_profit


# ── Tests ──────────────────────────────────────────────────
def test_standard():
    assert max_profit([7,1,5,3,6,4]) == 5

def test_no_profit():
    assert max_profit([7,6,4,3,1]) == 0

def test_single_day():
    assert max_profit([5]) == 0

def test_two_days_profit():
    assert max_profit([1, 5]) == 4

def test_two_days_loss():
    assert max_profit([5, 1]) == 0

def test_all_same():
    assert max_profit([3,3,3,3]) == 0

def test_peak_at_end():
    assert max_profit([1,2,3,4,5]) == 4
```

### Complexity
| | Value |
|---|---|
| Time | O(n) — single pass |
| Space | O(1) |

### Trace Through
```
prices = [7, 1, 5, 3, 6, 4]

day 0: price=7  min=7   profit=max(0, 7-7)=0
day 1: price=1  min=1   profit=max(0, 1-1)=0  ← new min
day 2: price=5  min=1   profit=max(0, 5-1)=4
day 3: price=3  min=1   profit=max(4, 3-1)=4
day 4: price=6  min=1   profit=max(4, 6-1)=5  ← max
day 5: price=4  min=1   profit=max(5, 4-1)=5

return 5 ✓
```

---

## 7. Contains Duplicate

### Problem
> Given an integer array `nums`, return `True` if any value appears **at least twice**, and `False` if every element is distinct.

**Example:**
```
Input:  [1, 2, 3, 1]     → True
Input:  [1, 2, 3, 4]     → False
Input:  [1, 1, 1, 3, 3]  → True
```

### Approach: Hash Set
Add each number to a set. If it's already there — duplicate found.

```python
def contains_duplicate(nums: list[int]) -> bool:
    seen = set()
    for num in nums:
        if num in seen:
            return True
        seen.add(num)
    return False


# One-liner (compare set size to list size)
def contains_duplicate_v2(nums: list[int]) -> bool:
    return len(nums) != len(set(nums))


# ── Tests ──────────────────────────────────────────────────
def test_has_duplicate():
    assert contains_duplicate([1,2,3,1]) is True

def test_no_duplicate():
    assert contains_duplicate([1,2,3,4]) is False

def test_all_same():
    assert contains_duplicate([1,1,1,1]) is True

def test_empty():
    assert contains_duplicate([]) is False

def test_single():
    assert contains_duplicate([99]) is False

def test_large_gap():
    assert contains_duplicate([1, 1000000, 1000000]) is True
```

### Complexity
| | Value |
|---|---|
| Time | O(n) — average case (hash set lookup) |
| Space | O(n) — set stores up to n elements |

> **Interview note:** The one-liner `len(nums) != len(set(nums))` is O(n) time but always processes the full list. The loop version short-circuits on first duplicate — better for large inputs with early duplicates.

---

## 8. Missing Number

### Problem
> Given an array `nums` containing `n` distinct numbers in the range `[0, n]`, return the **one number** missing from the range.

**Example:**
```
Input:  [3, 0, 1]     → 2
Input:  [0, 1]        → 2
Input:  [9,6,4,2,3,5,7,0,1]  → 8
```

### Approach 1: Gauss Formula (O(1) space)
The sum of `[0, n]` is `n*(n+1)/2`. Subtract the actual sum to get the missing number.

```python
def missing_number(nums: list[int]) -> int:
    n = len(nums)
    expected = n * (n + 1) // 2
    return expected - sum(nums)
```

### Approach 2: XOR (O(1) space, handles large numbers safely)
XOR every index and every value. Duplicates cancel, leaving the missing number.

```python
def missing_number_xor(nums: list[int]) -> int:
    result = len(nums)   # start with n
    for i, num in enumerate(nums):
        result ^= i ^ num
    return result


# ── Tests ──────────────────────────────────────────────────
def test_standard():
    assert missing_number([3,0,1]) == 2

def test_missing_last():
    assert missing_number([0,1]) == 2

def test_missing_first():
    assert missing_number([1,2,3]) == 0

def test_large():
    assert missing_number([9,6,4,2,3,5,7,0,1]) == 8

def test_single_zero():
    assert missing_number([0]) == 1

def test_single_one():
    assert missing_number([1]) == 0
```

### Complexity
| | Value |
|---|---|
| Time | O(n) |
| Space | O(1) — no extra collections |

### Trace Through (Gauss)
```
nums = [3, 0, 1],  n = 3
expected = 3*4//2 = 6
actual   = 3+0+1  = 4
missing  = 6 - 4  = 2 ✓
```

---

## 9. Reverse Linked List

### Problem
> Given the head of a singly linked list, reverse the list and return the new head.

**Example:**
```
Input:  1 → 2 → 3 → 4 → 5 → None
Output: 5 → 4 → 3 → 2 → 1 → None
```

### Setup
```python
class ListNode:
    def __init__(self, val=0, next=None):
        self.val  = val
        self.next = next

# Helper: build list from Python list
def build_list(vals: list[int]) -> ListNode:
    if not vals: return None
    head = ListNode(vals[0])
    curr = head
    for v in vals[1:]:
        curr.next = ListNode(v)
        curr = curr.next
    return head

# Helper: flatten list to Python list
def to_list(head: ListNode) -> list[int]:
    result = []
    while head:
        result.append(head.val)
        head = head.next
    return result
```

### Approach 1: Iterative (O(1) space)
```python
def reverse_list(head: ListNode) -> ListNode:
    prev = None
    curr = head

    while curr:
        next_node  = curr.next   # save next
        curr.next  = prev        # reverse the pointer
        prev       = curr        # move prev forward
        curr       = next_node   # move curr forward

    return prev   # prev is now the new head
```

### Approach 2: Recursive
```python
def reverse_list_recursive(head: ListNode) -> ListNode:
    if not head or not head.next:
        return head

    new_head       = reverse_list_recursive(head.next)
    head.next.next = head    # node after head points back to head
    head.next      = None    # head becomes the new tail
    return new_head


# ── Tests ──────────────────────────────────────────────────
def test_standard():
    head = build_list([1,2,3,4,5])
    assert to_list(reverse_list(head)) == [5,4,3,2,1]

def test_two_nodes():
    head = build_list([1,2])
    assert to_list(reverse_list(head)) == [2,1]

def test_single_node():
    head = build_list([1])
    assert to_list(reverse_list(head)) == [1]

def test_empty():
    assert reverse_list(None) is None

def test_recursive():
    head = build_list([1,2,3])
    assert to_list(reverse_list_recursive(head)) == [3,2,1]
```

### Complexity
| | Iterative | Recursive |
|---|---|---|
| Time | O(n) | O(n) |
| Space | O(1) | O(n) call stack |

### Trace Through
```
1 → 2 → 3 → None

Step 1: prev=None, curr=1
        next=2, 1.next=None, prev=1, curr=2

Step 2: prev=1, curr=2
        next=3, 2.next=1, prev=2, curr=3
        State: 3 → None,  2 → 1 → None

Step 3: prev=2, curr=3
        next=None, 3.next=2, prev=3, curr=None
        State: 3 → 2 → 1 → None

curr is None → return prev (head=3) ✓
```

---

## 10. Maximum Depth of Binary Tree

### Problem
> Given the root of a binary tree, return its **maximum depth** — the number of nodes along the longest path from root to the farthest leaf.

**Example:**
```
      3
     / \
    9  20
       / \
      15   7

Input:  root = [3,9,20,null,null,15,7]
Output: 3
```

### Setup
```python
class TreeNode:
    def __init__(self, val=0, left=None, right=None):
        self.val   = val
        self.left  = left
        self.right = right
```

### Approach 1: Recursive DFS
```python
def max_depth(root: TreeNode) -> int:
    if not root:
        return 0
    return 1 + max(max_depth(root.left), max_depth(root.right))
```

### Approach 2: Iterative BFS (Level Order)
Count how many levels exist using a queue.

```python
from collections import deque

def max_depth_bfs(root: TreeNode) -> int:
    if not root:
        return 0

    queue = deque([root])
    depth = 0

    while queue:
        depth += 1
        for _ in range(len(queue)):   # process one level at a time
            node = queue.popleft()
            if node.left:  queue.append(node.left)
            if node.right: queue.append(node.right)

    return depth


### Approach 3: Iterative DFS (Stack)
def max_depth_dfs_iter(root: TreeNode) -> int:
    if not root:
        return 0

    stack  = [(root, 1)]
    result = 0

    while stack:
        node, depth = stack.pop()
        result = max(result, depth)
        if node.left:  stack.append((node.left,  depth + 1))
        if node.right: stack.append((node.right, depth + 1))

    return result


# ── Tests ──────────────────────────────────────────────────
def build_tree():
    #       3
    #      / \
    #     9  20
    #        / \
    #       15   7
    root       = TreeNode(3)
    root.left  = TreeNode(9)
    root.right = TreeNode(20)
    root.right.left  = TreeNode(15)
    root.right.right = TreeNode(7)
    return root

def test_standard():
    assert max_depth(build_tree()) == 3

def test_empty():
    assert max_depth(None) == 0

def test_single_node():
    assert max_depth(TreeNode(1)) == 1

def test_left_skewed():
    root = TreeNode(1)
    root.left = TreeNode(2)
    root.left.left = TreeNode(3)
    assert max_depth(root) == 3

def test_bfs_matches_dfs():
    tree = build_tree()
    assert max_depth_bfs(tree) == max_depth(build_tree())
```

### Complexity
| | Recursive DFS | BFS | Iterative DFS |
|---|---|---|---|
| Time | O(n) | O(n) | O(n) |
| Space | O(h) — height | O(w) — max width | O(h) |

> For a balanced tree: h = O(log n). For a skewed tree: h = O(n).

### Trace Through (Recursive)
```
max_depth(3)
= 1 + max(max_depth(9), max_depth(20))
= 1 + max(1, 1 + max(max_depth(15), max_depth(7)))
= 1 + max(1, 1 + max(1, 1))
= 1 + max(1, 2)
= 1 + 2
= 3 ✓
```

---

## Quick Reference Summary

| # | Problem | Pattern | Key Insight |
|---|---|---|---|
| 1 | Reverse String | Two Pointers | Swap from both ends inward |
| 2 | Palindrome Number | Math | Reverse half, compare to first half |
| 3 | FizzBuzz | Modulo | Concatenate tokens; scales to new rules |
| 4 | Single Number | XOR | `a ^ a = 0`, duplicate pairs cancel |
| 5 | Climbing Stairs | DP / Fibonacci | `f(n) = f(n-1) + f(n-2)` |
| 6 | Best Time to Buy & Sell Stock | Greedy | Track running minimum, maximize spread |
| 7 | Contains Duplicate | Hash Set | Set membership is O(1); short-circuit |
| 8 | Missing Number | Gauss / XOR | Expected sum minus actual sum |
| 9 | Reverse Linked List | Linked List | Three-pointer iterative in-place |
| 10 | Max Depth Binary Tree | DFS / BFS | Depth = 1 + max(left depth, right depth) |

---

## Patterns to Know Cold

```
Two Pointers   → works on sorted arrays, strings, palindromes
Hash Set/Map   → O(1) lookup, duplicates, frequency counting
Greedy         → local optimal = global optimal (stock, intervals)
Sliding Window → contiguous subarray/substring problems
DP             → overlapping subproblems (Fibonacci pattern)
XOR Trick      → cancel duplicates, find unique element
Recursion/DFS  → trees, graphs, nested structures
BFS            → shortest path, level-order traversal
```

---

*Last updated: April 2026 | Python 3.10+ | LeetCode #344, #9, #412, #136, #70, #121, #217, #268, #206, #104*
