# Two Pointers

## What Is It?

The Two Pointers pattern processes linear data structures (arrays, strings, linked lists) by using two integer variables (pointers) to traverse the data simultaneously. The core philosophy is **search space reduction** — by leveraging the order or structure of the data, you intelligently move the pointers to skip unnecessary comparisons, reducing an O(N^2) or O(N^3) brute-force down to O(N) or O(N log N).

## When to Use It

- **The data is linear** — Array, String, or Linked List
- **Order matters** — The data is sorted, or you're looking for subarrays, palindromes, or specific sequential patterns
- **Pairs or boundaries are involved** — Finding two numbers that sum to a target, reversing elements, or tracking boundaries of a partition
- **O(1) auxiliary space is required** — In-place modifications

## Real-Life Analogies

| Analogy | Maps to |
|---------|---------|
| **Organizing a bookshelf** — Scan books one-by-one (read pointer); every time you find an unread book, place it at the next available slot on the left (write pointer) | Read/Write pattern |
| **Finding a meeting time** — You and a colleague both have sorted free-time lists. Whoever's slot ends earlier moves to their next slot | Two pointers on two lists |
| **Traffic flow** — A sensor counts cars in the fast lane for every one car in the slow lane | Fast/Slow spacing |

---

## Archetypes

### Group 1: Opposite Direction (Converging / Symmetry)

**Problems**: 15, 151, 246, 2824, 977, 344, 2193

**Interview logistics**: Place `left` at index 0 and `right` at the end. Maintain a strict invariant — evaluate the condition and move pointers inward based on magnitude or a matching condition. The loop terminates when `left >= right`. If the data isn't sorted and magnitude matters (like 3Sum), sort first.

### Group 2: Same Direction (Read / Write / Partition)

**Problems**: 27, 283, 443, 75

**Interview logistics**: Both pointers start at 0. The fast (read) pointer scans unconditionally. The slow (write) pointer tracks the boundary of "processed" valid data. Only advance the write pointer when the read pointer finds an element that belongs in the processed section. For partitioning (like Sort Colors), use three pointers for three distinct regions.

### Group 3: Two Pointers on Two Iterables (Matching / Subsequence)

**Problems**: 408, 2486, 1537, 321

**Interview logistics**: Place `pointer_a` on the first string/array and `pointer_b` on the second. Usually matching a subsequence or merging sorted data. Advance `pointer_a` to scan the main data, advance `pointer_b` *only* on a successful match. If `pointer_b` reaches the end, you've found the subsequence.

### Group 4: Linked List & Reset Tricks

**Problems**: 19, 160, 1650

**Interview logistics**: You can't go backward in a singly linked list. To find the Nth node from the end, give the fast pointer an N-step head start, then move both together. For intersection of two lists with different lengths, when a pointer reaches the end of its list, reset it to the head of the *other* list to equalize traversal distances.

### Group 5: Boundary Expansion & Suffix Scanning (Greedy)

**Problems**: 763, 31, 1842, 189, 2444

**Interview logistics**: The hardest tier. Instead of simple convergence, you're manipulating boundaries. For string partitions (763), track the maximum last occurrence of characters to define the "cut". For permutations (31, 1842), scan right-to-left to find the first decrease (the pivot), swap it, and reverse the suffix.

---

## Problem Checklist

### 1. LC 15 — 3Sum `Medium` (Group 1: Converging)

**The spiel**: "To avoid an O(N^3) brute-force approach, I will sort the array first. This allows me to fix one number and reduce the remaining problem to a standard Two-Sum on a sorted array. For each fixed element, I'll use converging left and right pointers to find pairs that sum to the negative of my fixed element. Crucially, I will skip duplicate values for both my fixed element and my moving pointers to avoid duplicate triplets."

**Complexity**: Time O(N^2) — sorting is O(N log N) plus N iterations of an O(N) two-pointer search. Space O(1) to O(N) depending on the sort.

```python
def threeSum(nums: list[int]) -> list[list[int]]:
    res = []
    nums.sort()  # Sorting unlocks the two-pointer logic

    for i in range(len(nums)):
        # Skip duplicate fixed numbers
        if i > 0 and nums[i] == nums[i - 1]:
            continue

        l, r = i + 1, len(nums) - 1
        while l < r:
            total = nums[i] + nums[l] + nums[r]
            if total > 0:
                r -= 1
            elif total < 0:
                l += 1
            else:
                res.append([nums[i], nums[l], nums[r]])
                l += 1
                r -= 1
                # Skip duplicate left values to prevent duplicate triplets
                while l < r and nums[l] == nums[l - 1]:
                    l += 1

    return res
```

---

### 2. LC 977 — Squares of a Sorted Array `Easy` (Group 1: Converging)

**The spiel**: "Since the array is sorted but contains negative numbers, the largest squares will always be at the far edges (either the smallest negative or largest positive). I will initialize two pointers at the ends of the array. I'll compare the absolute values, square the larger one, and write it into a result array from right to left (filling the largest values first)."

**Complexity**: Time O(N), Space O(N) for the result.

```python
def sortedSquares(nums: list[int]) -> list[int]:
    n = len(nums)
    res = [0] * n
    l, r = 0, n - 1

    # Fill the result array from back to front
    for i in range(n - 1, -1, -1):
        if abs(nums[l]) > abs(nums[r]):
            res[i] = nums[l] ** 2
            l += 1
        else:
            res[i] = nums[r] ** 2
            r -= 1

    return res
```

---

### 3. LC 283 — Move Zeroes `Easy` (Group 2: Read/Write)

**The spiel**: "I will use a Read/Write two-pointer approach to modify the array in place. The left pointer tracks the next position for a non-zero element. The right pointer scans the array. Whenever the right pointer finds a non-zero element, I swap it with the element at the left pointer and increment left. This maintains relative order and naturally pushes zeroes to the end."

**Complexity**: Time O(N), Space O(1).

```python
def moveZeroes(nums: list[int]) -> None:
    left = 0  # Write pointer

    for right in range(len(nums)):  # Read pointer
        if nums[right] != 0:
            # Swap non-zero to the front
            nums[left], nums[right] = nums[right], nums[left]
            left += 1
```

---

### 4. LC 75 — Sort Colors `Medium` (Group 2: Three-Way Partition)

**The spiel**: "This is the Dutch National Flag problem. I'll maintain three regions using three pointers: `low` for 0s, `mid` as the current scanner, and `high` for 2s. If `mid` finds a 0, I swap it with `low` and move both forward. If it finds a 1, it's already in the correct middle region, so I just move `mid` forward. If it finds a 2, I swap it with `high` and decrement `high`, but I do *not* move `mid` because the swapped element needs to be evaluated."

**Complexity**: Time O(N), Space O(1).

```python
def sortColors(nums: list[int]) -> None:
    low, mid, high = 0, 0, len(nums) - 1

    while mid <= high:
        if nums[mid] == 0:
            nums[low], nums[mid] = nums[mid], nums[low]
            low += 1
            mid += 1
        elif nums[mid] == 1:
            mid += 1
        else:  # nums[mid] == 2
            nums[mid], nums[high] = nums[high], nums[mid]
            high -= 1
```

---

### 5. LC 2486 — Append Characters to String to Make Subsequence `Medium` (Group 3: Two Iterables)

**The spiel**: "To find the minimum characters to append, I need to find the longest matching prefix of string `t` that exists as a subsequence in string `s`. I use two pointers: one iterating through `s` and one through `t`. I greedily advance the `s` pointer, and only advance the `t` pointer when characters match. The remaining characters in `t` when `s` is exhausted is my answer."

**Complexity**: Time O(N), Space O(1).

```python
def appendCharacters(s: str, t: str) -> int:
    i, j = 0, 0  # i for s, j for t

    while i < len(s) and j < len(t):
        if s[i] == t[j]:
            j += 1  # Match found, advance target pointer
        i += 1  # Always advance source pointer

    # The remaining characters in t need to be appended
    return len(t) - j
```

---

### 6. LC 160 — Intersection of Two Linked Lists `Easy` (Group 4: Reset Trick)

**The spiel**: "To find the intersection without extra space, I need to account for the difference in lengths. I use two pointers starting at the respective heads. When a pointer reaches the end of its list, I redirect it to the head of the *other* list. Both pointers will travel the exact same total distance (Length A + Length B). They are mathematically guaranteed to collide at the intersection node on their second pass, or at `None` if no intersection exists."

**Complexity**: Time O(N+M), Space O(1).

```python
def getIntersectionNode(headA, headB):
    if not headA or not headB:
        return None

    ptrA, ptrB = headA, headB

    # If they intersect, they will meet. If not, both hit None simultaneously.
    while ptrA != ptrB:
        ptrA = ptrA.next if ptrA else headB
        ptrB = ptrB.next if ptrB else headA

    return ptrA
```

---

### 7. LC 19 — Remove Nth Node From End of List `Medium` (Group 4: Spaced Pointers)

**The spiel**: "To solve this in one pass, I'll use a fast and slow pointer with a fixed spacing. I introduce a dummy node to handle edge cases where the head itself needs to be removed. I advance the fast pointer N+1 steps ahead. Then, I move both together until fast hits the end. At this point, slow is positioned exactly before the node to delete."

**Complexity**: Time O(N), Space O(1).

```python
def removeNthFromEnd(head, n: int):
    # Dummy node handles the case where head is removed
    dummy = ListNode(0, head)
    slow = fast = dummy

    # Give fast pointer an n+1 step head start
    for _ in range(n + 1):
        fast = fast.next

    # Move both until fast reaches the end
    while fast:
        slow = slow.next
        fast = fast.next

    # Skip the nth node
    slow.next = slow.next.next

    return dummy.next
```

---

### 8. LC 763 — Partition Labels `Medium` (Group 5: Boundary Expansion)

**The spiel**: "The invariant is that all occurrences of a character must appear in the same partition. First, I iterate through the string to record the last index of every character. Then, I iterate again with a two-pointer window. I update the window's end boundary to the maximum last occurrence of characters I'm looking at. When my current index `i` reaches `end`, the window is self-contained — I record the partition length and start a new window."

**Complexity**: Time O(N), Space O(1) (hash map bounded by 26 letters).

```python
def partitionLabels(s: str) -> list[int]:
    # Record the last occurrence of each character
    last_idx = {char: i for i, char in enumerate(s)}

    res = []
    size, end = 0, 0

    for i, char in enumerate(s):
        size += 1
        # Expand the boundary to encompass the current character's last appearance
        end = max(end, last_idx[char])

        # When we reach the boundary, cut the partition
        if i == end:
            res.append(size)
            size = 0

    return res
```

---

### 9. LC 31 — Next Permutation `Medium` (Group 5: Suffix Scanning)

**The spiel**: "To find the next lexicographical permutation, I find the first point from the right where ascending order breaks — the 'pivot'. Then I scan from right to left again to find the smallest number strictly greater than the pivot. I swap them. Finally, because the suffix after the pivot is in descending order, I reverse it to make it ascending, giving the smallest possible next permutation."

**Complexity**: Time O(N), Space O(1).

```python
def nextPermutation(nums: list[int]) -> None:
    i = len(nums) - 2

    # 1. Find the pivot (first decrease from right to left)
    while i >= 0 and nums[i] >= nums[i + 1]:
        i -= 1

    if i >= 0:
        # 2. Find the rightmost successor to swap with
        j = len(nums) - 1
        while nums[j] <= nums[i]:
            j -= 1
        nums[i], nums[j] = nums[j], nums[i]

    # 3. Reverse the suffix to get the smallest lexicographical order
    left, right = i + 1, len(nums) - 1
    while left < right:
        nums[left], nums[right] = nums[right], nums[left]
        left += 1
        right -= 1
```

---

### 10. LC 2193 — Minimum Moves to Make Palindrome `Hard` (Group 5: Greedy Symmetrical Swap)

**The spiel**: "I use two pointers at the ends of the string to greedily construct the palindrome. I check if `s[left] == s[right]`. If not, I scan inward from right to find the closest matching character to `s[left]`. Once found, I simulate bubble-swapping it to the right position. If a character has no match (it belongs in the middle of an odd-length palindrome), I swap it toward the center."

**Complexity**: Time O(N^2) (inner scan and bubble swaps), Space O(N) (string converted to list).

```python
def minMovesToMakePalindrome(s: str) -> int:
    s = list(s)
    res = 0
    left, right = 0, len(s) - 1

    while left < right:
        if s[left] != s[right]:
            # Scan inward from the right to find the match for s[left]
            k = right
            while k > left and s[k] != s[left]:
                k -= 1

            if k == left:  # No match found — it's the center character
                s[left], s[left + 1] = s[left + 1], s[left]
                res += 1
                continue  # Do not advance left/right pointers yet

            # Bubble swap the matched character to the 'right' boundary
            while k < right:
                s[k], s[k + 1] = s[k + 1], s[k]
                k += 1
                res += 1

        left += 1
        right -= 1

    return res
```
