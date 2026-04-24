# 🧩 Top 10 LeetCode Questions for SDET Interviews
> Full Python Solutions + SDET Context + Complexity Analysis

---

## WHY LEETCODE IN SDET INTERVIEWS?

SDETs are expected to write **production-quality automation code**, design algorithms for test data generation, parse logs, and build efficient tooling. Interview coding rounds are typically **Easy–Medium** difficulty, focused on:

- String/array manipulation (log parsing, test data)
- Hash maps (deduplication, frequency analysis)
- Sliding window (performance metrics)
- Trees/graphs (dependency resolution, DOM traversal)
- Recursion (nested config/JSON traversal)

---

## TABLE OF CONTENTS

| # | Problem | Difficulty | Pattern |
|---|---|---|---|
| 1 | [Two Sum](#1-two-sum) | Easy | Hash Map |
| 2 | [Valid Parentheses](#2-valid-parentheses) | Easy | Stack |
| 3 | [Longest Substring Without Repeating Characters](#3-longest-substring-without-repeating-characters) | Medium | Sliding Window |
| 4 | [Group Anagrams](#4-group-anagrams) | Medium | Hash Map |
| 5 | [Valid Anagram](#5-valid-anagram) | Easy | Hash Map / Sort |
| 6 | [Merge Intervals](#6-merge-intervals) | Medium | Sorting + Greedy |
| 7 | [Find All Duplicates in an Array](#7-find-all-duplicates-in-an-array) | Medium | Array / Cycle |
| 8 | [Binary Search](#8-binary-search) | Easy | Divide & Conquer |
| 9 | [Maximum Subarray](#9-maximum-subarray) | Medium | Kadane's Algorithm |
| 10 | [LRU Cache](#10-lru-cache) | Medium | Design / LinkedHashMap |

---

## 1. Two Sum

### Problem
> Given an array of integers `nums` and an integer `target`, return the **indices** of the two numbers that add up to `target`. Assume exactly one solution exists. You may not use the same element twice.

**Example:**
```
Input:  nums = [2, 7, 11, 15], target = 9
Output: [0, 1]   # nums[0] + nums[1] = 2 + 7 = 9
```

### SDET Context
> Appears in log deduplication, finding paired test case IDs, or matching request/response pairs by a combined hash.

### Approach: Hash Map (One Pass)
Store each number's index as we iterate. For each number, check if its complement (`target - num`) is already in the map.

```python
def two_sum(nums: list[int], target: int) -> list[int]:
    seen = {}  # value → index

    for i, num in enumerate(nums):
        complement = target - num
        if complement in seen:
            return [seen[complement], i]
        seen[num] = i

    return []  # no solution (problem guarantees one exists)


# ── Tests ──────────────────────────────────────────────────
def test_two_sum_basic():
    assert two_sum([2, 7, 11, 15], 9) == [0, 1]

def test_two_sum_unordered():
    assert two_sum([3, 2, 4], 6) == [1, 2]

def test_two_sum_duplicates():
    assert two_sum([3, 3], 6) == [0, 1]

def test_two_sum_negative():
    assert two_sum([-1, -2, -3, -4, -5], -8) == [2, 4]
```

### Complexity
| | Value |
|---|---|
| Time | O(n) — single pass |
| Space | O(n) — hash map stores up to n entries |

### Trace Through
```
nums = [2, 7, 11, 15], target = 9

i=0: num=2,  complement=7,  seen={}        → 7 not in seen → seen={2:0}
i=1: num=7,  complement=2,  seen={2:0}     → 2 IS in seen  → return [0, 1] ✓
```

---

## 2. Valid Parentheses

### Problem
> Given a string `s` containing only `(`, `)`, `{`, `}`, `[`, `]`, determine if the input string is valid. An input string is valid if:
> - Open brackets are closed by the same type of bracket
> - Open brackets are closed in the correct order
> - Every close bracket has a corresponding open bracket

**Example:**
```
Input:  "()[]{}"  → True
Input:  "([)]"    → False
Input:  "{[]}"    → True
Input:  "("       → False
```

### SDET Context
> Validates JSON/XML structure, checks balanced test config files, verifies template syntax, or parses log message boundaries.

### Approach: Stack
Push opening brackets onto a stack. When a closing bracket is encountered, the top of the stack must be its matching opener.

```python
def is_valid(s: str) -> bool:
    stack = []
    closing = {')': '(', '}': '{', ']': '['}

    for char in s:
        if char in closing:
            # Stack must be non-empty AND top must match
            if not stack or stack[-1] != closing[char]:
                return False
            stack.pop()
        else:
            stack.append(char)  # opening bracket

    return len(stack) == 0  # valid only if nothing left unmatched


# ── Tests ──────────────────────────────────────────────────
def test_valid_mixed():
    assert is_valid("()[]{}") is True

def test_valid_nested():
    assert is_valid("{[()]}") is True

def test_invalid_interleaved():
    assert is_valid("([)]") is False

def test_invalid_unclosed():
    assert is_valid("(") is False

def test_empty_string():
    assert is_valid("") is True

def test_only_closing():
    assert is_valid(")") is False
```

### Complexity
| | Value |
|---|---|
| Time | O(n) — single pass through string |
| Space | O(n) — worst case all opening brackets on stack |

### Trace Through
```
s = "{[]}"

char='{'  → opening → stack=['{']
char='['  → opening → stack=['{','[']
char=']'  → closing → closing[']']='[', top='[' ✓ → stack=['{']
char='}'  → closing → closing['}']= '{', top='{' ✓ → stack=[]
return len([]) == 0 → True ✓
```

---

## 3. Longest Substring Without Repeating Characters

### Problem
> Given a string `s`, find the length of the **longest substring** without repeating characters.

**Example:**
```
Input:  "abcabcbb"  → 3   ("abc")
Input:  "bbbbb"     → 1   ("b")
Input:  "pwwkew"    → 3   ("wke")
```

### SDET Context
> Finding the longest unique log token sequence, extracting non-repeating test IDs from a stream, or detecting unique event windows in telemetry data.

### Approach: Sliding Window + Hash Map
Maintain a window `[left, right]`. Track the last seen index of each character. When a duplicate is found, move `left` past the previous occurrence.

```python
def length_of_longest_substring(s: str) -> int:
    last_seen = {}   # char → most recent index
    left = 0
    max_len = 0

    for right, char in enumerate(s):
        # If char was seen and is inside the current window, shrink from left
        if char in last_seen and last_seen[char] >= left:
            left = last_seen[char] + 1

        last_seen[char] = right
        max_len = max(max_len, right - left + 1)

    return max_len


# ── Tests ──────────────────────────────────────────────────
def test_standard():
    assert length_of_longest_substring("abcabcbb") == 3

def test_all_same():
    assert length_of_longest_substring("bbbbb") == 1

def test_end_window():
    assert length_of_longest_substring("pwwkew") == 3

def test_empty():
    assert length_of_longest_substring("") == 0

def test_single_char():
    assert length_of_longest_substring("a") == 1

def test_no_repeats():
    assert length_of_longest_substring("abcdef") == 6
```

### Complexity
| | Value |
|---|---|
| Time | O(n) — each character visited at most twice |
| Space | O(min(n, m)) — m = charset size (26 for lowercase alpha) |

### Trace Through
```
s = "abcabcbb"

right=0 char='a'  → seen={a:0}  left=0  window="a"    len=1
right=1 char='b'  → seen={a:0,b:1}  window="ab"       len=2
right=2 char='c'  → seen={...,c:2}  window="abc"      len=3  ← max
right=3 char='a'  → 'a' at 0 >= left(0) → left=1
                  → seen={a:3,...}  window="bca"       len=3
right=4 char='b'  → 'b' at 1 >= left(1) → left=2
                  → window="cab"                       len=3
...
return 3 ✓
```

---

## 4. Group Anagrams

### Problem
> Given an array of strings, group the anagrams together. Return the groups in any order.

**Example:**
```
Input:  ["eat","tea","tan","ate","nat","bat"]
Output: [["bat"],["nat","tan"],["ate","eat","tea"]]
```

### SDET Context
> Grouping test cases by equivalent input permutations, deduplicating test data with same characters in different orders, or clustering log entries by token signature.

### Approach: Sorted Key Hash Map
Two strings are anagrams if and only if their sorted characters are identical. Use `tuple(sorted(word))` as the grouping key.

```python
from collections import defaultdict

def group_anagrams(strs: list[str]) -> list[list[str]]:
    groups = defaultdict(list)

    for word in strs:
        key = tuple(sorted(word))   # "eat" → ('a','e','t')
        groups[key].append(word)

    return list(groups.values())


# ── Tests ──────────────────────────────────────────────────
def test_standard():
    result = group_anagrams(["eat", "tea", "tan", "ate", "nat", "bat"])
    # Sort inner lists and outer list for deterministic comparison
    normalized = sorted([sorted(g) for g in result])
    expected   = sorted([sorted(g) for g in [["bat"],["nat","tan"],["ate","eat","tea"]]])
    assert normalized == expected

def test_single_word():
    assert group_anagrams(["abc"]) == [["abc"]]

def test_all_unique():
    result = group_anagrams(["abc", "def", "ghi"])
    assert len(result) == 3

def test_empty_strings():
    result = group_anagrams(["", ""])
    assert result == [["", ""]]

def test_single_char():
    result = group_anagrams(["a"])
    assert result == [["a"]]
```

### Complexity
| | Value |
|---|---|
| Time | O(n · k log k) — n words, k = max word length |
| Space | O(n · k) — storing all words in the map |

### Alternative: Character Count Key (O(n·k))
```python
def group_anagrams_v2(strs: list[str]) -> list[list[str]]:
    groups = defaultdict(list)
    for word in strs:
        count = [0] * 26
        for c in word:
            count[ord(c) - ord('a')] += 1
        groups[tuple(count)].append(word)
    return list(groups.values())
```

---

## 5. Valid Anagram

### Problem
> Given two strings `s` and `t`, return `True` if `t` is an anagram of `s`, and `False` otherwise.

**Example:**
```
Input:  s="anagram", t="nagaram"  → True
Input:  s="rat",     t="car"      → False
```

### SDET Context
> Validating that a transformed string output contains exactly the same characters (e.g., encryption reversibility, shuffle-and-verify test data), or checking two log token sets are equivalent.

### Approach 1: Sort (Simple, O(n log n))
```python
def is_anagram_sort(s: str, t: str) -> bool:
    return sorted(s) == sorted(t)
```

### Approach 2: Counter (O(n), preferred)
```python
from collections import Counter

def is_anagram(s: str, t: str) -> bool:
    if len(s) != len(t):
        return False
    return Counter(s) == Counter(t)


# Manual counter without Counter (shows understanding)
def is_anagram_manual(s: str, t: str) -> bool:
    if len(s) != len(t):
        return False
    count = {}
    for c in s: count[c] = count.get(c, 0) + 1
    for c in t:
        if c not in count or count[c] == 0:
            return False
        count[c] -= 1
    return True


# ── Tests ──────────────────────────────────────────────────
def test_valid():
    assert is_anagram("anagram", "nagaram") is True

def test_invalid():
    assert is_anagram("rat", "car") is False

def test_different_lengths():
    assert is_anagram("ab", "abc") is False

def test_empty_strings():
    assert is_anagram("", "") is True

def test_repeated_chars():
    assert is_anagram("aaab", "aaba") is True

def test_unicode():
    # Follow-up: handle Unicode (Counter handles it natively)
    assert is_anagram("你好", "好你") is True
```

### Complexity
| | Sort | Counter |
|---|---|---|
| Time | O(n log n) | O(n) |
| Space | O(n) | O(1) for fixed alphabet, O(n) for Unicode |

---

## 6. Merge Intervals

### Problem
> Given an array of `intervals` where `intervals[i] = [start, end]`, merge all overlapping intervals and return the result.

**Example:**
```
Input:  [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]

Input:  [[1,4],[4,5]]
Output: [[1,5]]   # touching intervals merge
```

### SDET Context
> Merging overlapping time windows in performance test results, consolidating overlapping test execution slots, collapsing redundant date ranges in test schedules, or merging log timestamp ranges.

### Approach: Sort + Linear Merge
Sort by start time. Greedily merge the current interval with the last merged one if they overlap.

```python
def merge(intervals: list[list[int]]) -> list[list[int]]:
    if not intervals:
        return []

    intervals.sort(key=lambda x: x[0])   # sort by start time
    merged = [intervals[0]]

    for start, end in intervals[1:]:
        last_end = merged[-1][1]

        if start <= last_end:             # overlap (or touching)
            merged[-1][1] = max(last_end, end)   # extend if needed
        else:
            merged.append([start, end])   # no overlap, new interval

    return merged


# ── Tests ──────────────────────────────────────────────────
def test_standard():
    assert merge([[1,3],[2,6],[8,10],[15,18]]) == [[1,6],[8,10],[15,18]]

def test_touching():
    assert merge([[1,4],[4,5]]) == [[1,5]]

def test_contained():
    assert merge([[1,10],[2,3],[4,5]]) == [[1,10]]

def test_no_overlap():
    assert merge([[1,2],[3,4],[5,6]]) == [[1,2],[3,4],[5,6]]

def test_single():
    assert merge([[1,5]]) == [[1,5]]

def test_unsorted_input():
    assert merge([[3,5],[1,2],[6,8]]) == [[1,2],[3,5],[6,8]]
```

### Complexity
| | Value |
|---|---|
| Time | O(n log n) — dominated by sort |
| Space | O(n) — output list |

### Trace Through
```
intervals (sorted) = [[1,3],[2,6],[8,10],[15,18]]
merged = [[1,3]]

[2,6]:  2 <= 3 → overlap → merged[-1][1] = max(3,6)=6  → [[1,6]]
[8,10]: 8 > 6  → new     →                              → [[1,6],[8,10]]
[15,18]:15>10  → new     →                              → [[1,6],[8,10],[15,18]] ✓
```

---

## 7. Find All Duplicates in an Array

### Problem
> Given an integer array `nums` of length `n` where all integers are in range `[1, n]` and each integer appears **once or twice**, return all integers that appear **twice**. Solve in O(n) time and O(1) extra space.

**Example:**
```
Input:  [4, 3, 2, 7, 8, 2, 3, 1]
Output: [2, 3]
```

### SDET Context
> Finding duplicate test IDs in a test suite, detecting duplicate event records in a log, or identifying duplicate transaction IDs in a data validation scenario — all without extra memory.

### Approach: Index as Visited Marker
Since values are in `[1, n]`, each value `v` maps to index `v-1`. Negate the value at that index to mark it as "visited". If it's already negative when we visit it — it's a duplicate.

```python
def find_duplicates(nums: list[int]) -> list[int]:
    duplicates = []

    for num in nums:
        index = abs(num) - 1          # map value to index (1-based → 0-based)

        if nums[index] < 0:           # already marked → duplicate found
            duplicates.append(abs(num))
        else:
            nums[index] = -nums[index]  # mark as visited by negating

    return duplicates


# ── Tests ──────────────────────────────────────────────────
def test_standard():
    assert sorted(find_duplicates([4,3,2,7,8,2,3,1])) == [2,3]

def test_no_duplicates():
    assert find_duplicates([1,2,3,4]) == []

def test_all_duplicates():
    assert sorted(find_duplicates([1,1,2,2])) == [1,2]

def test_single_duplicate():
    assert find_duplicates([1,2,3,2]) == [2]
```

### Complexity
| | Value |
|---|---|
| Time | O(n) — single pass |
| Space | O(1) — no extra data structure (output list aside) |

### Trace Through
```
nums = [4, 3, 2, 7, 8, 2, 3, 1]

num=4 → idx=3 → nums[3]=7>0  → negate → nums=[4,3,2,-7,8,2,3,1]
num=3 → idx=2 → nums[2]=2>0  → negate → nums=[4,3,-2,-7,8,2,3,1]
num=2 → idx=1 → nums[1]=3>0  → negate → nums=[4,-3,-2,-7,8,2,3,1]
num=-7→ idx=6 → nums[6]=3>0  → negate → nums=[4,-3,-2,-7,8,2,-3,1]
num=8 → idx=7 → nums[7]=1>0  → negate → nums=[4,-3,-2,-7,8,2,-3,-1]
num=2 → idx=1 → nums[1]=-3<0 → DUPLICATE → result=[2]
num=-3→ idx=2 → nums[2]=-2<0 → DUPLICATE → result=[2,3]
num=-1→ idx=0 → nums[0]=4>0  → negate
return [2, 3] ✓
```

---

## 8. Binary Search

### Problem
> Given a sorted array of integers `nums` and a `target`, return the index of `target` if it exists, or `-1` if it does not.

**Example:**
```
Input:  nums=[-1,0,3,5,9,12], target=9  → 4
Input:  nums=[-1,0,3,5,9,12], target=2  → -1
```

### SDET Context
> Searching sorted test IDs, log timestamps, or performance benchmark results. Binary search is also the mental model behind **bisect** module usage in test data management, and foundational for many SDET tooling problems.

### Approach: Iterative Binary Search
```python
def binary_search(nums: list[int], target: int) -> int:
    left, right = 0, len(nums) - 1

    while left <= right:
        mid = left + (right - left) // 2   # avoid overflow (matters in other languages)

        if nums[mid] == target:
            return mid
        elif nums[mid] < target:
            left = mid + 1    # target is in right half
        else:
            right = mid - 1   # target is in left half

    return -1


# Recursive version
def binary_search_recursive(nums: list[int], target: int,
                             left: int = 0, right: int = None) -> int:
    if right is None:
        right = len(nums) - 1
    if left > right:
        return -1

    mid = (left + right) // 2
    if nums[mid] == target:   return mid
    if nums[mid] < target:    return binary_search_recursive(nums, target, mid+1, right)
    return binary_search_recursive(nums, target, left, mid-1)


# ── Tests ──────────────────────────────────────────────────
def test_found():
    assert binary_search([-1,0,3,5,9,12], 9) == 4

def test_not_found():
    assert binary_search([-1,0,3,5,9,12], 2) == -1

def test_first_element():
    assert binary_search([1,2,3,4,5], 1) == 0

def test_last_element():
    assert binary_search([1,2,3,4,5], 5) == 4

def test_single_element_found():
    assert binary_search([5], 5) == 0

def test_empty():
    assert binary_search([], 1) == -1
```

### Complexity
| | Value |
|---|---|
| Time | O(log n) — halves search space each step |
| Space | O(1) iterative / O(log n) recursive (call stack) |

### Python Built-in: bisect
```python
import bisect

nums = [1, 3, 5, 7, 9]
bisect.bisect_left(nums, 5)    # 2 — leftmost insertion point
bisect.bisect_right(nums, 5)   # 3 — rightmost insertion point
bisect.insort(nums, 6)         # insert in sorted order → [1,3,5,6,7,9]
```

---

## 9. Maximum Subarray

### Problem
> Given an integer array `nums`, find the contiguous subarray with the **largest sum** and return its sum. (Kadane's Algorithm)

**Example:**
```
Input:  [-2,1,-3,4,-1,2,1,-5,4]
Output: 6   # subarray [4,-1,2,1]
```

### SDET Context
> Finding the peak performance window in a benchmark run, identifying the maximum throughput interval in load test metrics, or finding the longest streak of passing tests in a flaky test history.

### Approach: Kadane's Algorithm
At each position, decide: is it better to **extend** the current subarray or **start fresh** from this element?

```python
def max_subarray(nums: list[int]) -> int:
    current_sum = nums[0]
    max_sum     = nums[0]

    for num in nums[1:]:
        # Either extend the existing subarray or start a new one here
        current_sum = max(num, current_sum + num)
        max_sum     = max(max_sum, current_sum)

    return max_sum


# Extended: also return the subarray itself
def max_subarray_with_indices(nums: list[int]) -> tuple[int, list[int]]:
    current_sum = max_sum = nums[0]
    start = end = temp_start = 0

    for i, num in enumerate(nums[1:], start=1):
        if num > current_sum + num:
            current_sum = num
            temp_start  = i
        else:
            current_sum += num

        if current_sum > max_sum:
            max_sum = current_sum
            start   = temp_start
            end     = i

    return max_sum, nums[start:end+1]


# ── Tests ──────────────────────────────────────────────────
def test_standard():
    assert max_subarray([-2,1,-3,4,-1,2,1,-5,4]) == 6

def test_all_negative():
    assert max_subarray([-1,-2,-3,-4]) == -1   # pick least negative

def test_all_positive():
    assert max_subarray([1,2,3,4,5]) == 15

def test_single():
    assert max_subarray([5]) == 5

def test_with_indices():
    total, subarray = max_subarray_with_indices([-2,1,-3,4,-1,2,1,-5,4])
    assert total == 6
    assert subarray == [4,-1,2,1]
```

### Complexity
| | Value |
|---|---|
| Time | O(n) — single pass |
| Space | O(1) — only two variables |

### Trace Through
```
nums = [-2, 1, -3, 4, -1, 2, 1, -5, 4]

start: cur=-2, max=-2

num=1:  max(1, -2+1=-1)=1   → cur=1,  max=1
num=-3: max(-3, 1-3=-2)=-2  → cur=-2, max=1
num=4:  max(4, -2+4=2)=4    → cur=4,  max=4
num=-1: max(-1, 4-1=3)=3    → cur=3,  max=4
num=2:  max(2, 3+2=5)=5     → cur=5,  max=5
num=1:  max(1, 5+1=6)=6     → cur=6,  max=6  ← peak
num=-5: max(-5, 6-5=1)=1    → cur=1,  max=6
num=4:  max(4, 1+4=5)=5     → cur=5,  max=6

return 6 ✓
```

---

## 10. LRU Cache

### Problem
> Design a data structure that follows the **Least Recently Used (LRU) cache** eviction policy.
>
> Implement the `LRUCache` class:
> - `LRUCache(capacity)` — initialize with positive capacity
> - `get(key)` — return value if key exists, else `-1`
> - `put(key, value)` — insert/update key. If capacity exceeded, evict the least recently used key.
>
> Both operations must run in **O(1)** average time.

**Example:**
```
cache = LRUCache(2)
cache.put(1, 1)   # {1:1}
cache.put(2, 2)   # {1:1, 2:2}
cache.get(1)      # 1     — 1 is now MRU
cache.put(3, 3)   # evict key 2 → {1:1, 3:3}
cache.get(2)      # -1    — was evicted
cache.put(4, 4)   # evict key 1 → {3:3, 4:4}
cache.get(1)      # -1
cache.get(3)      # 3
cache.get(4)      # 4
```

### SDET Context
> Core design pattern for test result caching, API response mocking with bounded memory, token caching in auth test helpers, and session management in UI automation frameworks. Understanding LRU is essential for system design questions in senior SDET roles.

### Approach 1: OrderedDict (Python Pythonic)
`OrderedDict` maintains insertion order and supports `move_to_end()` — perfect for LRU.

```python
from collections import OrderedDict

class LRUCache:
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache    = OrderedDict()   # key → value, ordered by recency

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        self.cache.move_to_end(key)     # mark as most recently used
        return self.cache[key]

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self.cache.move_to_end(key)   # refresh recency
        self.cache[key] = value

        if len(self.cache) > self.capacity:
            self.cache.popitem(last=False)   # evict LRU (first/oldest item)
```

### Approach 2: Doubly Linked List + Hash Map (Full Implementation)
Shows understanding of the underlying O(1) mechanics — expected in senior interviews.

```python
class Node:
    def __init__(self, key=0, val=0):
        self.key  = key
        self.val  = val
        self.prev = None
        self.next = None

class LRUCache:
    """
    Layout: [dummy_head] ↔ [LRU ... MRU] ↔ [dummy_tail]
    head.next = LRU (evict from here)
    tail.prev = MRU (insert here)
    """
    def __init__(self, capacity: int):
        self.capacity = capacity
        self.cache    = {}              # key → Node

        # Sentinel nodes eliminate edge cases
        self.head = Node()
        self.tail = Node()
        self.head.next = self.tail
        self.tail.prev = self.head

    def _remove(self, node: Node) -> None:
        node.prev.next = node.next
        node.next.prev = node.prev

    def _insert_at_tail(self, node: Node) -> None:
        """Insert node just before tail (= MRU position)."""
        prev       = self.tail.prev
        prev.next  = node
        node.prev  = prev
        node.next  = self.tail
        self.tail.prev = node

    def get(self, key: int) -> int:
        if key not in self.cache:
            return -1
        node = self.cache[key]
        self._remove(node)
        self._insert_at_tail(node)      # move to MRU position
        return node.val

    def put(self, key: int, value: int) -> None:
        if key in self.cache:
            self._remove(self.cache[key])

        node = Node(key, value)
        self.cache[key] = node
        self._insert_at_tail(node)

        if len(self.cache) > self.capacity:
            lru = self.head.next        # LRU = first real node
            self._remove(lru)
            del self.cache[lru.key]


# ── Tests ──────────────────────────────────────────────────
def test_lru_standard():
    cache = LRUCache(2)
    cache.put(1, 1)
    cache.put(2, 2)
    assert cache.get(1) == 1       # 1 is now MRU
    cache.put(3, 3)                # evict 2
    assert cache.get(2) == -1      # 2 was evicted
    cache.put(4, 4)                # evict 1
    assert cache.get(1) == -1
    assert cache.get(3) == 3
    assert cache.get(4) == 4

def test_lru_update():
    cache = LRUCache(2)
    cache.put(1, 1)
    cache.put(1, 10)               # update existing key
    assert cache.get(1) == 10

def test_lru_capacity_one():
    cache = LRUCache(1)
    cache.put(1, 1)
    cache.put(2, 2)                # evict 1
    assert cache.get(1) == -1
    assert cache.get(2) == 2

def test_lru_get_refreshes_order():
    cache = LRUCache(2)
    cache.put(1, 1)
    cache.put(2, 2)
    cache.get(1)                   # 1 becomes MRU → 2 is now LRU
    cache.put(3, 3)                # evict 2 (LRU), not 1
    assert cache.get(2) == -1
    assert cache.get(1) == 1
```

### Complexity
| Operation | OrderedDict | DLL + HashMap |
|---|---|---|
| get | O(1) avg | O(1) |
| put | O(1) avg | O(1) |
| Space | O(capacity) | O(capacity) |

---

## Quick Reference Summary

| # | Problem | Key Data Structure | Core Idea |
|---|---|---|---|
| 1 | Two Sum | Hash Map | Store complement, look up on the fly |
| 2 | Valid Parentheses | Stack | Match closing to last unmatched opening |
| 3 | Longest Substring No Repeat | Sliding Window + Map | Shrink left when duplicate enters window |
| 4 | Group Anagrams | Hash Map | Sorted word as canonical key |
| 5 | Valid Anagram | Counter / Hash Map | Frequency equality |
| 6 | Merge Intervals | Sort + Greedy | Sort by start, extend or append |
| 7 | Find All Duplicates | In-place Negation | Use value as index, sign as visited flag |
| 8 | Binary Search | Two Pointers | Halve the search space each step |
| 9 | Maximum Subarray | Kadane's | Extend or restart at each element |
| 10 | LRU Cache | OrderedDict / DLL+Map | Hash for O(1) lookup, list for O(1) order |

---

## SDET Interview Tips

**On writing tests:** Interviewers notice when you write tests for your own solution. Always include edge cases: empty input, single element, all-same values, negative numbers, and boundary conditions.

**On complexity:** Always state time AND space complexity. "I can trade O(n) space for O(n) time here" signals engineering maturity.

**On Python shortcuts:** Know when to use `Counter`, `defaultdict`, `OrderedDict`, and `bisect`. These aren't cheating — they show Python fluency that is directly relevant to writing efficient automation code.

**On the LRU Cache:** If asked about system design for test infrastructure (e.g., API mock caching, token reuse), connecting it back to LRU demonstrates senior-level thinking.

---

*Last updated: April 2026 | Python 3.10+ | LeetCode #1, #20, #3, #49, #242, #56, #442, #704, #53, #146*
